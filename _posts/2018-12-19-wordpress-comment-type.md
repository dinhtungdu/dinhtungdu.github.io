---
layout: post
title: WordPress Comment Type
---

This blog is my thought on the WordPress Comment type, not a code snippet as usual.

##IMO, the comment system of WordPress is underused.
If you look into the database or the `get_comments` function, you will notice a column or an argument named `comment_type`. By default, WordPress has three comment types: comment, pingback, and trackback and it doesn't have an official way to add `comment_type`.

There are many possibilities for `comment_type` to be used effectively:
- Multiple comment forms per post.
- Dedicate comment type for each custom post types.
- Ticket system.
- ...

Recently, I have to build a feature which adds the Question & Answer section to WooCommerce Product. Users can ask questions about the product before making a purchase. There are many plugins on wordpress.org which have the same functionality. But all of the existing plugins use custom post type for questions and answers, which is quite heavy for that feature. IMO, using the comment system for Q&A is faster and makes more sense.
