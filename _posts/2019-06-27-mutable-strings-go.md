---
layout: post
title: mutable strings in go
---

According to the Go [language specification](https://golang.org/ref/spec#String_types):

> Strings are immutable: once created, it is impossible to change the contents of a string.

We'll see about that.

```go
data := []byte("yellow submarine")
str := *(*string)(unsafe.Pointer(&data))

for i := range data {
    data[i] = 'x'
    fmt.Printf("%s\n%s\n\n", string(data), str)
}
```

[Try it out yourself](https://play.golang.org/p/IDLY5QFcGwW).
