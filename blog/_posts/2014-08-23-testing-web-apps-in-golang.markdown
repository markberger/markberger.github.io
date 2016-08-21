---
layout: post
title: Testing Web Apps in Go
date: 2014-08-23 11:59:23.000000000 -07:00
---
I've been playing around with Go in my spare time by writing a toy web app. The Go standard library has some great packages around writing web applications and I've really enjoyed using them. In fact, the official language wiki includes a small tutorial on [writing web apps](https://golang.org/doc/articles/wiki/). However, there is no mention of how to test web apps using the standard library, and searching for any answer doesn't seem to turn up any great results.

Just toying around with my own personal project, I've found that the key to testing web apps in Go is to use dependency injection with high-order functions.

### Dependency Injection

[Dependency injection](http://en.wikipedia.org/wiki/Dependency_injection) means that we want to supply everything our function requires to accomplish its goal. We don't want to rely on any global state or outside services.

However, its unclear how to do this at first. Most tutorials start out having you write applications like this:

```golang
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

*Source: [Writing Web Applications](https://golang.org/doc/articles/wiki/) - Golang Wiki*

This isn't a bad approach, but what happens when we want to add a database? Or an outside package to handle our sessions, such as [Gorilla Session](http://www.gorillatoolkit.org/pkg/sessions)? Oftentimes people will instantiate a database manager or a session handler globally and call it a day. But that will cause you difficulty when you try to test your handler. It is better to write a function which will generate your handler for you.

```golang
package main

import (
    "fmt"
    "net/http"
    "github.com/markberger/database"
)

type AppDatabase interface {
	GetBacon() string
}

func homeHandler(db AppDatabase) http.Handler {
	return http.HandlerFunc(func(w http.ResponseWriter, r *http.Request) {
    	fmt.Fprintf(w, "Hi there, I love %s!", db.GetBacon())
    })
}

func main() {
	db := database.NewDatabase()
    http.HandleFunc("/", homeHandler(db))
    http.ListenAndServe(":8080", nil)
}
```

This is great, because we no longer have to rely on global variables. Also, this makes mocking a breeze. We simply have to create a mock database that fulfills the interface AppDatabase, which means we only have to implement one function, GetBacon.

### Testing

Now that we have a program, we can start writing tests. The key to testing `http.Handler` is to use a `httptest.ResponseRecorder` found in the [net/http/httptest](http://golang.org/pkg/net/http/httptest/) package.

```golang
package main

import(
	"net/http"
	"net/http/httptest"
    "testing"
)

type MockDd struct {}

function (db MockDb) GetBacon() {
	return "bacon"
}

function TestHome(t *testing.T) {
	mockDb := MockDb{}
    homeHandle := homeHandler(mockDb)
    req, _ := http.NewRequest("GET", "", nil)
    w := httptest.NewRecorder()
    homeHandle.ServeHTTP(w, req)
    if w.Code != http.StatusOK {
    	t.Errorf("Home page didn't return %v", http.StatusOK)
    }
}
```

Of course, all that boilerplate can get pretty redundant, especially when testing POST requests. Therefore, I've been using these functions to test my handlers.

```golang
package main

import (
	"net/http"
	"net/http/httptest"
	"net/url"
	"testing"
)

type HandleTester func(
	method string,
	params url.Values,
) *httptest.ResponseRecorder

// Given the current test runner and an http.Handler, generate a
// HandleTester which will test its given input against the
// handler.

func GenerateHandleTester(
	t *testing.T,
	handleFunc http.Handler,
) HandleTester {

	// Given a method type ("GET", "POST", etc) and
	// parameters, serve the response against the handler and
	// return the ResponseRecorder.

	return func(
		method string,
		params url.Values,
	) *httptest.ResponseRecorder {

		req, err := http.NewRequest(
			method,
			"",
			strings.NewReader(params.Encode()),
		)
		if err != nil {
			t.Errorf("%v", err)
		}
		req.Header.Set(
			"Content-Type",
			"application/x-www-form-urlencoded; param=value",
		)
		w := httptest.NewRecorder()
		handleFunc.ServeHTTP(w, req)
		return w
	}
}

function TestHome(t *testing.T) {
	mockDb := MockDb{}
    homeHandle := homeHandler(mockDb)
    test := GenerateHandleTester(t, homeHandle)
    w := test("GET", url.Values{})
    if w.Code != http.StatusOK {
    	t.Errorf("Home page didn't return %v", http.StatusOK)
    }
}
```

For a more thorough example of these methods in action, check out my toy project's tests [here](https://github.com/markberger/carton/blob/master/api/auth_test.go).

If you have any better methods of testing web apps written in the standard library, feel free to leave a comment or send me email.

Cheers.
