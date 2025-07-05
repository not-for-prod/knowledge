# Goland Templates

> Commonly used file formats for my code. Usually preset them in my IDE for faster development

## File and Code Templates

`Editor > File and Code Templates`

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

## Live templates

`Editor > Live templates`

### Go Struct Tags

db:

- Abbreviation: `db`
- Description: `db:""`

```
`db:"$FIELD_NAME$"$END$`
```

`Edit Variables...`

| Name | Expression |
| - | - |
| FIELD_NAME | snakeCase(fieldName()) |

### Go:

service (struct with constructor):

- Abbreviation: `service`

```
type $NAME$ struct {}

func New$NAME$() *$NAME$ {
    return &$NAME${}
}
```

service method:

- Abbreviation: `service method`

```
type Request struct {}

type Response struct {}

func ($RECEIVER$ $TYPE_1$) $NAME$(ctx context.Context, req Request) (Response, error) {   
 $END$
}
```
