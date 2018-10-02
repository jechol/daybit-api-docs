# Pydaybit
Pydaybit is an API wrapper for Daybit exchange written in Python.


## **Installation**
  
  ```shell
      $ git clone https://github.com/daybit-exchange/pydaybit
      $ pip install -e pydaybit
  ```

## get_server_time()

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_get_server_time():
    async with Daybit() as daybit:
        pprint(await daybit.get_server_time())
```

> Example Response

```python
1537418605426 # unix_timestamp
```

* Descrition: Get current server time of Daybit backend server in unix timestamp format.

* Rate limit: 10


## create_order()

> Example Request

```python
from decimal import Decimal
from pprint import pprint
from pydaybit import Daybit

async def current_price(daybit, quote, base):
    summary_intvl = sorted((await daybit.market_summary_intvls()).keys())[0]
    price = (await (daybit.market_summaries / summary_intvl)())['{}-{}'.format(quote, base)]['close']
    return Decimal(price)

async def daybit_create_order_sell():
    async with Daybit() as daybit:
        quote = 'USDT'
        base = 'BTC'

        price = await current_price(daybit, quote, base)
        tick_amount = Decimal((await daybit.coins())[base]['tick_amount'])

        response = await daybit.create_order(
            sell=True, # False for buying
            role='both',
            quote=quote,
            base=base,
            price=price,
            amount=tick_amount * 10,
            cond_type='none',
        )
        pprint(response)
```

> Example Response

```python
{
  'amount': '0.00200000',
  'base': 'BTC',
  'cancel_reason': None,
  'close_type': None,
  'closed_at': None,
  'cond_arg1': None,
  'cond_arg2': None,
  'cond_type': 'none',
  'filled': '0.00000000',
  'filled_quote': '0.00000000',
  'id': 53026865,
  'placed_amount': '0.00200000',
  'placed_at': 1537419310682,
  'price': '6961.00000000',
  'quote': 'USDT',
  'received_at': 1537419310639,
  'role': 'both',
  'sell': True,
  'status': 'placed',
  'unfilled': '0.00200000'
}
```

* Description: Create order to sell or buy token.

* Rate limit: 100

* Response: [Order](#order-2)

### arguments

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | `true` for selling and `false` for buying.
`quote` | string | Required | Quote token symbol. ex) "BTC"
`base` | string | Required | Base token symbol. ex) "ETH"
`price` | decimal | Required | Asking price.
`amount` | decimal | Required | Order amount.
`role` | string | Required | `"both"`, `"maker_only"`, and `"taker_only"` are valid.
`cond_type` | string | Required | Conditional types of the order. <ul><li>When `role = "both"`: `"none"`, `"le"`(less or equal than), `"ge"`(greate or equal than), `"fall_from_top"`, and `"rise_from_bottom"` are valid.</li><li>When `role = "maker_only" or "taker_only"`: only `"none"` is valid.
`cond_value` | decimal | Optional | Conditional price of the order. This is required only when `cond_type` is not `"none"`.</li></ul>


## cancel_order()

> Example Request

```python
from pydaybit import Daybit

async def daybit_cancel_order():
    async with Daybit() as daybit:
        await daybit.cancel_order(12345678)
```

> Example Response

```python
{
  'amount': '0.00200000',
  'base': 'BTC',
  'cancel_reason': 'user',
  'close_type': 'canceled',
  'closed_at': 1537421465336,
  'cond_arg1': None,
  'cond_arg2': None,
  'cond_type': 'none',
  'filled': '0.00000000',
  'filled_quote': '0.00000000',
  'id': 53216861,
  'placed_amount': '0.00200000',
  'placed_at': 1537421443079,
  'price': '6595.00000000',
  'quote': 'USDT',
  'received_at': 1537421443074,
  'role': 'both',
  'sell': False,
  'status': 'closed',
  'unfilled': '0.00200000'
}
```

* Description: Cancel placed order. You must pass valid `id` to cancel the order.

* Rate limit: 100

* Response: [Order](#order-2)

### arguments

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`order_id` | integer | Required | id of order supposed to be canceled.


## cancel_orders()

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_cancel_orders():
    async with Daybit() as daybit:
        my_orders = await daybit.my_orders()
        open_orders = ([my_orders[key]['id'] for key in my_orders if my_orders[key]['status'] == 'placed'])
        pprint(open_orders)
        pprint(await daybit.cancel_orders(open_orders))
```

