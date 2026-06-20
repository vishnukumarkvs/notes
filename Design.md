Design
======

~~~
package l7

import (
	"bytes"
	"io"
	"unicode/utf8"

	"github.com/ClickHouse/ch-go/proto"
)

func safeStr(r *proto.Reader, maxLen int) (string, error) {
	l, err := r.StrLen()
	if err != nil {
		return "", err
	}
	if l > maxLen {
		return "", io.ErrUnexpectedEOF
	}
	buf := make([]byte, l)
	err = r.ReadFull(buf)
	return string(buf), err
}

func safeDecodeSetting(r *proto.Reader) (string, error) {
	key, err := safeStr(r, 1024)
	if err != nil {
		return "", err
	}
	if key == "" {
		return "", nil
	}
	if _, err := r.UVarInt(); err != nil {
		return "", err
	}
	if _, err := safeStr(r, 4096); err != nil {
		return "", err
	}
	return key, nil
}

func safeDecodeClientInfo(r *proto.Reader, version int) (int, error) {
	if _, err := safeStr(r, 1024); err != nil { // initial_user
		return 0, err
	}
	if _, err := safeStr(r, 1024); err != nil { // initial_query_id
		return 0, err
	}
	if _, err := safeStr(r, 1024); err != nil { // initial_address
		return 0, err
	}
	if proto.FeatureQueryStartTime.In(version) { // initial_time (int64)
		if _, err := r.Int64(); err != nil {
			return 0, err
		}
	}
	if _, err := r.UInt8(); err != nil { // interface (byte)
		return 0, err
	}
	if _, err := safeStr(r, 1024); err != nil { // os_user
		return 0, err
	}
	if _, err := safeStr(r, 1024); err != nil { // client_hostname
		return 0, err
	}
	if _, err := safeStr(r, 1024); err != nil { // client_name
		return 0, err
	}
	if _, err := r.UVarInt(); err != nil { // major
		return 0, err
	}
	if _, err := r.UVarInt(); err != nil { // minor
		return 0, err
	}
	pv, err := r.UVarInt() // protocol_version
	if err != nil {
		return 0, err
	}
	protocolVersion := int(pv)
	if proto.FeatureQuotaKeyInClientInfo.In(version) { // quota_key
		if _, err := safeStr(r, 1024); err != nil {
			return 0, err
		}
	}
	if proto.FeatureDistributedDepth.In(version) { // distributed_depth
		if _, err := r.UVarInt(); err != nil {
			return 0, err
		}
	}
	if proto.FeatureVersionPatch.In(version) { // version_patch (interface always TCP)
		if _, err := r.UVarInt(); err != nil {
			return 0, err
		}
	}
	if proto.FeatureOpenTelemetry.In(version) {
		hasTrace, err := r.Bool()
		if err != nil {
			return 0, err
		}
		if hasTrace {
			if _, err := r.ReadRaw(16); err != nil { // trace_id
				return 0, err
			}
			if _, err := r.ReadRaw(8); err != nil { // span_id
				return 0, err
			}
			if _, err := safeStr(r, 1024); err != nil { // trace_state
				return 0, err
			}
			if _, err := r.Byte(); err != nil { // trace_flags
				return 0, err
			}
		}
	}
	if proto.FeatureParallelReplicas.In(version) {
		if _, err := r.UVarInt(); err != nil { // collaborate_with_initiator
			return 0, err
		}
		if _, err := r.UVarInt(); err != nil { // count_participating_replicas
			return 0, err
		}
		if _, err := r.UVarInt(); err != nil { // number_of_current_replica
			return 0, err
		}
	}
	return protocolVersion, nil
}

func ParseClickhouse(payload []byte) (query string) {
	defer func() {
		if recover() != nil {
			query = ""
		}
	}()
	r := proto.NewReader(bytes.NewReader(payload))
	var err error
	if _, err = r.Byte(); err != nil {
		return ""
	}
	if _, err = safeStr(r, 1024); err != nil {
		return ""
	}
	version := int(proto.FeatureServerQueryTimeInProgress)
	protocolVersion, err := safeDecodeClientInfo(r, version)
	if err != nil {
		return ""
	}
	if protocolVersion > 0 {
		if protocolVersion > version {
			return ""
		}
		version = protocolVersion
	}
	for {
		key, err := safeDecodeSetting(r)
		if err != nil {
			return ""
		}
		if key == "" {
			break
		}
	}
	if _, err = safeStr(r, 1024); err != nil { // inter-server secret
		return ""
	}
	if stage, err := r.UVarInt(); err != nil { // stage
		return ""
	} else if stage > 2 { // invalid stage
		return ""
	}
	if c, err := r.UVarInt(); err != nil { // compression
		return ""
	} else if c > 1 { // invalid compression
		return ""
	}
	l, err := r.StrLen()
	if err != nil {
		return ""
	}
	buf := make([]byte, min(l, 1024))
	n, _ := r.Read(buf)
	buf = bytes.TrimSpace(buf[:n])
	if len(buf) == 0 {
		return ""
	}
	if !utf8.Valid(buf) { // not a real query: misclassified or corrupted payload
		return ""
	}
	if n < l {
		buf = append(buf[:len(buf)-1], []byte("...<TRUNCATED>")...)
	}
	return string(buf)
}

~~~