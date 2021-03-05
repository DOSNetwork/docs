#### Time Window & Data Granularity
* `function windowSize() public view returns (uint)`:
  - Length of sliding window in seconds. Stream data is guaranteed to be updated at least once in each sliding window. E.g.: 1200 represents 20 minutes (1200 seconds).

* `function deviation() public view returns (uint)`:
  - Threshold value in multiples of 1/1000.
  - Stream data is guaranteed to be updated if the data from data source is beyond `(1 ± deviation / 1000) * lastUpdatedValue`. E.g.: 5 represents 0.005 deviation threshold.

* `function decimal() public view returns (uint)`:
  - Number of decimals used in the price data. E.g.: With decimal value 8, 153765000000 represents 1537.65.



#### Number of Data Points
* `function numPoints() public view returns (uint)`:
  - Number of data points this data stream has ever been updated with since birth.

* `function num24hPoints() public view returns (uint)`:
  - Number of data points this data stream has been updated with within the last 24 hours.



#### Data Freshness
* `function stale(uint age) public view returns (bool)`:
  - Returns true if the last updated result is outdated: i.e. last updated timestamp is more than `age` seconds from `now`.



#### Latest and Historical Data
* `function latestResult() public view accessible returns (uint _lastPrice, uint _lastUpdatedTime)`:
  - Returns the latest reported data point consensused from the data source, as well as the latest update time.

* `function result(uint idx) public view accessible returns (uint _price, uint _timestamp)`:
  - Returns a specific historical data point and its timestamp given appropriate index into data point arrray.
  - Parameter idx can be computed using helper functions like `binarySearch(uint timeDelta)`, `numPoints()`, etc. Read [Stream.sol](https://github.com/DOSNetwork/smart-contracts/blob/master/contracts/Stream.sol) source code to learn more.


#### Time-Weighted-Average-Price (TWAP) Data
A series of prebuilt functions to compute TWAP price data for time window from 1 hour to 1 week:
```solidity
function TWAP1Hour() public view accessible returns (uint)
function TWAP2Hour() public view accessible returns (uint)
function TWAP4Hour() public view accessible returns (uint)
function TWAP6Hour() public view accessible returns (uint)
function TWAP8Hour() public view accessible returns (uint)
function TWAP12Hour() public view accessible returns (uint)
function TWAP1Day() public view accessible returns (uint)
function TWAP1Week() public view accessible returns (uint)
```

* `function twapResult(uint startIdx) public view accessible returns (uint)`:
  - Compute TWAP price data (observations[startIdx] : observations[lastIdx]).



#### Misc
* `function whitelistEnabled() public view returns (bool)`:
  - Returns true if whitelist access mode enabled. Once enabled, only whitelisted smart contracts, or EOA address can consume stream data.

* `function parser() public view returns (address)`:
  - Returns parser contract used to parse reported data from data source, as consensused reported data is encoded into `bytes` format for general purpose.

* `function hasAccess(address reader) public view returns (bool)`:
  - Returns whether `reader` address / contract has access to stream data. Can be skipped if `whitelistEnabled` is not turned on.

* `function shouldUpdate(uint price) public view returns (bool)`:
  - Returns whether `price` data should be updated. True if either last updated timestamp is outdated, or `data` is beyond `deviation` threshold of last result.
  - Note that this is only a reference function, `price` is not used to update the stream data. Stream data can only be updated in the `__callback__` function, invoked by off-chain nodes, with off-chain collectively generated group signature must also be verified on-chain before an update is invoked.

* `function binarySearch(uint timedelta) public view returns (uint)`:
  - All reported data is sorted by timestamp in ascending order. Given time range, return the maximum index {i} satisfying that: `data[i].timestamp <= data[end].timestamp - timedelta`. This helper function is mostly used to construct other useful features like TWAP.
