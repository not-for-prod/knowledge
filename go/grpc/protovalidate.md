# protovalidate

> To Install Buf CLI [Follow the instruction](../../proto/buf.md)

## Dependencies

```go
"buf.build/go/protovalidate"
```

## Usage

Proto:

```protobuf
syntax = "proto3";

package example.v1;

import "buf/validate/validate.proto";

message UUID {
  string value = 1 [
    (buf.validate.field).string.uuid = true,
    (buf.validate.field).required = true
  ];
}
```

Interceptor:

```go
func Validate(
    ctx context.Context,
    req any,
    info *grpc.UnaryServerInfo,
    handler grpc.UnaryHandler,
) (resp any, err error) {
    msg, ok := req.(proto.Message)
    if !ok {
        return nil, errors.New("invalid argument")
    }
    
    err = protovalidate.Validate(msg)
    if err != nil {
        return nil, err
    }
    
    return handler(ctx, req)
}
```