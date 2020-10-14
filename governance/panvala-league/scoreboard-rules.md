# Scoreboard Rules

Each quarter, the Panvala community allocates the tokens released from Panvala's inflation. We don’t do it by agreeing on everything or by letting a majority decide. Instead, we use the Panvala League to distribute power to you and your community.

The Panvala League’s funding rules are based on two principles:

1. Communities should receive more funding when they bring in more donations.
2. Communities should be rewarded as their share of staked tokens reaches their share of the donations.

The first principle **keeps Panvala sustainable**: we must build up a strong flow of donations while our inflation subsidies are high, because in a decade or two, the inflation subsidies will be lower as we approach Panvala’s maximum supply of 100 million PAN.

The second principle **allows Panvala to grow fairly**: communities can succeed in the League before they own much of Panvala’s token supply because ownership is only rewarded in proportion to donations—just like a cooperative only rewards its owners in proportion to their patronage. Since the Panvala League isn’t just about how much PAN you own, it’s just as attractive to join for early communities as it will be for Panvala’s 1000th community. We want Panvala to be easy to share with the world.

We want your community to join the Panvala League! Whether you’re a DeFi community with your own token [like DXdao](https://forum.panvala.com/t/defi-for-good-the-dxdao-approach/216), or a social community like Meta Gamma Delta, we want to share Panvala’s sustainable treasury with you. Head to [Join Panvala](../../join-panvala.md) to get started.

## Donations

Donations to each grant for the Panvala League’s communities are scored using _quadratic funding_. In quadratic funding, communities are rewarded not just for the size of their donations, but for the number of people who donated. Several small donations can count more than one huge donation. In Panvala, we’re trying to build up communities full of donors, not communities with a few wealthy benefactors. \(For more information on quadratic funding \(QF\), check out Gitcoin’s [WTFisQF.com](https://wtfisqf.com/).\)

**When you donate with PAN, you’re participating in Panvala’s governance.** We use each community’s share of donations \(calculated using quadratic funding\) to allocate each community’s share of funding from Panvala’s inflation. But there’s one other factor used to calculate each community’s funding: their share of the staked PAN.

## Staking

Before each donation matching round starts, the Panvala League’s existing and new communities have time to stake PAN tokens to earn donation matching capacity. When the round ends, communities will want to know if their share of staked tokens reached their share of donations. If it hasn’t, they’ve exceeded their **capacity**, and can stake more PAN to earn more of the quarter’s budget. For more on staking, head to [Staking PAN](../../the-pan-token/staking-pan.md).

## Full Scoreboard \(Advanced\)

The donation and staking summaries above should be enough to give a rough understanding of how the Panvala League’s funding scoreboards work, but if you’re interested in diving deeper, here’s a column by column breakdown of [the full scoreboard](https://docs.google.com/spreadsheets/d/1epswepiJTZF7NMtOfEAMcIviw5rXVYPm99hGUDhcnQs/edit#gid=2037752121) that we use to calculate each community’s funding.

### Staked Tokens

The total number of PAN tokens that have been staked for each community. Anyone can stake PAN for their community. Some communities own PAN together in a DAO or a multisig wallet, while other communities are supported purely by individuals staking PAN for them. The full list of stakers can be found in the Stakers tab of the spreadsheet.

### Capacity

A community’s capacity is their share of the total staked tokens.

### PAN Donated

The total PAN donated to each community’s grant. This includes PAN transfers on the Ethereum mainnet, as well as PAN transfers on zkSync. This value is not directly used in the funding calculation.

### Donation Count

The number of donations made to a community’s grant. This value is not directly used in the funding calculation.

### Quadratic Funding

The amount of funding the community would receive based on the size of each individual donation they received according to the [quadratic funding formula](https://wtfisqf.com/).

### Share of Quadratic Funding

The community’s quadratic funding score divided by the total quadratic funding scores.

### Utilization

The community’s share of quadratic funding divided by their capacity. Utilization is what we use to reward communities whose share of the staked tokens is at least their share of the donations.

### Capacity Overflow

The amount of utilization above 100%.

### Utilization After Overflow

The utilization we credit a community for after applying diminishing returns to their overflow. That is, donations within the first 100% of utilization get full credit, but donations above 100% of utilization get less and less credit. These linear diminishing returns are applied by calculating their quadratic integral. In the DXdao example above, applying diminishing returns to the 1162% of overflow results in just 392% of overflow credit, for a total of 492% utilization after adding back in the 100% portion that was within capacity.

### Subsidy Points

The community’s quadratic funding score, but decayed by the same proportion as their utilization decayed after overflow. Communities without any overflow have the same subsidy points as their quadratic funding score, but communities with overflow have fewer subsidy points than their quadratic funding score.

### Share of Subsidy

The community’s share of the total subsidy points.

### Subsidy

The PAN that the community will receive from Panvala’s inflation. This does not include the donations they’ve already received directly.

### Funding \(PAN\)

The PAN that the community will receive in total, including both the portion from Panvala’s inflation and the donations that were received directly.

Note that the total of this column will always equal the budget specified by the Panvala Caucus in their latest budget recommendation \(1,369,935.62 PAN for this quarter\). Donations do not affect this value because we’re using the donations as a signal to distribute a fixed budget, not as a way to increase the total amount of PAN that our communities will receive. In Panvala, all donations must flow back into Panvala’s token supply. Since donations are made directly to communities, we use a portion of the quarter’s inflation to transfer the same amount that was donated back into the token supply, which keeps the total amount of PAN that we distribute constant regardless of the amount donated.

### Funding \(USD\)

The estimated USD value of the community’s funding in PAN. Note that the USD value of PAN is very volatile: it can increase or decrease significantly.

### Multiplier

The community’s funding in PAN divided by the total donations they collected in PAN. Communities with more donors and more staked PAN will tend to have higher matching multipliers when they raise similar total amounts of PAN.

