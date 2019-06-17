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


### Create a small plugin to hold our code

There are many ways to create a plugin for WordPress. In this post, I use the scaffold command of WP CLI to kickstart our plugin:
```
$ wp scaffold plugin custom-elementor-source
```

### Unregister the default remote source

The first step we want to do is unregister the default remote source. Why? Because the Elementor Blocks and Pages are designed to work with the `remote` source only, so we must replace the Elementor `remote` source with our own. The title of this post now can change to "Override Elementor remote template".

Put the code bellow to the main plugin file we created above.
![Unregister default remote source]({{ site.baseurl }}/images/2019/unregister-default-remote-source.png)

### Create our Source_Custom

Now let's create a new file `includes/source.php` in our plugin folder and get our hand dirty.

![Create Source_Custom Class]({{ site.baseurl }}/images/2019/creating-source-custom-class.png)

We're creating a new remote source for Elementor, so it will look like the Elementor one. Copy the content of `Source_Remote` located at `elementor/includes/template-library/sources/remote.php` to our newly created `Source_Custom`. At this time, you can open Elementor editor and check the Template Library.

Nothing happened? No, it may look identically but things happened. The source is now controlled by our `Source_Custom`. The reason why it looks as before is we clone the Elementor remote source class. We gonna modify it to make it work for us very soon.

Now examine the `Source_Custom::get_items`:

![Source_Custom::get_items]({{ site.baseurl }}/images/2019/source-custom-get-items.png)

This method fetches templates information from Elementor server then save it to the database (`wp_options` table). The method uses `Elementor\Api::get_library_data` to get library information. To make source get templates information from our endpoint, we need to implement our own `get_library_data` method.

Rename `Api::get_library_data` to `self::get_library_data` then add these new properties and methods to our `Source_Custom`:

![Source_Custom::get_library_data]({{ site.baseurl }}/images/2019/source-custom-get-library-data.png)

From now, we can change the `$api_info_url` to our info url which returns the library data and get them displayed in the Elementor Template Library. (more on this later).

We have a remaining call to Elementor Api in `Source_Custom::get_data`. We do the same process as above, change `Api::get_template_content` to `self::get_template_content` and add the following code to `Source_Custom` class.

![Source_Custom::get_template_content]({{ site.baseurl }}/images/2019/source-custom-get-template-content.png)

Change `$api_get_template_content_url` to our endpoint and now we have fully working template source. :)

> You can grab the full source code of this plugin here: [dinhtungdu/custom-elementor-source](https://github.com/dinhtungdu/custom-elementor-source)

** Testing our new source

*To be continued..*
