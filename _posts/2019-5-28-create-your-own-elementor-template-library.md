---
layout: post
title: Create your own Elementor template library
---

Elementor has a great feature called Template Library which let user import predefined template by a single click. But it doesn't let us - the developers - include our templates into that library (by default).

But now we can : ).

Thanks to [@davelavoie](https://github.com/davelavoie) for sharing his idea and allowing me to use it.

## How does it work?

Elementor Template library is made from `Source`s. Elementor creates sources then registers them to use in the library.

![Elementor Template library]({{ site.baseurl }}/images/2019/elementor-template-library-register-default.png)

Guess what? We can follow this approach just fine by creating our source and register it.


### Create a small plugin to hold our code

There are many ways to create a plugin for WordPress. In this post, I use the scaffold command of WP CLI to kickstart our plugin:
```
$ wp scaffold plugin custom-elementor-source
```

### Unregister the default remote source

The first step we want to do is unregister the default remote source. Why? Because the Elementor Blocks and Pages are designed to work with the `remote` source only, so we must replace the Elementor `remote` source with our own. The title of this post now can change to "Override Elementor remote template".

Put the code bellow to the main plugin file we created above.
```php
/**
 * Register our custom source.
 */
add_action( 'elementor/init', function() {
    include 'includes/source.php';
    \Elementor\Plugin::instance()->templates_manager->unregister_source( 'remote' );
    \Elementor\Plugin::instance()->templates_manager->register_source( 'Elementor\TemplateLibrary\Source_Custom' );
}, 15 );
```

### Create our Source_Custom

Now let's create a new file `includes/source.php` in our plugin folder and get our hand dirty.
```php
<?php
namespace Elementor\TemplateLibrary;

use Elementor\Api;
use Elementor\Plugin;

/**
 * Custom template library remote source.
 */
class Source_Custom extends Source_Base {
    ...
}
```

We're creating a new remote source for Elementor so that it will look like the Elementor one. Copy the content of `Source_Remote` located at `elementor/includes/template-library/sources/remote.php` to our newly created `Source_Custom`. At this time, you can open Elementor editor and check the Template Library.

Nothing happened? No, it may look identically, but things happened. The source is now controlled by our `Source_Custom`. The reason why it seems as before is we clone the Elementor remote source class. We're going to modify it to make it work for us very soon.

Now examine the `Source_Custom::get_items`:

```php
public function get_items( $args = [] ) {
    $library_data = Api::get_library_data();

    $templates = [];

    if ( ! empty( $library_data['templates'] ) ) {
        foreach ( $library_data['templates'] as $template_data ) {
            $templates[] = $this->prepare_template( $template_data );
        }
    }

    return $templates;
}
```

This method fetches templates information from Elementor server then save it to the database (`wp_options` table). The method uses `Elementor\Api::get_library_data` to get library information. To make source get templates information from our endpoint, we need to implement our own `get_library_data` method.

Rename `Api::get_library_data` to `self::get_library_data` then add these new properties and methods to our `Source_Custom`:

```php
/**
 * New library option key.
 */
const LIBRARY_OPTION_KEY = 'custom_remote_info_library';

/**
 * Timestamp cache key to trigger library sync.
 */
const TIMESTAMP_CACHE_KEY = 'custom_remote_update_timestamp';

/**
 * API info URL.
 *
 * Holds the URL of the info API.
 *
 * @access public
 * @static
 *
 * @var string API info URL.
 */
public static $api_info_url = 'YOUR_API_INFO_URL_HERE';
```

, and these medthods:

```php
/**
 * Get templates data.
 *
 * Retrieve the templates data from a remote server.
 *
 * @access public
 * @static
 *
 * @param bool $force_update Optional. Whether to force the data update or
 *                                     not. Default is false.
 *
 * @return array The templates data.
 */
public static function get_library_data( $force_update = false ) {
    self::get_info_data( $force_update );

    $library_data = get_option( self::LIBRARY_OPTION_KEY );

    if ( empty( $library_data ) ) {
        return [];
    }

    return $library_data;
}

/**
 * Get info data.
 *
 * This function notifies the user of upgrade notices, new templates and contributors.
 *
 * @access private
 * @static
 *
 * @param bool $force_update Optional. Whether to force the data retrieval or
 *                                     not. Default is false.
 *
 * @return array|false Info data, or false.
 */
private static function get_info_data( $force_update = false ) {

    $elementor_update_timestamp = get_option( '_transient_timeout_elementor_remote_info_api_data_' . ELEMENTOR_VERSION );
    $update_timestamp = get_transient( self::TIMESTAMP_CACHE_KEY );

    if ( $force_update || ! $update_timestamp || $update_timestamp != $elementor_update_timestamp ) {
        $timeout = ( $force_update ) ? 25 : 8;

        $response = wp_remote_get( self::$api_info_url, [
            'timeout' => $timeout,
            'body' => [
                // Which API version is used.
                'api_version' => ELEMENTOR_VERSION,
                // Which language to return.
                'site_lang' => get_bloginfo( 'language' ),
            ],
        ] );

        if ( is_wp_error( $response ) || 200 !== (int) wp_remote_retrieve_response_code( $response ) ) {
            set_transient( self::TIMESTAMP_CACHE_KEY, [], 2 * HOUR_IN_SECONDS );

            return false;
        }

        $info_data = json_decode( wp_remote_retrieve_body( $response ), true );

        if ( empty( $info_data ) || ! is_array( $info_data ) ) {
            set_transient( self::TIMESTAMP_CACHE_KEY, [], 2 * HOUR_IN_SECONDS );

            return false;
        }

        if ( isset( $info_data['library'] ) ) {
            update_option( self::LIBRARY_OPTION_KEY, $info_data['library'], 'no' );
        }

        set_transient( self::TIMESTAMP_CACHE_KEY, $elementor_update_timestamp, 12 * HOUR_IN_SECONDS );
    }

    return $info_data;
}
```

From now, we can change the `$api_info_url` to our info URL which returns the library data and get them displayed in the Elementor Template Library. (more on this later).

We have a remaining call to Elementor Api in `Source_Custom::get_data`. We do the same process as above, change `Api::get_template_content` to `self::get_template_content` and add the following code to `Source_Custom` class.

```php
/**
 * API get template content URL.
 *
 * Holds the URL of the template content API.
 *
 * @access private
 * @static
 *
 * @var string API get template content URL.
 */
private static $api_get_template_content_url = 'YOUR_TEMPLATE_CONTENT_URL_HERE/%d';
```

```php
/**
 * Get template content.
 *
 * Retrieve the templates content received from a remote server.
 *
 * @access public
 * @static
 *
 * @param int $template_id The template ID.
 *
 * @return array The template content.
 */
public static function get_template_content( $template_id ) {
    $url = sprintf( self::$api_get_template_content_url, $template_id );

    $body_args = [
        // Which API version is used.
        'api_version' => ELEMENTOR_VERSION,
        // Which language to return.
        'site_lang' => get_bloginfo( 'language' ),
    ];

    /**
     * API: Template body args.
     *
     * Filters the body arguments send with the GET request when fetching the content.
     *
     * @param array $body_args Body arguments.
     */
    $body_args = apply_filters( 'elementor/api/get_templates/body_args', $body_args );

    $response = wp_remote_get( $url, [
        'timeout' => 40,
        'body' => $body_args,
    ] );

    if ( is_wp_error( $response ) ) {
        return $response;
    }

    $response_code = (int) wp_remote_retrieve_response_code( $response );

    if ( 200 !== $response_code ) {
        return new \WP_Error( 'response_code_error', sprintf( 'The request returned with a status code of %s.', $response_code ) );
    }

    $template_content = json_decode( wp_remote_retrieve_body( $response ), true );

    if ( isset( $template_content['error'] ) ) {
        return new \WP_Error( 'response_error', $template_content['error'] );
    }

    if ( empty( $template_content['data'] ) && empty( $template_content['content'] ) ) {
        return new \WP_Error( 'template_data_error', 'An invalid data was returned.' );
    }

    return $template_content;
}
```

Change `$api_get_template_content_url` to our endpoint and now we have fully working template source. :)

