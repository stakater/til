# Log information on console from unit tests function

## `ISSUE`

While I was writing unit-tests, I hit with an issue of how to log testing/debugging information on console from inside the unit test function. The default `fmt` package's `Println` & `Printf` methods doesn't work from inside the unit tests function. 

## `SOLUTION`

To log testing/debugging information from unit test function to console we can use the `testing` package's `Log` method. 

```go
import (
	"testing"
)

func TestSampleUnitTestFunction(t *testing.T) {

	t.Log("Hello World")

}
```
Run the tests using command given below:
```bash
$ go run -v <folder-name>
```

or, to run all the tests inside a folder

```bash
$ go run -v ./<folder-name>/...

```
