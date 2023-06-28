# Ajna Protocol security audits
## 1. Sherlock 1st contest (Jan 9, 2023 - Jan 30, 2023)
#### Contest: https://app.sherlock.xyz/audits/contests/32

#### Report: [Sherlock 1 Report](./sherlock/Contest1.md)
#### Scope: [Protocol](https://github.com/ajna-finance/ajna-core) and [Grant Coordination Fund](https://github.com/ajna-finance/ajna-grants) Contracts
#### Findings: 11 High, 22 Medium

| Finding | Severity | Resolution |
| --- | --- | --- |
| **Issue H-1**: `RewardsManager` doesn't delete old bucket snapshot info on unstaking | High | Fixed |
| **Issue H-2**: Anyone who approved quote tokens to a pool can be forced to take | High | Fixed |
| **Issue H-3**: `CryptoPunks` NFTs may be stolen via deposit frontrunning | High | Fixed |
| **Issue H-4**: `scaledQuoteTokenAmount` isn't updated to be collateral sell value in the quote token constraint case of `_calculateTakeFlowsAndBondChange` | High | Fixed |
| **Issue H-5**: `removeCollateral` miss bankrupcy logic and can make future `LP`s sharing losses with the current ones | High | Fixed |
| **Issue H-6**: Executing funded standard proposals can be prevented by a proposal slate with duplicate proposals | High | Fixed |
| **Issue H-7**: `ERC721Pool`'s `mergeOrRemoveCollateral` allows to remove collateral while auction is clearable | High | Fixed |
| **Issue H-8**: Remaining collateral used by `ERC721Pool` is missed in `Auctions` `take` and `bucketTake` return structures | High | Fixed |
| **Issue H-9**: `moveQuoteToken()` can cause bucket to go bankrupt but it is not reflected in the accounting | High | Fixed |
| **Issue H-10**: `ERC721Pool`'s take will proceed with truncated collateral amount and full debt when borrower's collateral is fractional | High | Fixed |
| **Issue H-11**: The deposit / withdraw / trade transaction lack of expiration timestamp check and slippage control | High | Fixed |
| **Issue M-1**: Buypunk function of `Cryptopunks` in `ERC721Pool` is used incorrectly | Medium | Fixed |
| **Issue M-2**: `ERC721Pool` taker callback misreports quote funds whenever there was collateral amount rounding | Medium | Fixed |
| **Issue M-3**: Anyone can transfer approved `LP` tokens | Medium | Fixed |
| **Issue M-4**: Incorrect `MOMP` calculation in neutral price calculation | Medium | Fixed |
| **Issue M-5**: Extraordinary proposals can receive more tokens than eligible | Medium | Fixed |
| **Issue M-6**: Claiming rewards from a future not yet existing epoch prevents claiming rewards for those epochs later on | Medium | Fixed |
| **Issue M-7**: Calculating new rewards is susceptible to precision loss due to division before multiplication | Medium | Fixed |
| **Issue M-8**: Claiming accumulated rewards while the contract is underfunded can lead to a loss of rewards | Medium | Fixed |
| **Issue M-9**: Incorrect Validation in `Pool.sol#transferLPs` lead to a DOS attack | Medium | Fixed |
| **Issue M-10**: Adversary can grief kicker by frontrunning `kickAuction` call with a large amount of loan | Medium | Fixed |
| **Issue M-11**: Settled collateral of a borrower aren't available for lenders until borrower's debt is fully cleared | Medium | Fixed |
| **Issue M-12**: Deposits are eliminated before currently unclaimed reserves when there is no reserve auction | Medium | No fix, Docs updated |
| **Issue M-13**: Flashloan end result isn't controlled | Medium | Fixed |
| **Issue M-14**: Interest rates can be raised above the market as a griefing, disabling the pool | Medium | Fixed |
| **Issue M-15**: Interest rate for pool is bounded wrongly | Medium | Fixed |
| **Issue M-16**: Auction timers following liquidity can fall through the floor price causing pool insolvency | Medium | No fix, Docs updated |
| **Issue M-17**: If borrower or kicker got blacklisted by asset contract their collateral or bond funds can be permanently frozen with the pool | Medium | Fixed |
| **Issue M-18**: user can `drawDebt` that is below dust amount | Medium | Fixed |
| **Issue M-19**: Quadratic voting tally done wrong | Medium | Fixed |
| **Issue M-20**: `CryptoKitty` and `CryptoFighter` NFT can be paused, which block borrowing / repaying / liquidating action in the `ERC721Pool` when borrowers still forced to pay the compounding interest | Medium | Fixed |
| **Issue M-21**: Minting an NFT with a position on the same bucket as a previously minted NFT changes its deposit time | Medium | Fixed |
| **Issue M-22**: Memorializing an NFT position on the same bucket of a previously memorialized NFT locks redemption | Medium | Fixed |

