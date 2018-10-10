# **Pydaybit**
Pydaybit is an API wrapper for Daybit exchange written in Python. The code is available in [Pydaybit Repository](https://github.com/daybit-exchange/pydaybit). 


## **Disclaimer**
USE THE SOFTWARE AT YOUR OWN RISK. THE AUTHORS AND ALL AFFILIATES ASSUME NO RESPONSIBILITY FOR YOUR TRADING RESULTS.


## **Installation**
  
> Pydaybit installation

```shell
$ git clone https://github.com/daybit-exchange/pydaybit
$ pip install -e pydaybit
```
  
For the installation, please look right column.

## **API Key Pair**

You need to generate a API key pair to use Pydaybit. Please refer [Authorization](#authorization).

### Environment Variables

> bash example

```shell
# ~/.bash_profile
export DAYBIT_API_KEY="YOUR_OWN_API_KEY"
export DAYBIT_API_SECRET="YOUR_WON_API_SECRET"
```

You can set generated key pair as environment variable.

* `DAYBIT_API_KEY`: Generated API KEY
* `DAYBIT_API_SECRET`: Generated API SECRET


### Without Environment Variables

> without environment settings

```python
import asyncio

from pydaybit import Daybit, PARAM_API_KEY, PARAM_API_SECRET


async def daybit_example():
    async with Daybit(params={PARAM_API_KEY: "YOUR_OWN_API_KEY",
                              PARAM_API_SECRET: "YOUR_OWR_API_SECRET"}) as daybit:
        pass


asyncio.get_event_loop().run_until_complete(daybit_example())
```

For using the key pair without settings them in environment variable, please refer the example.

## get_server_time()

> Example Request

```python
from pprint import pprint
from pydaybit import Daybit

async def daybit_get_server_time():
    async with Daybit() as daybit:
        pprint(await daybit.get_server_time())

asyncio.get_event_loop().run_until_complete(daybit_get_server_time())
```

> Example Response

```python
1537418605426 # unix_timestamp
```

Get current time of Daybit in unix timestamp format.

* Topic: `/api`

* Event: `get_server_time`
 
* Rate limit: 10


## create_order()

> Example Request

```python
import asyncio
from contextlib import suppress
from decimal import Decimal
from pprint import pprint

from pydaybit import Daybit
from pydaybit.exceptions import OrderAlreadyClosed


async def current_price(daybit, quote, base):
    summary_intvl = sorted((await daybit.market_summary_intvls()).keys())[0]
    price = (await (daybit.market_summaries / summary_intvl)())['{}-{}'.format(quote, base)]['close']
    return Decimal(price)


async def daybit_create_order_sell():
    async with Daybit() as daybit:
        quote = 'USDT'
        base = 'BTC'

        tick_price = Decimal((await daybit.markets())['{}-{}'.format(quote, base)]['tick_price'])
        tick_amount = Decimal((await daybit.coins())[base]['tick_amount'])

        # amount * price should be greater than 10 USDT.
        price = ((await current_price(daybit, quote, base)) * Decimal(1.2)).quantize(tick_price)
        amount = (Decimal(10.5) / price).quantize(tick_amount)

        response = await daybit.create_order(
            sell=True,
            role='both',
            quote=quote,
            base=base,
            price=price,
            amount=amount,
            cond_type='none',
        )
        pprint(response)

        with suppress(OrderAlreadyClosed):
            await daybit.cancel_order(response['id'])


asyncio.get_event_loop().run_until_complete(daybit_create_order_sell())
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

Create a order to sell or buy coin. There are five types of orders you can request to the server. For detail of orders, please refer followings - [Limit order](#limit-order), [Taker Order](#taker-order), [Maker Order](#maker-order), [Stop Limit Order](#stop-limit-order), and [Trailling Stop Order](#trailing-stop-order).
 
잘못된 주문(void) 의 조건은,
- reject 된 주문 (에러를 발생시키는 주문을 넣은 경우)
- 유저의 주문 취소
- 자전 거래로 인한 취소
- 오래된 주문 자동 취소

`sell`, `quote`, `base`, `amount`, `role`, `cond_type` are always required.

* Topic: `/api`

* Event: `create_order`

* Rate limit: 100

* Response: [Order](#order-2)

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | `true` for selling and `false` for buying.
`quote` | string | Required | Quote coin symbol. ex) "BTC"
`base` | string | Required | Base coin symbol. ex) "ETH"
`amount` | decimal | Required | Order amount.
`role` | string | Required | Role of order.
`cond_type` | string | Required | Conditional types of the order.
`price` | decimal | Optional | Asking price.
`cond_arg1` | decimal | Optional | First conditional argument of the order.
`cond_arg2` | decimal | Optional | Second conditional argument of the order.

### Limit Order

> Example Limit Order

```python
await daybit.create_order(
    sell=True, # True for selling, False for buying.
    role='both',
    quote=quote,
    base=base,
    price=price,
    amount=amount,
    cond_type='none',
)
```

If the order was placed, it is regarded as a [taker order](#taker-order). and if remain amount of the order exists, it is regarded as a [maker order](#maker-order).

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | `True` for selling and `False` for buying.
`role` | string | Required | `"both"`
`quote` | string | Required | Quote coin symbol. ex) "BTC"
`base` | string | Required | Base coin symbol. ex) "ETH"
`price` | decimal | Required | Asking price in terms of `price` = `base` / `quote`.
`amount` | decimal | Required | Required amount of `quote`.
`cond_type` | string | Required | `"none"`

<aside class="notice">
  Constraint: <code>amount</code> * <code>price in USD</code> ≥ 10.0 <code>USD</code>.
</aside>

### Taker Order

> Example Taker Order

```python
await daybit.create_order(
    sell=True, # True for selling, False for buying.
    role='taker_only',
    quote=quote,
    base=base,
    price=price,
    amount=amount,
    cond_type='none',
)
```

If the order was placed, before part or all of your orders are going on the order book, these are called as "taker". These trades are called as "taker" because it is "taking" the volume in the order book. This order is taking only volumes in the order book.

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | `True` for selling and `False` for buying.
`role` | string | Required | `"taker_only"`
`quote` | string | Required | Quote coin symbol.
`base` | string | Required | Base coin symbol.
`price` | decimal | Required | Asking price in terms of `price` = `base` / `quote`.
`amount` | decimal | Required | Required amount of `quote`.
`cond_type` | string | Required | `"none"`

<aside class="notice">
  Constraint:  <code>amount</code> * <code>price in USD</code> ≥ 10.0 <code>USD</code>.
</aside>

### Maker Order

> Example Maker Order

```python
await daybit.create_order(
    sell=True, # True for selling, False for buying.
    role='maker_only',
    quote=quote,
    base=base,
    price=price,
    amount=amount,
    cond_type='none',
)
```

If the order was placed, after part or all of your orders filled the order book, these are called as "maker". This order is only valid when it fills the volume in the order book.

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | `True` for selling and `False` for buying.
`role` | string | Required | `"maker_only"`
`quote` | string | Required | Quote coin symbol.
`base` | string | Required | Base coin symbol.
`price` | decimal | Required | Asking price in terms of `price` = `base` / `quote`.
`amount` | decimal | Required | Required amount of `quote`.
`cond_type` | string | Required | `"none"`

<aside class="notice">
  Constraint: <code>amount</code> * <code>price in USD</code> ≥ 10.0 <code>USD</code>.
</aside>

### Stop Limit Order

> Example Stop Limit Order

```python
await daybit.create_order(
    sell=True, # True for selling, False for buying.
    role='both',
    quote=quote,
    base=base,
    price=price,
    amount=amount,
    cond_type='ge', # cond_type could be 'ge' or 'le'.
    cond_arg1=condition_price
)
```

When current price is equal or greater/less than `cond_arg1`, it places Limit Order.

* If you sent request with `cond_type` = `"le"`, it places Limit Order for the price of `price` when it becomes `current_price` ≤ `conditional_price`(= `cond_arg1`).

* If you sent request with `cond_type` = `"ge"`, it places Limit Order for the price of `price` when it becomes `current_price` ≥ `conditional_price`(= `cond_arg1`).

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | `True` for selling and `False` for buying.
`role` | string | Required | `"both"`
`quote` | string | Required | Quote coin symbol.
`base` | string | Required | Base coin symbol.
`price` | decimal | Required | Asking price in terms of `price` = `base` / `quote`.
`amount` | decimal | Required | Required amount of `quote`.
`cond_type` | string | Required | `"le"` or `"ge"`
`cond_arg1` | decimal | Required | `conditional_price` of the order. This value is compared with `current_price`.

<aside class="notice">
  Constraint: <code>amount</code> * <code>price in USD</code> ≥ 10.0 <code>USD</code>.
</aside>


### Trailing Stop Order

> Example Trailing Stop Order

```python
await daybit.create_order(
    sell=True, # True for 'fall_from_top', False for 'rise_from_bottom'.
    role='both',
    quote=quote,
    base=base,
    amount=Decimal('0.1'),
    cond_type='fall_from_top', # cond_type could be 'fall_from_top' or 'rise_from_bottom'.
    cond_arg1=Decimal('-0.01'),
    cond_arg2=Decimal('-0.005'),
)
```

You can place trailing stop order to sell the coin, with certain rate of discount compared with current price, when the price has dropped at specific rate compared with highest price. Likewise, you can also buy the coin, with certain rate of extra charge compared with current price, when the price has risen at specific rate compared with lowest price, by placing trailing stop order.

For example, let say you placed a trailing stop order with (sell=True, role='both', quote='BTC', base='USDT', amount='0.1', cond_type='fall_from_top', cond_arg1='-0.01', cond_arg2='-0.005') parameters. After the trailing stop order has placed, if the price has dropped since it reached highest price at 10,000 USDT, it will be triggered at 9,000 USDT (= 1000 USDT * (1 - 0.1)) and create selling limit order at the price of 8955 USDT (= 9,000 USDT * (1 - 0.005)).

* In *Fall From Top* case, when `current_price` ≤ `top_price` * (1 + `cond_arg1`), it places a limit order for the price of `current_price` * (1 + `cond_arg2`). 

* In *Rise From Bottom* case, when `current_price` ≥ `bottom_price` * (1 + `cond_arg1`), it places a limit order for the price of `current_price` * (1 + `cond_arg2`). 

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | `true` for `"fall_from_top"` and `false` for `"rise_from_bottom"`.
`role` | string | Required | `"both"`
`quote` | string | Required | Quote coin symbol.
`base` | string | Required | Base coin symbol.
`amount` | decimal | Required | Order amount.
`cond_type` | string | Required | `"fall_from_top"` or `"rise_from_bottom"`
`cond_arg1` | decimal | Required | In *Fall From Top* case, price discount rate compared with top price<br/>In *Rise From Bottom* case, price rise rate compared with bottom price.   
`cond_arg2` | decimal | Required | When the condition as above is meet, it places a limit order for the price of `current_price` * (1 + `cond_arg2`).


<aside class="notice">
 Constraint: <br />
 <ul>
 <li>
 <code>amount</code> * <code>price in USD at a trailing stop order created</code> ≥ 10.0 <code>USD</code>.
 </li>
 <li>
 In <em>Fall From Top</em> case:<br/> -0.1≤<code>cond_arg1</code>≤-0.02<br/> -0.1≤<code>cond_arg2</code>≤-0.01
 </li>
 <li>
 In <em>Rise From Bottom</em> case:<br/> 0.02≤<code>cond_arg1</code>≤0.1<br/> 0.01≤<code>cond_arg2</code>≤0.1
 </li>
 </ul>
</aside>

## cancel_order()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_cancel_order():
    async with Daybit() as daybit:
        pprint(await daybit.cancel_order(53216861))


asyncio.get_event_loop().run_until_complete(daybit_cancel_order())
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

Cancel placed order. You must pass valid `id` to cancel the order.

* Topic: `/api`

* Event: `cancel_order`

* Rate limit: 100

* Response: [Order](#order-2)

### Arguments

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`order_id` | integer | Required | id of order supposed to be canceled.


## cancel_orders()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_cancel_orders():
    async with Daybit() as daybit:
        my_orders = await daybit.my_orders()
        open_orders = ([my_orders[key]['id'] for key in my_orders if my_orders[key]['status'] == 'placed'])
        pprint(open_orders)
        pprint(await daybit.cancel_orders(open_orders))


asyncio.get_event_loop().run_until_complete(daybit_cancel_orders())
```

> Example Response

```python
{
  'num_canceled_orders': 5
}
```

Cancel multiple orders. If one or more of `order_ids` are invalid, the API simply ignores it and cancels only valid ones. You can check number of canceled ids from `num_canceled_orders` in response.

* Topic: `/api`

* Event: `cancel_orders`

* Rate limit: 5

### Arguments

Parameter | Type | Required | Description
----------|------|----------|-----------|
`order_ids` | array | Required | ids of order supposed to be canceled.

### Response

Field | Description
---------|------------
`num_canceled_orders` | Number of successfully canceled orders.

<aside class="notice">
 Backend server is expecting the orders in CSV format (ex, "1,2,3"). However you should use array of integer if you are using Pydaybit.
 </aside>


## cancel_all_my_orders()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_cancel_all_my_orders():
    async with Daybit() as daybit:
        response = await daybit.cancel_all_my_orders()
        pprint(response)


asyncio.get_event_loop().run_until_complete(daybit_cancel_all_my_orders())
```

> Example Response

```python
{
  'num_canceled_orders': 2
}
```

Cancel all my orders (both sell and buy orders).

* Topic: `/api`

* Event: `cancel_all_my_orders`

* Rate limit: 5

### Response

Field | Description
---------|------------
`num_canceled_orders` | Number of successfully canceled orders.


## create_wdrl()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_create_wdrl():
    async with Daybit() as daybit:
        pprint(await daybit.create_wdrl(coin='BTC', to_addr='fake_address', amount='0.01'))


asyncio.get_event_loop().run_until_complete(daybit_create_wdrl())
```

> Example Response

```python
{
  'amount': '0.01000000',
  'coin': 'BTC',
  'completed_at': None,
  'confirm': 0,
  'created_at': 1538639363972,
  'deposit_status': None,
  'id': 5294,
  'req_confirm': 10,
  'tx_link_url': 'https://live.blockcypher.com/btc/tx/',
  'txid': None,
  'type': 'wdrl',
  'wdrl_status': 'queued',
  'wdrl_to_addr': 'fake_address',
  'wdrl_to_org': None,
  'wdrl_to_tag': None
}
```

Request withdraw to `to_addr` for `amount` of `coin`.

* Topic: `/api`

* Event: `create_wdrl`

* Rate limit: 10

* Response: [Wdrl](#wdrl-2)

### Arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`coin` | string | Required | Withdraw coin.
`to_addr` | string | Required | Withdraw receiving address.
`to_tag` | string | Optional | Withdraw tag. Used by `XRP` and so on.
`amount` | decimal | Required | Amount to withdraw.


## coins()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_coins():
    async with Daybit() as daybit:
        pprint(await daybit.coins())


asyncio.get_event_loop().run_until_complete(daybit_coins())
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

Subscribe to get coin data.

* Topic: `/subscription:coins` 

* Request: `init`

* Notification: `insert`, `update`

* Rate limit: 2

* Response: [Coin](#coin)

* Sort: -


## coin_prices()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


# For every coin
async def daybit_coin_prices():
    async with Daybit() as daybit:
        pprint(await daybit.coin_prices())


# For specific coin
async def daybit_coin_prices_with_sym(sym='ETH'):
    async with Daybit() as daybit:
        pprint(await (daybit.coin_prices / sym)())


asyncio.get_event_loop().run_until_complete(daybit_coin_prices())
# asyncio.get_event_loop().run_until_complete(daybit_coin_prices_with_sym())
```

> Example Response

```python
# If you didn't specify the coin
{
  'ADA': {
    'sym': 'ADA',
    'usd_price': '0.09000000'
  },

  # ...

  'ZRX': {
    'sym': 'ZRX',
    'usd_price': '0.67047870'
  }
}

# If you specified the coin
{
  'ETH': {
    'sym': 'ETH',
    'usd_price': '291.88000000'
  }
}
```

### coin_prices()

Coin to USD exchange rate for every coin. You will get `Noficiation` event whenever price of any coin gets changed. Please note that only updated coin price will be returned.

* Topic: `/subscription:coin_prices`

* Request: `init`

* Notification: `update`

* Rate limit: 5

* Response: [Coin price](#coin-price)

* Sort: -

### (coin_prices / `<sym>`)()

Coin to USD exchange rate for specific coin. You will get `Noficiation` event whenever price of the specified coin gets changed.

* Topic: `/subscription:coin_prices;<sym>`

* Request: `init`

* Notification: `update`

* Rate limit: 5

* Response: [Coin price](#coin-price)

* Sort: -


## quote_coins()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_quote_coins():
    async with Daybit() as daybit:
        pprint(await daybit.quote_coins())


asyncio.get_event_loop().run_until_complete(daybit_quote_coins())
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

Subscribe to get quote coin list.

* Topic: `/subscription:quote_coins`

* Request: `init`

* Notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: [Quote coin](#quote-coin)

* Sort: -

## markets()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_markets():
    async with Daybit() as daybit:
        pprint(await daybit.markets())


asyncio.get_event_loop().run_until_complete(daybit_markets())
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

Subscribe to get basic market data. 

* Topic: `/subscription:markets`

* Request: `init`

* Notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: [Market](#market)

* Sort: -


## market_summary_intvls()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_market_summary_intvls():
    async with Daybit() as daybit:
        pprint(await daybit.market_summary_intvls())


asyncio.get_event_loop().run_until_complete(daybit_market_summary_intvls())
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

Time intervals of market price (unit: second). Return value can be used for input of [Market summaries](#market_summaries).

* Topic: `/subscription:market_summary_intvls`

* Request: `init`

* Notification: `insert`, `update`, `delete`

* Rate limit: 2

* Response: [Market summary interval](#market-summary-interval)

* Sort: by `seconds` in `desc`

## market_summaries()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_market_summaries():
    async with Daybit() as daybit:
        intvls = sorted((await daybit.market_summary_intvls()).keys())
        pprint(await (daybit.market_summaries / intvls[0])())


asyncio.get_event_loop().run_until_complete(daybit_market_summaries())
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

Subscribe to get market summaries. For the valid `seconds`, please refer [Market summary intervals](#market_summary_intvls).

* Topic: `/subscription:market_summaries;<market_summary_intvl>`

* Request: `init`

* Notification: `init`

* Rate limit: 5

* Response: [Market summary](#market-summary)

* Sort: -


## order_books()

> Example Request

```python
import asyncio
from decimal import Decimal
from pprint import pprint

from pydaybit import Daybit


async def daybit_order_books():
    async with Daybit() as daybit:
        quote = 'USDT'
        base = 'BTC'
        price_intvl = Decimal((await daybit.markets())['{}-{}'.format(quote, base)]['tick_price']) * 10
        pprint(await (daybit.order_books / quote / base / price_intvl)())


asyncio.get_event_loop().run_until_complete(daybit_order_books())
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



Order book by unit price. In the response, there's no `id` so you need to compare it with `min_price` or `max_price` to identify the order book. (min_price, max_price] is the range for selling and [min_price, max_price) is the range for buying. In case of `sell_vol` and `buy_vol` exist at the same time, please make sure that they are located in difference range by above range conditions.

* Topic: `/subscription:order_books;<quote>;<base>;<price_intvl>`

* Request: `init`

* Notification: `init`, `upsert`

* Rate limit: 5

* Response: [Order book](#order-book)

* Sort: by `min_price` in `desc`


## price_history_intvls()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_price_history_intvls():
    async with Daybit() as daybit:
        pprint(await daybit.price_history_intvls())


asyncio.get_event_loop().run_until_complete(daybit_price_history_intvls())
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

Time intervals of past market price data (unit: second). Return value can be used for input of [Price histories](#price_histories).

* Topic: `/subscription:price_history_intvls`

* Request: `init`

* Notification: - (nothing to be reported)

* Rate limit: 2

* Response: [Price history interval](#price-history-interval)

* Sort: `seconds` in `asc`


## price_histories()

> Example Request

```python
import asyncio
import time
from pprint import pprint

from pydaybit import Daybit


async def daybit_price_histories():
    async with Daybit() as daybit:
        quote = 'USDT'
        base = 'BTC'
        intvl = sorted((await daybit.price_history_intvls()).keys())[0]
        pprint(
            await (daybit.price_histories / quote / base / intvl)(from_time=int(time.time() * 1000 - intvl * 10 * 1000),
                                                                  to_time=int(time.time() * 1000)))


asyncio.get_event_loop().run_until_complete(daybit_price_histories())
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

Past market price data. For the valid `intvl`, please refer [Price history intervals](#price_history_intvls).

* Topic: `/subscription:price_histories;<quote>;<base>;<intvl>`

* Request: `upsert`

* Notification: `insert`, `update`

* Rate limit: 20

* Response: [Price history](#price-history)

* Sort: -

### Arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`from_time` | unix_timestamp | Required | Start time of the price history range.
`to_time` | unix_timestamp | Required | End time of the price history range.



## trades()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_trades():
    async with Daybit() as daybit:
        quote = 'USDT'
        base = 'BTC'
        pprint(await (daybit.trades / quote / base)(size=10))


asyncio.get_event_loop().run_until_complete(daybit_trades())
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

Subscribe to get trade data per market. API doesn't support for getting past trade data.

* Topic: `/subscription:trades/<quote>/<base>` 

* Request: `init`

* Notification: `insert`

* Rate limit: 5

* Response: [Trade](#trade)

* Sort: by `id` in `desc`

### Arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`num_trades` | integer | Required | Trade count for retrieving.


## my_users()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_my_users():
    async with Daybit() as daybit:
        pprint(await daybit.my_users())


asyncio.get_event_loop().run_until_complete(daybit_my_users())
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

Subscribe to get information of my account.

* Topic: `/subscription:my_users`

* Request: `init`

* Notification: `init`

* Rate limit: 2

* Response: [User](#user)

* Sort: -


## my_assets()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_my_assets():
    async with Daybit() as daybit:
        pprint(await daybit.my_assets())


asyncio.get_event_loop().run_until_complete(daybit_my_assets())
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


Subscribe to get information of my assets.

* Topic: `/subscription:my_assets`

* Request: `init`

* Notification: `insert`, `update`

* Rate limit: 5

* Response: [Asset](#asset)

* Sort: -


## my_orders()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_my_orders():
    async with Daybit() as daybit:
        pprint(await daybit.my_orders(statuses='placed'))


asyncio.get_event_loop().run_until_complete(daybit_my_orders())
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

Subscribe to get information of my orders.

* Topic: `/subscription:my_orders`

* Request: `upsert`

* Notification: `insert`, `update`

* Rate limit: 5

* Response: [Order](#order-2)

* Sort: by `id` in `desc`

### Arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`quote` | string | Optional | Quote coin symbol. ex) "BTC"
`base` | string | Optional | Base coin symbol. ex) "ETH"
`to_id` | integer | Optional | Get my orders that are `id` is smaller than `to_id`.
`size` | integer | Optional | Order count for retrieving. `size` ≤  30. 
`sell` | boolean | Optional | `true` for selling order and `false` for buying order.
`statuses` | csv | Optional | Conditions of `status` to retrieve. csv form of `received`, `placed`, `completed`, and `canceled`. (ex, `"received, placed"`)



## my_trades()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_my_trades():
    async with Daybit() as daybit:
        pprint(await daybit.my_trades(sell=True, quote='USDT'))


asyncio.get_event_loop().run_until_complete(daybit_my_trades())
```

> Example Response

```python
{
  40807547: {
    'base': 'BTC',
    'base_amount': '0.00020000',
    'coin_fee': '0.00141820',
    'counterpart': 'user',
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
    'counterpart': 'user',
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

Subscribe to get information of my trade data.

* Topic: `/subscription:my_trades`

* Request: `upsert`

* Notification: `insert`

* Rate limit: 5

* Response: [Trade](#trade)

* Sort: by `id` in `desc`

### Arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`quote` | string | Optional | Quote coin symbol. ex) "BTC"
`base` | string | Optional | Base coin symbol. ex) "ETH"
`to_id` | integer | Optional | Get my orders that are `id` is smaller than `to_id`.
`size` | integer | Optional | Order count for retrieving. `size` ≤  30.
`sell` | boolean | Optional | `true` for selling order and `false` for buying order.


## my_tx_summaries()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_my_tx_summaries():
    async with Daybit() as daybit:
        pprint(await daybit.my_tx_summaries(type='deposit'))
        pprint(await daybit.my_tx_summaries(type='wdrl'))


asyncio.get_event_loop().run_until_complete(daybit_my_tx_summaries())
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

Subscribe to get my transaction summaries.

* Topic: `/subscription:my_tx_summaries`

* Request: `upsert`

* Notification: `insert`, `update`

* Rate limit: 5

* Response: [Deposit](#deposit)

* Sort: by `id` in `desc`

### Arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`type` | string | Required | One of `deposit` or `wdrl` per your purpose.


## my_airdrop_histories()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_my_airdrop_histories():
    async with Daybit() as daybit:
        pprint(await daybit.my_airdrop_histories())


asyncio.get_event_loop().run_until_complete(daybit_my_airdrop_histories())
```

> Example Response

```python
{}
```

Subscribe to get list of my airdrops.

* Topic: `/subscription:my_airdrop_histories`

* Request: `upsert`

* Notification: `insert`

* Sort: by `id` in `desc`

* Response: [Airdrop](#airdrop)

### arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`to_id` | integer | Optional | Get my orders that are `id` is smaller than `to_id`.
`size` | integer | Optional | Order count for retrieving. `size` ≤  30.
