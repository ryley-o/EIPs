---
title: NFT Provenance Message Registry
description: Allow owners of NFTs to post messages to enrich on-chain NFT provenance capabilities.
author: Ryley Ohlsen (@ryley-o), Lindsay Gilbert (@lindsgil)
discussions-to: https://ethereum-magicians.org/t/eip-TBD...
status: Draft
type: Standards Track
category: ERC
created: 2022-04-01
requires: 721
---

## Abstract

This EIP describes the details of the NFT Provenance Message Registry, a standard for allowing owners of NFTs to post messages to enrich on-chain NFT provenance capabilities.

<!--
  The Abstract is a multi-sentence (short paragraph) technical summary. This should be a very terse and human-readable version of the specification section. Someone should be able to read only the abstract to get the gist of what this specification does.

  TODO: Remove this comment before submitting
-->

## Motivation

<!--
  This section is optional.

  The motivation section should include a description of any nontrivial problems the EIP solves. It should not describe how the EIP solves those problems, unless it is not immediately obvious. It should not describe why the EIP should be made into a standard, unless it is not immediately obvious.

  With a few exceptions, external links are not allowed. If you feel that a particular resource would demonstrate a compelling case for your EIP, then save it as a printer-friendly PDF, put it in the assets folder, and link to that copy.

  TODO: Remove this comment before submitting
-->

Traditionally, the primary purpose of tracing the provenance of an object or entity is to provide contextual and circumstantial evidence for its original production or discovery, by establishing, as far as practicable, its later history, especially the sequences of its formal ownership, custody and places of storage.

While blockchains provide a strong foundation for tracking the provenance of digital assets, they currently lack a specification to capture the rich context and narrative that is often associated with physical assets. This EIP aims to address this limitation by allowing owners of NFTs to post messages to enrich on-chain NFT provenance capabilities.

**_Examples:_**

- An artist could post a message for an NFT to provide context around the creation of the artwork.
- An artist could post a message for an NFT to provide display details such as resolution, preferred filetype, codec, etc. This ensures that as display technologies advance, the presentation of the artwork can continue to meet the artist's vision.
- A collector could post a message for an NFT detailing its role in representing a logo, brand, or company.
- A collector could post a message for an NFT to provide context around the acquisition of the artwork.
- A museum could post a message for an NFT to provide context around the exhibition of the artwork.

### Proposal: Use of an NFT Provenance Message Registry

This EIP proposes the creation of an NFT Provenance Message Registry, which would allow owners of NFTs to post messages to enrich on-chain NFT provenance capabilities.

## Specification

<!--
  The Specification section should describe the syntax and semantics of any new feature. The specification should be detailed enough to allow competing, interoperable implementations for any of the current Ethereum platforms (besu, erigon, ethereumjs, go-ethereum, nethermind, or others).

  It is recommended to follow RFC 2119 and RFC 8170. Do not remove the key word definitions if RFC 2119 and RFC 8170 are followed.

  TODO: Remove this comment before submitting
-->

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in RFC 2119 and RFC 8174.

Let:

- `NFT` represent a ERC-721 non-fungible token.
- `owner` represent the address that owns the NFT, as indicated by the NFT's `ownerOf` function returning the address when called with the `NFT`s token ID.
- `message` represent a string message that the `owner` of an `NFT` can post to the NFT Provenance Message Registry.

** The NFT Provenance Message Registry must implement INFTProvenanceMessageRegistry **

