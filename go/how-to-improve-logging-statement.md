# How to improve logging statement

Golang's log package have some flags that can be used change the log statement structure. Default log message have less information because it show only date and time. 

Example:
```bash
2019/04/12 07:08:04 Message
```

So, to improve logging we can use these flags:

```go

    Ldate         = 1 << iota     // the date in the local time zone: 2009/01/23
    Ltime                         // the time in the local time zone: 01:23:23
    Lmicroseconds                 // microsecond resolution: 01:23:23.123123.  assumes Ltime.
    Llongfile                     // full file name and line number: /a/b/c/d.go:23
    Lshortfile                    // final file name element and line number: d.go:23. overrides Llongfile
    LUTC                          // if Ldate or Ltime is set, use UTC rather than the local time zone
    LstdFlags     = Ldate | Ltime // initial values for the standard logger
```

### Code to set Flags
Code example given shows to how to set flags to log packge object

```go

package main
import (
    "log"      
)
func init(){
    log.SetPrefix("LOG: ")
    log.SetFlags(log.Ldate | log.Lmicroseconds | log.Llongfile)
    log.Println("init started")
}
func main() {
// Println writes to the standard logger.
    log.Println("main started")
 
// Fatalln is Println() followed by a call to os.Exit(1)
    log.Fatalln("fatal message")
 
// Panicln is Println() followed by a call to panic()
    log.Panicln("panic message")
}
```
output

```bash

LOG: 2019/04/12 12:47:03.999548 log-test.go:8: init started
LOG: 2019/04/12 12:47:03.999604 log-test.go:12: main started
LOG: 2019/04/12 12:47:03.999609 log-test.go:15: fatal message
exit status 1


```