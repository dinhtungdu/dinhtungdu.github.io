---
layout: post
title: WordPress Comment Type
---

The comment system of WordPress is underused. Look at the `get_comments` function; you will notice an argument named `comment_type`. By default, WordPress has three comment types: comment, pingback, and trackback. We don't have any official way to add `comment_type`.

There are many possibilities for comment_type to be useful:
- More than one comment form per post.
- Dedicate comment type for each custom post types.
- Ticket system.
- ...

Recently, I have to build a Question & Answer feature for WooCommerce products. Users can ask questions about the product before making a purchase. There are many plugins on wordpress.org which have the same functionality. But all existing plugins use custom post type for that feature. IMO, using the comment system for Q&A is faster and makes more sense.
