#### Time Window & Data Granularity
* windowSize()
* deviation()
* decimal()


#### Number of Data Points
* numPoints()
* num24hPoints()


#### Data Freshness
* stale(uint age)


#### Latest and Historical Data
* latestResult()
* result(uint idx)


#### Time-Weighted-Average-Price (TWAP) Data
* TWAP1Hour()
* TWAP2Hour()
* TWAP4Hour()
* TWAP6Hour()
* TWAP8Hour()
* TWAP12Hour()
* TWAP1Day()
* TWAP1Week()

* twapResult(uint startIdx)


#### Misc
* whitelistEnabled()
* parser()
* hasAccess(address reader)
* shouldUpdate(uint price)
* megaUpdate(uint price)
* binarySearch(uint timeDelta)

