
# Notional Finance Convex Leveraged Vault contest details

- Join [Sherlock Discord](https://discord.gg/MABEWyASkp)
- Submit findings using the issue page in your private contest repo (label issues as med or high)
- [Read for more details](https://docs.sherlock.xyz/audits/watsons)

# Resources

- [Leveraged Vault Documentation](https://docs.notional.finance/developer-documentation/how-to/leveraged-vaults)
- [Leveraged Vault Controller Code]()

# On-chain context

DEPLOYMENT: mainnet
ERC20: CVX, CRV, Curve LP Tokens, USDC, DAI, USDT, ETH, ETH Staking Derivatives (stETH, rETH, etc), and potentially any other assets supported by Curve and/or Convex.
ERC721: none
ERC777: none
FEE-ON-TRANSFER: none, Leveraged Vaults are explicitly prevented from using fee on transfer tokens as borrow assets.
REBASING TOKENS: stETH
ADMIN: restricted, admins are expected to be able to settle, reinvest rewards and upgrade the smart contracts to mitigate security issues. Settlement and reinvestments should be done with slippage limits, however, it is not always feasible to calculate these on chain due to a lack of reliable oracles.
EXTERNAL-ADMINS: restricted (Convex, Curve)

Please answer the following questions to provide more context: 
### Q: Are there any additional protocol roles? If yes, please explain in detail:

A: 
These four roles are defined for each vault.
```
    // Allowed to settle vaults during the settlement window prior to maturity. Can sell assets to repay vault debts
    // under certain slippage conditions and without significantly over-repaying debts.
    bytes32 internal constant NORMAL_SETTLEMENT_ROLE = keccak256("NORMAL_SETTLEMENT_ROLE");

    // Allowed to settle vaults during emergency conditions, when vault assets exceed some threshold of total assets
    // held in the target liquidity pool. Similar to normal settlement but allowed to happen outside of the settlement
    // window if the condition is present. The role is expected not to actively manipulate the pool to generate an emergency
    // settlement condition.
    bytes32 internal constant EMERGENCY_SETTLEMENT_ROLE = keccak256("EMERGENCY_SETTLEMENT_ROLE");

    // A backstop for settlement if for some reason settlement does not occur prior to maturity. Held as a separate role since
    // there must be some manual override during post maturity.
    bytes32 internal constant POST_MATURITY_SETTLEMENT_ROLE = keccak256("POST_MATURITY_SETTLEMENT_ROLE");

    // This role is allowed to claim reward tokens and reinvest them into vault assets
    bytes32 internal constant REWARD_REINVESTMENT_ROLE = keccak256("REWARD_REINVESTMENT_ROLE");
```

Additionally, the Notional Owner can perform various vault actions such as listing, pausing, and setting various governance parameters
for the vault within the Notional Leveraged Vault Controller framework. (see resources above)

___
### Q: Is the code/contract expected to comply with any EIPs? Are there specific assumptions around adhering to those EIPs that Watsons should be aware of?
A:

No.

___

### Q: Please list any known issues/acceptable risks that should not result in a valid finding.
A: 

We have specific oracle checks for oracle freshness in the Trading Module, we do not rely on the chainlink updatedInRound logic.
We do not use .call() to transfer ETH, we use .transfer(). This is explicitly designed to prevent re-entrancy issues including read only re-entrancy.

____
### Q: Please provide links to previous audits (if any).
A:

- https://app.sherlock.xyz/audits/contests/31
- https://app.sherlock.xyz/audits/contests/2
___

### Q: Are there any off-chain mechanisms or off-chain procedures for the protocol (keeper bots, input validation expectations, etc)? 
A: 

Reinvestment and Settlement bots should be assumed to be working properly and setting proper slippage thresholds.
_____

### Q: In case of external protocol integrations, are the risks of an external protocol pausing or executing an emergency withdrawal acceptable? If not, Watsons will submit issues related to these situations that can harm your protocol's functionality. 
A: Not Acceptable for Curve or Convex to withdraw user funds such that they are not redeemable.


# Audit scope

[leveraged-vaults @ ec790f931988904f99da5c3514e8e1c74bad050b](https://github.com/notional-finance/leveraged-vaults/tree/ec790f931988904f99da5c3514e8e1c74bad050b)
- [leveraged-vaults/contracts/trading/adapters/CurveV2Adapter.sol](leveraged-vaults/contracts/trading/adapters/CurveV2Adapter.sol)
- [leveraged-vaults/contracts/vaults/Curve2TokenConvexVault.sol](leveraged-vaults/contracts/vaults/Curve2TokenConvexVault.sol)
- [leveraged-vaults/contracts/vaults/curve/CurveVaultTypes.sol](leveraged-vaults/contracts/vaults/curve/CurveVaultTypes.sol)
- [leveraged-vaults/contracts/vaults/curve/external/Curve2TokenConvexHelper.sol](leveraged-vaults/contracts/vaults/curve/external/Curve2TokenConvexHelper.sol)
- [leveraged-vaults/contracts/vaults/curve/internal/CurveConstants.sol](leveraged-vaults/contracts/vaults/curve/internal/CurveConstants.sol)
- [leveraged-vaults/contracts/vaults/curve/internal/CurveVaultStorage.sol](leveraged-vaults/contracts/vaults/curve/internal/CurveVaultStorage.sol)
- [leveraged-vaults/contracts/vaults/curve/internal/pool/Curve2TokenPoolUtils.sol](leveraged-vaults/contracts/vaults/curve/internal/pool/Curve2TokenPoolUtils.sol)
- [leveraged-vaults/contracts/vaults/curve/mixins/ConvexStakingMixin.sol](leveraged-vaults/contracts/vaults/curve/mixins/ConvexStakingMixin.sol)
- [leveraged-vaults/contracts/vaults/curve/mixins/Curve2TokenPoolMixin.sol](leveraged-vaults/contracts/vaults/curve/mixins/Curve2TokenPoolMixin.sol)
- [leveraged-vaults/contracts/vaults/curve/mixins/Curve2TokenVaultMixin.sol](leveraged-vaults/contracts/vaults/curve/mixins/Curve2TokenVaultMixin.sol)
- [leveraged-vaults/contracts/vaults/curve/mixins/CurvePoolMixin.sol](leveraged-vaults/contracts/vaults/curve/mixins/CurvePoolMixin.sol)
- [leveraged-vaults/contracts/trading/adapters/CurveAdapter.sol](leveraged-vaults/contracts/trading/adapters/CurveAdapter.sol)

# About Notional Finance

Notional Finance is a lending protocol that provides fixed rate lending and borrowing as well as leveraged yield products. 

Leveraged Vaults are available via our beta UI: https://beta.notional.finance/

# Running Unit Tests

NOTE: if you get an import error for LeveragedVaultsProject you need to rename the folder that you pulled the repository into to `leveraged-vaults` so that brownie can properly resolve the import.

### Install brownie
```
python3 -m pip install --user pipx
python3 -m pipx ensurepath
pipx install eth-brownie
```
https://eth-brownie.readthedocs.io/en/stable/install.html

### Install hardhat
```
yarn install
```

### Add mainnet-fork
* Add the following YAML block to ~/.brownie/network-config.yaml under development
```
- name: Hardhat (Mainnet Fork)
  id: mainnet-fork
  cmd: "npx.cmd hardhat node"
  host: http://127.0.0.1
  timeout: 120
  cmd_settings:
    port: 8545
    fork: mainnet
```
https://eth-brownie.readthedocs.io/en/stable/network-management.html#

### Execute tests
* Balancer tests
```
brownie test tests/dex_lp/balancer --network mainnet-fork
```
* Curve tests
```
brownie test tests/dex_lp/curve --network mainnet-fork
```