> Example Response

```python
{
  'num_canceled_orders': 5
}
```

* Description: Cancel multiple orders. If one or more of `order_ids` are invalid, the API simply ignores it and cancels only valid ones. You can check number of canceled ids from `num_canceled_orders` in response.

* Rate limit: 5

### arguments

Parameter | Type | Required | Description
----------|------|----------|-----------|
`order_ids` | array | Required | ids of order supposed to be canceled.

### response

Field | Description
---------|------------
`num_canceled_orders` | Number of successfully canceled orders.

<aside class="notice">
 Backend server is expecting the orders in CSV format (ex, "1,2,3"). However you can use array of integer if you are using Pydaybit.
 </aside>


## cancel_all_my_orders()

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_cancel_all_my_orders():
    async with Daybit() as daybit:
        response = await daybit.cancel_all_my_orders()
        pprint(response)
```

> Example Response

```python
{
  'num_canceled_orders': 2
}
```

* Description: Cancel all my orders (both sell and buy orders).

* Rate limit: 5

### response

Field | Description
---------|------------
`num_canceled_orders` | Number of successfully canceled orders.


## create_wdrl()

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_create_wdrl():
    async with Daybit() as daybit:
        pprint(await daybit.create_wdrl(coin='ADA', to_addr='fake_address', amount='100'))
```

> Example Response

```python
{
  'amount': '100.00000000',
  'coin': 'ADA',
  'completed_at': None,
  'fee': '50.00000000',
  'id': 47291,
  'requested_at': 1537419011126,
  'to_addr': 'fake_address',
  'to_org': None,
  'to_tag': None,
  'tx_created_at': None,
  'txid': None,
  'usdt_amount': '9.00000000'
}
```

* Description: Request withdraw to `to_addr` for `amount` of `coin`.

* Rate limit: 10

