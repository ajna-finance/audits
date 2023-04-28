# Ajna Protocol security audits
## 1. Sherlock 1st contest (Jan 9, 2023 - Jan 30, 2023)
#### Contest: https://app.sherlock.xyz/audits/contests/32

#### Report: [Sherlock 1 Report](./sherlock/Contest1.md)
#### Scope: [Protocol](https://github.com/ajna-finance/ajna-core) and [Grant Coordination Fund](https://github.com/ajna-finance/ajna-grants) Contracts
#### Findings: 11 High, 22 Medium

| Finding | Resolution |
| --- | --- |
| **Issue H-1**: `RewardsManager` doesn't delete old bucket snapshot info on unstaking | Fixed |
| **Issue H-2**: Anyone who approved quote tokens to a pool can be forced to take | Fixed |
| **Issue H-3**: `CryptoPunks` NFTs may be stolen via deposit frontrunning | Fixed |
| **Issue H-4**: `scaledQuoteTokenAmount` isn't updated to be collateral sell value in the quote token constraint case of `_calculateTakeFlowsAndBondChange` | Fixed |
| **Issue H-5**: `removeCollateral` miss bankrupcy logic and can make future `LP`s sharing losses with the current ones | Fixed |
| **Issue H-6**: Executing funded standard proposals can be prevented by a proposal slate with duplicate proposals | Fixed |
| **Issue H-7**: `ERC721Pool`'s `mergeOrRemoveCollateral` allows to remove collateral while auction is clearable | Fixed |
| **Issue H-8**: Remaining collateral used by `ERC721Pool` is missed in `Auctions` `take` and `bucketTake` return structures | Fixed |
| **Issue H-9**: `moveQuoteToken()` can cause bucket to go bankrupt but it is not reflected in the accounting | Fixed |
| **Issue H-10**: `ERC721Pool`'s take will proceed with truncated collateral amount and full debt when borrower's collateral is fractional | Fixed |
| **Issue H-11**: The deposit / withdraw / trade transaction lack of expiration timestamp check and slippage control | Fixed |
| **Issue M-1**: Buypunk function of `Cryptopunks` in `ERC721Pool` is used incorrectly | Fixed |
| **Issue M-2**: `ERC721Pool` taker callback misreports quote funds whenever there was collateral amount rounding | Fixed |
| **Issue M-3**: Anyone can transfer approved `LP` tokens | Fixed |
| **Issue M-4**: Incorrect `MOMP` calculation in neutral price calculation | Fixed |
| **Issue M-5**: Extraordinary proposals can receive more tokens than eligible | Fixed |
| **Issue M-6**: Claiming rewards from a future not yet existing epoch prevents claiming rewards for those epochs later on | Fixed |
| **Issue M-7**: Calculating new rewards is susceptible to precision loss due to division before multiplication | Fixed |
| **Issue M-8**: Claiming accumulated rewards while the contract is underfunded can lead to a loss of rewards | Fixed |
| **Issue M-9**: Incorrect Validation in `Pool.sol#transferLPs` lead to a DOS attack | Fixed |
| **Issue M-10**: Adversary can grief kicker by frontrunning `kickAuction` call with a large amount of loan | Fixed |
| **Issue M-11**: Settled collateral of a borrower aren't available for lenders until borrower's debt is fully cleared | Fixed |
| **Issue M-12**: Deposits are eliminated before currently unclaimed reserves when there is no reserve auction | No fix, Docs updated |
| **Issue M-13**: Flashloan end result isn't controlled | Fixed |
| **Issue M-14**: Interest rates can be raised above the market as a griefing, disabling the pool | Fixed |
| **Issue M-15**: Interest rate for pool is bounded wrongly | Fixed |
| **Issue M-16**: Auction timers following liquidity can fall through the floor price causing pool insolvency | No fix, Docs updated |
| **Issue M-17**: If borrower or kicker got blacklisted by asset contract their collateral or bond funds can be permanently frozen with the pool | Fixed |
| **Issue M-18**: user can `drawDebt` that is below dust amount | Fixed |
| **Issue M-19**: Quadratic voting tally done wrong | Fixed |
| **Issue M-20**: `CryptoKitty` and `CryptoFighter` NFT can be paused, which block borrowing / repaying / liquidating action in the `ERC721Pool` when borrowers still forced to pay the compounding interest | Fixed |
| **Issue M-21**: Minting an NFT with a position on the same bucket as a previously minted NFT changes its deposit time | Fixed |
| **Issue M-22**: Memorializing an NFT position on the same bucket of a previously memorialized NFT locks redemption | Fixed |

## 2. Trail Of Bits Audit (Feb 13, 2023 - April 3, 2023)
#### Report: [Trail Of Bits Report](./tob/Ajna_Final_Report+Fix_Review-4.24.2023.pdf) 
#### Scope: [Protocol](https://github.com/ajna-finance/ajna-core) and [Grant Coordination Fund](https://github.com/ajna-finance/ajna-grants) Contracts
#### Findings: 1 High, 1 Medium, 3 Lows

| Finding | Severity | Resolution |
| --- | --- | --- |
| **TOB-AJNA-1**: Solidity compiler optimizations can be problematic | Undetermined | No fix (contracts exceed spurious dragon limit) |
| **TOB-AJNA-2**: `findIndexAndSumOfSums` ignores global scalar | Informational |  No fix, improved invariant testing |
| **TOB-AJNA-3**: Incorrect inflator arithmetic in view functions | Low |  Fixed |
| **TOB-AJNA-4**: Interest rates can become extreme, allowing DoS attacks | Medium |  Fixed |
| **TOB-AJNA-5**: Older versions of external libraries are used | Informational |  `OZ` updated to 4.8.2 in contracts repo. `PRBMaths` not updated (requires code change) |
| **TOB-AJNA-6**: Extraordinary proposal can be used to steal extraordinary amounts of Ajna | High |  Fixed |
| **TOB-AJNA-7**: `findMechanismOfProposal` function could shadow an extraordinary proposal | Low |  Fixed |
| **TOB-AJNA-8**: Array lengths are not checked in `LP` allowance update functions | Low |  Fixed |

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
