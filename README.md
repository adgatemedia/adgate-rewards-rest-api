# AdGate Rewards API v1 Documentation

## DEPRECATION NOTICE: This API is deprecated. Please use our [offer API](https://panel.adgatemedia.com/affiliate/api/offers).

If you wish to display our wall on your website, we highly recommend you use our [AdGate Rewards page in an iframe](https://github.com/adgatemedia/adgaterewards) to display it to your users. However, should you require a more customized experience for your users, we provide a complete HTTP REST API to build your own wall, which is documented below. It is the one we use in production, so anything that is done in the wall we provide can be accomplished on your own.

All successful API calls return a `200` response code and a top-level `status` attribute with a value of `success`. Returned data is provided in the `data` attribute. The base URL for all API endpoints is `https://wall.adgaterewards.com/apiv1/`. HTTP and HTTPS requests are accepted. POST and PUT urls accept a JSON string with the required data.

Errors return a `4xx` or `5xx` response code, a `status` attribute with the value of `error`, and an `error` object with additional information. For example:

```json
{
    "status": "error",
    "error": {
        "code": 404,
        "message": "Invalid VC Wall requested."
    }
}
```

## API Reference

#### Get User Info

GET: `/apiv1/vc/{vcCode}/users/{userid}`: Get information about the given user for the given offer wall. `{userid}` and `{vcCode}` can be alphanumeric strings. Currently only returns a list of devices the user owns. Devices can be one of "iphone", "ipad", or "android".

Example request: `/apiv1/vc/DfW/users/23432`

Sample response:

```json
{
   "status":"success",
   "data":{
      "devices":[
         "iphone",
         "android"
      ]
   }
}
```

#### Save User Info

POST: `/apiv1/vc/{vcCode}/users/{userid}`: Save information for a given user. Currently only user devices supported.

Request attributes:

 * **devices**: array that accept 0 or more devices. "iphone", "android", and "ipad" are supported.

Example request:

```json
{
   "devices":[
      "iphone",
      "android"
   ]
}
```

Example response:

```json
{
   "status":"success"
}
```

#### Fetch All Offers

GET: `/apiv1/vc/{vcCode}/users/{userid}/offers`: Fetch all offers as an array for the provided VC wall and user, taking into account the user's devices. Currently uses the requestor's user agent to determine the users' device. You may pass a `country_code` GET parameter containing a two-character country code OR an `ip` GET parameter of the user's IP that we will use to determine geolocation. If none of these two parameters are provided, we will use the API request's IP to determine geo location to show relevant offers. Also accepts 4 subid GET variables, s2 - s5, e.g. `s2=string`. The returned offers will have these subids appended to their URLs.

You may pass a two-letter language code to retrieve the offers in their native language (if available), such as `de`. The default is `en` for English.

You may fetch offers for mobile devices by passing a `device` GET param with a value of `android`, `iphone`, or `ipad`. If your users are coming from a native app, pass a `android_id` or `ios_id` GET variable with the corresponding unique device ID provided by the app (`ANDROID_ID` for Android, `IDFA` for iOS). This will return offers that require these identifiers to be preset. You may also fetch offers matching a user's user-agent string by passing a `ua` GET variable (this supercedes the `device` param.)

Returned attributes for each offer:

* **id** - the offer id
* **anchor** - the title text
* **requirements** - requirements text
* **description** - description text
* **web2mobile** - whether or not the user should complete this offer on their mobile device. This value can only be true for desktop user agents in order to display a web2mobile link. If value is true, the click_url below must be turned into a shortlink to be opened on the user's device.
* **click_url** - offer URL
* **icon_url** - icon URL
* **confirmation_time** - how long it takes the offer to confirm. string.
* **categories** - an array of string categories. Currently only `apps`, `free`, or both.
* **points** - the amount of points the offer will earn the user.
* **rank** - integer starting from 1 used for sorting popular offers.
* **date** - unix timestamp of offer creation. Used for sorting by "New".


```json
{
    "status": "success",
    "data": [
        {
            "id": 17328,
            "anchor": "EarningStation",
            "requirements": "1.  Register an account.  2.  Confirm your email  3. Earn 500 StationDollars ($5) in your account",
            "description": "EarningStation provides users with the easiest way to earn gift cards online. With these high-paying survey offers, discount shopping offers, and other fun ways to earn- users choose how they earn!",
            "web2mobile": false,
            "click_url": "http://agm.mobi/vc/nQ/users/USERIDHERE/offers/17328?",
            "icon_url": "https://afd78622d2e7a4badf1e-9ee71c0ecf9194b9daa6e92728b3332c.ssl.cf5.rackcdn.com/17328/hb6CG7Pk.png",
            "confirmation_time": "Confirms Instantly",
            "categories": [],
            "points": 144,
            "rank": 1,
            "date": 1433775316
        },
        {
            "id": 17269,
            "anchor": "Home Insurance Spotter",
            "requirements": "Fill out the survey.",
            "description": "Let us spot the best home insurance rates for you with a short survey.",
            "web2mobile": false,
            "click_url": "http://agm.mobi/vc/nQ/users/USERIDHERE/offers/17269?",
            "icon_url": "https://afd78622d2e7a4badf1e-9ee71c0ecf9194b9daa6e92728b3332c.ssl.cf5.rackcdn.com/17269/gbH3OJNw.png",
            "confirmation_time": "Confirms Instantly",
            "categories": [
                "Free"
            ],
            "points": 51,
            "rank": 2,
            "date": 1433354348
        },
        ...
    ]
}
```

#### Hide Offer

PUT: `/apiv1/vc/{vcCode}/users/{userid}/offers/{offerid}/hide`: Hides an offer from the offer list due to a provided reason. Next time offers are fetched the hidden offers will be excluded. Accepts a `reason` attribute as a string. Defaults to use are `not_interested`, `offensive`, `repetitive`, or `misleading`. Custom strings allowed.

Request:

```json
{
    "reason": "not_interested"
}
```

Response:

```json
{
    "status": "success"
}
```

#### Get Shortcode

GET: `/apiv1/vc/{vcCode}/users/{userid}/web2mobile`: Get the shortcode for a click_url. The shortcode should be appended to the shortlink domain. Requres a `url` GET variable with the click URL.

Example request: `/apiv1/vc/DfW/users/23432/web2mobile?url=http%3A%2F%2Fagm.mobi%2Fvc%2FnQ%2Fusers%2FUSERIDHERE%2Foffers%2F17269%3F`

Response:

```json
{
    "status": "success",
    "data": {
        "shortcode": "g"
    }
}
```

#### Send Shortlink To Email or Phone Number

POST: `/apiv1/vc/{vcCode}/users/{userid}/offers/{offerid}/send_shortlink`: Sends a shortlink via email or via SMS (US only). Accepts three attributes via JSON:

* **shortcode** - shortcode (not link) to send.
* **to** - valid email or 10-digit US phone number
* **devices** - an array of devices this link can be opened on. Can be one or more of `android`, `ipad`, or `iphone`.

Example request:

```json
{
    "to": "example@email.com",
    "devices": [
        "iphone",
        "android"
    ],
    "shortcode": "g"
}
```

Response:

```json
{
    "status": "success"
}
```

Response containing error message to display to user:

```json
{
    "status": "error",
    "error": {
        "code": 400,
        "message": "There was a problem sending your shortlink. Please double check your input and try again."
    }
}
```

#### Get User History

GET: `/apiv1/vc/{vcCode}/users/{userid}/history`: Get complete user history of offer activity.

Returned attributes:

* **id** - id of this history item
* **anchor** - title text of this offer
* **status** - one of `viewed` (offer was clicked but not completed), `completed` (offer has been completed and paid out), or `cancelled` (was completed in the past but due to various reasons was charged back).
* **points** - the amount of points rewarded for this offer. This item is positive for all `completed` items, and 0 for cancelled and viewed items.
* **latest_date** - unix timestamp of the last action for this offer (sort by this)
* **created_date** - unix timestamp when the offer was viewed for the first time (e.g. used for not showing contact link for the first hour after the initial view)
* **contacted** - boolean indicating whether the user has already contacted us regarding this offer/point. If already contacted, don't show the contact link.

Sample response:

```json
{
    "status": "success",
    "data": [
        {
            "id": 7,
            "anchor": "Atkins Diet Coupon",
            "status": "viewed",
            "points": 0,
            "latest_date": 1432135046,
            "created_date": 1432134221,
            "contacted": true
        },
        {
            "id": 6,
            "anchor": "Videostripe",
            "status": "viewed",
            "points": 0,
            "latest_date": 1432134545,
            "created_date": 1432131231,
            "contacted": false
        },
        {
            "id": 1,
            "anchor": "Smilebox",
            "status": "viewed",
            "points": 0,
            "latest_date": 1432053140,
            "created_date": 1432131131,
            "contacted": false
        }
    ]
}
```
