# Easymarketing API

## Conversion pixel

Integrate the conversion pixel on your checkout success page whenever a sale
has been generated.

All requests go to

    http://api.easymarketing.de/v1/

**Route**

    GET /conversion_tracker/:website_url_of_vendor

**Params**

    :website_url = url of the vendor without http, required

**Example**

    GET /conversion_tracker/google.de

**Response**

    Status: 200

    {
      "user_id": 1,
      "code": '<script type="text/javascript"> /* <![CDATA[ */ var google_conversion_id = 982579417; var google_conversion_language = "en"; var google_conversion_format = "3"; var google_conversion_color = "ffffff"; var google_conversion_label = "cdl_CJe02gcQ2fHD1AM"; var google_conversion_value = 0; var google_remarketing_only = false; /* ]]> */ </script> <script type="text/javascript" src="//www.googleadservices.com/pagead/conversion.js"> </script> <noscript> <div style="display:inline;"> <img height="1" width="1" style="border-style:none;" alt="" src="//www.googleadservices.com/pagead/conversion/982579417/?value=0&amp;label=cdl_CJe02gcQ2fHD1AM&amp;guid=ON&amp;script=0"/> </div> </noscript>'
    }


## Extract Products/Categories from remote Shopsystem

Easymarketing can read a vendor's product/category data via a JSON Interface.
The webservice is called when the easymarketing user selects to import data
via an interface.

We save our partners routes in our database:

    /easymarketing_api/categories
    /easymarketing_api/products

We will then send requests to:

    http://meinshop.de/easymarketing_api/categories
    http://meinshop.de/easymarketing_api/products

Depending on the user's shopsystem.

So once you are done developing the module, let us know the respective urls
for your shopsystem.

### Extracting the category tree

**Params:**

    parent_id (Integer | String)


**Example:**

    {
      "parent_id": 1
    }

**Response:**

  {
    "parent": {
      "id": 1,
      "name": "Category One"
    },
    "children": [
      {
        "id": 100,
        "name": "Category 100"
      },
      {
        "id": 101,
        "name": "Category 101"
      }
    ]
  }


The response always includes the data identified through the parent_id. When
parent_id = 0 is requested, the root categories of the shop are requested. If
there are no children for a category, children must be empty. This way
EASYMARKETING can recursively fetch your category tree.

### Extracting product data

**Params:**

        offset (Integer), limit (Integer)

**Example:**

    {
        "offset": 0,
        "limit": 10
    }

**Response:**

  {
    "offset": 0,
    "products": [
      {
        "id": 0,
        "name": "Name of product",
        "categories": [
          0,
          1,
          2,
          3
        ],
        "price": 24.50,
        "url": "http://example.com/products/1",
        "colors": [
          "green",
          "blue",
          "red"
        ],
        "description": "Description of product",
        "specification": "specifications of product",
        "dependency": "dependencies of product",
        "margin": 0.56
      }
    ]
  }

#### Required Attributes

* The `id` of a product must be an `Integer` or `String`.

* The `categories` Array must be an `Array of Integer` or `Array of String`. These category ids must be out of the set of ids of the previously extracted categories. That way the product can be associated to it's actual categories.

* For the `condition` attribute there are only three accepted `String` values:

* 'new': The product is brand-new and has never been used. It's in its original packaging which has not been opened.

* 'refurbished': The product has been restored to working order. This includes, for example, remanufactured printer cartridges.

* 'used': If the two previous values don't apply. For example, if the product has been used before or the original packaging has been opened or is missing.

* For the `availability` attribute only has four accepted values:

  * 'in stock': Include this value if you are certain that it will ship (or be in-transit to the customer) in 3 business days or less. For example, if you have the item available in your warehouse.
  * 'available for order': Include this value if it will take 4 or more business days to ship it to the customer. For example, if you don’t have it in your warehouse at the moment, but are sure that it will arrive in the next few days. For unreleased products, use the value 'preorder'
  * 'out of stock': You’re currently not accepting orders for this product. (Important tip: When your products are out of stock on your website, don't remove them from your data feed. Provide this value instead).
  * 'preorder': You are taking orders for this product, but it’s not yet been released.

* The `price` attribute is of the type `Integer` or `Float`. The price includes taxes.

* The `image_url` is of the type `String`.

* The `shipping` attribute is an `Array of Object` and the Objects have the following attributes:

  * `country` The country to which an item will be delivered (as an ISO 3166 country code). Example: "DE" for Germany. Type `String`.
  * `service` **OPTIONAL** The service class or delivery speed. For example "Standard". Type `String`.
  * `price` Fixed delivery price (including tax). For example 4.95. Type `Integer` or `Float`.

##### Additional Attributes for the non Apparel/Custom Good Category

* `gtin` are the Global Trade Item Numbers (GTINs). GTINs include UPC, EAN (in Europe), JAN (in Japan), and ISBN; Example: "8808992787426". Type `String`.
* `mpn` Manufacturer Part Number (MPN). Example: "M2262D-PC". Type `String`.
* `brand` The manufacturer's brand name. Example: "Sony". Type `String`.

##### Additional Attributes for the Apparel Category
* `shipping_weight` The shipping weight of this item. Example: "800 g". Type `String`
* `gender`. Type `String`. The accepted values are:
  * `Male`
  * `Female`
  * `Unisex`

* `age_group`. Type `String`. The accepted values are:
  * `Adult`
  * `Kids`

* Variants. For example if you have a shirt in 4 colors and 5 sizes:

    {
      "red": ["S", "M", "L", "XL", "XXL"],
      "blue": ["S", "M", "L", "XL", "XXL"]
    }


##### Additional Attributes for the Media Category


#### Optional Attributes

FOOBAR


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


“categories” are the category ids of the product. The first element of categories is the root/highest parent category. The second, the lower category etc. These category ids must match the ids of existing categories that have been extracted previously through the category tree extraction.

The following response data is optional:

    colors, description, specification, dependency, margin (the profit the vendor on sale of the product)

Please see the above example of data types that should be returned.


## Facebook API integration

The vendor can make use of the easymarketing facebook integration. The route
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

    {
      "user_id": 1,
      "website_url": "easymarketing.de",
      "code": '<iframe src="//www.facebook.com/plugins/like.php?href=https%3A%2F%2Fwww.facebook.com%2FGoogle&amp;width=450&amp;height=46&amp;colorscheme=light&amp;layout=button_count&amp;action=like&amp;show_faces=true&amp;send=true&amp;appId=270892269593470" scrolling="no" frameborder="0" style="border:none; overflow:hidden; width:450px; height:46px;" allowTransparency="true"></iframe>'
    }

Note this retuns the default facebook like button. You can optionally adjust
width/size of the like button if required on your own by modifying the html.

For a full documentation on the like button refer to:

    https://developers.facebook.com/docs/reference/plugins/like/