## 2. Trail Of Bits Audit (Feb 13, 2023 - April 3, 2023)
#### Report: [Trail Of Bits Report](./tob/Ajna_Final_Report+Fix_Review-4.24.2023.pdf) 
#### Scope: [Protocol](https://github.com/ajna-finance/ajna-core) and [Grant Coordination Fund](https://github.com/ajna-finance/ajna-grants) Contracts
#### Findings: 1 High, 1 Medium, 3 Lows

| Finding | Severity | Resolution |
| --- | --- | --- |
| **TOB-AJNA-1**: Solidity compiler optimizations can be problematic | Undetermined | No fix (contracts exceed spurious dragon limit) |
| **TOB-AJNA-2**: `findIndexAndSumOfSums` ignores global scalar | Informational | No fix, improved invariant testing |
| **TOB-AJNA-3**: Incorrect inflator arithmetic in view functions | Low | Fixed |
| **TOB-AJNA-4**: Interest rates can become extreme, allowing DoS attacks | Medium | Fixed |
| **TOB-AJNA-5**: Older versions of external libraries are used | Informational | `OZ` updated to 4.8.2 in contracts repo. `PRBMaths` not updated (requires code change) |
| **TOB-AJNA-6**: Extraordinary proposal can be used to steal extraordinary amounts of Ajna | High | Fixed |
| **TOB-AJNA-7**: `findMechanismOfProposal` function could shadow an extraordinary proposal | Low | Fixed |
| **TOB-AJNA-8**: Array lengths are not checked in `LP` allowance update functions | Low | Fixed |

#### Improvements:

| Description |
| --- |
| `UPDATE_CLAIM_REWARD` state var should be constant |
| Check for `0x` address in `RewardsManager` constructor |
| Use consistent variable and function naming conventions |
| Ensure that comments in code always reflect the code’s intended behavior |
| Use defined helper functions instead of duplicating code |
| Emit event when `Flashloan` taken |
| In Funding.sol#L117 perform check for an invalid proposal outside the loop |

## 3. Quantstamp Audit (April 24, 2023 - May 3, 2023)
#### Report: [Quantstamp Report](./quantstamp/Ajna_Finance_Governance_-_Report.pdf) | [Quantstamp Certificate](https://certificate.quantstamp.com/full/ajna-finance-governance/9ef9afc2-a7ab-42d3-bfbb-a2f6695c4cac/index.html)
#### Scope: [Grant Coordination Fund](https://github.com/ajna-finance/ajna-grants) Contracts
#### Findings: 3 High, 4 Lows, 4 Informational, 2 Undetermined

| Finding | Severity | Resolution |
| --- | --- | --- |
| **AJN-1**: Delegate Rewards Calculation Takes Into Account All Funding Voters | High | Fixed |
| **AJN-2**: Locked Funds Due to Non-Executable Proposals | High | Fixed |
| **AJN-3**: Impact of Unaccounted Delegation Rewards on Treasury and GBC Calculation | High | Fixed |
| **AJN-4**: Gas Usage / Loop Concerns | Low | Acknowledged |
| **AJN-5**: Missing Input Validation | Low | Fixed |
| **AJN-6**: Inconsistent Time Period | Low | Fixed |
| **AJN-7**: Zero ERC20 Transfer | Low | Fixed |
| **AJN-8**: Old Version of Solidity | Informational | Fixed |
| **AJN-9**: Multiple Possible 0-1 Knapsack Solutions, but only the First One Gets Accepted  | Informational | Fixed |
| **AJN-10**: Outdated Proposal Hashing Function | Informational | Fixed |
| **AJN-11**: Greedy Contract | Informational | Acknowledged |
| **AJN-12**: Extraordinary Funding Proposal Success Condition Discrepancies | Undetermined | Fixed |
| **AJN-13**: Success of the Extraordinary Funding Is Susceptible to Treasury Balance Variability | Undetermined | Fixed |

