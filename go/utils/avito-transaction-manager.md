# Application layer transactions

https://github.com/avito-tech/go-transaction-manager

## Usage

repository layer (sqlx):
```go
import (
	trmsqlx "github.com/avito-tech/go-transaction-manager/sqlx"
)

_, err = i.ctxGetter.DefaultTrOrDB(ctx, i.db).ExecContext()
```
main.go
```go
import (
	txManager "github.com/avito-tech/go-transaction-manager/sqlx"
	trmsqlx "github.com/avito-tech/go-transaction-manager/sqlx"
)

txManager := manager.Must(txManager.NewDefaultFactory(pg))
repo := repository.New(db, trmsqlx.DefaultCtxGetter)
```
application layer
```go
import (
	"github.com/avito-tech/go-transaction-manager/trm/manager"
)

type Handler struct {
	txManager         *manager.Manager
}
```

## Error restoration

```go

```