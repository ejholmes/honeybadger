# Honeybadger Go Client

This is an unofficial notifier library for integrating Go applications with [Honeybadger](http://honeybadger.io).

## Usage

### Getting Started

First you'll need to import the library and set your Honeybadger API key and an environment name for your application:

```go
import "github.com/jcoene/honeybadger"

func main() {
  // Set the API key
  honeybadger.ApiKey = "abcdef"
  // Set the application environment
  honeybadger.Environment = "production"
}
```

Later (probably when recovering from some kind of panic or error), you can create and send an error report:

```go
func DoStuff() (err error) {
  if err = doOtherThing(); err != nil {
    // Create a new report with 0 call stack inflation (more on that later)
    // Give it the error we received as the message (could be anything)
    report, err2 := honeybadger.NewReport(0, err)

    // Send the error (asynchronously in a Goroutine)
    report.Dispatch()
  }
}
```

### Adding Context

It's possible (I'd say advisable) to add some context for the failure:

```go
// Create the report
report, _ := honeybadger.NewReport(0, err)

// Set the request URL
report.Request.URL = myHttpReq.URL

// Set all of the incoming request headers. Could be anything.
for k, v := range myHttpReq.Header {
  report.AddContext(k, v[0])
}
```

The Report object you create by calling NewReport is self describing and can be manipulated as you see fit. Inspect it and make it work for you.

There are a few additional convenience methods for adding context of various types. *AddContext*, *AddParam*, and *AddSession* all take a key and value to build on the hash for their respective category.

### Labels and Backtraces

Your error reports are automatically labeled and given backtraces based on the call stack. This is accomplished using the Go runtime package, and has a few quirks:

The automatically generated labels will only be accurate if the library can properly determine the origin of the error. As such you'll need to supply the depth of stack inflation as the first argument to NewReport:

- **0** means that *honeybadger.NewReport* is called directly from the function reporting the error
- **1** indicates the error is one step removed - useful for a helper function.

For example, let's say you want to have a single helper function in your service to report errors, it should call NewReport with a depth of 1:

```go
  func Get(req, resp, ...) {
    if err = doOtherThing; err != nil {
      // Oh no, let ops know that things aren't going so well!
      reportError(req, resp, err)
    }
  }

  // ...

  func reportError(req, resp, err) {
    // Create the report with an inflated stack depth of 1
    report, err2 := honeybadger.NewReport(1, err)

    // Fill in a bunch of useful information from the request
    report.Request.URL = req.URL
    
    // ...

    // Send
    report.Dispatch()
  }
```

### Sending Error Reports

Use the **Dispatch** method to send a report asynchronously.

If you'd like to see the result of the send operation, you can call **Send** (which returns an error or nil).

## License

MIT License.
