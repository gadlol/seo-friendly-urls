# osCommerce SEO Friendly Urls!

SEO Friendly Urls is an addon for the osCommerce CMS that will change the default urls into
what a modern website's urls should be!
Read more: [https://www.johnbarounis.com/coding/oscommerce/seo-friendly-urls-addon](https://www.johnbarounis.com/coding/oscommerce/seo-friendly-urls-addon)

# INSTALLATION GUIDE

It's super easy and you don't need to make big core changes.

## STEP 1:

Upload the seo_friendly_urls.php file into catalog/includes/classes/

## STEP 2:

Edit catalog/includes/application_top.php

Find:

    // Shopping cart actions

around line 314 and add ABOVE:

    require('includes/classes/seo_friendly_urls.php');

If you have set the **Display Cart After Adding Product** option to “false” then we need to tell SEO
Friendly Urls to return to same page after adding a product to the cart.

So find:

    if (DISPLAY_CART == 'true') {
    	$goto = 'shopping_cart.php';
	    $parameters = array('action', 'cPath', 'products_id', 'pid');
    } else {
	    $goto = $PHP_SELF;
	    if ($_GET['action'] == 'buy_now') {
		    $parameters = array('action', 'pid', 'products_id');
	    } else {
		    $parameters = array('action', 'pid');
	    }
	}

which is located a few lines bellow 314 line, and replace with:

    if (DISPLAY_CART == 'true') {
	    $goto = 'shopping_cart.php';
	    $parameters = array('action', 'cPath', 'products_id', 'pid');
    } else {
	    $goto = $PHP_SELF;
	    if ($_GET['action'] == 'buy_now') {
		    $parameters = array('action', 'pid', 'products_id');
	    } else {
		    $parameters = array('action', 'pid');
	    }
	    //SEO Friendly Urls Modification get the right return url when we dont display cart after certain action
	    if(isset($seo_friendly_urls) && $seo_friendly_urls->enabled){
	        $gt = $seo_friendly_urls->process_goto_link();
		    if($gt != ''){
			    $goto=$gt;
			}
	    }
    }
    //SEO Friendly Urls Modification so to exclude atts GET variable used when user has selected attributes
    $parameters[] = 'atts';

*If you change **Display Cart After Adding Product** to “true” then there is no need to replace the
above code with the original as it also works.*

## STEP 3:

Edit catalog/index.php.

find `require('includes/application_top.php');`

and add after it, the code bellow:

    if( isset($seo_friendly_urls) && $seo_friendly_urls->enabled && $seo_friendly_urls->page_type !== 'category' ){
	    if( $seo_friendly_urls->include_page !== '' ){
	        $PHP_SELF = $seo_friendly_urls->include_page;
		    include $seo_friendly_urls->include_page;
		    exit;
	    }
    }

## STEP 4:

Edit catalog/includes/functions/html_output.php

> Here we must to tell the tep_href_link function to construct the
> proper urls so:

Inside the **tep_href_link** function find:

    if (tep_not_null($parameters)) {
	    $link .= $page . '?' . tep_output_string($parameters);
	    $separator = '&';
    }else{
	    $link .= $page;
	    $separator = '?';
    }

and replace with:

    //SEO Friendly Urls
    global $seo_friendly_urls;
    if( isset($seo_friendly_urls) && $seo_friendly_urls->enabled ){
	    extract($seo_friendly_urls->process_link($page,$parameters));
	    $link .= $seolink;
    } else {
        if(tep_not_null($parameters)) {
		    $link .= $page . '?' . tep_output_string($parameters);
		    $separator = '&';
	    }else{
		    $link .= $page;
		    $separator = '?';
	    }
    }

> As you can see we also keep the default functionality of the
> tep_href_link in case we disable the addon.

## STEP 5:

Edit catalog/.htaccess

Add at the bottom:

    <IfModule mod_rewrite.c>
	    Options +FollowSymLinks
	    RewriteEngine on
	    RewriteCond %{REQUEST_FILENAME} !-f
	    RewriteCond %{REQUEST_FILENAME} !-d
	    RewriteRule .* index.php [L]
    </IfModule>

> Note: same - equivalent rules must applied for not apache servers such
> as nginx.

## STEP 6:

Edit catalog/product_info.php and change:

    require('includes/application_top.php');
to

    require_once('includes/application_top.php');

> *Above the same micro edit needs to be done into every page in the root, only if you want to alias it. i.e. shopping_cart.php to cart or
> even specials.php to just specials.*

When customers add to cart a product that has attributes then the default functionality produces a
url like this: www.mystore.com/product_info.php?products_id=8{4}4{13}7 that way when you go
to cart and click on that link it redirects you to the products page having the attributes selected to
those customer had selected during add to cart method. So in order for SEO Friendly Urls to keep
that functionality it produces the standard products url and it adds a get variable called atts i.e.
www.mystore.com/dvd-movies/action/hero?atts=8{4}4{13}7 .

If you want that feature you must find in product_info.php the:

    if (is_string($HTTP_GET_VARS['products_id']) && isset($cart->contents[$HTTP_GET_VARS['products_id']]['attributes'][$products_options_name['products_options_id']])) {
	    $selected_attribute = $cart->contents[$HTTP_GET_VARS['products_id']]['attributes'][$products_options_name['products_options_id']];
    } else {
	    $selected_attribute = false;
    }

and replace with:

    //SEO Friendly Urls Modification for displaying the users attributes selections
    if(isset($seo_friendly_urls) && $seo_friendly_urls->enabled && isset($HTTP_GET_VARS['atts']) ){
	    if (is_string($HTTP_GET_VARS['atts']) && isset($cart->contents[$HTTP_GET_VARS['atts']]['attributes'][$products_options_name['products_options_id']])) {
		    $selected_attribute = $cart->contents[$HTTP_GET_VARS['atts']]['attributes'][$products_options_name['products_options_id']];
	    } else {
		    $selected_attribute = false;
	    }
    }else{
	    if (is_string($HTTP_GET_VARS['products_id']) && isset($cart->contents[$HTTP_GET_VARS['products_id']]['attributes'][$products_options_name['products_options_id']])) {
		    $selected_attribute = $cart->contents[$HTTP_GET_VARS['products_id']]['attributes'][$products_options_name['products_options_id']];
	    } else {
		    $selected_attribute = false;
	    }
    }
    //SEO Friendly Urls Modification for displaying the users attributes selections

> As you can see we also keep the default functionality in case we
> disable the addon.

Also make a search for tep_get_all_get_params and add the 'atts' if missing:

    tep_href_link(FILENAME_PRODUCT_REVIEWS, tep_get_all_get_params(array('atts')))


    <?php echo tep_draw_form('cart_quantity', tep_href_link(FILENAME_PRODUCT_INFO,
    tep_get_all_get_params(array('action','atts')). 'action=add_product', 'NONSSL'), 'post', 'class="form-
    horizontal" role="form"'); ?>


## STEP 7:

Navigate to a category or a product page so the SEO Friendly Urls to be instantiated and then go to
admin panel and expand the Configuration Menu. Inside that, you'll find a link called:

**SEO Friendly Urls PRO Edition**

Click on the Enable SEO Friendly Urls and set to True. After that your store's urls have been
changed.

# Notes:

**IMPORTANT! Set the default oscommerce option Use search-engine safe urls to “false”**

SEO Friendly Urls cache needs to be reset every time changed or added a product/category from
admin panel. That is normal because there is not a default way to reset the cache. You could easily
do that if you edit the admin/categories.php and add a small piece of code on every action.
Something that can be pain in the ass. So there is a work around that:.

Here is a small piece of code to be added in admin/includes/functions/general.php inside the
tep_reset_cache_block before the last “}” (closing curly bracket) :

    //reset SEO FRIENDLY URLS cache
    if(defined('SEO_FRIENDLY_URLS_STATUS') && SEO_FRIENDLY_URLS_STATUS=='True'){
	    tep_db_query("DELETE FROM seo_friendly_urls WHERE seo_friendly_urls_key='cache_aliases' ");
	    if(file_exists(DIR_FS_CACHE.'seo_friendly_urls.cache')) @unlink(DIR_FS_CACHE.'seo_friendly_urls.cache');
	    if(extension_loaded('apc')) apc_delete('seo_friendly_urls_cache_aliases');
    }
    //reset SEO FRIENDLY URLS cache

- *Note: you must have set default osc cache mechanism to true in order for this to work because
tep_reset_cache_block called only when default osc cache is set to true*.

# THATS IT:

If you have any difficulty, issues about installation do not hesitate contacting me to:

https://www.johnbarounis.com/coding/oscommerce/seo-friendly-urls-addon/support
or
https://www.johnbarounis.com/contact

Donations: https://www.johnbarounis.com/coding/oscommerce/seo-friendly-urls-addon/donate