## 4. Prototech Audit (April 26, 2023 - June 5, 2023)
#### Report: [Prototech Report](./prototech/13062023-Prototech-Ajna-Audit.pdf)
#### Scope: [Protocol Contracts](https://github.com/ajna-finance/ajna-core)
#### Findings: 1 Critical, 3 High, 2 Medium, 15 Low, 12 Informational

| Finding | Severity | Resolution |
| --- | --- | --- |
| **PROTOTECH-37**: Position Manager allows for stealing all LPs | Critical | Fixed |
| **PROTOTECH-57**: Collateral Can Be Extracted Without Redeeming LP Shares | High | Fixed |
| **PROTOTECH-42**: General Recommendation: Rounding In Favor of the Interacting User Is Dangerous | High | Fixed |
| **PROTOTECH-39**: Pools round against themselves and in favor of borrower | High | Fixed |
| **PROTOTECH-46**: Unsafe casts in KickerActions | Medium | Fixed |
| **PROTOTECH-18**: Missing Scaling for ERC-721 Pools’ Quote Token on repayment | Medium | Fixed |
| **PROTOTECH-55**: Extremely large debt positions could prevent new borrowers to a pool while severely limiting existing borrowers | Low | Acknowledged |
| **PROTOTECH-53**: Large position size may discourage kicking because of the resulting large bond size | Low | Acknowledged |
| **PROTOTECH-51**: Limitations on NFT subset pool size | Low | Acknowledged |
| **PROTOTECH-50**: TokenIdsAllowed returns false for eligible Collection Pool NFT’s | Low | Fixed |
| **PROTOTECH-49**: .decimals() is an optional feature of ERC20 | Low | Fixed |
| **PROTOTECH-45**: Revert after instead of on expire_ | Low | Fixed |
| **PROTOTECH-44**: Unsafe Casts In RewardsManager | Low | Fixed |
| **PROTOTECH-43**: PositionManager.moveLiquidity() May Leave Internal State Inconsistent  | Low | Fixed |
| **PROTOTECH-38**: Loan Origination Fee Has Extreme Values At High Interest Rates | Low | Acknowledged |
| **PROTOTECH-20**: New Borrower Debt overcounted with < 18 decimal Quote Tokens  | Low | Fixed |
| **PROTOTECH-19**: removeCollateral can be used to bypass dust protection check in addCollateral | Low | Fixed |
| **PROTOTECH-17**: Non-18 Decimal Quote Token Debt cannot be FULLY repaid as expected  | Low | Fixed |
| **PROTOTECH-16**: Any Address Can Execute memorializePositions For Another Address | Low | Fixed |
| **PROTOTECH-14**: Correction to Meaningful Deposit for In-Auction Debt Neglects Inflator | Low | Fixed |
| **PROTOTECH-12**: Below-LUP Deposit Fee Has Extreme Values At High Interest Rates | Low | Fixed |
| **PROTOTECH-56**: Think through the various implications block.timestamp and network downtime | Informational | Acknowledged |
| **PROTOTECH-52**: Streamline NFT Collection Pool Creation | Informational | Fixed |
| **PROTOTECH-35**: Code Improvement: consider implementing isValidSignature directly | Informational | Fixed |
| **PROTOTECH-33**: isValidSignature doesn’t check for well formed signature | Informational | Fixed |
| **PROTOTECH-32**: Code clean up - RewardsManager Reentrancy Guard | Informational | Fixed |
| **PROTOTECH-31**: Code Redundancy: ERC721 is inherited multiple times in PositionManager | Informational | Fixed |
| **PROTOTECH-29**: Code Inconsistency: PermitERC721 uses require where elsewhere custom errors are adopted  | Informational | Fixed |
| **PROTOTECH-27**: ERC Compliance: PermitERC721 and PositionManager implementations doesn’t match with ERC-4494 | Informational | Fixed |
| **PROTOTECH-26**: Reserve Auction DOS Attack | Informational | Acknowledged |
| **PROTOTECH-25**: Unit Tests & non-18 decimals tokens | Informational | Acknowledged |
| **PROTOTECH-23**: Solidity Version: consider 0.8.16-0.8.18 for deployment | Informational | Fixed |
| **PROTOTECH-47**: Avoid param structs/tuples on external functions | Informational | Fixed |

#### Improvements:

