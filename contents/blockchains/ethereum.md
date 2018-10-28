## Quick Start
DOSOnChainSDK exposes several APIs in the Alpha release that are helpful to developers.

#### Retrieving external data:
<!-- tabs:start -->

#### ** For Alpha release **
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

#### ** Post-alpha & Beta release **
- ```function DOSQuery(uint timeout, string dataSource, string selector) returns (uint)```: Smart contract requests for external data by calling this function. The whole process is an asynchronous one - i.e. ```DOSQuery()``` merely returns a unique queryId that caller caches for bookkeeping and future identification, with the real response coming back through the ```__callback__``` function below. Example usage:
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
- ```function __callback__(uint queryId, bytes result) external```: The raw response data would be backfilled through the ```__callback__``` function without any parsing (for the Alpha release), we will soon integrate both on-chain and off-chain parsers for ease of use. Example usage:
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

#### Requesting for secure and unmanipulatable random numbers:
- ```function DOSRandom(uint8 mode, uint seed) returns (uint)```:
- ```function __callback__(uint requestId, uint generatedRandom) external```:


## Deployed Contracts on Rinkeby testnet
* [DOSAddressBridge]() - A connector contains all system contratcs' addresses, simply for *upgradable* contract development pattern. *(Neither developers nor node runners need to worry about this contract.)*
* [DOSOnChainSDK]() - APIs smart contract developers can take advantage of to request external off-chain data and leverage off-chain verifiable computation power for computation-intensive tasks. **(Developers need to pay attention to this one and see below section for details.)**
* [DOSProxy]() - Conceals implementation details such as request handling, random group selection, threshold signature verification, user-defined callback function invocation, response parsing, etc. *(Neither developers nor node runners need to worry about this contract.)*
* [DOSPayment]() - To be released in Beta **(Node runners need to pay attention to this one and see below section for details.)** 
* [DOSRegistry]() - To be released in Beta **(Node runners need to pay attention to this one and see below section for details.)** 


## Security deposit and payment
In Beta

## Node registry
In Beta

## Acquire Testnet DOS Tokens
In Beta.



## More
#### Examples


#### Selector syntax (To be supported in post-alpha)
For query requests, instead of returning the bulk raw data it's always encouraged to specify what exact components are needed in the smart contract, to save caller's money and speed up processing time. The selector is following the   [JSONPath](https://www.npmjs.com/package/jsonpath) and [XPath]() syntax to filter components from responses.
##### Json example and selector expression:
```json
{
  "store": {
    "book": [ 
      {
        "category": "reference",
        "author": "Nigel Rees",
        "title": "Sayings of the Century",
        "price": 8.95
      }, {
        "category": "fiction",
        "author": "Evelyn Waugh",
        "title": "Sword of Honour",
        "price": 12.99
      }, {
        "category": "fiction",
        "author": "Herman Melville",
        "title": "Moby Dick",
        "isbn": "0-553-21311-3",
        "price": 8.99
      }, {
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
  }
}
```
Example selector expressions for the above json object:

Selector expression           | Description
------------------------------|------------
$.store.book[*].author        | The authors of all books in the store
$..author                     | All authors
$.store.*                     | All things in store, which are some books and a red bicycle
$.store..price                | The price of everything in the store
$..book[2]                    | The third book
$..book[(@.length-1)]         | The last book via script subscript
$..book[-1:]                  | The last book via slice
$..book[0,1]                  | The first two books via subscript union
$..book[:2]                   | The first two books via subscript array slice
$..book[?(@.isbn)]            | Filter all books with isbn number
$..book[?(@.price<10)]        | Filter all books cheaper than 10
$..book[?(@.price==8.95)]     | Filter all books that cost 8.95
$..book[?(@.price<30 && @.category=="fiction")]        | Filter all fiction books cheaper than 30
$..*                          | All members of JSON structure

* Use this [online tool](http://jsonpath.com/) to get familar with JSONPath selector.

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
library/*/isbn                                   |  "0836217462"
/library/book/../book/./isbn                     |  "0836217462"
/library/book/character[2]/name                  |  "Snoopy"
/library/book/character[born='1950-10-04']/name  |  "Snoopy"
/library/book//node()[@id='PP']/name             |  "Peppermint Patty"
//book[author/@id='CMS']/title                   |  "Being a Dog Is a Full-Time Job",
/library/book/preceding::comment()               |  " Great book. "
//*[contains(born,'1922')]/name                  |  "Charles M Schulz"
//*[@id='PP' or @id='Snoopy']/born               |  {"1966-08-22", "1950-10-04"}

* Use this [online tool]() to get familar with XPath selector.
