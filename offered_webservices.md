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

	http://api.easymarketing.de/

### Versioning

The versioning of all API routes is handled via the HTTP `Accept` Header.

An example use of this is the following request:

	curl -H 'Accept: application/vnd.easymarketing.com; version=1' http://api.easymarketing.de/

If you don't provide this version header the API will fallback to the default value which is always the most current API version. At the time of writing that is version 1 so currently there is no difference with or without providing the header. Should the API major version get increased to the next number and your requests do not provide the version then you might experience unexpected results because the version increase might break the behaviour of certain routes.

### Authentication

#### Authenticating calls to the Easymarketing API

The authentication of these API calls is handled via the HTTP `Authorization` Header or URL Params.

An example use of this is the following request:

**Authorization header**:

	curl http://api.easymarketing.de/ -H 'Authorization: Token token="c576f0136149a2e2d9127b3901015545"'

or **URL Parameters**:

	curl 'http://api.easymarketing.de/?access_token=c576f0136149a2e2d9127b3901015545' -I

If a route requires authentication and the caller fails to provide any credentials a **HTTP 401 UNAUTHORIZED** status will be returned.


## User performance data (iFrame)

Returns the user's data from his performance page. The performance page shows
information about clicks, best advertisements and much more. You can embed it
as an iframe.

**Route**

    GET /users/:website_url/performance

**Params**

    website_url: the url of the vendor without http, required.
    compact: return a compact version of the performance page. This option
      removes the header and the footer from the layout. Default is false.
    width: the width in px of the performance page. Default is 1040
    height: the height in px of the performance page. Default is 1300

**Example**

    GET /users/foo.com/performance
    GET /users/foo.com/performance?compact=true&width=300&height=300

**Response**

    <iframe src="..."></iframe>

## User performance data (JSON)

Returns the same data as the previous example, just in JSON format so you can
build your own UI. The `start_date` and `end_date` parameters can be omitted.
You will receive all time data then.

**Route**

    GET /users/:website_url/performance.json

**Params**

    :website_url - the url of the vendor without http, required.
    :start_date - the start date that should be used to filter the data.
      (Optional String) in the format: YYYY-MM-DD
    :end_date - the end date that should be used limit the data.
      (Optional String) in the format: YYYY-MM-DD
    :group_by_day - Group the result data by day if there are start_date and
      end_date params. This is optional and only works if the start_date and
      end_date params are provided.


**Example**

    GET /users/foo.com/performance.json

**Response**

    {
      "clicks": 100,
      "impressions": 0,
      "click_through_rate": 0.12,
      "costs": 0,
      "average_cpc": 1.5,
      "conversions": 0,
      "currency": "EUR"
    }

**Example**

    GET /users/foo.com/performance.json?start_date=2013-02-15&end_date=2013-02-16&group_by_day=true

**Response**

    {
      "2013-02-15":
        {
          "clicks": 100,
          "impressions": 0,
          "click_through_rate": 0.12,
          "costs": 0,
          "average_cpc": 1.5,
          "conversions": 0,
          "currency": "EUR"
        },

      "2013-02-16":
        {
          "clicks": 100,
          "impressions": 0,
          "click_through_rate": 0.12,
          "costs": 0,
          "average_cpc": 1.5,
          "conversions": 0,
          "currency": "EUR"
        },
    }


**Response explanation**

    clicks: The amount of clicks the user received. Integer.
    impressions: The amount of impressions indicating how often the ad has been shown. Integer
    click_through_rate: The percentage of how many impressions resulted in clicks. Float between 0 and 1.
    costs: The net costs that were generated. This does not include VAT and EASYMARKETING margin. Float.
    average_cpc: The average price each clicked cost. Float.
    conversions: The amount of sales that have been generated. Integer.
    currency: ISO code of the user's currency. String.

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

    GET /analysis?website_url=foo.com&partner_id=fooshop&width=1200&height=1500

    GET /analysis?website_url=foo.com&partner_id=fooshop&small=true

**Response**

    <iframe height="1200px" seamless="seamless" src="http://easymarketing.de/analysis/new?small=true" width="800px"></iframe>

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
      src="http://api.easymarketing.de/demo_chart?website_url=easymarketing.de&partner_id=foo&version=mini">

    medium version:
    <iframe style="background-color: transparent; border: 0px none transparent;
      padding: 0px; overflow: hidden;" seamless="seamless" scrolling="no"
      frameborder="0" allowtransparency="true" width="300px" height="167px"
      src="http://api.easymarketing.de/demo_chart?website_url=easymarketing.de&partner_id=foo&version=medium">

    medium version:
    <iframe style="background-color: transparent; border: 0px none transparent;
      padding: 0px; overflow: hidden;" seamless="seamless" scrolling="no"
      frameborder="0" allowtransparency="true" width="325px" height="175px"
      src="http://api.easymarketing.de/demo_chart?website_url=easymarketing.de&partner_id=foo&version=medium_two">

    large version:
    <iframe style="background-color: transparent; border: 0px none transparent;
      padding: 0px; overflow: hidden;" seamless="seamless" scrolling="no"
      frameborder="0" allowtransparency="true" width="300px" height="250px"
      src="http://api.easymarketing.de/demo_chart?website_url=easymarketing.de&partner_id=foo&version=large">


## Conversion tracker