```solidity
/**
 * @title An immutable registry contract to be deployed as a standalone primitive
 * @dev New project launches may read previous registry messages from here and integrate them into their flow
 */
interface INFTProvenanceMessageRegistry {
    /**
     * @notice Info about a message posted about an ERC-721 token
     * @param owner Address of the owner of the token who sent the message
     * @param timestamp Timestamp when the message was posted
     * @param message The message that was posted
     */
    struct Message {
        address owner;
        uint40 timestamp;
        string message;
    }

    /**
     * @notice Emitted when a provenance message is posted about an ERC-721 token by its owner
     * @param tokenAddress Address of the ERC-721 token contract being posted about
     * @param tokenId ID of the token being posted about
     * @param owner Address of the owner of the token sending the message
     * @param index Index of the message in the token's messages storage array
     */
    event MessagePosted(
        address indexed tokenAddress,
        uint256 indexed tokenId,
        address indexed owner,
        uint256 index,
    );

    /**
     * -----------  WRITE -----------
     */

    /**
     * @notice Post a new message about an ERC-721 token.
     * Reverts if the sender is not the owner of the token.
     * @param tokenAddress Address of the ERC-721 token contract being posted about
     * @param tokenId ID of the token being posted about
     * @param message Message to be posted about the token
     */
    function postMessage(address tokenAddress, uint256 tokenId, string memory message) external;


    /**
     * -----------  READ -----------
     */

    /**
     * @notice Get the number of messages posted about an ERC-721 token.
     * @param tokenAddress Address of the ERC-721 token contract to be queried
     * @param tokenId ID of the token to be queried
     * @return count Number of messages posted about the token
     */
    function getMessageCountForToken(address tokenAddress, uint256 tokenId) external view returns (uint256);

    /**
     * @notice Get a message posted about an ERC-721 token by index.
     * Reverts if the index is out of bounds
     * @param tokenAddress Address of the ERC-721 token contract to be queried
     * @param tokenId ID of the token to be queried
     * @param index Index of the message to be queried
     * @return message Info about the message
     */
    function getMessageAtIndexForToken(address tokenAddress, uint256 tokenId, uint256 index) external view returns (Message memory);

    /**
     * @notice Get all messages posted about ERC-721 tokens by a specific owner address.
     * @param owner Address of the owner who posted the messages
     * @return messages Array of Message structs containing the messages posted about the tokens
     */
    function getMessagesByOwner(address owner) external view returns (Message[] memory);

    /**
     * @notice Get all messages posted about all token ids on a specific ERC-721 token contract.
     * @param tokenAddress Address of the ERC-721 token contract
     * @return messages Array of Message structs containing the messages posted about the tokens
     */
    function getAllMessagesForTokenAddress(address tokenAddress) external view returns (Message[] memory);
}
```

## Rationale

<!--
  The rationale fleshes out the specification by describing what motivated the design and why particular design decisions were made. It should describe alternate designs that were considered and related work, e.g. how the feature is supported in other languages.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

### Storing message data on-chain

The NFT Provenance Message Registry stores messages on-chain to ensure that the messages are immutable and available as long as the NFT asset itself is available. This allows for the messages to be accessed far into the future, laying the foundation for a rich history of an NFT's provenance in the hundreds of years to come.

### Owner-curated messages

The NFT Provenance Message Registry only allows the owner of an NFT to post messages about the NFT. The owner of an asset is properly incentivized to post messages that add value to the NFT asset. This aligned incentive will prevent a great deal of spam on the registry, especially for tokens of historical relevance.

### Message Count, Messages by Owner, and Messages by Token Address

The NFT Provenance Message Registry provides comprehensive functions for accessing provenance data. It allows for retrieving all messages posted by a specific owner address, fetching the number of messages posted about a particular NFT, and accessing messages by their index within a token's message array. Additionally, the registry can retrieve all messages associated with each token ID under a specific token address. These functions allow for gas-bounded, simple access to the messages posted about an NFT, enabling projects to integrate the messages into their flow.

## Backwards Compatibility

<!--

  This section is optional.

  All EIPs that introduce backwards incompatibilities must include a section describing these incompatibilities and their severity. The EIP must explain how the author proposes to deal with these incompatibilities. EIP submissions without a sufficient backwards compatibility treatise may be rejected outright.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

Implementations may choose to implement specific backwards compatibility strategies for historical tokens.

For example, a project may choose to integrate with historically relevant, but non-ERC-721 compliant tokens (e.g. CryptoPunks) by checking for token ownership outside of the ERC-721 standard on a case-by-case basis.

## Security Considerations

<!--
  All EIPs must contain a section that discusses the security implications/considerations relevant to the proposed change. Include information that might be important for security discussions, surfaces risks and can be used throughout the life cycle of the proposal. For example, include security-relevant design decisions, concerns, important discussions, implementation-specific guidance and pitfalls, an outline of threats and risks and how they are being addressed. EIP submissions missing the "Security Considerations" section will be rejected. An EIP cannot proceed to status "Final" without a Security Considerations discussion deemed sufficient by the reviewers.

  The current placeholder is acceptable for a draft.

  TODO: Remove this comment before submitting
-->

This EIP delegates determination of ownership to the ERC-721 standard. This results in a call an arbitrary contract, so proper checks-effects-interactions or non-reentrancy guards must be used.

The NFT provenance message registry is a public, immutable registry. As such, it is important to consider the privacy implications of posting messages to the registry. Owners of NFTs should be aware that the messages they post to the registry will be publicly available and should avoid posting sensitive information or spam-like messages. Likely, off-chain filtering mechanisms will need to be developed to prevent spam and/or inappropriate messages from being displayed, as deemed necessary by the community and/or implementers.

The NFT provenance message registry does not protect against an owner front-running a sale of an NFT by posting a message to the registry that could influence the sale price. Buyers of NFTs should be aware of this risk and should understand that last-minute posting of messages may occur that could future percieved value of an NFT.

## Copyright

Copyright and related rights waived via [CC0](../LICENSE.md).
