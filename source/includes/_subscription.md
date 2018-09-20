# **Topic: Subscription**

This section explains how you could implement various features of the Exchange API. If you subscribed to certain `/subscription:<sub_topic>`, you will get notification from the server when relevant modification happens.

* topic: `/subscription:<sub_topic>`

* event: `request` (push) or `notification` (pull). [Message](https://hexdocs.pm/phoenix/Phoenix.Socket.Message.html) transported from client and server have `request` and `notification` events, respectively. When you subscribe to the event with `request` event, you will get `init` action response from the API. After that, you would get one of `insert`, `update`, `upsert`, or `delete` from the API with `notification` event. For more information of actions, please look [Action](#action).

* rate limit: Limit of calls for every second. Only applicable for `request`.

> Response data types of `request` and `notification` events are identical.

```python
{
  "data": [
    {
      "action": string,
      "data": [
        {} # data
      ]
    }
  ]
}
```

### Action

Response holds `action` which helps you to understand how to handle the `data`.

* `init` : Dump all previous data and initialize everything with most recent data.
* `insert` : Add data to current data set, as most recent data.
* `update` : Search the given data and replace if it exists.
* `upsert` : Search the given data and replace if it exists, or insert if it doesn't exist.
* `delete` : Search the given data and remove if it exists.

# Public data

## Coins

### subtopic: `coins`

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_coins():
    async with Daybit() as daybit:
        pprint(await daybit.coins())
```

> Example Response

```python
{
  "data": [
    {
      "action":"init",
      "data": [
        {
          "sym":"USDT",
          "native_decimal_places":2,
          "tick_amount":"0.10000000",
          "deposit_confirm":3,
          "wdrl_confirm":10,
          "public":true,
          "tradable":true,
          "deposit_enabled":true,
          "wdrl_enabled":true,
          "wdrl_fee":"5.00000000",
          "min_deposit":"10.00000000",
          "min_wdrl":"10.00000000",
          "name":"테더",
          "has_tag":false,
          "has_org":false
        }
      ]
    }
  ]
}
```

* Description: Subscribe to get token data.

* request: `init`

* notification: `insert`, `update`

* Rate limit: 2

* Response: Array of [Coin](#coin)

* sort: -

<aside class="notice">
In case of API returns `public` as `false`, you can safely remove corresponding data.
</aside>

## Coin prices

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

# For every token
async def daybit_coin_prices():
    async with Daybit() as daybit:
        pprint(await daybit.coin_prices())

# For specific token
async def daybit_coin_prices_with_sym(sym='ETH'):
    async with Daybit() as daybit:
        pprint(await (daybit.coin_prices / sym)())
```

> Example Response

```python
# If you didn't specify the token
{
  "data": [
    {
      "action":"init",
      "data": [
        {"usdt_price":"1.00000000","sym":"USDT"},
        {"usdt_price":"0.00269932","sym":"AMO"},
        {"usdt_price":"0.09000000","sym":"ADA"},
        ...,
        {"usdt_price":"0.62462722","sym":"ZRX"}
      ]
    }
  ]
}

# If you specified the token
{
  "data": [
    {
      "action":"init",
      "data": [
        {"usdt_price":"267.54000000","sym":"ETH"}
      ]
    }
  ]
}
```

### subtopic: `coin_prices`

* Description: Token to USDT exchange rate for every token. You will get `noficiation` event whenever price of any token gets changed. Please note that only updated token price will be returned.

* request: `init`

* notification: `update`

* Rate limit: 5

* Response: Array of [Coin price](#coin-price)

* sort: -

### subtopic: `coin_prices;<sym>`

* Description: Token to USDT exchange rate for specific token. You will get `noficiation` event whenever price of the specified token gets changed.

* request: `init`

* notification: `update`

* Rate limit: 5

* Response: Array of [Coin price](#coin-price)

* sort: -

## Quote coins

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_quote_coins():
    async with Daybit() as daybit:
        pprint(await daybit.quote_coins())
```

> Example Response

```python
{
    "data": [
        {
            "action":"init",
            "data": [
                {"sym":"USDT"},{"sym":"BTC"},{"sym":"ETH"}
            ]
        }
    ]
}
```

### subtopic: `quote_coins`

* Description: Subscribe to get quote token list.

* request: `init`
* notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: Array of [Quote coin](#quote-coin)

* sort: -

## Markets

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_markets():
    async with Daybit() as daybit:
        pprint(await daybit.markets())
```

> Example Response

```python
{
  "data": [
    {
      "action":"init",
      "data": [
        {
          "tick_price":"0.00000001",
          "tick_levels":5,
          "sellable":true,
          "quote":"BTC",
          "buyable":true,
          "base":"ADA"
        },{
          "tick_price":"0.00000200",
          "tick_levels":5,
          "sellable":false,
          "quote":"BTC",
          "buyable":true,
          "base":"DASH"
        },

        # ...

        {
          "tick_price":"0.00000200",
          "tick_levels":5,
          "sellable":true,
          "quote":"ETH",
          "buyable":true,
          "base":"ETC"
        }
      ]
    }
  ]
}
```

### subtopic: `markets`

* Description: Subscribe to get basic market data.

* request: `init`
* notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: Array of [Market](#market-2)

* sort: -

## Market summary intervals

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_market_summary_intvls():
    async with Daybit() as daybit:
        pprint(await daybit.market_summary_intvls())
```

> Example Response

```python
{
  "data": [
    {
      "action":"init",
      "data": [
        {
          "seconds":30
        },{
          "seconds":60
        },{
          "seconds":360
        },{
          "seconds":720
        },{
          "seconds":1440
        }
      ]
    }
  ]
}
```

### subtopic: `market_summary_intvls`

* Description: Time intervals of market price (unit: second). Return value can be used for input of [Market summaries](#market-summaries).

* request: `init`
* notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: Array of [Market summary interval](#market-summary-interval)

* sort: by `seconds` in `desc`

## Market summaries

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async with Daybit() as daybit:
    intvls = sorted((await daybit.market_summary_intvls()).keys())
    pprint(await (daybit.market_summaries / intvls[0])())
```

> Example Response

```python
{
  "data": [
    {
      "action": "init",
      "data": [
        {
          "volatility":"0.0000",
          "seconds":30,
          "quote_vol":"0.000",
          "quote":"BTC",
          "open":"0.00000049",
          "low":"0.00000049",
          "high":"0.00000049",
          "close":"0.00000049",
          "base_vol":"0.000",
          "base":"AMO"
        },{
          "volatility":"0.0000",
          "seconds":30,
          "quote_vol":"0.000",
          "quote":"BTC",
          "open":"0.083755",
          "low":"0.083755",
          "high":"0.083755",
          "close":"0.083755",
          "base_vol":"0.000",
          "base":"BCH"
        },

        # ...

        {
          "volatility":"0.0000",
          "seconds":30,
          "quote_vol":"0.000",
          "quote":"USDT",
          "open":"285.34",
          "low":"285.34",
          "high":"285.34",
          "close":"285.34",
          "base_vol":"0.000",
          "base":"ETH"
        }
      ]
    }
  ]
}
```

### subtopic: `market_summaries;<seconds>`

* Description: Subscribe to get market price summaries. For the valid `seconds`, please refer [Market summary intervals](#market-summary-intervals).

* request: `init`
* notification: `init`

* Rate limit: 5

* Response: Array of [Market summary](#market-summary)

* sort: -

## Order books

> Example Request

```python
from decimal import Decimal
from pprint import pprint
from pydaybit import Daybit

async def daybit_order_books():
    async with Daybit() as daybit:
        quote = 'USDT'
        base = 'BTC'
        price_intvl = Decimal((await daybit.markets())['{}-{}'.format(quote, base)]['tick_price']) * 10
        pprint(await (daybit.order_books / quote / base / price_intvl)())
```

> Example Response

```python
{
  '6475.00000000-6480.00000000': {
    'base': 'BTC',
    'buy_vol': '0.02086000',
    'intvl': '5.00000000',
    'max_price': '6480.00000000',
    'min_price': '6475.00000000',
    'quote': 'USDT',
    'sell_vol': '0.00000000'
  },
 '6480.00000000-6485.00000000': {
   'base': 'BTC',
    'buy_vol': '0.22132000',
    'intvl': '5.00000000',
    'max_price': '6485.00000000',
    'min_price': '6480.00000000',
    'quote': 'USDT',
    'sell_vol': '0.00000000'
  },

  # ...

 '7640.00000000-7645.00000000': {
    'base': 'BTC',
    'buy_vol': '0.00000000',
    'intvl': '5.00000000',
    'max_price': '7645.00000000',
    'min_price': '7640.00000000',
    'quote': 'USDT',
    'sell_vol': '0.01060000'
  }
}
```

### subtopic: `order_books;<quote>;<base>;<price_intvl>`

* Description: Order book by unit price. In the response, there's no `id` so you need to compare it with `min_price` or `max_price` to identify the order book. (min_price, max_price] is the range for selling and [min_price, max_price) is the range for buying. In case of `sell_vol` and `buy_vol` exist at the same time, please make sure that they are located in difference range by above range conditions.

* request: `init`
* notification: `init`, `upsert`

* Rate limit: 5

* Response: Array of [Order book](#order-book)

* sort: by `min_price` in `desc`

## Price history intervals

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_price_history_intvls():
    async with Daybit() as daybit:
        pprint(await daybit.price_history_intvls())
```

> Example Response

```python
{
  60: {
    'seconds': 60
  },
  180: {
    'seconds': 180
  },
  300: {
    'seconds': 300
  },
  600: {
    'seconds': 600
  },
  900: {
    'seconds': 900
  },
  1800: {
    'seconds': 1800
  },
  3600: {
    'seconds': 3600
  },
  7200: {
    'seconds': 7200
  },
  14400: {
    'seconds': 14400
  },
  21600: {
    'seconds': 21600
  },
  86400: {
    'seconds': 86400
  }
}
```

### subtopic: `price_history_intvls`

* Description: Time intervals of past market price data (unit: second). Return value can be used for input of [Price histories](#price-histories).

* `request`: `init`
* `notification`: - (nothing to be reported)

* Rate limit: 2

* Response: Array of [Price history interval](#price-history-interval)

* sort: `seconds` in `asc`

## Price histories

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_price_histories():
    async with Daybit() as daybit:
        quote = 'USDT'
        base = 'BTC'
        intvl = sorted((await daybit.price_history_intvls()).keys())[0]
        pprint(
            await (daybit.price_histories / quote / base / intvl)(from_time=int(time.time() * 1000 - intvl * 10 * 1000), to_time=int(time.time() * 1000)))
```

> Example Response

```python
{
  'USDT-BTC-60-1537351080000': {
    'base': 'BTC',
    'base_vol': '0',
    'close': '6833.00000000',
    'end_time': 1537351140000,
    'high': '6833.00000000',
    'intvl': 60,
    'low': '6833.00000000',
    'open': '6833.00000000',
    'quote': 'USDT',
    'quote_vol': '0',
    'start_time': 1537351080000
  },
  'USDT-BTC-60-1537351140000': {
    'base': 'BTC',
    'base_vol': '0',
    'close': '6833.00000000',
    'end_time': 1537351200000,
    'high': '6833.00000000',
    'intvl': 60,
    'low': '6833.00000000',
    'open': '6833.00000000',
    'quote': 'USDT',
    'quote_vol': '0',
    'start_time': 1537351140000
  },

  # ...

  'USDT-BTC-60-1537351680000': {
    'base': 'BTC',
    'base_vol': '0',
    'close': '6833.00000000',
    'end_time': 1537351740000,
    'high': '6833.00000000',
    'intvl': 60,
    'low': '6833.00000000',
    'open': '6833.00000000',
    'quote': 'USDT',
    'quote_vol': '0',
    'start_time': 1537351680000
  }
}
```

### subtopic: `price_histories;<quote>;<base>;<intvl>`

* Description: Past market price data. For the valid `intvl`, please refer [Price history intervals](#price-history-intervals).

* request: `upsert`
* notification: `insert`, `update`

* Rate limit: 20

* Response: Array of [Price history](#price-history)

* sort: -

### arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`from_time` | unix_timestamp | Required | Start time of the price history range.
`to_time` | unix_timestamp | Required | End time of the price history range.

## Trades

> Example Request

```python
async def daybit_trades():
    async with Daybit() as daybit:
        quote = 'USDT'
        base = 'BTC'
        pprint(await (daybit.trades / quote / base)(num_trades=10))
```

> Example Response

```python
{
  40810689: {
    'base': 'BTC',
    'base_amount': '0.00266000',
    'exec_at': 1537350775639,
    'id': 40810689,
    'price': '6833.00000000',
    'quote': 'USDT',
    'quote_amount': '18.17578000',
    'taker_sold': True
  },
  40810690: {
    'base': 'BTC',
    'base_amount': '0.00044000',
    'exec_at': 1537350775677,
    'id': 40810690,
    'price': '6833.00000000',
    'quote': 'USDT',
    'quote_amount': '3.00652000',
    'taker_sold': True
  },

  # ...

  40811438: {
    'base': 'BTC',
    'base_amount': '0.00458000',
    'exec_at': 1537352156105,
    'id': 40811438,
    'price': '6833.00000000',
    'quote': 'USDT',
    'quote_amount': '31.29514000',
    'taker_sold': True
  }
}
```

### subtopic: `trades;<quote>;<base>`

* Description: Subscribe to get trade data per market. API doesn't support for getting past trade data.

* request: `init`
* notification: `insert`

* Rate limit: 5

* Response: Array of [Trade](#trade)

* sort: by `exec_at` in `desc`

### arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`num_trades` | integer | Required | Trade count for retrieving.

# Private data

## My users

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_my_users():
    async with Daybit() as daybit:
        pprint(await daybit.my_users())
```

> Example Response

```python
[
  {
    'maker_fee_rate': '0.001000',
    'one_day_wdrl_usdt_limit': '5000.00000000',
    'pay_fee_with_day': True,
    'taker_fee_rate': '0.001000'
  }
]
```

### subtopic: `my_users`

* Description: Subscribe to get information of my account.

* request: `init`
* notification: `init`

* Rate limit: 2

* Response: Array of [User](#user)

* sort: -

## My assets

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_my_assets():
    async with Daybit() as daybit:
        pprint(await daybit.my_assets())
```

> Example Response

```python
{
  'ADA': {
    'available': '700000.000000000000000000',
    'coin': 'ADA',
    'investment_usdt': '0.000000000000000000',
    'reserved': '0.000000000000000000',
    'total': '700000.000000000000000000'
  },
  'AMO': {
    'available': '20000000.000000000000000000',
    'coin': 'AMO',
    'investment_usdt': '0.000000000000000000',
    'reserved': '0.000000000000000000',
    'total': '20000000.000000000000000000'
  },

  # ...

  'ZRX': {
    'available': '90000.000000000000000000',
    'coin': 'ZRX',
    'investment_usdt': '0.000000000000000000',
    'reserved': '0.000000000000000000',
    'total': '90000.000000000000000000'
  }
}
```

### subtopic: `my_assets`

* Description: Subscribe to get information of my assets.

* request: `init`
* notification: `insert`, `update`

* Rate limit: 5

* Response: Array of [Asset](#asset)

* sort: -

# Trade

## My orders

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_my_orders():
    async with Daybit() as daybit:
        pprint(await daybit.my_orders(statuses='placed'))
```

> Example Response

```python
{
  81133319: {
    'amount': '0.00020000',
    'base': 'BTC',
    'cancel_reason': 'user',
    'close_type': 'canceled',
    'closed_at': 1537339108567,
    'cond_arg1': None,
    'cond_arg2': None,
    'cond_type': 'none',
    'filled': '0.00000000',
    'filled_quote': '0.00000000',
    'id': 81133319,
    'placed_amount': '0.00020000',
    'placed_at': 1537339108540,
    'price': '6040.50000000',
    'quote': 'USDT',
    'received_at': 1537339108516,
    'role': 'both',
    'sell': True,
    'status': 'closed',
    'unfilled': '0.00020000'
  },
  82622825: {
    'amount': '0.00020000',
    'base': 'BTC',
    'cancel_reason': 'user',
    'close_type': 'canceled',
    'closed_at': 1537347005236,
    'cond_arg1': None,
    'cond_arg2': None,
    'cond_type': 'none',
    'filled': '0.00000000',
    'filled_quote': '0.00000000',
    'id': 82622825,
    'placed_amount': '0.00020000',
    'placed_at': 1537346949166,
    'price': '7091.00000000',
    'quote': 'USDT',
    'received_at': 1537346949161,
    'role': 'both',
    'sell': False,
    'status': 'closed',
    'unfilled': '0.00020000'
  },

  # ...

  82722801: {
    'amount': '0.00020000',
    'base': 'BTC',
    'cancel_reason': None,
    'close_type': 'filled',
    'closed_at': 1537347509954,
    'cond_arg1': None,
    'cond_arg2': None,
    'cond_type': 'none',
    'filled': '0.00020000',
    'filled_quote': '1.41820000',
    'id': 82722801,
    'placed_amount': '0.00020000',
    'placed_at': 1537347483279,
    'price': '7091.00000000',
    'quote': 'USDT',
    'received_at': 1537347483275,
    'role': 'both',
    'sell': False,
    'status': 'closed',
    'unfilled': '0.00000000'
  }
}
```

### subtopic: `my_orders`

* Description: Subscribe to get information of my orders.

* request: `upsert`
* notification: `insert`, `update`

* Rate limit: 5

* Response: Array of [Order](#order-2)

* sort: by `id` in `desc`

### arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`quote` | string | Optional | Quote token symbol. ex) "BTC"
`base` | string | Optional | Base token symbol. ex) "ETH"
`to_id` | integer | Optional | Get my orders that are `id` is smaller than `to_id`.
`size` | integer | Optional | Order count for retrieving.
`sell` | boolean | Optional | `true` for selling order and `false` for buying order.
`statuses` | csv | Optional | Conditions of `status` to retrieve. csv form of `received`, `placed`, `completed`, and `canceled`. (ex, `"received, placed"`)

## My trades

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_my_trades():
    async with Daybit() as daybit:
        pprint(await daybit.my_trades(sell=True, quote='USDT'))
```

> Example Response

```python
{
  40807547: {
    'base': 'BTC',
    'base_amount': '0.00020000',
    'coin_fee': '0.00141820',
    'day_fee': '0.00000000',
    'exec_at': 1537347408941,
    'id': 40807547,
    'order_id': 82707048,
    'price': '7091.00000000',
    'quote': 'USDT',
    'quote_amount': '1.41820000',
    'sell': True,
    'taker_sold': True
  },
  40807562: {
    'base': 'BTC',
    'base_amount': '0.00020000',
    'coin_fee': '0.00000001',
    'day_fee': '0.03977098',
    'exec_at': 1537347414699,
    'id': 40807562,
    'order_id': 82707467,
    'price': '7091.00000000',
    'quote': 'USDT',
    'quote_amount': '1.41820000',
    'sell': True,
    'taker_sold': True
  },

  # ...

  40807691: {
    'base': 'BTC',
    'base_amount': '0.00020000',
    'coin_fee': '0.00000000',
    'day_fee': '0.04415411',
    'exec_at': 1537347509915,
    'id': 40807691,
    'order_id': 82722801,
    'price': '7091.00000000',
    'quote': 'USDT',
    'quote_amount': '1.41820000',
    'sell': False,
    'taker_sold': True
  }
}
```

### subtopic: `my_trades`

* Description: Subscribe to get informatino of my trade data.

* request: `upsert`
* notification: `insert`

* Rate limit: 5

* Response: Array of [Trade](#trade)

* sort: by `id` in `desc`

### arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`quote` | string | Optional | Quote token symbol. ex) "BTC"
`base` | string | Optional | Base token symbol. ex) "ETH"
`to_id` | integer | Optional | Get my orders that are `id` is smaller than `to_id`.
`size` | integer | Optional | Order count for retrieving.
`sell` | boolean | Optional | `true` for selling order and `false` for buying order.

# Transaction

## My transactions

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_my_tx_summaries():
    async with Daybit() as daybit:
        pprint(await daybit.my_tx_summaries(type='deposit'))
        pprint(await daybit.my_tx_summaries(type='wdrl'))
```

> Example Response

```python
**TODO: NEED RESPONSE EXAMPLE**
```

### subtopic: `my_tx_summaries`

* Description: Subscribe to get my transaction summaries.

* request: `upsert`
* notification: `insert`, `update`

* Rate limit: 5

* Response: Array of [Deposit](#deposit)

* sort: by `id` in `desc`

### arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`type` | string | Required | One of `deposit` or `wdrl` per your purpose.

## My airdrops

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_my_my_airdrops():
    async with Daybit() as daybit:
        pprint(await daybit.my_airdrops())
```

> Example Response

```python
**TODO: NEED RESPONSE EXAMPLE**
```

### subtopic: `my_airdrops`

* Description: Subscribe to get list of my airdrops.

* request: `upsert`
* notification: `insert`

* sort: by `id` in `desc`

* Response: Array of [Airdrop](#airdrop)

### arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`to_id` | integer | Optional | Get my orders that are `id` is smaller than `to_id`.
`size` | integer | Optional | Order count for retrieving.
