coroot savejob
==============

~~~
package collector

import (
	"bufio"
	"bytes"
	"crypto/tls"
	"errors"
	"fmt"
	"io"
	"net/http"
	"net/url"
	"sort"
	"sync"
	"time"

	"github.com/ClickHouse/ch-go"
	chproto "github.com/ClickHouse/ch-go/proto"
	"github.com/gogo/protobuf/proto"
	"github.com/golang/snappy"
	promModel "github.com/prometheus/common/model"
	"github.com/prometheus/prometheus/prompb"
	"k8s.io/klog"
)

var (
	secureClient = &http.Client{Transport: &http.Transport{
		TLSHandshakeTimeout: 10 * time.Second,
	}}
	insecureClient = &http.Client{Transport: &http.Transport{
		TLSHandshakeTimeout: 10 * time.Second,
		TLSClientConfig:     &tls.Config{InsecureSkipVerify: true},
	}}
)

func addLabelsIfNeeded(r *http.Request, body []byte, extraLabels map[string]string) ([]byte, error) {
	if len(extraLabels) == 0 {
		return body, nil
	}
	req, err := parseMetricsRequestBody(r, body)
	if err != nil {
		return nil, err
	}
	for i := range req.Timeseries {
		for k, v := range extraLabels {
			req.Timeseries[i].Labels = append(req.Timeseries[i].Labels, prompb.Label{Name: k, Value: v})
		}
	}
	decompressed, err := proto.Marshal(req)
	if err != nil {
		return nil, err
	}
	return snappy.Encode(nil, decompressed), nil
}

func (c *Collector) Metrics(w http.ResponseWriter, r *http.Request) {
	project, err := c.getProject(r.Header.Get(ApiKeyHeader))
	if err != nil {
		klog.Errorln(err)
		if errors.Is(err, ErrProjectNotFound) {
			http.Error(w, err.Error(), http.StatusNotFound)
			return
		}
		http.Error(w, "", http.StatusInternalServerError)
		return
	}
	cfg := project.PrometheusConfig(c.globalPrometheus)

	body, err := io.ReadAll(r.Body)
	if err != nil {
		klog.Errorln(err)
		http.Error(w, "", http.StatusBadRequest)
	}
	if cfg.UseClickHouse {
		req, err := parseMetricsRequestBody(r, body)
		if err != nil {
			klog.Errorln(err)
			http.Error(w, "", http.StatusBadRequest)
			return
		}
		c.getMetricsBatch(project).Add(req)
		return
	}

	var u *url.URL
	if cfg.RemoteWriteUrl == "" {
		u, err = url.Parse(cfg.Url)
		if err != nil {
			klog.Errorln(err)
			http.Error(w, "", http.StatusInternalServerError)
			return
		}
		u = u.JoinPath("/api/v1/write")
	} else {
		u, err = url.Parse(cfg.RemoteWriteUrl)
		if err != nil {
			klog.Errorln(err)
			http.Error(w, "", http.StatusInternalServerError)
			return
		}
	}

	if cfg.BasicAuth != nil {
		u.User = url.UserPassword(cfg.BasicAuth.User, cfg.BasicAuth.Password)
	}
	body, err = addLabelsIfNeeded(r, body, cfg.ExtraLabels)
	if err != nil {
		klog.Errorln(err)
		http.Error(w, "", http.StatusBadRequest)
		return
	}

	req, err := http.NewRequestWithContext(r.Context(), r.Method, u.String(), bytes.NewReader(body))
	if err != nil {
		klog.Errorln(err)
		http.Error(w, "", http.StatusInternalServerError)
		return
	}

	for _, h := range cfg.CustomHeaders {
		req.Header.Add(h.Key, h.Value)
	}
	for k, vs := range r.Header {
		if k == ApiKeyHeader {
			continue
		}
		for _, v := range vs {
			req.Header.Add(k, v)
		}
	}
	httpClient := secureClient
	if cfg.TlsSkipVerify {
		httpClient = insecureClient
	}
	res, err := httpClient.Do(req)
	if err != nil {
		klog.Errorln(err)
		http.Error(w, "", http.StatusInternalServerError)
		return
	}
	defer func() {
		io.Copy(io.Discard, res.Body)
		res.Body.Close()
	}()
	for k, vs := range res.Header {
		for _, v := range vs {
			w.Header().Add(k, v)
		}
	}
	if res.StatusCode == http.StatusBadRequest {
		scanner := bufio.NewScanner(io.LimitReader(res.Body, 1024))
		line := ""
		if scanner.Scan() {
			line = scanner.Text()
		}
		klog.Errorf("failed to write: got %d (%s) from prometheus, responding to the agent with 200 (to prevent retry)", res.StatusCode, line)
		w.WriteHeader(http.StatusOK)
		return
	} else if res.StatusCode > 400 {
		klog.Errorf("failed to write: got %d from prometheus", res.StatusCode)
	}
	w.WriteHeader(res.StatusCode)
	_, _ = io.Copy(w, res.Body)
}

