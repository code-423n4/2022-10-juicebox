# Juicebox contest details
- $47,500 USDC main award pot
- $2,500 USDC gas optimization award pot
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-10-juicebox-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts October 18, 2022 20:00 UTC
- Ends October 23, 2022 20:00 UTC
- The [Juicebox contracts](https://github.com/jbx-protocol/juice-nft-rewards/tree/4ac8cb18f5873a4f59341719450be6a91c4fa8e1) to be audited

In scope:
contracts/*
!contracts/forge-test/**
!contracts/scripts/**

## How to setup the project

Go to the [Juicebox Contribution NFT Reward Mechanism](https://github.com/jbx-protocol/juice-nft-rewards/tree/4ac8cb18f5873a4f59341719450be6a91c4fa8e1) and follow instructions in the readme.

Commit: 4ac8cb18f5873a4f59341719450be6a91c4fa8e1

## Motivation

If added to an existing project via a funding cycle, NFT rewards can provide additional incentive for contributors to participate in funding of a project.

## Mechanic

Within one collection, NFTs can be minted within any number of pre-programmed tiers.

Each tier has the following optional properties:

- a contribution floor amount.
- a max quantity.
- a number of voting units to associate with each unit within the tier.
- a reserved rate, allowing a proprtion of units within the tier to be minted to a pre-programmed beneficiary.
- URI, overridable by a URI resolver that can return dynamic values for each unit with the tier.
- a lock date, before which the tier must remain accessible.

New tiers can be added, so long as they respect the contract's `flags` that specify if new tiers can influence voting units, reserved quantities, or be manually minted.

Tiers can also be removed, so long as they are not locked.

An incoming payment can specify any number of tiers to mint as part of the payment, so long as the tier's prices are contained within the paid amount. If specific tiers aren't specified, the best available tier will be minted, unless a flag is specifically sent along with the payment telling the contract to not mint.

If a payment received does not meet a minting threshold or is in excess of the minted tiers, the balance is stored as a credit which will be added to future payments and applied to mints at that time. A flag can also be passed to avoid accepting payments that aren't applied to mints in full. 

The contract's owner can mint on demand from tier's that have been pre-programmed to allow manual token minting.

The NFTs from each tier can also be used for redemptions against the underlying Juicebox treasury. The rate of redemptions corresponds to the price floor of the tier being redeemed, compared to the total price floors of all minted NFTs.

The collection can be used for on-chain governance. Votes can be solicited from all tiers, or only from specific tiers.

## Deploy

The deployer copies the data of a pre-existing cononical version of the 721 contracts, which can be either GlobalGovernance, TierGovernance, or no governance. This was done to keep the deployer contract size small enough to be deployable, without the extra cost of the delegatecalls associated with a proxy pattern. 

## Contracts

|File|SLOC|Description|
|-|-|-|
|[contracts/JBGlobalGovernance.sol]()| 62 |Each NFT can be used for on chain governance, with votes delegatable globally across all tiers|
|[contracts/JBTieredGovernance]()| 329 |Same as Global Governance except delegation is done on a per tier basis|
|[contracts/JBTiered721Delegate.sol]()| 796 |The tiered NFT delegate core logic, without the governance|
|[contracts/JBTiered721DelegateDeployer.sol]()| 139 |The tiered NFT delegate cloner/deployer, allowing to pick a governance style|
|[contracts/JBTiered721DelegateProjectDeployer.sol]()| 257 |Deploy a delegate and create a new Juicebox Project using it|
|[contracts/JBTiered721DelegateStore.sol]()| 1303 |The state storing contract for tiered NFT delegates|
|[contracts/abstract/JB721Delegate.sol]()| 337 |A NFT delegate, offering mint and burn NFT based on pay and redeem|
|[contracts/libraries/JBBitmap.sol]()| 76 |A uint256 bitmap library, to handle removed tiers|
|[contracts/libraries/JBIpfsDecoder.sol]()| 89 |A library to store and read IPFS hashes as 32 bytes words|
|[contracts/libraries/JBTiered721FundingCycleMetadataResolver.sol]()| 18 |A library to read funding cycle metadata in the NFT delegate context|
|Total |3406||