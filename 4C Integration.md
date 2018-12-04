# MX Buyer Integration Guide

###### This document will give a brief overview of the MX media platform, and explain how to interact with its API.

### Platform Overview

#### Domain Concepts

Within the MX platform we think in terms of `Organizations`, `Users`, `Teams`, `Roles`, `Markets`, `Orders`, `Assets`, `Attributes`, and `Matches`.

* An `Organization` represents a single buyer or seller, and is analogous to a company account. There are many `Users` within an `Organization`, and any action a user takes looks, from the outside, as if it was done by the organization. Users are assigned to `Teams`, and given `Roles` within those teams which constrain which actions they are allowed to take.

* A `Market` is an environment in which transactions happen. There is an `Open Market` and user defined `Private Markets`. Initially, all transactions will happen in the latter. `Orders` in private markets can only be seen by market members, which are `Organizations`, and defined by the market owner.

* An `Order` is how we represent supply or demand entered into a `Market`. An order represents the intention of an `Organization` to buy/sell some amount of a certain `Asset` in a `Market`, at some price.

* A `Match` is the outcome of a market transaction; it represents an intersection of supply and demand, on terms meeting the requirements of both sides. Matches are analogous to non-preemptible orders in traditional media buys.

* An `Asset` is what we call media that is bought and sold. It represents the right to show an ad, in a particular place, to some audience, at a certain time. It is composed of `Attributes` and additional information, like the date and time.

* An `Attribute` represents some piece of information about the asset. This can be any sort of information, for any sort of asset. For example, to use a fruit market analogy, if its green, if its an apple, if its large or small. Splitting up information about assets in this way allows us to search through them in useful ways. We talk about attributes in the format `Type: Attribute Value`, ie, `Network: WeMakeTv`.

* Attributes in turn have `Types`, which describe what sort of information attributes of that type convey about assets. In the same analogy, the types would be `Color`, `Fruit`, `Size`. Thinking about types allows us to measure the specificity (defined later) of an asset.

### Market

#### Rules

* The market operates on **New York time (EDT, GMT-5)**, and is open from **9:30AM-4:30PM**. Orders may be submitted at any time, but will **only be processed** when the market is open.

* Orders are processed in the order they are submitted, even if submitted while the market was closed. If an order is edited, it goes to the end of the queue.

* Supply and demand are matched on, in order of priority, **asset, price, and submission order**.

* If identical orders are submitted into the market and compete for liquidity, the one **submitted first** will have a chance to match before the other one.

* Sellers may only enter sell orders for assets they actually have in their catalog. Buyers may enter any valid order, even if there is nothing available in the market to match it.

* Orders entered into the market represent an intention to buy/sell, and once an order is matched it is considered **non-cancellable**. Orders should not be submitted unless you are willing to commit to the outcome.

#### Order Semantics

* Asset matching is based on `Specificity`, which refers to the precision with which an order describes the asset. For example:
    * User A enters a `Sell Order` for `Fruit: Apple | Color: Green | Size: Small`
    * User B enters a `Buy Order` for `Color: Green | Size: Small`
    * Order A is `more specific` than Order B, as it describes an asset more precisely (provides more information about it). Since the demand here can be met with supply for ANY small green fruit, the fact that Order A is for an apple is irrelevant. If there were another sell order for a pear, Order B could also match against it.

* Orders can be partially filled, ie an order for 10 can match against an order for 5, and the remainder will stay in the market. The order can be matched against by orders that enter the market in the future, and maintain their priority in terms of submission time. All-or-none orders (orders that do not allow partial fills) currently are not supported, but will be in the future.

* Orders may be deactivated (canceled) after submission, but only if they have not matched. Orders that have partially matched may have their unmatched part deactivated, but not the matched part.

* Orders can be matched for assets running, at the earliest, tomorrow. Today's assets cannot be matched against; processing time is required to traffic matches. Sellers can configure a longer trafficking window if one day is not adequate. Once an order can no longer be matched against, it is expired and automatically remoed from the market.


### API Structure

All MX platform functionality is accessible through our API, which is available to all users. A Javascript SDK can be made available upon request.

The API is located at `https://<environment>.massexchange.com/api`. For testing purposes, use `sandbox`, and for the production implementation, `app`.

To gain access, you must generate an API key, which is done on the Organization Profile. Store these keys securely, as they are equivalent to user credentials.

