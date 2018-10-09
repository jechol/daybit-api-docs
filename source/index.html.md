---
title: Daybit API Reference

includes:
  - pydaybit
  
search: true

toc_footers:
 - <a href='https://www.daybit.com'>Daybit</a>
---

# Introduction

Target audience of this document is those who are capable of writing proper program source code. This document contains examples that might put your assets in danger, even losing your assets. You need to fully understand descriptions and functionalities of the source code before run them. Use this API at your own risk and all kinds of outcome of using this API is your responsibility.

You need to generate an API key pair before using Daybit API from [Daybit's website](https://www.daybit.com). An API key pair has multiple types of authorization - receiving [Candle data](https://en.wikipedia.org/wiki/Candlestick_chart) or [Order book](https://en.wikipedia.org/wiki/Order_book_(trading)), checking personal assets, trading personal assets, and withdrawal authorizations. Please refer [Authorization](#authorization) for details of authorization.

Basically there are two types of Daybit's APIs. First one is an [API Call](#api-calls) - client sends a request and server responds accordingly. Usually this type of a call is used for asset trading, deposit or withdraw.

Second type is a [Subscription](#subscriptions) which allows you to subscribe to an API and get a continuous notification from the server. Based on type of a notification, the notification include price change of coins, information of one's wallet, result of one's order and so on.

Daybit APIs are implemented based on websocket connection and follow the format defined in Phoenix Framework. Daybit also provides [Pydaybit](#pydaybit) written in Python which allows developers to easily use Daybit API.

If you are not familiar with programming, it would be better to take a look at examples of [Pydaybit](#pydaybit) first.
 
# Authorization

Before you are using Daybit APIs, you first need to generate API key pair with proper authorization. API has multiple types of authorization - receiving [candle data](https://en.wikipedia.org/wiki/Candlestick_chart) or a [order book](https://en.wikipedia.org/wiki/Order_book_(trading)), checking personal assets, trading personal assets, and withdrawal authorizations.

| Type | Description |
|------|-------------|
| public_data | Authorized to access public data (ex, Market Summary, Order Book and so on). To get this authorization level from an API key, please include 'Market Inquiry' at the API key pair creation.
| private_data | Authorized to access private data (ex, Asset and so on). To get this authorization level from an API key, please include 'Private Asset Inquiry' at the API key pair creation.
| trade | Authorized to call trade related APIs (ex, Order, Trade and so on). To get this authorization level from an API key, please include 'Trading API' at the API key pair creation.
| transaction | Authorized to call transaction related APIs (ex, Deposit, Wdrl and so on). To get this authorization level from API key, please include 'Withdrawal API' at the API key pair creation.

You can generate maximum 5 API key pairs per account in [API Keys](https://www.daybit.com/mypage/api-managements) tab in My Page. Each API Key pair can have following authorizations:

* `public_data`
* `public_data`, `private_data`, `trade`
* `public_data`, `private_data`, `transaction`
* `public_data`, `private_data`, `trade`, `transaction`

The usage of APIs is restricted by given right to each API key pair. You would get `unauthenticated` response error_code if you called a API that is not accessible from your API key pair. Please look above for types of a API key pair and details of it.

# Host address

* Exchange API endpoint : [wss://api.daybit.com/v1/user_api_socket/websocket/](wss://api.daybit.com/v1/user_api_socket/websocket/)

# APIs

Basically there are two types of Daybit's APIs. First one is an [API Call](#api-calls) - client sends a request and server responds accordingly. Usually this type of a call is used for asset trading, deposit or withdraw. Second type is a [Subscription](#subscriptions) which allows you to subscribe to an API and get continuous notification from the server. Based on type of a notification, each notification include price change of coins, information of one's wallet, result of one's order and so on.

## Channels
Channels are based on a simple idea - sending and receiving messages. Senders broadcast messages about topics. Receivers subscribe to topics so that they can get those messages. Senders and receivers can switch roles on the same topic at any time.

### Topics
Topic are string identifiers of channels that the various layers use in order to make sure messages end up in the right place. Daybit is using following types of topics: `/api`, `/subscription:coins`, `/subscription:market_summaries;<market_summary_intvl>`.

### Event
Event is `string` representing specific actions of the channel. `phx_join` is for joinning the channel and `phx_leave` is for leaving the channel. [API Calls](#api-calls) includes types of request in event.
 

### Message

> Example of `create_wdrl` 

```python
import asyncio
import logging

from pydaybit import Daybit

logger = logging.getLogger('pydaybit')
logger.setLevel(logging.DEBUG)

stream_handler = logging.StreamHandler()
stream_handler.setFormatter(logging.Formatter('%(message)s'))
logger.addHandler(stream_handler)


async def daybit_create_wdrl():
    async with Daybit() as daybit:
        await daybit.create_wdrl(coin='BTC', to_addr='ABTCADDRESS', amount='0.05')


asyncio.get_event_loop().run_until_complete(daybit_create_wdrl())
```

> Output of the example

```console
> {"join_ref": "1", "ref": "1", "topic": "/api", "event": "phx_join", "payload": {"timestamp": 1538739991045}, "timeout": 3000}
< {"topic":"/api","ref":"1","payload":{"status":"ok","response":{}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "2", "topic": "/api", "event": "create_wdrl", "payload": {"to_addr": "ABTCADDRESS", "timestamp": 1538739991059, "coin": "BTC", "amount": "0.05"}, "timeout": 3000}
< {"topic":"/api","ref":"2","payload":{"status":"ok","response":{"data":{"wdrl_to_tag":null,"wdrl_to_org":null,"wdrl_to_addr":"A_BTC_ADDRESS","wdrl_status":"queued","type":"wdrl","txid":null,"tx_link_url":"https://live.blockcypher.com/btc/tx/","req_confirm":2,"id":5882,"deposit_status":null,"created_at":1538739991072,"confirm":0,"completed_at":null,"coin":"BTC","amount":"0.050000000000000000"}}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "3", "topic": "/api", "event": "phx_leave", "payload": {"timestamp": 1538739991124}, "timeout": 3000}
< {"topic":"/api","ref":"3","payload":{"status":"ok","response":{}},"event":"phx_reply"}
< {"topic":"/api","ref":"1","payload":{},"event":"phx_close"}
```

Following information is delievered in the format of [json](https://en.wikipedia.org/wiki/JSON) object.

`topic` - The string topic or topic:subtopic pair namespace, for example “messages”, “messages:123”
`event` - The string event name, for example “phx_join”
`payload` - The message payload
`ref` - The unique string ref
`join_ref` - ref of joinning the channel

## API Calls
[`create_order`](#create_order), [`cancel_order`](#cancel_order), [`cancel_orders`](#cancel_orders), [`cancel_all_my_orders`](#cancel_all_my_orders), [`create_wdrl`](#create_wdrl), [`get_server_time`](#get_server_time) are types of API calls. You can use this API by sending required event and proper `payload` value in `/api` channel.

* Topic: `/api`

* Event: `get_server_time`, `create_order`, `cancel_order`, `cancel_orders`, `cancel_all_my_orders`, or `create_wdrl`

* Rate limit: Limit of calls for every second.

<aside class="notice">
It is recommended to retrieve data from `notification` of `/subscription:<sub_topic>` topic not `response` from `/api` topic. It might cause confliction at `insert` action from `notification` because of two separate data roots.
</aside>

> Request `create_wdrl`

```console
{"join_ref": "1", "ref": "2", "topic": "/api", "event": "create_wdrl", "payload": {"to_addr": "ABTCADDRESS", "timestamp": 1538739991059, "coin": "BTC", "amount": "0.05"}, "timeout": 3000}
```

> Response

```console
{"topic":"/api","ref":"2","payload":{"status":"ok","response":{"data":{"wdrl_to_tag":null,"wdrl_to_org":null,"wdrl_to_addr":"ABTCADDRESS","wdrl_status":"queued","type":"wdrl","txid":null,"tx_link_url":"https://live.blockcypher.com/btc/tx/","req_confirm":2,"id":5882,"deposit_status":null,"created_at":1538739991072,"confirm":0,"completed_at":null,"coin":"BTC","amount":"0.050000000000000000"}}},"event":"phx_reply"}
```

## Subscriptions
For using API Subscription, first you need to join `/subscription:<subtopic>` channel and send `request` event. Once you are successfully subscribed by joinning the channel, the server sends `notfication` for any kind of updated data. [`coins`](#coins), [`coin_prices`](#coin_prices), [`quote_coins`](#quote_coins), [`markets`](#markets), [`market_summary_intvls`](#market_summary_intvls), [`market_summaries`](#market_summaries), [`order_books`](#order_books), [`price_history_intvls`](#price_history_intvls), [`price_histories`](#price_histories), [`trades`](#trades), [`my_users`](#my_users), [`my_assets`](#my_assets), [`my_orders`](#my_orders), [`my_trades`](#my_trades), [`my_tx_summaries`](#my_tx_summaries), [`my_airdrop_histories`](#my_airdrop_histories) are the topics you can use in API subscription.


* Topic: `/subscription:<sub_topic>`

* Event: `request` (push) or `notification` (pull). [Message](https://hexdocs.pm/phoenix/Phoenix.Socket.Message.html) transported from client and server have `request` and `notification` events, respectively. When you subscribe to the event with `request` event, you will get either `init` or `upsert` action response from the API. After that, you would get one of `insert`, `update`, `upsert`, or `delete` from the API with `notification` event. For more information of actions, please look following [Action](#action).

* Rate limit: Limit of calls for every second. Only applicable for `request`.

> Example of `coins`

```python
import asyncio
import logging

from pydaybit import Daybit

logger = logging.getLogger('pydaybit')
logger.setLevel(logging.DEBUG)

stream_handler = logging.StreamHandler()
stream_handler.setFormatter(logging.Formatter('%(message)s'))
logger.addHandler(stream_handler)


async def daybit_coins():
    async with Daybit() as daybit:
        await daybit.coins()


asyncio.get_event_loop().run_until_complete(daybit_coins())
```

> Output

```console
> {"join_ref": "1", "ref": "1", "topic": "/subscription:coins", "event": "phx_join", "payload": {"timestamp": 1538740890363}, "timeout": 3000}
< {"topic":"/subscription:coins","ref":"1","payload":{"status":"ok","response":{}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "2", "topic": "/subscription:coins", "event": "request", "payload": {"timestamp": 1538740890374}, "timeout": 3000}
< {"topic":"/subscription:coins","ref":"2","payload":{"status":"ok","response":{"data":[{"data":[{"wdrl_fee":"5.00000000","wdrl_enabled":true,"wdrl_confirm":2,"tradable":true,"tick_amount":"0.10000000","sym":"USDT","native_decimal_places":2,"name":"Tether","min_wdrl":"10.00000000","min_deposit":"10.00000000","has_tag":false,"has_org":false,"deposit_enabled":true,"deposit_confirm":2} ... ],"action":"init"}]}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "3", "topic": "/subscription:coins", "event": "phx_leave", "payload": {"timestamp": 1538740890386}, "timeout": 3000}
< {"topic":"/subscription:coins","ref":"3","payload":{"status":"ok","response":{}},"event":"phx_reply"}
< {"topic":"/subscription:coins","ref":"1","payload":{},"event":"phx_close"}
```

## Rate limit

Each API has limit of calls for every second. You will get `api_exceeded_rate_limit` error_code in response if you exceed the limit.

## Timestamp

All request takes `timestamp` and `timeout` to prevent unexpected calls because of network delay and so on.

- `timestamp`: `unix_timestamp`
- `timeout`(**optional**): `ms` unit time in `integer`. Default value is `3000`.

Request will be rejected in next condition: `server time` - `timestamp` > `timeout`.

If there was a problem, below error_code will be returned in response.
- `api_invalid_timestamp_or_timeout`: `timestamp` and/or `timeout` are not existed or they are not `integer` (unix timestamp in millisecond).
- `api_timeout`: Rejected because of request time out.

## Response format

Basically there are two types of response formats. Based on the result of API calls, you would get one of success or fail models. Please see success and fail response examples on right column. You can also find [Error List](#error-list) in below.

> Success

```python
{
  # plain json object with proper data
}
```

> Fail

```python
{
  "error_code": 'string',
  "message": 'string'
}
```

# Libraries

Exchange is using [Elixir](https://elixir-lang.org/) language and built on [Phoenix framework](https://phoenixframework.org/). Exchange API can be accessed from Phoenix client, which makes it easy to connect to Phoenix sockets. For now, Daybit officially provides wrapper for Python language only.

* PyDaybit(Python): [https://github.com/daybit-exchange/pydaybit](https://github.com/daybit-exchange/pydaybit/)

For other languages, please refer below libraries to implement the features of this Exchange API.

* Phoenix.js(Javascript): [https://github.com/phoenixframework/phoenix](https://github.com/phoenixframework/phoenix/)
* SwiftPhoenixClient(Swift): [https://github.com/davidstump/SwiftPhoenixClient](https://github.com/davidstump/SwiftPhoenixClient/)
* PhoenixSharp(C#): [https://github.com/Mazyod/PhoenixSharp](https://github.com/Mazyod/PhoenixSharp/)

# Error list

### General

* unauthenticated: Unauthenticated user action
* invalid_arguments: Invalid arguments in request
* resource_not_found: Resource not found

### Api

* api_invalid_timestamp_or_timeout: `timestamp` and/or `timeout` of request is not valid
* api_timeout: Timeout happens by requested `timestamp` and/or `timeout`
* api_exceeded_rate_limit: Rate limit exceeded
* api_invalid_param_types: Invalid request parameter type
* api_required_params_not_provided: Missing required parameter

### Order

* order_invalid_market: Invalid market(`quote`, `base`)
* order_not_tradable_coin: Coin trade suspended
* order_not_sellable_market: Selling is suspended in the market
* order_not_buyable_market: Buying is suspended in the market
* order_invalid_price: Invalid price
* order_invalid_amount: Invalid amount
* order_only_both_role_can_be_cond: Conditional order is available only when `role` is `both`
* order_out_of_price_range: Out of price range (Selling: 20% ~ 200%, Buying: 50% ~ 500%)
* order_exceeded_max_tstops: Exceeded maximum Trailing*Stop order count
* order_suspended_due_to_frequent_canceling: Order suspended due to frequent canceling
* order_exceeds_my_asset_values: Order exceeded my asset values
* order_violates_min_quote: Order amount is less than minimum quote
* order_already_closed: Order already closed

### Wdrl

* wdrl_suspended_coin: Coin was Suspended to withdraw
* wdrl_precision_error: Decimal place accuracy error
* wdrl_under_min_amount: One time withdrawal amount is less than minimum
* wdrl_over_daily_wdrl_limit: Withdrawal amount exceeded daily limit
* wdrl_exceeds_my_asset_values: Withdrawal amount exceeded my asset
* wdrl_needs_to_tag: Missing `to_tag` parameter
* wdrl_invalid_addr: Invalid address

# Action

Response holds `action` which helps you to understand how to handle the response.

* `init` : Dump all previous data and initialize everything with most recent data.
* `insert` : Add data to data set, as most recent data.
* `update` : Search in data set and replace if it was found.
* `upsert` : Search in data set and replace if it was found, or insert if there's no matching data.
* `delete` : Search in data set and remove if it was found.

# Types

### integer 

`integer` data type.
 
* `123`
* `20181010`

### decimal
Decimal number. This is `string` data type to precisely express the exact amount of number which can't be expressed in ordinary decimal number types.

* `"880.524"`
* `"59.55000000000000000:`

### string

`string`.
 
* `"string"`
* `"bitcoin"`
* `"Pydaybit"`

 
### boolean

`boolean`.
 
* `true`
* `false`

### unix_timestamp

`millisecond` unit unix timestamp.
 
* `1528269989516`

### csv

string based comma separated values.
 
* `"1, 2, 3"`

# Models

Below are generic data models.

### Coin

identifier: `sym`

| Name | Type | Description |
|---|---|---|
| sym | string | Coin symbol |
| native_decimal_point | integer | Withdrawal amount decimal restriction |
| amount_decimal_point | integer | Order amount decimal point restriction |
| tick_amount | decimal | Order amount unit |
| deposit_confirm | integer | Number of confirms required for deposit completion |
| wdrl_confirm | integer | Number of confirms required for withdrawal completion |
| public | boolean | Listed on the exchange (`true`) or not (`false`) |
| name | string | Name of coin (locale applied) |
| tradable | boolean | Tradable or not |
| deposit_enabled | boolean | Deposit enabled or not |
| wdrl_enabled | boolean | Withdrawal enabled or not |
| wdrl_fee | decimal | Withdrawal fee |
| min_deposit | decimal | Minimum deposit amount |
| min_wdrl | decimal | Minimum withdrawal amount |
| has_tag | boolean | to_tag required or not for deposit/withdrawal |
| has_org | boolean | to_org required or not for deposit/withdrawal |

### Coin price

identifier: `sym`

| Name | Type | Description |
|---|---|---|
| sym | string | Coin symbol |
| usdt_price | decimal | USDT exchanged price |

### Quote coin

identifier: `sym`

| Name | Type | Description |
|---|---|---|
| sym | string | Coin symbol |

### Market

identifier: `quote`, `base`

| Name | Type | Description |
|---|---|---|
| quote | string | Quote coin |
| base | string | Base coin |
| tick_price | decimal | Order price unit |
| sellable | boolean | Sellable or not |
| buyable | boolean | Buyable or not |
| tick_levels | integer | Number of levels for order book existence per `tick_price`. Order book is increased ten times for each level. ex) tick_price = 0.01, order book intvl = 0.01, 0.1, 1, 10, 100 |

### Market summary interval

identifier: `seconds`

| Name | Type | Description |
|---|---|---|
| seconds | integer | Market summary interval unit |

### Market summary

identifier: `quote`, `base`, `seconds`

| Name | Type | Description |
|---|---|---|
| quote | string | Quote coin |
| base | string | Base coin |
| seconds | integer | Time unit |
| open | decimal | Opening price |
| close | decimal | Closing price |
| high | decimal | Highest price |
| low | decimal | Lowest price |
| volatility | decimal | Price volatility |
| quote_vol | decimal | Trading volume per quote coin |
| base_vol | decimal | Trading volume per base coin |

### Order book

identifier: `quote`, `base`, `price_intvl`, `min_price`

| Name | Type | Description |
|---|---|---|
| quote | string | Quote coin |
| base | string | Base coin |
| price_intvl | decimal | Price interval unit |
| min_price | decimal | Minimum price |
| max_price | decimal | Maximum price |
| sell_vol | decimal | Selling volume |
| buy_vol | decimal | Buying volume |

### Price history interval

identifier: `seconds`

| Name | Type | Description |
|---|---|---|
| seconds | integer | Price history interval unit |

### Price history

identifier: `quote`, `base`, `intvl`, `start_time`

| Name | Type | Description |
|---|---|---|
| quote | string | Quote coin |
| base | string | Base coin |
| intvl | integer | Time interval |
| start_time | unix_timestamp | Start time of price history |
| end_time | unix_timestamp | End time of price history |
| open | decimal | Opening price |
| close | decimal | Closing price |
| high | decimal | Highest price |
| low | decimal | Lowest price |
| base_vol | decimal | Trading volume per base coin |
| quote_vol | decimal | Trading volume per quote coin |

### User

identifier: -

| Name | Type | Description |
|---|---|---|
| maker_fee_rate | decimal | Maker fee rate |
| taker_fee_rate | decimal | Taker fee rate |
| one_day_wdrl_usdt_limit | decimal | Withdrawal limit for 24 hours in USDT |

### Asset

identifier: `coin`

| Name | Type | Description |
|---|---|---|
| coin | string | Coin symbol |
| total | decimal | Total amount |
| reserved | decimal | Reserved amount for order and so on |
| available | decimal | Availalbe amount for order |
| investment_usdt | decimal | Total investment in USDT |

### Order

identifier: `id`

| Name | Type | Description |
|---|---|---|
| id | integer | id |
| sell | boolean | Selling (`true`) or buying (`false`) |
| quote | string | Quote coin |
| base | string | Base coin |
| price | decimal | Price |
| role | string | Order role. `"both"`, `"maker_only"`, `"taker_only"` |
| cond_type | string | Conditional order type. `"none"`, `"le"`, `"ge"`, `"fall_from_top"`, `"rise_from_bottom"` |
| cond_value | decimal or null | Conditional order price value |
| coin_fee | decimal | Fee |
| amount | decimal | Order amount |
| filled | decimal | Order filled (per base at selling) |
| filled_quote | decimal | Order filled (per quote at buying) |
| unfilled | decimal | Order unfilled |
| received_at | unix_timestamp | Order received time |
| placed_at | unix_timestamp or null | Order placed time (to order book) |
| closed_at | unix_timestamp or null | Order closed time |
| status | string | Order status (received, placed, completed, canceled) |

### Trade

identifier: `id`

| Name | Type | Description |
|---|---|---|
| id | integer | id |
| quote | string | Quote coin |
| base | string | Base coin |
| quote_amount | decimal | Quote coin trade amount |
| base_amount | decimal | Base coin trade amount |
| price | decimal | Trade closing price |
| taker_sold | boolean | Taker is seller (`true`) or buyer (`false`) |
| price_dec_digit | integer | Valid price decimal digit |
| base_amount_dec_digit | integer | Valid base amount decimal digit |
| exec_at | unix_timestamp | Execution time |
| Below are only available from [my_trades](#my-trades)  |
| order_id | integer | Trade order id |
| fee | decimal | Fee |
| sell | boolean | Selling (`true`) or buying (`false`) |

### Deposit

identifier: `id`

| Name | Type | Description |
|---|---|---|
| id | integer | id |
| coin | string | Coin symbol |
| txid | string | Deposit transaction id |
| amount | decimal | Deposit amount |
| usdt_amount | decimal | Deposit USDT amount |
| found_at | unix_timestamp | Deposit transaction found time |
| confirm | integer | Confirm count |
| confirm_checked_at | unix_timestamp | Confirm checked time |
| applied_to_asset_at | unix_timestamp or null | Asset applied time |

### Wdrl

identifier: `id`

| Name | Type | Description |
|---|---|---|
| id | integer | id |
| coin | string | Withdrawal coin |
| to_addr | string | Withdrawal receiving address |
| to_tag | string or null | Withdrawal tag |
| to_org | string or null | Withdrawal organization |
| amount | decimal | Withdrawal amount |
| usdt_amount | decimal | Withdrawal amount in USDT |
| fee | decimal | Withdrawal fee |
| requested_at | unix_timestamp | Withdrawal requested time |
| txid | string or null | Withdrawal transaction id |
| tx_created_at | unix_timestamp or null | Withdrawal transaction created time |
| completed_at | unix_timestamp or null | Withdrawal completed time |

### Airdrop

identifier: `id`

| Name | Type | Description |
|---|---|---|
| id | integer | id |
| coin | string | coin symbol |
| amount | decimal | Airdrop amount |
| description | string | Airdrop description |
| airdropped_at | unix_timestamp | Airdrop time |
