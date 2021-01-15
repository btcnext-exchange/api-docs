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

The assets endpoint is to provide a detailed summary for each currency available on the exchange.

- Interface

```
Path: /api/v1/assets
Request type: GET
```

- Assets response descriptions:

| Name                     |  Type    | Description                                                          |
| ------------------------ |:--------:| ---------------------------------------------------------------------|
| name                     | string   | Full name of cryptocurrency                                          |
| can_withdraw             | boolean  | Identifies whether withdrawals are enabled or disabled               |
| can_deposit              | boolean  | Identifies whether deposits are enabled or disabled                  |
| min_withdraw             | decimal  | Identifies the single minimum withdrawal amount of a cryptocurrency  |
| max_withdraw             | decimal  | Identifies the single maximum withdrawal amount of a cryptocurrency  |
| unified_cryptoasset_id   | integer  | Unique ID of cryptocurrency assigned by Unified Cryptoasset ID.      |

- Examples of returned results:

````json
{
  "CNYQ": {
    "name": "cnyq",
    "can_withdraw": "true",
    "can_deposit": "true",
    "min_withdraw": "0.00000001",
    "max_withdraw": "100000000",
    "unified_cryptoasset_id": null
  },
  "XEM": {
    "name": "xem",
    "can_withdraw": "true",
    "can_deposit": "true",
    "min_withdraw": "0.00000001",
    "max_withdraw": "100000000",
    "unified_cryptoasset_id": 873
  },
  "PRIZM": {
    "name": "prizm",
    "can_withdraw": "true",
    "can_deposit": "true",
    "min_withdraw": "0.00000001",
    "max_withdraw": "100000000",
    "unified_cryptoasset_id": null
  }
}
````

---

#### <span id="summaryInfo">2. All summary information</span>

The summary endpoint is to provide an overview of market data for all tickers and all market pairs on the exchange.

- Interface

```
Path: /api/v1/summary
Request type: GET
```

- Summary response descriptions:

| Name                     |  Type    | Description                                                                                              |
| ------------------------ |:--------:| ---------------------------------------------------------------------------------------------------------|
| trading_pairs            | string   | Identifier of a ticker with delimiter to separate base/quote, eg. BTC-USD (Price of BTC is quoted in USD)|
| last_price               | decimal  | Last transacted price of base currency based on given quote currency                                     |
| lowest_ask               | decimal  | Lowest Ask price of base currency based on given quote currency                                          |
| highest_bid              | decimal  | Highest bid price of base currency based on given quote currency                                         |
| base_volume              | decimal  | 24-hr volume of market pair denoted in BASE currency                                                     |
| quote_volume             | decimal  | 24-hr volume of market pair denoted in QUOTE currency                                                    |
| price_change_percent_24h | decimal  | 24-hr volume of market pair denoted in QUOTE currency                                                    |
| highest_price_24h        | decimal  | Highest price of base currency based on given quote currency in the last 24-hrs                          |
| lowest_price_24h         | decimal  | Lowest price of base currency based on given quote currency in the last 24-hrs                           |
| isFrozen                 | integer  | Indicates if the market is currently enabled (0) or disabled (1)                                         |

- Examples of returned results:

````json
{
  "code": 200,
  "msg": "success",
  "data": {
    "NEO_BTC": {
      "trading_pairs": "NEO_BTC",
      "last_price": 0.00065261,
      "lowest_ask": 0.00067908,
      "highest_bid": 0.00063344,
      "base_volume": 45416.0442629,
      "quote_volume": 27.08912357,
      "price_change_percent_24h": 0.10083835,
      "highest_price_24h": 0.00066065,
      "lowest_price_24h": 0.00057284,
      "isFrozen": 0
    },
    "DASH_USDT": {
      "trading_pairs": "DASH_USDT",
      "last_price": 132.10311575,
      "lowest_ask": 131.56808669,
      "highest_bid": 130.36191331,
      "base_volume": 12435.3215763,
      "quote_volume": 1643753.67962894,
      "price_change_percent_24h": -0.02291988,
      "highest_price_24h": 138.51048635,
      "lowest_price_24h": 127.76041843,
      "isFrozen": 0
    },
    "QDEFI_USDC": {
      "trading_pairs": "QDEFI_USDC",
      "last_price": 530.45619326,
      "lowest_ask": 535.74514215,
      "highest_bid": 515.00963439,
      "base_volume": 1576.9987895,
      "quote_volume": 851803.99399242,
      "price_change_percent_24h": -0.00486689,
      "highest_price_24h": 554.77324889,
      "lowest_price_24h": 522.21109338,
      "isFrozen": 0
    }
  }
}
````

---

#### <span id="currentLatest">3. Current latest market</span>

Get the current latest market:

- Interface

```
Path: /api/v1/tickers
Request type: GET
```

- Ticker response descriptions:

