# Introduction to Namecoin for DNS Administrators

## Audience

This document provides an introduction to Namecoin for persons who are familiar
with DNS and the DNS directory model, but which have little or no familiarity
with cryptocurrency technologies such as Bitcoin or Namecoin.

## Introduction

Namecoin is a key-value store, mapping small string keys to small string or
binary values. Unlike a typical key-value store, Namecoin is distributed.

Distributed key-value stores can be categorized into two common categories of use:

  - **Global, public, zero-config designs.** These are designed to run across
    the global internet, with arbitrary latency and public membership. They are
    designed to be zero or near-zero configuration and require no maintenance.
    They are generally integrated into products for end-user use and their
    operation is seamless and transparent.

    The most common type of global, zero-config distributed key-value store is
    a Distributed Hash Table (DHT), and probably the most popular DHT in use today
    is the “Mainline” DHT used by BitTorrent.

  - **Datacentre designs.** These are designed for use in datacentres. They may not
    cope as well with arbitrary latency and require professional administration.

    An example of a datacentre-type distributed key-value store is Amazon Dynamo
    (including implementations such as Riak), or Google's Spanner.

Namecoin is a global, public, zero-config distributed key-value store. Anyone
can join the Namecoin network, and the store requires no maintenance. As such,
Distributed Hash Tables might be the existing technology closest to Namecoin in
terms of the manner in which it can be used and deployed.

However, Distributed Hash Tables cannot be effectively secured, and they
provide no guarantee that data will be stored for any particular length of
time. They are also vulnerable to attacks such as Sybil attacks. As such, while
DHTs are distributed key-value stores, they simply do not provide the
properties necessary for a directory service.

Namecoin is secured via a blockchain and proof of work, like Bitcoin, and closely
mirrors the Bitcoin codebase. However, you don't need to understand Bitcoin or
the principles of blockchains to use Namecoin or understand the properties it offers.

Here's Namecoin in a nutshell:

  - Namecoin is a **global, distributed, consistent, ordered append-only log**
    of name (key-value) registrations and updates. Each key-value pair is called
    a“name”.

    Currently, keys are limited to 255 bytes and values to 520 bytes.

  - **Registration.** Names are registered on a first-come, first-serve basis,
    just like traditional domain names. Once a name is registered, it can only
    be updated by the person who holds the private key for the name until it
    expires. There is no opportunity for hijacking, takedowns or procedural or
    legal attacks against the continued registration of names so long as the
    private key is kept private.

  - **Expiry.** Names expire every 36,000 blocks; that's about every 250 days.
    The last value of an expired name can still be viewed, but the name can
    then be reregistered by anyone else. The expiry counter is reset whenever a
    name's value is updated; if you don't wish to change the value of the name
    when renewing, you can renew a name simply by issing an update with the
    same value as is currently set.

    When you register a name, you make an entry in the append-only log at a
    given “block height”. This is simply the integer representing the block
    number containing the log entry. You should note this height because it is
    an easy way to track name expiry; if you register a name at height 320,147,
    your name will expire on block 356147 and should arrange to update the name
    (thereby renewing it) at most a few hundred blocks before its expiry time.

The Namecoin namespace is a single global namespace. It is subdivided into
a number of namespaces, which form prefixes for keys under those namespaces:

  - `d/`: The domain names namespace. This namespace allows domains to be
    registered under the `.bit` TLD.

  - `id/`: User identity namespace. This namespace allows users to register
    names and bind user information such as contact information or GPG key
    fingeprints to the name.

Keys are opaque; the `/` separator is not processed specially by Namecoin, it
is simply a convention. Anyone can create a new namespace, but the need for new
namespaces is rare and you should use an existing namespace where possible.

## Namecoin for Domain Names

Domain names manifest under the `.bit` TLD by convention, and are registered
under the `d/` namespace, in lowercase (Namecoin is case-sensitive). For
example, if you wanted to register the domain `example.bit`, you would register
the Namecoin name `d/example`.

