# **Advanced**

This section provides helpful information for the developers who are interested in implementing Exchange API features in low level and want to have total control of using the API.

## Topic

Exchange API has basically two layers of [Topic](https://hexdocs.pm/phoenix/channels.html#topics)s - [`/api`](#topic-api) and [`/subscription:<sub_topic>`](#topic-subscription) - and expects certain events to properly handle the request.

## Event

You need to send `phx_join` event message to join the channel before sending necessary events if you decided to access the API without using the wrapper. If there was no problem of joining the topic, you will receive `phx_reply` event with `"status":"ok"` payload from the socket connection. If you reached this step and confirmed successful join, you can send events specified in this document and complete your task.

* `/api` : In `/api` topic, API expects following events - `get_server_time`, `create_order`, `cancel_order`, `cancel_orders`, `cancel_all_my_orders`, or `create_wdrl`.

* `/subscription:<sub_topic>` : In `/subscription` topic, you should state `<sub_topic>` to specify your purpose of calling the API. After you successfully join the `/subscription:<sub_topic>` topic by sending `phx_join` event, you need to send `request` event and let the API know you want to get notification regarding the `<sub_topic>`. The notifications from API contains `notification` event with proper `action` (one of `insert`, `update`, `upsert`, or `delete`) for you to update the data set, if necessary.

## Timestamp

All request takes `timestamp` and `timeout` to prevent unexpected calls because of network delay and so on.

- `timestamp`: `unix_timestamp`
- `timeout`(**optional**): `ms` unit time in `integer`. Default value is `3000`.

Request will be rejected in next condition: `server time` - `timestamp` > `timeout`.

If there was a problem, below error_code will be returned in response.
- `api_invalid_timestamp_or_timeout`: `timestamp` and/or `timeout` are not existed or they are not `integer` (unix timestamp in millisecond).
- `api_timeout`: Rejected because of request time out.

## Rate limit

Each API has limit of calls for every second. You will get `api_exceeded_rate_limit` error_code in response if you exceed the limit.