| Name                     |  Type    | Description                                                          |
| ------------------------ |:--------:| ---------------------------------------------------------------------|
| base_id                  | integer  | The quote pair Unified Cryptoasset ID                                |
| quote_id                 | integer  | The base pair Unified Cryptoasset ID.                                |
| last_price               | decimal  | Last transacted price of base currency based on given quote currency |
| base_volume              | decimal  | 24-hour trading volume denoted in BASE currency                      |
| quote_volume             | decimal  | 24 hour trading volume denoted in QUOTE currency                     |
| isFrozen                 | integer  | Indicates if the market is currently enabled (0) or disabled (1)     |
| base_name                | string   | Base name                                                            |
| quote_name               | string   | Quote name                                                           |

- Examples of returned results:

````json
{
  "NEO_BTC": {
    "base_id": 1376,
    "quote_id": 1,
    "last_price": 0.00065735,
    "base_volume": 45293.4394285,
    "quote_volume": 27.15526656,
    "isFrozen": 0,
    "base_name": "NEO",
    "quote_name": "BTC"
  },
  "DASH_USDT": {
    "base_id": 131,
    "quote_id": 825,
    "last_price": 128.05695508,
    "base_volume": 12434.7417591,
    "quote_volume": 1641391.38344901,
    "isFrozen": 0,
    "base_name": "DASH",
    "quote_name": "USDT"
  },
  "QDEFI_USDC": {
    "base_id": null,
    "quote_id": 3408,
    "last_price": 530.45619326,
    "base_volume": 1576.9987895,
    "quote_volume": 851803.99399242,
    "isFrozen": 0,
    "base_name": "QDEFI",
    "quote_name": "USDC"
  }
}
````

---

#### <span id="recentMarket">4. Recent market transactions</span>

The order book endpoint is to provide a complete level 2 order book (arranged by best asks/bids) with full depth
returned for a given market pair.

- Interface

```
Path: /api/v1/orderbook?market_pair=
Request type: GET
```

- Parameter

```
btc_usdt
```

- Order book response descriptions:

| Name      |  Type                        | Description                                                                     |
| ----------|:----------------------------:| --------------------------------------------------------------------------------|
| timestamp | Integer (UTC timestamp in ms)| Unix timestamp in milliseconds for when the last updated time occurred.         |
| bids      | decimal                      | An array containing 2 elements. The offer price and quantity for each bid order.|
| asks      | decimal                      | An array containing 2 elements. The ask price and quantity for each ask order.  |


- Examples of returned results:

````json
{
  "name": "btc_usdt",
  "timestamp": "1610723505",
  "bids": [
    [
      "36523.34",
      "0.03800598"
    ],
    [
      "36485.46312268",
      "0.02243572"
    ],
    [
      "36459.43788744",
      "0.02168365"
    ]
  ],
  "asks": [
    [
      "36567.02",
      "0.03618596"
    ],
    [
      "36593.04523524",
      "0.03434187"
    ],
    [
      "36621.11",
      "0.03356017"
    ]
  ]
}
````

---

#### <span id="recentTrades">5. Recent trades</span>
The trades endpoint is to return data on all recently completed trades for a given market pair.

| Name        |  Type    | Description            |
| ------------|:--------:|------------------------|
| market_pair | string   | A pair such as btc_usdt|

- Interface
```
Path: /api/v1/trades?market_pair=
```

- Parameter

```
btc_usdt
eth_usdt
```

- Trades response descriptions:
  
| Name                     |  Type                        | Description                                                                                                                                                                                       |
| ------------------------ |:----------------------------:| --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| trade_id                 | integer                      | A unique ID associated with the trade for the currency pair transaction (Note: Unix timestamp does not qualify as trade_id)                                                                       |
| price                    | decimal                      | Last transacted price of base currency based on given quote currency                                                                                                                              |
| base_volume              | decimal                      | Transaction amount in BASE currency                                                                                                                                                               |
| quote_volume             | decimal                      | Transaction amount in QUOTE currency                                                                                                                                                              |
| trade_timestamp          | decimal (UTC timestamp in ms)| Unix timestamp in milliseconds for when the transaction occurred.                                                                                                                                 |
| type                     | string                       | Used to determine whether or not the transaction originated as a buy or sell. (Buy – Identifies an ask was removed from the order book. Sell – Identifies a bid was removed from the order book)  |


- Examples of returned results:

```json
[
  {
    "trade_id": 1440237,
    "price": 39881.21019433,
    "base_volume": 0.01,
    "quote_volume": 398.8121019433,
    "trade_timestamp": 1610638460,
    "type": "sell"
  },
  {
    "trade_id": 1440238,
    "price": 39881.01436764,
    "base_volume": 0.259,
    "quote_volume": 10329.18272121876,
    "trade_timestamp": 1610638481,
    "type": "sell"
  },
  {
    "trade_id": 1440239,
    "price": 39879.31844183,
    "base_volume": 0.3506194,
    "quote_volume": 13982.462704483369,
    "trade_timestamp": 1610638525,
    "type": "buy"
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
