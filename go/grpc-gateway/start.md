# gRPC-Gateway

> To Install Buf CLI [Follow the instruction](../../proto/buf.md)

## Dependencies

```go
"github.com/go-chi/chi/v5"
"github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
"google.golang.org/grpc"
```

## Usage

```go
package main

import (
    "context"
    "net"
    "net/http"
    "testing"
    
    "github.com/grpc-ecosystem/grpc-gateway/v2/runtime"
    "github.com/stretchr/testify/require"
    sum "github.com/utrack/clay/doc/example/implementation"
    desc "github.com/utrack/clay/doc/example/pb"
    "google.golang.org/grpc"
)

func TestGRPCGateway(t *testing.T) {
	// implement gRPC server interface from service_grpc.pb.go file
	summatorService := sum.NewSummator()

	// register gRPC
	// register gRPC listener
	grpcListener, err := net.Listen("tcp", net.JoinHostPort("", "5000"))
	require.NoError(t, err)

	// initiate gRPC server and register gRPC call handlers
	grpcServer := grpc.NewServer()
	desc.RegisterSummatorServer(grpcServer, summatorService)

	// register HTTP
	// register HTTP listener
	httpListener, err := net.Listen("tcp", net.JoinHostPort("", "8080"))
	require.NoError(t, err)

	// initiate gRPC-gateway server mux and register HTTP call handlers)
	err = desc.RegisterSummatorHandlerServer(context.Background(), runtimeMux, summatorService)
	require.NoError(t, err)

	// start listeners
	errChan := make(chan error, 2)

	go func() {
		err := http.Serve(httpListener, httpServer)
		errChan <- err
	}()
	go func() {
		err := grpcServer.Serve(grpcListener)
		errChan <- err
	}()

	<-errChan
}
```

As original `runtime.Mux` provides very few methods:

- `Handle(meth string, pat runtime.Pattern, h runtime.HandlerFunc)`
- `HandlePath(meth string, pathPattern string, h runtime.HandlerFunc) error`
- `ServeHTTP(w http.ResponseWriter, r *http.Request)`
- `GetForwardResponseOptions() []func(context.Context, http.ResponseWriter, proto.Message) error`

You can extend your `mux` with others (ex. `github.com/go-chi/chi/v5`). To do this simply add this:

```go
runtimeMux := runtime.NewServeMux()
httpServer := chi.NewMux()
httpServer.Mount("/", runtimeMux)
```