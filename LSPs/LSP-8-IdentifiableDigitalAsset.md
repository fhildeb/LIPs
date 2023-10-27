---
lip: 8
title: Identifiable Digital Asset
author: Claudio Weck <claudio@fanzone.media>, Fabian Vogelsteller <fabian@lukso.network>, Matthew Stevens <@mattgstevens>, Ankit Kumar <@ankitkumar9018>
discussions-to: https://discord.gg/E2rJPP4 (LUKSO), https://discord.gg/PQvJQtCV (FANZONE)
status: Draft
type: LSP
created: 2021-09-02
requires: ERC165, ERC725Y, LSP1, LSP2, LSP4, LSP17
---

<!--You can leave these HTML comments in your merged LIP and delete the visible duplicate text guides, they will not appear and may be helpful to refer to if you edit it again. This is the suggested template for new LIPs. Note that an LIP number will be assigned by an editor. When opening a pull request to submit your LIP, please use an abbreviated title in the filename, `lip-draft_title_abbrev.md`. The title should be 44 characters or less.-->

## Simple Summary

<!--"If you can't explain it simply, you don't understand it well enough." Provide a simplified and layman-accessible explanation of the LIP.-->

The LSP8 Identifiable Digital Asset Standard defines a standard interface for uniquely identifiable digital assets. It allows tokens to be uniquely traded and given with metadata using [ERC725Y][ERC725] and [LSP4](./LSP-4-DigitalAsset-Metadata.md#lsp4metadata).

## Abstract

<!--A short (~200 word) description of the technical issue being addressed.-->

This standard defines an interface for tokens that are identified with a `tokenId`, based on [ERC721][ERC721]. A `bytes32` value is used for `tokenId` to allow many uses of token identification including numbers, contract addresses, and any other unique identifiers (_e.g:_ serial numbers, NFTs with unique names, hash values, etc...).

This standard defines a set of data-key value pairs that are useful to know what the `tokenId` represents, and the associated metadata for each `tokenId`.

## Motivation

<!--The motivation is critical for LIPs that want to change the Lukso protocol. It should clearly explain why the existing protocol specification is inadequate to address the problem that the LIP solves. LIP submissions without sufficient motivation may be rejected outright.-->

This standard aims to support use cases not covered by [LSP7 DigitalAsset][LSP7], by using a `tokenId` instead of an amount of tokens to mint, burn, and transfer tokens. Each `tokenId` may have metadata (either as a on-chain [ERC725Y][ERC725] contract or off-chain JSON) in addition to the [LSP4 DigitalAsset-Metadata][LSP4#erc725ykeys] metadata of the smart contract that mints the tokens. In this way a minted token benefits from the flexibility & upgradability of the [ERC725Y][ERC725] standard, and transfering a token carries the history of ownership and metadata updates. This is beneficial for a new generation of NFTs.

A commonality with [LSP7 DigitalAsset][LSP7] is desired so that the two token implementations use similar naming for functions, events, and using hooks to notify token senders and receivers using LSP1.

## Specification

[ERC165] interface id: `0x1ae9ba1f`

The LSP8 interface ID is calculated as the XOR of the LSP8 interface (see [interface cheat-sheet below](#interface-cheat-sheet)) and the [LSP17 Extendable interface ID](./LSP-17-ContractExtension.md#erc165-interface-id).

### ERC725Y Data Keys - LSP8 Contract

These are the expected data keys for an LSP8 contract that can mints identifiable tokens (NFTs).

This standard can also be combined with the data keys from [LSP4 DigitalAsset-Metadata.][LSP4#erc725ykeys].

#### LSP8TokenIdType

This data key describes the type of the `tokenId` and can take one of the following enum values described in the table below.

In the context of LSP8, a contract implementing the LSP8 standard represents a collection of unique non-fungible tokens (NFT). The LSP8 collection contract is responsible for minting these tokens.

Each token part of the collection is identifiable through its unique `tokenId`.

However, these NFTs can be represented differently depending on the use case. This is referred to as the **type of the tokenId**.

The `LSP8TokenIdType` metadata key provides this information and describes how to treat the NFTs parts of the LSP8 collection.

This MUST NOT be changeable, and set only during initialization of the LSP8 token contract.

| Value |   Type    | Description                                                                                                                                       |
| :---: | :-------: | :------------------------------------------------------------------------------------------------------------------------------------------------ |
|  `0`  | `uint256` | each NFT is represented with a **unique number**. <br> This number is an incrementing count, where each minted token is assigned the next number. |
|  `1`  | `string`  | each NFT is represented using a **unique name** (as a short **utf8 encoded string**, no more than 32 characters long)                             |
|  `2`  | `bytes32` | each NFT is represented using a 32 bytes long **unique identifier**.                                                                              |
|  `3`  | `bytes32` | each NFT is represented using a 32 bytes **hash digest**.                                                                                         |
|  `4`  | `address` | each NFT is represented as its **own smart contract** that can hold its own metadata (_e.g [ERC725Y] compatible_).                                |

```json
{
  "name": "LSP8TokenIdType",
  "key": "0x715f248956de7ce65e94d9d836bfead479f7e70d69b718d47bfe7b00e05b4fe4",
  "keyType": "Singleton",
  "valueType": "uint256",
  "valueContent": "Number"
}
```

#### LSP8MetadataTokenURI:<tokenId>

This data key stores the URI of the metadata for a specific `tokenId`.

It consists of a `bytes4` identifier of the hash function and the URI string.

When metadata JSON is created for a tokenId, the URI COULD be stored in the storage of the LSP8 contract.

The value stored under this data key is a tuple `(bytes4,string)` that contains the following elements:

- `bytes4` = the 4 bytes identifier of the hash function used to generate the URI:
  - if the tokenId is a hash (`LSP8TokenIdType` is `3`): _see details below_.
  - if the tokenId is any other LSP8TokenIdType: MUST be `0x00000000`.
- `string` = the URI where the metadata for the `tokenId` can be retrieved.

```json
{
  "name": "LSP8MetadataTokenURI:<address|uint256|bytes32|string>",
  "key": "0x1339e76a390b7b9ec9010000<address|uint256|bytes32|string>",
  "keyType": "Mapping",
  "valueType": "(bytes4,string)",
  "valueContent": "(Bytes4,URI)"
}
```

> For construction of the Mapping data key see: [LSP2 ERC725Y JSON Schema > `keyType = Mapping`][LSP2#mapping]

**When `bytes4 = 0x00000000`**

The URI of some NFTs could be alterable, for example in the case of NFTs that need their metadata to change overtime.

In this case, the first `bytes4` in the tuple MUST be set to `0x00000000` (4 x zero bytes), which describes that the URI can be changed over the lifetime of the NFTs.

**When `bytes4 = some 4 bytes value` (Example)**

To represent the hash function `keccak256`:

- `bytes4` value in the tuple to represent the hash function `keccak256` = **`0x6f357c6a`**

This can be obtained as follow:

`keccak256('keccak256(utf8)')` = `0x`**`6f357c6a`**`956bf6b8a917ccf88cc1d3388ff8d646810d0393fe69ae7ee228004f`

#### LSP8TokenMetadataBaseURI

This data key defines the base URI for the metadata of each `tokenId`s present in the LSP8 contract.

The complete URI that points to the metadata of a specific tokenId MUST be formed by concatenating this base URI with the `tokenId`.
As `{LSP8TokenMetadataBaseURI}{tokenId}`.

⚠️ TokenIds MUST be in lowercase, even for the tokenId type `address` (= address not checksumed).

- LSP8TokenIdType `2` (= `uint256`)<br>
  e.g. `http://mybase.uri/1234`
- LSP8TokenIdType `1` (= `address`)<br>
  e.g. `http://mybase.uri/0x43fb7ab43a3a32f1e2d5326b651bbae713b02429`
- LSP8TokenIdType `3` or `4` (= `bytes32`)<br>
  e.g. `http://mybase.uri/e5fe3851d597a3aa8bbdf8d8289eb9789ca2c34da7a7c3d0a7c442a87b81d5c2`
- LSP8TokenIdType `5`
  e.g. `http://mybase.uri/my_string`

Some Base URIs could be alterable, for example in the case of NFTs that need their metadata to change overtime.

In this case, the first `bytes4` in the tuple (for the `valueType`/`valueContent`) MUST be set to `0x00000000` (4 x zero bytes), which describes that the URI can be changed over the lifetime of the NFTs.

If the tokenId type is a hash (LSP8TokenIdType `3`), the first `bytes4` in the tuple represents the hash function.

_Example:_

To represent the hash function `keccak256`:

- `bytes4` value in the tuple to represent the hash function `keccak256` = **`0x6f357c6a`**

This can be obtained as follow:

`keccak256('keccak256(utf8)')` = `0x`**`6f357c6a`**`956bf6b8a917ccf88cc1d3388ff8d646810d0393fe69ae7ee228004f`

```json
{
  "name": "LSP8TokenMetadataBaseURI",
  "key": "0x1a7628600c3bac7101f53697f48df381ddc36b9015e7d7c9c5633d1252aa2843",
  "keyType": "Singleton",
  "valueType": "(bytes4,string)",
  "valueContent": "(Bytes4,URI)"
}
```

### ERC725Y Data Keys of external contract for tokenID type 4 (`address`)

When the LSP8 contract uses the [tokenId type `4`](#lsp8tokenidtype) (= `address`), each tokenId minted is an ERC725Y smart contract that can have its own metadata.
We refer to this contract as the **tokenId metadata contract**.

In this case, each tokenId present in the LSP8 contract references an other ERC725Y contract.

The **tokenId metadata contract** SHOULD contain the following ERC725Y data key in its storage.

#### LSP8ReferenceContract

This data key stores the address of the LSP8 contract that minted this specific `tokenId` (defined by the address of the **tokenId metadata contract**).

It is a reference back to the LSP8 Collection it comes from.

If the `LSP8ReferenceContract` data key is set, it MUST NOT be changeable.

```json
{
  "name": "LSP8ReferenceContract",
  "key": "0x708e7b881795f2e6b6c2752108c177ec89248458de3bf69d0d43480b3e5034e6",
  "keyType": "Singleton",
  "valueType": "(address,bytes32)",
  "valueContent": "(Address,bytes32)"
}
```

### LSP8 TokenId Metadata

The metadata for a specific of a uniquely identifiable digital asset (when this tokenId is represented by its own ERC725Y contract) can follow the JSON format of the [`LSP4Metadata`](./LSP-4-DigitalAsset-Metadata.md#lsp4metadata) data key.

This JSON format includes an `"attributes"` field to describe unique properties of the tokenId.

### Methods

#### totalSupply

```solidity
function totalSupply() external view returns (uint256);
```

Returns the number of existing tokens.

**Returns:** `uint256` the number of existing tokens.

#### balanceOf

```solidity
function balanceOf(address tokenOwner) external view returns (uint256);
```

Returns the number of tokens owned by `tokenOwner`.

_Parameters:_

- `tokenOwner` the address to query.

**Returns:** `uint256` the number of tokens owned by this address.

#### tokenOwnerOf

```solidity
function tokenOwnerOf(bytes32 tokenId) external view returns (address);
```

Returns the `tokenOwner` address of the `tokenId` token.

_Parameters:_

- `tokenId` the token to query.

_Requirements:_

- `tokenId` must exist

**Returns:** `address` the token owner.

#### tokenIdsOf

```solidity
function tokenIdsOf(address tokenOwner) external view returns (bytes32[] memory);
```

Returns the list of `tokenIds` for the `tokenOwner` address.

_Parameters:_

- `tokenOwner` the address to query.

**Returns:** `bytes32[]` the list of owned token ids.

#### authorizeOperator

```solidity
function authorizeOperator(address operator, bytes32 tokenId, bytes memory operatorNotificationData) external;
```

Makes `operator` address an operator of `tokenId`.

MUST emit an [AuthorizedOperator event](#authorizedoperator).

_Parameters:_

- `operator` the address to authorize as an operator.
- `tokenId` the token to enable operator status to.
- `operatorNotificationData` the data to send when notifying the operator via LSP1.

_Requirements:_

- `tokenId` must exist
- caller must be current `tokenOwner` of `tokenId`.
- `operator` cannot be calling address.
- `operator` cannot be the zero address.

#### revokeOperator

```solidity
function revokeOperator(address operator, bytes32 tokenId, bytes memory operatorNotificationData) external;
```

Removes `operator` address as an operator of `tokenId`.

MUST emit a [RevokedOperator event](#revokedoperator).

_Parameters:_

- `operator` the address to revoke as an operator.
- `tokenId` the token to disable operator status to.
- `operatorNotificationData` the data to send when notifying the operator via LSP1.

_Requirements:_

- `tokenId` must exist
- caller must be current `tokenOwner` of `tokenId`.
- `operator` cannot be calling address.
- `operator` cannot be the zero address.

#### isOperatorFor

```solidity
function isOperatorFor(address operator, bytes32 tokenId) external view returns (bool);
```

Returns whether `operator` address is an operator of `tokenId`.
Operators can send and burn tokens on behalf of their owners. The tokenOwner is their own operator.

_Parameters:_

- `operator` the address to query operator status for.
- `tokenId` the token to query.

_Requirements:_

- `tokenId` must exist
- caller must be current `tokenOwner` of `tokenId`.

**Returns:** `bool`, TRUE if `operator` address is an operator of `tokenId`, FALSE otherwise.

#### getOperatorsOf

```solidity
function getOperatorsOf(bytes32 tokenId) external view returns (address[] memory);
```

Returns all `operator` addresses of `tokenId`.

_Parameters:_

- `tokenId` the token to query.

_Requirements:_

- `tokenId` must exist
- caller must be current `tokenOwner` of `tokenId`.
- `operator` cannot be calling address.

**Returns:** `address[]` the list of operators.

#### transfer

```solidity
function transfer(address from, address to, bytes32 tokenId, bool force, bytes memory data) external;
```

Transfers `tokenId` token from `from` to `to`. The `force` parameter will be used when notifying the token sender and receiver.

MUST emit a [Transfer event](#transfer) when transfer was successful.

_Parameters:_

- `from` the sending address.
- `to` the receiving address.
- `from` and `to` cannot be the same address.
- `tokenId` the token to transfer.
- `force` when set to TRUE, `to` may be any address; when set to FALSE `to` must be a contract that supports [LSP1 UniversalReceiver][LSP1] and successfully processes a call to `universalReceiver(bytes32 typeId, bytes memory data)`.
- `data` additional data the caller wants included in the emitted event, and sent in the hooks to `from` and `to` addresses.

_Requirements:_

- `from` cannot be the zero address.
- `to` cannot be the zero address.
- `tokenId` token must be owned by `from`.
- If the caller is not `from`, it must be an operator of `tokenId`.

**LSP1 Hooks:**

- If the token sender is a contract that supports LSP1 interface, it SHOULD call the token sender's [`universalReceiver(...)`] function with the parameters below:

  - `typeId`: `keccak256('LSP8Tokens_SenderNotification')` = `0xb23eae7e6d1564b295b4c3e3be402d9a2f0776c57bdf365903496f6fa481ab00`
  - `data`: The data sent SHOULD be ABI encoded and contain the `sender` (address), `receiver` (address), `tokenId` (bytes32) and the `data` (bytes) respectively.

<br>

- If the token recipient is a contract that supports LSP1 interface, it SHOULD call the token recipient's [`universalReceiver(...)`] function with the parameters below:

  - `typeId`: `keccak256('LSP8Tokens_RecipientNotification')` = `0x0b084a55ebf70fd3c06fd755269dac2212c4d3f0f4d09079780bfa50c1b2984d`
  - `data`: The data sent SHOULD be ABI encoded and contain the `sender` (address), `receiver` (address), `tokenId` (bytes32) and the `data` (bytes) respectively.

**Note:** LSP1 Hooks MUST be implemented in any type of token transfer (mint, transfer, burn, transferBatch).

#### transferBatch

```solidity
function transferBatch(address[] memory from, address[] memory to, bytes32[] memory tokenId, bool force, bytes[] memory data) external;
```

Transfers many tokens based on the list `from`, `to`, `tokenId`. If any transfer fails, the call will revert.

MUST emit a [Transfer event](#transfer) for each transfered token.

_Parameters:_

- `from` the list of sending addresses.
- `to` the list of receiving addresses.
- `tokenId` the list of tokens to transfer.
- `force` when set to TRUE, `to` may be any address; when set to FALSE `to` must be a contract that supports [LSP1 UniversalReceiver][LSP1] and successfully processes a call to `universalReceiver(bytes32 typeId, bytes memory data)`.
- `data` the list of additional data the caller wants included in the emitted event, and sent in the hooks to `from` and `to` addresses.

_Requirements:_

- `from`, `to`, `tokenId` lists are the same length.
- no values in `from` can be the zero address.
- no values in `to` can be the zero address.
- `from` and `to` cannot be the same address at the same.
- each `tokenId` token must be owned by `from`.
- If the caller is not `from`, it must be an operator of each `tokenId`.

### Events

#### Transfer

```solidity
event Transfer(address operator, address indexed from, address indexed to, bytes32 indexed tokenId, bool force, bytes data);
```

MUST be emitted when `tokenId` token is transferred from `from` to `to`.

#### AuthorizedOperator

```solidity
event AuthorizedOperator(address indexed operator, address indexed tokenOwner, bytes32 indexed tokenId);
```

MUST be emitted when `tokenOwner` enables `operator` for `tokenId`.

#### RevokedOperator

```solidity
event RevokedOperator(address indexed operator, address indexed tokenOwner, bytes32 indexed tokenId);
```

MUST be emitted when `tokenOwner` disables `operator` for `tokenId`.

## Rationale

<!--The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages. The rationale may also provide evidence of consensus within the community, and should discuss important objections or concerns raised during discussion.-->

There should be a base token standard that allows tracking unique assets for the LSP ecosystem of contracts, which will allow common tooling and clients to be built. Existing tools and clients that expect [ERC721][ERC721] can be made to work with this standard by using "compatability" contract extensions that match the desired interface.

### Token Identifier

Every token is identified by a unique `bytes32 tokenId` which SHALL NOT change for the life of the contract. The pair `(contract address, uint256 tokenId)` is globally unique and a fully-qualified identifier for a specific asset on-chain. While some implementations may find it convenient to use the tokenId as an `uint256` that is incremented for each minted token, callers SHALL NOT assume that tokenIds have any specific pattern to them, and MUST treat the tokenId as a "black box". Also note that a tokenId MAY become invalid (when burned).

The choice of `bytes32 tokenId` allows a wide variety of applications including numbers, contract addresses, and hashed values (ie. serial numbers).

### Operators

To clarify the ability of an address to access tokens from another address, `operator` was chosen as the name for functions, events and variables in all cases. This is originally from [ERC777][ERC777] standard and replaces the `approve` functionality from [ERC721][ERC721].

### Token Transfers

There is only one transfer function, which is aware of operators. This deviates from [ERC721][ERC721] and [ERC777][ERC777] which added functions specifically for the token owner to use, and for those with access to tokens. By having a single function to call this makes it simple to move tokens, and the caller will be exposed in the `Transfer` event as an indexed value.

### Usage of hooks

When a token is changing owners (minting, transfering, burning) an attempt is made to notify the token sender and receiver using [LSP1 UniversalReceiver][LSP1] interface. The implementation uses `_notifyTokenSender` and `_notifyTokenReceiver` as the internal functions to process this.

The `force` parameter sent during `function transfer` SHOULD be used when notifying the token receiver, to determine if it must support [LSP1 UniversalReceiver][LSP1]. This is used to prevent accidental token transfers, which may results in lost tokens: non-contract addresses could be a copy paste issue, contracts not supporting [LSP1 UniversalReceiver][LSP1] might not be able to move tokens.

## Implementation

<!--The implementations must be completed before any LIP is given status "Final", but it need not be completed before the LIP is accepted. While there is merit to the approach of reaching consensus on the specification and rationale before writing code, the principle of "rough consensus and running code" is still useful when it comes to resolving many discussions of API details.-->

A implementation can be found in the [lukso-network/lsp-smart-contracts][LSP8.sol].

ERC725Y JSON Schema `LSP8IdentifiableDigitalAsset`:

```json
[
  {
    "name": "LSP8TokenIdType",
    "key": "0x715f248956de7ce65e94d9d836bfead479f7e70d69b718d47bfe7b00e05b4fe4",
    "keyType": "Singleton",
    "valueType": "uint256",
    "valueContent": "Number"
  },
  {
    "name": "LSP8MetadataTokenURI:<address|uint256|bytes32|string>",
    "key": "0x1339e76a390b7b9ec9010000<address|uint256|bytes32|string>",
    "keyType": "Mapping",
    "valueType": "(bytes4,string)",
    "valueContent": "(Bytes4,URI)"
  },
  {
    "name": "LSP8TokenMetadataBaseURI",
    "key": "0x1a7628600c3bac7101f53697f48df381ddc36b9015e7d7c9c5633d1252aa2843",
    "keyType": "Singleton",
    "valueType": "(bytes4,string)",
    "valueContent": "(Bytes4,URI)"
  },
  {
    "name": "LSP8ReferenceContract",
    "key": "0x708e7b881795f2e6b6c2752108c177ec89248458de3bf69d0d43480b3e5034e6",
    "keyType": "Singleton",
    "valueType": "(address,bytes32)",
    "valueContent": "(Address,bytes32)"
  }
]
```

## Interface Cheat Sheet

```solidity
interface ILSP8 is /* IERC165 */ {

    // ERC173

    event OwnershipTransferred(address indexed previousOwner, address indexed newOwner);


    function owner() external view returns (address);

    function transferOwnership(address newOwner) external override; // onlyOwner

    function renounceOwnership() external virtual; // onlyOwner


    // ERC725Y

    event DataChanged(bytes32 indexed dataKey, bytes dataValue);


    function getData(bytes32 dataKey) external view returns (bytes memory value);

    function setData(bytes32 dataKey, bytes memory value) external; // onlyOwner

    function getDataBatch(bytes32[] memory dataKeys) external view returns (bytes[] memory values);

    function setDataBatch(bytes32[] memory dataKeys, bytes[] memory values) external; // onlyOwner


    // LSP8

    event Transfer(address operator, address indexed from, address indexed to, bytes32 indexed tokenId, bool force, bytes data);

    event AuthorizedOperator(address indexed operator, address indexed tokenOwner, bytes32 indexed tokenId);

    event RevokedOperator(address indexed operator, address indexed tokenOwner, bytes32 indexed tokenId);


    function totalSupply() external view returns (uint256);

    function balanceOf(address tokenOwner) external view returns (uint256);

    function tokenOwnerOf(bytes32 tokenId) external view returns (address);

    function tokenIdsOf(address tokenOwner) external view returns (bytes32[] memory);

    function authorizeOperator(address operator, bytes32 tokenId, bytes memory operatorNotificationData) external;

    function revokeOperator(address operator, bytes32 tokenId, bytes memory operatorNotificationData) external;

    function isOperatorFor(address operator, bytes32 tokenId) external view returns (bool);

    function getOperatorsOf(bytes32 tokenId) external view returns (address[] memory);

    function transfer(address from, address to, bytes32 tokenId, bool force, bytes memory data) external;

    function transferBatch(address[] memory from, address[] memory to, bytes32[] memory tokenId, bool force, bytes[] memory data) external;
}

```

## Copyright

Copyright and related rights waived via [CC0](https://creativecommons.org/publicdomain/zero/1.0/).

[ERC165]: https://eips.ethereum.org/EIPS/eip-165
[ERC721]: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-721.md
[ERC725]: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-725.md
[ERC725Y]: https://github.com/ERC725Alliance/ERC725/blob/develop/docs/ERC-725.md#erc725y
[ERC777]: https://github.com/ethereum/EIPs/blob/master/EIPS/eip-777.md
[LSP1]: ./LSP-1-UniversalReceiver.md
[LSP2#jsonurl]: ./LSP-2-ERC725YJSONSchema.md#JSONURL
[LSP2#mapping]: ./LSP-2-ERC725YJSONSchema.md#mapping
[LSP4#erc725ykeys]: ./LSP-4-DigitalAsset-Metadata.md#erc725ykeys
[LSP7]: ./LSP-7-DigitalAsset.md
[LSP8]: ./LSP-8-IdentifiableDigitalAsset.md
[LSP8.sol]: https://github.com/lukso-network/lsp-universalprofile-smart-contracts/blob/develop/contracts/LSP8IdentifiableDigitalAsset/LSP8IdentifiableDigitalAsset.sol
