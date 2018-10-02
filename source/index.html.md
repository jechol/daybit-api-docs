---
title: Daybit API Reference

language_tabs:
  - python

includes:
  - pydaybit
  
search: true
---

# **Introduction**

Daybit's API works in socket server. Based on your task and subject of required action, you can either subscribe to `api` or `subscription`s channels.

This document provides basic information of the API, usage of wrapper, and working examples of the wrapper.

## Libraries

Exchange is using [Elixir](https://elixir-lang.org/) language and built on [Phoenix framework](https://phoenixframework.org/). Exchange API can be accessed from Phoenix client, which makes it easy to connect to Phoenix sockets. For now, Daybit officially provides wrapper for Python language only.

* PyDaybit(Python): [https://github.com/daybit-exchange/pydaybit](https://github.com/daybit-exchange/pydaybit/)

For other languages, please refer below libraries to implement the features of this Exchange API.

* Phoenix.js(Javascript): [https://github.com/phoenixframework/phoenix](https://github.com/phoenixframework/phoenix/)
* SwiftPhoenixClient(Swift): [https://github.com/davidstump/SwiftPhoenixClient](https://github.com/davidstump/SwiftPhoenixClient/)
* PhoenixSharp(C#): [https://github.com/Mazyod/PhoenixSharp](https://github.com/Mazyod/PhoenixSharp/)

## Host address

```shell
    $ git clone https://github.com/daybit-exchange/pydaybit
    $ pip install -e pydaybit
```

* Exchange API endpoint : [wss://api.daybit.com/v1/user_api_socket/websocket/](wss://api.daybit.com/v1/user_api_socket/websocket/)

* API Python wrapper : [https://github.com/daybit-exchange/pydaybit/](https://github.com/daybit-exchange/pydaybit/)

For wrapper installation, please look right column.


# Common

## Terms

* `quote`: Asking token. ex) `BTC` from `ETH/BTC`.
* `base`: Base token. ex) `ETH` from `ETH/BTC`.
* [Channels](https://hexdocs.pm/phoenix/channels.html) are a part of Phoenix that allow us to easily add soft-realtime features to our applications. Channels are based on a simple idea - sending and receiving messages. Senders broadcast messages about topics. Receivers subscribe to topics so that they can get those messages. Senders and receivers can switch roles on the same topic at any time.
* [Topic](https://hexdocs.pm/phoenix/channels.html#topics) are string identifiers - names that the various layers use in order to make sure messages end up in the right place. As we saw above, topics can use wildcards. This allows for a useful “topic:subtopic” convention. Often, you’ll compose topics using record IDs from your application layer, such as `users:123`.

## Types

* `integer`: `integer` data type. ex) `123`
* `decimal`: Decimal number. This is `string` data type to precisely express the exact amount of number that is not expressed in ordinary decimal numbers. ex) `"880.524"`
* `string`: `string`. ex) `"string"`
* `boolean`: `boolean`. ex) `true`, `false`
* `unix_timestamp`: `millisecond` unit unix timestamp. ex) `1528269989516`
* `csv`: string based comma separated values. ex) `"1, 2, 3"`

## Authorization

The usage of API is restricted by given right to each API key. You would get `unauthenticated` response error_code if you called API that is not accessible from your API key. Please look below for the types of API key and details of it.

| Type | Description |
|------|-------------|
| public_data | Authorized to access public data. ex) Market Summary, Order Book and so on
| private_data | Authorized to access private data. ex) Asset and so on
| trade | Authorized to call trade related APIs. ex) Order, Trade and so on
| transaction | Authorized to call transaction related APIs. ex) Deposit, Wdrl and so on

## Response format

Basically there are two types of response formats. Based on the result of API calls, you would get one of success or fail models. If there was an error while running the API call, proper message will be returned along with error_code.

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

## Size

Maximum value for `size` is `30`. API will replace `size` to `30` if you placed the number larger than `30`.

## Market

`quote` and `base` are both required if request asked certain market's information by `quote` and `base`.

## Error list

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

## Models

Below are generic data models.

### Coin

identifier: `sym`

| Name | Type | Description |
|---|---|---|
| sym | string | Token symbol |
| native_decimal_point | integer | Withdrawal amount decimal restriction |
| amount_decimal_point | integer | Order amount decimal point restriction |
| tick_amount | decimal | Order amount unit |
| deposit_confirm | integer | Number of confirms required for deposit completion |
| wdrl_confirm | integer | Number of confirms required for withdrawal completion |
| public | boolean | ㅣ Listed on the exchange (`true`) or not (`false`) |
| name | string | Name of token (locale applied) |
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
| sym | string | Token symbol |
| usdt_price | decimal | USDT exchanged price |

### Quote coin

identifier: `sym`

| Name | Type | Description |
|---|---|---|
| sym | string | Token symbol |

### Market

identifier: `quote`, `base`

| Name | Type | Description |
|---|---|---|
| quote | string | Quote token |
| base | string | Base token |
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
| quote | string | Quote token |
| base | string | Base token |
| seconds | integer | Time unit |
| open | decimal | Opening price |
| close | decimal | Closing price |
| high | decimal | Highest price |
| low | decimal | Lowest price |
| volatility | decimal | Price volatility |
| quote_vol | decimal | Trading volume per quote token |
| base_vol | decimal | Trading volume per base token |

### Order book

identifier: `quote`, `base`, `price_intvl`, `min_price`

| Name | Type | Description |
|---|---|---|
| quote | string | Quote token |
| base | string | Base token |
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
| quote | string | Quote token |
| base | string | Base token |
| intvl | integer | Time interval |
| start_time | unix_timestamp | Start time of price history |
| end_time | unix_timestamp | End time of price history |
| open | decimal | Opening price |
| close | decimal | Closing price |
| high | decimal | Highest price |
| low | decimal | Lowest price |
| base_vol | decimal | Trading volume per base token |
| quote_vol | decimal | Trading volume per quote token |

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
| coin | string | Token symbol |
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
| quote | string | Quote token |
| base | string | Base token |
| price | decimal | Price |
| role | string | Order role. `"both"`, `"maker_only"`, `"taker_only"` |
| cond_type | string | COnditional order type. `"none"`, `"le"`, `"ge"`, `"fall_from_top"`, `"rise_from_bottom"` |
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
| quote | string | Quote token |
| base | string | Base token |
| quote_amount | decimal | Quote token trade amount |
| base_amount | decimal | Base token trade amount |
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
| coin | string | Token symbol |
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
| coin | string | Withdrawal token |
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
| coin | string | Token symbol |
| amount | decimal | Airdrop amount |
| description | string | Airdrop description |
| airdropped_at | unix_timestamp | Airdrop time |
