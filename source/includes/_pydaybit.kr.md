# **Pydaybit**
Pydaybit은 파이썬으로 작성된 데이빗 거래소 API 레퍼 _warppaer_ 입니다. 코드는 [Pydaybit 저장소](https://github.com/daybit-exchange/pydaybit)에 있습니다. 


## **Disclaimer**
_이 소프트웨어의 사용에 대한 책임은 사용자 본인에게 있습니다. 작성자와 모든 관계자들은 당신의 거래의 결과에 대해 책임을 지지 않습니다._

## **Installation**
  
> Pydaybit installation

```shell
$ git clone https://github.com/daybit-exchange/pydaybit
$ pip install -e pydaybit
```

설치 방법은 오른쪽을 참고 하십시오.

## **API Key Pair**

Pydaybit을 사용하기 위해서는 API 키페어 생성을 해야합니다. [권한](#authorization) 항목을 참조하십시오.

### Environment Variables

> bash example

```shell
# ~/.bash_profile
export DAYBIT_API_KEY="YOUR_OWN_API_KEY"
export DAYBIT_API_SECRET="YOUR_WON_API_SECRET"
```

환경 변수를 세팅하여 사용할 수 있습니다

* `DAYBIT_API_KEY`: 생성한 API 키
* `DAYBIT_API_SECRET`: 생성한 API 시크릿


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

API 키패어는 환경 변수 설정 없이도 사용할 수 있습니다. 예제를 참고하십시오.

## get_server_time()

> Example Request

```python
import asyncio
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

데이빗 API 서버의 현재 시간을 밀리초 단위의 유닉스 타임스탬프를 받아옵니다.

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

코인은 사거나 파는 주문을 생성합니다. 데이빗 API 서버에 요청할 수 있는 다섯 가지의 주문 종류가 있습니다 - [Limit order](#limit-order), [Taker Order](#taker-order), [Maker Order](#maker-order), [Stop Limit Order](#stop-limit-order), [Trailling Stop Order](#trailing-stop-order).
 
잘못된 주문(주문 무효)의 경우는 다음과 같습니다

- 거부된 주문
- 유저의 요청으로 인한 취소
- 자전 거래로 인한 취소
- 오래된 주문의 자동 취소

`sell`, `quote`, `base`, `amount`, `role`, `cond_type` 파라미터 값은 모든 주문에서 필요합니다.

* Topic: `/api`

* Event: `create_order`

* Rate limit: -

* Response: [Order](#order-2)

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | 판매일 경우 `True`, 구매일 경우 `False`.
`quote` | string | Required | 호가 코인 기호. 예) "BTC"
`base` | string | Required | 기준 코인 기호. 예) "ETH"
`amount` | decimal | Required | 주문 수량.
`role` | string | Required | 마켓에서의 주문의 역할 (`"both"`, `"maker_only"`, `"taker_only"`). 
`cond_type` | string | Required | 주문의 조건 종류 (`"none"`, `"le"`, `"ge"`, `"fall_from_top"`, `"rise_from_bottom"`).
`price` | decimal | Optional | 주문 가격.
`cond_arg1` | decimal | Optional | 조건 주문의 첫번째 인자.
`cond_arg2` | decimal | Optional | 조건 주문의 두번째 인자.

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

이 주문이 접수되면 먼저 [테이커 주문](#taker-order) _taker order_ 로 여겨지고, 만약 주문의 잔량이 있다면 [메이커 주문](#maker-order) _maker order_ 가 됩니다.

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | 판매일 경우 `True`, 구매일 경우 `False`.
`role` | string | Required | `"both"`
`quote` | string | Required | 호가 코인 기호. 예) "BTC"
`base` | string | Required | 기준 코인 기호. 예) "ETH"
`price` | decimal | Required | 주문 가격. `price` = `base` / `quote`.
`amount` | decimal | Required | `quote` 기준 주문 수량.
`cond_type` | string | Required | `"none"`

<aside class="notice">
  제한 조건: <code>주문 수량</code> * <code>USD 환산 가격</code> ≥ 10.0 <code>USD</code>.
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

이 주문은 오더북에 있는 거래량 _volume_ 을 가져가서 _tacking_ 오더북에 등록되기 전에 거래를 체결하기 때문에 `테이커` _tacker_ 라고 합니다.

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | 판매일 경우 `True`, 구매일 경우 `False`.
`role` | string | Required | `"taker_only"`
`quote` | string | Required | 호가 코인 기호.
`base` | string | Required | 기준 코인 기호.
`price` | decimal | Required | 주문 가격. `price` = `base` / `quote`.
`amount` | decimal | Required | `quote` 기준 주문 수량.
`cond_type` | string | Required | `"none"`

<aside class="notice">
  제한 조건:  <code>주문 수량</code> * <code>USD 환산 가격</code> ≥ 10.0 <code>USD</code>.
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

주문이 오더북을 채워 마켓을 만들기 _making_ 하기 때문에 이러한 주문은 `메이커` _maker_ 라고 불립니다. 이 주문은 오더북을 채울 때만 유효합니다.

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | 판매일 경우 `True`, 구매일 경우 `False`.
`role` | string | Required | `"maker_only"`
`quote` | string | Required | 호가 코인 기호.
`base` | string | Required | 기준 코인 기호.
`price` | decimal | Required | 주문 가격. `price` = `base` / `quote`.
`amount` | decimal | Required | `quote` 기준 주문 수량.
`cond_type` | string | Required | `"none"`

<aside class="notice">
  제한 조건:  <code>주문 수량</code> * <code>USD 환산 가격</code> ≥ 10.0 <code>USD</code>.
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
    cond_arg1=price * Decimal(1.5)
)
```

현재 가격이 `cond_arg`보다 같거나 클 때/ 같거나 작을 때, 리미트 _limit_ 주문을 접수합니다.

* `cond_type` = `"le"`를 요청하면, `현재가` ≤ `조건가`(= `cond_arg1`)일 때에 `price` 가격에 리미트 주문을 접수합니다.

* `cond_type` = `"ge"`를 요청하면, `현재가` ≥ `조건가`(= `cond_arg1`)일 때에 `price` 가격에 리미트 주문을  접수합니다.

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | 판매일 경우 `True`, 구매일 경우 `False`.
`role` | string | Required | `"both"`
`quote` | string | Required | 호가 코인 기호.
`base` | string | Required | 기준 코인 기호.
`price` | decimal | Required | 주문 가격. `price` = `base` / `quote`.
`amount` | decimal | Required | `quote` 기준 주문 수량.
`cond_type` | string | Required | `"le"` 혹은 `"ge"`.
`cond_arg1` | decimal | Required | 주문의 `조건가`. 이 값을 `현재가`와 비교합니다.

<aside class="notice">
  제한 조건: <code>주문 수량</code> * <code>USD 환산 가격</code> ≥ 10.0 <code>USD</code>.
</aside>


### Trailing Stop Order

> Example Trailing Stop Order

```python
await daybit.create_order(
    sell=True,  # True for 'down_from_high', False for 'up_from_low'.
    role='both',
    quote='USDT',
    base='BTC',
    amount=Decimal('0.1'),
    cond_type='down_from_high',  # cond_type could be 'down_from_high' or 'up_from_low'.
    cond_arg1=Decimal('-0.01'),
    cond_arg2=Decimal('-0.005'),
)
```

 트래일링 스탑 주문 _trailing stop order_ 을 사용하면, 가격이 고가에 크게 떨어질 때 현재 가격보다 조금 낮은 가격에 매도 리미트 주문을 접수할 수 있다. 마찬가지로, 현재 가격이 저가에 비해 크게 상승하면 현재가격보다 조금 높은 가격에 매수 리미트 주문을 접수할 수 있다.

예를들어, (`sell`=True, `role`='both', `quote`='BTC', `base`='USDT', `amount`='0.1', `cond_type`='down_from_high', `cond_arg1`='-0.01', `cond_arg2`='-0.005')로 주문을 냈다고 하자. 트래일링 스탑 주문이 접수되고, 가격이 10,000 USDT에 도달한뒤 떨어지기 시작한 상황에서는 이 주문은 9,000 USDT (= 1000 USDT * (1 + `cond_arg1`))에 트리거 되어 리미트 오더를 8,955 USDT (= 9,000 USDT * (1 + `cond_arg2`))에 접수한다.

* 고점에서 하락 *down from high* 의 경우, `현재가` ≤ `고가` * (1 + `cond_arg1`)일 때, 가격이 `현재가` * (1 + `cond_arg2`)인 매도 리미트 오더를 접수한다.

* 저점에서 상승 *up from low* 의 경우, `현재가` ≥ `저가` * (1 + `cond_arg1`)일 때, 가격이 `현재가` * (1 + `cond_arg2`)인 매수 리미트 오더를 접수한다. 

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`sell` | boolean | Required | `"down_from_high"`일 경우 `True`, `"up_from_low"`일 경우 `false`. 
`role` | string | Required | `"both"`
`quote` | string | Required | 호가 코인 기호.
`base` | string | Required | 기준 코인 기호.
`amount` | decimal | Required | 주문 수량.
`cond_type` | string | Required | `"down_from_high"` 혹은 `"up_from_low"`.
`cond_arg1` | decimal | Required | 고점에서 하락 *down from high* 의 경우, 고점 대비 가격 하락 비율.<br/>저점에서 상승 *up from low* 의 경우, 저점 대비 가격 상승 비율.   
`cond_arg2` | decimal | Required | `cond_arg1` 관련 조건이 만족되면, `현재가` * (1 + `cond_arg2`) 가격에 리미트 주문을 접수한다.


<aside class="notice">
 제한 조건: <br />
 <ul>
 <li>
 <code>주문 수량</code> * <code>트래일링 스탑 주문을 접수할 때 USD 환산 가격</code> ≥ 10.0 <code>USD</code>.
 </li>
 <li>
 고점에서 하락 <em>down from high</em> 의 경우:<br/> -0.1≤<code>cond_arg1</code>≤-0.02<br/> -0.1≤<code>cond_arg2</code>≤-0.01
 </li>
 <li>
 저점에서 상승 <em>up from low</em> 의 경우:<br/> 0.02≤<code>cond_arg1</code>≤0.1<br/> 0.01≤<code>cond_arg2</code>≤0.1
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

접수된 주문을 취소합니다. 주문 취소를 위해서는 유효한 주문 `id`를 넣어야 합니다.

* Topic: `/api`

* Event: `cancel_order`

* Rate limit: -

* Response: [Order](#order-2)

### Arguments

Parameter | Type | Required | Description
----------|------|----------|------|----------|------------
`order_id` | integer | Required | 취소하려고 하는 주문의 `id`


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

여러 개의 주문을 한번에 취소할 때 사용합니다. `order_ids`에서 일치하는 `id`를 가지는 유효한 주문을 취소합니다. 취소된 주문의 갯수는 응답의 `num_canceled_orders`를 참조하십시오.

* Topic: `/api`

* Event: `cancel_orders`

* Rate limit: 1

### Arguments

Parameter | Type | Required | Description
----------|------|----------|-----------|
`order_ids` | array | Required | 쥐소하려고 하는 주문들의 id.

### Response

Field | Description
---------|------------
`num_canceled_orders` | 취소한 주문의 갯수.

<aside class="notice">
데이빗 API 서버는 주문들의 id를 CSV 형식(예를 들어, "1,2,3")으로 입력받습니다. 하지만 Pydaybit을 사용할 때에는 주문 <code>id</code>의 리스트를 사용하십시오.
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

내 모든 주문을 취소합니다.

* Topic: `/api`

* Event: `cancel_all_my_orders`

* Rate limit: 1

### Response

Field | Description
---------|------------
`num_canceled_orders` | 취소한 주문의 갯수.


## create_wdrl()

> Example Request

```python
import asyncio
from pprint import pprint

from pydaybit import Daybit


async def daybit_create_wdrl():
    async with Daybit() as daybit:
        pprint(await daybit.create_wdrl(coin='BTC', to_addr='<YOUR BTC ADDRESS>', amount='0.01'))


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
  'wdrl_to_addr': '<YOUR BTC ADDRESS>',
  'wdrl_to_org': None,
  'wdrl_to_tag': None
}
```

`to_addr`로 `amount`만큼의 `coin`의 출금을 요청합니다.

* Topic: `/api`

* Event: `create_wdrl`

* Rate limit: 50

* Response: [Transaction summary](#transaction-summary)

### Arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`coin` | string | Required | 출금 코인.
`to_addr` | string | Required | 출금을 받을 주소.
`to_tag` | string | Optional | 출금 태그. `XRP` 등에서 사용.
`amount` | decimal | Required | 출금 수량.


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
    'sym': 'ZRX',
    'tick_amount': '0.20000000',
    'tradable': True,
    'wdrl_confirm': 10,
    'wdrl_enabled': True,
    'wdrl_fee': '6.30000000'
  }
}
```

코인 데이타를 구독합니다.

* Topic: `/subscription:coins` 

* Request: `init`

* Notification: `insert`, `update`

* Rate limit: 3

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

모든 코인의 USD 환산 가격을 받아 옵니다. 어떤 코인이라도 가격에 변화가 있으면 `notification` 이벤트를 받을 것입니다. 가격 변화가 있는 코인만 업데이트 됩니다.

* Topic: `/subscription:coin_prices`

* Request: `init`

* Notification: `update`

* Rate limit: 3

* Response: [Coin price](#coin-price)

* Sort: -

### (coin_prices / `<sym>`)()

특정 코인의 USD 환산 가격을 받아 옵니다. 명시한 코인의 가격이 변화가 있을 때 마다 `notification` 이벤트를 받을 것입니다.

* Topic: `/subscription:coin_prices;<sym>`

* Request: `init`

* Notification: `update`

* Rate limit: 3

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

* Rate limit: 3

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

* Rate limit: 3

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

* Rate limit: 3

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

* Rate limit: 3

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

* Rate limit: 3

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

* Rate limit: 3

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

* Rate limit: 8

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

* Rate limit: 3

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
    'one_day_wdrl_usd_limit': '5000.00000000',
    'pay_fee_with_day': True,
    'taker_fee_rate': '0.001000'
  }
]
```

Subscribe to get information of my account.

* Topic: `/subscription:my_users`

* Request: `init`

* Notification: `init`

* Rate limit: 3

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
    'investment_usd': '0.000000000000000000',
    'reserved': '0.000000000000000000',
    'total': '700000.000000000000000000',
    'visible': True
  },

  # ...

  'ZRX': {
    'available': '90000.000000000000000000',
    'coin': 'ZRX',
    'investment_usd': '0.000000000000000000',
    'reserved': '0.000000000000000000',
    'total': '90000.000000000000000000',
    'visible': True
  }
}
```


Subscribe to get information of my assets.

* Topic: `/subscription:my_assets`

* Request: `init`

* Notification: `insert`, `update`

* Rate limit: 3

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
        pprint(await daybit.my_orders(closed=False))


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

* Rate limit: 3

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
`closed` | boolean | Optional | Conditions of `status` to retrieve. `false` will retrieve orders in `received` and `placed` status. `true` will retrieve orders in `closed` status.

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

* Rate limit: 3

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

* Rate limit: 3

* Response: [Transaction Summary](#transaction-summary)

* Sort: by `id` in `desc`

### Arguments

Parameter | Type | Required | Description
----------|------|----------|------------
`type` | string | Required | One of `deposit` or `wdrl` per your purpose.
`to_id` | integer | Optional | Get your transactions that are `id` is smaller than `to_id`.
`size` | integer | Optional | The count of your transactions for retrieving. `size` ≤  30.


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
{10: {'amount': '200.00000000',
      'category': 'project',
      'coin': 'DAY',
      'exec_at': 1539148862461,
      'id': 10}}
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
`to_id` | integer | Optional | Get your airdrop histories that are `id` is smaller than `to_id`.
`size` | integer | Optional | The count of your airdrop histories for retrieving. `size` ≤  30.
