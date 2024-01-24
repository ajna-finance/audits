# Ajna Protocol Security Review

Date: **07.01.24**

Produced by **Kirill Fedoseev** (telegram: [kfedoseev](http://t.me/kfedoseev),
twitter: [@k1rill_fedoseev](http://twitter.com/k1rill_fedoseev))

## Introduction

An independent security review of the Ajna Protocol was conducted by **kfedoseev** from 13.10.23 to 25.10.23. The fixes
review was conducted intermittently from 09.12.23 to 07.01.24. The following methods were used for conducting a security review:

- Manual source code review
- Manual penetration testing
- Manual theoretical and economic analysis of the Ajna Protocol whitepaper
- Manual equivalence analysis between Ajna Protocol whitepaper and it’s Solidity implementation

## Disclaimer

No security review can guarantee or verify the absence of vulnerabilities. This security review is a time-bound process
where I tried to identify as many potential issues and vulnerabilities as possible, using my personal expertise in the
smart contract development and review.

## About Ajna Protocol

The Ajna protocol is a non-custodial, peer-to-pool, permissionless lending, borrowing and trading system that requires
no governance or external price feeds to function. The protocol consists of pools: pairings of quote tokens provided by
lenders and collateral tokens provided by borrowers. Ajna is capable of accepting fungible tokens as quote tokens and
both fungible and non-fungible tokens as collateral tokens.

## Observations and Limitations

* Rebasing or fee on transfer ERC20/ERC721 tokens are not supported by the protocol.
* Quote and collateral ERC20 tokens are required to implement `decimals()` interface and have no more than 18 decimals.
* Quote/collateral price relation (accounted for decimals) is expected to consistently stay within reasonable price
  bounds defined by
  the protocol (i.e. `MIN_PRICE` / `MAX_PRICE`).
* Quote/collateral cannot be added/moved to the bucket at the same block after bucket becomes bankrupt.
* Pool contract does not explicitly prevent internal arithmetic overflows caused by prolonged periods of high interest
  rates. In practice, a maximum interest rate of 400% may break certain pool functionality after several years. Pools
  with already extreme accumulator values close to causing `uint256` overflow in internal arithmetic should not be used.

## Severity classification

| **Severity**           | **Impact: High** | **Impact: Medium** | **Impact: Low** |
|------------------------|------------------|--------------------|-----------------|
| **Likelihood: High**   | Critical         | High               | Medium          |
| **Likelihood: Medium** | High             | Medium             | Low             |
| **Likelihood: Low**    | Medium           | Low                | Low             |

**Impact** - the economic, technical, reputation or other damage to the protocol implied from a successful exploit.

**Likelihood** - the probability that a particular finding or vulnerability gets exploited.

**Severity** - the overall criticality of the particular finding.

## Scope summary

Reviewed whitepaper - https://www.ajna.finance/pdf/Ajna_Protocol_Whitepaper_10-12-2023.pdf (MD5 hash:
ddc29820b45cd9664ff9ace5dd7bae1a)

Reviewed
commit [v0.10.0-rc8](https://github.com/ajna-finance/ajna-core/releases/tag/v0.10.0-rc8) ([dc49848696be62e107689be3295e85994964890c](https://github.com/ajna-finance/ajna-core/tree/dc49848696be62e107689be3295e85994964890c))

Reviewed fixes commit - [v0.10.0-rc9](https://github.com/ajna-finance/ajna-core/releases/tag/v0.10.0-rc9) ([2d6bbcb39a020a041d5243eadd8f4cb77e85c41b](https://github.com/ajna-finance/ajna-core/tree/2d6bbcb39a020a041d5243eadd8f4cb77e85c41b))

Reviewed contracts:

- `src/interfaces/**`
- `src/base/**`
- `src/libraries/**`
- `src/ERC20Pool.sol`
- `src/ERC20PoolFactory.sol`
- `src/ERC721Pool.sol`
- `src/ERC721PoolFactory.sol`
- `src/PoolInfoUtils.sol`
- `src/PoolInfoUtilsMulticall.sol`
- `src/PositionManager.sol`

---

# Findings Summary

| ID     | Title                                                                                   | Severity      | Status       |
|--------|-----------------------------------------------------------------------------------------|---------------|--------------|
| [H-01] | Large borrowers bear a substantial economic risk of spurious liquidations               | High          | Acknowledged |
| [M-01] | Reserve auction starting price calculation contradicts the whitepaper                   | Medium        | Fixed        |                                                                                                                                                                               
| [M-02] | Positions with threshold price below `MIN_PRICE` cannot be kicked                       | Medium        | Fixed        |                                                                                                                                                                              
| [M-03] | ERC20 pool creation may be grieved by frontrunners                                      | Medium        | Acknowledged |                                                                                                                                                                             
| [M-04] | Bond reward amount implicitly depends on the order of takes                             | Medium        | Fixed        |                                                                                                                                                                            
| [M-05] | `addCollateral` allows to manipulate `LUP` and bypass deposit fee                       | Medium        | Fixed        |                                                                                                                                                                             
| [M-06] | Dust collateral checks may lead to unexpected reverts in external integrations          | Medium        | Acknowledged |
| [M-07] | Liquidation of positions with bad debt can cause losses for `HPB` depositors            | Medium        | Fixed        |
| [L-01] | Dust checks for quote tokens are inconsistent                                           | Low           | Fixed        |
| [L-02] | Some unit tests are not properly executed                                               | Low           | Fixed        |
| [L-03] | Redundant call to `ecrecover` in PermitERC721                                           | Low           | Acknowledged |
| [L-04] | PermitERC721 allows replay attacks                                                      | Low           | Acknowledged |
| [L-05] | `maxQuoteToken_` parameter in `_lpToQuoteToken` is redundant                            | Low           | Fixed        |
| [L-06] | Outdated `prb-math` external dependency                                                 | Low           | Acknowledged |
| [L-07] | Unbounded storage list iteration in `getDeployedPoolsList`                              | Low           | Fixed        |
| [L-08] | Reserve auction kick condition can be simplified                                        | Low           | Fixed        |
| [L-09] | Inconsistency between auction curve price calculations                                  | Low           | Acknowledged |
| [L-10] | Missing boundary check on `auctionPrice`                                                | Low           | Acknowledged |
| [L-11] | `_rebalanceTokens` does nothing in `ERC721Pool::take`                                   | Low           | Acknowledged |
| [L-12] | Confusing rounding in `_indexOf`                                                        | Low           | Acknowledged |
| [L-13] | Non-optimal `takeReserves` in case Ajna token is also the quote token in the pool       | Low           | Acknowledged |
| [L-14] | Error-prone logic in `memorializePositions`                                             | Low           | Acknowledged |
| [L-15] | Bucket bankruptcy checks are inconsistent                                               | Low           | Acknowledged |
| [L-16] | Unassigned return value in `findIndexAndSumOfSum`                                       | Low           | Acknowledged |
| [L-17] | `kickReserveAuction` uses outdated pool state                                           | Low           | Acknowledged |
| [I-01] | Precompute value of `PRBMathSD59x18.log2(FLOAT_STEP_INT)`                               | Informational | Acknowledged |
| [I-02] | Simplify EMAs weight calculation                                                        | Informational | Acknowledged |
| [I-03] | Unused error and event definitions                                                      | Informational | Fixed        |
| [I-04] | Assembly optimization in `lsb`                                                          | Informational | Acknowledged |
| [I-05] | Gas optimization in `_auctionPrice`                                                     | Informational | Acknowledged |
| [I-06] | Unnecessary cap for unutilized deposit fee                                              | Informational | Fixed        |
| [I-07] | Simplify `_getCollateralDustPricePrecisionAdjustment`                                   | Informational | Acknowledged |
| [I-08] | Simplify index boundary checks in `_priceAt`                                            | Informational | Acknowledged |
| [I-09] | Redundant `SSTORE` in `ERC721Pool::repayDebt`                                           | Informational | Acknowledged |
| [I-10] | Reimplement `ERC721::removeCollateral` using `LenderActions.removeMaxCollateral`        | Informational | Acknowledged |
| [I-11] | Miners / validators / sequencers are able to manipulate block timestamps to extract MEV | Informational | Acknowledged |
| [I-12] | Simplify condition in `_updateT0Debt2ToCollateral`                                      | Informational | Acknowledged |
| [I-13] | Maths usage optimization                                                                | Informational | Acknowledged |
| [I-14] | Redundant checks in `prefixSum`                                                         | Informational | Acknowledged |
| [I-15] | Redundant `pool_` function parameter in `PositionManager`                               | Informational | Acknowledged |
| [I-16] | Redundant return values in `findIndexAndSumOfSum`                                       | Informational | Acknowledged |
| [I-17] | Duplicated `lpToQuoteTokens` computation in `_removeMaxDeposit`                         | Informational | Acknowledged |
| [I-18] | Unused local variables                                                                  | Informational | Fixed        |

Note: several low-severity and informational findings were identified but not included in the final report, as they were
considered as duplicates from the prior audit reports available for Ajna protocol
from [https://github.com/ajna-finance/audits](https://github.com/ajna-finance/audits) and team has already acknowledged
or accepted them earlier.

# Security & Economic Findings

## [H-01] Large borrowers bear a substantial economic risk of spurious liquidations

Due to the lack of an external oracle in the Ajna protocol design, there is effectively no hard restriction on which
position can be kicked into the liquidation auction. Effectively, any position, even significantly overcollateralized,
can be kicked given `TP >= LUP` and the kicker is willing to put the required bond amount. Moreover, as the is no any
grace period for the kicked liquidations, nor there is a way to re-collateralize and unkick an ongoing auction, borrower
has no ability to prevent their liquidation auction from going through or repay their debt, other than buying back their
own collateral at or slightly above the market price. Let’s call such unreasonable liquidations that are expected to
clear well above the neutral price of the loan - spurious liquidations.

The first `TP >= LUP` condition can be manipulated through arbitrary `LUP` manipulation. There exists two primary ways
to manipulate `LUP`:

1. Via quote token removal trades in higher buckets (see **[M-5]** for deeper analysis of this issue on its own).
2. Via creation of large short-lived borrow positions.

Both approaches are not free for the attacker (although the first one is almost free and should be addressed first),
however their execution cost is very unlikely to become high enough to disincentivize a potential attacker.

The second requirement for the kicker bond simply means that an attacker will need to be ready to forfeit up to 3% of
the kicked debt, as a cost of their spurious liquidation.

There is no direct value that can be extracted by an attacker in such case, however, there exist 2 indirect “incentives”
for them to try to do so:

1. To directly hurt the borrower by making them pay the highest take penalty (150% of the kicker bond), which may be
   higher that the total cost for the attacker.
2. To profit from the market movement predictions following the large collateral market sells. (e.g. by leverage
   shorting a collateral just before kicking a spurious liquidation)
    1. Note that although a mechanism within Ajna protocol exists to settle the debt against the book, collateral is
       still likely to first reach an external market at higher prices.
    2. The amount of extractable profit highly depends on the volatility, market depth, position size and other aspects
       of the specific asset. Though, it’s reasonable to expect that under certain conditions the amount of profit can
       easily surpass forfeited kicker bond capped at 3% of the debt.
    3. Inability to efficiently sell large amounts of collateral at current market prices will imply direct losses for
       the borrower (in addition to the take penalty capped at 4.5% of their debt), as their collateral is being forced
       sold on the open market, until the entire position debt is settled.

### Recommendation

Consider adding back a limited way to re-collateralize, repay or unkick the loan from the ongoing liquidation auction.
Consider increasing the kicker bond size cap to increase the cost for the attacker. Consider implementing suggestions
from **[M-5]**.

## [M-01] Reserve auction starting price calculation contradicts the whitepaper

Ajna protocol whitepaper defines an initial reserve auction price
as `1,000,000,000 AJNA / (total claimable reserves) QT`. Such definition implies that the whole AJNA supply will be
bought back and burned if auction is cleared at the initial price, which is a good choice for initial price bound.

The implementation in `PoolHelper::_reserveAuctionPrice`, however, defines the initial auction price differently
as `1,000,000,000 AJNA / 1 QT`. This adds an extra assumption on the market price of a single quote token.

For example, an artificial ERC20 token with 18 decimals, a reasonable market cap of `$10,000,000` and a total supply
of `1,000,000,000,000 wei` will cause a reserve auction to start at `1 AJNA = $10,000`, which is unreasonably high.

No existing ERC20 tokens with such properties and significant lending market demand were identified.

### Recommendation

Align implementation with the whitepaper by using the total claimable reserves amount in the initial auction price
calculation.

## [M-02] Positions with threshold price below `MIN_PRICE` cannot be kicked

In order for the borrower’s position to be kicked, it should be under-collateralized with respect to the current LUP.
However, as per the pool constraints, LUP cannot go lower than `MIN_PRICE`. Thus, positions with threshold price
below `MIN_PRICE` can’t ever be kicked, irrespective of kicker or lender actions. Even when `lenderKick` is being used,
the total debt is increased by the deposit amount and starts to exceed the overall deposit in the pool, `LUP` will still
be bounded by the `MIN_PRICE`.

### Recommendation

Allow to kick any position, irrespective of its collateralization, if the `LUP` index ever reaches `MAX_FENWICK_INDEX`.

## [M-03] ERC20 pool creation may be grieved by frontrunners

Ajna set of contracts for ERC20 pools assume that at most one lending pool can be created per each unique
quote/collateral token pair. This restriction simplifies the UX and prevents a potential liquidity fragmentation.
However, this also opens an undesired opportunity for grieving. In particular, a malicious party is able to identify a
token pair with high potential demand for borrowing, which is not yet deployed in Ajna. Then, they are able to deploy
the pool pair themselves and spam it with a large number of infinitesimal borrow positions. As the pool is new and does
not have existing positions in it, restriction on min debt amount won’t apply here.

Such spam attack might diminish the UX of core Ajna contracts, as well as all periphery contracts that are using the
fixed factory contract addresses (e.g. PositionManager / RewardsManager).

Same issue applies for ERC721 pools created with `ERC721_NON_SUBSET_HASH` subset hash. Note that for all non-zero subset
hashes, it should always be practically possible to define a new subset hash by adding an arbitrary non-existent token
id in it.

Note that for all existing pools, min debt amount still can be manipulated, although dust borrow DoS attack is mitigated
by a much higher execution cost for the attacker. It’s estimated that for an attacker to reduce the min debt limit by
50%, they will need to effectively double the number of borrow positions in the pool and increase overall pool debt by
another ~7%.

### Recommendation

Allow to deploy duplicate ERC20/ERC721 pools with zero subset hashes through the same factory contract, e.g. by adding a
deployer-defined salt to the last 4 bytes of `ERC20_NON_SUBSET_HASH`/`ERC721_NON_SUBSET_HASH` values.

## [M-04] Bond reward amount implicitly depends on the order of takes

Kicker reward payoff function depends on the current auction price, and it’s relation to the `TP` and `NP` of the
loan. `NP` of the loan gets stamped to the liquidation and does not change throughout the auction. `TP` however is
updated with each successful take (likely decreases).

Impact of the pool’s interest rate to the `TP` throughout the auction during 72 hours is negligible with relation
to `NP` under normal circumstances, as `NP/TP` ratio is bounded from below at `~1.056`

Altering `TP` impacts the bond payoff factor, making it harder for system actors to predict its behavior over time. For
positive bond reward payouts, takers are incentivized to make `TP` as low as possible, as it decreases kicker reward,
increases take penalty reward and the amount of collateral available for purchase. Therefore, rational taker might see
an incentive for themselves to make as many small takes as possible, instead of a single larger one. Rational kicker, on
the other hand, might use this reasoning of receiving a smaller reward than expected, to not participate at all or delay
their kick actions, which is undesirable.

No severe implications of such behavior were identified, however, as it introduces additional disturbance into the
system and allows takers to manipulate, payout curves for everyone in the protocol, it’s looks like an unnecessary
complication. At the same time, as Ajna liquidation mechanism does not allow re-collateralization, one can expect that
multiple takes at the same block should lead cumulatively to the exactly same result as a single larger take, which is
not currently the case.

### Recommendation

Stamp threshold price used for all payout curves computations at the kick time to the liquidation struct, similar to
neutral and reference prices.

## [M-05] `addCollateral` allows to manipulate `LUP` and bypass deposit fee

Each lender deposit in quote tokens in Ajna below `LUP` is subject to a fee, equal to 24 hours of interest. This fee
intends to prevent any future MEV attacks on the pool, e.g. involving unnecessary liquidations.

Under normal circumstances, however, this fee can be bypassed or significantly reduced by manipulating `LUP` downwards.

Depositors at price `P < LUP`, can manipulate `LUP` and avoid deposit fee by sandwiching their quote token deposit in
the following batch:

1. Flash loan a significant amount of collateral token from elsewhere.
2. `addCollateral` to a bucket `index` (with `_priceAt(index) > LUP`) with a large-enough quote token deposit.
3. `removeQuoteToken` from a bucket `index` using obtained LPB, pushing `LUP` just above `HTP`.
4. If `P < HTP`, do the following:
    1. `addCollateral` to all buckets above `P` and below `LUP` that have a significant deposit.
    2. `removeQuoteToken` from all buckets from the previous step.
    3. `drawDebt` a `_minDebtAmount` of quote tokens to push `LUP` below `HTP` and `P` . Only a small debt amount is
       required to push `LUP` lower, as all buckets with significant deposits were drained in the previous step. It’s
       assumed that debt origination fee will be insignificant compared to the unutilized deposit fee on a larger
       amount.
5. `addQuoteToken` to a bucket with price `P`, as `LUP < P` , deposit fee does not apply here.
6. If `P < HTP`, do the following:
    1. `addQuoteToken` to buckets from step 4(ii).
    2. `repayDebt` on a position from step 4(iii).
    3. `removeCollateral` from buckets from step 4(i).
7. `addQuoteToken` to the bucket `index`.
8. `removeCollateral` from the bucket `index`.
9. Repay the flash loan.

Similar sequence of actions can be designed to bypass an unutilized deposit fee applied in `moveQuoteTokens`

### Recommendation

To partially mitigate an issue, consider applying additional restrictions on `addCollateral` operation below `LUP` (e.g.
prohibit them completely, or charge a transitive fee on collateral deposits in buckets with existing quote tokens by
reducing both, added LPB amount and a bucket quote deposit). A more aggressive yet more effective solution might be to
remove `addCollateral` completely, so that collateral can be put into the bucket only in the result of a liquidation
auction.

## [M-06] Dust collateral checks may lead to unexpected reverts in external integrations

Whenever `_bucketCollateralDust` for some bucket index is greater than 1 (i.e. collateral token has less than 18
decimals, or bucket index is greater than 3935), the following sequence of actions may lead to the revert:

```solidity
pool.addCollateral(1 ether, index, block.timestamp);
(uint256 removed, ) = pool.removeQuoteToken(type(uint256).max, index);
pool.addQuoteToken(removed, index, block.timestamp, true);
pool.removeCollateral(type(uint256).max, index); // reverts with DustAmountNotExceeded()
```

Revert above happens as `1 wei` of LPB is lost due to rounding as a result of `removeQuoteToken` + `addQuoteToken`.
Reduced LPB balance is then not enough to claim the whole collateral amount, while the remaining collateral in bucket
can’t stay below dust limit. This may lead to unexpected reverts in surrounding infrastructure and automated contracts
interacting with Ajna contracts.

### Recommendation

Consider implementing a different approach for dealing with dust collateral remainder (e.g. rounding withdrawn
collateral amount up to the total bucket collateral, if the difference between them is less than the dust collateral
amount).

## [M-07] Liquidation of positions with bad debt can cause losses for `HPB` depositors

Liquidation auction settlement prioritizes settling remaining debt with pool reserves to bad debt write-off
against `HPB` deposits. However, if `kickReserveAuction` is invoked after all takes but before auction settlement, pool
reserves might be incorrectly send to the reserve auction instead of going towards bad debt coverage. If the remaining
amount of reserves after `kickReserveAuction` is not enough to cover the remaining debt, than the remaining debt will be
settled against `HPB` deposits and may cause bucket bankruptcy. In the most extreme case, `HPB` depositors risk to
socialize up to `2.5 * bondSize` more bad debt than needed due to the take penalty and kick bond going to the reserve
auction instead of bad debt coverage.

In practice though, this is hardly achievable, as `NP/TP` ratio implied over-collateralization is greater than the
maximum possible take penalty.

### Recommendation

Don’t allow calls to `kickReserveAuction` until all pending auctions are fully settled (e.g. revert
if `auctions.noOfAuctions > 0`).

## [L-01] Dust checks for quote tokens are inconsistent

`LenderActions::RemoveDepositParams::dustLimit` is defined but is left unused in `moveQuoteToken`
and `removeQuoteToken`.

Also, `LenderActions::moveQuoteToken` constrains `maxAmountToMove` from below with `quoteTokenScale` as a dust limit.
However, `LenderActions::removeQuoteToken` is missing such a constraint.

### Recommendation

1. Remove `LenderActions::RemoveDepositParams::dustLimit`.
2. Add dust limit check to the `LenderActions::removeQuoteToken`.

```solidity
if (params_.maxAmount < poolState_.quoteTokenScale)
    revert DustAmountNotExceeded();
```

3. Simplify constraint in `LenderActions::moveQuoteToken`.

```diff
if (params_.maxAmountToMove == 0)
    revert InvalidAmount();
...
-if (params_.maxAmountToMove != 0 && params_.maxAmountToMove < poolState_.quoteTokenScale)
+if (params_.maxAmountToMove < poolState_.quoteTokenScale)
    revert DustAmountNotExceeded();
```

## [L-02] Some unit tests are not properly executed

Forge unit tests `PoolHelperTest::testPriceToIndex` and `PoolHelperTest::testIndexToPrice` do not revert on fail due to
incorrect `vm.expectRevert` checks on the top-level function calls.

See [https://github.com/foundry-rs/foundry/issues/3723](https://github.com/foundry-rs/foundry/issues/3723), [https://github.com/foundry-rs/foundry/issues/5820](https://github.com/foundry-rs/foundry/issues/5820)
for more context.

### Recommendation

Consider rewriting unit tests to check reverts on external calls only.

## [L-03] Redundant call to `ecrecover` in PermitERC721

Chosen EIP4944 implementation allows to call `permit` in 3 different scenarios:

1. To approve a token using an ECDSA signature from the owner EOA.
2. To approve a token using an ECDSA signature from the pre-approved spender EOA.
3. To approve a token using an ERC1271 signature from the owner contract.

Both adopted and used reference EIP4944 implementations perform a duplicated call to the `ecrecover`. First
time, `ecrecover` is called in an explicit call to OpenZeppelin’s `ECDSA.tryRecover`, then another call
to `ECDSA.tryRecover` with the same data is made during the call to `SignatureChecker::isValidSignatureNow`.

### Recommendation

Consider rewriting `permit` function manually to handle all 3 cases in a more efficient and transparent way.

## [L-04] PermitERC721 allows replay attacks

Chosen EIP4944 is designed to prevent permit replay attacks by using a per-token nonce counter. However, stored nonce
counter is only being ever incremented during the token transfer, and not during the call to the permit itself.

Although not mentioned in the draft EIP specification, this approach contradicts overall concept of replay attack
protection, as the same permit signature might potentially be used more than once.

Here is the example of a potential problem it may cause:

1. User issues a “permit” signature for his token to a malicious party through a phishing website.
2. Malicious party successfully relays a permit and obtains a token approval, however does not perform an actual
   transfer yet.
3. User realizes their mistake and revokes an approval by calling an `approve` with zero address.
4. Although the issued permit has already been used and revoked, malicious party is still able to re-use it to get a
   replayed token approval back.

### Recommendation

Increment per-nonce on each successful call to `permit`.

## [L-05] `maxQuoteToken_` parameter in `_lpToQuoteToken` is redundant

In all calls to `_lpToQuoteToken` function, values of `deposit_` and `maxQuoteToken_` are equal to the current bucket
deposit. One of the values along with max cap constraint can be safely removed.

### Recommendation

Remove one of the parameters and one of the max cap constraints.

## [L-06] Outdated `prb-math` external dependency

Ajna source code heavily relies on correctness and effectiveness of fixed-point math implemented in `prb-math` of
version `v2.4.3`. Newer `v4.0.1` version of `prb-math` already exists which addresses several low-severity issues, gas
optimizations and domain expansions that impact `log2`, `exp2`, `exp`, `sqrt`, `pow` used across Ajna codebase:

- [https://github.com/PaulRBerg/prb-math/pull/77](https://github.com/PaulRBerg/prb-math/pull/77)
- [https://github.com/PaulRBerg/prb-math/pull/91](https://github.com/PaulRBerg/prb-math/pull/91)
- [https://github.com/PaulRBerg/prb-math/pull/135](https://github.com/PaulRBerg/prb-math/pull/135)
- [https://github.com/PaulRBerg/prb-math/pull/182](https://github.com/PaulRBerg/prb-math/pull/182)

`v4` version of `prb-math` also have a higher chance of being audited by a third party in the future.

### Recommendation

Upgrade `prb-math` dependency to the latest `v4.0.1` version.

## [L-07] Unbounded storage list iteration in `getDeployedPoolsList`

Function `getDeployedPoolsList` performs an unbounded storage access, while retrieving a list of all deployed pools,
which may eventually lead to an out-of-gas error if performed by an on-chain integrator.

### Recommendation

Remove `getDeployedPoolsList` or specify that it should only be used by off-chain integrations.

## [L-08] Reserve auction kick condition can be simplified

Ajna pool contracts correctly ensure that reserve auctions can be kicked at most once every 2 weeks using the following
condition:

```solidity
// check that at least two weeks have passed since the last reserve auction completed, and that the auction was not kicked within the past 72 hours
if (block.timestamp < lastBurnTimestamp + 2 weeks || block.timestamp - reserveAuction_.kicked <= 72 hours) {
    revert ReserveAuctionTooSoon();
}
```

The comment on the condition incorrectly states that “two weeks have passed since the last reserve auction completed”,
although the code suggests that 2 weeks should have passed since it was kicked.

It can also be noted, however, that `lastBurnTimestamp` is always equal to `reserveAuction_.kicked`, as they both are
being set to the `block.timestamp` at the end of the previous successful kick. Therefore, the condition can be
simplified to the following one:

```solidity
// check that at least two weeks have passed since the last reserve auction was kicked
if (block.timestamp < reserveAuction_.kicked + 2 weeks) {
    revert ReserveAuctionTooSoon();
}
```

This will also save gas on one redundant `SLOAD` by removing the need in reading `lastBurnTimestamp`.

### Recommendation

Update the misleading comment, simplify the condition, remove redundant `lastBurnTimestamp` `SLOAD`.

## [L-09] Inconsistency between auction curve price calculations

Ajna protocol uses two dutch auctions in its design: liquidation and reserve auctions. Although their design is very
similar, implementation uses different approaches for computing a price on each of two curve.

Liquidation curve computation uses `PRBMathSD59x18.exp2` exponentiation with 1 second price step granularity. Reserve
auction curve computation uses `Maths.rpow` exponentiation with 60 seconds price step granularity. Intentions behind
using different approaches for computing similar curves are unclear.

Whitepaper section 9.1 on reserve auctions also incorrectly defines the reserve auction price curve, which is likely a
copy-paste error from the sections related to the liquidation auctions.

### Recommendation

Consider using the same approach and price step granularity when computing auction curve prices, update the whitepaper
section 9.1.

## [L-10] Missing boundary check on `auctionPrice`

There is the following code in `SettlerActions::_settleAuction`:

```solidity
uint256 auctionPrice = _auctionPrice(
    auctions_.liquidations[borrowerAddress_].referencePrice,
    auctions_.liquidations[borrowerAddress_].kickTime
);

// determine the bucket index to compensate fractional collateral
bucketIndex = auctionPrice > MIN_PRICE ? _indexOf(auctionPrice) : MAX_FENWICK_INDEX;
```

The last statement misses a boundary check at `MAX_PRICE`. The `auctionPrice` can theoretically be higher
than `MAX_PRICE`, as it can start as high as `256 * 50 * MAX_PRICE`.

This, though, cannot cause any long-term issues, as an auction settler can always wait a few hours for and auction price
to organically drop within an accepted range.

### Recommendation

Add a boundary check at `MAX_PRICE`.

## [L-11] `_rebalanceTokens` does nothing in `ERC721Pool::take`

Function `ERC721Pool::take` ensures that bought collateral is transferred to the taker
using `_transferFromPoolToAddress` which removes all excess NFTs from `borrowerTokenIds[borrowerAddress_]` list. No
collateral is expected to be transferred to the claimable NFTs list. Therefore, a call to `_rebalanceTokens`
in `ERC721Pool::take` will effectively do nothing and can be removed.

### Recommendation

Remove the redundant call to `_rebalanceTokens`.

## [L-12] Confusing rounding in `_indexOf`

The following piece of code exists in `_indexOf`:

```solidity
int256 ceilIndex = PRBMathSD59x18.ceil(index);
if (index < 0 && ceilIndex - index > 0.5 * 1e18) {
    return uint256(4157 - PRBMathSD59x18.toInt(ceilIndex));
}
return uint256(4156 - PRBMathSD59x18.toInt(ceilIndex));
```

Effectively, this code uses different rounding approaches for different indices: flooring for positive indices and
rounding to closest for negative. Intentions behind such choice are unclear and cause inconsistency in price ranges. For
example, bucket `4156` now covers only half of its supposed price interval - `[0.9975..1]` instead of `[0.9975..1.0025]`
or `[0.995..1]`.

### Recommendation

Remove confusing if statement.

## [L-13] Non-optimal `takeReserves` in case Ajna token is also the quote token in the pool

In the rare case when the quote token in the pool is the Ajna token itself, the reserve auction will attempt to sell
accumulated Ajna tokens in reserves for a likely lower Ajna token amount, which is not efficient.

### Recommendation

In `kickReserveAuction` consider immediately burning all claimable reserves in case quote token address is equal to the
Ajna token address.

## [L-14] Error-prone logic in `memorializePositions`

Function `memorializePositions` does not explicitly reject duplicated indices passed in the function arguments. In such
case, function proceeds to update an internal state and double accounts for lender’s LPB, before an actual LPB transfer.
The execution flows with duplicated indices finally reaches `transferLP` call and reverts with a
non-obvious `NoAllowance` error.

Relying on such revert is error-prone and may cause non-obvious high severity issues in case of refactoring allowance
logic in the future.

### Recommendation

Consider explicitly restricting uniqueness of passed indices in `memorializePositions`, e.g. by requiring them to be
passed in ascending order.

## [L-15] Bucket bankruptcy checks are inconsistent

Functions `moveQuoteToken`, `removeQuoteToken`, `removeCollateral`, `mergeOrRemoveCollateral` and `settle` include a
bucket bankruptcy logic, which intends to nullify all remaining LPBs in the corresponding bucket, when the sum of bucket
assets reaches zero.

However, there are two different ways in which the protocol checks for potential bucket bankruptcy:

```solidity
// pseudocode for the approach used in the moveQuoteToken, removeQuoteToken,
// removeCollateral, mergeOrRemoveCollateral and settle functions
if (remainingCollateral == 0 && remainingDeposit == 0 && remainingLP != 0) {
    bucket.lps            = 0;
    bucket.bankruptcyTime = block.timestamp;

    emit BucketBankruptcy(index, remainingLP);
}

// pseudocode for the approach used in the settle::_forgiveBadDebt function
if (remainingCollateral * price + remainingDeposit * Maths.WAD <= remainingLP) {
    bucket.lps            = 0;
    bucket.bankruptcyTime = block.timestamp;

    emit BucketBankruptcy(index, remainingLP);
}
```

The second approach may seem better as it nullifies any buckets with infinitesimal assets, although it also leaves the
small value of the remaining collateral permanently locked in the contract, as no existing depositor will be able to
withdraw it due to an updated bankruptcy timestamp.

As the code also handles bucket bankruptcy differently in different places, external integrations may have troubles
restoring the correct state of the pool based on the `BucketBankruptcy` emitted events.

### Recommendation

Consider choosing a single bankruptcy handling logic and apply it in all cases uniformly.

## [L-16] Unassigned return value in `findIndexAndSumOfSum`

One of the return values `sumIndexScale_` may be left unassigned (i.e. 0) if `targetSum_ > treeSum()`. Although no
negative implications of such behavior were identified in the current code, this is generally error-prone, as the
returned `scale` value is later used in multiplications and divisions.

### Recommendation

Initialize `sumIndexScale` to `Maths.WAD` or implement suggestions from **[I-16]**

## [L-17] `kickReserveAuction` uses outdated pool state

Function `kickReserveAuction` calculates the reserve amounts based on the up-to-date pool quote token balance, but
outdated pool size and total debt information. This is generally error-prone and can lead to small difference in the
result of the calculation.

### Recommendation

Update the pool state in the start of the `kickReserveAuction` execution using `_accruePoolInterest()`, similar to other
pool functions.

# Informational & Gas Optimizations

## [I-01] Precompute value of `PRBMathSD59x18.log2(FLOAT_STEP_INT)`

Precompute value of `PRBMathSD59x18.log2(FLOAT_STEP_INT)` in `PoolHelper.sol` to reduce the runtime gas costs of all
common operations that are using `_priceAt` and `_indexOf` functions.

```diff
-/// @dev step amounts in basis points. This is a constant across pools at `0.005`, achieved by dividing `WAD` by `10,000`
-int256 constant FLOAT_STEP_INT = 1.005 * 1e18;
+/// @dev log2 of step amounts in basis points. This is a constant across pools at `0.005`, achieved by dividing `WAD` by `200`, and then taking a log2
+int256 constant FLOAT_STEP_INT_LOG2 = 0.007195501404203907 * 1e18;
```

## [I-02] Simplify EMAs weight calculation

Current implementation of EMAs weights calculation in `PoolCommons::updateInterestState` uses `PRBMathSD59x18.exp`.
However, `PRBMathSD59x18.exp` implementation uses `PRBMathSD59x18.exp2` under the hood. Therefore, it’ll be more
efficient to use it directly whenever base 2 exponentiation is needed:

```diff
-vars.elapsed   = int256(Maths.wdiv(block.timestamp - vars.lastEmaUpdate, 1 hours));
-vars.weightMau = PRBMathSD59x18.exp(PRBMathSD59x18.mul(NEG_H_MAU_HOURS, vars.elapsed));
-vars.weightTu  = PRBMathSD59x18.exp(PRBMathSD59x18.mul(NEG_H_TU_HOURS,  vars.elapsed));
+vars.elapsed   = -int256(Maths.wad(block.timestamp - vars.lastEmaUpdate));
+vars.weightMau = PRBMathSD59x18.exp2(vars.elapsed / 12 hours);
+vars.weightTu  = PRBMathSD59x18.exp2(vars.elapsed / 84 hours);
```

## [I-03] Unused error and event definitions

The following errors from the `IPoolErrors.sol` are unused and their definitions can be safely removed from the
codebase:

- `error AllowanceAlreadySet()`
- `error LUPGreaterThanTP()`
- `error PoolUnderCollateralized()`

The following errors and events from the `IRewardsManagerErrors.sol` and `IRewardsManagerEvents.sol` are unused and
their definitions can be safely removed from the codebase:

- `error MoveStakedLiquidityInvalid()`
- `event MoveStakedLiquidity(uint256 tokenId, uint256[] fromIndexes, uint256[] toIndexes)`

## [I-04] Assembly optimization in `lsb`

Current implementation of `lsb` in `Deposits.sol` can be significantly simplified and gas-optimized by using an assembly
script:

```diff
function lsb(
    uint256 i_
) internal pure returns (uint256 lsb_) {
-   if (i_ != 0) {
+   assembly {
        // "i & (-i)"
-       lsb_ = i_ & ((i_ ^ 0xffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffffff) + 1);
+       lsb_ := and(i_, add(not(i_), 1))
    }
}
```

## [I-05] Gas optimization in `_auctionPrice`

Consider the following gas optimization and simplification of the `_auctionPrice`, which removes unnecessary
arithmetics.

```solidity
function _auctionPrice(
    uint256 referencePrice_,
    uint256 kickTime_
) view returns (uint256 price_) {
    uint256 elapsedSeconds = block.timestamp - kickTime_;

    int256 timeAdjustment;
    if (elapsedSeconds < 2 hours) {
        timeAdjustment = -int256(Maths.wad(elapsedSeconds) / 20 minutes);
        price_ = 256 * Maths.wmul(referencePrice_, uint256(PRBMathSD59x18.exp2(timeAdjustment)));
    } else if (elapsedSeconds < 14 hours) {
        timeAdjustment = -int256(Maths.wad(elapsedSeconds - 2 hours) / 2 hours);
        price_ = 4 * Maths.wmul(referencePrice_, uint256(PRBMathSD59x18.exp2(timeAdjustment)));
    } else {
        timeAdjustment = -int256(Maths.wad(elapsedSeconds - 14 hours) / 1 hours);
        price_ = Maths.wmul(referencePrice_, uint256(PRBMathSD59x18.exp2(timeAdjustment))) / 16;
    }
}
```

It’s estimated that this optimization will save up to `800` gas on each call to `_auctionPrice`.

## [I-06] Unnecessary cap for unutilized deposit fee

Function `_depositFeeRate` caps fee rate at 10% from the above, however, as the max interest rate is capped at 400%, 24
hours of interest is already practically capped at ~1.1%, making the 10% cap obsolete.

## [I-07] Simplify `_getCollateralDustPricePrecisionAdjustment`

Function `_getCollateralDustPricePrecisionAdjustment` performs unnecessary fixed-point operations, although the final
result of those operations is still being floored to the nearest integer. Consider using integer math instead to save on
gas costs and code complexity:

```diff
function _getCollateralDustPricePrecisionAdjustment(
    uint256 bucketIndex_
) pure returns (uint256 pricePrecisionAdjustment_) {
    // conditional is a gas optimization
-   if (bucketIndex_ > 3900) {
-       int256 bucketOffset = int256(bucketIndex_ - 3900);
-       int256 result = PRBMathSD59x18.sqrt(PRBMathSD59x18.div(bucketOffset * 1e18, int256(36 * 1e18)));
-       pricePrecisionAdjustment_ = uint256(result / 1e18);
+   if (bucketIndex_ > 3935) {
+       uint256 bucketOffset = bucketIndex_ - 3900;
+       pricePrecisionAdjustment_ = PRBMath.sqrt(bucketOffset / 36);
    }
}

```

Keep in mind the following issue in
the `prb-math` `v2.4.3` - [https://github.com/PaulRBerg/prb-math/issues/97](https://github.com/PaulRBerg/prb-math/issues/97)
from the **[L-6]** finding.

As a side note, the reasoning behind choosing such specific formula is unclear. A more straightforward and simpler way
to impose a lower bound on the exchange rate may be as follows:

```solidity
if (bucketIndex_ > 3460) {
    // 461 == floor(log(1.005, 10))
    pricePrecisionAdjustment_ = (bucketIndex_ - 3000) / 461;
}
```

## [I-08] Simplify index boundary checks in `_priceAt`

In `_priceAt`, out of bounds check can be simplified in the following way:

```diff
-int256 bucketIndex = MAX_BUCKET_INDEX - int256(index_);
-if (bucketIndex < MIN_BUCKET_INDEX || bucketIndex > MAX_BUCKET_INDEX) revert BucketIndexOutOfBounds();
+if (index_ > MAX_FENWICK_INDEX) revert BucketIndexOutOfBounds();
+int256 bucketIndex = MAX_BUCKET_INDEX - int256(index_);
```

## [I-09] Redundant `SSTORE` in `ERC721Pool::repayDebt`

The following `SSTORE` can be performed only when `noOfNFTsToPull_ != 0`, this will also make it consistent
with `ERC20Pool::repayDebt` and `ERC721::drawDebt`.

```solidity
// update pool balances pledged collateral state
poolBalances.pledgedCollateral = poolState.collateral;
```

## [I-10] Reimplement `ERC721::removeCollateral` using `LenderActions.removeMaxCollateral`

Currently `ERC721::removeCollateral` is the only function in the codebases that relies on
the `LenderActions.removeCollateral`. At the same time, a very similar `LenderActions.removeMaxCollateral` function
already exists. It seems to be possible to implement `ERC721::removeCollateral` using the
same `LenderActions.removeMaxCollateral` function, which will allow to get rid of the `LenderActions.removeCollateral`
and shorten the codebase.

```diff
-removedAmount_ = Maths.wad(noOfNFTsToRemove_);
+maxRemovedAmount_ = Maths.wad(noOfNFTsToRemove_);
-removedAmount_ = LenderActions.removeCollateral(
+(removedAmount_, redeemedLP_) = LenderActions.removeMaxCollateral(
     buckets,
     deposits,
+    1,
-    removedAmount_,
+    maxRemovedAmount_,
     index_
 );
+if (removedAmount_ != maxRemovedAmount_) revert InsufficientLP();
```

## [I-11] Miners / validators / sequencers are able to manipulate block timestamps to extract MEV

Depending on the particular consensus mechanism used in a specific L1/L2, block producers will almost always be able to
manipulate produced block timestamp to extract an additional profit from the protocol.

In general, this risk can be also combined with a risk of a DoS attack on the network preventing transaction inclusion.

In Ajna, the most prominent way to extract MEV via timestamp manipulation is to attempt delaying a large liquidation
auction, in which an affiliated actor is both, a kicker and a taker. For example, if a block producer is able to find a
reliable way to delay their block by 10 minutes at the right time, they’ll be able to extract up
to `1 - 0.5^(10 minutes / 2 hours) ~= 5.6%` of the auctioned debt.

In the PoS Ethereum, block timestamp are fixed at 12 second intervals. However, a malicious block proposer can still
manipulate block timestamps to a certain level by deliberately choosing not to produce several blocks in row or include
specific transactions in them. In other L1 / L2 networks, the realistic margin for manipulation may be even higher and
should be carefully assessed on a case-by-case basis prior to the protocol deployment.

## [I-12] Simplify condition in `_updateT0Debt2ToCollateral`

Function `_updateT0Debt2ToCollateral` is designed to update a state metric whenever borrower’s t0 debt or collateral
changes. Currently, the following condition determines if state update is required, which can be simplified as follows:

```diff
-if (debt2ColAccumPreAction != 0 || debt2ColAccumPostAction != 0) {
+if (debt2ColAccumPreAction != debt2ColAccumPostAction) {
     uint256 curT0Debt2ToCollateral = interestState.t0Debt2ToCollateral;
     curT0Debt2ToCollateral += debt2ColAccumPostAction;
     curT0Debt2ToCollateral -= debt2ColAccumPreAction; 

     interestState.t0Debt2ToCollateral = curT0Debt2ToCollateral;
 }
```

## [I-13] Maths usage optimization

Math WAD-based library is being extensively used in the code. As library provides multiple functions using different
rounding techniques for multiplication and division, with `floorWmul` and `floorWdiv` being the cheapest gas-wise
option, as it’s a default solidity behavior for integer division. In many places in the code in which rounding is not
important and cannot cause any errors, the `wmul` and `wdiv` are being used as the default choice for doing arithmetic.
Consider rewriting such computations to use `floorWmul` and `floorWdiv` instead, to save on gas.

```diff
// function _minDebtAmount
-minDebtAmount_ = Maths.wdiv(Maths.wdiv(debt_, Maths.wad(loansCount_)), 10**19);
+minDebtAmount_ = Maths.floorWdiv(debt_, Maths.wad(loansCount_ * 10));

// function accrueInterest
-Maths.wmul(pendingFactor - Maths.WAD, Maths.wad(10))
+Maths.floorWmul(pendingFactor - Maths.WAD, Maths.wad(10))

// ...
```

In some places, code can be further simplified by inlining some of the `Maths` functions involving integer constants:

```diff
// function _minDebtAmount
-minDebtAmount_ = Maths.wdiv(Maths.wdiv(debt_, Maths.wad(loansCount_)), 10**19);
+minDebtAmount_ = debt_ / (loansCount_ * 10);

// function accrueInterest
-Maths.wmul(pendingFactor - Maths.WAD, Maths.wad(10))
+(pendingFactor - Maths.WAD) / 10

// function _lenderInterestMargin
-return 1e18 - Maths.wdiv(Maths.wmul(crpud, 0.15 * 1e18), CUBIC_ROOT_1000000);
+return 1e18 - Maths.floorWmul(crpud, 0.0015 * 1e18);

// ...
```

In some places, combination of `wdiv` + `wmul` can be efficiently replaced by `mulDiv` operation:

```diff
// function _dwatp
-return t0Debt_ == 0 ? 0 : Maths.wdiv(Maths.wmul(inflator_, t0Debt2ToCollateral_), t0Debt_);
+return t0Debt_ == 0 ? 0 : Math.mulDiv(inflator_, t0Debt2ToCollateral_, t0Debt_);

// function _lenderInterestMargin
-return 1e18 - Maths.wdiv(Maths.wmul(crpud, 0.15 * 1e18), CUBIC_ROOT_1000000);
+return 1e18 - Math.mulDiv(crpud, 0.15 * 1e18, CUBIC_ROOT_1000000);
```

## [I-14] Redundant checks in `prefixSum`

The following if condition in `prefixSum` are always evaluated to `false` as `index` is being bit-by-bit reconstructed
from `sumIndex` and therefore cannot be greater than `sumIndex`:

```solidity
// Skip considering indices outside bounds of Fenwick tree
if (curIndex > SIZE) continue;
```

This condition can be either completely removed, if it’s assumed that input `sumIndex_` is not greater than `SIZE` , or
moved to the top of the function to check that `sumIndex <= SIZE` only once.

The following condition in `prefixSum` can evaluate to true only during the last loop iteration (as the loop only goes
down to the least significant bit of the `sumIndex`) and therefore can be removed due to redundancy:

```solidity
// terminate if we've already matched sumIndex_
if (index == sumIndex_) break;
```

The initial value for local variable `j` can be reduced from `8192` to `4096` if its assumed that indices do not
exceed `MAX_FENWICK_INDEX` and root fenwick node is never being scaled. This helps to save gas on 1 redundant `SLOAD`.

## [I-15] Redundant `pool_` function parameter in `PositionManager`

Multiple functions in `PositionManager` accept `pool_` parameter together with `tokenId_`:

- `burn`
- `memorializePositions`
- `moveLiquidity`
- `redeemPositions`

However, as pool address is stamped to the token id at its creation and cannot be changed later, it's not necessary to
cross-check it each time in the `mayInteract` modifier. As such, all of the above functions can be simplified by
removing `pool_` parameter and using the pool address from the token associated struct.

## [I-16] Redundant return values in `findIndexAndSumOfSum`

One of the return values `sumIndexSum_` of the function `findIndexAndSumOfSum` is not used in any of the calls and can
be removed to save a little bit of gas.

One of the return values `sumIndexScale_` is assigned to the local variable `runningScale` each time `runningScale` is
changed. Consider getting rid of `runningScale` in favor of assigning `sumIndexScale` directly.

Return value `sumIndexScale_` is also only used whenever `targetSum_ == 1`. Such specific case can be gas-optimised
using a simple lookup for the highest bucket with non-zero unscaled deposit:

```solidity
function findHighestDeposit(DepositsState storage deposits_) internal view returns (uint256 hpbIndex_) {
    uint256 i = 4096;

    while (i > 0) {
        if (deposits_.values[hpbIndex_ + _i] == 0) {
            hpbIndex_ += _i;
        }
        i = i >> 1;
    }
}
```

This would save 13 `SLOAD` for finding an `HPB` index (as we don't need scale values here), although it will
require a separate lookup for a current scale value at the `HPB` bucket, which uses <13 `SLOAD` on average.

## [I-17] Duplicated `lpToQuoteTokens` computation in `_removeMaxDeposit`

The following computation can be simplified, as the same value has been already computed previously:

```diff
-removedAmount_ = Buckets.lpToQuoteTokens(
-    params_.bucketCollateral,
-    params_.bucketLP,
-    scaledDepositAvailable,
-    redeemedLP_,
-    params_.price,
-    Math.Rounding.Down
-);
+removedAmount_ = scaledLpConstraint;
```

## [I-18] Unused local variables

The following local variables are unused after being set and therefore can be safely removed from the codebase:

1. `TakeLocalVars::factor`
2. `DrawDebtLocalVars::compensatedCollateral`
3. `RepayDebtLocalVars::compensatedCollateral`
4. `KickResult::poolDebt`
5. `RemoveDepositParams::dustLimit` (see **[L-01]** for details)
6. `ConstructTokenURIParams::pool`
7. `ConstructTokenURIParams::indexes`

# Out-of-scope findings

Conducted security review has also unveiled the following high-severity issue in the `src/RewardsManager.sol` contract.
The contract was not deemed to be part of the defined scope of work and therefore was excluded from the main report.

| ID     | Title                                                                   | Severity |
|--------|-------------------------------------------------------------------------|----------|
| [X-01] | Anyone can steal all rewards allocated for bucket exchange rate updates | High     |

## [X-01] Anyone can steal all rewards allocated for bucket exchange rate updates

In the `RewardsManager` contract, amount of rewards paid for bucket exchange rate updates linearly depends on the bucket
deposit amount, as per `_updateBucketExchangeRateAndCalculateRewards` implementation:

```solidity
// retrieve current deposit of the bucket
(, , , uint256 bucketDeposit, ) = IPool(pool_).bucketInfo(bucketIndex_);

uint256 burnFactor = Maths.wmul(totalBurned_, bucketDeposit);

// calculate rewards earned for updating bucket exchange rate 
rewards_ = interestEarned_ == 0 ? 0 : Maths.wdiv(
    Maths.wmul(
        UPDATE_CLAIM_REWARD,
        Maths.wmul(
            burnFactor,
            curBucketExchangeRate - prevBucketExchangeRate
        )
    ),
    Maths.wmul(curBucketExchangeRate, interestEarned_)
);
```

An attacker can manipulate `bucketDeposit` and the above formula in the following way:

1. Wait until a new epoch starts in the Ajna pool with the reserve auction kick.
2. Flash loan/mint a large amount of quote tokens from elsewhere.
3. Add borrowed quote tokens to any bucket above `LUP` that is earning interest.
4. Call `updateBucketExchangeRatesAndClaim` with bucket index chosen above.
5. Remove quote tokens from the chosen bucket.
6. Repay the flash loan/mint.

Essentially, an attacker convinces `RewardsManager` that their bucket earned all the interest in pool, and therefore
should get `UPDATE_CAP * totalBurnedInEpoch` as a reward.

Total continuous profit for the attacker is limited by 10% of all burned Ajna tokens, as they are able to repeat this
attack for all pools in the protocol.

As the attacker withdraws maximum amount of rewards allocated for bucket exchange rate updates, no one else will be able
to receive such rewards, unless they try to frontrun the attacker.

### Recommendation

Change the rewards formula for bucket exchange rate updates, or remove them completely.
