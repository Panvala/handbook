# The Token Capacitor

The _token capacitor_ is the smart contract that releases tokens for grants and accepts tokens as donations. The tokens in the capacitor are released at a rate that decays exponentially over time. Panvala’s token capacitor is configured with a half-life of 1456 days \(four 52-week years\), like Bitcoin’s block reward decay. This half-life is informed by the practices of other digital currencies, as well as common practices for issuing shares of corporations. However, it’s still just a guess. We’ve hardcoded this value not because it’s definitely the right choice forever, but because we believe that making it easy to alter the release curve would deter participation.

Withdrawing tokens from the token capacitor requires permission to be granted through the [slate governance](slate-governance.md) process. That process has its own timeline for granting permissions, but the token capacitor itself does not enforce restrictions on the timing of withdrawals. It only restricts the amount of tokens that can be withdrawn based on the balance after the last withdrawal or donation, the time of that change, and the amount of time that has elapsed since then.

## Exponential Decay

The token capacitor releases tokens at rates such that its balance decays exponentially. Ideally, this decay would follow the formula for exponential decay:

$$
N(t) = N_{0}(\frac{1}{2})^{\frac{t}{t_{\frac{1}{2}}}}
$$

$$
\text{where}\\
N(t)\text{ is the new balance,}\\
N_{0}\text{ is the previous balance,}\\
t\text{ is the amount of time that has elapsed since tokens were last released, and}\\
t_{\frac{1}{2}}\text{ is the half life of the token capacitor, 1456 days.}
$$

However, since the floating point operations needed to implement this formula have determinism issues, it’s a poor fit for execution on a blockchain, where thousands of nodes need to agree on the result. The Ethereum Virtual Machine does not include floating point instructions for this reason. This leaves us two attractive approaches for implementing exponential decay: store a lookup table for pre-calculated values of the decay factor for selected values of t, or create a schedule of release rates to approximate exponential decay with a piecewise function.

It is easier to verify that a particular implementation of a piecewise schedule is free of any flaws that could throw off the supply policy of the system. Piecewise functions are deterministic, while attempting to approximate the curve more closely leads to behavior that depends on the prior sequence of balances and multipliers used from the lookup table.  In addition, since the goal of these smart contracts is to build consensus within a large community, it’s useful to be able to communicate exactly how many tokens should be released when using math that the public can do in their heads. Bitcoin’s block reward schedule also approximates exponential decay in this manner.

However, Panvala’s token capacitor releases are based on the current balance, not the current time like Bitcoin. Bitcoin can read from the clock to determine how many halvings have occurred, but Panvala would have to store or calculate the balance boundaries for each release rate. With donations, the balance can fluctuate unpredictably, and any piecewise schedule implementation would have to account for releases that cross boundaries of the schedule. Together, these concerns increase the complexity of the implementation to a degree that accepting the flaws of the lookup table approach is the right tradeoff to make.

### Creating the Lookup Table

To create the lookup table, we must first select the smallest time interval that the table will support. The smaller the interval, the larger the error from truncation that compounds with every iteration. To use these multipliers with integers, we must choose a precision level to multiply by before using the multiplier, then divide out the precision factor when we’re done. We’ve chosen one day as the smallest interval and 1 x 10^12 as our precision factor. Together, they produce an error of about 531 tokens out of 50,000,000 over one half life.

We fill the rest of the lookup table with powers of two to be able to maintain more accuracy when more time has elapsed between the capacitor’s balance changes. However, we expect to achieve a flow of donations that exceeds one per day, which would cause the multiplier for one day to be used far more often than any other.

Each time we multiply by a multiplier, any present error compounds. As a result, using multipliers for fewer elapsed days over and over releases slightly more tokens than performing fewer multiplications using multipliers for more elapsed days.  


| **Days Elapsed** | **Multiplier** | **Integer Multiplier** | **Balance at Half-Life** |
| :--- | :--- | :--- | :--- |
| 1 | 0.9995240507 | 999524050675 | 49,999,469 |
| 2 | 0.9990483279 | 999048327879 | 49,999,733 |
| 4 | 0.9980975614 | 998097561438 | 49,999,872 |
| 8 | 0.9961987421 | 996198742149 | 49,999,937 |
| 16 | 0.9924119339 | 992411933860 | 49,999,968 |
| 32 | 0.9848814465 | 984881446469 | 49,999,981 |
| 64 | 0.9699914636 | 969991463599 | 49,999,990 |
| 128 | 0.9408834395 | 940883439455 | 49,999,995 |
| 256 | 0.8852616466 | 885261646641 | 49,999,997 |
| 512 | 0.783688183 | 783688183013 | 49,999,999 |
| 1024 | 0.6141671682 | 614167168195 | 49,999,999 |
| 2048 | 0.3772013105 | 377201310488 | N/A |

## Locked and Unlocked Tokens

