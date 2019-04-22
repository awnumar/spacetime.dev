---
layout: post
title: 'memory security in go'
---

A few months ago, while working on [dissident](https://github.com/awnumar/dissident), I started looking around for guidance on how I should manage encryption keys. I found a few references here and there, but the best I happened to put together with the limited information available was something like:

1. Call [mlock(2)](https://linux.die.net/man/2/mlock) (or [VirtualLock](https://msdn.microsoft.com/en-us/library/windows/desktop/aa366895(v=vs.85).aspx)) on any sensitive resources.
2. Overwrite the resources when finished with them.
3. Allow them to run out of scope.
4. Hope for the garbage-collector to clean them up, or force it with [`runtime.GC()`](https://golang.org/pkg/runtime/#GC).

So, I patched together a [little library](https://github.com/awnumar/memguard) that handled some of it for me, [chucked it on HN](https://news.ycombinator.com/item?id=14173716), and moved on.

A short time later, I realised that [beef was kicking off](https://github.com/awnumar/memguard/issues/3). As it turned out, the aforementioned approach was fundamentally flawed as one thing hadn't been accounted for: the garbage-collector. It goes around doing whatever it feels like doing; making a copy here; moving something around there; it's a real pain from a security standpoint.

I really didn't have a choice at that point: I had to dedicate time to the project, for the ten days it took to develop and release the fix.

A few people had mentioned wrapping [libsodium](https://github.com/jedisct1/libsodium), but I wanted a pure-go solution, so that wasn't ideal. Instead myself and [dotcppfile](https://twitter.com/dotcppfile) began analysing how libsodium actually worked. He began auditing that while I researched some protection strategies and implemented APIs for the relevant system calls.

Within a few days, we had a pretty solid understanding of libsodium and we were ready with a new and improved plan. I think the best way to explain it is to introduce you to the end product: [memguard](https://github.com/awnumar/memguard).

Alright, now say you need to generate an encryption key and store it securely. What's the process?

Well first we need some memory from the OS, so we need to determine the number of pages that we have to allocate. In this case the length of the buffer is 32 bytes and we can assume the system page-size to be 4096 bytes. The data is stored between two guard pages and is prepended with a random canary of length 32 bytes (more on these later). So, since the data and the canary together will comfortably fit into a single page, we need to allocate just three pages.

But we can't ask the Go runtime for the memory---since then it is free to mess around with it---so how do we do it? Well, there are a few ways to accomplish this, but we decided to go with [Joseph Richey](https://github.com/josephlr)'s suggestion of using [mmap(2)](https://linux.die.net/man/2/mmap) (or [VirtualAlloc](https://msdn.microsoft.com/en-us/library/windows/desktop/aa366887(v=vs.85).aspx) on Windows), since the system-call is [natively implemented](https://godoc.org/golang.org/x/sys/unix#Mmap), and that allowed us to avoid a dirty cgo solution.

But actually, of course only the Unix system-calls were natively-implemented, the Windows ones were not. Luckily, there was [this](https://github.com/alexbrainman/winapi) library by [Alex Brainman](https://github.com/alexbrainman) that we could vendor instead, and it proved invaluable. (I did later [add](https://github.com/golang/sys/commit/d18155cb60f6162f5e706d7f26e4e50a3d4da857) the missing system-calls to the standard library to remove the dependency.)

Now that our pages are allocated, we should configure the guard pages. We tell the kernel to disallow all reads and writes to the first and last pages, so if anything does try to do so, a SIGSEGV access violation is thrown and the process panics. This way buffer overflows can be detected immediately, and it becomes almost impossible for other processes to locate and access the data.

The remaining page, the one sandwiched between the guard pages, needs to be protected too. You see, as system memory runs out, the kernel copies over the memory of inactive processes to the disk, and that is something we would like to avoid. So, we tell the kernel to leave this middle page alone.

The last thing is the canary: a random value placed just before the data. If it ever changes, we know that something went wrong---probably a buffer underflow. When the program first ran, we generated a global value for the canary, so we just set the canary bytes to that, and the container is pretty much ready for use.

![xkcd_protocol](/assets/images/memguard_memory_layout.png){: .center}
<center>The current state of our three pages.</center>

All that is left to do now is handle the data itself. In our case the function [`NewImmutableRandom()`](https://godoc.org/github.com/awnumar/memguard#NewImmutableRandom) was called, which fills the created buffer with cryptographically-secure random bytes after it is created. A read-only status was also requested, so after the buffer is filled, we tell the kernel to only allow reads from the middle page. As before, any attempts to write to the buffer will trigger a SIGSEGV access violation and the process will panic.

This project is under active development and so there may be new features and breaking changes in the future. You can view the source code [here](https://github.com/awnumar/memguard), and full documentation can be found [here](https://godoc.org/github.com/awnumar/memguard).

Note that while we can try to do the best we can we will only ever be lowering the likelihood of sensitive data being exposed, not eliminating the possibility altogether.