* Response: [Wdrl](#wdrl-2)

### arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`coin` | string | Required | Withdraw token.
`to_addr` | string | Required | Withdraw receiving address.
`to_tag` | string | Optional | Withdraw tag. Used by `XRP` and so on.
`amount` | decimal | Required | Amount to withdraw.


## **Subscriptions**


## coins()

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



## coin_prices()

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

### coin_prices()

* Description: Token to USDT exchange rate for every token. You will get `noficiation` event whenever price of any token gets changed. Please note that only updated token price will be returned.

* request: `init`

* notification: `update`

* Rate limit: 5

* Response: [Coin price](#coin-price)

* sort: -

### (coin_prices / `<sym>`)()

* Description: Token to USDT exchange rate for specific token. You will get `noficiation` event whenever price of the specified token gets changed.

* request: `init`

* notification: `update`

* Rate limit: 5

* Response: [Coin price](#coin-price)

* sort: -


## quote_coins()

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

* Description: Subscribe to get quote token list.

* request: `init`
* notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: [Quote coin](#quote-coin)

* sort: -

## markets()

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

* Description: Subscribe to get basic market data.

* request: `init`

* notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: [Market](#market-2)

* sort: -


## market_summary_intvls()

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

* Description: Time intervals of market price (unit: second). Return value can be used for input of [Market summaries](#market-summaries).

* request: `init`
* notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: [Market summary interval](#market-summary-interval)

* sort: by `seconds` in `desc`

## market_summaries()

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
`market_summaries/<seconds>`

* Description: Subscribe to get market summaries. For the valid `seconds`, please refer [Market summary intervals](#market-summary-intervals).

* request: `init`
* notification: `init`

* Rate limit: 5

* Response: [Market summary](#market-summary)

* sort: -


## order_books()

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

`order_books/<quote>/<base>/<price_intvl>`

* Description: Order book by unit price. In the response, there's no `id` so you need to compare it with `min_price` or `max_price` to identify the order book. (min_price, max_price] is the range for selling and [min_price, max_price) is the range for buying. In case of `sell_vol` and `buy_vol` exist at the same time, please make sure that they are located in difference range by above range conditions.

* request: `init`
* notification: `init`, `upsert`

* Rate limit: 5

* Response: [Order book](#order-book)

* sort: by `min_price` in `desc`


## price_history_intvls()

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

* Description: Time intervals of past market price data (unit: second). Return value can be used for input of [Price histories](#price-histories).

* `request`: `init`
* `notification`: - (nothing to be reported)

* Rate limit: 2

* Response: [Price history interval](#price-history-interval)

* sort: `seconds` in `asc`


## price_histories()

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

`price_histories;<quote>/<base>/<intvl>`

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



## trades()

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit


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

`trades/<quote>/<base>`

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


## my_users()

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

* Description: Subscribe to get information of my account.

* request: `init`
* notification: `init`

* Rate limit: 2

* Response: [User](#user)

* sort: -


## my_assets()

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


* Description: Subscribe to get information of my assets.

* request: `init`
* notification: `insert`, `update`

* Rate limit: 5

* Response: [Asset](#asset)

* sort: -


## my_orders()

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



## my_trades()

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

* Description: Subscribe to get information of my trade data.

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


## my_tx_summaries()

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
# response of daybit.my_tx_summaries(type='deposit')
{
  47283: {
    'amount': '1.2977',
    'coin': 'LTC',
    'completed_at': 1537417829032,
    'confirm': 3847,
    'created_at': 1537417829029,
    'deposit_status': 'completed',
    'id': 47283,
    'req_confirm': 3,
    'tx_link_url': 'https://live.blockcypher.com/ltc/tx/a809a19d194d48967f1c8b6aa1dae33d412ece2570fe8d9b751d63a703e15183',
    'txid': 'a809a19d194d48967f1c8b6aa1dae33d412ece2570fe8d9b751d63a703e15183',
    'type': 'deposit',
    'wdrl_status': None,
    'wdrl_to_addr': None,
    'wdrl_to_org': None,
    'wdrl_to_tag': None
  },

  # ...
  
  47287: {
    'amount': '0.01448502',
    'coin': 'BTC',
    'completed_at': 1537417829091,
    'confirm': 955,
    'created_at': 1537417829087,
    'deposit_status': 'completed',
    'id': 47287,
    'req_confirm': 3,
    'tx_link_url': 'https://live.blockcypher.com/btc/tx/af9648f4c1622b3872649019d4391248d9c93bdd9630e16128431d475d2263a4',
    'txid': 'af9648f4c1622b3872649019d4391248d9c93bdd9630e16128431d475d2263a4',
    'type': 'deposit',
    'wdrl_status': None,
    'wdrl_to_addr': None,
    'wdrl_to_org': None,
    'wdrl_to_tag': None
  }
}

# example of daybit.my_tx_summaries(type='wdrl')
{
  47291: {
    'amount': '100.00000000',
    'coin': 'ADA',
    'completed_at': None,
    'confirm': 0,
    'created_at': 1537419011126,
    'deposit_status': None,
    'id': 47291,
    'req_confirm': 10,
    'tx_link_url': None,
    'txid': None,
    'type': 'wdrl',
    'wdrl_status': 'queued',
    'wdrl_to_addr': '0x2b7cd7f27d21da93395a4c19aea80f8467f93596',
    'wdrl_to_org': None,
    'wdrl_to_tag': None
  }
}
```

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


## my_airdrops()

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