> You can grab the full source code of this plugin here: [dinhtungdu/custom-elementor-source](https://github.com/dinhtungdu/custom-elementor-source)

## Testing our new source

To test your new source, we need a working API that returns library information and template detail.

The Elementor API returns data in JSON format; then we need to do the same: returning the JSON which has the same structure as Elementor. But we don't need to be complicated here. I created a dummy API using ... Github. Yes, Github, and it's [here](https://github.com/dinhtungdu/custom-elementor-library-dummy-api). Just storing some JSON files on Github, linking to them in our source, and we've done.

> I use this method for our testing purpose only. I don't recommend using this method in production.

Now we have a working API. Let's edit `includes/source.php` to make it work with that API.

```php
    /**
     * API info URL.
     *
     * Holds the URL of the info API.
     *
     * @access public
     * @static
     *
     * @var string API info URL.
     */
    public static $api_info_url = 'https://raw.githubusercontent.com/dinhtungdu/custom-elementor-library-dummy-api/master/info.json';
    /**
     * API get template content URL.
     *
     * Holds the URL of the template content API.
     *
     * @access private
     * @static
     *
     * @var string API get template content URL.
     */
    private static $api_get_template_content_url = 'https://raw.githubusercontent.com/dinhtungdu/custom-elementor-library-dummy-api/master/templates/%d.json';
```

> You can grab the modified plugin [here](https://github.com/dinhtungdu/custom-elementor-source/tree/example/) to get started right away.

After syncing the library, you should see the Template Library has only one template, which is the Hero UI Kit.

![New Template Library with our custom source]({{ site.baseurl }}/images/2019/new-elementor-library-with-custom-source.png)

## Conclusion

Now, nothing stops you from bringing a better user experience for your solution/product. The customers can browse your templates and easily insert them. No more manual importing, no more headache!

To override the Elementor template library, we need two components: a working API server and a remote source. By following this blog, you should be able to create your remote source.

About the API server, you can use anything to build it. Even hosting raw JSON files as I did in this blog works (but I don't recommend using it in production). REST API has been merged into WordPress core since the 4.7, so it's a good idea to using WordPress as your API server.

> FYI: I'm building a WordPress plugin to help you build your API server. I'll update here when I release it.

Hope this help! Feel free to leave some comments, I'd love to reply to them ;).
