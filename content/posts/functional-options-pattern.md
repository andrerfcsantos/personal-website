---
title: "Functional Options Pattern"
date: 2025-02-28T20:05:08Z
draft: false
---

This blog post is about the Functional Options Pattern, which provides a way to pass configuration options into a function or object while keeping the module's API clean, flexible, and easy to use.

This pattern became very popular in the Go programming language, but it can be applied in other languages.
All the examples will use Go, but they should be readable even if you don't know Go.

In this blog post, we'll start by illustrating the problem the Functional Options Pattern tries to solve using a simple text rendering function that we want to take some options.
We'll explore some traditional solutions for the problem and see how the Functional Options Pattern has some advantages over them.

## Building a text rendering function

Let's say we want to implement a function to render text:

```go
func Render(text string) {
    // Render text
}

// We would call it like this:
Render("Hello, World!")
```

This works, but this function is not very flexible.
We can't customize how the text is rendered.
We might want to change the color or font size.
A natural first step might be to add more parameters:

```go
func Render(text string, textColor string, fontSize int) {
    // Render text using options
}

// We would call it like this:
Render("Hello, World!", "red", 15)
```

While this allows for more customization, it introduces some problems:

- **Parameter bloat** - As we add more options, the parameter list becomes unwieldy, hurting readability.
- **Mandatory options** - All parameters are implicitly mandatory - we must provide values for all of them even if we only want to change one option.

We want a clean API with minimal parameters, and the ability to specify only the options we need.


> **NOTE:** If your language supports optional named arguments, that would be a reasonable way to solve these issues. 
> However, Go doesn't, so we need another approach.
> Even if the language you are using has named arguments, this pattern can still be beneficial, keep reading ;)

To solve the huge amount of parameters, we can group them into a struct and pass a struct instance as a parameter:

```go
// Group all options in a struct
type RenderingOptions struct {
    TextColor string
    FontSize  int
}

func Render(text string, options RenderingOptions) {
    // Render text using options
}

// We would call it like this:
Render("Hello, World!", RenderingOptions{
    TextColor: "red",
})
```

This is nicer.
We've reduced the possible huge amount of parameters for options to just a single one.
And if we imagine that there is an instance of `RenderingOptions` with all the default values, we can write some code to merge those default options with the instance we pass into the `Render` function, allowing us to specify just the options we want to change!

```go
// Group all options in a struct
type RenderingOptions struct {
    TextColor string
    FontSize  int
}

var defaultRenderingOptions = RenderingOptions{
    TextColor: "black",
    FontSize:  12,
}

func Render(text string, options RenderingOptions) {
    // Merge defaultRenderingOptions with options

    // Render text using the result of the merge as the options
}

// We would call it like this:
Render("Hello, World!", RenderingOptions{
    TextColor: "red",
})
```

This allows the caller to pass just the options it wants.
We also just have a single parameter related with options.
But, can we do better?
Looking closely, this approach still has some issues that we can address:

- **Passing no options** - We'd like to call `Render("Hello, World!")` without any options if we just want to use the defaults. Right now, we still have to pass `RenderingOptions{}` as a second parameter to use the default options.
  
- **Options encapsulation** - We are completely exposing all the fields of `RenderingOptions`. This means that any change to any of its existing fields would be a breaking change. Ideally, we want to encapsulate `RenderingOptions` in a way that allows for changing implementation details without breaking changes.

> **Challenge:** If you're new to the Functional Options Pattern, try solving these 2 issues in your favorite language before reading on!
> The only rule is using named arguments is not allowed.

## Introducing the Functional Options Pattern

To allow calling `Render("Hello, World!")` with no options, but also allow calling the function with an arbitrary number of options, we need a variadic parameter:

```go
func Render(text string, options ...RenderingOption) {
    // Render text
}
```

This way, we can pass no options, or an arbitrary number of them!
However, how should we define the `RenderingOption` type?

One neat thing you can do in Go is to define new types based on other ones, including function types.
We can say something like this:

```go
type RenderingOptions struct {
    TextColor string
    FontSize  int
}

type RenderingOption func(*RenderingOptions)
```

A `RenderingOption` is a function that takes a pointer to `RenderingOptions`, hence can modify its contents.

`TextColorBright` and `TextFontSmall` are examples of functions that are a `RenderingOption`:

```go
func TextColorBright(options *RenderingOptions) {
    options.TextColor = "white"
}

func TextFontSmall(options *RenderingOptions) {
    options.FontSize = 8
}

// Keep the default options
Render("Hello, World!")
// Use the default options, except for the color which should be bright (white in this case)
Render("Hello, World!", TextColorBright)
// Use a bright color in a small text
Render("Hello, World!", TextFontSmall, TextColorBright)
```

We can go a step further.
Notice that `TextColorBright` and `TextFontSmall` set specific colors and font sizes respectively.
For these parameters to be truly options, we would like to give the caller an option to set them!
We can do this by making a function that takes a value for the option and returns a `RenderingOption` function that sets the value to it:

