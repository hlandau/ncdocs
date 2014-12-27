Introduction to Namecoin Deployment Types
=========================================

The greatest challenge facing Namecoin's adoption is long tail deployment.
While not the sole application for Namecoin, this document focuses on the
Namecoin domain name system. Effective adoption of this system requires that
people be able to access the `.bit` namespace, ideally with minimal technical
skill. Software packages such as the Tor Browser Bundle are demonstrative in
their ease of use, but infrastructural or operating system-based deployments
are also useful to consider.

The Full Node is the centrepiece of Namecoin infrastructure and needs no
introduction. It provides the greatest security and privacy Namecoin is able to
offer. It is vulnerable to 51% attacks, but may also be vulnerable to Sybil
attacks if those attacks are 100% effective and ensure a target does not
connect to a single legitimate node even for a brief time.

All other proposed depoloyment modes are lighter in their network, storage and
computational footprint, and their vulnerabilities are always a superset of
those of a full node.

The modes are ordered in ascending order of security.

TS: Trusted Server
------------------

In the Trusted Server model, a client simply makes a secure connection to a
known trusted party (the "trustee"), say with TLS, which answers name queries.
The client does not need to store any data.

The vulnerabilities of a full node are assigned to the trustee rather than the
client. This may be advantageous if for example a trustee can take measures to
reduce the likelihood of a successful Sybil attack, which most full node users
will not.

However the further vulnerabilities created by the TS model are severe and, by
relying on a central party, destroy the advantages offered by Namecoin.

Specifically, an adversary who is the trustee or is capable of acting as the
trustee, or any adversary who is capable of stealing the private keys of the
trustee, can return arbitrary data for name queries or deny the existence of
extant names, as well as logging queries which are made, compromising the
privacy of users.

SPV36: Minimal Mode
-------------------

In minimal mode, the last 36,000 block headers are downloaded and verified.
This is similar to Bitcoin's SPV mode but since any non-expired name must have
at least one name transaction in the last 36,000 blocks, there is no need for a
client to concern itself with the contents of blocks with a depth greater than
36,000.

The client does not store transactions and must rely on the network to provide
name transactions on demand. While the existing Bitcoin protocol supports SPV,
SPV for name lookup would require a new "get transaction by name" wire protocol
command.

This command would return the most recent name transaction for a name, along
with a merkle branch proving its inclusion in a block, and block-identifying
information. This information could be a block header, block hash or block height,
or all three to maximize flexibility in client implementations.

This mode inherits all of the vulnerabilities of a full node. However it is
vulnerable to forged denial of existence attacks (the node queried can falsely
deny the existence of a name) and replay of non-expired values (a client has no
way of knowing that the transaction returned is current or not, only that it
is not expired). It is also vulnerable to privacy attacks, as the node queried
can see all names accessed; this can be reliably exploited by anyone who has
the power to wiretap "anywhere", or by anyone who can Sybil their target node.

The rate of change of data stored is zero, as block headers are constant in
size. Merged mining means that block headers are not constant in size, though
they are small; however, auxpow data need not necessarily be stored, only
verified on reception.

The size of 36,000 headers without auxpow data is about 2.75 MiB.

This mode could optionally be reinforced by using it in conjunction with a
trusted server, either exclusively or non-exclusively (to try and mitigate
Sybil attacks).

SPV36+UTXO CB: Reinforcing Minimal Mode
---------------------------------------

Minimal mode could be reinforced by certain proposed extensions to Namecoin.
However there are unsolved problems regarding the efficient construction of the
necessary data structures.

The first desirable reinforcement is Unspent Transaction Output Tree/Coinbase
(UTXO CB). In this, a cryptographic tree data structure is deterministically
computed over the unspent transaction output tree, and the resulting root hash
is placed in the coinbase transaction of the next block. Once this is adopted
by a high percentage of miners, it can become mandatory and the correctness of
the root hash can be enforced by root nodes.

The cryptographic data structure is designed to enable the efficient conveyence
of a proof of inclusion for a given unspent transaction, for example via Merkle
branches. (Since expired and updated names are marked as spent, proving that a
name transaction is not expired at a given height is proof of its validity at
that height.)

A new wire protocol command is added that allows clients to request a name
transaction by name. The data returned is a superset of that returned by the
Minimal Mode query command; not only does it return the name transaction and
the Merkle branch proving inclusion in a specific block, but also a Merkle
branch (or other proof of inclusion) proving that that transaction is in the
UTXO tree expressed by a given UTXO tree root hash. It also returns the
coinbase transaction containing that root hash and a Merkle branch proving the
inclusion of that transaction in a recent block.

This allows a client to quickly and efficiently verify that a name transaction
is current and valid. Since this is a variant of SPV, the client trusts the
UTXO root hash contained in the block at a given depth.

However there are some unsolved issues regarding this mode. It is necessary for
full nodes to be able to efficiently update the cryptographic data structure
expressing the UTXO set as new unspent transactions are created and old ones
are spent. Further, since different clients may have different depth
thresholds, nodes must be able to efficiently generate proofs of inclusion for
the UTXO tree valid at any given height. These are both issues not solved by
standard Merkle trees.

The latter problem could be solved by standardising on a specific depth value
(e.g. 12), or a small number of depth values from which clients must choose.

This mode can be referred to as "SPV36+UTXO CB". It inherits the
vulnerabilities of full nodes, and the privacy vulnerablities of minimal mode.
Like minimal mode, it is vulnerable to forged denial of existence. However,
non-current name transactions cannot be replayed.

Like Minimal Mode, further reinforcement by the exclusive or non-exclusive use
of a trusted server is possible ("SPV36+UTXO CB+TS").

SPV36+UTXO CB+NX CB: Preventing Forged Denial of Existence
----------------------------------------------------------

Forged denial of existence can be prevented by creating a cryptographic tree of
the keys of all current names and placing the root hash of that tree in every
coinbase transaction. In this regard it is very similar to UTXO CB, but is
structured by key rather than by transaction. Nodes could be required to
provide a Merkle branch or similar proof of inclusion proving the existence of
adjacent names in the tree, and thereby proving that no name exists between
those names. This allows a node to prove that a name does not exist, and allows
clients to verify this.

This mode adds to UTXO CB and can be referred to as "SPV36+UTXO CB+NX CB" and
prevents forged denial of existence. However deployment of this reinforcement
should probably be considered low priority until such time that forged denial
of existence becomes a problem.

SPV36+UTXO: Hybrid Modes
------------------------

A variety of hybrid modes are available. These use more storage, but may be
easier to implement and provide reasonable security.

In "SPV36+UTXO", 36k headers are stored, but all name transactions in the UTXO
set are also stored. Each name transaction is received along with proof of
inclusion for a block. In addition to the vulnerabilities of a full node, this
mode renders the client vulnerable to Sybil attacks which selectively prevent
the client from learning about new name transactions. This allows updates to
existing names and registration of new names to be suppressed selectively.

The name transactions stored can be a subset of the entire name database. For
example, a client could choose to store only names in d/ with valid values.

A variant of this is "SPV36+UTXO FBR" (Full Block Receive). In this mode, only
the UTXO set or a subset is stored, but full blocks are still received. The
vulnerabilities of this mode are equivalent to FN36, described below.

FN36: Full Node 36k
-------------------

To come full circle, a full node can be made to store only the last 36,000
blocks. This reduces the data which must be stored without compromising the
security of the full node in any name-relevant way.