This returns a tracking code that is to be integrated on the vendor's checkout
success page. This way easymarketing can track actual sales and optimize the
Adwords campaign. This conversion tracker should be stored in your local DB
for later use. This can be requested when the user proceeds and installs the
easymarketing extension in your system.

**Route**

	GET /conversion_tracker/:website_url

**Params**

	:website_url = url of the vendor without http, required

**Example**

	GET /conversion_tracker/google.de

**Response**

	Status: 200

	{
	  "user_id": 1,
	  "code": '<script type="text/javascript"> /* <![CDATA[ */ var google_conversion_id = 982579417; var google_conversion_language = "en"; var google_conversion_format = "3"; var google_conversion_color = "ffffff"; var google_conversion_label = "cdl_CJe02gcQ2fHD1AM"; var google_conversion_value = 0; var google_remarketing_only = false; /* ]]> */ </script> <script type="text/javascript" src="//www.googleadservices.com/pagead/conversion.js"> </script> <noscript> <div style="display:inline;"> <img height="1" width="1" style="border-style:none;" alt="" src="//www.googleadservices.com/pagead/conversion/982579417/?value=0&amp;label=cdl_CJe02gcQ2fHD1AM&amp;guid=ON&amp;script=0"/> </div> </noscript>',
	  "img": '<img height="1" width="1" style="border-style:none;" alt="" src="//www.googleadservices.com/pagead/conversion/982579417/?value=0&amp;label=cdl_CJe02gcQ2fHD1AM&amp;guid=ON&amp;script=0"/>'
	}

## Lead tracker

This returns a tracking code that is to be integrated on the vendor's registration
success page. This way easymarketing can track leads for you. This lead tracker should 
be stored in your local DB for later use. This can be requested when the user proceeds 
and installs the easymarketing extension in your system.

**Route**

        GET /lead_tracker/:website_url

**Params**

        :website_url = url of the vendor without http, required

**Example**

        GET /lead_tracker/google.de

**Response**

        Status: 200

        {
          "user_id": 1,
          "code": '<script type="text/javascript"> /* <![CDATA[ */ var google_conversion_id = 982579417; var google_conversion_language = "en"; var google_conversion_format = "3"; var google_conversion_color = "ffffff"; var google_conversion_label = "cdl_CJe02gcQ2fHD1AM"; var google_conversion_value = 0; var google_remarketing_only = false; /* ]]> */ </script> <script type="text/javascript" src="//www.googleadservices.com/pagead/conversion.js"> </script> <noscript> <div style="display:inline;"> <img height="1" width="1" style="border-style:none;" alt="" src="//www.googleadservices.com/pagead/conversion/982579417/?value=0&amp;label=cdl_CJe02gcQ2fHD1AM&amp;guid=ON&amp;script=0"/> </div> </noscript>',
	        "img": '<img height="1" width="1" style="border-style:none;" alt="" src="//www.googleadservices.com/pagead/conversion/982579417/?value=0&amp;label=cdl_CJe02gcQ2fHD1AM&amp;guid=ON&amp;script=0"/>'
        }

## Request Site verification

**Route**

	GET /site_verification/?website_url=xxx
	
**Params**

	website_url: The website URL of the user. This will be used to find the user.
	
**Example**	

	GET https://api.easymarketing.de/site_verification/?website_url=test.de?access_token=123
	
**Response**

    {
      website_url: test.de,
      user_id: 123,
      meta_tag: <meta name=\"google-site-verification\" content=\"bF6M-w27WdtgyuCEXSC293tMA1JfjeN6DcEF20Up8_w\" />,
      html_content: google-site-verification: google646644efd7de5b2d.html,
      html_file_name: google646644efd7de5b2d.html,
    }

## Verify the website is properly configured

**Route**

  POST /verify_website/?website_url=xxx

**Params**

  website_url: The website URL of the user. This will be used to find the user.

**Example**

  POST https://api.easymarketing.de/confirm_website/?website_url=test.de?access_token=123

**Response**

If success:

    Status: 200

    {
      website_url: test.de,
      user_id: 123,
      errors: []
    }

If there is an error:

    Status: 400

    {
      website_url: test.de,
      user_id: 123,
      errors: ["Could not verify ownership of website", "API Check setup did not complete", "No categories found", "No products found"]
    }



## Facebook Like badge

The vendor can make use of the easymarketing facebook like badge. The route
returns a like button code that should be integrated on the vendor's checkout
success page. Triggering it will like the vendor's facebook page. The vendor
provides the details to his facebook fanpage in the easymarketing system.

**Route**

	GET /facebook_badge/:website_url

**Params**

	:website_url = the website url of the easymarketing user. Required.

**Example**

	GET /facebook_badge/easymarketing.de

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

	POST users/:website_url/products/:product_id/facebook_status

**Params**

	:product_id = the product id of a specific product that should be promoted.
	:website_url = the website url of the easymarketing user. Required.

**Example**

	POST users/foobar.com/products/1/facebook_status

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


## Changelog

Jan 27 2013 

* Split documents in two.



Oct 14th 2013

* Added more information concerning the versioning of the API and this document
* Added "Authenticating calls from Easymarketing towards your API Endpoints" section
* Added `url` attribute to Categories
* Added `google_category`, `discount_absolute` and `discount_percentage` to Products
	
