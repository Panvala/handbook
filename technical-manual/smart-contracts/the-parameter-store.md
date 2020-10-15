# The Parameter Store

The _parameter store_ holds key-value pairs that are subject to Panvala’s governance process. The parameter store gives us a common API for proposing, approving, and executing changes to parameters instead of needing different functions to be called to change different parameters. This pattern was inspired by the “[Parameterizer](https://github.com/skmgoldin/tcr/blob/master/contracts/Parameterizer.sol)” contract from the original token-curated registry implementation.

## Initial Parameters

| Parameter Name | Parameter Type | Description | Value |
| :--- | :--- | :--- | :--- |
| `gatekeeperAddress` | `address` | The address of the active gatekeeper contract used to govern Panvala. | Determined at deploy-time |
| `slateStakeAmount` | `uint` | The number of PAN that must be staked in order for a slate recommendation to be considered by pan holders. | 50,000 PAN |
| `archives` | `bytes32` | The git hash of the Panvala archives, which contain documents that have been endorsed through Panvala’s governance process. | TBD. The initial repository will be hosted at [https://github.com/Panvala/archives](https://github.com/Panvala/archives). |

