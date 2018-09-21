# **Topic: Subscription**

This section explains how you could implement various features of the Exchange API. If you subscribed to certain `/subscription:<sub_topic>`, you will get notification from the server when relevant modification happens.

* Topic: `/subscription:<sub_topic>`

* Event: `request` (push) or `notification` (pull). [Message](https://hexdocs.pm/phoenix/Phoenix.Socket.Message.html) transported from client and server have `request` and `notification` events, respectively. When you subscribe to the event with `request` event, you will get either `init` or `upsert` action response from the API. After that, you would get one of `insert`, `update`, `upsert`, or `delete` from the API with `notification` event. For more information of actions, please look [Action](#action).

* Rate limit: Limit of calls for every second. Only applicable for `request`.

## Action

Response holds `action` which helps you to understand how to handle the response.

* `init` : Dump all previous data and initialize everything with most recent data.
* `insert` : Add data to data set, as most recent data.
* `update` : Search in data set and replace if it was found.
* `upsert` : Search in data set and replace if it was found, or insert if there's no matching data.
* `delete` : Search in data set and remove if it was found.

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
  'ADA': {
    'deposit_confirm': 3,
    'deposit_enabled': True,
    'has_org': False,
    'has_tag': False,
    'min_deposit': '100.00000000',
    'min_wdrl': '100.00000000',
    'name': 'Ada',
    'native_decimal_places': 8,
    'public': True,
    'sym': 'ADA',
    'tick_amount': '2.00000000',
    'tradable': False,
    'wdrl_confirm': 10,
    'wdrl_enabled': True,
    'wdrl_fee': '50.00000000'
  },

  # ...       
      
  'ZRX': {
    'deposit_confirm': 3,
    'deposit_enabled': True,
    'has_org': False,
    'has_tag': False,
    'min_deposit': '13.00000000',
    'min_wdrl': '13.00000000',
    'name': '0x Protocol',
    'native_decimal_places': 4,
    'public': True,
    'sym': 'ZRX',
    'tick_amount': '0.20000000',
    'tradable': True,
    'wdrl_confirm': 10,
    'wdrl_enabled': True,
    'wdrl_fee': '6.30000000'
  }
}
```

* Description: Subscribe to get token data.

* request: `init`

* notification: `insert`, `update`

* Rate limit: 2

* Response: [Coin](#coin)

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
  'ADA': {
    'sym': 'ADA',
    'usdt_price': '0.09000000'
  },

  # ...

  'ZRX': {
    'sym': 'ZRX',
    'usdt_price': '0.67047870'
  }
}

# If you specified the token
{
  'ETH': {
    'sym': 'ETH',
    'usdt_price': '291.88000000'
  }
}
```

### subtopic: `coin_prices`

* Description: Token to USDT exchange rate for every token. You will get `noficiation` event whenever price of any token gets changed. Please note that only updated token price will be returned.

* request: `init`

* notification: `update`

* Rate limit: 5

* Response: [Coin price](#coin-price)

* sort: -

### subtopic: `coin_prices;<sym>`

* Description: Token to USDT exchange rate for specific token. You will get `noficiation` event whenever price of the specified token gets changed.

* request: `init`

* notification: `update`

* Rate limit: 5

* Response: [Coin price](#coin-price)

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
  'BTC': {
    'sym': 'BTC'
  },
  'ETH': {
    'sym': 'ETH'
  },
  'USDT': {
    'sym': 'USDT'
  }
}
```

### subtopic: `quote_coins`

* Description: Subscribe to get quote token list.

* request: `init`
* notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: [Quote coin](#quote-coin)

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
  'BTC-ADA': {
    'base': 'ADA',
    'buyable': True,
    'quote': 'BTC',
    'sellable': True,
    'tick_levels': 5,
    'tick_price': '0.00000001'
  },

  # ...
             
  'USDT-ETH': {
    'base': 'ETH',
    'buyable': True,
    'quote': 'USDT',
    'sellable': True,
    'tick_levels': 5,
    'tick_price': '0.02000000'
  }
}
```

### subtopic: `markets`

* Description: Subscribe to get basic market data.

* request: `init`
* notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: [Market](#market-2)

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
  30: {
    'seconds': 30
  },

  # ...

  1440: {
    'seconds': 1440
  }
}
```

### subtopic: `market_summary_intvls`

* Description: Time intervals of market price (unit: second). Return value can be used for input of [Market summaries](#market-summaries).

* request: `init`
* notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: [Market summary interval](#market-summary-interval)

* sort: by `seconds` in `desc`

## Market summaries

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_market_summaries():
    async with Daybit() as daybit:
        intvls = sorted((await daybit.market_summary_intvls()).keys())
        pprint(await (daybit.market_summaries / intvls[0])())
```

> Example Response

```python
{
  'BTC-AMO': {
    'base': 'AMO',
    'base_vol': '141900.000',
    'close': '0.00000046',
    'high': '0.00000046',
    'low': '0.00000046',
    'open': '0.00000046',
    'quote': 'BTC',
    'quote_vol': '0.065',
    'seconds': 30,
    'volatility': '0.0000'
  },

  # ...

  'USDT-ETH': {
    'base': 'ETH',
    'base_vol': '0.275',
    'close': '291.82',
    'high': '291.82',
    'low': '291.82',
    'open': '291.82',
    'quote': 'USDT',
    'quote_vol': '80.251',
    'seconds': 30,
    'volatility': '0.0000'
  }
}
```

### subtopic: `market_summaries;<seconds>`

* Description: Subscribe to get market summaries. For the valid `seconds`, please refer [Market summary intervals](#market-summary-intervals).

* request: `init`
* notification: `init`

* Rate limit: 5

* Response: [Market summary](#market-summary)

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

* Response: [Order book](#order-book)

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
  
  # ...

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

* Response: [Price history interval](#price-history-interval)

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

* Response: [Price history](#price-history)

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

* Response: [Trade](#trade)

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

* Response: [User](#user)

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

* Response: [Asset](#asset)

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

* Response: [Order](#order-2)

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

* Response: [Trade](#trade)

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

* Response: [Deposit](#deposit)

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

* Response: [Airdrop](#airdrop)

### arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`to_id` | integer | Optional | Get my orders that are `id` is smaller than `to_id`.
`size` | integer | Optional | Order count for retrieving.
