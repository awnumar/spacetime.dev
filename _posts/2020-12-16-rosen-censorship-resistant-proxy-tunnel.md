---
layout: post
title: 'rosen: censorship-resistant proxy tunnel'
---

> GitHub: [https://github.com/awnumar/rosen](https://github.com/awnumar/rosen)

Many governments and other well-resourced actors around the world implement some form of censorship to assert control over the flow of information on the Internet, either because it is deemed "sensitive" or because it is inconvenient for those with power.

Suppose there is some adversary, Eve, that wants to prevent users from accessing some content. There are many ways of implementing such censorship but they broadly fall into one of two categories: **endpoint-based** or **flow-fingerprinting** attacks.

<div class="image">
<img src="/assets/images/rosen-isp-model.png" alt="A user attempting to access censored material through an adversarial Internet service provider." class="center" width="60%" />
<center><p>A user attempting to access censored material through an adversarial Internet service provider.</p></center>
</div>

Eve could maintain a list of banned services and refuse to serve any network request she receives if it is on this list. Here Eve is deciding based on the _destination_ of the request: this is endpoint-based censorship. In response a user, Alice, could use a [VPN](https://en.wikipedia.org/wiki/Virtual_private_network) or [TOR](https://en.wikipedia.org/wiki/Tor_(anonymity_network)) to disguise the destination so that from Eve's perspective, the destination will appear to be the VPN server or the TOR entry node instead of the censored service.

This is a working solution in many places, but Eve is not beaten. In response she can add the IP addresses of known VPN providers as well as [public TOR nodes](https://www.dan.me.uk/tornodes) to her blocklist, reasoning that only a user who wants to bypass her blocking efforts would use these services. Alice could then setup her own VPN server or access the TOR network through a non-public [TOR bridge](https://2019.www.torproject.org/docs/bridges.html.en) that is not blocked.

Eve could actively probe the servers that Alice connects to in order to find out if they are TOR entry nodes, for example, but apart from this she has stretched endpoint-based censorship to its limits. An alternative is to censor a connection based on characteristics of the network flow instead of its destination: this is flow-fingerprinting. This is usually accomplished using some kind of [deep packet inspection](https://en.wikipedia.org/wiki/Deep_packet_inspection) engine that can detect and group traffic into protocols and applications. With this capability Eve can block any traffic that is detected as a proxy regardless of whether any particular server is known.

<div class="image">
<img src="/assets/images/rosen-dpi-layout.png" alt="An adversary using a deep packet inspection engine to decide whether to censor traffic." class="center" width="60%" />
<center><p>An adversary using a deep packet inspection engine to decide whether to censor traffic.</p></center>
</div>

To bypass this technique, Alice must disguise the fingerprint of her traffic so that a DPI engine does not classify it as a blocked protocol or application. There are a few approaches to this:

1. **Randomisation**. The goal here is to make the traffic look indistinguishable from randomness, or put another way, to make it look like "nothing". This would successfully hide which category traffic belongs to, but a lack of a fingerprint is a fingerprint itself and that's a vulnerability.

    Examples of randomising obfuscators include [Obfsproxy](https://blog.torproject.org/obfsproxy-next-step-censorship-arms-race) and [ScrambleSuit](https://github.com/NullHypothesis/scramblesuit).

2. **Mimicry**. Instead of making traffic look like random noise, mimicry-based obfuscation makes packets look like they belong to a specific protocol or application that is assumed to be unblocked. For example, [StegoTorus](http://sri-csl.github.io/stegotorus/) and [SkypeMorph](http://cacr.uwaterloo.ca/techreports/2012/cacr2012-08.pdf) produce traffic that looks like HTTP and Skype, respectively, but they are prohibitively slow.

    Another option is [LibFTE](https://libfte.org/) which is roughly a cryptographic cipher that produces ciphertext conforming to a given regular expression. DPI engines also commonly use regular expressions so with LibFTE it is possible to precisely force misclassification of a protocol.

    Mimicry only tries to make packet payloads look like some cover protocol and so the syntax and semantics of the overall network flow can deviate substantially from the protocol specification or any known implementation. This makes mimicry-based obfuscators [easily detectable](https://kpdyer.com/publications/ccs2015-measurement.pdf) and results in the approach being [fundamentally flawed](https://ieeexplore.ieee.org/stamp/stamp.jsp?tp=&arnumber=6547102).

3. **Tunnelling**. A tunnelling obfuscator encapsulates traffic within some cover protocol using an actual implementation of the cover protocol instead of simply trying to mimic the way its packets look. An example is [meek](https://github.com/arlolra/meek) which uses HTTPS to communicate between the client and the server and [domain fronting](https://en.wikipedia.org/wiki/Domain_fronting) to hide the true destination of the traffic, but since domain fronting relied on an undocumented feature of major CDNs, it no longer works.

    Tunnelling obfuscators have to be careful to look like a specific and commonly used implementation of a cover protocol since a custom implementation may be distinguishable. China and Iran managed to distinguish TOR TLS and non-TOR TLS first-hop connections even though TOR used a real implementation of TLS to tunnel traffic.

An important metric to consider is the **false positive** detection rate associated with each method. This is the proportion of traffic that a DPI engine falsely detects as coming from an obfuscation tool. A high false-positive rate results in lots of innocent traffic being blocked which will cause frustration for ordinary users. Therefore the goal of an obfuscator should be to look as much like innocent traffic as possible to maximise the "collateral damage" of any attempted censorship. Overall, it seems like tunnelling is the best approach.

This brings us to [Rosen](https://github.com/awnumar/rosen), a modular, tunnelling proxy that I have developed as part of my ongoing masters thesis. It currently only implements HTTPS as a cover protocol, but this has been tested against [nDPI](https://github.com/ntop/nDPI) and a commercial DPI engine developed by [Palo Alto Networks](https://en.wikipedia.org/wiki/Palo_Alto_Networks), **both of which detected TOR traffic encapsulated by Rosen as ordinary HTTPS**. The goals of Rosen are:

1. **Unobservability**. It should be difficult to distinguish obfuscated traffic from innocent background traffic using the same protocol.
2. **Endpoint-fingerprinting resistance**. It should be difficult to use active probing to ascertain that a given server is actually a proxy server. This is accomplished by responding as a proxy if and only if a valid key is provided and falling back to some default behaviour otherwise. For example, the HTTPS implementation serves some static content in this case.
3. **Modularity**. It should be relatively easy to add support for another cover protocol or configure the behaviour of an existing protocol to adapt to changing adversarial conditions. This is facilitated by a modular architecture.
4. **Compatibility**. It should be possible to route most application traffic through the proxy. This is why a SOCKS interface was chosen, but TUN support is also a goal.
4. **Usability**. It should be easy to use.

<div class="image">
<img src="/assets/images/rosen-high-level.png" alt="High-level overview of Rosen's architecture." class="center" width="75%" />
<center><p>High-level overview of Rosen's architecture.</p></center>
</div>

HTTPS was chosen as the first cover protocol to be implemented as it provides confidentiality, authenticity, and integrity; and it is ubiquitous on the Internet making it infeasible for an adversary to block. The implementation is provided by the Go standard library and most configuration options are set to their defaults so that it blends in with other applications. ~~There is a option to disable TLS 1.3 as it [could be blocked by some nation-state firewalls](https://www.theregister.com/2020/08/11/china_blocking_tls_1_3_esni/)~~ I was informed that [censors are blocking ESNI specifically](https://mailarchive.ietf.org/arch/msg/tls/Dae-cukKMqfzmTT4Ksh1Bzlx7ws/). The server will automatically provision a TLS certificate from LetsEncrypt and the client pins [LetsEncrypt's root](https://letsencrypt.org/certificates/) by default.

It's difficult to know how effective this truly is without further battle-testing by security researchers and users, but we can theorise to some extent.

1. **Endpoint-based censorship**. Users are able to setup Rosen on their own servers behind their own domains so there is no generic firewall rule that can block all of them. An adversary could instead try to actively probe a Rosen server in order to detect it.

    One option is to provide a key and detect a timing difference as the server checks it. The delta I measured between providing a 32 byte key and not providing a key is 29ns (on an AMD Ryzen 3700X). Since network requests have a latency in the milliseconds, I assume this attack is practically infeasible.

    A simpler attack is to look at the static files that the HTTPS server responds with. If the user does not replace the default files with their own, an easy distinguishing attack is possible. This could be easier to avoid with a different protocol. For example, if an incorrect SSH password is provided to an SSH server, it simply refuses the connection and there are no other obvious side-effects for an adversary to analyse.

2. **Flow-fingerprinting**. The cover protocol uses the standard library implementation of HTTPS which should be widely used by many different applications in various contexts. Default cipher suites are chosen and other aspects of the implementation are deliberately very typical.

    However, this does not cover the **behaviour** of Rosen clients. For example, HTTP requests to an ordinary website are usually a lot smaller than responses. Also, an adversary could compare the traffic between Alice and a Rosen HTTPS server with the static content available on that server to ascertain if something else is going on.

    To handle these attacks, the protocol could use some kind of random padding, limit the size and frequency of round trips, or replace the static decoy handler with a custom one that has different traffic characteristics.

    Timing patterns are particularly of importance. Currently the client waits a random interval between 0 and 100ms before polling the server for data. This choice was made to minimise latency but it is not typical of an ordinary website. Analysing timing patterns is what [allowed researchers to detect meek](https://kpdyer.com/publications/ccs2015-measurement.pdf), for example. There's no evidence that this attack is employed by real-world censors, but a configuration flag that implements a tradeoff between performance and behaving "more typically" will be added in the future.

If you have the capability to test out Rosen, especially if you are behind a firewall that implements censorship, I would greatly appreciate you telling me about about your experiences at my Email address (available on [GitHub](https://github.com/awnumar) and on this website's [homepage](https://spacetime.dev/)). If you want to contribute, you can open an issue or pull request on the project's [GitHub page](https://github.com/awnumar/rosen).
