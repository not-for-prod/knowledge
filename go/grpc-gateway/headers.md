# gRPC-Gateway: working with headers

`grpc-gateway` stores outhoing and incoming headers in `google.golang.org/grpc/metadata`. 
You can retrieve you headers from there. Unfortunately `grpc-gatway` modifies headers with `runtime.MetadataHeaderPrefix`

Only few headers are left unchanged (this list can be found in `isPermanentHTTPHeader` method in 
`github.com/grpc-ecosystem/grpc-gateway/v2/runtime/context.go`):

```go
// isPermanentHTTPHeader checks whether hdr belongs to the list of
// permanent request headers maintained by IANA.
// http://www.iana.org/assignments/message-headers/message-headers.xml
func isPermanentHTTPHeader(hdr string) bool {
    switch hdr {
        case
        "Accept",
        "Accept-Charset",
        "Accept-Language",
        "Accept-Ranges",
        "Authorization",
        "Cache-Control",
        "Content-Type",
        "Cookie",
        "Date",
        "Expect",
        "From",
        "Host",
        "If-Match",
        "If-Modified-Since",
        "If-None-Match",
        "If-Schedule-Tag-Match",
        "If-Unmodified-Since",
        "Max-Forwards",
        "Origin",
        "Pragma",
        "Referer",
        "User-Agent",
        "Via",
        "Warning":
            return true
    }
    return false
}
```

So if you're adding `Set-Cookie` header it will be modified into `Grpc-Metadata-Set-Cookie`. To fix this simply use 
`runtime.WithOutgoingHeaderMatcher` for outgoing headers.

Full example will look like:

```go
httpServer := runtime.NewServeMux(
    // Remove runtime.MetadataHeaderPrefix for Set-Cookie headers.
    runtime.WithOutgoingHeaderMatcher(
        func(s string) (string, bool) {
            if s == "set-cookie" {
                return "set-cookie", true
            }
            return runtime.MetadataHeaderPrefix + s, false
        },
    ),
    // Extract particular Cookie data into metadata.
    runtime.WithMetadata(
        func(ctx context.Context, req *http.Request) metadata.MD {
            tokenCookie, _ := req.Cookie("summator-session")
            if tokenCookie == nil {
                return nil // metadata.Pairs()
            }
            return metadata.Pairs("summator-session", tokenCookie.Value)
        },
    ),
)
```