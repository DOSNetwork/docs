## Setup
- Besides consuming data streams, DOS Network can connect smart contracts with any external API / data by utilizing [DOSOnChainSDK](https://github.com/DOSNetwork/eth-contracts/blob/master/contracts/DOSOnChainSDK.sol) contract for each supported chain. For each data request or VRF randomness request, **20 DOS token** is needed (the number may change depending on usage cost).

- In the calling contract's constructor, simply setup by calling `super.DOSSetup()`.

- Sending several DOS tokens as fees into the calling contract. The fees can be reclaimed by contract owner by calling `DOSRefund()`.

<!-- tabs:start -->

#### **Example Setup**
```js
contract Example is DOSOnChainSDK {
  // ...
  constructor() public {
    // @dev: setup and then transfer DOS tokens into deployed contract as oracle fees.
    // Unused fees can be reclaimed by calling DOSRefund() in the SDK.
    super.DOSSetup();
  }
  // ...
}
```

<!-- tabs:end -->


## Request Any Data
- Smart contract requests for external data by calling `DOSQuery()` function. The whole process is an asynchronous one - i.e. it merely returns a unique `queryId` that caller caches for bookkeeping and future identification, with the real response coming back through the `__callback__()` function.
- The response data will be backfilled through the `__callback__` function along with corresponding `queryId`. Instead of backfilling the whole raw response we're using [selector expression](#selector) to filter `json` and `xml/html` formated response, developers are able to specify interesting data fields in `DOSQuery()` function.
- Example usage:

<!-- tabs:start -->

#### **DOSQuery() API**
`function DOSQuery(uint timeout, string memory dataSource, string memory selector)`:
- `timeout`: An estimated timeout in seconds specified by the caller, e.g. `30`. Response is not guaranteed if client side processing time exceeds this value.
- `dataSource`: Path to the data source specified by caller.
- `selector`: A `selector expression` provided by caller to filter out specific data fields out of the raw response, with the response data format (json, xml, or more) to be identified from the selector expression. Check the [selector expression](#selector) part for details.
- Example usage:
```js
contract Example is DOSOnChainSDK {
    mapping(uint => bool) private _valid_queries;
    ...
    function CoinbaseEthPriceFeed() public {
        // Returns a unique queryId that caller caches for future verification
        uint queryId = DOSQuery(30, "https://api.coinbase.com/v2/prices/ETH-USD/spot", "$.data.amount");
        _valid_queries[queryId] = true;
        ...
    }
    ...
}
```

#### **__callback__() API**

<p>* Note that the caller must override `__callback__` function to receive and process the response. A user-defined event may be added to notify the Dapp frontend that the response is ready.</p>

`function __callback__(uint queryId, bytes calldata result)`:
- `queryId`: A unique `queryId` returned by `DOSQuery()` to differenciate parallel responses.
- `result`: Corresponding response in `bytes`.
- Example usage:
```js
function __callback__(uint queryId, bytes calldata result) external auth {
    // Check whether @queryId corresponds to a previous cached one
    require(_valid_queries[queryId], "Err-resp-with-invalid-queryId");

    // Deal with result
    ...
    delete _valid_queries[queryId];
}
```
<!-- tabs:end -->

- `DOSOnChainSDK` also provides built-in utility functions for developers to easily process `string / bytes`: 
```js
  library utils {
      function subStr(string a, uint start, uint len) internal pure returns(string);
      function subStr(string a, uint start) internal pure returns(string);
      function subStr(bytes a, uint start, uint len) internal pure returns(bytes);
      function subStr(bytes a, uint start) internal pure returns(bytes);
      function indexOf(string haystack, string needle) internal pure returns(int);
      function indexOf(bytes haystack, bytes needle) internal pure returns(int);
      function strConcat(string a, string b) internal pure returns(string);
      function bytesConcat(bytes a, bytes b) internal pure returns(bytes);
      function strCompare(string a, string b) internal pure returns(int);
      function bytesCompare(bytes a, bytes b) internal pure returns(int);
      function strEqual(string a, string b) internal pure returns(bool);
      function bytesEqual(bytes a, bytes b) internal pure returns(bool);

      function uint2Str(uint x) internal pure returns(string);
      function uint2HexStr(uint x) internal pure returns(string);
      function addr2Str(address x) internal pure returns(string);
      function str2Addr(string a) internal pure returns(address);
      function str2Uint(string a) internal pure returns(uint);
      function hexStr2Uint(string a) internal pure returns(uint);
      function byte2Uint(byte b) internal pure returns(uint);
      function hexByte2Uint(byte b) internal pure returns(uint);
  }

  contract DOSOnChainSDK {
      using utils for *;
      // ...
  }
```


## Request VRF Randomness
- Randomness is particularly important for many Dapps and games, however, it's impossible to generate a secure and unpredictable random number in pure deterministic environment:
<p align="center">
  <img width="500" height="250" src="https://raw.githubusercontent.com/DOSNetwork/docs/master/_media/random.png">
</p>

- DOS Network is providing a provably secure, unstoppable and unpredictable random source for on-chain smart contracts to use. For technical details and cryptographic proofs please check our [whitepaper](#).

<!-- tabs:start -->

#### **DOSRadom() API**
`function DOSRandom(uint seed) returns (uint)`:
- `seed`: An *optional* random seed provided by caller to get more entropy. The generated random number is secure and unpredictable in safe mode even without providing this `seed`.
- Example usage:
```js
function requestSafeRandom() public {
    uint requestId = DOSRandom(now);
    _valid[requestId] = true;
    emit RandomRequested(requestId);
}
```

#### **__callback__() API**
`function __callback__(uint requestId, uint generatedRandom) external`:
- `requestId`: A unique `requestId` returned by `DOSRandom()` to process parallelly generated random numbers.
- `generatedRandom`: Generated secure random number for the specific `requestId`.
- Example usage:

```js
modifier auth(uint id) {
    // Exclude malicious callback responses.
    require(msg.sender == fromDOSProxyContract(),
            "Unauthenticated response from non-DOS.");
    // Check whether id mapps to a previously requested one.
    require(_valid[id], "Response with invalid request id!");
    _;
}

function __callback__(uint requestId, uint generatedRandom)
    external
    auth(requestId)
{
    emit RandomGenerated(generatedRandom);
    delete _valid[requestId];

    // Deal with generated random number
    random = generatedRandom;
    ...
}
```

<!-- tabs:end -->


