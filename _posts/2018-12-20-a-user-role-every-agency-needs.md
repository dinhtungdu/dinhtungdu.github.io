---
layout: post
title: A user role every agency needs
---

```php
// Add new Manager role for customer.
add_role( 'manager', __( 'Manager', 'text-domain' ),
	array(
		'delete_others_pages'    => true,
		'delete_others_posts'    => true,
		'delete_pages'           => true,
		'delete_posts'           => true,
		'delete_private_pages'   => true,
		'delete_private_posts'   => true,
		'delete_published_pages' => true,
		'delete_published_posts' => true,
		'edit_dashboard'         => true,
		'edit_others_pages'      => true,
		'edit_others_posts'      => true,
		'edit_pages'             => true,
		'edit_posts'             => true,
		'edit_private_pages'     => true,
		'edit_private_posts'     => true,
		'edit_published_pages'   => true,
		'edit_published_posts'   => true,
		'edit_theme_options'     => true,
		'manage_categories'      => true,
		'manage_links'           => true,
		'manage_options'         => true,
		'moderate_comments'      => true,
		'publish_pages'          => true,
		'publish_posts'          => true,
		'read_private_pages'     => true,
		'read_private_posts'     => true,
		'read'                   => true,
		'customize'              => true,
		'list_users'             => true,
		'promote_users'          => true,
		'edit_users'             => true,
		'create_users'           => true,
		'delete_users'           => true,
	)
);
```

## When the default user roles are not enough

I had done many WordPress projects for my customers for six years. There is a common problem which drives me crazy: the customers break their site. Despite detailed documentation and even video tutorials, they still break their site in some magic way. 

At the first attempt, I solve this problem by doing the daily backup. So whenever the customer breaks the sites, I can restore it to the most recent backup. This solution works, but I'm not satisfied with it. There must be a way to prevent the customers from breaking their sites.

The customers here are not the customers of websites, they are the site owners. Most of them will be the administrator of their sites. And here is the point, they're not technical persons. The Admin role is too powerful for non-technical owners, while the Editor role is not powerful enough. Besides, nontechnical site owners feel bloated with all menus and screens an admin has.

## Welcome Manager role

I need a new role for those site owners which they can:
- Have all capabilities of Editor.
- Add new users including Editor, Author.
- Manage other post types: Products, event, form...
- Clear the cache.

That new role won't allow site owners to:
- Switch theme.
- Manage plugins.
- Edit theme/plugin.
- Change site and plugins settings.

By providing site owner enough capabilities, they can't break their sites. Besides, a new role will make the dashboard simpler and easier to use. 