func parseMetricsRequestBody(r *http.Request, body []byte) (*prompb.WriteRequest, error) {
	if r.Header.Get("Content-Type") != "application/x-protobuf" {
		return nil, fmt.Errorf("expected application/x-protobuf content-type")
	}
	if r.Header.Get("Content-Encoding") != "snappy" {
		return nil, fmt.Errorf("expected snappy content-encoding")
	}

	decompressed, err := snappy.Decode(nil, body)
	if err != nil {
		return nil, err
	}

	var req prompb.WriteRequest
	if err = proto.Unmarshal(decompressed, &req); err != nil {
		return nil, err
	}

	return &req, nil
}

type saveJob struct {
	Timestamp        *chproto.ColDateTime64
	MetricHash       *chproto.ColUInt64
	Value            *chproto.ColFloat64
	MetricName       *chproto.ColLowCardinality[string]
	Labels           *chproto.ColMap[string, string]
	MetricFamilyName *chproto.ColLowCardinality[string]
	Type             *chproto.ColLowCardinality[string]
	Help             *chproto.ColStr
	Unit             *chproto.ColLowCardinality[string]
}

type MetricsBatch struct {
	limit int
	exec  func(query ch.Query) error

	lock sync.Mutex
	done chan struct{}

	Timestamp  *chproto.ColDateTime64
	MetricHash *chproto.ColUInt64
	Value      *chproto.ColFloat64
	MetricName *chproto.ColLowCardinality[string]
	Labels     *chproto.ColMap[string, string]

	MetricFamilyName *chproto.ColLowCardinality[string]
	Type             *chproto.ColLowCardinality[string]
	Help             *chproto.ColStr
	Unit             *chproto.ColLowCardinality[string]

	saveCh chan *saveJob
}

func NewMetricsBatch(limit int, timeout time.Duration, exec func(query ch.Query) error) *MetricsBatch {
	b := &MetricsBatch{
		limit: limit,
		exec:  exec,
		done:  make(chan struct{}),

		Timestamp:  new(chproto.ColDateTime64).WithPrecision(chproto.PrecisionMilli),
		MetricHash: new(chproto.ColUInt64),
		Value:      new(chproto.ColFloat64),
		MetricName: new(chproto.ColStr).LowCardinality(),
		Labels:     chproto.NewMap[string, string](new(chproto.ColStr).LowCardinality(), new(chproto.ColStr)),

		MetricFamilyName: new(chproto.ColStr).LowCardinality(),
		Type:             new(chproto.ColStr).LowCardinality(),
		Help:             new(chproto.ColStr),
		Unit:             new(chproto.ColStr).LowCardinality(),

		saveCh: make(chan *saveJob, 1),
	}

	go b.saveLoop()

	go func() {
		ticker := time.NewTicker(timeout)
		defer ticker.Stop()
		for {
			select {
			case <-b.done:
				return
			case <-ticker.C:
				b.lock.Lock()
				if b.Timestamp.Rows() == 0 {
					b.lock.Unlock()
					continue
				}
				job := b.newSaveJob()
				b.lock.Unlock()
				select {
				case b.saveCh <- job:
				case <-b.done:
				}
			}
		}
	}()

	return b
}

func (b *MetricsBatch) newSaveJob() *saveJob {
	job := &saveJob{
		Timestamp:        b.Timestamp,
		MetricHash:       b.MetricHash,
		Value:            b.Value,
		MetricName:       b.MetricName,
		Labels:           b.Labels,
		MetricFamilyName: b.MetricFamilyName,
		Type:             b.Type,
		Help:             b.Help,
		Unit:             b.Unit,
	}
	b.Timestamp = new(chproto.ColDateTime64).WithPrecision(chproto.PrecisionMilli)
	b.MetricHash = new(chproto.ColUInt64)
	b.Value = new(chproto.ColFloat64)
	b.MetricName = new(chproto.ColStr).LowCardinality()
	b.Labels = chproto.NewMap[string, string](new(chproto.ColStr).LowCardinality(), new(chproto.ColStr))
	b.MetricFamilyName = new(chproto.ColStr).LowCardinality()
	b.Type = new(chproto.ColStr).LowCardinality()
	b.Help = new(chproto.ColStr)
	b.Unit = new(chproto.ColStr).LowCardinality()
	return job
}

