## Quick Start
### Retrieving external data:
- Smart contract requests for external data by calling `DOSQuery()` function. The whole process is an asynchronous one - i.e. it merely returns a unique `queryId` that caller caches for bookkeeping and future identification, with the real response coming back through the `__callback__()` function.
- The response data will be backfilled through the `__callback__` function along with corresponding `queryId`. Instead of backfilling the whole raw response we're using [selector expression](#selector) to filter `json` and `xml/html` formated response, developers are able to specify interesting data fields in `DOSQuery()` function.
- Example usage:

<!-- tabs:start -->
#### **DOSQuery() API**
`function DOSQuery(uint timeout, string dataSource, string selector) returns (uint)`:
- `timeout`: An estimated timeout in seconds specified by the caller, e.g. `30`. Response is not guaranteed if client side processing time exceeds this value.
- `dataSource`: Path to the data source specified by caller.
- `selector`: A `selector expression` provided by caller to filter out specific data fields out of the raw response, with the response data format (json, xml, or more) to be identified from the selector expression. Check the [selector expression](#selector) part for details.
- Example usage:
```solidity
contract Example is Ownable, DOSOnChainSDK {
    mapping(uint => bool) private _valid_queries;
    ...
    function fetchCoinbaseEthUsd() public onlyOwner {
        // Returns a unique queryId that caller caches for future verification
        uint queryId = DOSQuery(30, "https://api.coinbase.com/v2/prices/ETH-USD/spot", "$.data.amount");
        _valid_queries[queryId] = true;
        ...
    }
    ...
}
```

#### **\_\_callback\_\_() API**
<p>Note that the caller must override `__callback__` function to receive and process the response. A user-defined event may be added to notify the Dapp frontend that the response is ready.</p>
`function __callback__(uint queryId, bytes result) external`:
- `queryId`: A unique `queryId` returned by `DOSQuery()` to differenciate parallel responses.
- `result`: Corresponding response in `bytes`.
- Example usage:
```solidity
function __callback__(uint queryId, bytes result) external {
    // Exclude malicious callback responses.
    require(msg.sender == fromDOSProxyContract());
    // Check whether @queryId corresponds to a previous cached one
    require(_valid_queries[queryId], "Response with invalid queryId!");

    // Deal with result
    ...
    delete _valid_queries[queryId];
}
```
<!-- tabs:end -->

- `DOSOnChainSDK` also provides built-in utility functions for developers to easily process `string / bytes`: 
```solidity
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


### Requesting for secure and unpredictable random numbers:
- Randomness is particularly important for many Dapps and games, however, it's impossible to generate a secure and unpredictable random number in pure deterministic environment: ![peter\_szilagyi](../../_media/random.png)
- DOS Network is providing a provably secure, unstoppable and unpreditable random source for on-chain smart contracts to use. For technical details and cryptographic proofs please check our [whitepaper]().

<!-- tabs:start -->

#### **DOSRadom() API**
`function DOSRandom(uint8 mode, uint seed) returns (uint)`:
- `mode`: Currently we support 2 modes:
  - `1 (safe mode)`: An asynchronous but safe way to generate a new secure random number using *VRF and Threshold Signature* by a randomly selected group of off-chain clients. The newly generated secure random number is the threshold signature of the collectively signed message `(requestId || last round's secure randomness selecting the group || seed)`. Like queries, `DOSRandom()` returns a unique `requestId` for safe mode and the generated secure random number will be backfilled through the *(overloaded)* `__callback__()` function. There would be a fee to run in safe mode in future mainnet release.
  - `0 (fast mode)`: Return an *insecure* random number directly, which is the sha3 hash of `(last round's secure random number || seed)`. Note that `fast mode` is for testing purpose only and it should NOT be deemed as safe in production usage. It's always free of charge.
- `seed`: An *optional* random seed provided by caller to get more entropy. The generated random number is secure and unpreditable in safe mode even without providing this `seed`.
- Example usage:
```solidity
function requestSafeRandom() public {
    uint requestId = DOSRandom(1, now);
    _valid[requestId] = true;
    emit RandomRequested(requestId);
}
```

#### **\_\_callback\_\_() API**
`function __callback__(uint requestId, uint generatedRandom) external`:
- `requestId`: A unique `requestId` returned by `DOSRandom()` to process parallelly generated random numbers.
- `generatedRandom`: Generated secure random number for the specific `requestId`.
- Example usage:

```solidity
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



## Deployed Contracts on testnet
* [DOSAddressBridge]() - A connector contains all system contratcs' addresses, simply for *upgradable* contract development pattern. *(Neither developers nor node runners need to worry about this contract.)*
* [DOSOnChainSDK]() - APIs smart contract developers can take advantage of to request external off-chain data and leverage off-chain verifiable computation power for computation-intensive tasks. **(Developers need to pay attention to this one and see below section for details.)**
* [DOSProxy]() - Conceals implementation details such as request handling, random group selection, threshold signature verification, user-defined callback function invocation, response parsing, etc. *(Neither developers nor node runners need to worry about this contract.)*
* [DOSPayment]() - To be released in Beta **(Node runners need to pay attention to this one and see below section for details.)** 
* [DOSRegistry]() - To be released in Beta **(Node runners need to pay attention to this one and see below section for details.)** 



## More examples
- **Example 1**: `Query` latest ETH-USD price from Coinbase. Try this gist on [remix](http://remix.ethereum.org/#gist=f39845c47564c9ff98085749bd542d44&optimize=false&version=soljson-v0.4.25+commit.59dbf8f1.js). This example is also [deployed]() on rinkeby testnet.
```solidity
  pragma solidity ^0.4.24;

  import "./Ownable.sol";
  import "./DOSOnChainSDK.sol";

  contract CoinbaseEthUsd is Ownable, DOSOnChainSDK {
      // Struct to hold parsed floating string "123.45"
      struct ethusd {
          uint integral;
          uint fractional;
      }
      uint queryId;
      string public price_str;
      ethusd public prices;

      function check() public {
          queryId = DOSQuery(30, "https://api.coinbase.com/v2/prices/ETH-USD/spot", "$.data.amount");
      }

      modifier auth {
          // Filter out malicious __callback__ callers.
          require(msg.sender == fromDOSProxyContract(), "Unauthenticated response");
          _;
      }

      function __callback__(uint id, bytes result) external auth {
          require(queryId == id, "Unmatched response");

          price_str = string(result);
          prices.integral = price_str.str2Uint();
          int delimit_idx = price_str.indexOf('.');
          if (delimit_idx != -1) {
              prices.fractional = price_str.subStr(uint(delimit_idx + 1)).str2Uint();
          }
      }
  }
```
- **Example 2**: A `SimpleDice` game with no house edge based on secure and unpredictable random number generation. Try it out on [remix](http://remix.ethereum.org/#gist=3b2ca0410af407497bdc70ffe79ee123&optimize=false&version=soljson-v0.4.25+commit.59dbf8f1.js). This example is also [deployed]() on rinkeby testnet.
```solidity
  pragma solidity ^0.4.24;

  import "./Ownable.sol";
  import "./DOSOnChainSDK.sol";

  contract SimpleDice is Ownable, DOSOnChainSDK {
      address public devAddress = 0x1738A04F8942489E68d39409bA9C5c7864C2D754;
      // 1% winning payout goes to developer account
      uint public developCut = 1;
      // precise to 4 digits after decimal point.
      uint public decimal = 4;
      // gameId => gameInfo
      mapping(uint => DiceInfo) games;

      struct DiceInfo {
          uint rollUnder;  // betted number, player wins if random < rollUnder
          uint amountBet;  // amount in wei
          address player;  // better address
      }

      event ReceivedBet(
          uint gameId,
          uint rollUnder,
          uint weiBetted,
          address better
      );
      event PlayerWin(uint generated, uint betted, uint amountWin);
      event PlayerLose(uint generated, uint betted);

      modifier auth {
          // Filter out malicious __callback__ callers.
          require(msg.sender == fromDOSProxyContract(), "Unauthenticated response");
          _;
      }

      // 100 / (rollUnder - 1) * (1 - 0.01) => 99 / (rollUnder - 1)
      // Not using SafeMath as this function cannot overflow anyway.
      function computeWinPayout(uint rollUnder) public view returns(uint) {
          return 99 * (10 ** decimal) / (rollUnder - 1);
      }

      // 100 / (rollUnder - 1) * 0.01
      function computeDeveloperCut(uint rollUnder) public view returns(uint) {
          return 10 ** decimal / (rollUnder - 1);
      }

      function play(uint rollUnder) public payable {
          // winChance within [1%, 95%]
          require(rollUnder >= 2 && rollUnder <= 96, "rollUnder should be in 2~96");
          // Make sure contract has enough balance to cover payouts before game.
          // Not using SafeMath as I'm not expecting this demo contract's
          // balance to be very large.
          require(address(this).balance * (10 ** decimal) >= msg.value * computeWinPayout(rollUnder),
                  "Game contract doesn't have enough balance, decrease rollUnder");

          // Request a safe, unmanipulatable random number from DOS Network with
          // optional seed.
          uint gameId = DOSRandom(1, now);

          games[gameId] = DiceInfo(rollUnder, msg.value, msg.sender);
          // Emit event to notify Dapp frontend
          emit ReceivedBet(gameId, rollUnder, msg.value, msg.sender);
      }

      function __callback__(uint requestId, uint generatedRandom) external auth {
          address player = games[requestId].player;
          require(player != address(0x0));

          uint gen_rnd = generatedRandom % 100 + 1;
          uint rollUnder = games[requestId].rollUnder;
          uint betted = games[requestId].amountBet;
          delete games[requestId];

          if (gen_rnd < rollUnder) {
              // Player wins
              uint payout = betted * computeWinPayout(rollUnder) / (10 ** decimal);
              uint devPayout = betted * computeDeveloperCut(rollUnder) / (10 ** decimal);

              emit PlayerWin(gen_rnd, rollUnder, payout);
              player.transfer(payout);
              devAddress.transfer(devPayout);
          } else {
              // Lose
              emit PlayerLose(gen_rnd, rollUnder);
          }
      }
  }
```


## Security deposit and payment
In Beta

## Node registry
In Beta

## Acquire Testnet DOS Tokens
In Beta.


## Appendix
### Selector
The selector expression is following [JSONPath](https://www.npmjs.com/package/jsonpath) and [XPath](https://en.wikipedia.org/wiki/XPath) rules to filter components from responses.
##### Json example and selector expression:
```js
{
    "store": {
        "book": [
            {
                "category": "reference",
                "author": "Nigel Rees",
                "title": "Sayings of the Century",
                "price": 8.95
            },
            {
                "category": "fiction",
                "author": "Evelyn Waugh",
                "title": "Sword of Honour",
                "price": 12.99
            },
            {
                "category": "fiction",
                "author": "Herman Melville",
                "title": "Moby Dick",
                "isbn": "0-553-21311-3",
                "price": 8.99
            },
            {
                "category": "fiction",
                "author": "J. R. R. Tolkien",
                "title": "The Lord of the Rings",
                "isbn": "0-395-19395-8",
                "price": 22.99
            }
        ],
        "bicycle": {
            "color": "red",
            "price": 19.95
        }
    },
    "expensive": 10
}
```
Example selector expressions for the above json object:


Selector expression           | Description
----------------------------- | --------------
$.expensive 			                               | 10
$.store.book[0].price                            | 8.95
$.store.book[-1].isbn                            | "0-395-19395-8"
$.store.book[0,1].price                          | [8.95, 12.99]
$.store.book[0:2].price                          | [8.95, 12.99, 8.99]
$.store.book[?(@.isbn)].price                    |  [8.99, 22.99]
$.store.book[?(@.price > 10)].title              | ["Sword of Honour", "The Lord of the Rings"]
$.store.book[?(@.price < $.expensive)].price     | [8.95, 8.99]
$.store.book[:].price                            | [8.9.5, 12.99, 8.9.9, 22.99]

* Use this [online tool](https://codebeautify.org/jsonpath-tester) to get familar with JSONPath selector.

##### XML example and selector expression:
```xml
<library>
  <!-- Great book. -->
  <book id="b0836217462" available="true">
    <isbn>0836217462</isbn>
    <title lang="en">Being a Dog Is a Full-Time Job</title>
    <quote>I'd dog paddle the deepest ocean.</quote>
    <author id="CMS">
      <?echo "go rocks"?>
      <name>Charles M Schulz</name>
      <born>1922-11-26</born>
      <dead>2000-02-12</dead>
    </author>
    <character id="PP">
      <name>Peppermint Patty</name>
      <born>1966-08-22</born>
      <qualification>bold, brash and tomboyish</qualification>
    </character>
    <character id="Snoopy">
      <name>Snoopy</name>
      <born>1950-10-04</born>
      <qualification>extroverted beagle</qualification>
    </character>
  </book>
</library>
```
Example selector expressions for the above xml document:

Selector expression           | Description
------------------------------|------------
/library/book/isbn                               |  "0836217462"
/library/*/isbn                                  |  "0836217462"
/library/book/../book/./isbn                     |  "0836217462"
/library/book/character[2]/name                  |  "Snoopy"
/library/book/character[born='1950-10-04']/name  |  "Snoopy"
/library/book//node()[@id='PP']/name             |  "Peppermint Patty"
//book[author/@id='CMS']/title                   |  "Being a Dog Is a Full-Time Job",
/library/book/preceding::comment()               |  " Great book. "
//*[contains(born,'1922')]/name                  |  "Charles M Schulz"
//*[@id='PP' or @id='Snoopy']/born               |  {"1966-08-22", "1950-10-04"}

* Use this [online tool](http://www.utilities-online.info/xpath) to get familar with XPath selector.
