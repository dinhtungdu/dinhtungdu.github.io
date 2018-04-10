---
layout: post
title: Change shop_thumbnail size in WooCommerce 3.3.3 and later versions
---

![WooCommerce]({{ site.baseurl }}/images/woocommerce-tutorial.jpg)

After updated WooCommerce, you can't find anywhere to change shop_thumbnail size? You're not alone! We all got that problem because The option had been hiden.

We can solve this problem by using following snippet. Paste it in your theme's `functions.php` file and change the size you need:

```
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

## Finding the answer
> In this section I will describe the process when I find the answer to this problem.

First all know the image size of WooCommerce is `shop_thumbnail`. Then I look up at register image size function inside the plugin. I found a function name `add_image_sizes()` in class `woocommerce/includes/class-woocommerce.php` which is:

```
public function add_image_sizes() {
	$thumbnail         = wc_get_image_size( 'thumbnail' );
	$single            = wc_get_image_size( 'single' );
	$gallery_thumbnail = wc_get_image_size( 'gallery_thumbnail' );

	add_image_size( 'woocommerce_thumbnail', $thumbnail['width'], $thumbnail['height'], $thumbnail['crop'] );
	add_image_size( 'woocommerce_single', $single['width'], $single['height'], $single['crop'] );
	add_image_size( 'woocommerce_gallery_thumbnail', $gallery_thumbnail['width'], $gallery_thumbnail['height'], $gallery_thumbnail['crop'] );

	// Registered for bw compat. @todo remove in 4.0.
	add_image_size( 'shop_catalog', $thumbnail['width'], $thumbnail['height'], $thumbnail['crop'] );
	add_image_size( 'shop_single', $single['width'], $single['height'], $single['crop'] );
	add_image_size( 'shop_thumbnail', $gallery_thumbnail['width'], $gallery_thumbnail['height'], $gallery_thumbnail['crop'] );
}
```
_From this function we know that the team change name of images size come with WooCommerce. The old sizes are still added but will be removed in 4.0. So it's good to change image size related code in your theme or plugin._

Take a look at last line, we can see the `shop_thumbnail` use `$gallery_thumbnail` dimension from `wc_get_image_size()`. Go to the definition of that function, we have: (I stripped some parts)
```php
function wc_get_image_size( $image_size ) {
...
	} elseif ( 'gallery_thumbnail' === $image_size ) {
		$size['width']  = absint( wc_get_theme_support( 'gallery_thumbnail_image_width', 100 ) );
		$size['height'] = $size['width'];
		$size['crop']   = 1;

	} 
...

return apply_filters( 'woocommerce_get_image_size_' . $image_size, $size );
}
```
That is how gallery thumbnail size is setup. We don't have any customizer option to change it but we have a filter in return statement. From here we got the solution for our problem as describe in the begining of this blog, using a filter to change the size.
