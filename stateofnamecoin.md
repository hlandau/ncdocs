<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->
**Table of Contents**

- [State of Namecoin](#state-of-namecoin)
  - [1. Node Types; Strategies for Deployment](#1-node-types-strategies-for-deployment)
    - [Charter of Vulnerabilities](#charter-of-vulnerabilities)
    - [Full Node](#full-node)
    - [SPV](#spv)
    - [SPV+UTXO CB](#spvutxo-cb)
    - [SPV+UTXO CB+NX CB](#spvutxo-cbnx-cb)
    - [SPV+UTXO](#spvutxo)
    - [TS](#ts)
  - [2. Full Node Infrastructure](#2-full-node-infrastructure)
    - [Block Validation Rules](#block-validation-rules)
    - [Seeding Methods](#seeding-methods)
    - [Wire Protocol](#wire-protocol)
  - [3. Primary Access Modes](#3-primary-access-modes)
    - [Local Recursive-Passthrough DNS Server, Node](#local-recursive-passthrough-dns-server-node)
    - [Local Authoritative DNS Server, Unbound, Node](#local-authoritative-dns-server-unbound-node)
    - [Local Proxy-based solutions, browser extensions...](#local-proxy-based-solutions-browser-extensions)
  - [4. Auxillary Access Modes](#4-auxillary-access-modes)
    - [Central Authoritative DNS Service](#central-authoritative-dns-service)
    - [Central Recursive DNS Service](#central-recursive-dns-service)
    - [Central DNS Suffix](#central-dns-suffix)
    - [Central Proxy](#central-proxy)
  - [5. Strategies for Deployment](#5-strategies-for-deployment)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

State of Namecoin
=================

1. Node Types; Strategies for Deployment
----------------------------------------

The Namecoin ecosystem can be characterised by the different node modes and
auxillary services supported.

The centre of the Namecoin ecosystem is the full node. The renovation of the
full node infrastructure in the form of namecore is thus of importance; see
section 2.

Wider deployment requires viable means of developing lightweight clients. The
following table lists all proposed node types:

Node Type              | Weight | Security | Storage | Vulnerabilities              | Data Stored         | Storage Rate of Change | Name Data Stored   | Deployability       |
---------------------- | ------ | -------- | ------- | ---------------------------- | ------------------- | ---------------------- | ------------------ | ------------------- |
FN                     | 6      | 5        | 6       | 51%,Sy(a)                    | Blocks              | Increases perpetually  | All                | Lowest              |
FN36                   | 5      | 5        | 5       | 51%,Sy(a)                    | 36k Blocks          | Based on tx volume     | All current        | Low                 |
SPV36+UTXO CB+NX CB+TS | 2      | 4        | 2       | 51%+T(a), T(p)               | 36k Headers         | Zero                   | None               | High, softfork, WPC |
SPV36+UTXO CB+NX CB    | 2      | 4        | 2       | 51%,Sy(a), W,Sy(p)           | 36k Headers         | Zero                   | None               | High, softfork, WPC |
SPV36+UTXO CB+TS       | 2      | 4        | 2       | 51%+T(a), T(b,p)             | 36k Headers         | Zero                   | None               | High, softfork, WPC |
SPV36+UTXO CB          | 2      | 4        | 2       | 51%,Sy(a), b, W,Sy(p)        | 36k Headers         | Zero                   | None               | High, softfork, WPC |
SPV36+UTXO FBR S       | 4      | 3        | 4       | 51%,Sy(a)                  * | 36k Headers+UTXO    | Based on name database | Some current       | Medium              |
SPV36+UTXO S           | 4      | 3        | 4       | 51%,Sy(a), Sy(b,c)         * | 36k Headers+UTXO    | Based on name database | Some current       | Medium              |
SPV36+UTXO FBR         | 4      | 3        | 4       | 51%,Sy(a)                    | 36k Headers+UTXO    | Based on name database | All current        | Medium              |
SPV36+UTXO             | 4      | 3        | 4       | 51%,Sy(a), Sy(b,c)           | 36k Headers+UTXO    | Based on name database | All current        | Medium              |
SPV36                  | 2      | 2        | 2       | 51%,Sy(a), b, c, W,Sy(p)     | 36k Headers         | Zero                   | None               | High, WPC           |
TS                     | 1      | 1        | 1       | T(a, b, c, d, p)             | None                | Zero                   | None               | Highest             |

    * Vulnerability list accurate only for queries for names included in selection criteria.

<!--
SPV+UTXO CB+NX CB+TS   | 3      | 4        | 3       | 51%+T(a), T(p)           | Headers             | Linearly w.r.t. time   | None               | High, softfork, WPC |
SPV+UTXO CB+NX CB      | 3      | 4        | 3       | 51%,Sy(a), W,Sy(p)       | Headers             | Linearly w.r.t. time   | None               | High, softfork, WPC |
SPV+UTXO CB+TS         | 3      | 4        | 3       | 51%+T(a), T(b,p)         | Headers             | Linearly w.r.t. time   | None               | High, softfork, WPC |
SPV+UTXO CB            | 3      | 4        | 3       | 51%,Sy(a), b, W,Sy(p)    | Headers             | Linearly w.r.t. time   | None               | High, softfork, WPC |
SPV+UTXO               | 4      | 3        | 4       | ?                        | Headers+UTXO        | Based on name database | All                | Medium              |
-->

1=Low, 5=High

Weight, Storage: Lower is better

Security: Higher is better

FN: Full Node

SN: Selective Node

SPV: Simplified Payment Verification

UTXO: Unspent Transaction Output Tree

S: Selective

FBR: Full Block Receive

CB: Coinbase attestation via Merkle root (enforced by softfork)

NX: Non-expired names tree

TS: Trusted server (e.g. with TLS)

WPC: Would require new wire protocol commands

### Charter of Vulnerabilities

The chart above references vulnerabilities (i.e., bad name-related things that
can happen) by letter. They are as follows:

  - a. FNV     - Forge Name Value/Steal Name (implies b, c, d)
  - b. FDoE    - Forge Denial of Existence
  - c. RNXV    - Replay Non-Expired Value
  - d. RXV     - Replay Expired Value
  - p. Privacy - Can see names accessed

It is useful to describe vulnerabilities in terms of the prerequisites for their exploitation.
The following vulnerability condition codes are used:

  - 51% - Adversary is able to perform 51% attack
  - T - Adversary is the trustee in the TS node type
  - W - Adversary can wiretap everywhere
  - Sy - Adversary can perform Sybil attacks or MitM everywhere
  - S - For names in selection criteria only

Vulnerabilities are notated in the form Conditions(Vulnerabilities).
A comma in the Conditions indicates 'or' and a plus indicates 'and'.
For example, `A,B(a,b)` means that any adversary meeting condition A or B can
engage in a and/or b; `A+B(a,b)` means that any adversary meeting conditions A
and B can engage in a and/or b.

Some common adversaries include:

Adversary | MitM  | Compute | Implies          |
--------- | ----- | ------- | ---------------- |
Agency    | Y     | Y       | W,Sy, probably T |
Network   | Y     | N       | W,Sy             |
Botnet    | Sybil | N       | Sy               |
User      | N     | N       |                  |
Trustee   | -     | -       | T                |

### Full Node

For reference, the vulnerabilities of a full node are considered.

The vulnerabilities are `51%,Sy(a)`. The reasoning is as follows:

51%: If an adversary controls more than 50% of the hashpower of the network, the chain
they build will end up winning any highest-block race as time tends toward
infinity. By excluding name renewal transactions, the adversary can force a
name to expire, then register it itself and thus committing a.

The attacker would have to mine up to 36,000 blocks, depending on how long the
name has left to expire. Since difficulty adjusts automatically, the attack
would take about 8 months in the 'worst' case and would have to be sustained for
that time.

Sybil: If an adversary can wholly 'surround' a full node with nodes it
controls, it can deliver to the victim node only blocks that it mines. After
mining enough blocks, target names will expire and can be registered, thus
committing a.

Assuming the attacker does not also possess over 50% of hashpower, they will
mine blocks at a slower rate, based on their hashpower. The difficulty will
eventually be retargeted based on the reduction in block rate, eventually
allowing the attacker to mine blocks at the same rate as the network. However
at this point the network will have the lead. Thus successful communication
with the true Namecoin network foils this attack, even if such communication
is only brief.

Since these vulnerabilities require a sustained attack over many months they
may be considered quite theoretical. If they ever become problematic, solutions
include:

  - Operating offical, TLS-secured nodes and configuring Namecore to always
    connect to at least one. (Imposes the T vulnerability condition.)

  - Having “trusted” mining pools generate a signing key to be signed by a
    Namecoin key hardcoded in all full nodes. Have full nodes reject any block
    if it means that none of the last N blocks are from endorsed miners. (This
    scheme is complex and is broken by the compromise of any mining pool's
    key.)

### SPV

An SPV client verifies block depth rather than block height. Thus a SPV client
trusts any block which can be proven to possess a certain depth (i.e., to have
a number of valid blocks following on from it).

A Namecoin SPV client necessitates new wire protocol commands where Bitcoin
does not, as it must be able to look up transactions by name. The command
would look something like:

  gettxbyname(name) -> tx, block header, block height, merkle branch proof of inclusion
                    or (unauthenticated denial of existence)

Like a full node, SPV is vulnerable to `51%,Sy(a)`. However, SPV is also
vulnerable to `b, c, W,Sy(p)`; the command above does not preclude forged
denial of existence or replay of non-expired values. It also allows attackers
to reliably see names accessed if they can wiretap anywhere/everywhere or sybil
a victim. As always, these risks could be reduced mitigated by tacking on TS.
However, note that it is trivial to verify that a transaction falls within the
last 36,000 blocks as SPV clients do store headers; thus expired values at
least cannot be replayed.

A variant of SPV is to store only the last 36,000 headers ("SPV36"). This does
not appear to affect the security of clients performing name lookups. All
further mentions of SPV in this document implicitly include SPV36 except where
otherwise specified.

### SPV+UTXO CB

In SPV+UTXO CB, miners are upgraded to calculate a cryptographic tree data
structure expressing the Unspent Transaction Output Set in a deterministic
fashion (e.g. a Merkle tree). The root hash is placed in the coinbase
transaction along with a magic number to make it easy to find. All other nodes
verify that the included root hash is correct and reject nodes with incorrect
values. Thus the depth of a block becomes a potentially reliable indicator of
the consensus of the network as to the validity of the root hash contained
within.

The root hash is useful because the underlying cryptographic data structure
allows the generation of a succinct proof-of-inclusion (e.g. a Merkle branch).

A new wire protocol command would be required, something like:

  gettxbyname(name) -> tx, block header, block height,
                       merkle branch proof of inclusion in block,
                       proof of inclusion in UTXO tree,
                       coinbase transaction containing root used,
                       merkle branch proof of inclusion of coinbase in block,
                       block header for that block, height of that block

This is a superset of the wire protocol functionality required for
SPV described above; thus one additional command can be used to enable both modes.

There are unsolved problems regarding the deployment of UTXO CB:

  - A Merkle tree can be efficiently recomputed when a value is appended to the
    list. But inserting or removing a value at arbitrary points in the list in
    general will require full recomputation.

    How can a cryptographic hash tree data structure be designed such that
    items can be efficiently removed from anywhere within it (i.e. transactions
    which have just been spent)?
    
    The ability to generate succinct proofs of inclusion must be preserved.

  - Different lightweight clients may want to use different threshold depths at
    which to trust blocks. This in turn means that different UTXO tree root
    hashes and corresponding UTXO trees must be used to generate proof of
    inclusion. Unless a data structure can be found which enables full nodes
    to efficiently generate proofs of inclusion for a wide number of related
    but distinct UTXO trees, this essentially requires full nodes to keep UTXO
    trees in memory for any depth at which a query may be issued.

    This could be solved by supporting only a specific depth threshold value.

Instead of an Unexpired Transaction Output (UTXO) set, an Unexpired Name Output (UXNO)
set could be used. However, name transactions are marked as spent once they expire,
so the UTXO set contains only current name transactions (as well as non-name transactions).
Thus there is no need for a separate UXNO tree.

### SPV+UTXO CB+NX CB

NX CB further reinforces the network by creating a list of keys sorted by key
and placing a root hash of some cryptographic data structure calculated over it
in the coinbase. As with UTXO CB, this is verified by all nodes, and is useful only
if proof-of-inclusion can be efficiently generated.

This allows the secure denial of existence of names by generating proofs of inclusion
for adjacent items in the tree in such a way that it is proven that no names exist
between them. This is similar to the strategy DNSSEC takes with NSEC/NSEC3.

This reinforcement technique thus mitigates vulnerability b for a number of
node types.

That this would be a useful addition only when deployed in conjunction to UTXO
CB. This scheme also inherits the unsolved problems posed by UTXO CB stated
above.

Implementation of NX CB is unlikely to be a priority unless exploitation of
vulnerability b becomes a problem.

### SPV+UTXO

TODO

### TS

Use of TS alone essentially discards all of the key advantages of Namecoin. As
such it is not an advisable deployment mode. However, it may be of some use
where website operators decide to make use of .bit due to threats posed by the
ICANN domain name system, but their users do not have high security
requirements and are unwilling to shoulder the corresponding inconveniences.

In such cases, successful deployment would require a suitable operating jurisdiction
and a minimum capacity to demonstrate a commitment to operational integrity, e.g.
by technical, procedural and organizational safeguards.

TS is a more likeable proposal when combined with other deployment modes, as
many trustee-based attacks are then mitigated, especially if TS is not used
exclusively but in combination with non-TS network-based connections.

In particular TS has the potential to provide privacy for name requests
where other deployment modes cannot, so long as the trustee is trustworthy,
which is at least an improvement.

2. Full Node Infrastructure
---------------------------

This section discusses the current state of Namecoin's full node infrastructure.

Currently, the predominant implementation of a Namecoin full node is namecoind,
which is based on Bitcoin Core 0.3. It has not tracked subsequent versions of
Bitcoin Core.

A replacement, `namecore` is under development. This is based on a current
version of Bitcoin Core adapted for Namecoin and will track Bitcoin Core.

### Block Validation Rules

Deployment of namecore will lead to changes in the chain validation rules
of Namecoin full nodes. One of the consequences of the lack of updates to
namecoind is that upgrades to Bitcoin orthogonal to the differences beteen
Bitcoin and Namecoin have not been adopted. For example, BIP 34 specifies a
new block format with new version number incorporating a height fiel.
BIP 34 activates automatically when it is used by a threshold number of
the last thousand blocks mined. Therefore, if 95% of Namecoin miners
switch to using namecore, all remaining users of namecoind will be unable
to validate new blocks.

BIP 30 specifies a validation rule to reject transactions with duplicate
transaction IDs; however there are a number of transactions in Namecoin
which violate this rule, though they all precede the latest checkpoint.

BIP 16, Pay to Script Hash is designed to activate automatically when
a threshold number of miners have upgraded to support it.

The infamous Bitcoin Core 0.8 accidental hardfork also demonstrates the
compatibility issues between pre-0.8 and 0.8 codebases.

### Seeding Methods

namecoind seeds via IRC, whereas namecore, being based on a modern Bitcoin Core
codebase, supports DNS seeds only. No DNS seed service is currently provided for
Namecoin. The move to namecore will necessitate hosting such a service.

Fortunately, open source software for hosting Bitcoin DNS seeds is available,
and has been found to work with the Namecoin network with minimal modification.
Thus a seed node need merely be set up and rendered official.

### Wire Protocol

The wire protocol has also changed in ways not tracked by namecoind. For example,
namecoind does not support `getheaders`. This may impede compatibility with
namecore, as namecore is likely to attempt to use `getheaders` commands with
namecoin nodes. Similar issues apply to bloom filtering commands, etc.

Alarmingly, there appears to be very little documentation as to the various
version numbers used by the Bitcoin wire protocol and the corresponding changes.

3. Primary Access Modes
-----------------------

### Local Recursive-Passthrough DNS Server, Node

The local machine is configured to use itself as a DNS resolver. Queries for
domains not under .bit are passed through to the real resolver, but queries
for domains under .bit are handled by the DNS server itself in collaboration
with a node running locally.

The canonical example of this access mode is NMControl.

### Local Authoritative DNS Server, Unbound, Node

The local machine is configured to use the Unbound DNS resolver (which is going
to end up being deployed widely on Linux anyway as a vehicle to DNSSEC validation).

A specially constructed DNS server authoritative only for .bit is run on
the local machine. Unbound is configured to divert requests for .bit to this
server. The server obtains Namecoin data from a node running locally.

### Local proxy-based solutions, browser extensions...

...

4. Auxillary Access Modes
-------------------------

The following low-security, trust-based deployment modes may be desirable
in some circumstances. Some are already in use.

### Central Authoritative DNS Service

A central trustee provides DNSSEC-enabled authoritative nameservers serving the
.bit zone. People can configure their machines to use these nameservers to
resolve .bit names. For example, the Unbound validating resolver can be
configured to use a specific authoritative nameserver for a specific domain
name using the `stub-zone` directive:

    server:
      ...
      trust-anchor-file: "/etc/unbound/keys/bit.key"
      stub-zone:
        name: bit.
        stub-addr: some.trusted.namecoin.nameserver.com.
        stub-prime: yes

### Central Recursive DNS Service

A central trustee provides a recursive name resolution service.

This deployment mode is likely to be problematic due to the current hazards
of operating open DNS resolvers, namely reflected DDoS attacks.

For use with DNSSEC it also still requires users to configure an explicit trust
anchor for .bit.

Failure of the resolver means a failure of all name resolution service, which
is functionally equivalent to a loss of connectivity for most users.

### Central DNS Suffix

A specially designed nameserver is run for a domain name such as `bit.tld`.
Users can access `.bit` websites by appending the TLD. The suffix operator
is necessarily trusted. A DNSSEC delegation can be obtained from the TLD.

A disadvantage of this mode is that it will not work with webservers and
nameservers which are not specifically configured to work with it. Thus
services provided via .bit must be designed in a “suffix-aware” manner.

### Central Proxy

This is similar to the DNS Suffix access mode, except that the solution is
not DNS-based. Instead, traffic is proxied by the operator.

This avoids the issues with websites and nameservers having to be
“suffix-aware”, but introduces legal issues (DMCA, etc.) and means the operator
is privy to any unencrypted information exchanged, even if the operator is not
acting maliciously.

`bit.pe` is a currently operational example of such a service.

5. Strategies for Deployment
----------------------------

...

