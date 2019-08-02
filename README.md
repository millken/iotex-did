# IoTeX DID Method Specification
### First Draft 1 Aug 2019

A DID is an idenetfier which allow you to look up your own DID documentation on our IoTeX blockchain. This document specifies the IoTeX
[DID Method](https://w3c-ccg.github.io/did-spec/#specific-did-method-schemes) [`did:iotex`].

## Method Name

We use the `iotex` to be our method name and a formal DID using this method need begin with following prefix: `did:iotex` . Furthermore, all the chaaracter in the prefix need be in lowercase. The string after this prefix is `idString` part and its generated algorithm will be mentioned below in the section on [Specific Identifiers](#specific_identifiers).

## Specific Identifiers

IoTeX DIDs conform with [the Generic DID Scheme](https://w3c-ccg.github.io/did-spec/#the-generic-did-scheme) which generally defined the schema of DID design. The specific iotex-idstring is described below in [ABNF](https://tools.ietf.org/html/rfc5234):

```

iotex-did 	   = "did:iotex" iotex-idString

iotex-idString = 28*31(base58char)
base58char     = "1" / "2" / "3" / "4" / "5" / "6" / "7" / "8" / "9" / "A" / "B" / "C"
                          / "D" / "E" / "F" / "G" / "H" / "J" / "K" / "L" / "M" / "N" / "P" / "Q"
                          / "R" / "S" / "T" / "U" / "V" / "W" / "X" / "Y" / "Z" / "a" / "b" / "c"
                          / "d" / "e" / "f" / "g" / "h" / "i" / "j" / "k" / "m" / "n" / "o" / "p"
                          / "q" / "r" / "s" / "t" / "u" / "v" / "w" / "x" / "y" / "z"

```

### Generating a unique idstring

Every device need to register their own DID by upload their certificate which contains a unique public key. Then a unique `idstring` is created as follows by using this public key:

1.  Hash the public key which is uploaded by devices using the following hash function [multihash](https://multiformats.io/multihash/).
2.  Using the [multihash prefix](https://github.com/multiformats/multicodec/blob/master/table.csv) hash function name to be our prefix and concatente the hash function name and hash value generated by previous step.
3.  [Base58](https://en.wikipedia.org/wiki/Base58) encoded the idString we generated before.

The following example code in golang illustrates this process of generating a unique `idString`:

```go
package main

import (
	"crypto/rand"
	"encoding/hex"
	"fmt"

	"golang.org/x/crypto/ed25519"
	"golang.org/x/crypto/sha3"

	"github.com/mr-tron/base58"
	"github.com/multiformats/go-multihash"
	"github.com/ockam-network/did"
)

func main(pbKey) {
	// hash the public key
	pbHash := sha3.Sum256(pbKey)
	idString := pbHash[:]
	idString = pbHash[len(idString)-20:]
	// prepend the multihash label for the hash algo, skip the varint length of the multihash, since that is fixed to 20
	idString = append([]byte{multihash.SHA3_256}, idString...)
	// base58 encode the above value
	id := base58.Encode(idString)
	d := &did.DID{Method: "iotex", ID: id}
	// we got our DID in d variable
}
```

### Example

An example IoTeX DID:

```
did:iotex:2MpPfHH14dhLbbDV8Va1SPJrCWZNf
```

## DID Documentation Resolution

### Create/Register

IoTeX clients can register an new entity such as device on our blockchain by submitting a certificate as a transactin.

#### Request example

Here is a example for such a transaction:

```
{
  "header": {
    "operation": "create",
    "alg": "ES256K"
  },
  "certificate": "Encoded certificate"
}
```
Then a DID documentation will be created on our blockchain like this example:

```
{
  "@context": "",
  "id": "did:iotex:2MpPfHH14dhLbbDV8Va1SPJrCWZNf",
  "publicKey": [{
    "id": "#key1",
    "type": "Secp256k1VerificationKey2018",
    "publicKeyHex": "029a4774d543094deaf342663ae672728e12f03b3b6d9816b0b79995fade0fab23"
  }],
}
```