| Finding | Category | Resolution |
| --- | --- | --- |
| **PROTOTECH-54**: Consider checking token id sort order in the pool | Gas Optimization | Acknowledged |
| **PROTOTECH-48**: Consider granularizing Info functions | Gas Optimization | Acknowledged |
| **PROTOTECH-36**: Consider removing redundant check in PermitERC721 | Gas Optimization | Fixed |
| **PROTOTECH-28**: Consider using block.chainid instead of inline assembly chainid | Gas Optimization | Fixed |
| **PROTOTECH-21**: bytes32ToString can be improved to reduce gas costs | Gas Optimization | Fixed |
| **PROTOTECH-34**: Informational Non-security Code Changes/Recommendations  | Code Quality | Fixed |
| **PROTOTECH-59**: Consider declaring RAY constant directly | Code Quality | Fixed |

## 5. Code4rena Audit (May 3, 2023 - May 11, 2023)
#### Report: [Code4rena Report](./code4rena/2023-05-ajna-draft-report.md)
#### Scope: [Core Position & Rewards Manager](https://github.com/ajna-finance/ajna-core) and [Grant Coordination Fund](https://github.com/ajna-finance/ajna-grants) Contracts
#### Findings: 11 High, 14 Medium, 54 Low

*Note*: Grant fund extraordinary mechanism and move staked liquidity functionalities were reevaluated and removed

| Finding | Severity | Resolution |
| --- | --- | --- |
| **H-01**: PositionManager's moveLiquidity can freeze funds by removing destination index even when the move was partial | High | Fixed |
| **H-02**: PositionManager's moveLiquidity can set wrong deposit time and permanently freeze LP funds moved | High | Fixed |
| **H-03**: Position NFT can be spammed with insignificant positions by anyone until rewards DoS | High | Fixed |
| **H-04**: Delegation rewards are not counted toward granting fund | High | Fixed |
| **H-05**: Incorrect calculation of the remaining updatedRewards leads to possible underflow error | High | Fixed |
| **H-06**: The lender could possibly lose unclaimed rewards in case a bucket goes bankrupt | High | Acknowledged |
| **H-07**: User can exponentially increase the value of their position through the memorializePositions function | High | Fixed |
| **H-08**: Claiming accumulated rewards while the contract is underfunded can lead to a loss of rewards | High | Fixed |
| **H-09**: User can avoid bankrupting by calling PositionManager.moveLiquidity where to index is bankrupted index | High | Fixed |
| **H-10**: missing isEpochClaimed validation | High | Fixed |
| **H-11**: RewardsManager fails to validate pool_ when updating exchange rates allowing rewards to be drained | High | Fixed |
| **M-01**: It is possible to steal the unallocated part of every delegation period budget | Medium | Fixed |
| **M-02**: Delegate rewards system is unfair to delegates with less tokens and reduces decentralization | Medium | Fixed |
| **M-03**: _updateBucketExchangeRateAndCalculateRewards reward calculation accuracy loss | Medium | Fixed |
| **M-04**: Potential unfair distribution of Rewards due to MEV in updateBucketExchangeRatesAndClaim | Medium | Acknowledged |
| **M-05**: Calculating new rewards is susceptible to precision loss due to division before multiplication | Medium | Fixed |
| **M-06**: An Optimizer Bug in PositionManager.getPositionIndexesFiltered | Medium | Fixed |
| **M-07**: StandardFunding.screeningVote function and ExtraordinaryFunding.voteExtraordinary function when block.number equals respective start block and when block.number is bigger than respective start block can result in different available votes for same voter | Medium | Fixed |
| **M-08**: The voting thresholds in Ajna's Extraordinary Funding Mechanism can be manipulated to execute proposals below the expected threshold | Medium | Fixed |
| **M-09**: Adversary can prevent the creation of any extraordinary funding proposal by frontrunning proposeExtraordinary() | Medium | Fixed |
| **M-10**: Unsafe casting from uint256 to uint128 in RewardsManager | Medium | Fixed |
| **M-11**: StandardFunding.fundingVote should not allow users who didn't vote in screening stage to vote | Medium | Fixed |
| **M-12**: Governance attack on Extraordinary Proposals | Medium | Fixed |
| **M-13**: PositionManager & PermitERC721 Failure to comply with the EIP-4494 | Medium | Fixed |
| **M-14**: PositionManager.moveLiquidity could revert due to underflow | Medium | Fixed |
