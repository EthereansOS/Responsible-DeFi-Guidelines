# The Responsible DeFi Manifesto
A proposal for Ethereans to buidl Decentralized Finance in a more pragmatic and responsible way
By AleToschi.eth, CD and Vasapower.eth

Original Manifesto: https://medium.com/dfohub/the-responsible-defi-manifesto-1c6ea42f5835

## ABSTRACT
The DeFi ecosystem is one of the most interesting experiments ever conducted in crypto. It aims to disrupt finance by creating a new censorship-resistant and anonymous layer on top of Ethereum. DeFi dApps have been around for a while, but this era truly began with one of the most ambitious Eth projects ever, in 2019: MakerDAO, with its stable coin $DAI.
At this early stage, other useful DeFi dApps appeared, like MakerDAO (Debt-Based Stability), Uniswap (AMM Protocol), WBTC (BTC on Ethereum), Nexus Mutual (Smart Contract Insurance), Aave (Flash Loans), Compound (Lending/Borrowing), Set Protocol (Basket Portfolio) and DyDx (Margin Trading and Derivatives).
Next, the craziness of the crypto world intensified, and the DeFi ecosystem set off on a furious race to chase capital and speculative hype. There are now hundreds of DeFi dApps and tokens, and it has become impossible to track them all, even for the website https://defipulse.com. This is even without taking into account fake DeFi dApps that are actually CeFi, but which are trying to sell themselves as DeFi…
And now, the speculative side of DeFi has created the wildest monster in the field (from a developer's perspective): YIELD FARMING.
As things stand, DeFi isn't trending toward the development of visionary dApps that will change the world, but rather toward that of short-sighted ones that will change portfolios. 
This strikes me as reckless; these dApps are incentivized to rush mass adoption ASAP, rather than after thorough test, test, testing…

## The Auditing Chicken-Egg Paradox
For some people, audits are the solution to the security problem of DeFi smart contracts. In reality, they're just a boomer-trap. "Auditing" has the reputation of a holy grail, but really it just refers to the process whereby coders test the code of others. This is undoubtedly helpful; the more experts who test a program, the less human-based errors it will experience. 
However, audits are ultimately overvalued, for three reasons:
- Humans cannot fully trust other humans; auditors can have different incentives than issuers, and may actually be less-skilled than them.
- In general, bugs only end up revealing themselves in unpredictable conditions. Even with audits it is often impossible to be sure if software is 100% secure, and in finance, you have to take this into account.
- Trusting auditors as the final security measure is on par with trusting Credit Rating Agencies in 2008. As we all know, that led to lobbyism, and worse… 💥!

Consider some of audited DeFi startups that have experienced the consequences of bugs and multiple points of failure:
MakerDAO | Balancer | dForce and others. 
These organizations have lost user funds and are over-reliant on centralized, unsustainable emergency strategies. 
(Ironically, some unaudited DeFi dapps never experience such problems, even in weird market conditions.)
To summarize: auditing doesn't equal security.
Indeed, it's often nothing more than a marketing buzzword. We need a more pragmatic and reliable approach to ensure dApp security. This is why we are proposing a new one: Responsible DeFi!

# Responsible DeFi

This new approach will enable coders to
- 🐞🔫 reduce unpredictable bugs,
- 🤖🖖 gain independence from humans
- 🤡🚩and easily redflag scammers.

It is based on 5 simple architectural ideas:

## 💎 Simple Is Better
Keep smart contracts simple; keep smart contracts secure.

### Modular Programming:
Every part of a complex function should be split into standalone modules that can work together.

Code must "talk." A complex algorithm must be composed as a sequence of simple functions, one that can be read as a complete sentence by a stranger without the aid of documentation. If possible, these functions should work as standalone modules as well as together in unity.

```
function setDoubleProxy(address newDoubleProxy) public _byCommunity
{
    _doubleProxy = newDoubleProxy;
}

function setAllowedPairs(address[] memory newAllowedPairs) public _byCommunity
{
    _allowedPairs = newAllowedPairs;
}
```

The following setter methods contain a very simple logic, and both have the _byCommunity modifier:

```
modifier _byCommunity() {
    require(
        IMVDFunctionalitiesManager(
            IMVDProxy(IDoubleProxy(_doubleProxy).proxy())
                .getMVDFunctionalitiesManagerAddress()
        )
        .isAuthorizedFunctionality(msg.sender),
        "Unauthorized Action!"
    );
    _;
}
```

Its body calls the Functionalities Manager, passing on chain first from the DoubleProxy and then from the Proxy, ensuring that the caller of these setters is an authorized Functionality of the DFO to which this contract refers. For it to be an authorized Functionality, it must first of all have been Proposed through a Survey, which was then voted on by the community of token holders, who together accepted it to perform the action.

### Simple Math:

Solve mathematical problems in the simplest way possible. If your solution is complex, it may include unnecessary points of failure, particularly in calculations of decimals vs Wei or actions based on prices. The Responsible DeFi approach is to always ask the questions, "does this dApp really need this level of complexity?" and "can this problem be solved with simpler calculations?"

Solidity is a great language, but still a nascent one, and it's not always easy to figure out what exactly is under the hood of an algorithm. To be truly sure of what you're doing, keep the logic as basic as possible. especially in the mathematical calculations, which are the most delicate part.

```
function _calculateRewardToMint(uint256 burntWei) private returns(uint256 rewardWei) {
    rewardWei = _standardRewardWei;
    rewardWei = rewardWei * (burntWei / (10**decimals()));
    rewardWei =  (rewardWei * _rebalanceRewardMultiplier[0]) / _rebalanceRewardMultiplier[1];
}
```

This code shows how it is possible to generate a reward starting from a base value, expressed in wei (18 decimal places), with the burnt tokens, also expressed in wei, by multiplying them by a certain exchange factor. To do this, the amount of tokens burned is converted into 'ether' (i.e. without decimals) in order to make the calculation consistent. But what happens when the amount of tokens burned is less than 1? There are no floating point numbers in Solidity, so converting to 'ether' will truncate the numbers after the comma, resulting in 0.

The best way to solve this problem is to make sure that the amount of tokens burned is always greater than or equal to 1 (i.e. 10 ^ 18), to avoid inconsistencies in the mathematics.

```
function rebalance() public override {
    uint256 burnable = calculateBurnable();
    uint256 available = balanceOf(msg.sender);
    burnable = available >= burnable ? burnable : burnable - available;
    require(burnable >= 10**decimals(), "Insufficient amount to burn");
    _burn(msg.sender, burnable);
    IMVDProxy(IDoubleProxy(_doubleProxy).proxy()).submit(
        "mintNewVotingTokens",
         abi.encode(
            address(0),
            0,
            _calculateRewardToMint(burnable),
            msg.sender
        )
    );
}
```

### Don't Overuse the State:

The State on Ethereum is a very cool tool, but at the same time, it can be your worst nightmare.
While developing on Ethereum, you must always keep in mind that you can't track everything; you have to allow anonymous people to use an application without needing to track them.

The Ethereum State can help you achieve more, but remember to think 999999 times before using it for unnecessary things, and always remember to generate the right limitation, not just because it's expensive for the users, but also because you can receive an OUT of GAS.
Solidity allows you to save different types of data, both standard and customized. 

The space available for the storage of a contract is vast, but keeping a lot of information can mean costly transactions! It is recommended to save exclusively those data that are useful to the contract itself or to the others from which it is called.

Everything else, such as information to populate the UI of the dApp of which the contract is a part, should be reconstructed externally, starting from both the few contract data points available, and the Events that the contract issues whenever it is called in a transaction.

```
function _emitEvent(address proxy, address sender) private {
    IMVDProxy(msg.sender).emitEvent("DFODeployed(address_indexed,address_indexed,address,address)", abi.encodePacked(proxy), abi.encodePacked(sender), abi.encode(proxy, sender));
}
```

### Make Smart Contracts Readable:

If you follow all the steps outlined above, your smart contracts will be very simple to read, even for non-coders!

Documentation and comments are a cool approach to help people understand them, but humans make them, so trusting them is not the ideal way to understand the contract. Rendering the code legible to a layperson even without comments and documentation is a far better approach.

## ⏳ Scalable Usage Limitation

A new security measure that raises a simple red flag to expose scammers is a limit on the usage of a DeFi dApp in the early stages.

If you're performing complex calculations for the first time on Ethereum, this will protect people from FOMO: a hard-coded cap of lockable value that limits usage in the first year or longer.

For example, for a new stablecoin, you can set a weekly $20,000 mintable as a max cap, so even if people FOMO into your invention, they can't invest a ridiculous amount of money before the dApp is live for at least a year or more. This approach differentiates who is coding dApps for the long run on Ethereum from who just wants short term profits.

## Responsible Integration Mindset

In the DeFi ecosystem, the most famous maxim is "Lego Money." This is a very fresh approach, but before interacting with external dApps, you must keep in mind that every integration could add points of failure to your dApp.

Let's consider three levels of security dependent on this:
- Low Dependence: The integration can be considered secure because the core of the dApp is independent from external integration failure.
- Mid Dependence: The integration should be regarded as safe, but external factors and unpredictable bugs leave the core of the dApp at risk.
- High Dependence: The integration can be considered high risk because the core of the dApp is entirely susceptible to external integration failure.

### A low dependence example is Uniswap:
Uniswap is indirectly integrated with all of the ERC20 contracts, but by design, these contracts can't compromise the Uniswap core, even if they are killed.
### A mid dependence example is Balancer:
Balancer is indirectly integrated with all of the ERC20 Contracts, but some of them, due to Balancer's complex math, can compromise the system's security.
### High dependence examples are Synthetix, MakerDAO and Compound:
Synthetix logic is highly dependent on Chainlink Oracles to feed prices of the integrated assets. A failure of the prices or the ChainLink dApp can actively compromise all of the core math.

The Compound and MakerDAO systems are highly related to the price of integrated assets; in the case of a crash due to failure of their WBTC, USDC or USDT custodians, the entire system can crash.
The best practice in terms of Lego Money is to integrate external dApps based on varying risks of dependence:
- Low dependence: No risks. All dApps are ok.
- Mid dependence: Some risks. Whitelist dApps, or solve the related issue.
- High dependence: Many risks. Integrate only with decentralized dApps, with no external dependencies, on the mainnet after at least one year or more.

Even though unpopular in the DeFi ecosystem, a Responsible Integration Mindset is highly recommended to mitigate the risks associated with a future Domino scenario, which may befall an external dApp due to a failure.

## 🌈 Flexible Environment

One other important security layer needed in DeFi is to make dApps as independent as possible from an off-chain company. Projects in Ethereum need to manage bugs and updates. Usually, coders will go about this by issuing a proxy contract and using their private key to update the code if needed.

This approach does not guarantee total security for dApp users, because their funds are usually in the hands of the dApp issuers. dApp issuers are not a bank! Remember there are no bailouts or legitimate refunds in crypto. Trusting a dApp with code driven by a single company, even if they're regulated in the real world, is always a bad idea. Some dApps are developing DAOs to mitigate this risk, but at the same time a DAO can reduce problems by voting. Because the code of the DAO is in the hands of issuers, this is better, but still not a concrete solution.

On the contrary, there are new approaches to tackle this problem - like Decentralized Flexible Organizations (DFOs) - that remove the need for issuers to have special keys by using a protocol that is 100% modular and only upgradable or managed by voting.


```
contract MVDProxy {

    //... Some code

    function newProposal(string memory codeName, bool emergency, address sourceLocation, uint256 sourceLocationId, address location, bool submitable, string memory methodSignature, string memory returnAbiParametersArray, bool isInternal, bool needsSender, string memory replaces) public override returns(address proposalAddress) {
        emergencyBehavior(emergency);

        IMVDFunctionalityModelsManager(_delegates[3]).checkWellKnownFunctionalities(codeName, submitable, methodSignature, returnAbiParametersArray, isInternal, needsSender, replaces);

        IMVDFunctionalitiesManager functionalitiesManager = IMVDFunctionalitiesManager(_delegates[4]);

        IMVDFunctionalityProposal proposal = IMVDFunctionalityProposal(proposalAddress = IMVDFunctionalityProposalManager(_delegates[1]).newProposal(codeName, location, methodSignature, returnAbiParametersArray, replaces));
        proposal.setCollateralData(emergency, sourceLocation, sourceLocationId, submitable, isInternal, needsSender, msg.sender, functionalitiesManager.hasFunctionality("getVotesHardCap") ? toUint256(read("getVotesHardCap", "")) : 0);

        if(functionalitiesManager.hasFunctionality("onNewProposal")) {
            submit("onNewProposal", abi.encode(proposalAddress));
        }

        if(!IMVDFunctionalitiesManager(_delegates[4]).hasFunctionality("startProposal") || !IMVDFunctionalitiesManager(_delegates[4]).hasFunctionality("disableProposal")) {
            proposal.start();
        }

        emit Proposal(proposalAddress);
    }

    //... Other code
}
```

This is where it all begins: the creation of a Proposal is carried out by calling the Proxy of the DFO, which in turn consults its delegates, who physically take care of controlling, creating and keeping track of all the data of the Surveys that the Token Holders vote on.

```
contract MVDProxy {

    //... Some code

    function setProposal() public override {

        IMVDFunctionalityProposalManager(_delegates[1]).checkProposal(msg.sender);

        emit ProposalCheck(msg.sender);

        IMVDFunctionalitiesManager functionalitiesManager = IMVDFunctionalitiesManager(_delegates[4]);

        (address addressToCall,,string memory methodSignature,,) = functionalitiesManager.getFunctionalityData("checkSurveyResult");

        (bool surveyResult, bytes memory response) = addressToCall.staticcall(abi.encodeWithSignature(methodSignature, msg.sender));

        surveyResult = toUint256(response) > 0;

        IMVDFunctionalityProposal proposal = IMVDFunctionalityProposal(msg.sender);

        if(!surveyResult) {
            proposal.set();
            emit ProposalSet(msg.sender, surveyResult);            
            return;
        }

        if(collateralCallResult) {
            try functionalitiesManager.setupFunctionality(msg.sender) returns(bool managerResult) {
                collateralCallResult = managerResult;
            } catch {
                collateralCallResult = false;
            }
        }
        
        if(collateralCallResult) {
            proposal.set();
            emit ProposalSet(msg.sender, collateralCallResult);
        }
    }

    //... Other code
}
```

When a user finalizes a Vote Proposal, it automatically calls the Proxy of the DFO, which in turn asks its delegates to verify the result of the calling Proposal (via the "checkSurveyResult" Microservice) and then call, if successful, the finalization of the Microservice (via its Functionalities Manager).

```
contract MVDFunctionalitiesManager {

  function setupFunctionality(address proposalAddress) public override returns(bool result) {

        require(_proxy == msg.sender, "Only Proxy can call This!");

        IMVDFunctionalityProposal proposal = IMVDFunctionalityProposal(proposalAddress);

        string memory codeName = proposal.getCodeName();
        bool hasCodeName = !compareStrings(codeName, "");
        string memory replaces = proposal.getReplaces();
        bool hasReplaces = !compareStrings(replaces, "");

        if(!hasCodeName && !hasReplaces) {
            (result,) = IMVDProxy(_proxy).callFromManager(_callingContext = proposal.getLocation(), abi.encodeWithSignature("callOneTime(address)", proposalAddress));
            _callingContext = address(0);
            return result;
        }
        //continue with other cases...
    }
}
```

On the other hand, the Microservice setup can only be performed exclusively if the FunctionalitiesManager is called by the Proxy, as you can see from the 'require' sentence in the code.

If the conditions are met, the FunctionalitiesManager calls the Proxy again to execute the Microservice code. In summary, we have Token Holders voting for a Proposal, which triggers a constrained automated process that leads to its finalization, thanks to the orchestrated collaboration between all the Contracts that make up a DFO, following two fundamental principles of Software Engineering: 'Divide Et Impera' and 'Single Responsibility'.

This new experimental approach can create a new DeFi standard, by making dApps as resilient as ever and independent from the issuer which is to say the issuer can only rule the code if he holds tokens. Crucially, in the case of issuer failure, the dApp can still continue to function and be updated by token holders.
 more info at https://dfohub.com

Flexible environments are cool. Nevertheless, they need some limitations to be totally fair for DeFi dApps:

### Voting Limitations:
Token holders in a DFO can vote to run code, so they can vote to manage every asset and function. Some things, however, shouldn't be changed by voting. When building a DeFi application on top of a DFO, you need to create an external Functionality to store user funds.

For example, in the code of the Liquidity Staking Contract, all the values ​​that identify the times and parameters for generating the rewards of the various tiers are entered in the construction phase, and it is no longer possible to change them. Rewards are automatically sent to the External Smart Contract, so DFOs can only change the rules for newcomers and not for already staked positions.

Every time a new staking position will be opened, the staking contract will receive from the DFO the voting tokens necessary for the reward, by calling the "stakingTransfer" Microservice, as seen in the method called calculateRewardAndAddStakingPosition. These tokens will be transferable only by the owner of the position, according to the desired times.


```
contract Stake {

    //Other code...

    constructor(uint256 startBlock, address doubleProxy, address[] memory tokens, uint256[] memory timeWindows, uint256[] memory rewardMultipliers, uint256[] memory rewardDividers, uint256[] memory rewardSplitTranches) public {

        _startBlock = startBlock;

        _doubleProxy = doubleProxy;

        for(uint256 i = 0; i < tokens.length; i++) {
            TOKENS.push(tokens[i]);
        }

        assert(timeWindows.length == rewardMultipliers.length && rewardMultipliers.length == rewardDividers.length && rewardDividers.length == rewardSplitTranches.length);
        for(uint256 i = 0; i < timeWindows.length; i++) {
            TIME_WINDOWS.push(timeWindows[i]);
        }

        for(uint256 i = 0; i < rewardMultipliers.length; i++) {
            REWARD_MULTIPLIERS.push(rewardMultipliers[i]);
        }

        for(uint256 i = 0; i < rewardDividers.length; i++) {
            REWARD_DIVIDERS.push(rewardDividers[i]);
        }

        for(uint256 i = 0; i < rewardSplitTranches.length; i++) {
            REWARD_SPLIT_TRANCHES.push(rewardSplitTranches[i]);
        }
    }

    //Other code...

    function calculateRewardAndAddStakingPosition(uint256 tier, uint256 poolPosition, uint256 firstAmount, uint256 secondAmount, uint256 poolAmount, IMVDProxy proxy) private {
        uint256 partialRewardSingleBlockTime = TIME_WINDOWS[tier] / REWARD_SPLIT_TRANCHES[tier];
        uint256[] memory partialRewardBlockTimes = new uint256[](REWARD_SPLIT_TRANCHES[tier]);
        if(partialRewardBlockTimes.length > 0) {
            partialRewardBlockTimes[0] = block.number + partialRewardSingleBlockTime;
            for(uint256 i = 1; i < partialRewardBlockTimes.length; i++) {
                partialRewardBlockTimes[i] = partialRewardBlockTimes[i - 1] + partialRewardSingleBlockTime;
            }
        }
        uint256 reward = firstAmount * REWARD_MULTIPLIERS[tier] / REWARD_DIVIDERS[tier];
        StakeInfo memory stakeInfo = StakeInfo(msg.sender, poolPosition, firstAmount, secondAmount, poolAmount, reward, block.number + TIME_WINDOWS[tier], partialRewardBlockTimes, reward / REWARD_SPLIT_TRANCHES[tier]);
        _add(tier, stakeInfo);
        proxy.submit("stakingTransfer", abi.encode(address(0), 0, reward, address(this)));
        emit Staked(msg.sender, tier, poolPosition, firstAmount, secondAmount, poolAmount, reward, stakeInfo.endBlock, partialRewardBlockTimes, stakeInfo.splittedReward);
    }

    //Other code...
}
```

```
function stakingTransfer(address sender, uint256, uint256 value, address receiver) public {
    IMVDProxy proxy = IMVDProxy(msg.sender);
    require(IStateHolder(proxy.getStateHolderAddress()).getBool(_toStateHolderKey("staking.transfer.authorized", _toString(sender))), "Unauthorized action!");
    proxy.transfer(receiver, value, proxy.getToken());
}
```

In turn, the stakingTransfer Microservice will check that it is being called only and exclusively by an active staking contract. Therefore, if the community of token holders chooses to have new staking rules, they will vote to deactivate the old staking contract and activate the new one with new rules.
By doing so, all the staking positions of the old staking contract will always be available and redeemable, but it will not be possible to open new positions. At the same time, the new staking contract with the new rules will work normally.

## 🆘 On-Chain Decentralized Exit Strategy

Yet another new security layer for building responsible DeFi dApps is that of predetermined exit strategies for users in a bugged contract. Using Flexible Organizations, this procedure can also be activated by voting.

The circulation of a Stablecoin, for example, should always be based on one or more sources of underlying collateral, which in any case has a value even if taken individually. So it is important to always make it possible to remove it if there is a need, even if only a temporary one.

```
contract AStableCoin {
    function mint(
        uint256 pairIndex,
        uint256 amount0,
        uint256 amount1,
        uint256 amount0Min,
        uint256 amount1Min
    ) public override _forAllowedPair(pairIndex) returns (uint256 minted) {
        require(
            IStateHolder(
                IMVDProxy(IDoubleProxy(_doubleProxy).proxy())
                    .getStateHolderAddress()
            )
                .getBool(
                _toStateHolderKey(
                    "authorizedStableCoin",
                    _toString(address(this))
                )
            ),
            "Unauthorized action!"
        );
        (address token0, address token1, ) = _getPairData(pairIndex);
        _transferTokensAndCheckAllowance(token0, amount0);
        _transferTokensAndCheckAllowance(token1, amount1);
        (uint256 firstAmount, uint256 secondAmount, ) = _createPoolToken(
            token0,
            token1,
            amount0,
            amount1,
            amount0Min,
            amount1Min
        );
        minted =
            fromTokenToStable(token0, firstAmount) +
            fromTokenToStable(token1, secondAmount);
        require(minted <= availableToMint(), "Minting amount is greater than availability");
        _mint(msg.sender, minted);
    }

    function burn(
        uint256 pairIndex,
        uint256 pairAmount,
        uint256 amount0,
        uint256 amount1
    )
        public
        override
        _forAllowedPair(pairIndex)
        returns (uint256 removed0, uint256 removed1)
    {
        (address token0, address token1, address pairAddress) = _getPairData(pairIndex);
        _checkAllowance(pairAddress, pairAmount);
        (removed0, removed1) = IUniswapV2Router(UNISWAP_V2_ROUTER)
            .removeLiquidity(
            token0,
            token1,
            pairAmount,
            amount0,
            amount1,
            msg.sender,
            block.timestamp + 1000
        );
        _burn(
            msg.sender,
            fromTokenToStable(token0, removed0) +
                fromTokenToStable(token1, removed1)
        );
    }
}
```

The code above shows that it is possible to create value in the Stablecoin if and only if it is fully active and working, and as long as its underlying collateral can be withdrawn by the owner in any circumstance.

# 🤓 Conclusion

This Manifesto is designed to start research for a measurable standard in DeFi development. We have already created a repo in Github. We invite every Etherean to update it with us and find the best pragmatic general purposes standard.

Responsible DeFi aims to create an explorer and automatic ratings system for DeFi dApp security, helping users read and evaluate Smart Contracts before trusting them, independent of trust in a Company or Auditor!