The total balance of the capacitor is divided between _locked_ tokens and _unlocked_ tokens. Unlocked tokens are the only ones that can be withdrawn, and locked tokens are the only tokens involved in decay calculations. Each time tokens are received or sent by the capacitor, we move tokens from the locked balance to the unlocked balance _before_ adjusting to any request to deposit or withdraw tokens. If the unlocked balance were updated _after_ a deposit, that new donation would be included in the balance of tokens to be decayed as if the tokens had been there since the last balance update. If the unlocked balance were updated after a withdrawal, withdrawal might be incorrectly rejected if the tokens to withdraw weren’t unlocked yet.

A standalone function is available to sweep the appropriate number of locked tokens to the unlocked balance by the following method:

1. Calculate the elapsed days since tokens were last unlocked.
2. If the number of elapsed days is odd, multiply the locked balance by the lowest multiplier. Divide by the precision factor to determine the number of tokens that remain in the locked balance.
3. Add the difference between the previous and new total of locked tokens to the balance of unlocked tokens in the contract’s storage.
4. Divide the elapsed days by two, shift to the next multiplier, and repeat steps 2-5 until no time is remaining. This will take log2\(t\) iterations, and will release tokens for up to 4095 days in one transaction.
5. Store the difference between the total token balance of the contract and the balance of unlocked tokens as the new locked balance.
6. Add the elapsed time to the last unlocked time.

To illustrate the process more precisely as code, here is an excerpt from `TokenCapacitor.sol`:

```text
    function calculateDecay(uint256 _days) public view returns(uint256) {
        require(_days <= (2 ** decayMultipliers.length) - 1, "Time interval too large");

        uint256 decay = scale;
        uint256 d = _days;

        for (uint256 i = 0; i < decayMultipliers.length; i++) {
           uint256 remainder = d % 2;
           uint256 quotient = d >> 1;

           if (remainder == 1) {
                uint256 multiplier = decayMultipliers[i];
                decay = decay.mul(multiplier).div(scale);
           } else if (quotient == 0) {
               // Exit early if both quotient and remainder are zero
               break;
           }

           d = quotient;
        }

        return decay;
    }
```

Anyone can send a transaction that unlocks tokens and advances the last unlocked time by a given number of days that is less than or equal to the total elapsed time since tokens were last unlocked. Multiple transactions will be needed to bring the contract up to date if 4096 days or more have elapsed since tokens were last unlocked. Before processing a donation or a withdrawal, this function is called within the same transaction to minimize the number of transactions needed to keep the capacitor state up to date.

When permission has been granted to withdraw tokens, the number of unlocked tokens is reduced by the withdrawal amount. Withdrawals greater than the number of unlocked tokens revert the transaction. Donations are added directly to the locked token balance.

## Token Release Schedule

For reference, an approximate schedule of the first eight years of token capacitor releases is included below. This schedule assumes that the calculations to release tokens from the capacitor are done once per quarter, when they will likely be done more often, leading to slight variances in the numbers of tokens released. Donations to the token capacitor are not considered in this time-based chart. As a reminder, the token capacitor actually releases based on its _current balance_, not based on the _current time_ as in Bitcoin.

This chart can be used to consider donations by thinking of each donation as rewinding the token capacitor back in time by the equivalent number of tokens. Without donations, the balance of the capacitor moves down the curve at a predictable rate. Each donation restores the balance to an earlier point on the curve, so while the dates on this chart won’t match the balances in real life, you can look up the current balance on this chart to understand the current status of the system.  
****

