```go
import (
	sq "github.com/Masterminds/squirrel"
)

sq.StatementBuilder.PlaceholderFormat(sq.Dollar)
```