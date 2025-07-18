```go
import (
    "github.com/go-chi/cors"
)

type CORS struct {  
    AllowedOrigins   []string  
    AllowedMethods   []string  
    AllowedHeaders   []string  
    ExposedHeaders   []string  
    AllowCredentials bool  
    MaxAge           int  
}

hmux.Use(  
    cors.Handler(  
       cors.Options{  
          // AllowedOrigins:   []string{"https://foo.com"}, // Use this to allow specific origin hosts  
          AllowedOrigins: corsCfg.AllowedOrigins,  
          // AllowOriginFunc:  func(r *http.Request, origin string) bool { return true },  
          AllowedMethods:   corsCfg.AllowedMethods,  
          AllowedHeaders:   corsCfg.AllowedHeaders,  
          ExposedHeaders:   corsCfg.ExposedHeaders,  
          AllowCredentials: corsCfg.AllowCredentials,  
          MaxAge:           corsCfg.MaxAge, // Maximum value not ignored by any of major browsers  
       },  
    ),  
)
```
Default(YAML):
```yaml
CORS:
  AllowedOrigins: [ "https://*", "http://*" ]
  AllowedMethods: [ "GET", "POST", "PUT", "DELETE", "OPTIONS" ]
  AllowedHeaders: [ "Accept", "Authorization", "Content-Type", "X-CSRF-Token" ]
  ExposedHeaders: [ "Link" ]
  AllowCredentials: false
  MaxAge: 30
```
Default(JSON):
```json
{
  "CORS": {
    "AllowedOrigins": ["https://*", "http://*"],
    "AllowedMethods": ["GET", "POST", "PUT", "DELETE", "OPTIONS"],
    "AllowedHeaders": ["Accept", "Authorization", "Content-Type", "X-CSRF-Token"],
    "ExposedHeaders": ["Link"],
    "AllowCredentials": false,
    "MaxAge": 30
  }
}
```
