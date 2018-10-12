# 4C integration doc

###### This document will give a brief overview of the MX media-buying platform, and explain how to interact with its API.

### Platform Overview

#### Domain Concepts

Within the MX platform we think in terms of `Organizations`, `Users`, `Teams`, `Roles`, `Markets`, `Orders`, `Assets`, `Attributes`, and `Matches`.

* An `Organization` represents a single buyer or seller, and is analogous to a company account. There are many `Users` within an `Organization`, and any action a user takes looks, from the outside, as if it was done by the organization. Users are assigned to `Teams`, and given `Roles` within those teams which constrain which actions they are allowed to take.

* A `Market` is an environment in which transactions happen. There is an `Open Market` and user defined `Private Markets`. Initially, all transactions will happen in the latter. `Orders` in private markets can only be seen by market members, which are `Organizations`, and defined by the market owner.

* An `Order` is how we represent supply or demand entered into a `Market`. An order represents the intention of an `Organization` to buy/sell some amount of a certain `Asset` in a `Market`, at some price.

* An `Asset` is what we call media that is bought and sold. It is composed of `Attributes` and additional information, like the date and time. An `Attribute` represents some piece of information about the asset. `Attributes` in turn have `Types`, which can be thought of as classifications of information about assets. This is displayed in the UI in the format of `Type: Attribute Value`, ie, `Network: AMC`.

* A `Match` is the outcome of a market transaction; it represents an intersection of supply and demand, on terms meeting the requirements of both sides. Matches are analogous to non-preemptible orders in traditional media buys.

#### Components

The buy-side MX platform consists of several areas of functionality; `Organization Management`, the `Market`, and `Brokerage`. There are others, but these are all that is relevant for this implementation.

* Organization Management
    * Profile
    * Settings
    * Permissions
* Market
    * Query (aka View)
    * *Watch (Planned)*
    * Private
* Brokerage
    * Campaigns
    * Active
    * History
    * Log

### Market

#### Rules

* The market operates on **New York time (EDT, GMT-5)**, and is open from **9:30AM-4:30PM**. Orders may be submitted at any time, but will **only be processed** when the market is open.

* Orders are processed in a **first-in-first-out** queue; if your order is submitted before another one, it is guaranteed to get access to supply first. If an order is edited, its position in the queue is reset.

* Supply and demand are matched by **asset first, and by price second**. Submission order is only relevant when two competing orders have the same price.

* Sellers may only enter sell orders for assets they actually have in their catalog. Buyers may enter any valid order, even if there is nothing available in the market to match it.

* Orders entered into the market represent an intention to buy/sell, and once an order is matched it is considered **non-cancellable**. Orders should not be submitted unless you are willing to commit to the outcome.

#### Order Semantics

* Asset matching is based on `Specificity`, which refers to the precision with which an order describes the asset. For example:
    * User A enters a `Sell Order` for `Fruit: Apple | Color: Green | Size: Small`
    * User B enters a `Buy Order` for `Color: Green | Size: Small`
    * Order A is `more specific` than Order B, as it describes an asset more precisely (provides more information about it). Since the demand here can be met with supply for ANY small green fruit, the fact that Order A is for an apple is irrelevant. If there were another sell order for a pear, Order B could also match against it.

* Orders can be partially filled, ie an order for 10 can match against an order for 5, and the remainder will stay in the market.

* Orders cannot be matched FOR the current day, ie an order for today's media can match, at the latest, yesterday.

#### Workflow
1. Seller submits their supply into the market.
2. Buyer queries the market to see avails.
3. Buyer submits their demand into the market.
    a. create a `Campaign` per brand.
    b. create `OrderGroups` for all desired `Assets` in that campaign.
    c. submit ordergroups into the market.
4. If there is is no match, the orders rest until cancelled or expired.
5. If there is a match, the inventory is transferred and taken off the market.
6. After a match, buyer can see results on the Log.

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

#### Entities

All entities have `Ids`; they are unique, but not globally. There will never be duplicate ids within a single entity, but there may be across entities.

Not all fields are documented here; the ones left off are safe to ignore.

##### `AvailablilityGroup`

Represents a group of avails with the same asset.

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

##### `Deal`

* network: `String`
* unitId: `String`
* broadcastDate: `Date`
* airTime: `Time`
* status: `DealStatus`
* marketplace: `Marketplace`
* dealSalesType: `DealSalesType`
* unitStatus: `UnitStatus`
* logStatus: `LogStatus`
* spotType: `SpotType`
* guaranteed: `Boolean`
* onLog: `Boolean`
* agency: `String`
* advertiser: `String`
* brand: `String`
* category: `String`
* grossDollars: `Currency`
* asset: `Set<Pair<String, String>>`

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

##### Buy Order Status Query
Query the current status of your `OrderGroup`

`GET /orderGroup/{groupId}`
Path variables:
* groupId: `Id`
`OrderGroup` being queried

Returns: `OrderGroup`

##### Match Query

Query your `Match` history

`GET /match`
Query params: `MatchQuery`

##### Log Query

Query `Deal` status for a particular advertiser

`GET /deal/advertiser/{advName}`
Path variables:
* advName: `String`

Returns: `Set<Deal>`
