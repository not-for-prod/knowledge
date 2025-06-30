> Commonly used file formats for my code. Usually preset them in my IDE

Goland: `Editor \> File and Code Templates`

Command:
```go
package ${GO_PACKAGE_NAME}

import (
    "context"
)

type Command struct {}

type Result struct {}

type Handler struct {}

func New() *Handler {
    return &Handler{}
}

func (h *Handler) Execute(ctx context.Context, cmd Command) (Result, error) {
    return Result{}, nil
} 
```
Query:
```go
package ${GO_PACKAGE_NAME}

import (
    "context"
)

type Query struct {}

type Result struct {}

type Handler struct {}

func New() *Handler {
    return &Handler{}
}

func (h *Handler) Execute(ctx context.Context, query Query) (Result, error) {
    return Result{}, nil
} 
```
Test Suite:
```go
package ${GO_PACKAGE_NAME}

import (
	"testing"

	"github.com/stretchr/testify/suite"
)

type TestSuite struct {
	suite.Suite
}

func (suite *TestSuite) SetupSuite() {}

func (suite *TestSuite) TearDownSuite() {}

func TestTestSuite(t *testing.T) {
	suite.Run(t, new(TestSuite))
}
```