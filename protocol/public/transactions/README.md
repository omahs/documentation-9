# Transactions

| Tx type # | Tx type name                                                                   |
| --------- | ------------------------------------------------------------------------------ |
| 1         | Genesis Transaction                                                            |
| 4         | [Transfer Transaction](transfer-transaction.md)                                |
| 8         | [Lease Transaction](../../../node/public-node/rest-api/lease\_transactions.md) |
| 9         | [Cancel Lease Transaction](cancel-lease-transaction.md)                        |
| 11        | [Mass Transfer Transaction](mass\_transfer\_transaction.md)                    |
| 12        | [Data Transaction](data.md)                                                    |
| 13        | [Set Script Transaction](set-script.md)                                        |
| 15        | [Anchor Transaction](anchor.md)                                                |
| 16        | [Association Transaction](association.md)                                      |
| 17        | [Revoke Association Transaction](revoke-association.md)                        |
| 18        | [Sponsor Transaction](sponsor.md)                                              |
| 19        | [Cancel Sponsor Transaction](cancel-sponsor.md)                                |
| 20        | [Register Transaction](register.md)                                            |
| 21        | [Burn Transaction](burn.md)                                                    |
| 22        | Mapped Anchor Transaction                                                      |
| 23        | Statement Transaction                                                          |

## Signing a transaction

The process is as follows: create a binary message for signing, then create a signature using the private key.

To validate a signature, the same binary message must be constructed. For this, the order of the fields matter, if you switch the order, the message will be different. The public key can be used for validation.

{% hint style="info" %}
The binary message differs for each transaction type. Please check the documentation.
{% endhint %}

### Example

Transaction data:

| Field                                           | Value                                        |
| ----------------------------------------------- | -------------------------------------------- |
| Sender address (not used, just for information) | 3N9Q2sdkkhAnbR4XCveuRaSMLiVtvebZ3wp          |
| Private key (used for signing, not in tx data)  | 7VLYNhmuvAo5Us4mNGxWpzhMSdSSdEbEPFUDKSnA6eBv |
| Public key                                      | EENPV1mRhUD9gSKbcWt84cqnfSGQP5LkCu5gMBfAanYH |
| Recipient address                               | 3NBVqYXrapgJP9atQccdBPAgJPwHDKkh6A8          |
| Amount                                          | 1                                            |
| Fee                                             | 1                                            |
| Timestamp                                       | 1479287120875                                |
| Attachment (as byte array)                      | \[1, 2, 3, 4]                                |

Bytes:

| # | Field name              | Type  | Position | Length | Value         | Base58 bytes value                           |
| - | ----------------------- | ----- | -------- | ------ | ------------- | -------------------------------------------- |
| 1 | Transaction type (0x04) | Byte  | 0        | 1      | 4             | 5                                            |
| 2 | Sender's public key     | Bytes | 1        | 32     | ...           | EENPV1mRhUD9gSKbcWt84cqnfSGQP5LkCu5gMBfAanYH |
| 3 | Timestamp               | Long  | 33       | 8      | 1479287120875 | 11frnYASv                                    |
| 4 | Amount                  | Long  | 41       | 8      | 1             | 11111112                                     |
| 5 | Fee                     | Long  | 49       | 8      | 1             | 11111112                                     |
| 6 | Recipient's address     | Bytes | 57       | 26     | ...           | 3NBVqYXrapgJP9atQccdBPAgJPwHDKkh6A8          |
| 7 | Attachment's length (N) | Short | 83       | 2      | 4             | 15                                           |
| 8 | Attachment's bytes      | Bytes | 85       | N      | \[1,2,3,4]    | 2VfUX                                        |

_**Total data bytes for sign:**_

`Ht7FtLJBrnukwWtywum4o1PbQSNyDWMgb4nXR5ZkV78krj9qVt17jz74XYSrKSTQe6wXuPdt3aCvmnF5hfjhnd1gyij36hN1zSDaiDg3TFi7c7RbXTHDDUbRgGajXci8PJB3iJM1tZvh8AL5wD4o4DCo1VJoKk2PUWX3cUydB7brxWGUxC6mPxKMdXefXwHeB4khwugbvcsPgk8F6YB`

_**Signature of transaction data bytes**_ **(one of an infinite number of valid signatures):**

`2mQvQFLQYJBe9ezj7YnAQFq7k9MxZstkrbcSKpLzv7vTxUfnbvWMUyyhJAc1u3vhkLqzQphKDecHcutUrhrHt22D`

_**Total transaction bytes with signature:**_

`6zY3LYmrh981Qbzj7SRLQ2FP9EmXFpUTX9cA7bD5b7VSGmtoWxfpCrP4y5NPGou7XDYHx5oASPsUzB92aj3623SUpvc1xaaPjfLn6dCPVEa6SPjTbwvmDwMT8UVoAfdMwb7t4okLcURcZCFugf2Wc9tBGbVu7mgznLGLxooYiJmRQSeAACN8jYZVnUuXv4V7jrDJVXTFNCz1mYevnpA5RXAoehPRXKiBPJLnvVmV2Wae2TCNvweHGgknioZU6ZaixSCxM1YzY24Prv9qThszohojaWq4cRuRHwMAA5VUBvUs`

### Key type

By default, transactions are signed using the ED25519 algorithm. However, LTO supports multiple algorithms and curves like secp256k1, NIST P-256, and RSA. When broadcasting a transaction, it's required to include the [key type](../../accounts.md#key-types) in addition to the sender's public key.

{% hint style="warning" %}
RSA public keys are too large to store for each request. For RSA, the sender public key field must contain the SHA256 hash of the public key. This means that the transaction can't be validated by itself. RSA is only available through a [smart account](set-script.md) or by publishing an X.509 certificate to the public chain.
{% endhint %}

### Proofs

Proofs are a flexible way to authorize a transaction. Each proof is a Base58 encoded byte string and can be a signature, a secret, or anything else – the semantics of the proof is dictated by the smart contract that interprets it. There can be up to 8 proofs at most 64 bytes each.

By default, only one proof is used, which must be the transaction signature by the sender. It should be the very first element in the proofs array, while all the other elements are ignored. The JSON looks like

`"proofs": [ "21jgWvYq6XZuke2bLE8bQEbdXJEk..." ]`

## Transaction id

The transaction id is not stored in the transaction bytes. It's calculated from the binary message for signing as `blake2b256(binary_message)`.
