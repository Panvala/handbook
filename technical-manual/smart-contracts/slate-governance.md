# Slate Governance

Panvala makes decisions using _slate governance_. Each quarter, the system approves one slate of actions and all of the individual proposals it contains. PAN holders who don’t believe that a slate represents the consensus of the community can propose a competing slate of proposals. PAN must be staked on each proposed slate in order for that slate to appear on the ballot, and the tokens staked on losing slates are donated to the token capacitor. 

Each slate is associated with the _advisory group_ that authored the slate. While most on-chain decision-making systems involve approving or rejecting individual proposals, the main question we answer each quarter is “should we abandon our last decision-making process?” The advisory group of a slate always represents a particular decision-making process, even if it is poorly defined. When done well, advisory groups make their decision-making process clear, and their recommended slate represents the output of that process. Some example processes for advisory groups include voting off-chain, electing representative bodies, or relying on reputable authorities. Challenging individual proposals is done within the process that the advisory group has already defined. Challenging a slate is like amending a constitution: you change the rules when they no longer serve the community’s goals, but not because you’re unsatisfied with a particular outcome.

## Design Goals

Slate governance was designed to avoid common pitfalls we’ve seen in other decentralized systems. The systems that have been deployed so far often see large numbers of decisions made, but low voter turnout has been a signal that token-based voting might not actually be rewarding the effort it takes to evaluate decisions. Other systems see too few decisions: Bitcoin has been famously resistant to change over the years, for better or for worse. We see this inertia as an emergent outcome of Bitcoin’s fork-based governance rules. The principles that justify resistance to change have been post hoc rationalizations of an emergent phenomenon.

