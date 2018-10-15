---
title: 데이빗 API 레퍼런스

includes:
  - pydaybit
  
search: true

toc_footers:
 - <a href='https://www.daybit.com'>Daybit</a>
 - <a href='https://www.daybit.com/policy/terms-api'>Terms of Service for Daybit API</a> 
---

# Introdution

이 문서는 기본적인 프로그래밍 능력을 갖춘 독자를 대상으로 작성되었습니다. 또한 당신의 자산에 손해를 가져다 줄 수 있는 예제가 포함되어 있습니다. 각각의 기능에 대한 설명과 코드를 충분히 이해하고 실행하도록 합니다. API 사용에 대한 모든 책임은 본인에게 있습니다. 자세한 내용은 [데이빗 API 이용약관](https://www.daybit.com/policy/terms-api)을 참고하십시오.

데이빗 API를 이용하기에 앞서 [데이빗](https://www.daybit.com)에서 API 키페어를 발급받아야 합니다. API 키페어는 여러가지 권한 - [캔들 차트](https://en.wikipedia.org/wiki/Candlestick_chart)나 [오더북](https://en.wikipedia.org/wiki/Order_book_(trading)) 가져오기, 자산을 확인하기, 자산을 거래하기, 자산 출금하기 권한 - 을 가집니다. 자세한 내용은 [권한](#authorization) 항목을 참고하십시오.

Daybit API에는 두가지 타입이 있습니다. 첫 번째는 클라이언트가 요청을 보내고 서버가 알맞은 응답을 보내는 방식으로 [API 호출](#api-calls)이라 합니다. 이러한 API는 보통 자산의 거래, 입출금에 관련되어 있습니다. 두 번째는 클라이언트가 API를 구독하면 데이빗 API 서버에서 지속적으로 알림을 보내 주는 [API 구독](#subscriptions)입니다. API 구독에는 코인 가격 변화, 지갑 정보 구독, 주문 결과 구독 등이 있습니다.

데이빗 API는 웹소켓을 기반으로 구현되어 있어 있고, [Phoenix](https://phoenixframework.org)에서 정의한 방식으로 메시지를 주고 받습니다. 또한 데이빗은 사용자가 API를 쉽게 사용할 수 있도록 [Pydaybit](#pydaybit)를 제공합니다. Pydaybit은 파이썬으로 작성된 데이빗 API 레퍼(Wrapper)입니다. 

만약 프로그래밍에 익숙하지 않다면 [Pydaybit](#pydaybit)의 예제를 먼저 읽어보는 것을 추천합니다.
 
# Authorization

데이빗 API를 사용하기 전에, API 키페어를 적절한 권한으로 생성해야 합니다. 하나의 API 키페어는 여러 종류의 권한 - [캔들 차트](https://en.wikipedia.org/wiki/Candlestick_chart)나 [오더북](https://en.wikipedia.org/wiki/Order_book_(trading)) 가져오기, 자산을 확인하기, 자산을 거래하기, 자산 출금하기 권한 - 을 가집니다. 

API 사용은 API 키페어에 부여된 권한에 의해 제한됩니다. 만약 접근 권한이 없는 API 키페어를 이용해 API를 사용한다면, `unauthenticated` 응답을 받습니다.

| 종류 | 설명 |
|------|-------------|
| public_data | 공개 정보(시장 요약 정보, 오더북 등)에 대한 접근 권한. API 키페어 생성 시에 `시장 조회`를 포함합니다.
| private_data | 개인 자산에 대한 접근 권한. API 키페어 생성 시에 `개인 자산 조회`를 포함합니다.
| trade | 거래 관련 API에 대한 접근 권한. API 키페어 생성 시에 `거래 관련 API`를 포함합니다.
| transaction | 입금과 출금 관련 API에 대한 접근 권한. API 키페어 생성 시에 `출금 관련 API`를 포함합니다.

마이페이지의 [API Keys](https://www.daybit.com/mypage/api-managements) 탭에서 계정당 5개의 API 키패어를 생성 할 수 있습니다. 각 API 키페어는 다음 중 하나의 권한들을 갖습니다.

* `public_data`
* `public_data`, `private_data`, `trade`
* `public_data`, `private_data`, `transaction`
* `public_data`, `private_data`, `trade`, `transaction`

# Host address

* 데이빗 API 엔드포인트 : [wss://api.daybit.com/v1/user_api_socket/websocket/](wss://api.daybit.com/v1/user_api_socket/websocket/)

# APIs
Daybit API에는 두가지 타입이 있습니다. 첫 번째는 클라이언트가 요청을 보내고 서버가 알맞은 응답을 보내는 방식으로 [API 호출](#api-calls)이라 합니다. 이러한 API는 보통 자산의 거래, 입출금에 관련되어 있습니다. 두 번째는 클라이언트가 API를 구독하면 서버에서 지속적으로 알림을 보내 주는 [API 구독](#subscriptions)입니다. API 구독에는 코인 가격 변화, 지갑 정보 구독, 주문 결과 구독 등이 있습니다.

## Channels
채널에서는 메시지를 보내고 받습니다. 발신자는 어떤 [주제](#topics)에 관해 메시지를 브로드캐스트(채널 참가자 모두에게 발신함) 할 수 있습니다. 수신자는 [주제](#topics)를 구독하면 그러한 메시지들을 수신할 수 있습니다. 언제든지 수신자가 발신자가 되거나, 발신자가 수신자가 될 수 있습니다. 자세한 정보는 [Phoenix 문서](https://hexdocs.pm/phoenix/channels.html)를 참고하십시오.

### Topics
주제 _Topic_ 는 다양한 층에서의 메시지가 올바른 곳으로 전달 될 수 있도록하는 [채널](#channels)의 문자열 식별자입니다. 데이빗 API는 다음과 같은 주제를 사용하고 있습니다.

* [`/api`](#api-calls)
* [`/subscription:coins`](#coins)
* [`/subscription:market_summaries;<market_summary_intvl>`](#market_summaries)

### Events
이벤트 _Event_ 는 채널에서 특별한 행동을 나타내기 위한 문자열입니다. 예를 들어 `"phx_join"`은 채널 참가를 의미하는 이벤트이고, `"phx_leave"`는 채널 나오기를 의미하는 이벤트입니다. [데이빗 API 호출](#api-calls) 은 `"/api"` 채널에서 이벤트를 사용해 요청을 보냅니다.
 
### Messages

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
< {"topic":"/api","ref":"2","payload":{"status":"ok","response":{"data":{"wdrl_to_tag":null,"wdrl_to_org":null,"wdrl_to_addr":"ABTCADDRESS","wdrl_status":"queued","type":"wdrl","txid":null,"tx_link_url":"https://live.blockcypher.com/btc/tx/","req_confirm":2,"id":5882,"deposit_status":null,"created_at":1538739991072,"confirm":0,"completed_at":null,"coin":"BTC","amount":"0.050000000000000000"}}},"event":"phx_reply"}
> {"join_ref": "1", "ref": "3", "topic": "/api", "event": "phx_leave", "payload": {"timestamp": 1538739991124}, "timeout": 3000}
< {"topic":"/api","ref":"3","payload":{"status":"ok","response":{}},"event":"phx_reply"}
< {"topic":"/api","ref":"1","payload":{},"event":"phx_close"}
```

다음 속성들은 [JSON](https://en.wikipedia.org/wiki/JSON) 객체 형식으로 전달됩니다. 예제는 `/api` 채널에서 `create_wdrl` API 호출할 때의 웹소켓 위에서 주고 받는 메시지를 보여 줍니다.

* `topic` - 문자열 타입의 채널 식별자. 예를 들어, [`"/api"`](#api-calls)나 [`"/subscription:coins"`](#coins)가 있다.
* `event` - 문자열 타입의 이벤트 이름, 예를 들어, [`"create_order"`](#create_order), [`"cancel_order"`](#cancel_order), `"phx_join"`, `"phx_leave"`가 있다.
* `payload` - 메시지 페이로드. JSON 객체 형식으로 API를 사용할 때 주요 정보가 전달된다.
* `ref` - 문자열 타입의 메시지 고유 레퍼런스 번호.
* `join_ref` - 문자열 타입의 채널 고유 레퍼런스 번호. 채널에 참가했을 때의 `ref`와 같다.

## API Calls
웹소켓을 통해 [데이빗 API 엔드포인트](#host-address)와 연결하고 `/api` 채널에 참가합니다. 해당하는 API 이름을 `event`에 넣고, 필요한 인자 `payload`에 넣어 메시지를 보냅니다.

### Channel of API Calls
모든 API Call은 `/api` 채널을 사용합니다.

### API Call List
`/api` 채널에서 `event` 속성에 API 이름을 넣습니다. 사용가능한 목록은 다음과 같습니다.

* [`create_order`](#create_order)
* [`cancel_order`](#cancel_order)
* [`cancel_orders`](#cancel_orders)
* [`cancel_all_my_orders`](#cancel_all_my_orders)
* [`create_wdrl`](#create_wdrl)
* [`get_server_time`](#get_server_time)

<aside class="notice">
거래의 결과 등을 확인하기 위해서 <code>/api</code> 채널에서의 응답을 사용하는 것보다, <code>subscription:&ltsub_topic&gt</code> 채널에서 구독을 통하여 필요한 정보를 추적하는 것을 권장합니다.
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

API 구독을 하기 위해서, 먼저 `/subscription:<subtopic>` 채널에 참가하고, `request` 이벤트를 보냅니다. 채널에 참가해서 구독에 성공하면, 데이빗 API 서버는 데이터에 업데이트가 있을 때마다 `notification` 이벤트를 보내 줍니다.

### API Subscription Channels
다음과 같은 구독 채널이 있습니다. 채널의 토픽은 `/subscription:<subtopic>` 형식입니다. 예를 들어, `coins` API의 경우에는 채널 토픽이 `/subscription:coins` 입니다.

* [`coins`](#coins)
* [`coin_prices`](#coin_prices)
* [`quote_coins`](#quote_coins)
* [`markets`](#markets)
* [`market_summary_intvls`](#market_summary_intvls)
* [`market_summaries`](#market_summaries)
* [`order_books`](#order_books)
* [`price_history_intvls`](#price_history_intvls)
* [`price_histories`](#price_histories)
* [`trades`](#trades), [`my_users`](#my_users)
* [`my_assets`](#my_assets)
* [`my_orders`](#my_orders)
* [`my_trades`](#my_trades)
* [`my_tx_summaries`](#my_tx_summaries)
* [`my_airdrop_histories`](#my_airdrop_histories) are the topics you can use in API subscriptions.

### Event of API Subscriptions
 `request`는 보내기 이벤트, `notification`는 받기 이벤트입니다. 클라이언트와 서버가 전송하는 [메세지](https://hexdocs.pm/phoenix/Phoenix.Socket.Message.html)는 각각 `request` 이벤트나 `notification` 이벤트를 가집니다. 한 API를 구독하면, 다시 말해, 해당하는 채널에 참가하여 `request` 이벤트를 보내면, 그 채널에서 `init`이나 `upsert` [액션](#action)을 응답으로 받습니다. 그 이후로는 `notification`이벤트와 함께 `insert`, `update`, `upsert`, `delete` 중 하나의 액션을 받습니다. 더 자세한 사항은 [액션](#action) 항목을 참고하십시오.

### Rate Limit of API Subscriptions
`request` 이벤트를 보내는 데에 초당 제한이 있습니다.

## Rate limit

각 API는 매 초마다 호출에 제한이 있습니다. 만약 제한을 넘으면 `api_exceeded_rate_limit` 에러 코드를 받습니다.

## Timestamp

모든 요청은 네트워크 지연 등에 의한 예상치 못한 요청을 방지하기 위하여 `timestamp`와 `timeout`을 가집니다.

- `timestamp`: `unix_timestamp`
- `timeout`(**optional**): `integer` 타입의 `ms` 시간 단위 . 기본 값은 3초를 의미하는 `3000` 입니다.

요청은 다음 조건을 만족하지 못하면 거부됩니다: `server time` - `timestamp` > `timeout`.

만약 문제가 있다면, 아래의 에러코드가 응답으로 옵니다.

- `api_invalid_timestamp_or_timeout`: `timestamp`이 존재하지 않거나 `timestamp`나 `timeout`가 `integer` 타입의 유닉스 밀리초 단위가 아닙니다.
- `api_timeout`: 시간 초과로 인해 거부합니다.

## Response format

여기에는 두가지 응답 형식이 있습니다. API 호출이나 구독의 결과에 따라 성공, 실패 모델 중에 하나를 받을 것 입니다. 오른쪽의 성공과 실패 예제를 참고 하십시오. 또한 아래에서 [오류 목록](#error-list)을 찾아 보십시오.

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

데이빗은 [엘릭서](https://elixir-lang.org/) 언어를 사용하고 [피닉스 프레임워크](https://phoenixframework.org/)를 기반하여 만들어 졌습니다. 데이빗 API는 피닉스 서버와 소켓 통신을 하는 피닉스 클라이언트를 통해 접근할 수 있습니다. 데이빗은 공식적으로 파이썬 언어로된 레퍼 _wrapper_ 를 제공합니다. 

* PyDaybit(Python): [https://github.com/daybit-exchange/pydaybit](https://github.com/daybit-exchange/pydaybit/)

다른 언어로 데이빗 API 를 사용하기 위해서는 다음 라이브러리를 참고하십시오.

* Phoenix.js(Javascript): [https://github.com/phoenixframework/phoenix](https://github.com/phoenixframework/phoenix/)
* SwiftPhoenixClient(Swift): [https://github.com/davidstump/SwiftPhoenixClient](https://github.com/davidstump/SwiftPhoenixClient/)
* PhoenixSharp(C#): [https://github.com/Mazyod/PhoenixSharp](https://github.com/Mazyod/PhoenixSharp/)

# Error list

### General

* `unauthenticated`: 인증되지 않은 유저 혹은 이용
* `invalid_arguments`: 잘못된 인자를 가진 요청 _request_
* `resource_not_found`: 리소스가 없음

### Api

* `api_invalid_timestamp_or_timeout`: `timestamp`이 존재하지 않거나 `timestamp`나 `timeout`가 `integer` 타입의 유닉스 밀리초 단위가 아님.
* `api_timeout`: 시간초과가 요청된 `timestamp`, `timeout` 에서 발생함
* `api_exceeded_rate_limit`: 요청 제한 초과
* `api_invalid_param_types`: 잘못된 인자 타입
* `api_required_params_not_provided`: 필요한 인자가 누락됨

### Order

* `order_invalid_market`: 잘못된 마켓. _마켓은 (`quote`, `base`) 쌍으로 표현된다_
* `order_not_tradable_coin`: 해당 코인의 거래가 중단됨
* `order_not_sellable_market`: 해당 마켓에서의 판매가 중단됨
* `order_not_buyable_market`: 해당 마켓에서의 구매가 중단됨
* `order_invalid_price`: 잘못된 가격
* `order_invalid_amount`: 잘못된 수량
* `order_only_both_role_can_be_cond`: 조건 주문의 `role`은 항상 `both`이어야 한다. [`create_order()`](#create_order) 참고.
* `order_out_of_price_range`: 가격의 범위가 잘못됨. (판매: 현재가의 20% ~ 200%, 구매: 50% ~ 500%)
* `order_exceeded_max_tstops`: 트레일링 스탑 주문의 갯수 제한을 넘음 (마켓당 2개)
* `order_suspended_due_to_frequent_canceling`: 빈번한 주문 취소로 인한 주문 중지
* `order_exceeds_my_asset_values`: 주문이 내 자산을 초과함
* `order_already_closed`: 주문이 이미 체결됨
* `order_exceeded_void_rate`: 잘못된 주문의 비율이 제한을 초과해 10분간 주문 중지 (80%)
* `order_exceeded_max_orders`: 채결되지 않은 주문이 제한을 초과함 (100개)
* `order_violates_min_usd`: USD 환산 금액이 최소 주문량 미만임 ($10)
* `order_unplaceable_maker_only`: 주문이 `maker_only` 조건을 만족하지 않음
* `order_unplaceable_taker_only`: 주문이 `taker_only` 조건을 만족하지 않음

### Wdrl

* `wdrl_suspended_coin`: 코인의 출금 중지
* `wdrl_precision_error`: 소수점 자릿수의 정밀도가 잘못됨
* `wdrl_under_min_amount`: 출금 금액이 최소 출금 한도 미만임
* `wdrl_over_daily_wdrl_limit`: 출금이 하루 제한을 넘어섬
* `wdrl_exceeds_my_asset_values`: 출금이 내 자산보다 큼
* `wdrl_needs_to_tag`: `to_tag` 인자가 빠짐
* `wdrl_invalid_addr`: 잘못된 주소

# Action

API를 구독하면, 서버에서 오는 메시지는 데이터를 어떻게 처리하면 될지를 알려주는 `action`을 가지고 있습니다.

* `init` : 이전의 모든 로컬 데이터를 지우고, 현재 최신 데이터로 초기화합니다.
* `insert` : 응답받은 데이터를 로컬 데이터셋에 추가합니다.
* `update` : 로컬 데이터셋에서 일치하는 것이 존재하면 업데이트합니다.
* `upsert` : 로컬 데이터셋에서 일치하는 것이 존재하면 업데이트합니다. 없다면 추가합니다.
* `delete` : 로컬 데이터셋에서 일치하는 것이 존재하면 삭제합니다.

# Types

### Integer 

`integer` 타입.
 
* `123`
* `20181010`

### Decimal
정확한 정밀도로 표현하기 위해 `string`으로 표현된 10진수.

* `"880.524"`
* `"59.55000000000000000"`

### String

`string`.
 
* `"string"`
* `"bitcoin"`
* `"Daybit"`

 
### Boolean

`boolean`.
 
* `true`
* `false`

### Unix Timestamp

`millisecond` 단위의 Unix 타임스탬프.
 
* `1528269989516`

### CSV

각 값이 쉼표 _`,`_ 로 구분된 `string
 
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
| usd_price | decimal | USD exchanged price |

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
| high | decimal or null | Highest price |
| low | decimal or null | Lowest price |
| volatility | decimal | Price volatility |
| quote_vol | decimal or null | Trading volume per quote coin |
| base_vol | decimal or null | Trading volume per base coin |

### Order book

identifier: `quote`, `base`, `intvl`, `min_price`

| Name | Type | Description |
|---|---|---|
| quote | string | Quote coin |
| base | string | Base coin |
| intvl | decimal | Price interval unit |
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
| one_day_wdrl_usd_limit | decimal | Withdrawal limit for 24 hours in USD |

### Asset

identifier: `coin`

| Name | Type | Description |
|---|---|---|
| coin | string | Coin symbol |
| total | decimal | Total amount |
| reserved | decimal | Reserved amount for order and so on |
| available | decimal | Availalbe amount for order |
| investment_usd | decimal | Total investment in USD |

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
| cond_arg1 | decimal or null | First conditional order price value |
| cond_arg2 | decimal or null | Second conditional order price value |
| coin_fee | decimal | Fee |
| amount | decimal | Order amount |
| filled | decimal | Order filled (per base at selling) |
| filled_quote | decimal | Order filled (per quote at buying) |
| unfilled | decimal | Order unfilled |
| received_at | unix_timestamp | Order received time |
| placed_at | unix_timestamp or null | Order placed time (to order book) |
| closed_at | unix_timestamp or null | Order closed time |
| status | string | Order status (`"received"`, `"placed"`, `"closed"`) |
| close_type | string | Closing type (`"rejected"`, `"filled"`, `"canceled"`) |
| cancel_reason | string | Cancel reason (`"user"`, `"conflicting_self_order"`, `"expired"`, `"no_placed_amount"`, `"partially_filled_taker_only"`, `"maintenance"`, `"admin"`,) |

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
| exec_at | unix_timestamp | Execution time |
| Below are only available from [my_trades](#my_trades)  |
| order_id | integer | Trade order id |
| fee | decimal | Fee |
| sell | boolean | Selling (`true`) or buying (`false`) |
| counterpart | string | (`"user"`, `"daybit"`, `"project"`), The counterparty of the trade. `"daybit"` and `"project"` means Daybit exchange and another coin project team, respectively. |

### Transaction Summary

identifier: `id`

| Name | Type | Description |
|---|---|---|
| id | integer | id |
| coin | string | Transaction coin |
| type | string | Type of the transaction (`"wdrl"`, `"deposit"`) |
| amount | decimal | Transaction amount |
| txid | string or null | Withdrawal transaction id |
| confirm | integer | Confirm count |
| req_confirm | integer | |
| deposit_status | string | Deposit status |
| wdrl_status | string or null | Withdrawal status |
| wdrl_to_addr | string or null | Withdrawal receiving address |
| wdrl_to_tag | string or null | Withdrawal tag |
| wdrl_to_org | string or null | Withdrawal organization |
| created_at | unix_timestamp | Transaction created time |
| completed_at | unix_timestamp or null | Transaction completed time |
| tx_link_url | string ||


### Airdrop

identifier: `id`

| Name | Type | Description |
|---|---|---|
| id | integer | id |
| coin | string | coin symbol |
| amount | decimal | Airdrop amount |
| description | string | Airdrop description |
| airdropped_at | unix_timestamp | Airdrop time |
