# EASYMARKETING API

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

#### Authenticating calls from Easymarketing towards your API Endpoints

The authentication of these API calls is handled via URL Params.

The `shop_token` will be appended to each request URL so that your site can authenticate our API calls. This token can be set through the Easymarketing Dashboard, just like all the API endpoints described in the next sections. If you do not set a `shop_token` we will assume no authentication is required and all calls can be made without sending a `shop_token`.

An example use of this is the following request:

**URL Parameters**:

	curl 'http://example.com/easymarketing_api/categories?parent_id=1;shop_token=1234567890abcdefghi' -I

## Extract Products/Categories From Remote Shop

EASYMARKETING can read a vendor's product/category data via a JSON Interface.
The webservice is called when the easymarketing user selects to import data
via an interface. The data is then used to generate ads in the client's
adwords account. This interface is to be developed by the easymarketing
partners.

The process is usually the following:

* The user creates an EASYMARKETING account.
* The user reads about a plugin developed for his shopsystem.
* The user then installs this plugin per our requirements.
* User proceeds and connects EASYMARKETING with his Shop via the plugin.

For this the user has to enter several API endpoints in his easymarketing
account. The following routes are mandatory and need to be provided.

API Endpoint for categories. Returns the shop's categories.

	http://example.com/easymarketing_api/categories

API Endpoint for products. Returns the shop's products.

    http://example.com/easymarketing_api/products

API Endpoint for best selling products. Returns a list of best selling products.

    http://example.com/easymarketing_api/best_products

API Endpoint for new products. Returns a list of new products.

    http://example.com/easymarketing_api/new_products

API Endpoint for a single product.

	http://example.com/easymarketing_api/product_by_id

Please refer below on how the those API endpoints are accessed by
easymarketing. The sample url will be replaced with the url the user entered
in his easymarketing account. What remains the same are the query string
parameters appeneded to the url.

### API Endpoint for Categories

**Route**

	GET http://example.com/easymarketing_api/categories

**Params**

	parent_id (Integer | String)

**Example**

	curl http://example.com/easymarketing_api/categories?parent_id=1

**Response**

	  {
		"id": 1,
		"name": "Category One",
		"url": "http://example.com/categories/1",
		"children": [100, 101]
	  }


The response always includes the data identified through the parent_id. If
there are no children for a category, children must be empty. This way
EASYMARKETING can recursively fetch your category tree. The extraction starts with the root category id provided along with the API endpoints in the EASYMARKETING dashboard.

The `url` attribute is the URL of the Category's page that customers can visit.

The `children` array must contain the ids of the category's children. The type of these ids can be `Integer` or `String`. These ids will be used to recursively fetch all categories.


### API Endpoint for Products.

The route turns exactly or less than products that specified through the limit
parameter. Products are returned in the range [offset..offset + limit).
Example:

	Request: offset: 0, limit: 10
	Response: 10 or less products in the range [0..9]

	Request offset: 10, limit 10
	Response: 10 or less products in the range [10..19]

The order of the items in the response does not matter, you can freely choose.
No duplicate products may be returned.

If less than limit products are returned, we assume that the limit is reached.
Products can also be empty.

**Route**

	GET http://example.com/easymarketing_api/products

**Params**

    offset (Integer), limit (Integer)

**Example**

    http://example.com/api/products?offset=0&limit=10

**Response**

	{
	  "offset": 0,
	    "products": [
	    {
	      "id": 0,
	      "name": "Levis 501",
	      "categories": [
	        0,
	      	1,
	      	2,
	      	3
	      ],
	      "condition": "new",
	      "availability": "in stock",
	      "price": 74.50,
	      "url": "http://example.com/products/1",
	      "image_url": "http://example.com/product_images/1.jpeg",
	      "shipping": [
	      	  {
				"country": "DE",
				"service": "Standard",
				"price": 4.95
			  }
	      ],
	      "colors": [
	        "red",
	        "black",
	        "blue"
	      ],
	      "description": "Description of product",
	      "margin": 2,
	      "gtin": "123123123123",
	      "adult": false
	    }
	  ]
	}



#### Required Attributes

* The `id` of a product must be an `Integer` or `String`.

* The `name` is the name of that Product. Type `String`.

* The `categories` Array must be an `Array of Integer` or `Array of String`. These category ids must be out of the set of ids of the previously extracted categories. That way the product can be associated to it's actual categories.

* The `price` attribute is of the type `Float`. The price includes taxes. **Not including any discounts**.

* The `currency` attribute is of the type `String`. The international 3-letter code as defined by the ISO 4217 standard. Like "EUR" or "USD". Only one currency per product allowed.

* The `url` attribute is of the type `String`. This is the URL of this product in your shopsystem. Example: `http://example.com/products/1`


* The `description` attribute. Include only information relevant to the item. Describe its most relevant attributes, such as size, material, intended age range, special features, or other technical specs. Also include details about the item’s most visual attributes, such as shape, pattern, texture, and design, since we may use this text for finding your item. Type `String`.


#### Optional Attributes

