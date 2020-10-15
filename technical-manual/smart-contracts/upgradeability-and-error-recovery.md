# Upgradeability and Error Recovery

## Upgradeability

While the token capacitor and parameter store contracts cannot be changed, the gatekeeper contract is designed to be upgraded over time. Rather than the upgradeable contracts pattern popularized by OpenZeppelin and Aragon where a contract maintains its address and storage while the code changes, we use an older “EternalDB” pattern popularized by [Peter Borah](https://github.com/ConsenSys/dapp-store-contracts/blob/master/contracts/EternalDB.sol) and the [Colony](https://blog.colony.io/writing-upgradeable-contracts-in-solidity-6743f0eecc88/) team. In the latter pattern, state is stored separately from code, and the address of the code that controls access to the state changes with each upgrade.

Our “EternalDB” is the parameter store. Rather than having a modifiable owner that pushes authorized changes into the contract, the parameter store pulls changes from the gatekeeper contract, which is specified by a parameter as well. As long as the new gatekeeper contract follows the permissions API from the original contract, the new version can implement whatever decision-making logic that is needed.

Updating the gatekeeper points all relevant contracts away from the old gatekeeper and towards the new one. Since the epoch in which the gatekeeper is changed can contain many other decisions as well, the upgrade approach taken by the community has significant potential effects. Permissions will function as expected during the transition as long as resources store the address of the active gatekeeper alongside each permission to ensure that permission lookups aren’t misdirected by gatekeeper upgrades.

Token balances and delegations can be transferred by individual voters, but care must be taken to inform voters of the transition with enough time to prepare. Incumbents are harder to transfer: since the new gatekeeper must be deployed before the permission has been granted to upgrade, the new gatekeeper won’t know the incumbents from that epoch unless it was written to be able to fetch them.

In this context, gatekeeper upgrades have three options when it comes to transitioning state: they can do a _clean break_ that loses incumbent data, they can _fetch_ the incumbent data and anything else they’d like to transition in an initialization function on the new gatekeeper, or they can _preserve all state_ using Merkle proofs or a new gatekeeper contract that uses an upgradability pattern that updates the code within a contract. If no other known contracts are using the incumbent data, clean break upgrades should be fine, but third-party contracts may have begun relying on the data without notifying you. Fetching the desired state to transition avoids breaking known or unknown contracts that rely on incumbent data.

## Error Recovery

Smart contracts are rigid programs that are difficult and risky to modify after they have been deployed. Many contracts in other applications have had bugs and security flaws. While we’ve taken the best practice precautions to avoid these issues, it’s still possible that problems will arise that we haven’t foreseen.

In the event of a serious error, the incumbent advisory group for the token capacitor resource should make a recommendation for recovering from the error. In some cases, it might be easy to deploy fixed versions of the contracts with a new token that copies the balances at the time the error occurred. More complicated errors might be more difficult to recover from.

Since Panvala only holds its own token, all errors can be recovered with a new instance of the system. Determining the initial state of that new instance is the hard part, and the incumbent advisory group for the token capacitor should lead the community towards a consensus.  


