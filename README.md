PHP [serialize()](http://php.net/manual/en/function.serialize.php) and
[unserialize()](http://php.net/manual/en/function.unserialize.php) for Go.

# Install / Update

```bash
go get -u github.com/printesoi/phpserialize
```

`phpserialize` requires Go 1.8+.

# Example

```go
package main

import (
	"fmt"
	"github.com/printesoi/phpserialize"
)

func main() {
	out, err := phpserialize.Marshal(3.2, nil)
	if err != nil {
		panic(err)
	}

	fmt.Println(string(out))

	var in float64
	err = phpserialize.Unmarshal(out, &in)

	fmt.Println(in)
}
```

### Using struct field tags for marshalling

```go
package main

import (
	"fmt"
	"github.com/printesoi/phpserialize"
)

type MyStruct struct {
	// Will be marhsalled as my_purpose
	MyPurpose string `php:"my_purpose"`
	// Will be marshalled as my_motto, and only if not a nil pointer
	MyMotto *string `php:"my_motto,omitnilptr"`
	// Will not be marshalled
	MySecret string `php:"-"`
}

func main() {
	my := MyStruct{
		MyPurpose: "No purpose",
		MySecret:  "Has a purpose",
	}

	out, err := phpserialize.Marshal(my, nil)
	if err != nil {
		panic(err)
	}

	fmt.Println(out)
}
```

### Custom marshaling/unmarshaling

```go

package main

import (
	"errors"
	"fmt"
	"reflect"
	"github.com/printesoi/phpserialize"
	"github.com/shopspring/decimal"
)

type Amount decimal.Decimal

func (a Amount) PhpSerialize() reflect.Value {
	v := reflect.New(reflect.TypeOf("")).Elem()
	v.SetString(decimal.StringFixed(2))
	return v, nil
}

func (a *Amount) PhpUnserialize(data interface{}) error {
	switch v := data.(type) {
	case string:
		dec, err := decimal.NewFromString(v)
			if err != nil {
				return err
			}
		*a = Amount(dec)
	case float64:
		*a = Amount(decimal.NewFromFloat(v))
	default:
		return fmt.Errorf("Unsupported type: `%T`", data)
	}
	return nil
}

func main() {
	a1 := Amount(decimal.NewFromFloat(1.23))
	out, err := phpserialize.Marshal(a1, nil)
	if err != nil {
		panic(err)
	}

	fmt.Println(out)

	var a2 Amount
	err = phpserialize.Unmarshal(out, &a2)
	if err != nil {
		panic(err)
	}

	fmt.Println(a2.String())
}
```
