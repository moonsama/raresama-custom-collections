# ERC721 contract requirements
These are the guidelines to get a custom collection listed on [Raresama](https://raresama.com).

## Standard & implementation
Please follow & extend the official [ERC721](https://eips.ethereum.org/EIPS/eip-721) standard.
We recommend using the well audited [OpenZeppelin](https://github.com/OpenZeppelin/openzeppelin-contracts/tree/master/contracts/token/ERC721) libraries. Security comes first. Don't get rekt.

## Collection URI
We support and rely on collection/contract level metadata. On top of the specification, your contract must implement the following function too.
```solidity
function contractURI() public view returns (string memory);
```
This returns an URL to a `JSON` object containing contract level metadata. See metadata structure below.

## Recognized ownership
At Moonsama we use OpenZeppelin's AccessControl extensions to manage contract roles and ownership. To be recognized owner of your collection:
- make sure your contract uses AccessControl
- Have the owner account of your contract the following role:
   ```
   bytes32 public constant ADMIN_ROLE = keccak256("ADMIN_ROLE"); // a49807205ce4d355092ef5a8a18f56e8913cf4a201fbe287825b095693c21775
   ```
- This means that a `hasRole('a49807205ce4d355092ef5a8a18f56e8913cf4a201fbe287825b095693c21775', <owner_account>)` call on your contract must return `true` for the owner address. That's how Raresama recognizes ownership.

Besides these requirements we also recommend to add more refined roles, like
```
bytes32 public constant MINTER_ROLE = keccak256("MINTER_ROLE");
```
or
```
bytes32 public constant URI_SETTER_ROLE = keccak256("URI_SETTER_ROLE");
```
or 
```
bytes32 public constant GOVERNANCE_ROLE = keccak256("GOVERNANCE_ROLE"); // grants/revokes admin roles
```
which can result in safer key management in a team.

## Indexability
Your contract must follow the ERC721 specifications.
Apart from that we use the following conventions to make indexing of a collections easier:

- URI change on a token is signaled by `URI(uint256 tokeId);` event
- URI change of the whole collection is signaled by `URIAll()` event
- change of the collection/contract URI is signaled by the `ContractURI()` event

The indexer listens to these specific events to know when to update the metadata stored for a token or collection. As long as you follow these rules, any change u make on the token URIs, Raresama will automatically pick them up. It is necessary e.g. in a blind auction then reveal type of distribution, or for dynamically changing NFTs.

## Royalties
We support [EIP2981](https://eips.ethereum.org/EIPS/eip-2981). Pleae implement and manage it yourself if you want royalty payments to be made on each sale.

## Metadata

### IPFS links
Best practice is to upload everything to IPFS, as it has some guarantees against rugging.
IPFS links are in the format of `ipfs://<ipfs_hash>` or `ipfs://<ipfs_folder_hash>/path/..`. Make sure the IPFS content is pinned and our gateway can reach it. If you give the IPFS link with your own IPFS gateway substituted, make sure to maintain it so it's accessible at all times. We will not pin your IPFS content ourselves.
### Metadata
They are exclusively `JSON` objects.
### Token URI

This is the individual URI for a token in a collection. Our base is the [OpenSea's erc721 metadata standard](https://docs.opensea.io/docs/metadata-standards). We have extended it with some of our conventions:
```json
{
  "image": "String. URL pointing to a media type. Example: ipfs://QmVZDhCCDdJs8LDXAxfYT1KjuFpdcH33eazDAzvNVYoNbj/exo_2.jpg",
  "name": "String. Token name. Example: Moonsama",
  "description": "String. Text. Example: This is an exo test.",
  "external_url": "String. Optional. Example: https://moonsama.com",
  "artist": "String. LOWERCASED ETHEREUM ADDRESS OF THE ARTIST OF THE TOKEN (if it differs in the collection). E.g.: 0x685a9fbba1ba311a27d3f5132e08f11a70f57be6. This needs to match with the registered address/profile of the Artist to appear with the tag of the artist on Raresama.",
  "artist_url": "String. Optional. https://twitter.com/KyilkhorSama",
  "attributes": [
    {
      "trait_type": "Minecraft Skin",
      "value": "ipfs://QmeQwmw5Dhsvcxyy9KELjhzW23A2ZAPfHKQeuLnt83vxB6"
    },
    {
      "trait_type": "Minecraft Skin External URL",
      "value": "https://minesk.in/a97d454937ba4eb5ae3a98c26b82ae30"
    },
    {
      "trait_type": "Skin Tone",
      "value": "Tan"
    },
    {
      "display_type": "range_1_100", 
      "trait_type": "Aqua Power", 
      "value": 40
    }
  ]
}

```
It's better if attribute trait types have capitalized letters, and have space instead of underscore in between words.

Of course you can add as many arbitraty fields in the metadata as you want, on top of what we need to support your token on Raresama.

#### Minecraft support

You can add out-of-the-box Minecraft support (e.g. Carnage) for your tokens by adding the followings to the attribute list of your token:
```json
    {
      "trait_type": "Minecraft Skin",
      "value": "https://moonsama.mypinata.cloud/ipfs/QmeQwmw5Dhsvcxyy9KELjhzW23A2ZAPfHKQeuLnt83vxB6"
    },
    {
      "trait_type": "Minecraft Skin External URL",
      "value": "https://minesk.in/a97d454937ba4eb5ae3a98c26b82ae30"
    },
```

## Contract URI
This URI is meant to convey information about your whole collection, and it also controls how the collection page is displayed on Raresama. These values are the default values understood for all the tokens in the collection. You can overwrite them with values included in the Token URI, e.g. artist.
```json
{
    "name": "String. Name of your whole collection. Example: Moonsama Multiverse Art",
    "description": "String. Description/text for your whole collections. Example: Collection of art that transcends the boundaries of blockchains, metaverses and the imagination.",
    "image": "String. URL pointing to your collection cover media. Best is a 4500 x 1500 pixel large picture. Example: ipfs://QmUdFKHa6GhwTxQL5hkeHwxyoHaigcsXrisSRb2EmP5H3S",
    "external_link": "String. Example: https://moonsama.com",
    "artist": "String. LOWERCASED ETHEREUM ADDRESS OF THE ARTIST OF THE WHOLE COLLECTION. E.g.: 0x685a9fbba1ba311a27d3f5132e08f11a70f57be6. This needs to match with the registered address/profile of the Artist to appear with the tag of the artist on Raresama.",
    "artist_url": "String. Optional. https://twitter.com/KyilkhorSama",
}
```

That should be all for now.