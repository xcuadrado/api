# Scope of work EASYMARKETING Integration

## General

This document is versioned using [Semantic Versioning](http://semver.org/). That means:

Given a version number MAJOR.MINOR.PATCH, increment the:

	MAJOR version when you make incompatible API changes,
	MINOR version when you add functionality in a backwards-compatible manner, and
	PATCH version when you make backwards-compatible bug fixes.

The version of this document is **1.0.0**.

This document describes the EASYMARKETING JSON API major version **1**.

There is a changelog of what has changed in between versions at the very bottom of this document.

---------------------------------------

## Your TODO

### Build a webservice

1. Setup a sample shopsystem, where you will deploy the the code. This shopsystem needs to stay online/live for as long as you work together with EASYMARKETING. Please note we can not do this, as we have no expertise with your shopsystem. We will test the API against this shopsystem.
2. Send us instructions how we can log into the shopsystem. Provide us an admin user and admin password to do so.
3. Integrate the specifications from this document.
4. Test your setup via our developer account: User `dev@easymarketing.de`, Password: `dev@easymarketing.de`
5. Make sure the API setup returns no errors.
6. Wait for us to confirm the integrity of the webservice.
7. We will then proceed and start a live implementation with one of our customers. Please suggest us a possible prospect of your choice.

### Build a UI in your backend

1. Build a UI in the shop backend, where the user can view his `shop_token`. This will be used for authorizing our requests to the webservice you will implement.
2. Build a UI where the user can view his `endpoints`. The user needs to copy those routes and enter them in his EASYMARKETING account.
3. Write detailed instructions for the user on your UIs. Make sure the UI is intuitive and easy to understand.
4. Provide a UI for entering an `access_token`. The user needs to copy+paste this from his EASYMARKETING account. This is used to access EASYMARKETING webservices like returning daily user statistics, or returning a conversion tracker to measure sales.

### Publish your code

1. Please create a public repository on Github where you host the code. Make one of our developers admin.
2. We will add a link on our website that will download the `master` branch of the repository as a .zip file.
3. Publish all the code in your repository on Github.
4. Write detailed instructions in the `README.md` file. This is shown when opening the repository.



### Integrate the EASYMARKETING API

On top of creating a webservice we access, you also need to integrate some of our webservices.

1. Integrate the [Conversion Tracker](offered_webservices)
2. Integrate additional services if specified (must be added to scope of work)


---------------------------------------


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

Remark: We recommend the usage of https only if you have a valid certificate
for your webserver installed.

API Endpoint for categories. Returns the shop's categories.

	https://example.com/easymarketing_api/categories

API Endpoint for products. Returns the shop's products.

    https://example.com/easymarketing_api/products

API Endpoint for best selling products. Returns a list of best selling products.

    https://example.com/easymarketing_api/best_products

API Endpoint for new products. Returns a list of new products.

    https://example.com/easymarketing_api/new_products

API Endpoint for a single product.

	https://example.com/easymarketing_api/product_by_id

API Endpoint for a API version.

	https://example.com/easymarketing_api/api_version

Please refer below on how the those API endpoints are accessed by
easymarketing. The sample url will be replaced with the url the user entered
in his easymarketing account. What remains the same are the query string
parameters appeneded to the url.

### Authenticating calls from Easymarketing towards your API Endpoints

The authentication of these API calls is handled via URL Params.

The `shop_token` will be appended to each request URL so that your site can authenticate our API calls. This token can be set through the Easymarketing Dashboard, just like all the API endpoints described in the next sections. Thus you need to build a UI in your shop's backend that shows the `shop_token`. If you do not set a `shop_token` we will assume no authentication is required and all calls can be made without sending a `shop_token`. This is a possible security risk.

An example use of this is the following request:

**URL Parameters**:

	curl 'https://example.com/easymarketing_api/categories?id=1&shop_token=1234567890abcdefghi' -I
	

### API Endpoint for Categories

**Route**

	GET https://example.com/easymarketing_api/categories

**Params**

	id (Integer | String)

**Example**

	curl https://example.com/easymarketing_api/categories?id=1&shop_token=1234567890abcdefghi

**Response**

	  {
		"id": 1,
		"name": "Category One",
		"url": "http://example.com/categories/1",
		"children": [100, 101]
	  }


The response always includes the data identified through the id. If
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

	GET https://example.com/easymarketing_api/products

**Params**

    offset (Integer), limit (Integer)

**Example**

    https://example.com/api/products?offset=0&limit=10&shop_token=1234567890abcdefghi

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

* The `price` attribute is of the type `Float`. The price includes taxes. **Not including any discounts**. The price may not be ´0´. Products with price `0` are discarded by our client.

* The `currency` attribute is of the type `String`. The international 3-letter code as defined by the ISO 4217 standard. Like "EUR" or "USD". Only one currency per product allowed.

* The `url` attribute is of the type `String`. This is the URL of this product in your shopsystem. Example: `http://example.com/products/1`


* The `description` attribute. Include only information relevant to the item. Describe its most relevant attributes, such as size, material, intended age range, special features, or other technical specs. Also include details about the item’s most visual attributes, such as shape, pattern, texture, and design, since we may use this text for finding your item. Type `String`.


* `gtin` are the Global Trade Item Numbers (GTINs). GTINs include UPC, EAN (in Europe), JAN (in Japan), and ISBN; Example: "8808992787426". Type `String`. If you do not have any of the values in your database, provide `null` as value. Output is required.



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

* `discount_absolute` shows that there is an absolute discount on a product. For example a 2.00 Euro discount. Type `Float`.

* `discount_percentage` shows that there is a relative discount of x-percent of the original price. Type `Float`. For example `9.5` would equal a discount of 9.5% of the original product price.

### API Endpoint for a single product.

Returns a single product by its ID.

**Route**

    GET https://example.com/easymarketing_api/product_by_id

**Params**

    id: Integer | String

**Example**

    https://example.com/easymarketing_api/product_by_id?id=1&shop_token=1234567890abcdefghi

**Response**

    JSON representation of a product. Like the example above.


### API Endpoint for new products.

This returns the newest products since a given timestamp until today.
The product id's need to be returned. We will match this internally with our own database.

**Route**

    GET https://example.com/easymarketing_api/new_products

**Params**

    limit: Integer, newer_than: Integer(UNIX timestamp)

**Example**

    https://example.com/easymarketing_api/new_products?limit=1&newer_than=1380646110&shop_token=1234567890abcdefghi

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

    GET https://example.com/easymarketing_api/best_products


**Params**

    limit: Integer, most_sold_since: Integer(UNIX timestamp)

**Example**

    https://example.com/easymarketing_api/best_products?limit=2&most_sold_since=1380646110&shop_token=1234567890abcdefghi

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

### API Endpoint for api version.

This returns the api version used.

**Route**

    GET https://example.com/easymarketing_api/api_version

**Example**

    https://example.com/easymarketing_api/api_version?shop_token=1234567890abcdefghi

**Response**

    {
      "api_version": "1.0.2"
    }

    
### API Endpoint for returning the shopsystem.

Return the shopsystem the user has.

**Route**

    GET https://example.com/easymarketing_api/shopsystem_info

**Example**

    https://example.com/easymarketing_api/shopsystem_info?shop_token=1234567890abcdefghi

**Response**

    {
      "shopsystem": "jtl",
      "shopsystem_human": "JTL Software",
      "shopsystem_version": "1.2"
    }



## Changelog

* Jan 27th 2014
	* Split documents in two.
	* Add clearer TODOs.
	* Make `gtin` required output for products. It is safe to ignore this for old APIs.
	* Add a new route for returning the api version and the shopsystem.
	 
* Oct 14th 2013
	* Added more information concerning the versioning of the API and this document
	* Added "Authenticating calls from Easymarketing towards your API Endpoints" section
	* Added `url` attribute to Categories
	* Added `google_category`, `discount_absolute` and `discount_percentage` to Products
	
