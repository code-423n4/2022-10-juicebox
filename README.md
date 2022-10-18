# Juicebox contest details
- $47,500 USDC main award pot
- $2,500 USDC gas optimization award pot
- Join [C4 Discord](https://discord.gg/code4rena) to register
- Submit findings [using the C4 form](https://code4rena.com/contests/2022-10-juicebox-contest/submit)
- [Read our guidelines for more details](https://docs.code4rena.com/roles/wardens)
- Starts October 18, 2022 20:00 UTC
- Ends October 23, 2022 20:00 UTC
- The [Juicebox contracts](https://github.com/jbx-protocol/juice-nft-rewards/tree/f9893b1497098241dd3a664956d8016ff0d0efd0) to be audited

## How to setup the project

Go to the [Juicebox Contribution NFT Reward Mechanism](https://github.com/jbx-protocol/juice-nft-rewards/tree/f9893b1497098241dd3a664956d8016ff0d0efd0) and follow instructions in the readme.

Commit: f9893b1497098241dd3a664956d8016ff0d0efd0

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

If a tier's contribution floor is specified in a currency different to the incoming payment, a `JBPrices` contract will by used for trying to normalize the values.

If a payment received does not meet a minting threshold or is in excess of the minted tiers, the balance is stored as a credit which will be added to future payments and applied to mints at that time. A flag can also be passed to avoid accepting payments that aren't applied to mints in full. 

The contract's owner can mint on demand from tier's that have been pre-programmed to allow manual token minting.

The NFTs from each tier can also be used for redemptions against the underlying Juicebox treasury. The rate of redemptions corresponds to the price floor of the tier being redeemed, compared to the total price floors of all minted NFTs.

The NFTs can serve as utilities for on-chain governance if specified during the collection's deployment. Voting delegation can be made on a per-tier basis, or on a global basis.

## Architecture

An understanding of how the Juicebox protocol's pay and redeem functionality works is an important prereq to understanding how this repo's contracts work and attach themselves to Juicebox's regular operating behavior. This contract specifically makes use of the DataSource+Delegate pattern. See https://info.juicebox.money/dev/.

In order to use NFT rewards, a Juicebox project should launched from `JBTiered721DelegateProjectDeployer` instead of a `JBController`. This Deployer will deploy a `JBTiered721Delegate` (through it's reference to a `JBTiered721DelegateDeployer`) and attach it to the first funding cycle of the newly launched project as a DataSource and Delegate. Funding cycle reconfigurations can also be done using the `JBTiered721DelegateProjectDeployer`, though it will need to have Operator permissions from the project's owner.

The abstract `JB721Delegate` implementation of the ERC721 Juicebox DataSource+Delegate extension can be used for any distribution mechanic. This repo includes one implementation – the `JBTiered721Delegate` – as well as two extensions that offer on-chain governance capabilities to the distributed tokens. 

All `JBTiered721Delegate`'s use a generic `JBTiered721DelegateStore` to store it's data.

## Deploy

The deployer copies the data of a pre-existing cononical version of the 721 contracts, which can be either GlobalGovernance, TierGovernance, or no governance. This was done to keep the deployer contract size small enough to be deployable, without the extra cost of the delegatecalls associated with a proxy pattern. 

## Contracts

|File|Description|
|-|-|
|[contracts/JBGlobalGovernance.sol]()| Each NFT can be used for on chain governance, with votes delegatable globally across all tiers|
|[contracts/JBTieredGovernance]()| Same as Global Governance except delegation is done on a per tier basis|
|[contracts/JBTiered721Delegate.sol]()| The tiered NFT delegate core logic, without the governance|
|[contracts/JBTiered721DelegateDeployer.sol]()| The tiered NFT delegate cloner/deployer, allowing to pick a governance style|
|[contracts/JBTiered721DelegateProjectDeployer.sol]()| Deploy a delegate and create a new Juicebox Project using it|
|[contracts/JBTiered721DelegateStore.sol]()| The state storing contract for tiered NFT delegates|
|[contracts/abstract/JB721Delegate.sol]()| A NFT delegate, offering mint and burn NFT based on pay and redeem|
|[contracts/libraries/JBBitmap.sol]()| A uint256 bitmap library, to handle removed tiers|
|[contracts/libraries/JBIpfsDecoder.sol]()| A library to store and read IPFS hashes as 32 bytes words|
|[contracts/libraries/JBTiered721FundingCycleMetadataResolver.sol]()| A library to read funding cycle metadata in the NFT delegate context|


## Scope
### Files in scope
|File|[SLOC](#nowhere "(nSLOC, SLOC, Lines)")|[Coverage](#nowhere "(Lines hit / Total)")|
|:-|:-:|:-:|
|_Contracts (6)_|
|[contracts/JB721GlobalGovernance.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JB721GlobalGovernance.sol)|[24](#nowhere "(nSLOC:13, SLOC:24, Lines:62)")|-|
|[contracts/JBTiered721DelegateDeployer.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateDeployer.sol) [🖥](#nowhere "Uses Assembly") [🌀](#nowhere "create/create2")|[62](#nowhere "(nSLOC:59, SLOC:62, Lines:139)")|-|
|[contracts/JBTiered721DelegateProjectDeployer.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateProjectDeployer.sol)|[121](#nowhere "(nSLOC:83, SLOC:121, Lines:257)")|-|
|[contracts/JB721TieredGovernance.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JB721TieredGovernance.sol) [Σ](#nowhere "Unchecked Blocks")|[135](#nowhere "(nSLOC:93, SLOC:135, Lines:329)")|-|
|[contracts/JBTiered721Delegate.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721Delegate.sol) [Σ](#nowhere "Unchecked Blocks")|[328](#nowhere "(nSLOC:273, SLOC:328, Lines:796)")|-|
|[contracts/JBTiered721DelegateStore.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/JBTiered721DelegateStore.sol) [🖥](#nowhere "Uses Assembly") [Σ](#nowhere "Unchecked Blocks")|[547](#nowhere "(nSLOC:472, SLOC:547, Lines:1333)")|-|
|_Abstracts (1)_|
|[contracts/abstract/JB721Delegate.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/abstract/JB721Delegate.sol) [💰](#nowhere "Payable Functions") [Σ](#nowhere "Unchecked Blocks")|[146](#nowhere "(nSLOC:117, SLOC:146, Lines:337)")|-|
|_Libraries (3)_|
|[contracts/libraries/JBTiered721FundingCycleMetadataResolver.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/libraries/JBTiered721FundingCycleMetadataResolver.sol)|[13](#nowhere "(nSLOC:13, SLOC:13, Lines:18)")|-|
|[contracts/libraries/JBBitmap.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/libraries/JBBitmap.sol)|[37](#nowhere "(nSLOC:25, SLOC:37, Lines:76)")|-|
|[contracts/libraries/JBIpfsDecoder.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/libraries/JBIpfsDecoder.sol)|[54](#nowhere "(nSLOC:50, SLOC:54, Lines:89)")|-|
|Total (over 10 files):| [1467](#nowhere "(nSLOC:1198, SLOC:1467, Lines:3436)")| -|


### All other source contracts (not in scope)
|File|[SLOC](#nowhere "(nSLOC, SLOC, Lines)")|[Coverage](#nowhere "(Lines hit / Total)")|
|:-|:-:|:-:|
|_Contracts (1)_|
|[contracts/abstract/ERC721.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/abstract/ERC721.sol) [🖥](#nowhere "Uses Assembly") [♻️](#nowhere "TryCatch Blocks")|[214](#nowhere "(nSLOC:158, SLOC:214, Lines:447)")|-|
|_Abstracts (1)_|
|[contracts/abstract/Votes.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/abstract/Votes.sol)|[86](#nowhere "(nSLOC:73, SLOC:86, Lines:178)")|-|
|_Interfaces (6)_|
|[contracts/interfaces/IJB721Delegate.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/interfaces/IJB721Delegate.sol)|[13](#nowhere "(nSLOC:13, SLOC:13, Lines:23)")|-|
|[contracts/interfaces/IJBTiered721DelegateDeployer.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/interfaces/IJBTiered721DelegateDeployer.sol)|[15](#nowhere "(nSLOC:12, SLOC:15, Lines:19)")|-|
|[contracts/interfaces/IJBTiered721DelegateProjectDeployer.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/interfaces/IJBTiered721DelegateProjectDeployer.sol)|[28](#nowhere "(nSLOC:16, SLOC:28, Lines:35)")|-|
|[contracts/interfaces/IJB721TieredGovernance.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/interfaces/IJB721TieredGovernance.sol)|[39](#nowhere "(nSLOC:31, SLOC:39, Lines:51)")|-|
|[contracts/interfaces/IJBTiered721Delegate.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/interfaces/IJBTiered721Delegate.sol)|[59](#nowhere "(nSLOC:43, SLOC:59, Lines:82)")|-|
|[contracts/interfaces/IJBTiered721DelegateStore.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/interfaces/IJBTiered721DelegateStore.sol)|[89](#nowhere "(nSLOC:47, SLOC:89, Lines:131)")|-|
|_Structs (14)_|
|[contracts/structs/JBBitmapWord.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBBitmapWord.sol)|[5](#nowhere "(nSLOC:5, SLOC:5, Lines:11)")|-|
|[contracts/structs/JBTiered721MintForTiersData.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBTiered721MintForTiersData.sol)|[5](#nowhere "(nSLOC:5, SLOC:5, Lines:11)")|-|
|[contracts/structs/JBTiered721MintReservesForTiersData.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBTiered721MintReservesForTiersData.sol)|[5](#nowhere "(nSLOC:5, SLOC:5, Lines:11)")|-|
|[contracts/structs/JBTiered721SetTierDelegatesData.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBTiered721SetTierDelegatesData.sol)|[5](#nowhere "(nSLOC:5, SLOC:5, Lines:11)")|-|
|[contracts/structs/JBTiered721FundingCycleMetadata.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBTiered721FundingCycleMetadata.sol)|[6](#nowhere "(nSLOC:6, SLOC:6, Lines:13)")|-|
|[contracts/structs/JBTiered721Flags.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBTiered721Flags.sol)|[7](#nowhere "(nSLOC:7, SLOC:7, Lines:15)")|-|
|[contracts/structs/JB721PricingParams.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JB721PricingParams.sol)|[9](#nowhere "(nSLOC:9, SLOC:9, Lines:18)")|-|
|[contracts/structs/JBStored721Tier.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBStored721Tier.sol)|[10](#nowhere "(nSLOC:10, SLOC:10, Lines:21)")|-|
|[contracts/structs/JB721TierParams.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JB721TierParams.sol)|[12](#nowhere "(nSLOC:12, SLOC:12, Lines:25)")|-|
|[contracts/structs/JB721Tier.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JB721Tier.sol)|[13](#nowhere "(nSLOC:13, SLOC:13, Lines:27)")|-|
|[contracts/structs/JBReconfigureFundingCyclesData.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBReconfigureFundingCyclesData.sol)|[13](#nowhere "(nSLOC:13, SLOC:13, Lines:25)")|-|
|[contracts/structs/JBLaunchFundingCyclesData.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBLaunchFundingCyclesData.sol)|[15](#nowhere "(nSLOC:15, SLOC:15, Lines:27)")|-|
|[contracts/structs/JBLaunchProjectData.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBLaunchProjectData.sol)|[17](#nowhere "(nSLOC:17, SLOC:17, Lines:30)")|-|
|[contracts/structs/JBDeployTiered721DelegateData.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/structs/JBDeployTiered721DelegateData.sol)|[23](#nowhere "(nSLOC:23, SLOC:23, Lines:41)")|-|
|_Other (1)_|
|[contracts/enums/JB721GovernanceType.sol](https://github.com/jbx-protocol/juice-nft-rewards/blob/f9893b1497098241dd3a664956d8016ff0d0efd0/contracts/enums/JB721GovernanceType.sol)|[6](#nowhere "(nSLOC:6, SLOC:6, Lines:8)")|-|
|Total (over 23 files):| [694](#nowhere "(nSLOC:544, SLOC:694, Lines:1260)")| -|

## API Docs

https://info.juicebox.money/dev/api/contracts/or-delegates/or-abstract/jb721delegate/

## Slither

Slither does not currently work on the repo. If you find a workaround, please share in the Discord.

## Scoping Details 
```
- If you have a public code repo, please share it here:   https://github.com/jbx-protocol/juice-nft-rewards 
- How many contracts are in scope?:   5
- Total SLoC for these contracts?:   ~1460
- How many external imports are there?:   1
- How many separate interfaces and struct definitions are there for the contracts within scope?:   5 interfaces, 11 structs
- Does most of your code generally use composition or inheritance?:   50/50
- How many external calls?:   0
- What is the overall line coverage percentage provided by your tests?:   100%
- Is there a need to understand a separate part of the codebase / get context in order to audit this part of the protocol?:   true
- Please describe required context:   pay dataSource/delegate, redeem dataSource/delegate from the core Juicebox protocol
- Does it use an oracle?:   false
- Does the token conform to the ERC20 standard?:   no
- Are there any novel or unique curve logic or mathematical models?:   no
- Does it use a timelock function?:   no
- Is it an NFT?:   yeah
- Does it have an AMM?:   no
- Is it a fork of a popular project?:   false
- Does it use rollups?:   false
- Is it multi-chain?:   false
- Does it use a side-chain?:   false
```

## API Docs

API docs for the NFT Rewards system are evolving. See https://info.juicebox.money/dev/api/contracts/or-delegates/or-abstract/jb721delegate/, as well as the build/ and learn/ sections.
