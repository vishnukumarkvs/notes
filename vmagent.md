vmagent
=======

# Navigate to the monorepo root
cd VictoriaMetrics

# Cross-compile a pure, statically-linked binary for Linux AMD64
CGO_ENABLED=0 GOOS=linux GOARCH=amd64 go build -mod=vendor ./app/vmagent/




