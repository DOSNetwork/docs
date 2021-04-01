## Introducing Data Streams
<p align="center">
  <img width="880" height="660" src="https://raw.githubusercontent.com/DOSNetwork/docs/heco/_media/streams_ui.png">
</p>

* Data Streams provide real-time market prices of popular assests from high-quality data sources, secured by off-chain decentralized oracle network and validated by on-chain smart contracts before making available to dependent smart contracts.

* Stream data is guaranteed to be updated either every time window (e.g. 20 minutes) or when the latest data from data source is beyond the predetermined threshold (e.g. ±0.5%) of the last updated data, whichever happens first.


* Data streams' results can be read by External Own Accounts (`EOA`) or consumed (through `EVM call`) by downstream smart contracts, **freely** for now.

* Please [contact](mailto:info@dos.network) DOS Team if you're developers and want to access Data Streams in your smart contracts freely. We're glad to onboard more projects / developers and we have development grants to boost the usage and ecosystem.

* Please note that each data stream has a switch called `whitelistEnabled`, which is currently disabled. Once turned on, only whitelisted smart contracts can consume price data. Data streams are made for the public good and open to everyone for now to bootstrap both DeFi & DOS ecosystems on supported chains, but there're both technical & operational efforts to keep data streams secure and accurate, thus we highly encourage projects / developers to contact the team to get your smart contracts whitelisted beforehead, as well as receive news of latest supported streams.


## Get Latest Price Data
* For your smart contracts to read latest price data, simply plugin [StreamInterface](https://github.com/DOSNetwork/smart-contracts/blob/master/contracts/interfaces/StreamInterface.sol) and call `latestResult()` like below.

<!-- tabs:start -->

#### **Example Code**
```solidity
pragma solidity 0.5.17;

import "github.com/DOSNetwork/smart-contracts/contracts/interfaces/StreamInterface.sol";

contract BTCLatestPriceExample {
    IStream public stream;
    uint public decimal;

    /**
     * Network: Heco Testnet
     * Description: BTC/USD
     * Decimal: stream.decimal()
     * Address: 0x2022737ddC95b15d55E452C9E8418899063fc196
     */
    constructor() public {
        stream = IStream(0x2022737ddC95b15d55E452C9E8418899063fc196);
        decimal = stream.decimal();
    }

    /**
     * Returns the latest price
     */
    function getLatestBTCPrice() public view returns (uint) {
        (uint price, uint timestamp) = stream.latestResult();
        return price;
    }
}
```

<!-- tabs:end -->



## Get TWAP Price Data
* `Time-Weighted-Average-Price (TWAP)` data is widely used to smooth out abnormal spikes. A set of TWAP functions, ranging from 1 hour TWAP to 24 hours TWAP, are builtin for the ease of developers, simply calling `TWAP*Hour` (* = 1, 2, 4, 6, 8, 12) like below:

<!-- tabs:start -->

#### **Example Code**
```solidity
pragma solidity 0.5.17;

import "github.com/DOSNetwork/smart-contracts/contracts/interfaces/StreamInterface.sol";

contract BTCTwapPriceExample {
    IStream public stream;
    uint public decimal;

    /**
     * Network: Heco Testnet
     * Description: BTC/USD
     * Data Decimal: stream.decimal()
     * Address: 0x2022737ddC95b15d55E452C9E8418899063fc196
     */
    constructor() public {
        stream = IStream(0x2022737ddC95b15d55E452C9E8418899063fc196);
        decimal = stream.decimal();
    }

    // Returns Time-Weighted-Average-Price for 1 hour.
    function getBTC1HourTwapPrice() public view returns (uint) {
        return stream.TWAP1Hour();
    }
    function getBTC2HourTwapPrice() public view returns (uint) {
        return stream.TWAP2Hour();
    }

    // ... 
    // ...

    // Returns Time-Weighted-Average-Price for 1 day.
    function getBTC1DayTwapPrice() public view returns (uint) {
        return stream.TWAP1Day();
    }
}
```

<!-- tabs:end -->



## Get Historical Price Data
* Each Data Stream is either updated every windowSize (e.g. 20 minutes) or when the latest data from data source is beyond the pre-determined threashold (e.g. ±0.5%) of last updated result, whichever happenes first.
* Any specific historical data point can be obtained by calling `result(uint idx)`, along with other helper functions like `numPoints()`, `num24hPoints()`, `binarySearch(uint timedelta)` to get optimal parameter `idx`. Check [Stream.sol](https://github.com/DOSNetwork/smart-contracts/blob/master/contracts/Stream.sol) source code to meet advanced requirements.



## Supported Assets & Data Sources
* **Crypto**:
  - Data Source 1: [Coingecko](https://www.coingecko.com/en/api)
  - Supported Assets: `BTC/USD`, `ETH/USD`, `DOT/USD`, `HT/USD`, `DOS/USD`, `FIL/USD`, `HPT/USD`

* **Stocks**:
  - Data Source 1: [IEXCloud](#)
  - Data Source 2: [AlphaVantage](#)
  - Supported Assets: [...]

* **Forex**: [...]

* **Commodities**: [...]