* `google_category` is the Google Category from the [Google Product Taxonomy](https://www.google.com/basepages/producttype/taxonomy.en-US.txt). This attribute of the type `String`.

* The `colors` attribute is an `Array of String`. Each array element represents a color in which this product is available.

* The `margin` of this product. How much does the shop make on every sale?
  This is useful for us in order to better promote specific products. Type `Float`.

* `adult` The adult status assigned to your product listings through the ‘adult’ attribute affects where product listings can show. For example, "adult" or "non-family safe" product listings aren't allowed to be shown in certain countries or to a certain audience. Type `Boolean`

* For the `condition` attribute there are only three accepted `String` values:

	* 'new': The product is brand-new and has never been used. It's in its original packaging which has not been opened.

	* 'refurbished': The product has been restored to working order. This includes, for example, remanufactured printer cartridges.

	* 'used': If the two previous values don't apply. For example, if the product has been used before or the original packaging has been opened or is missing.

* For the `availability` attribute only has four accepted `String` values:

  * 'in stock': Include this value if you are certain that it will ship (or be in-transit to the customer) in 3 business days or less. For example, if you have the item available in your warehouse.
  * 'available for order': Include this value if it will take 4 or more business days to ship it to the customer. For example, if you don’t have it in your warehouse at the moment, but are sure that it will arrive in the next few days. For unreleased products, use the value 'preorder'
  * 'out of stock': You’re currently not accepting orders for this product. (Important tip: When your products are out of stock on your website, don't remove them from your data feed. Provide this value instead).
  * 'preorder': You are taking orders for this product, but it’s not yet been released.

* The `image_url` is of the type `String`.

* The `shipping` attribute is an `Array of Object` and the Objects have the following attributes:

	* `country` The country to which an item will be delivered (as an ISO 3166 country code). Example: "DE" for Germany. Type `String`.
	* `service` **OPTIONAL** The service class or delivery speed. For example "Standard". Type `String`.
	* `price` Fixed delivery price (including tax). For example 4.95. Type `Float`.

Example:

	{
		"country": "DE",
		"service": "Standard",
		"price": 4.95
	}


* `gtin` are the Global Trade Item Numbers (GTINs). GTINs include UPC, EAN (in Europe), JAN (in Japan), and ISBN; Example: "8808992787426". Type `String`.

* `discount_absolute` shows that there is an absolute discount on a product. For example a 2.00 Euro discount. Type `Float`.

* `discount_percentage` shows that there is a relative discount of x-percent of the original price. Type `Float`. For example `9.5` would equal a discount of 9.5% of the original product price.

### API Endpoint for a single product.

Returns a single product by its ID.

**Route**

    GET http://example.com/easymarketing_api/product_by_id

**Params**

    id: Integer | String

**Example**

    http://example.com/easymarketing_api/product_by_id?id=1

**Response**

    JSON representation of a product. Like the example above.


### API Endpoint for new products.

This returns the newest products since a given timestamp until today.
The product id's need to be returned. We will match this internally with our own database.

**Route**

    GET http://example.com/easymarketing_api/new_products

**Params**

    limit: Integer, newer_than: Integer(UNIX timestamp)

**Example**

    http://example.com/easymarketing_api/new_products?limit=1&newer_than=1380646110

**Response**

    {
      "time": 1380646110,
      "newer_than": 1,
      "products": [
        // Please refer to the API Endpoint for products for the exact JSON
        // that is returnd
      ]

    }

### API Endpoint for best selling products.

Returns an array of product ids, sales that were most sold since a given timeframe
until today. It should be ordered descending with most sales as first product.
We will internally match the product ids with our own database.

**Route**

    GET http://example.com/easymarketing_api/best_products


**Params**

    limit: Integer, most_sold_since: Integer(UNIX timestamp)

**Example**

    http://example.com/easymarketing_api/best_products?limit=2&most_sold_since=1380646110

**Response**

    {
      "limit": 2,
      "most_sold_since": 1,
      "products": [
        {
          "id": 1,
          "sales": 10
        },
        {
          "id": 2,
          "sales": 5
        }
      ]

    }

# EASYMARKETING Routes

## User performance data (iFrame)

Returns the user's data from his performance page. The performance page shows
information about clicks, best advertisements and much more. You can embed it
as an iframe.

**Route**

    GET /users/:website_url/performance

**Params**

    :website_url - the url of the vendor without http, required.

**Example**

    GET /users/foo.com/performance

**Response**

    <iframe src="..."></iframe>

## User performance data (JSON)

Returns the same data as the previous example, just in JSON format so you can
build your own UI.

**Route**

    GET /users/:website_url/performance.json

**Params**

    :website_url - the url of the vendor without http, required.

**Example**

    GET /users/foo.com/performance.json

**Response**

    {
      "clicks": 0,
      "cr": 0,
      "cpc": 0,
      "sales": 0,
      "impressions": 0,
      "daily_budget": 5,
      "keywords": 2375,
      "ads": 1170,
      "due_amount": 0,
      "revenue": 0,
      "time_saved": 320,
      "average_conversion": 0.032,
      "products_searched_per_month": 0,
      "time_frame_impressions": 0
    }

## User Analysis page

This returns the analysis page in an iFrame. The analysis page contains a
bunch of charts and data giving the user hints about how well his website
could perform in AdWords. It does not require an auth token.

**Route**

    GET /analaysis


**Example**

    GET /analysis

**Response**

    <iframe src="..."></iframe>

## Quick chart

The quick chart can be found on the easymarketing homepage. It gives the user
a quick gadget to play around with. Does not require an auth token.

**Route**

    GET /homepage_chart

**Example**

    GET /homepage_chart

**Response**

    <iframe src="..."></iframe>

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
* Oct 14th 2013
	* Added more information concerning the versioning of the API and this document
	* Added "Authenticating calls from Easymarketing towards your API Endpoints" section
	* Added `url` attribute to Categories
	* Added `google_category`, `discount_absolute` and `discount_percentage` to Products
	
