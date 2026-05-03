crio
====

Prerequisites
- Go (check go.mod for required version), basic knowledge of Kubernetes CRI, OCI specs, Linux containers (namespaces/cgroups), gRPC
- Tools: make, git, optional bats (integration tests)
Understanding Roadmap
1. Project Overview: Read README.md, AGENTS.md (full context), tutorial.md, roadmap.md
2. Core Architecture: Study cmd/crio/main.go (entry point), server/server.go (CRI gRPC server), explore internal/ packages (lib/, oci/, storage/, config/)
3. Implementation Details: Read internal/oci/runtime_oci.go, internal/lib/container_server.go, pkg/config/config.go, server/sandbox_run_linux.go
4. Testing: Read CONTRIBUTING.md, test/ctr.bats, run make testunit (unit tests)
5. Workflow: Review Makefile (make help), dependencies.yaml, run make lint/make prettier (required checks)


