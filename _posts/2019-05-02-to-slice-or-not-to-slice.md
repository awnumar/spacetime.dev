---
layout: post
title: to slice or not to slice
---

Go is an incredibly useful programming language because it hands you a fair amount of **power** while remaining fairly succinct. Here are are few bits of knowledge I've picked up in my time spent with it.

Say you have a fixed-size byte array and you want to pass it to a function that only accepts slices. That's easy, you can "slice" it:

```go
var bufarray [32]byte
bufslice := bufarray[:] // []byte
```

Going the other way is harder. The standard solution is to allocate a new array and copy the values over:

```go
bufslice := make([]byte, 32)
var bufarray [32]byte
copy(bufarray[:], bufslice)
```

"What if I don't want to make a copy?", I hear you ask. You could be handling sensitive data or maybe you're just optimizing the shit out of something. In any case we can grab a pointer and do it ourselves:

```go
bufarrayptr := (*[32]byte)(unsafe.Pointer(&buf[0])) // *[32]byte (same memory region)
bufarraycpy := *(*[32]byte)(unsafe.Pointer(&buf[0])) // [32]byte (copied to new memory region)
```

A pointer to the first element of the slice is passed to [`unsafe.Pointer`](https://golang.org/pkg/unsafe/#Pointer) which is then cast to "pointer to fixed-size 32 byte array". Dereferencing this will return a copy of the data as a **new** fixed-size byte array.

The [unsafe cat](https://en.wikipedia.org/wiki/Memory_safety) is out of the bag so why not get funky with it? We can make our own slices, with blackjack and hookers:

```go
func ByteSlice(ptr *byte, len int, cap int) []byte {
    var sl = struct {
        addr uintptr
        len  int
        cap  int
    }{uintptr(unsafe.Pointer(ptr)), len, cap}
    return *(*[]byte)(unsafe.Pointer(&sl))
}
```

This function will take a pointer, a length, and a capacity; and return a slice with those attributes. Using this, another way to convert an array to a slice would be:

```go
var bufarray [32]byte
bufslice := ByteSlice(&bufarray[0], 32, 32)
```

We can take this further to get slices of arbitrary types, `[]T`, as long as the memory region being mapped to divides the size of `T`. For example, to get a `[]uint32` representation of our `[32]byte` we would divide the length and capacity by four (a `uint32` value consumes four bytes) and end up with a slice of size eight:

```go
var sl = struct {
    addr uintptr
    len  int
    cap  int
}{uintptr(unsafe.Pointer(&bufarray[0])), 8, 8}
uint32slice := *(*[]uint32)(unsafe.Pointer(&sl))
```

**But there is a catch**. This "raw" construction converts the `unsafe.Pointer` object into a `uintptr`---a "dumb" integer address---which will not describe the region of memory you want if the runtime or garbage collector moves the original object around. To ensure that this doesn't happen you can allocate your own memory using system-calls or a C allocator like [`malloc`](https://linux.die.net/man/3/malloc). This is exactly what we had to in [memguard](https://github.com/awnumar/memguard): the system-call wrapper is available [here](https://godoc.org/github.com/awnumar/memguard/memcall#Alloc). To avoid memory leaks, remember to [free](https://godoc.org/github.com/awnumar/memguard/memcall#Free) your allocations!

It seems a bit wasteful to have a garbage collector and not use it though, so why don't we let it catch some of the freeing for us? First create a container structure to work with:

```go
type buffer struct {
    Bytes []byte
}
```

Add some generic constructor and destructor functions:

```go
import "github.com/awnumar/memguard/memcall"

func alloc(size int) *buffer {
    if size < 1 {
        return nil
    }
    return &buffer{memcall.Alloc(size)}
}

func (b *buffer) free() {
    if b.Bytes == nil {
        // already been freed
        return
    }
    memcall.Free(b.Bytes)
    b.Bytes = nil
}
```

We use [`runtime.SetFinalizer`](https://golang.org/pkg/runtime/#SetFinalizer) to inform the runtime about our object and what to do if it finds it some time after it becomes unreachable. Modifying `alloc` to include this looks like:

```go
func alloc(size int) *buffer {
    if size < 1 {
        return nil
    }

    buf := &buffer{memcall.Alloc(size)}

    runtime.SetFinalizer(buf, func(buf *buffer) {
        go buf.free()
    })

    return buf
}
```

Alright I think that's enough shenanigans for one post.