The value of a `d/` name must be a JSON object conformant with the Namecoin
[Domain Names
specification](https://github.com/ifa-wg/proposals/blob/master/ifa-0001.md).
You can read the specification for the full details, but below are some quick
examples of common use cases.

Note that whenever domain names are referenced in values, they are relative to
the domain. If you need to specify a fully-qualified domain name, you must
terminate it with a `.` just like in a BIND zone file. Omitting a trailing `.`
is an easy mistake to make.

    // Point to a single IPv4 address. Translates to a single DNS A record.
    {
      "ip": "1.2.3.4"
    }

    // Point to multiple IPv6 addresses. Translates to multiple DNS AAAA records.
    {
      "ip6": ["::1", "::2"]
    }

    // Point to nameservers.
    {
      "ns": ["ns1.example.com.", "ns2.example.com."]
    }

    // Point to nameservers with glue. Also demonstrates how to configure
    // subdomains and use of relative names.
    {
      "ns": ["ns1", "ns2"],
      "map": {
        "ns1": {
          "ip": ["1.2.3.4"],
          "ip6": ["::1"]
        },
        "ns2": {
          "ip": ["1.2.3.4"],
          "ip6": ["::2"]
        }
      }
    }

Note that JSON forbids the use of trailing commas, single quotes to delimit
strings, JavaScript-style bareword key names (`{foo:"bar"}`), and comments.
Comments are shown above for clarity but must be removed when setting names in
Namecoin.

### Subdomains

Subdomains are expressed as follows:

    {
      // (record items, e.g. "ip")
      "map": {
        "subdomain-name": {
          // (record items, e.g. "ip")
        },
        "subdomain-name-2": {
          // (record items, e.g. "ip")
        }
      }
    }

The "map" structure can be nested as much as necessary. You cannot express deep
subdomains by placing '.' inside the subdomain name (`{"map": {"foo.bar": {}}}`
will not work).

### DNS Record Type Equivalence

The following table shows the equivalence between key-value items in the JSON
object set as the value of a `d/` name and DNS resource record types. Note that
arrays are often used to express multiple items of the same type. This is shown
below as `, ...`. (This is not valid JSON syntax and is present for
illustrative purposes only.)

<table>
<tr><th>DNS RRType</th><th><tt>d/</tt> Example</th><th>Notes</th></tr>
<tr><td>A</td><td><code>{"ip":["192.0.2.1", ...]}</code></td></tr>
<tr><td>AAAA</td><td><code>{"ip6":["::1", ...]}</code></td></tr>
<tr><td>NS</td><td><code>{"ns":["ns1.example.com.", ...]}</code></td></tr>
<tr><td>CNAME</td><td><code>{"alias":"example.com."}</code></td></tr>
<tr><td>DNAME</td><td><code>{"translate":"example.com."}</code></td></tr>
<tr><td>SRV</td><td><code>{"srv":[[Priority, Weight, Port, "www1.example.com."], ...]}</code></td><td>Priority, Weight and Port are the integers in the same order from the SRV record.</td></tr>
<tr><td>TLSA</td><td><code>{"tls":[[Usage, Selector, MatchType, "Base64-encoded Fingerprint"], ...]}</code></td><td>Usage, Selector and MatchType are the integers in the same order from the TLSA record. Hex-encoded fingerprint from TLSA record must be converted to base64.</td></tr>
<tr><td>TXT</td><td><code>{"txt": ["Some text record", ...]}</code></td></tr>
<tr><td>DS</td><td><code>{"ds":[[KeyTag, Algorithm, DigestType, "Base64-encoded Fingerprint"], ...]}</code></td><td>KeyTag, Algorithm and DigestType are the integers in the same order from the DS record. Hex-encoded fingerprint from DS record must be converted to base64.</td></tr>
<tr><td>SSHFP</td><td><code>{"sshfp":[[2,1,"Base64-encoded fingerprint"], ...]}</code></td><td>Algorithm Type, Fingerprint Type, Base64-encoded fingerprint string</td></tr>
<tr>
</table>

Where no syntax has been specified for a given DNS resource record type, you can use opaque items
to express such records:

    {
      "o": [[RRTypeNum, "Base64-encoded RData"], ...]
    }

The RRTypeNum is the integer value assigned to represent the RRType, and the
Base64-encoded RData is the base64-encoded raw contents of the resource
record's RData field. (Note: Because the value is presented in a context-free
fashion, name compression must not be used when forming the RData field.)

## How to Register Names

To register or update names, you will need to install Namecoin Core, the
preferred Namecoin node implementation. After installing Namecoin Core,
you will need to wait for it to synchronize with the network. Due to the
size of the Namecoin blockchain, this may take a considerable amount of time
and a substantial amount of data will be downloaded.

You communicate with the Namecoin Core daemon using the `namecoin-cli` command.
The `namecoin-cli` command interfaces with the JSON-RPC interface of the
Namecoin Core daemon. It accepts a number of subcommands. Of interest are:

  - `getinfo`: Retrieves information about the node's current synchronization
    status and the height of the newest block.

  - `name_show <key>` (e.g. `name_show d/example`): Shows information about a
    name, if it exists. If the name exists, it tells you the value of the name,
    in how many blocks time it expires, and whether it has already expired. You
    can use this to determine when your names will expire, and to determine
    whether a name is available for registration.

  - `name_new <key>`: Pre-registers a name for registration. You register a name
    by executing the `name_firstupdate` command, but you must first execute the
    `name_new` command to pre-register the name. (This step facilitates a
    cryptographic feature used to prevent other parties from preempting your
    attempt to register the name and obtaining it for themself.)

    After executing the command you will receive two values, a long hexadecimal
    value and a short hexadecimal value. **You should save both of these values
    as you may need them for the `name_firstupdate` step.**

    **You must wait** 12 blocks (about two hours) after executing `name_new`
    before the corresponding `name_firstupdate` command can be issued.

  - `name_firstupdate <key> <rand> <value>`: Finalizes your registration.
    You must wait 12 blocks (about two hours) after `name_new` before issuing
    this command. The `<value>` (for domain names) must be a valid JSON
    value, and the `<key>` must match that which was specified for `name_new`.

    The `<rand>` value is the shorter random hex value which was shown after
    executing `name_new`.

    The `name_firstupdate` command finalizes your registration, and all
    subsequent updates to the name's value must use `name_update`, not
    `name_firstupdate` (unless the name expires, in which case the
    `name_new`/`name_firstupdate` sequence must be repeated).

  - `name_update <key> <value>`: Updates the name given with the value
    given.

    There is also an optional third argument, for a Namecoin address to
    transfer the name to. If this is provided, the name is transferred to that
    address and only that address can update the name in the future.

Before you can register names, you must obtain enough Namecoin (NMC) to pay the
registration fee. You must transfer enough NMC to your node's Namecoin address.

Once you have registered a name, the name is bound to the private key stored in
your node's `wallet.dat` file. It is essential that you back up this file and
keep it secure. Loss of the file means loss of control of the name and its
inevitable expiry, and anyone with access to the file can update or transfer
the name.
