---
layout: post
title: Change shop_thumbnail size in WooCommerce 3.3.3 and later versions
---

({{ site.baseurl }}/images/woocommerce-tutorial.jpg)

After updated WooCommerce, you can't find anywhere to change shop_thumbnail size? You're not alone! We all got that problem because The option had been hiden.

We can solve this problem by using following snippet. Paste it in your theme's `functions.php` file and change the size you need:

```php
add_filter( 'woocommerce_get_image_size_gallery_thumbnail', 'your_theme_change_shop_thumbnail' );
function your_theme_change_shop_thumbnail( $size ) {

	return array(
		'width'  => 126,
		'height' => 93,
		'crop'   => 1,
	);
}
```

If you run newer PHP version, you can use shorter version with anonymous function:

```php
add_filter( 'woocommerce_get_image_size_gallery_thumbnail', function () {

	return array(
		'width'  => 126,
		'height' => 93,
		'crop'   => 1,
	);
} );
```
