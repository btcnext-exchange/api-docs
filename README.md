### Exchange https://www.btcnext.io/

## API Explanation

- [Introduction](#Introduction)
- [Trading API ver. 0.1.3](#tradingApi)
- [Public](#public)
    - [Supported instruments](supportedInstruments)
    - [Order book snapshot](#orderbookSnapshot)
    - [Real-time order book data](#realtimeOrderBook)
    - [Methods](#methods)
        - [Assets data](#assetsData)
        - [All summary information](#summaryInfo)
        - [Current latest market](#currentLatest)
        - [Recent market transactions](#recentMarket)
        - [Recent trades](#recentTrades)
- [Private API](#privateApi)
    - [Place the order](#placeOrder)
    - [Cancel the order](#cancelOrder)
    - [Historical orders](#historicalOrder)
    - [Trading history](#tradingHistory)
    - [Real-time balances and orders data](#realTimeBalances)
    - [How to subscribe on private channel using websocket](#howtoSubscribe)
- [Errors](#errors)

---

## <span id="Introduction">Introduction</span>

### <span id="tradingApi">Trading API ver. 0.1.3</span>

#### General information

The base endpoint for REST requests is: https://api.btcnext.io:8443/trading

---

## <span id="public">Public</span>

### <span id="supportedInstruments"> Supported instruments </span>

```
GET /frontoffice/api/info
```

Response:

```
{
    "serverTime": 636880696809972200,
    "pairs": {
        "btc_usdt": {
            "baseAsset": "btc",
            "quoteAsset": "usdt",
            "minPrice": 0,
            "maxPrice": 0,
            "minAmount": 0,
            "hidden": 0,
            "fee": 0,
            "makerFee": 0,
            "makerFeeLimit": 0,
            "takerFee": 0.001,
            "takerFeeLimit": 0,
            "priceScale": 6,
            "amountScale": 6,
            "createdAt": "2019-11-14T16:18:49.253354",
            "updatedAt": "2019-11-14T16:18:49.253354"
        },
        ...
        "gnt_usdt": {
            "baseAsset": "gnt",
            "quoteAsset": "usdt",
            "minPrice": 0,
            "maxPrice": 0,
            "minAmount": 0,
            "hidden": 0,
            "fee": 0,
            "makerFee": 0,
            "makerFeeLimit": 0,
            "takerFee": 0.001,
            "takerFeeLimit": 0,
            "priceScale": 5,
            "amountScale": 5,
            "createdAt": "2019-11-14T16:18:49.253354",
            "updatedAt": "2019-11-14T16:18:49.253354"
        }
    }
}
```

### <span id="orderbookSnapshot">Order book snapshot</span>

``
GET marketdata/instruments/{instrument}/depth
``
Response:

```
{
    "instrument": "eth_btc",
    "bids": [
        {
            "amount": 0.3092258,
            "price": 0.01734264
        },
        {
            "amount": 51.61494099,
            "price": 0.01734363
        }
        ...
    ],
    "asks": [
        {
            "amount": 133.52370356,
            "price": 0.01739337
        },
        {
            "amount": 9.16854518,
            "price": 0.01739838
        }
        ...
    ],
    "version": 1891724,
    "askTotalAmount": 1849.11363582,
    "bidTotalAmount": 809.23878372,
    "snapshot": true
}

```

---

### <span id="realtimeOrderBook">Real-time order book data</span>

Open connection with websocket:

```
https://api.btcnext.io:8443/trading/marketdata/info
```

```
const WebSocket = require('ws')
const ws = new WebSocket(`${baseUrl}/marketdata/info`)

ws.on('open', () => {
    ws.send('{"protocol":"json","version":1}')
    ws.send('{"arguments":["btc_usdt"],"invocationId":"0","streamIds":[],"target":"Book","type":4}')
});

ws.on('message', msg => {
    // 
});
```

Update:

```
{
    "type": 2,
    "invocationId": "1",
    "item": {
        "instrument": "btc_usdt",
        "bids": [
            {
                "amount": 1.3393,
                "price": 8348.98
            }
            ...
        ],
        "asks": [
            {
                "amount": 0,
                "price": 8394.84
            }
            ...
        ],
        "version": 367398,
        "askTotalAmount": 115.24421295,
        "bidTotalAmount": 108.16757362,
        "snapshot": false
    }
}
```

---

### <span id="methods">Methods:</span>

#### <span id="assetsData">1. Assets data</span>

- Interface

```
Path: /api/v1/assets
Request type: GET
```

- Parameter
```
no
```

- Return result
```
name - asset abbreviation
can_withdraw - "true" or "false", if true, the asset can be withdrawn
can_deposit - "true" or "false", if true, the asset can be deposited
min_withdraw - minimum withdrawal/deposit amount
max_withdraw - maximum withdrawal/deposit amount
```

- Examples of returned results:
```
 {
  "ETH": {
    "name": "eth",
    "can_withdraw": "true",
    "can_deposit": "true",
    "min_withdraw": "0.00000001",
    "max_withdraw": "100000000"
  },
  
  "NOAH ARK": {
    "name": "noah ark",
    "can_withdraw": "false",
    "can_deposit": "false",
    "min_withdraw": "0.00000001",
    "max_withdraw": "100000000"
  }
}
```

---

#### <span id="summaryInfo">2. All summary information</span>

- Interface

```
Path: /api/v1/summary
Request type: GET
```
- Parameter
```
no
```

- Return result
```
id - exchange pairs tickers
last - final transaction price
lowestAsk - current lowest bid price
highestBid - current highest bid price
percentChange - percentage of price change
base_volume - average price over last 24h
quote_volume - total 24h traded quote cost
isFrozen - frozen status: 0 — working, 1 — temporarily suspended
high24hr - highest price in the last 24 hours
low24hr - lowest price in the last 24 hours
```

- Examples of returned results:
```
{
  "code": 200,
  "msg": "success",
  "data": {
    "DOGE_BTC": {
      "id": "doge_btc",
      "last": "0.00000023",
      "lowestAsk": "0.00000025",
      "highestBid": "0.00000022",
      "percentChange": "0",
      "baseVolume": "238882079.6813492",
      "quoteVolume": "57.05787792",
      "isFrozen": "0",
      "high24hr": "0.00000026",
      "low24hr": "0.00000023"
    }
}
```
---

#### <span id="currentLatest">3. Current latest market</span>

Get the current latest market:

- Interface

```
Path: /api/v1/tickers
Request type: GET
```

- Parameter

```
no
```

- Return result

```
base_name - base name
quote_name - quote name
last_price - trade price in the last 24 hours
base_volume - average price over last 24h
quote_volume - total 24h traded quote cost
isFrozen - frozen status: 0 — working, 1 — temporarily suspended
```

Examples of returned results:

```
{
    "DOGE_BTC": {
        "base_name": "DOGE",
        "quote_name": "BTC",
        "last_price": "0.00000023",
        "base_volume": "238882079.6813492",
        "quote_volume": "57.05787792",
        "isFrozen": "0"
    }
}
  
```
---

#### <span id="recentMarket">4. Recent market transactions</span>

- Interface

```
Path: /api/v1/orderbook?market_pair=
Request type: GET
```

- Parameter

```
btc_usdt
```

- Return result

```
name - pair tickers
timestamp - unix timestamp(s) of trade
bids - number fields amount and price
```

- Examples of returned results:

```
{
  "name": "btc_usdt",
  "timestamp": "1610485860",
  "bids": [
    [
      "34330.35",
      "0.02968251"
    ],
    [
      "34290.49",
      "0.03815408"
    ],
    [
      "34249.74905904",
      "0.02947742"
    ],
}
```

---

#### <span id="recentTrades">5. Recent trades</span>

 - Interface

```
Path: /api/v1/trades?market_pair=
```

  - Parameter

```
btc_usdt
eth_usdt
```

- Return result

```
tradeID - order identifier
price - trade price
base_volume - amount in base currency
quote_volume - amount in quote currency
trade_timestamp - unix timestamp(s) of trade
type - transaction type "buy" or "sell"
```

 Examples of returned results:

```
[
  {
    "tradeID": "1431582",
    "price": "31866.359002",
    "base_volume": "0.0001",
    "quote_volume": "3.1866359002",
    "trade_timestamp": "1610397055",
    "type": "buy"
  },
  {
    "tradeID": "1431583",
    "price": "31969.87999449",
    "base_volume": "0.116",
    "quote_volume": "3708.50607936084",
    "trade_timestamp": "1610397125",
    "type": "sell"
  }
]
```
---

## <span id="privateApi">Private API</span>

Notice

- All REST requests should contain 2 HTTP header fields:
    - "Key": public_key
    - "Sign": hash of payload
- For hashing, the HMAC SHA512 algorithm is used.
- API keys are UUID string(UTF-8). Example: "ca3a03e1-fc5c-4954-99dc-876db3997d8f".
- The ts parameter contains a string representation of the current time in UTC datetime (example: "2019-12-20T08: 27:
  51").
- The POST requests must include the required ts field in body. The payload is a JSON body:

```
const axios = require('axios')
const hmacSHA512 = require('crypto-js/hmac-sha512')

// keys
const publicKey = '06879020-2476-49f4-89ef-617004328120' 
const privateKey = 'ca3a03e1-fc5c-4954-99dc-876db3997d8f'

const ts = new Date().toISOString()

const order = {
    order: {
        instrument: 'btc_usdt',
        type: 'sell',
        amount: 1,
        price: 1,
        isLimit: true,
        isStop: false,
    },
    ts: ts,
}

const signature = hmacSHA512(JSON.stringify(order), privateKey).toString().toUpperCase()
axios.post('https://api.btcnext.io:8443/trading/frontoffice/api/order', order, {
    headers: {
        Key: publicKey,
        Sign: signature
    }
}).then(() => {
    //
})
```

---

### <span id="placeOrder">Place the order</span>

```
POST /frontoffice/api/order
```

Headers:

```
Key  : ca3a03e1-fc5c-4954-99dc-876db3997d8f
Sign : 61DB64474F54891C434B4AEEBB8D90584E4FE4B597BCE13BB0F8C3E3862ACB08658CED4A4AC819BA89FD9B0305935A2D8623D6000441E93C0F1A3899A5B401D0
```

Request body:

```
{
    "ts": "2019-12-12T01:01:01",
    "order": {
        "instrument": "btc_usdt",
        "type": "sell",
        "amount": 1,
        "price": 1,
        "isLimit": true,
        "isStop": false,
        "activationPrice": 0
    }
}
```

Response:

```
{
    "order": {
        "orderId": "-72057594037927935",
        "total": 0.0,
        "orderType": 0,
        "commission": 0.0,
        "createdAt": "2019-03-13T12:01:32.8880912Z",
        "unitsFilled": 0.0,
        "isPending": true,
        "status": "Working",
        "type": "sell",
        "amount": 1.0,
        "price": 1.0,
        "isLimit": true,
        "instrument": "btc_usdt"
    }
}
```

---

### <span id="cancelOrder">Cancel the order</span>

```
DELETE /frontoffice/api/orders/{orderId}
```

Headers:

```
Key : ca3a03e1-fc5c-4954-99dc-876db3997d8f
Sign : 61DB64474F54891C434B4AEEBB8D90584E4FE4B597BCE13BB0F8C3E3862ACB08658CED4A4AC819BA89FD9B0305935A2D8623D6000441E93C0F1A3899A5B401D0
```

```
const signature = hmacSHA512(`?ts=${ts}`, privateKey)

axios.delete(`${baseUrl}/frontoffice/api/orders/${orderId}?ts=${ts}`, {
    headers: {
        Key: publicKey,
        Sign: signature
    }
}).then(() => {
    //
})
```

Response:

```
{
  "order": {
    "orderId": "-1441151880758294523",
    "total": 0,
    "orderType": 0,
    "commission": 0,
    "createdAt": "2020-07-10T08:17:14.7622962Z",
    "unitsFilled": 0,
    "isPending": false,
    "status": "cancelled",
    "type": "sell",
    "amount": 250,
    "remaining": 250,
    "price": 9405,
    "stopPrice": 0,
    "isLimit": true,
    "instrument": "btc_usdt",
    "side": 1
  }
}
```

---

### <span id="historicalOrder">Historical orders</span>

```
GET /frontoffice/api/order_history
```

Possible URL parameters:

- market - market filter (optional)
- side - order side buy/sell (optional)
- status - order status working/rejected/cancelled/completed (optional)
- startDate (optional)
- endDate (optional)

Request URL:

```
GET /frontoffice/api/order_history?market=btc_usdt&ts=2019-12-12T01:01:01
```

Headers:

```
Key : ca3a03e1-fc5c-4954-99dc-876db3997d8f
Sign: 61DB64474F54891C434B4AEEBB8D90584E4FE4B597BCE13BB0F8C3E3862ACB08658CED4A4AC819BA89FD9B0305935A2D8623D6000441E93C0F1A3899A5B401D0
```

```
const params = {
    ts,
    market: 'btc_usdt'
}

const signature = hmacSHA512(`?ts=${params.ts}&market=${params.market}`, privateKey)

axios.get(`${baseUrl}/frontoffice/api/order_history`, {
    params,
    headers: {
        Key: publicKey,
        Sign: signature
    }
}).then(v => {
    //    
})
```

Response:

```
{
    "filters": {},
    "paging": {
        "page": 1,
        "per_page": 15,
        "total": 0
    }
    "data": [
        {
            "orderId": "-72057594037927933",
            "total": 1.0,
            "orderType": 0,
            "commission": 0.0,
            "createdAt": "2019-03-13T08:24:50.578088Z",
            "unitsFilled": 1.0,
            "isPending": false,
            "status": "Completed",
            "type": "sell",
            "amount": 1.0,
            "price": 1.0,
            "isLimit": true,
            "instrument": "btc_usdt"
        },
        ...
        {
            "orderId": "-72057594037927935",
            "total": 0.0,
            "orderType": 0,
            "commission": 0.0,
            "createdAt": "2019-03-13T07:54:06.4232625Z",
            "unitsFilled": 0.0,
            "isPending": false,
            "status": "Cancelled",
            "type": "sell",
            "amount": 1.0,
            "price": 1.0,
            "isLimit": true,
            "instrument": "btc_usdt"
        }
    ]
}

```

---

### <span id="tradingHistory">Trading history</span>

```
GET /frontoffice/api/trade_history&ts=2019-12-12T01:01:01 Possible URL parameters:
```

Possible URL parameters:

- market - market filter (optional)
- side - order side buy/sell (optional)
- startDate (optional)
- endDate (optional)

Request URL

```
GET /frontoffice/api/trade_history?market=btc_usdt
```

Headers:

```
Key : ca3a03e1-fc5c-4954-99dc-876db3997d8f
Sign : 61DB64474F54891C434B4AEEBB8D90584E4FE4B597BCE13BB0F8C3E3862ACB08658CED4A4AC819BA89FD9B0305935A2D8623D6000441E93C0F1A3899A5B401D0
```

```
axios.get(`${baseUrl}/frontoffice/api/trade_history`, {
    params,
    headers: {
        Key: publicKey,
        Sign: signature
    }
}).then(v => {
    //
})
```

Response:

```
{
    "filters": {
        "market": "btc_usdt"
    },
    "paging": {
        "page": 1,
        "per_page": 15,
        "total": 0
    },
    "data": [
        {
            "tradeSeq": 0,
            "tradeTime": "2019-12-20T06:17:03.093597",
            "amount": 0.00000001,
            "executionPrice": 0.00000001,
            "instrument": "btc_usdt",
            "side": 0,
            "commission": 0.00000000
        },
        {
            "tradeSeq": 3927,
            "tradeTime": "2019-12-20T06:17:03.093597",
            "amount": 0.00000001,
            "executionPrice": 0.00000001,
            "instrument": "btc_usdt",
            "side": 1,
            "commission": 0.00000000
        },
        ...
    ]
}
```

---

### <span id="realTimeBalances">Real-time balances and orders data</span>

Open connection with private websocket

```
https://api.btcnext.io:8443/trading/frontoffice/ws/account
```

Open orders channel name is "OpenOrders".

Balance updates channel name is "Balance". Headers:

```
Key : 06879020-2476-49f4-89ef-617004328120
Sign : 61DB64474F54891C434B4AEEBB8D90584E4FE4B597BCE13BB0F8C3E3862ACB08658CED4A4AC819BA89FD9B0305935A2D8623D6000441E93C0F1A3899A5B401D0
Payload : 2019-12-12T01:01:01
```

---

### <span id="howtoSubscribe">How to subscribe on private channel using websocket</span>

```
const signature = hmacSHA512(ts, privateKey).toString().toUpperCase()

const headers = {
    Key: publicKey,
    Sign: signature,
    Payload: ts
};

const ws = new WebSocket(`${baseUrl}/frontoffice/ws/account`, {
    headers
});

ws.on('open', () => {
    ws.send('{"protocol":"json","version":1}')
    ws.send('{"arguments":[],"invocationId":"0","streamIds":[],"target":"OpenOrders","type":4}')
});

ws.on('message', msg => {
    //    
});
```

---

## <span id="errors">Errors</span>

1. Requests with a time (ts field) different from the current time by more than 5 seconds will be returned with an error
    401.

2. Response to the request the message will arrive:

```
GET /frontoffice/api/info
```

```
500 ​Internal Server Error
{
    "errors": [
        {
            "code": "unexpected",
            "message": "Stop orders are not supported."
        }
    ]
}

```

3.If the system is overloaded you will receive:

```
503 ​(Sevice Unavailable)
{
    "errors": [
        {
            "code": ​ ""​,
            "message": ​ "System is currently overloaded"
        }
    ]
}
```
---