#### Design

The MX API is **RESTful** in design, aiming to adhere as closely as possible to **REST** principles. Functionality is organized by subdomain, and endpoints represent resources/collections wherever possible.

#### Security

* All API requests must be made over **HTTPS**.
* Each request must be authenticated with an `X-Auth-Token` header.

#### Data Format

The MX API accepts and returns `JSON` data, using a format called `JSOG` (`Javascript Object Graph`) to encode entity graphs with cyclic references. There is a [Javascript library](https://github.com/jsog/jsog) available for encoding/decoding this format, as well as Python, Ruby, and Java.

##### Types

* `Dates` are formatted as `ISO 8601` with timezone UTC.
* `Ids` are long-valued integers; any field called `id` will always be of this type.
* `Sets` and `Lists` are both sent as arrays, distinguished only by semantics.
* `Pages` are a wrapper object for paged endpoints, with these fields:
    * **content**
        * the current page of results
    * **number**
        * the current page number
    * **totalPages**
        * the total number of pages
    * **first/last**
        * `Booleans` indicating whether the current page is a terminal
* `Currency` is a `Number` with 2 decimal places of precision

#### Entities

All entities have `Ids`; they are unique, but not globally. There will never be duplicate ids within a single entity, but there may be across entities.

Not all fields are documented here; the ones left off are safe to ignore.

##### `Market`

A distinct liquidity pool, isolated from other orders

* owner: `Organization`
* members: `Set<Organization>`
* name: `String`

##### `AvailabilityGroup`

Represents a group of avails (available supply) with the same asset.

* asset: `Set<Attribute>`
* avails: `List<Availablility>`
* marketId: `Id`

##### `Availablility`

* start: `Date`
* end: `Date`
* qty: `Number`
* price: `Currency`

##### `Attribute`

* type: `AttributeType`
* value: `String`

##### `AttributeType`

* name: `String`

##### `Campaign`

* name: `String`
* advertiser: `String`
* brand: `String`
* flightStartDate: `Date`
* flightEndDate: `Date`

##### `BuyOrder`

* qty: `Number`
    * Quantity of units desired
* flightStartDate: `Date`
* flightEndDate: `Date`
* marketId: `Id`
    * Market the `BuyOrder` will eventually be submitted into
* price: `Currency`
* asset: `Set<Attribute>`
* status: `OrderStatus`
    * this field is read-only

##### `Enum<OrderStatus>`

* Unsubmitted
    * This order has not been submitted into the market yet
* Pending
    * This order is pending submission and is not in the market yet
* Submitted
    * This order has been submitted and is active in the market
* Filled
    * This order has been fully filled and is now inactive
* Expired
    * This order's flight date has passed and it has been automatically deactivated
* Deactivated
    * This order has been deactivated and is no longer in the market


##### `OrderQuery`

* start: `Date`
* end: `Date`
* attributes: `Set<Id>`
* role: `Enum<MarketParticipantType>`
    * set this to `ADVERTISER`
* pageIndex: `Number`

##### `MatchQuery`

* start: `Date`
* end: `Date`
* byMatchDate: `Boolean`
    Indicates the query mode, ie if the dates refer to
    * match date
    * asset date
* pageIndex: `Number`


#### Endpoints

##### Market Availability Query

Query avails for a given `Asset` in a `Market`

`GET /market/sells`

Query params: `OrderQuery`

Returns: `Page<AvailablilityGroup>`

##### Market Query

Query all markets your organization is in

`GET /market`

Returns: `Set<Market>`

##### Attribute Query

Query all known attributes

`GET /attr`

Returns: `Attribute`

##### Campaign Creation

Create a new `Campaign`

`POST /campaign`

Request body: `Campaign`

Returns: `Campaign`

##### Buy Order Creation

Add a new `BuyOrder` to a `Campaign`

`POST /campaign/{campId}/orders`

Path variables:
* campId: `Id` - `Campaign` being added to

Request body: `BuyOrder`
Returns: `BuyOrder`

##### Buy Order Submission

Submit a `BuyOrder` into a `Market`.

`POST /market/{marketId}/orders/group/{groupId}`

Path variables:
* marketId: `Id` - `Market` being submitted into
* groupId: `Id` - `BuyOrder` being submitted

Returns: `BuyOrder`

##### Buy Order Query

Get the current version of your `BuyOrder`

`GET /orderGroup/{groupId}`

Path variables:
* groupId: `Id` - `BuyOrder` being queried

Returns: `BuyOrder`

##### Buy Order Deactivation

Deactivate your `BuyOrder` to remove it from the market

`PUT /orderGroup/{groupId}/status`

Path variables:
* groupId: `Id` - `BuyOrder` being queried

Request body: `OrderStatus`
* Must be the string `Deactivated`

##### Match Query

Query your `Match` history

`GET /match`

Query params: `MatchQuery`

Returns: `Page<Match>`

### Workflow

1. Seller submits their supply into the market.
2. Buyer queries the market to see availible supply.
3. Buyer submits their demand into the market.
4. If there is is no match, the orders rest until canceled or expired.
5. If there is a match, the inventory is transferred and taken off the market.
6. After a match, buyer can see results on the Log.

### Examples

Provided are several example interactions with the MX API. It is suggested to read them in order, as they will be progressively less detailed, to not contain redundant information.

#### Querying the market

You will often want to query the market to see the avails (available supply). You can filter the query by market and by asset, or get all avails across all markets. This example assumes the latter.

The market query endpoint is `/market/sells` and it accepts `GET` requests. It takes an `OrderQuery` as its parameter, so we will construct one:
```js
{
    start: "2018-11-01T00:00:00.000+0000",
    end: "2018-11-24T00:00:00.000+0000",
    role: "ADVERTISER",
    pageIndex: 0
}
```

Now, since this is a `GET` request, we can't pass a body, so we will serialize this as a querystring:
```
start=2018-11-01T00:00:00.000+0000&
end=2018-11-24T00:00:00.000+0000&
role=ADVERTISER&
pageIndex=0
```
and then URL-encode it and concatenate it to the end of the endpoint path:
```
/market/sells?start=2018-11-01T00%3A00%3A00.000%2B0000&end=2018-11-24T00%3A00%3A00.000%2B0000&role=ADVERTISER&pageIndex=0
```
Now we take the path and concatenate that to the API URL, which is
```
https://sandbox.massexchange.com/api
```
and then make a `GET` request with your `HTTP` client to the full URL:
```
https://sandbox.massexchange.com/api/market/sells?start=2018-11-01T00%3A00%3A00.000%2B0000&end=2018-11-24T00%3A00%3A00.000%2B0000&role=ADVERTISER&pageIndex=0
```

If there are avails (lets assume there are, for this example), we will get back a `Page<AvailabilityGroup>`:
```json
{
    "content": [
        {
            "asset": [
                {
                    "id": 123,
                    "type": {
                        "id": 456,
                        "name": "Network"
                    },
                    "value": "WeMakeTv"
                }
            ],
            "avails": [
                {
                    "start": "2018-11-01T00:00:00.000+0000",
                    "end": "2018-11-03T00:00:00.000+0000",
                    "qty": 10,
                    "price": 1000.00
                },
                ...
            ],
            "marketId": 1
        },
        ...
    ],
    "number": 0,
    "totalPages": 3,
    "first": true,
    "last": false
}
```
Notice how the results are wrapped with page metadata, indicating that this is 1/3 pages. If you want to get the rest of the results, you will need to make 2 additional requests, incrementing the `pageIndex` each time.

In the `content` field, you can see an `AvailabilityGroup`, with an embedded `Attribute`, and an `Availability`, as examples. There can, and often will be, multiple objects in those fields.

This sample data indicates that for the date range we queried, there are 10 units available of `Network: WeMakeTv` inventory, from `11/01/2018`-`11/03/2018` at `$1000.00` each.

#### Creating a `Campaign`

Buy orders are organized by `Campaign`, so before you can submit any, you must create a grouping campaign first.

The campaign creation endpoint is `/campaign` and it accepts `POST` requests. It takes a `Campaign` object in the request body, so we will construct one:
```js
{
    name: "Q4 Buy",
    advertiser: "Sample Advertiser",
    brand: "Sample Brand",
    flightStartDate: "2018-11-01T00:00:00.000+0000",
    flightEndDate: "2018-12-01T00:00:00.000+0000"
}
```

Campaign names must be unique, advertiser/brand must be supplied. All string fields must be under 65 characters. The start date must be after today, and the end date must be after the start date.

Once we have constructed a valid `Campaign`, we make the request:
```http
POST /campaign

{"name":"Q4 Buy","advertiser":"Sample Advertiser","brand":"Sample Brand","flightStartDate":"2018-11-01T00:00:00.000+0000","flightEndDate":"2018-12-01T00:00:00.000+0000"}
```
and if there are no errors, the persisted version will be returned, containing the `Id`.

>Note: the request body was sent over minified; this is not mandatory, but recommended.

##### Creating a `BuyOrder`

Once you have a `Campaign`, you then create a `BuyOrder` within it. There can be any amount of orders in a campaign.

The buy order creation endpoint is `/campaign/{campId}/orders` and it accepts `POST` requests. It takes a `BuyOrder` object in the request body, so we will construct one:
```
{
    qty: 10,
    flightStartDate: "2018-11-01T00:00:00.000+0000",
    flightEndDate: "2018-11-03T00:00:00.000+0000",
    marketId: ???,
    price: 800,
    asset: ???
}
```
Seems like we're missing two bits of information to be able to fully construct this BuyOrder: the `marketId` and the `asset`.

To get the `marketId`, assuming we don't already know which market we're targeting, we need to query the markets we're in: `GET /market`, and when we get back a list of `Markets`, choose the `Id` of the appropriate one.

To construct the asset, we need to get the list of `Attributes` from somewhere; where specifically depends on how you're going about submitting demand. One option is to query all known attributes, and pick some from that list; another option is to query market availability, and use the attributes you find there. We're going to do the second one here.

Referring back to the "Querying the market" example, lets look at the asset in the result we got back:
```
{
    "asset": [
        {
            "id": 123,
            "type": {
                "id": 456,
                "name": "Network"
            },
            "value": "WeMakeTv"
        }
    ],
    "avails": [
        {
            "start": "2018-11-01T00:00:00.000+0000",
            "end": "2018-11-03T00:00:00.000+0000",
            "qty": 10,
            "price": 1000.00
        },
        ...
    ],
    "marketId": 1
}
```
Typically there would be more attributes in the asset, but lets just use this one as is, and add that to the `BuyOrder` we're constructing, Lets also use the `marketId` present here:
```
{
    qty: 10,
    flightStartDate: "2018-11-01T00:00:00.000+0000",
    flightEndDate: "2018-11-03T00:00:00.000+0000",
    marketId: 1,
    price: 800,
    asset: [
        {
            "id": 123,
            "type": {
                "id": 456,
                "name": "Network"
            },
            "value": "WeMakeTv"
        }
    ]
}
```
You only actually need to provide the `Id` field of the attributes, and in the future will be able to just provide a list of ids.

Now that we have a full `BuyOrder`, we're almost ready to make the request; now we just need to construct the full endpoint path. Replace the `campId` parameter in the endpoint with the `Id` of the `Campaign` we created in the previous step, and make the request:
```
POST /campaign/123/orders

{"qty":10,"flightStartDate":"2018-11-01T00:00:00.000+0000","flightEndDate":"2018-11-03T00:00:00.000+0000","marketId":1,"price":800,"asset":[{"id":123,"type":{"id":456,"name":"Network"},"value":"WeMakeTv"}]}
```
and then, if there are no errors, the persisted version of the `BuyOrder` will be returned, with the `Id`.

##### Submitting a `BuyOrder`

Once you've created one or more `BuyOrders` and validated that they accurately represent your intentions, you then want to submit them into the market.

The `BuyOrder` submission endpoint is `/market/{marketId}/group/{groupId}` and it accepts `POST` requests with no request body.

Take the `marketId` and `groupId` from the previous step, substitute, and then make the request:
```
POST /market/1/orders/group/456
```
and if there are no errors, the `BuyOrder` will be returned, with a `status` of `Submitted`.

##### Deactivating a `BuyOrder`

Sometimes, you'll want to deactivate an order you've submitted, either if you made a mistake, changed your mind, or have filled your demand elsewhere.

The `BuyOrder` deactivation endpoint is `/orderGroup/{groupId}/status` ad it accepts `PUT` requests with an `OrderStatus` in the request body. The only currently accepted status is `Deactivated`.

Take the `groupId` from the previous step, and construct your request:
```
PUT /orderGroup/123/status

"Deactivated"
```
and if there are no errors, your `BuyOrder` has been deactivated and is no longer active in the market. There is no return besides a `200 OK`.
