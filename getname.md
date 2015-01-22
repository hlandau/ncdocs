getname
=======

This document specifies a Namecoin wire protocol command to allow SPV clients
to query name information over the network.

Three new message types are defined, `getname`, `authtx` and `namenotfound`.

This document is released into the public domain.

getname
-------

The `getname` message payload requests that the receiving peer respond with
information about a name. It shall have the following form:

    uint32_t   Flags (reserved; must be sent as zero and ignored on receive)
    var_str    Name

Implementations shall not transmit any trailing data, but any such data shall
be ignored by receivers.

nametx
------

The `nametx` message payload is sent in response to a `getname` message if the
name is found to exist and not be expired. It is never sent unsolicited. It
shall have the following form:

    uint32_t       Flags (reserved; must be sent as zero and ignored on receive)
    var_str        Name (for ease of correlation with request)
    tx             Unexpired unspent transaction containing name operation
    merkle_branch  Merkle branch proving inclusion of transaction in block (uses auxpow Merkle branch format)
    block_header   Block header
    auxpow_header  Block auxpow (if applicable)
    var_int        Block height

The message shall be transmitted with no trailing data. Receivers shall ignore
trailing data if it is present.

namenotfound
------------

The `namenotfound` message payload is sent in response to a `getname` message
if the name is found to not exist or be expired. It is never sent unsolicited.
It shall have the following form:

    uint32_t      Flags (reserved; must be sent as zero and ignored on receive)
    var_str       Name (for correlation with request)

Protocol
--------

A SPV lightweight resolver shall resolve a name by choosing a peer and issuing
a `getname` wire protocol command to that peer for the given name. The peer, if
compliant with this specification, shall respond with either an `authtx`
message (if a non-expired name with the given key was found) or a
`namenotfound` message (if the name is expired or does not exist).

Resolvers MUST cache returned results (including non-existence results) at
least until another block is mined.

The flags field allows for future extension, so that additional proof fields
can be solicited and returned as future lightweight verification technologies
are deployed.

Allocation of Service Bit
-------------------------
Bit 63 (1<<63) of the services field in the version message of the Namecoin
wire protocol is allocated as follows:

    NODE_NAMEINFO    This node can process getname queries.

Nodes shall set this bit if and only if they are conformant with this
specification. This allows resolvers to rapidly identify peers which can assist
them. The bit is allocated from the opposite end of the services field to
minimize the chance of collision with future Bitcoin protocol developments.
