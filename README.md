# BIP352 Silent Payments Index Server Specification (WIP)

## Abstract

This specification defines a unified standard for Silent Payments indexing services, enabling wallet developers to implement efficient Silent Payments receive support through standardized APIs. The specification covers three service deployment models and their corresponding API requirements.

## Motivation

The goal of this specification is to provide a guide for wallet and service developers to implement Silent Payments receive support. The Electrum protocol provides a great model for how wallet <-> server communication works today, demonstrating efficient client-server interactions for lightweight Bitcoin wallets. This specification is not meant to replace the Electrum protocol but to extend it to support Silent Payments, leveraging its proven architecture to enhance privacy and efficiency.

Although it might appear that index servers are only necessary for [light client support](https://github.com/setavenger/BIP0352-light-client-specification), reducing scanning operations for wallets improves the user experience, speeds up, and simplifies wallet receiving implementations. Minimizing the time it takes for users to confirm on-chain balances when opening a wallet aligns with existing user experience expectations. Another benefit of having a common specification for tweak services is that developers can iterate on wallet or server improvements once, without requiring extensive coordination for upgrade compatibility. Additionally common endpoints across service providers will allow users and developers the freedom to migrate to the best service provider without requiring architectural changes.

## Current Landscape

[blindbit](https://github.com/setavenger/blindbit-oracle), [cake wallet](https://github.com/cake-tech/blockstream-electrs) and [frigate](https://github.com/sparrowwallet/frigate) have completed indexing server solutions that are being leveraged by wallet developers today. The current approaches successfully dervive tweaks and perform additional scan operations but there is no clear definition of what capabilities each service should provide to the wallet. A wallet team could not swap services without signficant changes to wallet architecture. Once this specification is solidified each of the current implementations will have a framework to align data formats and validate data integrity.

- blindbit-oracle
  - dana-wallet
  - BDK + kyoto
  - blindbit-desktop
- cake-esplora
  - cake wallet
- frigate
  - BDK

## Hosted Service Scanner Models

### Self hosted

A user can setup a bitcoin node and add an indexing service to same infrastructure, wallet registers scan private key and silent payment address + birthdate with service for most efficient initialization and UTXO management.

### Hosted Services

Some users may choose to forgo setting up bitcoin node infrastructure and choose to outsource scanning for payments. Hosted services providing Tweak Server stack can be used anonymously if care is taken by the wallet developer.  A hosted Remote Scanner has equivalent privacy to current Electrum protocol servers.

## Stacks

Stack implementations combine one or more [service roles](#service-roles) to create unique offerings for different privacy objectives. Each of these stacks would likely have different endpoint requirements.  This section summarizes the high level characteristics of each stack.  Refer to the [Stack Comparison](#stack-comparison) table for detailed comparisons.

### Remote Scanner (Ephemeral)

- Configured to respond to any wallet
  - Must index entire chain from Taproot activation
  - Wallet provides scanning key material to server which identifies transactions of interest
- Service is trusted
  - To preserve forward privacy a user should rotate wallet when leaving service
- Novice or Technical User
  - Share scan key pair + spend public key with**server during a session**
  - Relies on well organized UI/UX to educate user on*"scanning concepts"* delayed payment recognition
- Additional Services
  - Wallet Recovery
    - Wallet should maintain SP transaction archive to avoid rescan from wallet birthdate
    - Services may rate limit bulk scanning
  - Mempool Monitoring
    - May be rate limited to a few checks per minute or max per day
  - Labels
    - Change label scanning required
    - Non change labels (optional)
      - Each additional label requires 2 additional EC mult per transaction on the server
      - May be rate limited or require additional fee

### Tweak Server (Anonymous)

- Configured to respond to any wallet
  - Must index entire chain from Taproot activation
  - Wallet request tweaks for a block or range of blocks, use block filters to identify transactions
- Service cannot be fully trusted
  - Users should be able to verify received data against other sources or prove legitimacy independently
- Novice or Technical User
  - Simple to understand workflow to configure and use (set it and forget)
  - Relies on well organized UI/UX to educate user on*"scanning concepts"* delayed payment recognition
- Wallet Recovery
  - Wallet should maintain SP transaction archive to avoid rescan from wallet birthdate
  - Services may rate limit bulk scanning

### My Scanner (Personalized)

- Configured to support a single wallet or small group of wallets
  - Wallet birthdate limits scanning requirements
  - Register scan key pair + spend public key with server
- Trusted service
  - Wallet can assume that any data received from the service does not need to be verified
  - Wallet would receive a list of UTXOs
  - User could be notified when payment is detected by server
- Technical User
  - Willing to learn how to optimize for their situation (configure)
- Additional Services
  - Wallet Recovery
    - Fast - based on filtered UTXO set
  - Mempool Monitoring
  - Receive Notifications

---

#### Stack Comparison

|                                               |                                                                                                                      | Silent Payment Stack                                                                                                  |                                                                                    |
| --------------------------------------------- | -------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **Feature**                             | **Remote Scanner (Ephemeral)**                                                                                 | **Tweak Server (Anonymous)**                                                                                    | **My Scanner (Personalized)**                                                |
| *Privacy*                                   | Privacy from all but Indexer Service<br>(share spend public key & scan private key) for a session <br>Hosted | Mask interest in ScriptPubKeys and Transaction<br>(Care should be taken when filtering for blocks) <br>Hosted | Full<br>(register spend public key & scan private key) <br>Self Hosted     |
| *Skill (User)*                              | Novice / Moderate                                                                                                    | Novice / Moderate                                                                                                     | Experienced / Technical                                                            |
| *Security (Trust)*                          | Trusted (Paid/Free)                                                                                                  | Untrusted (Free)<br>Should download full block on filtered match (no simplified utxo)                             | Trusted (Self hosted)                                                              |
| *Wallet Requirements*                       | Send / Spend SP<br>[Remote Scan](#remote-scanner-integration) <br>Manage UTXOs                                  | Send / Spend SP<br>[Scan For ScriptPubKeys](#tweak-server-integration) <br>Manage UTXOs                          | Send / Spend SP<br>[Fetch Payments](#my-scanner-integration) <br>Manage UTXOs |
| *Wallet Backup / Recovery*                  | Most important<br>Label backup + utxo backup would be best <br>(restore could be rate limited)               | Most important<br>Label backup + utxo backup would be best <br>(restore could be rate limited)                | Label backup is necessary, recovering transactions should be quick                 |
| *Dust*                                      | wallet preference                                                                                                    | wallet preference                                                                                                     | wallet preference                                                                  |
| *Spent Tx Filtering* (Cut Through)          | wallet preference                                                                                                    | Possible                                                                                                              | wallet preference                                                                  |
| *Payment Notifications*                     | Optional                                                                                                             | Not Feasible                                                                                                          | Optional                                                                           |
| *Mempool Monitoring*                        | Optional                                                                                                             | Not Feasible                                                                                                          | Optional                                                                           |
| *Performance / UX (Offline catch up delay)* | Wallet will need to sync (catch up) to find all UTXOs - distributed scan.                                            | Wallet will need to sync (catch up) to find all UTXOs                                                                 | Wallet can have same UX as traditional payment                                     |
| *Server Storage Requirements*               | Large                                                                                                                | Large                                                                                                                 | Medium                                                                             |
| *Client Bandwidth  (wallet <-> service)*    | Low                                                                                                                  | Moderate                                                                                                              | Low                                                                                |
| *Client Transaction Rate*                   | < ~20 day                                                                                                            | < 1 day                                                                                                               | unlimited                                                                          |
| *Labels*                                    | Managed by wallet                                                                                                    | Should labels be offered in wallet at all?                                                                            | Supported - register labels with service                                           |

## Server API endpoints

API endpoints are grouped by Service Roles, thought has been given to include what each implementation should support "Required" vs "Optional".

|                                       |                         |         Stack         |                     |                                                                                |                           |                   |                       |
| ------------------------------------- | :----------------------: | :--------------------: | :------------------: | ------------------------------------------------------------------------------ | :-----------------------: | :---------------: | :--------------------: |
| **Endpoint**                    | **Remote Scanner** | **Tweak Server** | **My Scanner** | ***Description***                                                      | **blindbit-oracle** | **frigate** | **cake esplora** |
|                                       |                         |                       |                     |                                                                                |                           |                   |                       |
| **REST**                        |                         |                       |                     |                                                                                |                           |                   |                       |
| /getinfo                              |         Required         |        Required        |                     | returns basic information about the indexing server instance                   |             Y             |      Similar      |                       |
| /tweaks/:blockheight                  |                         |        Required        |                     | returns tweak data; optional parameters filterSpent + dustLimit                |           Y [1]           |                   |        Similar        |
| /compute-index/:blockheight           |                         |        Optional        |                     | compact transaction index with tweak mappings                                  |           Y [1]           |                   |                       |
| /utxos/:blockheight                   |                         |        Optional        |                     | returns UTXO information for a specific block                                  |           Y [1]           |                   |                       |
|                                       |                         |                       |                     |                                                                                |                           |                   |                       |
| **gRPC**                        |                         |                       |                     |                                                                                |                           |                   |                       |
| StreamComputeIndex                    |                         |        Optional        |                     | returns a compact transaction index with tweak mappings and output information |             Y             |                   |                       |
| StreamBlockScanDataShort              |                         |        Optional        |                     | adds outputs spent to StreamComputeIndex                                       |             Y             |                   |                       |
| GetInfo                               |                         |        Optional        |                     | gRPC counterpart of /getinfo, includes enabled index-mode flags                |             Y             |                   |                       |
| GetBestBlockHeight                    |                         |        Optional        |                     | current indexed tip height                                                     |             Y             |                   |                       |
| GetBlockHashByHeight                  |                         |        Optional        |                     | block hash for a height; errors below the sync start height                    |             Y             |                   |                       |
| GetSpentOutputsShort                  |                         |        Optional        |                     | 8-byte x-only pubkey prefixes of outputs spent in a block (unsalted)           |             Y             |                   |                       |
| GetFullBlock                          |                         |        Optional        |                     | full raw block, so a matched client needs no separate block source             |             Y             |                   |                       |
|                                       |                         |                       |                     |                                                                                |                           |                   |                       |
| **Electrum**                    |                         |                       |                     |                                                                                |                           |                   |                       |
| blockchain.silentpayments.subscribe   |            Y            |                       |                     | subscribes to txid and tweak from start_height to mempool                      |                           |         Y         |                       |
| blockchain.silentpayments.unsubscribe |            Y            |                       |                     | used to cancel long running scans (inverse subscribe)                          |                           |         Y         |                       |
| blockchain.scripthash.subscribe       |           1.1+           |                       |                     |                                                                                |                           |                   |           Y           |
| blockchain.scripthash.get_balance     |           1.1+           |                       |                     |                                                                                |                           |                   |           Y           |
| blockchain.scripthash.get_history     |           1.1+           |                       |                     |                                                                                |                           |                   |           Y           |


[1] blindbit-oracle v2: JSON routes answer but are deprecated in favor of gRPC,
and response shapes changed from v1. The dustLimit / filterSpent query
parameters above are ignored by the v2 REST handlers, and the gRPC request's
dustlimit / cut_through fields are documented upstream as reserved, not yet
applied. v1's filter endpoints were removed; spent outputs are unsalted 8-byte
x-only pubkey prefixes. Verified against a production v2 deployment, 2026-09-02.

### API Request/Response Schemas

---

#### Core Information Endpoints

##### GET /getinfo

```json
{
  "version": "1.0.0",
  "network": "mainnet|testnet|regtest",
  "block_height": 850000,
  "dust_limit": 546,
  "filter_spent": "optional|N/A" # indicates if server supports optional filtering
}
```

#### Tweak Data Endpoints

##### GET /tweaks/:blockheight

```json
{
  "height": 850000,
  "hash": "00000000000000000002a7c4c1e48d76c5a37902165a270156b7a8d72728a054",
  "dust_limit": 546, # zero indicates disabled, requested limit is echoed for confirmation
  "filter_spent": 0|1, # zero indicates off, one == on
  "tweaks": [
     "02abc123...",
     "03def456...",
  ]
}
```

#### Remote Scanner Service Endpoints

##### GET /compute-index/:blockheight

```json
{
  "height": 850000,
  "hash": "00000000000000000002a7c4c1e48d76c5a37902165a270156b7a8d72728a054",
  "dust_limit": 546, # zero indicates disabled, requested limit is echoed for confirmation
  "filter_spent": 0|1, # zero indicates off, one == on
  "index": [
    {
      "txid": "39b8a8f4dedc77c226184eb64e30fd60897f9e4ee76c863a45fccd09f1d88100",
      "tweak": "0298caba5b281fd238bdf00bd357b8991dba9dea0ebf901e9865e43cd70581d270",
      "outputs": [
        "108fc10ba6fb3b29",
        "8363fbeefff27a70"
      ]
    },
    {
      "txid": "798e9a7a7005baf52999a802de299e02f12965049328192e42d1cec0ec649900",
      "tweak": "0324ea7135bcdaa7f5bc86a22e48f8ab8b12b1d64a486a787ea989d942222bfebd",
      "outputs": [
        "f382b8a3c968cb03"
      ]
    }
  ]
}
```

##### StreamComputeIndex

Request:

```json
	"start": 800000,
	"end":800001,
	"dust_limit": 546, # ?
	"filter_spent": true|false # ?
```

Response:

```json
{
  "blockIdentifier": {
    "blockHash": "AAAAAAAAAAAAAqfEweSNdsWjeQIWWicBVreo1ycooFQ=",
    "blockHeight": "800000"
  },
  "index": [
    {
      "txid": "Obio9N7cd8ImGE62TjD9YIl/nk7nbIY6RfzNCfHYgQA=",
      "tweak": "ApjKulsoH9I4vfAL01e4mR26neoOv5AemGXkPNcFgdJw",
      "outputsShort": "EI/BC6b7OymDY/vu//J6cA=="
    },
    {
      "txid": "eY6aenAFuvUpmagC3imeAvEpZQSTKBkuQtHOwOxkmQA=",
      "tweak": "AyTqcTW82qf1vIaiLkj4q4sSsdZKSGp4fqmJ2UIiK/69",
      "outputsShort": "84K4o8loywM=" # concatenation of first 8bytes of each output in transaction
    }
  ]
}
{
  "blockIdentifier": {
    "blockHash": "AAAAAAAAAAAAAqfEweSNdsWjeQIWWicBVreo1ycooFQ=",
    "blockHeight": "800001"
  },
  "index": []
}
```

##### StreamBlockScanDataShort

##### blockchain.silentpayments.subscribe(scan_private_key, spend_public_key, start)

Request:

```json
  "scan_sk": "abc123...", # 64 character hex string
  "spend_pk": "03def456...", # 66 character hex string
  "start": 850000 | <timestamp> # block height or timestamp to start scanning from
```

Response:

```json
{
  "subscription": {
    "address": "sp1qqgste7k9hx0qftg6qmwlkqtwuy6cycyavzmzj85c6qdfhjdpdjtdgqjuexzk6murw56suy3e0rd2cgqvycxttddwsvgxe2usfpxumr70xc9pkqwv",
    "start_height": 882000
  },
  "progress": 1.0,
  "history": [
    {
      "height": 890004,
      "tx_hash": "acc3758bd2a26f869fcc67d48ff30b96464d476bca82c1cd6656e7d506816412",
      "tweak_key": "0314bec14463d6c0181083d607fecfba67bb83f95915f6f247975ec566d5642ee8"
    },
    {
      "height": 905008,
      "tx_hash": "f3e1bf48975b8d6060a9de8884296abb80be618dc00ae3cb2f6cee3085e09403",
      "tweak_key": "024ac253c216532e961988e2a8ce266a447c894c781e52ef6cee902361db960004"
    },
    {
      "height": 0,
      "tx_hash": "f4184fc596403b9d638783cf57adfe4c75c605f6356fbc91338530e9831e9e16",
      "tweak_key": "03aeea547819c08413974e2ab2b12212e007166bb2058f88b009e082b9b4914a58"
    }
  ]
}
```

*Note: [Additional Information (Frigate)](https://github.com/sparrowwallet/frigate?tab=readme-ov-file#blockchainsilentpaymentssubscribe)*

#### Error Response Format

```json
{
  "error": {
    "code": 400,
    "message": "Invalid block height",
    "details": "Block height must be between 0 and current tip"
  }
}
```

## Diagrams

### Wallet Integration Patterns

##### [Tweak Server Integration](https://github.com/silent-payments/BIP0352-index-server-specification/blob/main/diagrams/dana-blindbit.mmd)

```
(per block)
1. Fetch tweaks `/tweaks-index/:height`
2. Compute the possible scriptPubKeys for n = 0 and each label n+1
3. Fetch filter `/filter/utxos/:height` (BIP 158)
4. Match scriptPubKeys against filter
    - If no match: go to 1. with height + 1
    - Else: continue with 5.
5. Fetch block, verify owned UTXOs and add to wallet
    - Go to 1. with height + 1
```

##### [Remote Scanner Integration](https://github.com/silent-payments/BIP0352-index-server-specification/blob/main/diagrams/remote_scanner_sequence.mmd)

```
1. Wallet requests tweak + txid for a block or range of blocks - provide sp address and scan private key
2. Wallet performs Electrum style transaction discovery using txid + tweak
3. Wallet maintain local UTXO state for spending
```

##### [My Scanner Integration](https://github.com/silent-payments/BIP0352-index-server-specification/blob/main/diagrams/my_scanner_sequence.mmd)

```
1. Register wallet sp address and scan private key with /register endpoint
2. Periodically query `/wallet/utxos` for updates
3. Handle notifications of unconfirmed transactions if supported
4. Maintain local UTXO state for spending
```

## Testing

Reference implementations or specific test vectors for section 2 and 3. Section 1 is fully defined in BIP352 Spec

1. Verify Tweak Derivation -[BIP352 Test Vectors](https://github.com/bitcoin/bips/blob/master/bip-0352/send_and_receive_test_vectors.json)
2. Block Processing
   1. Block Vectors
      1. expected tweaks for a given block
   2. Pruning
      1. Spent Tx Filtering (cut-through)
      2. Dust

### Reference Test Vectors

#### Block Processing Test Vector

- [Embedded Service Comparison [Block Test Vectors]](https://github.com/silent-payments/tweak-service-auditor/issues/1)

#### Interoperability Tests

- Service switching without data loss
- Reorg handling consistency
- Rate limiting behavior verification

## Security and Authentication

### Authentication Models

###### Anonymous Services (Tweak Servers)

- No authentication required for public endpoints
- Rate limiting applied per IP address
- No user registration or key storage

###### Scanner Services

- Token or wallet ID based authentication
- Key storage should be ephemeral for the life of a session
  - Use token for fetching scan results

### Security Considerations

##### Rate Limiting Standards

- Anonymous services: Maximum 10 blocks / minute
- Scanner services: Depends on load and service offering

##### Privacy Protection

- Services should not log request patterns that could deanonymize users
- Tweak servers should implement request batching to reduce timing correlation

##### Data Integrity

- Block hash verification required for reorg detection
- Tweak data should be verifiable against full node data
- Checksums recommended for bulk data transfers (How does a wallet know all tweaks were received for a given block request?)

### Other Considerations

- Pagination for large bulk results

---

# Developer Technical Reference

## Service Roles

There are three distinct service roles (steps) required to find silent payments. Either the wallet or service has to do the work. The following section details different service components and describes the key role it provides in scanning for payments. Even though the services can be combined into one [Stack](#stacks) of capabilities all standard endpoints for a given service must be implemented. Additional endpoints can be offered but will not be captured in this specification. Each wallet implementation will have to choose which Stack to build based on the wallet / user objectives.

### Roles:

##### Block Processor

- *requires access to full node*
- Compute tweaks for all blocks
- Serve Tweaks and relevant TXOs for each block
- Examples
  - blindbit-oracle, Electrs, ElectrumX, Esplora, Fullcrum, silentiumd

##### Block Filter Manager

- *requires access to full node or BP*
- Serve CBF (BIP158) headers & filters
- Examples
  - blindbit-oracle, kyoto (bdk-sp), silentiumd?

##### SP Scanner

- *requires access to full node or BP or BFM*
- Wallet shares: scan_sk, scan_pk, spend_pk with service
  - Permenant (Hosted) or Ephemeral (Session)
- Find owned UTXOs since last scan or for a range of blocks
- Examples
  - [sp-client::SpScanner](https://github.com/cygnet3/sp-client/blob/master/src/scanner/scanner.rs),[sp-poc](https://github.com/nymius/sp-poc/tree/feat/use_bdk_sp) (Kyoto + bdk-sp)

---

#### Service Role Comparison

|                                               |                           | Service Roles                    |                                  |
| --------------------------------------------- | ------------------------- | -------------------------------- | -------------------------------- |
| **Feature**                             | **Block Processor** | **Block Filter Manager**   | **SP Scanner**             |
| *Privacy*                                   | Yes                       | Yes                              | No                               |
| *Security (Trust)*                          | N/A                       | N/A                              | Yes                              |
| *Dust*                                      | Possible                  | Possible                         | Possible                         |
| *Spent Tx Filtering*                        | Possible                  | Possible                         | Possible                         |
| *Payment Notifications*                     | No                        | Maybe                            | Yes                              |
| *Mempool Monitoring*                        | No                        | Maybe                            | Yes                              |
| *Performance / UX (Offline catch up delay)* | Slow - sync required      | Depends on wallet implementation | Depends on wallet implementation |
| *Server Storage Requirements*               | Large                     | Small                            | Small                            |
| *Client Bandwidth  (wallet <-> service)*    | Large/Moderate            | Large/Moderate                   | Small                            |
| *Client Transaction Rate*                   | < 2 per day               | < 2 per day                      | unlimited                        |
| *Labels*                                    | No                        | No                               | Yes                              |
