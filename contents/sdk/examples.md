- **Example 1**: `DOSQuery()` to get latest ETH-USD price from Coinbase.
```solidity
  pragma solidity ^0.5.0;

  import "./utils.sol";
  import "./DOSOnChainSDK.sol";

  // An example get latest ETH-USD price from Coinbase
  contract CoinbaseEthPriceFeed is DOSOnChainSDK {
    using utils for *;

    // Struct to hold parsed floating string "123.45"
    struct ethusd {
        uint integral;
        uint fractional;
    }
    uint queryId;
    string public price_str;
    ethusd public price;

    event GetPrice(uint integral, uint fractional);

    constructor() public {
        // @dev: setup and then transfer DOS tokens into deployed contract
        // as oracle fees.
        // Unused fees can be reclaimed by calling DOSRefund() in the SDK.
        super.DOSSetup();
    }

    function getEthUsdPrice() public {
        queryId = DOSQuery(30, "https://api.coinbase.com/v2/prices/ETH-USD/spot", "$.data.amount");
    }

    function __callback__(uint id, bytes calldata result) external auth {
        require(queryId == id, "Unmatched response");

        price_str = string(result);
        price.integral = price_str.subStr(1).str2Uint();
        int delimit_idx = price_str.indexOf('.');
        if (delimit_idx != -1) {
            price.fractional = price_str.subStr(uint(delimit_idx + 1)).str2Uint();
        }
        emit GetPrice(price.integral, price.fractional);
    }
  }
```
 - Try this gist on [remix](http://remix.ethereum.org/#gist=f39845c47564c9ff98085749bd542d44&optimize=true&version=soljson-v0.5.17+commit.d19bba13.js).
 - The example is also [deployed](https://etherscan.io/address/0x284e8386e94624d7a681b883d0a718ec22481536#code) on ethereum mainnet and seeded with 450 DOS tokens for anyone to try with.
 - (On Etherscan, click `Write Contract`, click `Connect to Web3`. Then select `getEthUSdPrice()` and you'll be paying a little gas. After 1 block, click `Read Contract` and see `price_str` and `price` fields. Click `Events` and `Internal Txns` to see more details.).

<p align="center">
  <img width="600" height="400" src="https://raw.githubusercontent.com/DOSNetwork/docs/master/_media/remix.png">
</p>


- **Example 2**: A `SimpleDice` game with no insider trading or house edge, based on smart contract plus secure and unpredictable random number generated through `DOSRandom()`.
```solidity
  pragma solidity ^0.5.0;

  import "./DOSOnChainSDK.sol";

  contract SimpleDice is DOSOnChainSDK {
    address payable public devAddress;
    uint public devContributed = 0;
    // 1% winning payout goes to developer account
    uint public developCut = 1;
    // precise to 4 digits after decimal point.
    uint public decimal = 4;
    // gameId => gameInfo
    mapping(uint => DiceInfo) public games;

    struct DiceInfo {
        uint rollUnder;  // betted number, player wins if random < rollUnder
        uint amountBet;  // amount in wei
        address payable player;  // better address
    }

    event ReceivedBet(
        uint gameId,
        uint rollUnder,
        uint weiBetted,
        address better
    );
    event PlayerWin(uint gameId, uint generated, uint betted, uint amountWin);
    event PlayerLose(uint gameId, uint generated, uint betted);

    modifier onlyDev {
        require(msg.sender == devAddress);
        _;
    }

    constructor() public {
        // @dev: setup and then transfer DOS tokens into deployed contract
        // as oracle fees.
        // Unused fees can be reclaimed by calling DOSRefund() in the SDK.
        super.DOSSetup();
        
        // Convert address to payable address.
        devAddress = address(uint160(owner()));
    }

    function min(uint a, uint b) internal pure returns(uint) {
        return a < b ? a : b;
    }
    // Only receive bankroll funding from developer.
    function() external payable onlyDev {
        devContributed += msg.value;
    }
    // Only developer can withdraw the amount up to what he has contributed.
    function devWithdrawal() public onlyDev {
        uint withdrawalAmount = min(address(this).balance, devContributed);
        devContributed = 0;
        devAddress.transfer(withdrawalAmount);
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
        uint gameId = DOSRandom(now);

        games[gameId] = DiceInfo(rollUnder, msg.value, msg.sender);
        // Emit event to notify Dapp frontend
        emit ReceivedBet(gameId, rollUnder, msg.value, msg.sender);
    }

    function __callback__(uint requestId, uint generatedRandom) external auth {
        address payable player = games[requestId].player;
        require(player != address(0x0));

        uint gen_rnd = generatedRandom % 100 + 1;
        uint rollUnder = games[requestId].rollUnder;
        uint betted = games[requestId].amountBet;
        delete games[requestId];

        if (gen_rnd < rollUnder) {
            // Player wins
            uint payout = betted * computeWinPayout(rollUnder) / (10 ** decimal);
            uint devPayout = betted * computeDeveloperCut(rollUnder) / (10 ** decimal);

            emit PlayerWin(requestId, gen_rnd, rollUnder, payout);
            player.transfer(payout);
            devAddress.transfer(devPayout);
        } else {
            // Lose
            emit PlayerLose(requestId, gen_rnd, rollUnder);
        }
    }
  }
```
 - Try this gist out on [remix](http://remix.ethereum.org/#gist=3b2ca0410af407497bdc70ffe79ee123&optimize=true&evmVersion=null&version=soljson-v0.5.17+commit.d19bba13.js).
 - This example is also [deployed](https://rinkeby.etherscan.io/address/0xb156032041f83eb6d96916aada5c7dbdd338dccb) on rinkeby testnet.
 - (On Etherscan, click `Write Contract`, click `Connect to Web3`. Then select `play()` and you need to fill in 2 arguments: the amount of `testnet ether` you'd like to bet, and a `rollUnder` number. If the off-chain collectively generated random number is less than the `rollUnder` you provide then you win, otherwise you lose the amount you bet. After 1 block, click `Internal Txns` and `ReceivedBet & PlayerWin / PlayerLose` Events to see more details.



- **Example 3**: See [Stream](https://github.com/DOSNetwork/smart-contracts/blob/master/contracts/Stream.sol) contract to learn more.
