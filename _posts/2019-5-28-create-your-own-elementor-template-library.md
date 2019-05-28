---
layout: post
title: Create your own Elementor template library
---

Elementor has a great feature called Template Library which let user import predefined template by a single click. But it doesn't let us - the developers - include our template into that library.

But now we can! You can find how to do it in the rest of this blog.

Thanks to [@davelavoie](https://github.com/davelavoie) for sharing his idea and allowing me to use it.

## How it work?

Elementor Template library is made from `Source`. Elementor create sources then register them to use in the library.

![Elementor Template library]({{ site.baseurl }}/images/2019/elementor-template-library-register-default.png)

Guess what? We can follow this approach just fine by create our own source and register it.
