# marshalable

## Dependency

```go
"github.com/redis/go-redis/v9"
```

## Usage

To be able to use:

```go
var dto model
client.Get(ctx, req.key()).Scan(&dto)
```

and 

```go
var dto model
client.Set(ctx, req.key(), dto, time.Until(req.ExpiresAt)).Err()
```

where `client` is `"github.com/redis/go-redis/v9".client` 
`model` must be `marshalable` (implement `marshalable` interface)

```go
// marshalable is the combination of encoding.BinaryMarshaler and
// encoding.BinaryUnmarshaler. Their method definitions are repeated here to
// avoid a dependency on the encoding package.
type marshalable interface {
	MarshalBinary() ([]byte, error)
	UnmarshalBinary([]byte) error
}
```

To do this simply:

> attention you can't use struct ptr in MarshalBinary. If you will do (r *rate) it won't work

```go
type model struct {}

func (r rate) MarshalBinary() ([]byte, error) {
	return json.Marshal(r)
}

func (r *rate) UnmarshalBinary(data []byte) error {
	return json.Unmarshal(data, r)
}
```