genny - Generics for Go
=====

(pron. Jenny) by Mat Ryer and Tyler Bunnell.

Until the Go core team include support for [generics in Go](http://golang.org/doc/faq#generics), `genny` is a code-generation generics solution. It allows you write normal buildable and testable Go code which, when processed by the `genny gen` tool, will replace the generics with specific types.

## How it works

Define your generic types using the special `generic.Type` placeholder type:

```
var KeyType generic.Type
var ValueType generic.Type
```

  * You can use as many as you like
  * Give them meaningful names

Then write the generic code referencing the types as your normally would:

```
func SetValueTypeForKeyType(key KeyType, value ValueType) { /* ... */ }
```

  * Generic type names will also be replaced in comments and function names (see Real example below)

Since `generic.Type` is a real Go type, your code will compile, and you can even write unit tests against your generic code.

#### Generating specific versions

Pass the file through the `genny gen` tool with the specific types as the argument:

```
cat generic.go | genny gen "KeyType=string,ValueType=interface{}"
```

The output will be the complete Go source file with the generic types replaced with the types specified in the arguments.

## Real example

Given [this generic Go code](https://github.com/metabition/genny/tree/master/examples/queue) which compiles and is tested:

```
package queue

import "github.com/metabition/genny/generic"

// NOTE: this is how easy it is to define a generic type
type Something generic.Type

// SomethingQueue is a queue of Somethings.
type SomethingQueue struct {
  items []Something
}

func NewSomethingQueue() *SomethingQueue {
  return &SomethingQueue{items: make([]Something, 0)}
}
func (q *SomethingQueue) Push(item Something) {
  q.items = append(q.items, item)
}
func (q *SomethingQueue) Pop() Something {
  item := q.items[0]
  q.items = q.items[1:]
  return item
}
```

When `genny gen` is invoked like this:

```
cat source.go | genny gen "Something=string"
```

It outputs:

```
// This file was automatically generated by genny.
// Any changes will be lost if this file is regenerated.
// see http://github.com/metabition/genny

package queue

// StringQueue is a queue of Strings.
type StringQueue struct {
  items []string
}

func NewStringQueue() *StringQueue {
  return &StringQueue{items: make([]string, 0)}
}
func (q *StringQueue) Push(item string) {
  q.items = append(q.items, item)
}
func (q *StringQueue) Pop() string {
  item := q.items[0]
  q.items = q.items[1:]
  return item
}
```

#### More examples

Check out the [test code files](https://github.com/metabition/genny/tree/master/parse/test) for more real examples.
