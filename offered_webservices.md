# EASYMARKETING APIs

## General

This document is versioned using [Semantic Versioning](http://semver.org/). That means:

Given a version number MAJOR.MINOR.PATCH, increment the:

	MAJOR version when you make incompatible API changes,
	MINOR version when you add functionality in a backwards-compatible manner, and
	PATCH version when you make backwards-compatible bug fixes.

The version of this document is **1.0.0**.

This document describes the EASYMARKETING JSON API major version **1**.

There is a changelog of what has changed in between versions at the very bottom of this document.

### Routes

All requests go to

	https://api.easymarketing.de/

### Versioning

The versioning of all API routes is handled via the HTTP `Accept` Header.

An example use of this is the following request:

	curl -H 'Accept: application/vnd.easymarketing.com; version=1' https://api.easymarketing.de/

If you don't provide this version header the API will fallback to the default value which is always the most current API version. At the time of writing that is version 1 so currently there is no difference with or without providing the header. Should the API major version get increased to the next number and your requests do not provide the version then you might experience unexpected results because the version increase might break the behaviour of certain routes.

### Authentication

#### Authenticating calls to the Easymarketing API

The authentication of these API calls is handled via the HTTP `Authorization` Header or URL Params.

An example use of this is the following request:

**Authorization header**:

	curl https://api.easymarketing.de/ -H 'Authorization: Token token="c576f0136149a2e2d9127b3901015545"'

or **URL Parameters**:

	curl 'https://api.easymarketing.de/?access_token=c576f0136149a2e2d9127b3901015545' -I

If a route requires authentication and the caller fails to provide any credentials a **HTTP 401 UNAUTHORIZED** status will be returned.


## User performance data (iFrame)

Returns the user's data from his performance page. The performance page shows
information about clicks, best advertisements and much more. You can embed it
as an iframe.

**Route**

    GET /users/performance?website_url=URL
    (GET /users/:website_url/performance deprecated)

**Params**

    website_url: the url of the vendor without http, required.
    compact: return a compact version of the performance page. This option
      removes the header and the footer from the layout. Default is false.
    width: the width in px of the performance page. Default is 1040
    height: the height in px of the performance page. Default is 1300

**Example**

    https://api.easymarketing.de/users/performance?website_url=foo.com&access_token=c576f0136149a2e2d9127b3901015545
    https://api.easymarketing.de/users/performance?website_url=foo.com&compact=true&width=300&height=300&access_token=c576f0136149a2e2d9127b3901015545

**Response**

    <iframe src="..."></iframe>

## List of campaigns (JSON)

Returns a list of all campaigns that exist.

**Route**

    GET /campaigns

**Example**

    GET https://api.easymarketing.de/campaigns?access_token=c576f0136149a2e2d9127b3901015545
    
**Response**

	[
		{
			label: "easymarketing Shopping",
			active: true,
			campaign_daily_budget: 25,
			campaign_id: 123
		},
		{
			label: "easymarketing Robog",
			active: true,
			campaign_daily_budget: 270,
			campaign_id: 127
		}
	]

## Performance data of campaign (JSON)

Returns detailed performance data on a per-day basis for one campaign. To obtain the ID of a campaign, use the campaign list request described above.

	Remark:
	
	cpo = cost per order
	
	cpl = cost per lead
	
	daily_budget = the budget which was effectively set for that day

**Route**

    GET /campaigns/:id
    
**Params**

    :start_date - the start date that should be used to filter the data.
      (String) in the format: YYYY-MM-DD
    :end_date - the end date that should be used limit the data.
      (String) in the format: YYYY-MM-DD

**Example**

    GET https://api.easymarketing.de/campaigns/409?access_token=c576f0136149a2e2d9127b3901015545&start_date=01-05-2014&end_date=02-05-2014
    
**Response**

	{
		label: "easymarketing Shopping",
		active: true,
		campaign_id: 123,
		campaign_daily_budget: 25,
		day_data: [
			{
				date: "2014-06-02",
				clicks: 100,
				impressions: 2000,
				costs: 25,
				cpo: 2.5,
				cpl: 1.25,
				daily_budget: 25,
				orders: 10,
				leads: 20
			},
			{
				date: "2014-06-03",
				clicks: 27,
				impressions: 4155,
				costs: 28.35,
				cpo: 28.35,
				cpl: 14.18,
				daily_budget: 25,
				orders: 1,
				leads: 2
			}
		]
	}

## Analysis page

This returns the analysis page in an iFrame. The analysis page contains a
bunch of charts and data giving the user hints about how well his website
could perform in AdWords. It does not require an auth token. It will look like this:

![Sample Analysis page embedded](http://i.imgur.com/Q20W2oQ.png)

**Route**

    GET /analysis

**Params**

    website_url: the website url we will prefill (String)
    partner_id: Your referral partner id (String)
    height: Optional height of the iFrame (Integer)
    width: Optional width of the iFrame (Integer)
    small: This will strip headers/footers and return an 800px iFrame. This
      overrides the height and width parameter.

**Example**

    https://api.easymarketing.de/analysis?website_url=foo.com&partner_id=fooshop&width=1200&height=1500&access_token=c576f0136149a2e2d9127b3901015545

    https://api.easymarketing.de/analysis?website_url=foo.com&partner_id=fooshop&small=true&access_token=c576f0136149a2e2d9127b3901015545

**Response**

    <iframe height="1200px" seamless="seamless" src="https://easymarketing.de/analysis/new?small=true" width="800px"></iframe>

## Quick chart

The quick chart can be found on the easymarketing homepage. It gives the user
a quick gadget to play around with. Does not require an auth token. We return
this chart as plain html. Just wrap an iFrame around the code. It will look like this:

![Sample Quick chart](http://i.imgur.com/AsomKpb.png)

**Params**

    website_url: the website url we will prefill (String)
    partner_id: Your referral partner id (String)
    version: The kind of chart you want to return. This determines the size of
    the chart iFrame you would like to integrate. Default is large.

`version=mini` Returns a `357px` * `167px` chart.

`version=medium` Returns a `300px` * `167px` chart.

`version=medium_two` Returns a `325px` * `175px` chart.

`version=large` Returns a `300px` * `250px` chart.

**Route**

    GET /demo_chart

**Example**

    mini version:
    <iframe style="background-color: transparent; border: 0px none transparent;
      padding: 0px; overflow: hidden;" seamless="seamless" scrolling="no"
      frameborder="0" allowtransparency="true" width="357px" height="167px"
      src="https://api.easymarketing.de/demo_chart?website_url=easymarketing.de&partner_id=foo&version=mini">

    medium version:
    <iframe style="background-color: transparent; border: 0px none transparent;
      padding: 0px; overflow: hidden;" seamless="seamless" scrolling="no"
      frameborder="0" allowtransparency="true" width="300px" height="167px"
      src="https://api.easymarketing.de/demo_chart?website_url=easymarketing.de&partner_id=foo&version=medium">

    medium version:
    <iframe style="background-color: transparent; border: 0px none transparent;
      padding: 0px; overflow: hidden;" seamless="seamless" scrolling="no"
      frameborder="0" allowtransparency="true" width="325px" height="175px"
      src="https://api.easymarketing.de/demo_chart?website_url=easymarketing.de&partner_id=foo&version=medium_two">

    large version:
    <iframe style="background-color: transparent; border: 0px none transparent;
      padding: 0px; overflow: hidden;" seamless="seamless" scrolling="no"
      frameborder="0" allowtransparency="true" width="300px" height="250px"
      src="https://api.easymarketing.de/demo_chart?website_url=easymarketing.de&partner_id=foo&version=large">


## Conversion tracker

This returns a tracking code that is to be integrated on the vendor's checkout
success page. This way easymarketing can track actual sales and optimize the
Adwords campaign. This conversion tracker should be stored in your local DB
for later use. This can be requested when the user proceeds and installs the
easymarketing extension in your system.

Please replace the conversion value with the value of the shopping basket __2 times__: In the javascript as well as in the img-tag.

There are two trackers provided, __code__ is for Google and __fb_code__ is for Facebook, as far as a Facebook tracker is available.

Please cache this value refreshing it once in a while (e.g. daily) and __do not call our API on each request__! 

Remark: The tracker may also not be available at the time when the module is configured if it has not been set-up yet in the easymarketing backend. The module should be able to handle this.

**Route**

	GET /conversion_tracker?&website_url=URL
	(GET /conversion_tracker/:website_url deprecated)

**Params**

	:website_url = url of the vendor without http, required

**Example**

	https://api.easymarketing.de/conversion_tracker?website_url=foo.com&access_token=c576f0136149a2e2d9127b3901015545

**Response**

	Status: 200

	{
	  "user_id": 1,
	  "code": '<script type="text/javascript"> /* <![CDATA[ */ var google_conversion_id = 982579417; var google_conversion_language = "en"; var google_conversion_format = "3"; var google_conversion_color = "ffffff"; var google_conversion_label = "cdl_CJe02gcQ2fHD1AM"; var google_conversion_value = 0; var google_remarketing_only = false; /* ]]> */ </script> <script type="text/javascript" src="//www.googleadservices.com/pagead/conversion.js"> </script> <noscript> <div style="display:inline;"> <img height="1" width="1" style="border-style:none;" alt="" src="//www.googleadservices.com/pagead/conversion/982579417/?value=0&amp;label=cdl_CJe02gcQ2fHD1AM&amp;guid=ON&amp;script=0"/> </div> </noscript>',
	  "fb_code": '<script>(function() {var _fbq = window._fbq || (window._fbq = []); ...&amp;cd[value]=0&amp;cd[currency]=EUR&amp;noscript=1" /></noscript>if (!_fbq.loaded) {var fbds = document.createElement('script');fbds.async = true;fbds.src='//connect.facebook.net/en_US/fbds.js';var s = document.getElementsByTagName('script')[0];s.parentNode.insertBefore(fbds, s);_fbq.loaded = true;}})();window._fbq = window._fbq || [];window._fbq.push(['track', '6015228631053', {'value':'0.00','currency':'EUR'}]);</script><noscript><img height="1" width="1" alt="" style="display:none" src="https://www.facebook.com/tr?ev=6015228631053&amp;cd[value]=0&amp;cd[currency]=EUR&amp;noscript=1" /></noscript>'
	}

## Lead tracker

This returns a tracking code that is to be integrated on two pages, the contact pages as 
well as the checkout page (after shopping-basket). This way easymarketing can track leads 
for you. This lead tracker should be stored in your local DB for later use. This can be 
requested when the user proceeds and installs the easymarketing extension in your system.

There are two trackers provided, __code__ is for Google and __fb_code__ is for Facebook, as far as a Facebook tracker is available.

Please cache this value refreshing it once in a while (e.g. daily) and __do not call our API on each request__! 

Remark: The tracker may also not be available at the time when the module is configured if it has not been set-up yet in the easymarketing backend. The module should be able to handle this.

**Route**

        GET /lead_tracker?website_url=URL
        (GET /lead_tracker/:website_url deprecated)

**Params**

        :website_url = url of the vendor without http, required

**Example**

        https://api.easymarketing.de/lead_tracker?website_url=google.de&access_token=c576f0136149a2e2d9127b3901015545

**Response**

        Status: 200

        {
          "user_id": 1,
          "code": '<script type="text/javascript"> /* <![CDATA[ */ var google_conversion_id = 982579417; var google_conversion_language = "en"; var google_conversion_format = "3"; var google_conversion_color = "ffffff"; var google_conversion_label = "cdl_CJe02gcQ2fHD1AM"; var google_conversion_value = 0; var google_remarketing_only = false; /* ]]> */ </script> <script type="text/javascript" src="//www.googleadservices.com/pagead/conversion.js"> </script> <noscript> <div style="display:inline;"> <img height="1" width="1" style="border-style:none;" alt="" src="//www.googleadservices.com/pagead/conversion/982579417/?value=0&amp;label=cdl_CJe02gcQ2fHD1AM&amp;guid=ON&amp;script=0"/> </div> </noscript>',
          "fb_code": '<script>(function() {var _fbq = window._fbq || (window._fbq = []); ...&amp;cd[value]=0&amp;cd[currency]=EUR&amp;noscript=1" /></noscript>if (!_fbq.loaded) {var fbds = document.createElement('script');fbds.async = true;fbds.src='//connect.facebook.net/en_US/fbds.js';var s = document.getElementsByTagName('script')[0];s.parentNode.insertBefore(fbds, s);_fbq.loaded = true;}})();window._fbq = window._fbq || [];window._fbq.push(['track', '6015228631053', {'value':'0.00','currency':'EUR'}]);</script><noscript><img height="1" width="1" alt="" style="display:none" src="https://www.facebook.com/tr?ev=6015228631053&amp;cd[value]=0&amp;cd[currency]=EUR&amp;noscript=1" /></noscript>'
        }

## Remarketing Code

This returns a tracking code that is to be integrated on each page of the site.

Please cache this value refreshing it once in a while (e.g. daily) and __do not call our API on each request__! 

Remark: The tracker may also not be available at the time when the module is configured if it has not been set-up yet in the easymarketing backend. The module should be able to handle this.

**Route**

        GET /google_remarketing_code?website_url=URL

**Params**

        :website_url = url of the vendor without http, required

**Example**

        https://api.easymarketing.de/google_remarketing_code?website_url=google.de&access_token=c576f0136149a2e2d9127b3901015545

**Response**

        Status: 200

        {
          "user_id": 1,
          "code": '<!-- Google Code für ein Remarketing-Tag --><!--------------------------------------------------Remarketing-Tags dürfen nicht mit personenbezogenen Daten verknüpft oder auf Seiten platziert werden, die sensiblen Kategorien angehören. Weitere Informationen und Anleitungen zur Einrichtung des Tags erhalten Sie unter: http://google.com/ads/remarketingsetup ---------------------------------------------------> <script type="text/javascript">var google_tag_params = {ecomm_prodid: 'REPLACE_WITH_VALUE',ecomm_pagetype: 'REPLACE_WITH_VALUE',ecomm_totalvalue: 'REPLACE_WITH_VALUE',};</script><script type="text/javascript">/* <![CDATA[ */var google_conversion_id = 12345678;vargoogle_custom_params = window.google_tag_params;var google_remarketing_only = true;/* ]]> */</script><script type="text/javascript" src="//www.googleadservices.com/pagead/conversion.js"></script><noscript><div style="display:inline;"><img height="1" width="1" style="border-style:none;" alt="" src="//googleads.g.doubleclick.net/pagead/viewthroughconversion/12345678/?value=0&amp;guid=ON&amp;script=0"/> </div></noscript>'
        }

If access token is wrong:

    Status: 401

    {}


If there is an error:

    Status: 400

    {
      errors: ["Error messages"]
    }
    
## Get google site verification data

Retrieve the data required for the google site verification. The returned data must be placed either as html-page in the
root directory of the shop (using html_content and html_file_name) or the data meta-tag can be added to the start page
of the shop instead.

Please cache this value refreshing it once in a while (e.g. daily) and __do not call our API on each request__! 

**Route**

	GET /site_verification_data?website_url
	
**Params**

	:website_url = url of the vendor without http, required
	
**Example**	

	https://api.easymarketing.de/site_verification_data?access_token=123&website_url=foo.com
	
**Response**

If success:

    Status: 200

    {
      user_id: 123,
      website_url: foo.com,
      meta_tag: <meta name=\"google-site-verification\" content=\"bF6M-w27WdtgyuCEXSC293tMA1JfjeN6DcEF20Up8_w\" />,
      html_content: google-site-verification: google646644efd7de5b2d.html,
      html_file_name: google646644efd7de5b2d.html,
    }

If access token is wrong:

    Status: 401

    {}


If there is an error:

    Status: 400

    {
      errors: ["Error messages"]
    }

## Perform google site verification

Executes the site verification. This should be done after the [site verification page](#get-google-site-verification-data) has been installed.
It must be specified if the verification should be performed by requesting the html-page or using the
meta-tag from the start page.

**Route**

	POST /perform_site_verification?website_url&verification_type

**Params**

	:website_url = url of the vendor without http, required
	:verification_type = HTML | META

**Example**

	https://api.easymarketing.de/perform_site_verification?website_url=foo.com&verification_type=HTML&access_token=123

**Response**

If success:

    Status: 200

    {}
    
If access token is wrong:

    Status: 401

    {}


If there is an error:

    Status: 400

    {
      errors: ["Error message"]
    }


## Configure the user's API endpoints

This will configure the user's API endpoints. You need to pass the
proper configurations to us as parameters. We will then check if the website is properly claimed by
the meta tag/html code. We will also proceed and test whether we can extract
data from your API. This request can take some time as several internal
services are called. It would be good to implement a loading animation to give
your user feedback.

See: [API Endpoints](https://github.com/EASYMARKETING/api/blob/master/scope_of_work_integration.md#extract-productscategories-from-remote-shop)

**Route**

    POST /configure_endpoints

**Params**

    {
      website_url: Website URL of the user. Must be the subdomain without http:// or https:// in front. Can only be a top level domain, directories are not allowed. (String)
      access_token: The user's access token (String)
      shop_token: The token that is used by our crawler to access your webservices (String)
      categories_api_endpoint: The URL to access the categories API endpoint (String)
      shop_category_root_id: The ID of the uppermost category. (String)
      products_api_endpoint: The URL of the API endpoint for products (String)
      product_by_id_api_endpoint: The URL of the API endpoint used to access your products (String)
      best_products_api_endpoint: The URL of the API endpoint for best products (String)
      new_products_api_endpoint: The URL of the API endpoint for new products (String)
      shopsystem_info_api_endpoint: The URL of the API endpoint for information about the shopsystem (String)
      api_setup_test_single_product_id: The ID of a product in the shopsystem that is used for testing (String)
    }

**Example**

    POST https://api.easymarketing.de/configure_endpoints?access_token=c576f0136149a2e2d9127b3901015545

    {
      website_url: "example.com",
      access_token: "123",
      shop_token: "123shoptoken123"
      categories_api_endpoint: "https://example.com/easymarketing_api/categories",
      shop_category_root_id: "1",
      products_api_endpoint: "https://example.com/easymarketing_api/products",
      product_by_id_api_endpoint: "https://example.com/easymarketing_api/product_by_id",
      best_products_api_endpoint: "https://example.com/easymarketing_api/best_products",
      new_products_api_endpoint: "https://example.com/easymarketing_api/new_products",
      shopsystem_info_api_endpoint: "https://example.com/easymarketing_api/shopsystem_info",
      api_setup_test_single_product_id: "10"
    }

**Response**


If success:

    Status: 200

    {
      website_url: "example.com",
      access_token: "123",
      shop_token: "123shoptoken123"
      categories_api_endpoint: "https://example.com/easymarketing_api/categories",
      shop_category_root_id: "1",
      products_api_endpoint: "https://example.com/easymarketing_api/products",
      product_by_id_api_endpoint: "https://example.com/easymarketing_api/product_by_id",
      best_products_api_endpoint: "https://example.com/easymarketing_api/best_products",
      new_products_api_endpoint: "https://example.com/easymarketing_api/new_products",
      shopsystem_info_api_endpoint: "https://example.com/easymarketing_api/shopsystem_info",
      api_setup_test_single_product_id: "10",
      api_properly_setup: true,
      website_claimed_by_google: true,
      errors: [],
      parser_errors: []
    }

If access token is wrong:

    Status: 401

    {}


If there is an error:

    Status: 400

    {
      website_url: "example.com",
      access_token: "123",
      shop_token: "123shoptoken123"
      categories_api_endpoint: "https://example.com/easymarketing_api/categories",
      shop_category_root_id: "1",
      products_api_endpoint: "https://example.com/easymarketing_api/products",
      product_by_id_api_endpoint: "https://example.com/easymarketing_api/product_by_id",
      best_products_api_endpoint: "https://example.com/easymarketing_api/best_products",
      new_products_api_endpoint: "https://example.com/easymarketing_api/new_products",
      shopsystem_info_api_endpoint: "https://example.com/easymarketing_api/shopsystem_info",
      api_setup_test_single_product_id: "10",
      website_claimed_by_google: false,
      api_properly_setup: false,
      errors: ["Could not verify ownership of website", "API Check setup did not complete", "No categories found", "No products found", "API Endpoints are missing"]
      parser_errors: ["id must be of type string", "category_id is null"]
    }


The `errors` array contains human readable error messages you can present in
your view to the user. The `parser_errors` array contains errors useful for
debugging.

## Check extraction status

Check the extraction status returning the number of products and categories extracted from the vendor's shop.

**Route**

    GET /extraction_status?website_url

**Params**

    website_url: the url of the vendor without http, required.

**Example**

    GET https://api.easymarketing.de/extraction_status?website_url=foo.com&access_token=c576f0136149a2e2d9127b3901015545

**Response**


If success:

    Status: 200

    {
      num_categories: 10,
      num_products: 3200,
      api_properly_setup: 1395682278,
      // The time when crawling started (UNIX timestamp)
      created_at: 1395682268,
      // The time when the crawler has last been updated (UNIX timestamp)
      updated_at: 1395682240
    }

If access token is wrong:

    Status: 401

    {}


If there is an error:

    Status: 400

    {
      errors: ["Error"]
    }


The `errors` array contains human readable error messages you can present in
your view to the user. 

## Facebook Like badge

The vendor can make use of the easymarketing facebook like badge. The route
returns a like button code that should be integrated on the vendor's checkout
success page. Triggering it will like the vendor's facebook page. The vendor
provides the details to his facebook fanpage in the easymarketing system.

**Route**

	GET /facebook_badge?website_url=URL
	(GET /facebook_badge/:website_url deprecated)

**Params**

	:website_url = the website url of the easymarketing user. Required.

**Example**

	https://api.easymarketing.de/facebook_badge?website_url=easymarketing.de?access_token=c576f0136149a2e2d9127b3901015545

**Response**

	Status: 200

    <iframe src="//www.facebook.com/plugins/like.php?href=https%3A%2F%2Fwww.facebook.com%2FGoogle&amp;width=450&amp;height=46&amp;colorscheme=light&amp;layout=button_count&amp;action=like&amp;show_faces=true&amp;send=true&amp;appId=270892269593470" scrolling="no" frameborder="0" style="border:none; overflow:hidden; width:450px; height:46px;" allowTransparency="true"></iframe>

Note this retuns the default facebook like button. You can optionally adjust
width/size of the like button if required on your own by modifying the html.

For a full documentation on the like button refer to:

	https://developers.facebook.com/docs/reference/plugins/like/

### Further suggestions for the like button.

We suggest adjusting the behavior of the like button slightly. This will
result in a higher marketing impact for the vendor.


**Case 1**
If the vendor purchased a single product, replace the url in the like button
to point to the URL of the product the user purchased. Example:

User purchased only baby diapers.

    http://foo.com/baby_diapers

Adjust the like code:

    <iframe src="//www.facebook.com/plugins/like.php?href=#{url_encoded_link_to_the_product}&amp;width=450&amp;height=46&amp;colorscheme=light&amp;layout=button_count&amp;action=like&amp;show_faces=true&amp;send=true&amp;appId=270892269593470" scrolling="no" frameborder="0" style="border:none; overflow:hidden; width:450px; height:46px;" allowTransparency="true"></iframe>

Replace the `#{url_encoded_link_to_the_product}` with your url.

With the above example that would result in:

    <iframe src="//www.facebook.com/plugins/like.php?href=http%3A%2F%2Ffoo.com%2Fbaby_diapers%0A&amp;width=450&amp;height=46&amp;colorscheme=light&amp;layout=button_count&amp;action=like&amp;show_faces=true&amp;send=true&amp;appId=270892269593470" scrolling="no" frameborder="0" style="border:none; overflow:hidden; width:450px; height:46px;" allowTransparency="true"></iframe>

You should make sure that the URL that is liked includes proper Facebook open
graph tags. Without the tags the like will not include images in the users'
friends steam. Refer to the following link for a full documentation:

    http://ogp.me/

**Case 2**
User purchased many products, use the default code:

    <iframe src="//www.facebook.com/plugins/like.php?href=https%3A%2F%2Fwww.facebook.com%2FGoogle&amp;width=450&amp;height=46&amp;colorscheme=light&amp;layout=button_count&amp;action=like&amp;show_faces=true&amp;send=true&amp;appId=270892269593470" scrolling="no" frameborder="0" style="border:none; overflow:hidden; width:450px; height:46px;" allowTransparency="true"></iframe>


## Push a product for special Promotion to EASYMARKETING

EASYMARKETING offers special promotions for individual products. We will
publish them on the vendor's Facebook page. This route is synchronous products
pushed to the route are instantly published on the vendor's facebook page.

**Route**

    POST users/facebook_status?website_url=URL&product_id=PRODUCT_ID
    (POST users/:website_url/products/:product_id/facebook_status deprecated)

**Params**

    :product_id = the product id of a specific product that should be promoted.
    :website_url = the website url of the easymarketing user. Required.

**Example**

    POST https://api.easymarketing.de/users/foobar.com/products/1/facebook_status?access_token=c576f0136149a2e2d9127b3901015545

**Response**

    Status: 200 if all went fine.

    {
      "website_url": "foobar.com",
      "product_id": 1
    }

    Status: 202 if product not found in our database. We will attempt to extract it..

    {
      "website_url": "foobar.com",
      "product_id": 1
    }

    Status: 400 if other errors occur and can not post to facebook.

    {
      "website_url": "foobar.com",
      "product_id": 1
    }

## IDs for retargeting

**Route**

    GET /retargeting_id

**Example**

    GET https://api.easymarketing.de/retargeting_id?access_token=c576f0136149a2e2d9127b3901015545

**Response**

If success:

    Status: 200

    {
      adscale_id: "123456789",
      conversion_id: "AAAAAAAA"
    }

If success but no retargeting data added to the account:

    Status: 202

    {
      adscale_id: null,
      conversion_id: null
    }

If access token is wrong:

    Status: 401

    {}

## Changelog

Jan 27 2013 

* Split documents in two.



Oct 14th 2013

* Added more information concerning the versioning of the API and this document
* Added "Authenticating calls from Easymarketing towards your API Endpoints" section
* Added `url` attribute to Categories
* Added `google_category`, `discount_absolute` and `discount_percentage` to Products
	
