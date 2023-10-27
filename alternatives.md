## Summary of did:web improving did methods

This section introduces existing proposals that aim to solve some of the
limitations of `did:web`.

### did:webs

The `did:webs` method [3] aims to benefit from the discoverability of `did:web`
while providing a separate trust anchor based on KERI (_Key Event Receipt
Infrastructure_) [5].

`did:webs` identifiers follow a structure similar to `did:web`, with an
additional _Autonomic Identifier_ (AID) obtained from KERI appended at the end,
for example: `did:webs:example.com:some:path:12124313423525`

In KERI, an inception event creates the initial key pair that sets the root of
trust, and the _Autonomous Identifier_ (AID) is derived from the inception
event's hash. The AID is self-certifying and becomes the first item in an
append-only chain of events known as the KEL (_Key Event Log_). The KEL provides
a secure mechanism to perform updates in the DID document that are chained
together and can be validated against the inception event that is encoded in the
DID itself.

(Add a diagram)

Like `did:web`, `did:webs` uses the HTTPS protocol to provide access to DID
documents, with URLs formed by first encoding the domain name and path in the
form `https://domain.tld/some/path/aid`, then appending `/did.json` to obtain
the latest version of the DID document. Additionally, `did:webs` publishes the
entire stream of KEL events on a separate URL (`/keri.cesr`), making it possible
for DID resolvers to reconstruct and validate the content of the DID document at
any point in time.

Since the AID represents the inception event tied to the subject's identity, the
AID together with the KEL are suficient to generate the DID document associated
with the subject, independently from the `did:webs`'s DID itself. This property
makes it possible to, for example, migrate a `did:webs` to another web domain,
or even to another DID method by using the AID as the unique identifier and the
KERI event stream for validation. From this perspective, `did:webs` could be
seen as a method for exposing a set of KERI mechanisms via HTTPS.

(Briefly mention additional features: whois folder, Witnesses, Signed files,
security, ...)

### did:webplus

The `did:webplus` method [4] augments `did:web` by maintaining an immutable and
auditable history of DID document versions.

This is realized by implementing a microledger in which the signature of the
initial document is incorporated into the DID itself, with each subsequent
document referencing the signature of its predecessor. Documents also contain
additional attributes, including a monotonically increasing _version number_ and
a _start of validity_ timestamp, which effectively establish a totally ordered
sequence of DID documents with non-overlapping periods of validity.

Identifiers in `did:webplus` are similar to the ones in `did:web`, with an
additional field corresponding to the self-signature of the initial document,
for example:
`did:webplus:example.com:0B2LYBZ06Bn0dq7ALo3kG5ie20sQKvv7yzmbA8KtKExC4PRiZ2io-hPxxOy-mQ2qb4yuGdAK0eKvipqcBlZSArDg`.

When the DID controller produces a signature, the DID URL specifying the signing
key must include the standard query parameters `versionId`, `versionTime` and
`hl` [6]. This uniquely identifies the document version within the ledger.

To our knowledge, a draft method specification of `did:webplus` has not yet been
published, but an overview of the method and a prototype implementation are
available [4].

#### Embedding self-signatures

To uniquely identify and link document versions, a self-signature of the
document content is computed and embedded in the document itself. This creates a
circular dependency problem: the signature should only be computed once the
content will not be further modified, so it is not possible to then modify the
document to include the signature inside it.

`did:webplus` solves this problem by reserving "slots" in the document that are
filled with zeroes. The signature is then generated on this data, and the zeroes
are then replaced with the computed signature to build the final self-signed
structure. During the signature verification process the inverse operation is
performed, extracting the signature first, then filling the slots with zeroes
and computing the signature.

(More features: authorization)

:::danger

### DID Web 2.0

We will state that the ideas from the did web 2.0 advance reading paper are
pretty implemented in in did:webplus and obviously webplus is much further in
terms of spelling things out, so we focus on this one. :::
