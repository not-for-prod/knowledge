# Buf Schema Registry

## Prerequisites

Install: https://buf.build/docs/cli/installation/

## Usage

`buf.yaml`

```yaml
version: v2
modules:
  - path: api
deps:
  - buf.build/googleapis/googleapis
  - buf.build/grpc-ecosystem/grpc-gateway
  - buf.build/bufbuild/protovalidate
lint:
  use:
    - STANDARD
breaking:
  use:
    - FILE
```

`buf.gen.yaml` with `swagger`, `grpc-gateway` (HTTP adapter for GRPC servers)

```yaml
version: v2
managed:
  enabled: true
  disable:
    - module: buf.build/protocolbuffers/go
    - module: buf.build/googleapis/googleapis
    - module: buf.build/grpc/go
    - module: buf.build/bufbuild/protovalidate
    - module: buf.build/grpc-ecosystem/grpc-gateway
  override:
    - file_option: go_package_prefix
      value: <go mod module + path for pb>
plugins:
  - remote: buf.build/protocolbuffers/go
    out: internal/pkg/pb
    opt:
      - paths=source_relative
  - remote: buf.build/grpc/go
    out: internal/pkg/pb
    opt:
      - paths=source_relative
      - require_unimplemented_servers=false
  - remote: buf.build/grpc-ecosystem/gateway
    opt:
      - generate_unbound_methods=false
      - logtostderr=true
      - paths=source_relative
    out: internal/pkg/pb
  - remote: buf.build/grpc-ecosystem/openapiv2
    out: internal/pkg/pb
    opt:
      - generate_unbound_methods=false
      - fqn_for_openapi_name=true
  - local: protoc-gen-goclay
    out: internal/pkg/pb
    opt:
      - paths=source_relative
```

Makefile:

```
.PHONY: proto
proto:
	buf dep update
	buf dep prune
	buf lint
	#buf breaking --against ".git#subdir=."
	buf generate
```