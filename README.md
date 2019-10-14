# go-tidbits

## 10/9/2019
- When running `go get` on a project that uses `ldflags`, they won't be set. The application will have blank data. (e.g. version, build). Something to consider when using this functionality and suggesting users to get your project via `go get` instead of providing a binary produced through your build system.

- When running `go get` outside of the main module for an application (i.e. working directory is not in the modules path), `replace`/`exclude` directives do not apply. This can result in `go get` operations that fail to compile if the module takes the liberty of replacing and excluding certain dependencies.

## 10/10/2019
- Go 1.13 introduced the `-trimpath` argument which can assist in creating reproducible builds. When an error occurs, the stack trace has information relating to the build environment, this means a binary can be different based on where its built from. `-trimpath` removes these paths and only uses the module path.

## 10/11/2019
- When resolving whether or not a type satisfies an interface, if the type has a `pointer` receiver, if you create that type as a value type, the type will not satisfy the interface. However, if the type has a `value` receiver, then creating that type as a pointer _or_ value will satisfy the interface.

```go
type Fooer interface {
	Foo()
}

type A struct { }

func (a A) Foo() { }

type B struct { }

func (b *B) Foo() { }

func main() {

	aVal := A{}
	aPtr := &A{}
	
	bVal := B{}
	
	var f Fooer
	
	f = aVal
	f = aPtr
    
    // This fails to compile because bVal was constructed
    // as a value type. It's receiver is a pointer type.
    // Thus, the interface is not satisfied.
	f = bVal

	
	fmt.Println(f)
}
```

- Maps don't appear to completely resolve to the type when indexed..
```go
package main

type S struct {
	data string
}

func (s S) Read() string {
	return s.data
}

func (s *S) Write(str string) {
	s.data = str
}

func main() {

	sVals := map[int]S{}
	sVals[1] = S{"A"}

	sVals[1].Read()
	// This line fails to compile. sVals[1] must not resolve to type S{}?
	// sVals[1].Write("ok")

	// .. however, moving it out of the map, it's fine
	noMap := S{"A"}
	noMap.Write("ok")

	// These are all OK
	sPtrs := map[int]*S{}
	sPtrs[1] = &S{"A"}

	sPtrs[1].Read()
	sPtrs[1].Write("ok")

	// Using a map is doing something wonky..
	// The semantics feel like interface resolution.
	// sVals (value type) can only use value receivers
	// sPtrs (pointer type) can use value and pointer receivers

}
```

- With embeded types, the embeded types semantic (i.e. pointer or value), takes precedences over the parent semantic. 

```go
    // Even though Admin is using value semantics, User will still use pointer.
    // If User is embedded into Admin, methods on User are still pointer semantics.
	admin := Admin{
		User: &User{
			Name:  "john smith",
			Email: "john@email.com",
		},
		Level: "super",
    }
```

- When embedding types, if methods collide, the parent method takes precedence.

## 10/14/2019

- When using `errors.As`, you must use the same value/pointer semantics for the exposed error. For example, if the API returns

```go
return MyError{} // not using &MyError{}
```

Then your `.As()` code must be

```go
var e someapi.MyError
errors.As(err, &e)
```
