Namecoin: Lightweight Resolution Roadmap
----------------------------------------

Currently, the only options for people wanting to resolve Namecoin names are to deploy a full node or to trust a full node operated by another person. However, there are viable means for the deployment of trustless lightweight resolvers which require only a minimal data set to be stored locally.

Namecoin <a href=https://en.bitcoin.it/wiki/Thin_Client_Security#Simplified_Payment_Verification_.28SPV.29>SPV</a> clients only need to store block headers for names that have not expired. Clients designed for domain names currently only need to store the past 36,000 blocks, which requires only a few megabytes.  We call this mode SPV-Current, or SPV-C for short.

To support SPV-C, the Namecoin wire protocol must be extended so that nodes can query for transactions by name. Responses can use Merkle branches to prove that the transaction was included in a block which the SPV-C client is aware of.

This basic implementation of SPV-C is vulnerable to a malicious peer responding with an older (but non-expired) transaction in response to a query.  However, a hash tree can be used to concisely represent the identity of a set (such as the set of all names) and a concise proof of inclusion (i.e. a Merkle branch) can be generated that proves the inclusion of a given item in that set. That proof of inclusion can be verified using only the root hash of the data structure.

By having miners include this root hash in the coinbase transaction of every block, the identity of the set of all current names is attested to continuously throughout the mining process. Once all miners adopt this modification to Namecoin, it can become a network rule; the inclusion and correctness of the root hash is then enforced by all miners.

We call this Unspent Transaction Output Tree/Coinbase attestation (UTXO-CB).  Since name operations are marked spent when they expire, the fact of a name transaction being unspent is proof that it is current.

Lightweight clients can then keep track of the current root hash (more accurately, the root hash of the 12th deepest block, as keeping the UTXO sets current at all blocks in memory separately is infeasible). Peers can generate concise proofs of inclusion in response to client queries. Thus, when a client queries for a name transaction by name, a peer responds not only with the name transaction and proof of inclusion in a given block, but also with a proof of inclusion of the name transaction in the set of current names as attested to by the root hash.

UTXO-CB constitutes a significant reinforcement to SPV-C clients, as it prevents peers from returning anything but current name data. This mode is referred to as 'SPV-C+UTXO-CB'.

However, deployment of this mode is made difficult by a number of unsolved problems in relation to the formulation of the cryptographic data structure used for representing the UTXO set. Similar UTXO coinbase attestation proposals have been made for Bitcoin, so some existing work (such as the <a href="https://github.com/maaku/bips/blob/master/drafts/auth-trie.mediawiki">Authenticated Prefix Trees BIP</a>) may be applicable.

The most fundamental of these issues is that since new transactions occur continuously, it is important that miners be able to efficiently compute updates to the UTXO set. In other words, it must not be necessary for the entire cryptographic data structure to be recomputed when a new transaction is added to the set. Of course, any solution must preserve the ability to concisely represent the identity of the set and the ability to generate concise proofs of inclusion.

The final security issue to be challenged is authenticated denial of existence. This can be solved using coinbase attestation of another set, the set of all adjacent keys, in sorted order. This tree must be formulated such that proofs of adjacency can be generated. This will allow peers to prove that no unexpired keys exist which are lexicographically between two given names. This directly comparable DNSSEC's method of authenticated denial of existence by the same method; by a sorted sequence of signed records stating that no names exist between two given names.  This mode is referred to as SPV36+UTXO CB+NX CB.

Note that the SPV client can be improved incrementally, as attacks on the basic SPV design require an attacker to control 100% of the peers connected to the target and even then it cannot arbitrarily forge data.  Bootstrapping nodes can also attempt to provide a diverse selection of nodes to clients, increasing the attack cost.  However, UTXO CN and NX CB reduce the amount of network traffic, reduce latency, and increase security, so they should be implemented eventually.

Hopefully this post has illustrated how both secure and lightweight resolution of Namecoin is not only possible but can become practical. This is in substantial constrast to projects like DNSChain, which introduces the most regressive form of third party trust and places unrealistic expectations on the majority of users (see Indolering's post for details on the <a href="http://www.indolering.com/dnschain-is-harmful">issues with DNSChain</a>). Only secure lightweight resolution, as described above, can make Namecoin domains robust against MITM attacks perpetrated by state actors.
