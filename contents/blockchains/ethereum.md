## Deployed Contracts on Mainnet
We're using [proxy-upgrade pattern](https://blog.openzeppelin.com/proxy-patterns/) for contracts that touching users' money (i.e. `DOSPayment` and `Staking` contract), so that in case of emergency situation we'll be able to upgrade to a patched version without interrupting user behavior.
* [DOSAddressBridge](https://etherscan.io/address/0x98a0e7026778840aacd28b9c03137d32e06f5ff1) - Connector that contains all system contratcs' addresses.
* [CommitReveal](https://etherscan.io/address/0x144ed0555269628049f76da2adbdcdf3aa488e0e) - Conceals details of [commit-reveal scheme](https://en.wikipedia.org/wiki/Commitment_scheme), which is used to generate a secure and unpredictable genesis random number by multiparties in every bootstrap phase.
* [DOSProxy](https://etherscan.io/address/0x1402c061e2aec7b41ae4911b40f7b340489dd1da) - Conceals details such as request handling, random group selection, threshold signature verification, user-defined callback function invocation, response parsing, etc.
* `DOSPayment` - Conseals details of oracle request payment schemes and other node runners' validity judgement.
  - Payment gateway (proxy): https://etherscan.io/address/0x7b8d5a37008382b2c7e8e15ecf3c2727e38a6ac6
  - Payment implementation: https://etherscan.io/address/0x24286C5a340bF99EDB2d7e7D114477987d34816F
* `Staking` - Both eligible node runners and normal token holders (delegators) are able to earn staking rewards:
  - Staking gateway (proxy): https://etherscan.io/address/0x5dbef8e9e83a17d4d1d4c65a1e26133edae851dc
  - Staking implementation: https://etherscan.io/address/0x6a829E0EB032FA39D0444D29DFd80Bd3AE91C5B9
  - Node runners earn staking rewards by staking at least 800K tokens (or lessen a bit by owning [DropBurn](https://medium.com/dos-network/introducing-dropburn-a-new-model-to-bootstrap-staking-network-3b2c605dd276) token) themselves and join the network to provide oracle services.
  - Normal token holders earn staking rewards by delegating to eligible nodes, they may need to pay a percentage of earned rewards to delegated nodes.
  - A user-friendly [frontend](https://dashboard.dos.network) is provided to help node runners and token holders to stake, delegate, withdraw rewards, register a node, etc.
  - (Note that running a node requires both on-chain stake bonding and off-chain setup of node software, which is detailed in [node runner tutorials](https://medium.com/dos-network/instructions-of-launching-a-node-in-dos-network-932e73a91a75))

<p align="center">
  <img src="https://raw.githubusercontent.com/DOSNetwork/docs/master/_media/staking.png">
</p>


## Deployed Contracts on Rinkeby Testnet
* [DOSAddressBridge](https://rinkeby.etherscan.io/address/0xeE2e9f35c9F91571535173902E7e7B4E67deE32b)
* [CommitReveal](https://rinkeby.etherscan.io/address/0x044D8D7028eC8Fc98247d072603F5316656EcfDe)
* [DOSProxy](https://rinkeby.etherscan.io/address/0xAb09D3A9998c918Ffa796F6449D8515e5C7DB8a2)
* `DOSPayment`:
  - Payment gateway (proxy): https://rinkeby.etherscan.io/address/0x306d78A9Cf1116513220C908C5D950914D797682
  - Payment implementation: https://rinkeby.etherscan.io/address/0x6b89f9C6bD11B14ae17DAfba4C578DdA527E7EF3
* `Staking`:
  - Staking gateway (proxy): https://rinkeby.etherscan.io/address/0x064fa6a739580a9bA8cfeFDc271fa5585BC274e3
  - Staking implementation: https://rinkeby.etherscan.io/address/0x9faaebe59eaf3132c3cf42a947bab11408d12296



## Acquire Testnet DOS Tokens
* [Testnet DOS Token](https://rinkeby.etherscan.io/address/0x214e79c85744cd2ebbc64ddc0047131496871bee)
* Please fill in this [form](https://docs.google.com/forms/d/e/1FAIpQLSe7Kf1RvGa2p5SjP4eGAp-fw2frauOl6CDORnHK0-TNbjho9w/viewform) to request testnet tokens.
* Testnet DOS token Faucet:
 - [x]


## Appendix
### Selector
The selector expression is following [JSONPath](https://www.npmjs.com/package/jsonpath) and [XPath](https://en.wikipedia.org/wiki/XPath) rules to filter components from responses.
##### Json example and selector expression:
```solidity
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