func (b *MetricsBatch) saveLoop() {
	for {
		select {
		case <-b.done:
			return
		case job := <-b.saveCh:
			b.saveJob(job)
		}
	}
}

func (b *MetricsBatch) saveJob(job *saveJob) {
	input := chproto.Input{
		chproto.InputColumn{Name: "MetricName", Data: job.MetricName},
		chproto.InputColumn{Name: "Labels", Data: job.Labels},
		chproto.InputColumn{Name: "Timestamp", Data: job.Timestamp},
		chproto.InputColumn{Name: "MetricHash", Data: job.MetricHash},
		chproto.InputColumn{Name: "Value", Data: job.Value},
	}
	if err := b.exec(ch.Query{Body: input.Into("@@table_metrics@@"), Input: input}); err != nil {
		klog.Errorln("failed to insert metrics:", err)
	}

	if job.MetricFamilyName.Rows() > 0 {
		input = chproto.Input{
			chproto.InputColumn{Name: "MetricFamilyName", Data: job.MetricFamilyName},
			chproto.InputColumn{Name: "Type", Data: job.Type},
			chproto.InputColumn{Name: "Help", Data: job.Help},
			chproto.InputColumn{Name: "Unit", Data: job.Unit},
		}
		if err := b.exec(ch.Query{Body: input.Into("@@table_metrics_metadata@@"), Input: input}); err != nil {
			klog.Errorln("failed to insert metrics metadata:", err)
		}
	}
}

func (b *MetricsBatch) Close() {
	close(b.done)
	b.lock.Lock()
	if b.Timestamp.Rows() > 0 {
		job := b.newSaveJob()
		b.saveJob(job)
	}
	b.lock.Unlock()
	for {
		select {
		case job := <-b.saveCh:
			b.saveJob(job)
		default:
			return
		}
	}
}

func (b *MetricsBatch) Add(req *prompb.WriteRequest) {
	b.lock.Lock()

	for _, md := range req.GetMetadata() {
		b.MetricFamilyName.Append(md.GetMetricFamilyName())
		b.Type.Append(md.GetType().String())
		b.Help.Append(md.GetHelp())
		b.Unit.Append(md.GetUnit())
	}

	for _, ts := range req.GetTimeseries() {
		labels := make(map[string]string, len(ts.Labels))
		sortable := make([]chproto.KV[string, string], 0, len(ts.Labels))
		var metricName string
		for _, label := range ts.Labels {
			if label.Name == promModel.MetricNameLabel {
				metricName = label.Value
			} else {
				sortable = append(sortable, chproto.KV[string, string]{Key: label.Name, Value: label.Value})
			}
			labels[label.Name] = label.Value
		}
		sort.Slice(sortable, func(i, j int) bool {
			return sortable[i].Key < sortable[j].Key
		})
		hash := promModel.LabelsToSignature(labels)
		for _, sample := range ts.Samples {
			b.MetricName.Append(metricName)
			b.Labels.AppendKV(sortable)
			b.Timestamp.Append(time.Unix(sample.Timestamp/1000, 0))
			b.MetricHash.Append(hash)
			b.Value.Append(sample.Value)

		}
		delete(labels, promModel.MetricNameLabel)
	}

	if b.Timestamp.Rows() < b.limit {
		b.lock.Unlock()
		return
	}
	job := b.newSaveJob()
	b.lock.Unlock()

	b.saveCh <- job
}

~~~\\



Remaining concerns (not blocking, but worth noting)
1. Context still TODO() — collector.go:285 passes context.TODO() to ClickHouse inserts. If ClickHouse hangs, the shard goroutine blocks and its buffer fills. Adding context.WithTimeout would prevent the shard from wedging.
2. Silent drops — when channel is full, agent gets 200 OK but data is lost. You could optionally return 503 when all shards are full to let the agent retry.
3. io.ReadAll unbounded — collector/metrics.go:413 reads entire request body with no size limit. A http.MaxBytesReader wrapper would be safer.
4. Shard count hardcoded to 8 — fine for 370 nodes, but configurable would be better for flexibility.
Better approach?
The current design is already well-optimized. The only architectural improvement I'd suggest is adding a timeout to the ClickHouse exec call (point 1 above) — that's the single remaining place where a slow operation can stall a shard. Everything else is solid.

