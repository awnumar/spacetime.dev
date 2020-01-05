---
layout: post
title: memory retention attacks
---

In my [post]({{ site.url }}/encrypting-secrets-in-memory) on implementing an in-memory encryption scheme to protect sensitive information, I referenced a mitigation strategy called a Boojum. It is described by Bruce Schneier, Niels Ferguson and Tadayoshi Kohno in their book, [Cryptography Engineering](https://www.schneier.com/books/cryptography_engineering/): Design Principles and Practical Applications.

A number of people asked me about the mechanisms of the attack and the scheme, so I am including the relevant parts here. It is an excellent resource and I recommend that you go and buy it.

![page one]({{ site.url }}/assets/images/cold-boot/page-1.png){: .center}
![page two]({{ site.url }}/assets/images/cold-boot/page-2.png){: .center}
![page three]({{ site.url }}/assets/images/cold-boot/page-3.png){: .center}

[17] Jurjen Bos. Booting problems with the JEC computer. Personal communications, 1983. [Page 125]

[24] Lewis Carroll. _The Hunting of the Snark: An Agony, in Eight Fits_. Macmillan and Co., London, 1876. [Page 126]

[32] Giovanni Di Crescenzo, Niels Ferguson, Russel Impagliazzo, and Markus Jakobsson. How to Forget a Secret. In Christoph Meinel and Sophie Tison, editors, STACS 99, volume 1563 of _Lecture Notes in Computer Science_, pages 500-509. Springer-Verlag, 1999.

[57] Peter Gutmann. Secure Deletion of Data from Magnetic and Solid-State Memory. In _USENIX Security Symposium Proceedings_, 1996. Available from http://www.cs.auckland.ac.nz/~pgut001/pubs/secure_del.html.

[59] J. Alex Halderman, Seth D. Schoen, Nadia Heninger, William Clarkson, William Paul, Joseph A. Calandrino, Ariel J. Feldman, Jacob Appelbaum, and Edward W. Felten. Lest We Remember: Cold Boot Attacks on Encryption Keys. In _USENIX Security Symposium Proceedings_, pages 45-60, 2008.