```go
type RenderingOptions struct {
    textColor string
    fontSize  int
}

type RenderingOption func(*RenderingOptions)

func WithTextColor(color string) RenderingOption {
    return func(options *RenderingOptions) {
        options.textColor = color
    }
}

func WithTextSize(size int) RenderingOption {
    return func(options *RenderingOptions) {
        options.fontSize = size
    }
}

// Keep the default options
Render("Hello, World!")

// Use the default options, except for the color which should be bright (white in this case)
Render("Hello, World!", WithTextColor("white"))

// Use a bright color in a small text
Render("Hello, World!", WithTextSize(8), WithTextColor("white"))
```

This accomplishes our goals.
It allows the module's user to provide just the options they want to set while requiring no option to be passed if all we want are the defaults.
Furthermore, this allows us to encapsulate `RenderingOption`.
Notice that in the last example, all fields are not exposed.
As long as we can keep the signature of the `With*` functions the same, we can change the implementation of the options struct without introducing breaking changes to our module.

> In Go, a field not exposed, i.e. private, is indicated by their name starting with a lowercase letter.

This example wouldn't be fully complete without seeing how we should handle merging the default values for the options with the options the user passes to the `Render` function.
A simple way to do this is to keep a `RenderingOptions` with the default options and make a copy of it.
Then we can create a pointer to it and let the `RenderingOption` functions the user passes change it:

```go
type RenderingOptions struct {
    TextColor string
    FontSize  int
}

type RenderingOption func(*RenderingOptions)

var defaultRenderingOptions = RenderingOptions{
    TextColor: "black",
    FontSize:  12,
}

func Render(text string, options ...RenderingOption) {
    // If RenderingOptions has only primitive types, := performs a copy of the struct.
    // Use other ways of cloning the struct if that's not the case
    finalOptions := defaultRenderingOptions

    // Go through all the options passed and apply/merge them to the defaults
    for _, opt := range options {
        opt(&finalOptions)
    }

    // Render text using finalOptions
}
```

## Variations on the Functional Options Pattern

In the example above, we've created a separate struct type to hold all the options we want to pass with `RenderingOptions`.
One popular variation of the Functional Options Pattern doesn't create this additional type, and instead, the option function modifies the subject of the options directly.

Suppose we have a `Server` type that represents an http server.
One of the options we could pass to a `Server` is its port.
But instead of having a type like `ServerOptions`, with a `Port` field inside it, we can have the field on the server directly, and have the `ServerOption` function modify it directly:

```go
type Server struct {
    port int
    // Other server fields
}

func (s *Server) Start() {
    // ...
}

type ServerOption func(*Server)

func WithPort(port int) ServerOption {
    return func(server *Server) {
        server.port = port
    }
} 

func NewServer(options ...ServerOption) *Server {
    // Apply options like in the previous example
    // ...
}

// We would call it like
NewServer(WithPort(80))
```

Starting the names of the option functions with  `With*` is common, but you'll also see libraries not using this convention.
In the previous example, we could have made the function `WithPort` be called `HTTPPort` instead:

```go
func HTTPPort(port int) ServerOption {
    return func(server *Server) {
        server.Port = port
    }
}

// We would call it like
NewServer(HTTPPort(80))
```

## Functional Options Pattern in the wild

Below are examples of some popular Go projects that use this pattern:

- [Docker client](https://github.com/moby/moby/blob/bbd0a17ccc67e48d4a69393287b7fcc4f0578683/client/client.go#L189) - Uses the Functional Options Pattern to provide options when [creating a new Docker client](https://github.com/moby/moby/blob/bbd0a17ccc67e48d4a69393287b7fcc4f0578683/client/client.go#L189).
 One interesting adaptation of the pattern is that the option function can return an error.

- [Uber's zap logging library](https://github.com/uber-go/zap) - Fast logging library by Uber, that uses this pattern to provide options when [creating a new logger](https://github.com/uber-go/zap/blob/fcf8ee58669e358bbd6460bef5c2ee7a53c0803a/logger.go#L69).
 Zap introduces a variation on the pattern where the option type is not a function, but rather an interface that wraps the option function.

- [BubbleTea](https://github.com/charmbracelet/bubbletea) - TUI framework which uses this pattern to [pass options to TUI Programs](https://github.com/charmbracelet/bubbletea/blob/b224818d994537a25de86e2658fb9f437ea0baf4/tea.go#L233).

## Final remarks

I find the Functional Options Pattern to be a good tool to have in your programming toolkit.
It provides a way for the users of your module/library to configure the behavior of objects and functions while providing a nice and clean API.
On the implementer's side of the library, it also allows for flexibility in handling those options.

While this pattern can be used in any language that allows [variadic arguments](https://en.wikipedia.org/wiki/Variadic_function) and where functions are [first-class citizens](https://en.wikipedia.org/wiki/First-class_citizen), it gained a lot of popularity in Go projects.
I think this has to do with the constraints of Go itself that are not present in other languages.
Languages like Python allow for an arbitrary number of named arguments to be passed to a function, that are then collected in a single `**kwargs` object, and it's easy to merge this object with some other object providing default options.
Javascript libraries usually do this by passing some options object that can then be merged easily with others using the spread operator or other utils in the language.

In Go, named arguments are not supported, and passing objects with an arbitrary number of properties is not trivial.
This makes it so passing options to an object/function with the Functional Options Pattern is probably the most ergonomic way to pass options.
However, I think it is handy to know about this pattern and to apply the core concepts of it in other languages.

