---
layout: post
title: Create your own Elementor template library
---

Elementor has a great feature called Template Library which let user import predefined template by a single click. But it doesn't let us - the developers - include our template into that library (by default).

But now we can : ).

Thanks to [@davelavoie](https://github.com/davelavoie) for sharing his idea and allowing me to use it.

## How does it work?

Elementor Template library is made from `Source`. Elementor creates sources then registers them to use in the library.

![Elementor Template library]({{ site.baseurl }}/images/2019/elementor-template-library-register-default.png)

Guess what? We can follow this approach just fine by creating our own source and register it.

The first step we want to do is unregister the default remote source. Why? Because the Elementor template library only designs for `remote` source, so we must replace the Elementor `remote` source with our own. The title of this post now can change to "Override Elementor remote template".

