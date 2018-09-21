# **Topic: API**

[Topic](#terms) are string identifiers of the channels. You can find trade and order related APIs in this section. It also shows how to use wrapper and expected response from it. For valid `topic` and `event` of the [message](https://hexdocs.pm/phoenix/Phoenix.Socket.Message.html), please look below.

* Topic: `/api`

* Event: `get_server_time`, `create_order`, `cancel_order`, `cancel_orders`, `cancel_all_my_orders`, or `create_wdrl`

* Rate limit: Limit of calls for every second.

<aside class="notice">
It is recommended to retrieve data from `notification` of `/subscription:<sub_topic>` topic not `response` from `/api` topic. It might cause confliction at `insert` action from `notification` because of two separate data roots.
</aside>

# Miscellaneous

## Get server time

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

### event: `get_server_time`

* Descrition: Get current server time of Daybit backend server in unix timestamp format.

* Rate limit: 10

# Trade

## Create order

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

### event: `create_order`

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

## Cancel order

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

### event: `cancel_order`

* Description: Cancel placed order. You must pass valid `id` to cancel the order.

* Rate limit: 100

* Response: [Order](#order-2)

### arguments

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`order_id` | integer | Required | id of order supposed to be canceled.

## Cancel orders

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

### event: `cancel_orders`

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
 Backend server is expecting the orders in CSV format (ex, "1,2,3"). However you can use array of integer if you are using Daybit official wrapper.
 </aside>

## Cancel my orders

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

### event: `cancel_all_my_orders`

* Description: Cancel all my orders (both sell and buy orders).

* Rate limit: 5

### response

Field | Description
---------|------------
`num_canceled_orders` | Number of successfully canceled orders.

# Transaction

## Create withdraw

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

### event: `create_wdrl`

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