Fork-based governance has also made its way to on-chain systems. TheDAO was a fork-based organization in which token holders could fork off a new organization after any decision they didn’t want to cooperate with. \([TheDAO was hacked](https://www.bloomberg.com/features/2017-the-ether-thief/) before we could see whether its design would work.\) Moloch DAO follows in TheDAO’s footsteps with its “ragequit” functionality: during the waiting period after each approval, anyone can withdraw their remaining Ether if they don’t want to support the proposal. These designs make it easy to decide to join since there’s no real commitment, but their potential is constrained by the need to play it safe to keep people from leaving. Panvala avoids fork-based governance with the goal of building a committed community that cooperates even when they don’t get their way.

On-chain voting has a poor track record. We believe it should be used as a mechanism of last resort rather than in the typical operation of a system. Our approach is similar to Plasma, a blockchain scaling strategy that builds child chains that can be entered and exited via a root contract. When a Plasma child chain is operating normally, very few transactions are sent on chain. When something goes wrong, that could trigger thousands of transactions to the root contract on the blockchain so people can remove their assets. Similarly, when everything is operating normally in Panvala, very few governance transactions are sent. During times of discord or attacks, thousands of transactions can be triggered for votes to be tallied to resolve the problem.

## Resources and Permissions

Many designs for on-chain governance are oriented around approving transactions for a shared account to send, allowing token holders to collectively perform the same actions that a person can send a transaction to execute. Traditional multisignature wallets are the simplest form of this design, and Aragon DAOs are complex, token-based designs that control a single Aragon Agent that sends arbitrary transactions. Panvala avoids this design primarily to avoid becoming a shared pool of assets. Since Panvala cannot send transactions, it can’t hold any assets other than its own token.

Instead of sending arbitrary transactions, Panvala’s _resources_ allow anyone to request _permission_ to interact with them. A resource is any smart contract \(like the token capacitor\) that defines permissions, which are then fed into the _gatekeeper_ for approval. The gatekeeper contract is where token holders create slates of permission proposals, and vote on them if necessary. Resources call two functions on the gatekeeper:

```text
function requestPermission(bytes memory metadataHash) public returns(uint)
function hasPermission(uint requestID) public view returns(bool)
```

Resources store the permission identifier returned from `requestPermission` along with bookkeeping information about each permission request so they can check if the permission has been granted, ensure that one-time use permissions haven’t been used already, and execute the desired action. For instance, the token capacitor stores `Proposal` structures for each request to withdraw tokens:

```text
struct Proposal {
    address gatekeeper;
    uint256 requestID;
    uint tokens;
    address to;
    bytes metadataHash;
    bool withdrawn;
}
```

Keeping track of the gatekeeper instance that was used for each proposal is particularly important to ensure that upgrades of the governance contracts go smoothly \(see [_Upgradeability_](upgradeability-and-error-recovery.md)\).

## Slates

A slate contains zero or more permission requests for a single resource. Slates are authored by advisory groups, who can choose whether to stake PAN on their slate to add it to the contest. If the advisory group doesn’t stake on the slate, someone else must stake on it or the slate will be ignored.

Each resource has its own set of slates that compete to approve permissions each quarter. As a result, each resource has a separate contest occurring each quarter. Some resources might trigger a vote during the same quarter that other resources have no contest. For example, grant slates are likely to be recommended almost every quarter, but even when there are competing grant slates, there probably won’t be a contest between governance parameter slates at the same time.

If a resource only has one staked slate during a quarter, that slate automatically wins, and its permissions are approved.

Slates submitted without any permission requests to approve are called blank slates. Recommending a blank slate is appropriate when the consensus of the token holders is to take no action for the quarter, typically because no consensus could be reached during periods of discord.

### Incumbency

Advisory groups represent an off-chain process for reaching consensus about which permissions to approve. Panvala highlights the identity of slates’ advisory groups to focus the on-chain decision away from the merits of the permissions on a slate and towards the process by which those permissions were added to the slate. The advisory group of the last successful slate for a resource is the _incumbent_ for that resource. The incumbent is effectively the embodiment of the bylaws that are currently in effect.

Creating a slate to challenge the incumbent is a serious action. If the incumbent’s slate loses a contest, the token holders have decided to _abandon the incumbent_: it’s not just their proposals that were rejected, it’s the implicit bylaws they represent that were rejected. Hopefully they were replaced by better bylaws.

If no one submitted a slate for a quarter, the last incumbent persists. Incumbency can never be vacated.

## Voting

Panvala uses a commit-reveal process to tally votes. During the commit phase, voters submit a hash of their vote, while keeping the vote itself and a random salt secret. During the reveal phase, no new commitments can be made, and earlier commitments can be revealed. This approximates the experience of typical elections where no running tally of votes is available until the polls close.

Each token can be used to acquire one vote. To acquire voting rights, your tokens must be deposited in the gatekeeper contract before or during the commit phase, and cannot be withdrawn until the commit phase is over. This prevents the same tokens from being used to acquire multiple votes. 

Voters can delegate their votes to another Ethereum account. This allows voters to store the keys that control their tokens safely while delegating to a frequently-used key that cannot withdraw the tokens.

If there is an active contest but no one votes, the default action is to reject all slates for that resource.

### Ranked Choices and Runoffs

When a contest has two competing slates, the slate with more votes wins. When a contest has three or more competing slates, voters can indicate their first and second choices for slates. If any slate gets more than half of the first choice votes, that slate wins. Otherwise, all slates but the top two recipients of first choice votes are eliminated. Any second choice votes for the top two slates are counted for voters whose first choice was eliminated. The remaining slate with the most votes wins.

{% hint style="info" %}
Here's an example runoff:
{% endhint %}

| Candidate | Round 1 | Round 2 |
| :--- | :--- | :--- |
| Slate A | 45 million | 52 million |
| Slate B | ~~20 million~~ | ~~~~ |
| Slate C | 35 million | 48 million |

## Epochs

Each period of governance is called an _epoch_, and lasts thirteen weeks. Epoch zero started on November 2, 2018 at 1700 UTC and ended with the issuance of Batch One of grants on February 1, 2019.

| Epoch Number | End of Epoch | Time \(UTC\) | Time \(Austin, TX\) |
| :--- | :--- | :--- | :--- |
| 0 | 2019-02-01 | 1700 | 11 am CST |
| 1 | 2019-05-03 | 1700 | 12 pm CDT |
| 2 | 2019-08-02 | 1700 | 12 pm CDT |
| 3 | 2019-11-01 | 1700 | 12 pm CDT |
| 4 | 2020-01-31 | 1700 | 11 am CST |
| 5 | 2020-05-01 | 1700 | 12 pm CDT |
| 6 | 2020-07-31 | 1700 | 12 pm CDT |
| 7 | 2020-10-30 | 1700 | 12 pm CDT |
| 8 | 2021-01-29 | 1700 | 11 am CST |

The first week of an epoch is the _quiet period_. During the quiet period, no governance actions can be proposed while permissions granted during the previous epoch are executed.

The first deadline within an epoch is the slate submission deadline. After this deadline, no more slates can be staked or recommended for the given resource. The hard deadline for slate submission is eleven weeks into an epoch, but to prevent slates from being snuck in at the last minute, a soft deadline starts 5.5 weeks into the epoch, and adjusts as slates are submitted. Each time a slate is staked for a resource, the soft deadline is reset to be halfway between the current time and the hard deadline, which makes the soft deadline approach the hard deadline with each slate submission. As a result, each resource can have a different soft deadline for slate submission.

At the end of week 11, the voting commit period begins and lasts for one week. At the end of week 12, the voting reveal period begins and lasts for one week. Once the epoch has ended, no more votes can be revealed, so the contests can be finalized and permissions approved for the winning slates. Each permission request expires at the end of the following epoch, which prevents approved grants from persisting over multiple epochs when the recipient fails to withdraw them.  