|  | **Epoch** | **Locked Tokens** | **New Unlocked Tokens** | **Allocated supply** | **Quarterly Inflation** | **Annualized Inflation** |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| 2/1/2019 | 1 | 47,880,164 | 2,119,836 | 52,119,836 | 4.24% | 18.07% |
| 5/3/2019 | 2 | 45,850,202 | 2,029,962 | 54,149,798 | 3.89% | 16.51% |
| 8/2/2019 | 3 | 43,906,303 | 1,943,899 | 56,093,697 | 3.59% | 15.15% |
| 11/1/2019 | 4 | 42,044,819 | 1,861,484 | 57,955,181 | 3.32% | 13.95% |
| 1/31/2020 | 5 | 40,262,256 | 1,782,563 | 59,737,744 | 3.08% | 12.88% |
| 5/1/2020 | 6 | 38,555,268 | 1,706,988 | 61,444,732 | 2.86% | 11.93% |
| 7/31/2020 | 7 | 36,920,651 | 1,634,617 | 63,079,349 | 2.66% | 11.07% |
| 10/30/2020 | 8 | 35,355,336 | 1,565,315 | 64,644,664 | 2.48% | 10.30% |
| 1/29/2021 | 9 | 33,856,385 | 1,498,951 | 66,143,615 | 2.32% | 9.60% |
| 4/30/2021 | 10 | 32,420,985 | 1,435,400 | 67,579,015 | 2.17% | 8.97% |
| 7/30/2021 | 11 | 31,046,441 | 1,374,544 | 68,953,559 | 2.03% | 8.39% |
| 10/29/2021 | 12 | 29,730,173 | 1,316,268 | 70,269,827 | 1.91% | 7.86% |
| 1/28/2022 | 13 | 28,469,711 | 1,260,462 | 71,530,289 | 1.79% | 7.37% |
| 4/29/2022 | 14 | 27,262,688 | 1,207,023 | 72,737,312 | 1.69% | 6.92% |
| 7/29/2022 | 15 | 26,106,839 | 1,155,849 | 73,893,161 | 1.59% | 6.51% |
| 10/28/2022 | 16 | 24,999,994 | 1,106,845 | 75,000,006 | 1.50% | 6.13% |
| 1/27/2023 | 17 | 23,940,076 | 1,059,918 | 76,059,924 | 1.41% | 5.77% |
| 4/28/2023 | 18 | 22,925,095 | 1,014,981 | 77,074,905 | 1.33% | 5.45% |
| 7/28/2023 | 19 | 21,953,146 | 971,949 | 78,046,854 | 1.26% | 5.14% |
| 10/27/2023 | 20 | 21,022,404 | 930,742 | 78,977,596 | 1.19% | 4.86% |
| 1/26/2024 | 21 | 20,131,123 | 891,281 | 79,868,877 | 1.13% | 4.59% |
| 4/26/2024 | 22 | 19,277,629 | 853,494 | 80,722,371 | 1.07% | 4.34% |
| 7/26/2024 | 23 | 18,460,320 | 817,309 | 81,539,680 | 1.01% | 4.11% |
| 10/25/2024 | 24 | 17,677,662 | 782,658 | 82,322,338 | 0.96% | 3.90% |
| 1/24/2025 | 25 | 16,928,187 | 749,475 | 83,071,813 | 0.91% | 3.69% |
| 4/25/2025 | 26 | 16,210,487 | 717,700 | 83,789,513 | 0.86% | 3.50% |
| 7/25/2025 | 27 | 15,523,215 | 687,272 | 84,476,785 | 0.82% | 3.32% |
| 10/24/2025 | 28 | 14,865,081 | 658,134 | 85,134,919 | 0.78% | 3.15% |
| 1/23/2026 | 29 | 14,234,850 | 630,231 | 85,765,150 | 0.74% | 2.99% |
| 4/24/2026 | 30 | 13,631,339 | 603,511 | 86,368,661 | 0.70% | 2.84% |
| 7/24/2026 | 31 | 13,053,414 | 577,925 | 86,946,586 | 0.67% | 2.70% |
| 10/23/2026 | 32 | 12,499,992 | 553,422 | 87,500,008 | 0.64% | 2.57% |

## Recording Donations

Donations are recorded along with metadata to let the public know the context of the donation. In particular, it’s valuable for the public to know the donor’s intent and their view of the market when the donation was made. The current price of ETH/USD, current price of PAN/ETH, the intended donation in USD, and the terms of the pledge the donor intends to fulfill are useful context to record as metadata for donations.

## Donation Strategies

The token capacitor coordinates donations in a new way that is difficult to reason about since no one has done it before. We’ve thought through a hypothesis of the dynamics that we expect to play out.

Large, one-time donations have no effect on the long-term flow of donations, so they do not increase the system’s ability to fund work over the long run. It’s not much different from directly funding work as an individual, but Panvala’s goal is to build up a flow of funding that teams can count on. On the other hand, long-term commitments to recurring donations can change the flow of donations for an extended period of time, which allows more work to be rewarded using fewer tokens. Teams that expect Panvala to receive a steady flow of donations can stabilize their expectations of what the tokens can fund, which allows them to plan ahead to do work for the Ethereum ecosystem rather than finding private companies to hire them.

Donors who are long-term token holders are faced with a choice: should they buy new tokens to donate, or should they donate from the tokens they already hold? While both options are valid donations, donating by reducing your long-term holdings of Panvala tokens has counterintuitive effects. The purchasing power of the next batch of grants is dependent on the flow of tokens acquired, not the balance of tokens in the token capacitor. Donations that add tokens to the token capacitor but don’t involve newly acquired tokens have no effect on the purchasing power of the system: the tokens released each quarter just allocate the value flowing into the system from workers and token buyers. The increase in tokens granted will be accompanied by a decrease in the token price if the flow of value to acquire tokens stays the same.

To make the largest impact with your donations, view them as coming from your income rather than your holdings. Donate tokens immediately after acquiring them, whether you earned them directly for work, or purchased them from someone else. To make an impact with tokens you held onto, hold them indefinitely and use their voting power to steer the system. Ideally, make an impact in both ways: holding tokens to vote while donating regularly from your income has the largest positive impact on Panvala’s capacity to fund the work that you enjoy donating to.  


