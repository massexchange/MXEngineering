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

* An `Attribute` represents some piece of information about the asset. This can be any sort of information, for any sort of asset. For example, to use a fruit market analogy, if its green, if its an apple, if its large or small. Splitting up information about assets in this way allows us to search through them in useful ways. We talk about attributes in the format `Type: Attribute Value`, ie, `Network: AMC`.

* Attributes in turn have `Types`, which describe what sort of information attributes of that type convey about assets. In the same analogy, the types would be `Color`, `Fruit`, `Size`. Thinking about types allows us to measure the specificity (defined later) of an asset.

### Market

#### Rules

* The market operates on **New York time (EDT, GMT-5)**, and is open from **9:30AM-4:30PM**. Orders may be submitted at any time, but will **only be processed** when the market is open.

* Orders are processed in the order they are submitted, even if submitted while the market was closed. If an order is edited, it goes to the end of the queue.

* Supply and demand are matched on, in order of priority, **asset, price, and submission order**.

* Sellers may only enter sell orders for assets they actually have in their catalog. Buyers may enter any valid order, even if there is nothing available in the market to match it.

* Orders entered into the market represent an intention to buy/sell, and once an order is matched it is considered **non-cancellable**. Orders should not be submitted unless you are willing to commit to the outcome.

#### Order Semantics

* Asset matching is based on `Specificity`, which refers to the precision with which an order describes the asset. For example:
    * User A enters a `Sell Order` for `Fruit: Apple | Color: Green | Size: Small`
    * User B enters a `Buy Order` for `Color: Green | Size: Small`
    * Order A is `more specific` than Order B, as it describes an asset more precisely (provides more information about it). Since the demand here can be met with supply for ANY small green fruit, the fact that Order A is for an apple is irrelevant. If there were another sell order for a pear, Order B could also match against it.

* Orders can be partially filled, ie an order for 10 can match against an order for 5, and the remainder will stay in the market.

* Orders can be matched for assets running, at the earliest, tomorrow. Today's assets cannot be matched against; processing time is required to traffic matches. Sellers can configure a longer trafficking window if one day is not adequate.

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

##### `OrderGroup`

* qty: `Number`
    * Quantity of units desired
* flightStartDate: `Date`
* flightEndDate: `Date`
* marketId: `Id`
    * Market the `OrderGroup` will eventually be submitted into
* price: `Currency`
* asset: `Set<Attribute>`
* status: `OrderStatus`
    * this field is read-only

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

##### Market Query

Query avails for a given `Asset` in a `Market`

`GET /market/sells`
Query params: `OrderQuery`

Returns: `Page<AvailablilityGroup>`

##### Attribute Query
Query all known attributes

`GET /attr`
Returns: `Attribute`

##### Campaign Creation
Create a new `Campaign`

`POST /campaign`
Request body: `Campaign`

##### Buy Order Submission

Add a new `OrderGroup` to a `Campaign`

`POST /campaign/{campId}/orders`
Path variables:
* campId: `Id`
`Campaign` being added to

Request body: `OrderGroup`

##### Buy Order Query

Get the current version of your `OrderGroup`

`GET /orderGroup/{groupId}`
Path variables:
* groupId: `Id`
`OrderGroup` being queried

Returns: `OrderGroup`

##### Match Query

Query your `Match` history

`GET /match`
Query params: `MatchQuery`

### Workflow
1. Seller submits their supply into the market.
2. Buyer queries the market to see availible supply.
3. Buyer submits their demand into the market.
4. If there is is no match, the orders rest until cancelled or expired.
5. If there is a match, the inventory is transferred and taken off the market.
6. After a match, buyer can see results on the Log.

### Examples

Provided are several example interactions with the MX API.

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
                    "value": "AMC"
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

This sample data indicates that for the date range we queried, there are 10 units available of `Network: AMC` inventory, from `11/01/2018`-`11/03/2018` at `$1000.00` each.
