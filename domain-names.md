Namecoin: Domain Names
======================

This is a draft specification. It has no official endorsement by the Namecoin
project. Do not use it; instead, use the Domain Names 2.0 specification
published on the Namecoin wiki.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD",
"SHOULD NOT", "RECOMMENDED",  "MAY", and "OPTIONAL" in this document are to be
interpreted as described in RFC 2119.

This document is released into the public domain.

**Table of Contents**

- [Namecoin: Domain Names](#namecoin-domain-names)
  - [Introduction](#introduction)
  - [Keys](#keys)
  - [Values](#values)
    - [Notes on JSON](#notes-on-json)
    - [The Domain Name Object Schema](#the-domain-name-object-schema)
      - [Abstract Constructs](#abstract-constructs)
      - [DNS-Compatible Records](#dns-compatible-records)
      - [Non-DNS Item Types](#non-dns-item-types)
      - [Administrative Constructs](#administrative-constructs)
      - [Experimental DNS-Compatible Item Types](#experimental-dns-compatible-item-types)
  - [Interpretation of DNS Names](#interpretation-of-dns-names)
  - [The WHOIS Entity Schema](#the-whois-entity-schema)
  - [Definitions of Valid Names](#definitions-of-valid-names)
  - [Notes on DNS Subtleties](#notes-on-dns-subtleties)
  - [Error Recovery Considerations](#error-recovery-considerations)
  - [Previously Deprecated Item Types](#previously-deprecated-item-types)
  - [Newly Deprecated Item Types](#newly-deprecated-item-types)
  - [Possible Future Directions](#possible-future-directions)

Introduction
------------
A Namecoin domain name is a domain name published under the .bit TLD. This is
accomplished by storing a key-value pair in the Namecoin key-value database.

Such domain names are compatible with DNS and can be used to serve DNS data.
They can also be used to serve non-DNS data such as .onion addresses for Tor.

Keys
----
The key of the key-value pair shall be an ASCII-encoded name in the form
`d/NAME`, where NAME does not exceed 63 characters in length and matches the
following case sensitive POSIX regex:

  `^(xn--)?[a-z0-9]+(-[a-z0-9]+)*$`

Note that as per the above regex, the standard rules for domain names are
enforced. Domain names cannot begin or end with a hyphen, and consecutive
hyphens are not allowed except as permitted by the Internationalized Domain
Names (IDN) specification.

The `d/` prefix identifies the Domain Names namespace within the Namecoin key
namespace. Names under this prefix are reserved for use in relation to this
specification and preceding versions of it. Keys not beginning with that prefix
are wholly unrelated to this specification and are not required to conform to
it.

The part of the name following the `d/` prefix shall be the name which
manifests in the .bit TLD. A key of `d/example` registers the name
`example.bit.`

Since keys in the Namecoin key-value store are case sensitive, it is essential
that the names be in lowercase. Names with any uppercase characters shall be
wholly ignored by compliant implementations and shall not manifest in the .bit
TLD.

Values
------
The value of the key-value pair SHALL be a UTF-8 encoded JSON object conforming
to the Domain Name Object Schema as described below. This specification
therefore incorporates by reference RFC 7159, which specifies JSON.

The JSON object SHOULD be encoded as compactly as possible, without unnecessary
whitespace.

No particular limit is imposed on the length of the encoded JSON value.
However, the Namecoin system itself imposes limitations, which must be respected
by values. Currently, values must not exceed 520 bytes in size. This limit may
be increased in the future.

### Notes on JSON

Probably the two most common errors in the manual construction of JSON forms is
using single quotes to delimit strings and placing trailing commas in objects
or arrays. While these are valid JavaScript forms, they are not valid JSON, and
care must be taken to conform precisely to the specification.

The JSON specification makes provision for Unicode escape sequences in quoted
strings in the form `\u####`, where `#` is a hexadecimal character. This
provision of the specification is flawed because it does not permit the
specification of Unicode codepoints not expressible in 16 bits. Indeed the
specification states that codepoints beyond the Basic Multilingual Plane shall
be encoded using the surrogate codepoints assigned for use by UTF-16. The JSON
specification therefore unnecessarily inherits the legacy of UTF-16. The
Unicode escape syntax MUST NOT be used except where it is necessary; namely, it
is necessary where a control character which does not have an alternate escape
sequence forms part of a string, as control characters as defined in the
specification are forbidden from being encoded directly. However
implementations MUST accept the use of such escape sequences. Where an escaped
value validly expresses a codepoint beyond the Basic Multilingual Plane using
UTF-16 surrogate codepoints, an implementation parsing it which uses UTF-8 or
UTF-32 as its internal representation MUST immediately convert it to the
appropriate surrogate-free encoding, as surrogates must not appear in a
valid UTF-8 or UTF-32 code unit sequence.

The JSON specification does not specify any format for comments. For
illustration purposes, this document will use JavaScript-style line comments
beginning with "//" at any point on a line and running until the end of the
line. Implementations MUST NOT generate or parse such comments.

### The Domain Name Object Schema

A Domain Name Object Schema SHALL possess zero or more of the following items.
Where each item is present, it MUST conform with the subschema for that item as
specified. The key name to be used for a given item is shown first, in quotes.

Except where otherwise specified, any item with a value of `null` shall be
treated as being absent; that is, it will be processed as though it was not
present in the object. Thus `null` is always a valid value for an item even
though it is not explicitly mentioned herein. (See the "import" item for an
example of an exception to this rule.)

#### Abstract Constructs

  - "map": Used to express a Domain Name Object for a subdomain of the current
    name.

    The value for this item shall be of the following form:

    - An object mapping zero or more subdomain names to objects.

      Each key shall be of one of the following forms:

        - A string containing a valid and ordinary DNS label; the key MUST be a
          single unqualified label and so MUST NOT contain any dots (ASCII
          '.').

          (Note that the label need not be a valid hostname; for example, it
          may contain underscores.)

        - The string "*", representing a wildcard subdomain as supported by
          DNS.

        - The string "". The corresponding value is treated specially. All
          items inside the corresponding object are processed as though they
          are directly present inside the object containing the "map" item
          containing this key (the Immediate Ancestor), but only where the
          Immediate Ancestor does not have present any item with the same key.
          In other words, items which are present in the Immediate Ancestor
          take precedence over the items under the map item.

      Each value shall be one of the following forms:

        - An object conforming to the Domain Name Object Schema.

          This is the canonical form.

        - A string. Where this form is encountered, it shall be substituted
          with an object containing a single item with key "ip" and a value of
          an array containing that string, and be processed as though that was
          what was encountered, as per the above form.

      Examples:

          "map": {}

          "map": {
            "www": {
              "ip": "192.0.2.1"
            }
          }

          "map": {
            "www": "192.0.2.1"
          }

          "map": {                    // This is equivalent to "ip": "192.0.2.2"
            "ip": "192.0.2.2"         // Takes precedence
            "": {
              "ip": "192.0.2.1"       // Ignored
            }
          }

          "map": {                    // This is equivalent to "ip": "192.0.2.1"
            "": {
              "ip": "192.0.2.1"
            }
          }

      The following example forms are NOT valid:

          "map": []
          "map": "192.0.2.1"
          "map": {
            "$": "192.0.2.1"
          }
          "map": {
            "a.b": "192.0.2.1"
          }
          "map": {
            "www*": "192.0.2.1"
          }

  - "import": Used to import data from the value of another Namecoin key-value pair
    identified by its key. The logical contents of the Domain Name Object which
    contains this item shall consist of copying all values from the imported
    Domain Name Object into the importing Domain Name Object. However, items other
    than "import" items specified in the importing Domain Name Object shall take
    precedence, and so the importing object can override items in the imported
    object. The exception to this is where the specification for a specific
    type of item explicitly specifies an alternate merge rule, in which case
    that rule must be applied for items of that type.

    Except where otherwise specified, for the purposes of precedence, an item
    is considered to be present if it is specified in the object, regardless of
    the semantic nullity expressed by the value. For example, an "ip" item with
    an empty array as its value is considered to be present for the purposes
    of precedence.
    
    This principle is extended to the value `null`; this constitutes an
    exception to the previously stated general rule that an item with a value
    of `null` is processed equivalently to the absence of that item. Thus an
    item with a value of `null` is considered to be present for the purposes
    of precedence. For example, if an imported domain specifies an item type of
    "info", the importing domain can nullify this by adding an item type of
    "info" with value `null`.

    It is permissible for an imported object to itself import other objects. However,
    a limitation on the degree of recursion is imposed. The degree of recursion is
    an integer expressing the number of "import" items processed in the course of
    processing a single object at a given level. (Other "import" items may appear
    in "map" items, but as these are at a different level they do not count.)

    For example, the following Domain Name Object has an import recursion
    degree of zero:

        {}

    The following Domain Name Object has an import recursion degree of one:

        {"import":"d/example2"}
        // The value for d/example2 is {}

    The following Domain Name Object has an import recursion degree of two:

        {"import":"d/example2"}
        // The value for d/example2 is {"import":"d/example3"}
        // The value for d/example3 is {}

    The following Domain Name Object also has an import recursion degree of two:

        {"import":["d/example2", "d/example3"]}
        // The value for d/example2 and d/example3 is {}

    Implementations MUST support an import recursion degree of at least four.

    The value for this item shall be one of the following forms:

    - An array of zero or more arrays. Each such array expresses a single
      importation operation (though that operation may recursively give rise to
      others, as described above) and shall have at least one item, the first
      of which shall be a string expressing the name, in the Namecoin key-value
      store, the value of which should be imported as a Domain Name object.

      The name shall be in Namecoin format and not DNS format, for example
      "d/example". The importation process fails if the name does not exist (or
      is expired), or if the Domain Name Object in the value corresponding to
      the name is not valid or is otherwise unprocessable.

      If a second item is present in the array, it shall be a string and
      that string shall constitute the Subdomain Selector. Otherwise, the
      Subdomain Selector shall be taken to be the empty string.

      This form is the canonical form when the Subdomain Selector is
      explicitly specified.

      If there are more than two items in the array, any further items are
      ignored.

      The Subdomain Selector is used to identify a location in the imported
      Domain Name Object which is to be merged with the current object. If
      the Subdomain Selector is the empty string, the entire imported Domain
      Name Object is merged. Otherwise, the Subdomain Selector shall be a
      sequence of DNS labels, separated by dots. (Note that the Subdomain
      Selector must not end in a dot.) In this case, the labels in the Subdomain
      Selector are processed in reverse order (i.e., in DNS order). Each label in
      the Subdomain Selector is used to move to the subdomain with that label,
      relative to the current position, until all labels in the Subdomain
      Selector have been processed and so the final Domain Name Object for
      merger has been determined.

      It is acceptable for a label in the Subdomain Selector to be "*", and
      this corresponds to the corresponding entry in the "map" item if it exists.

      If while processing a Subdomain Selector, a label cannot be identified in
      the "map" item at the current level, the label is substituted for the
      string "*" and the processing retried. If failure still occurs, the
      importation process fails. [TBD: should this wildcard substitution really
      be done?]

      Any import statements at any location within a Domain Name Object being
      imported must be processed prior to processing the Subdomain Selector.

      Import statements must be attempted in the order they are specified in the
      array. For types of item without custom merge rules, this determines the
      precedence of the items imported; the items specified directly in the
      importing object take ultimate precedence, then the first imported
      object's items, then the second imported object's items and so on.

      Where an import statement fails, any further import statements
      should still be attempted.

    - An array of strings. When this form is encountered, each string in the array
      shall be substituted with an array containing that string and be processed
      as though that was what was encountered, as per the above form.

  - "delegate": Similar to "import", though with different rules and less
    powerful. Since "import" constitutes a superset of the functionality of
    "delegate", use of "import" is recommended instead.

    The key differences between "import" and "delegate" is that "delegate" can
    import only a single other name (though that name can still recursively
    import or delegate to other names in turn), and that the use of "delegate"
    causes any other items in the current object to be ignored.

    The other items in the current object are ignored only if the delegation is
    processed successfully (though it does not matter if any imports or delegations
    made by the name delegated to are themselves processed successfully.) If it
    is not processed successfully, the other items in the current object are
    processed normally. This allows items to be placed in the current object as a
    sort of backup record in case the name delegated to expires.

    The import recursion degree limits imposed are the same as those for the
    "import" statement.

    The value for this item shall be one of the following forms:

    - An array of one or more items. The first item shall be a string expressing
      the Namecoin name to delegate to. The second item, if present, shall be
      a string expressing the Subdomain Selector, which is processed as described
      in the specification for "import". If the second item is not present, the
      Subdomain Selector is taken as "".

      This form is the canonical form when the Subdomain Selector is explicitly
      specified (i.e. the second item is present).

    - A string. When this form is encountered, it shall be substituted with an
      array containing that string and processed as though that was what was
      encountered, as per the above form.

    Examples:

        "delegate": "d/example",
        "delegate": ["d/example"],
        "delegate": ["d/example", "www"],
        "delegate": ["d/example", "london.england.uk"],

#### DNS-Compatible Records

  - "ip": Used to specify zero or more IPv4 addresses. This item shall map to
    one or more DNS resource records of type "A", and is semantically
    equivalent to that set of resource records.

    The value for this item shall be one of the following forms:

    - An array of zero or more strings. Each such string shall contain an IPv4
      address in dotted-decimal form. The address shall not use leading zeroes
      and no whitespace shall be used. The numeric form for IPv4 addresses MUST
      NOT be used.

      Note that it is NOT the role of an implementation to validate any semantics
      deriving from the address (e.g. globally routable, loopback, Class E).

      This is the canonical form.

    - A string. Where this form is encountered, it shall be substituted with an
      array containing that string and be processed as though that was what was
      encountered, as per the above form.

    Examples:

        "ip": []
        "ip": ["192.0.2.1"]
        "ip": ["192.0.2.1", "192.0.2.2"]
        "ip": "192.0.2.1"

    The following example forms are NOT valid:

        "ip": {}
        "ip": ["192.000.002.001"]
        "ip": ["192.0.2.001"]
        "ip": ["3221225985"]
        "ip": [3221225985]

  - "ip6": Used to specify zero or more IPv6 addresses. This item shall map to
    one or more DNS resource records of type "AAAA", and is semantically
    equivalent to that set of resource records.

    The value for this item shall be one of the following forms:

    - An array of zero or more strings. Each such string shall contain an IPv6
      address in the text form as specified by RFC 4291. Each address SHOULD be
      encoded as compactly as possible. The alternative form which represents
      the last 32 bits of the address in the IPv4 dotted decimal format MAY be
      used. No whitespace shall be used.

      This is the canonical form. If a canonical representation of IPv6 addresses
      is required, use the form specified in RFC 5952.

    - A string. Where this form is encountered, it shall be substituted with an
      array containing that string and be processed as though that was what was
      encountered, as per the above form.

    Examples:

        "ip6": []
        "ip6": ["2001::beef"]
        "ip6": ["2001::dead", "2001::beef"]
        "ip6": "2001::beef"
        "ip6": "::beef:192.0.2.1"

    The following example forms are NOT valid:

        "ip6": {}
        "ip6": "2001::bxxf"
        "ip6": ["::00000:beef"]

  - "alias": Used to specify a canonical name for which the current name is an
    alias.  This item shall map to one DNS resource record of type "CNAME", and
    is semantically equivalent to that resource record.

    The value for this item shall be of the following form:

    - A string containing a DNS name. This constitutes the canonical name nominated
      as the target of the alias. The string SHOULD be a valid hostname.

      This is the canonical form.

    Examples:

        "alias": "example.com."

    The following example forms are NOT valid:

        "alias": "ex$ample.com."
        "alias": ["a.example.com.", "b.example.com."]

  - "translate": Used to specify a name for which the current name and all names under
    it are delegated. This item shall map to one DNS resource record of type "DNAME",
    and is semantically equivalent to that resource record.

    The value for this item shall be of the following form:

    - A string containing a DNS name. This constitutes the name nominated as the target
      of the delegation. The string SHOULD be a valid hostname.

      This is the canonical form.

    Examples:

        "translate": "example.com."

    The following example forms are NOT valid:

        "translate": "ex$ample.com."
        "translate": ["a.example.com.", "b.example.com."]

  - "ns": Used to specify zero or more nameservers which are authoritative for the
    current name and all names under it (barring further delegations thereunder).
    This item shall map to zero or more DNS resource records of type "NS", and is
    semantically equivalent to that set of resource records.

    "dns" is an alias for this item. In other words, a "dns" item shall be
    processed equivalently. If both a "dns" and "ns" item are present, the
    "dns" item takes precedence.

    The value for this item shall be one of the following forms:

    - An array of zero or more strings. Each such string shall contain a DNS name
      identifying a nameserver which is authoritative for the domain.

      The string MUST NOT be an IP address. It SHOULD be a hostname.

      This is the canonical form.

    - A string. Where this form is encountered, it shall be substituted with an
      array containing that string and be processed as though that was what was
      encountered, as per the above form.

    Examples:

        "ns": []
        "ns": ["ns1.example.com.", "ns2.example.com."]
        "ns": "ns1.example.com."

    The following example forms are NOT valid:

        "ns": {}
        "ns": "ns$1.example.com."
        "ns": "ns1.example.com. ns2.example.com."

  - "ds": Used to identify DNSSEC signing keys for a delegation as expressed by
    corresponding NS records. This item shall map to zero or more DNS resource records
    of type "DS", and is semantically equivalent to that set of resource records.

    The value for this item shall be of the following form:

    - An array of zero or more items. Each such item shall represent a DS record,
      and each shall be an array of the following form:

      - The array shall contain at least four items. If it contains more than
        four items, only the first four are considered and the rest are ignored.

      - The first item in the array shall be an unsigned integer expressible in
        16 bits expressing the Key Tag field (RFC 4034 s. 5.1.1) of the corresponding
        DS record.

      - The second item in the array shall be an unsigned integer expressible in
        8 bits expressing the Algorithm field (RFC 4034 s. 5.1.2) of the corresponding
        DS record.

      - The third item in the array shall be an unsigned integer expressible in
        8 bits expressing the Digest Type field (RFC 4034 s. 5.1.3) of the corresponding
        DS record.

      - The fourth item in the array shall be a string containing the base64 encoding
        of the logical content of the Digest field (RFC 4034 s. 5.1.4) of the corresponding
        DS record.

        The textual expression of this field in RFC 4034 uses hex encoding.
        Therefore this field must be converted to the correct form by decoding
        it and reencoding it using base64.

    Examples:

        "ds": []
        "ds": [[12345,8,1,"EfatjsUqKYSrqv18O1FlA3hcIHI="]]
        "ds": [[12345,8,1,"EfatjsUqKYSrqv18O1FlA3hcIHI="],[12345,8,2,"LXEWQrcmsEQBYnyp+6wy9chTD7GQPMTbAiWHF5IaSIE="]]

    The following example forms are NOT valid:

        "ds": {}
        "ds": [[]]
        "ds": [12345,8,1,"11f6ad8ec52a2984abaafd7c3b516503785c2072"]

    The following example form is literally valid, however the Digest field has been erroneously
    expressed in hexadecimal format rather than base64 format. Since the field length is a multiple
    of four characters this is valid base64. Note that the decoded digest field will be of the wrong
    length for the given Digest Type. However, it is not the job of implementations to validate this
    and implementations MUST NOT do so.

        "ds": [[12345,8,1,"11f6ad8ec52a2984abaafd7c3b516503785c2072"]]

  - "service": Used to identify zero or more service location records. This
    item shall map to zero or more DNS resource records of type "SRV", as
    defined in RFC 2782, and is semantically equivalent to that set of resource
    records (once having applied the record name transformation described
    below).

    The value for this item shall be of the following form:

    - An array of zero or more items. Each such item shall represent a SRV record,
      and shall be of the following form:

      - An array of at least six items.

        The first item shall be a string expressing the application name (the
        “Service” in RFC 2782 parlance, but without the leading underscore).

        The second item shall be a string expressing the transport protocol
        name (the “Protocol” in RFC 2782 parlance, again without the leading
        underscore.)

        The third item shall be a non-negative integer expressible in 16 bits
        expressing the Priority of the SRV record.

        The fourth item shall be a non-negative integer expressible in 16 bits
        expressing the Weight of the SRV record.

        The fifth item shall be a non-negative integer expressible in 16 bits
        expressing the Port Number of the SRV record.

        The sixth item shall be a string expressing a DNS name expressing
        the Target of the SRV record.

        Any additional items in the array beyond the first six shall be
        ignored.

    The "service" item is processed by taking the SRV records expressed by it
    and, for each of them, prepending to the DNS record's name the string
    `_AppProto._TransProto.` where `AppProto` and `TransProto` are the values
    as specified in the above form. Thus the correct name for each SRV record
    is formed automatically from the specified values, without having to use
    "map" items.

  - "tls": Used to identify zero or more TLS anchor records. This item shall
    map to zero or more DNS resource records of type "TLSA", as defined in RFC
    6698, and is semantically equivalent to that set of resource records (once
    having applied the record name transformation described below).

    The value for this item shall be of the following form:

    - An array of zero or more items. Each such item shall represent a TLSA record,
      and shall be of the following form:

      - An array of at least six items.

        The first item shall be a string (or an integer, in which case it is
        converted to the string which is the decimal representation of it without
        leading zeroes) expressing the port number.

        The second item shall be a string expressing the transport protocol name.

        The third item shall be a non-negative integer expressible in 8 bits
        expressing the Certificate Usage Field of the TLSA record (RFC 6698 s. 2.1.1).

        The fourth item shall be a non-negative integer expressible in 8 bits
        expressing the Selector Field of the TLSA record (RFC 6698 s. 2.1.2).

        The fifth item shall be a non-negative integer expressible in 8 bits
        expressing the Matching Type Field of the TLSA record (RFC 6698 s. 2.1.3).

        The sixth item shall be a string containing the base64 encoding of the
        logical encoding of the Certificate Association Data Field of the TLSA record (RFC 6698 s. 2.1.4).

        The textual expression of this field in RFC 6698 uses hex encoding. Therefore this
        field must be converted to the correct form by decoding it and reencoding it
        using base64.

        Any additional items in the array beyond the first six shall be ignored.

    The "tls" item is processed by taking the TLSA records expressed by it and,
    for each of them, prepending to the DNS record's name the string
    `_PortNumber._TransProto.` where `PortNumber` and `TransProto` are the
    values as specified in the above form. Thus the correct name for each TLSA
    record is formed automatically from the specified values, without having to
    use "map" items.

  - "txt": Used to identify zero or more text data records. This item shall map
    to zero or more DNS resource records of type "TXT", and is semantically
    equivalent to that set of resource records.

    It is an important subtlety to the specification for the DNS TXT record type
    that the TXT format actually expresses a sequence of one or more text strings,
    each of which must not exceed 255 bytes in length.

    The most common use of the TXT record type appears to consider such records
    to logically represent the string formed by concatenating those component
    strings. For example, DomainKeys, which uses TXT records to store signing key
    identities in DNS, does this, as the key data may exceed 255 bytes in length.

    The value for this item shall be of one of the following forms:

    - An array of zero or more items. Each such item shall represent a TXT record,
      and shall take one of the following forms:

      - An array of one or more strings. Each such string shall not exceed 255
        bytes in its UTF-8 representation.

      - A string. Where this form is encountered, it shall be substituted with
        an array of one or more strings, such that all but the last string
        in the array is 255 bytes in the length of its UTF-8 representation,
        and be processed as though that was what was encountered, as per the above
        form.

        In other words, the string is chopped up so that it can be expressed as
        a sequence of strings each up to 255 bytes in length, such that the
        concatenation of those strings forms the original string. This is the
        behaviour expected by many formats which use TXT records, such as
        DomainKeys.

        If the sequence of strings logically represented by this record
        consists of a sequence of strings of length 255 bytes optionally
        terminated by a string with a length of less than 255 bytes, then this
        form is the canonical form. Otherwise, the above form (the
        array-of-arrays form) is the canonical form.

    - A string. Where this form is encountered, it shall be substituted with an
      array containing that string and be processed as though that was what was
      encountered, as per the above form.

    Examples:

        "txt": "This is a string."
        "txt": ["This is a string.", "Another string."]
        "txt": [["This", "is", "a", "string."], "Another string."]

    The following example forms are NOT valid:

        "txt": {}
        "txt": ["This is a string.", 1]
        "txt": [["(a string longer than 255 bytes)"]]

  - MX: MX records cannot be specified directly. Instead, where a domain name
    provides a service using the "service" item type for a service with an
    application name of "smtp" and a transport protocol name of "tcp" (`_smtp._tcp`),
    implementations generating DNS records shall generate an MX record for each
    endpoint specified for that service which specifies a port number of 25. Any
    `_smtp._tcp` SRV record with a port number other than 25 is ignored. The MX
    record shall be constructed using the priority and target fields of the
    SRV record. The weight and port fields shall be discarded.

    Implementations must still make the SRV records available in addition to the
    MX records.

  - "loc": Used to identify zero or more physical location records. This item
    shall map to zero or more DNS resource records of type "LOC", and is
    semantically equivalent to that set of resource records.

    The value for this item shall be of one of the following forms:
    
    - An array of zero or more items. Each such item shall be a string conforming
      to the textual format specified in RFC 1876 s. 3 for the type-specific data.

    - A string. Where this form is encountered, it shall be substituted with an
      array containing that string and be processed as though that was what as
      encountered, as per the above form.

    Examples:

        "loc": []
        "loc": "52 22 23.000 N 4 53 32.000 E -2.00m 0.00m 10000m 10m"

    The following example forms are NOT valid:

        "loc": "10 Downing Street"

#### Non-DNS Item Types

  - "tor": Provides a Tor onion address.
  
    The value for this item shall take the form of a string containing a Tor
    onion address, including the ".onion" suffix. (Note that the value MUST NOT
    be an URL.) The string must not have a trailing dot. This value denotes
    that the current object is mappable to a Tor hidden service with the given
    address.

  - "i2p": Provides an I2P eepsite address.

    TODO
    
  - "freenet": Provides a Freenet freesite key.

    The value for this item shall take the form of a string containing a Freenet
    freesite key, e.g. "USK@0I8g...xbZ4,AQACAAE/Example/42/".

#### Administrative Constructs

  - "info": This optional item can be used to provide WHOIS-like information.

    The value for this item shall be of one of the following forms:

    - An object containing zero or more of the following items:

      - "r": A registrant. This represents the “legal” owner of the domain.
        The value shall comply with the WHOIS Entity Schema as described in this document.

      - "a": An administrative contact. This represents the administrative
        operator of the domain. The value shall comply with the WHOIS Entity
        Schema as described in this document.

      - "t": A technical contact. This is the appropriate point of contact for
        nameserver or other DNS-related issues. The value shall comply with the
        WHOIS Entity Schema as described in this document.

    - A value directly complying with the WHOIS Entity Schema. This can be used
      where one entity takes on all three roles.

#### Experimental DNS-Compatible Item Types

  - "o": Used to opaquely express arbitrary DNS records. This item shall map
    to zero or more DNS resource records, and is semantically equivalent to
    that set of resource records.

    The value for this item shall be of the following form:

    - An array of zero or more items. Each such item shall represent a DNS
      resource record, and shall be of the following form:

      - An array of at least two items.

        The first item shall be a non-negative integer expressible in 16 bits
        expressing the resource record type number of the DNS resource record.
        For example, specifying 16 would mean a TXT record.

        The second item shall be a string containing base64-encoded data
        representing the type-specific resource record data in its binary form.

        Any additional items in the array shall be ignored.

    Each opaque DNS resource record expressed MUST be processed only where the
    type of resource record it expresses is not one of the prohibited types.

    The prohibited types are:

      - NS    (2)  -- Use "ns".
      - CNAME (5)  -- Use "alias".
      - SOA   (6)
      - DNAME (39) -- Use "translate".
      - RRSIG (46)
      - NSEC  (47)
      - NSEC3 (50)

    Discussion: This item type is experimental. It is possible that allowing
    arbitrary DNS resource record types to be placed in the `.bit` zone may
    constitute a security risk. Since no currently operated public TLD
    registry allows arbitrary records to be placed in their zone (only NS and
    DS records and A/AAAA glue records), there is little data on the
    practical implications of this mode. Therefore the prohibited types list
    is specified to provide at least minimal protection against resource
    record types with particularly infrastructural significance. All of the
    prohibited types listed above are processed specially by DNS resolvers
    and/or authoritative servers.

Interpretation of DNS Names
---------------------------
Some values in some item types, such as the value of the "alias" item type,
express DNS names. These names may be fully qualified, in which case this is
indicated by terminating them with a dot. Where this is not done, such names
are relative and must be interpreted according to the following rules.

As an exception to the normal rules of what constitutes a valid DNS label, the
last label of a relative DNS name may be "@". If this is the case, the rest of
the label is interpreted relative to the name apex. The name apex is the DNS
name of the form `NAME.bit.`, where NAME is the Namecoin domain name under
which the current object ultimately lies. Thus for name `d/example`, the
relative name `foo.@` is equivalent to the value `foo.example.bit.`

Where a relative name does not end with the label "@", it is a relative name
interpreted relative to the current object. If the current object is the
top-level object (that is, the object encoded directly into the value of the
Namecoin key-value pair representing the domain name), then the relative name
is interpreted relative to the name apex, and so `foo.@` and `foo` have
identical meanings.

If the current object is not the top-level object (for example, it is the value
of an item in a "map" item), the relative name is interpreted relative to the parent object (that is, the object containing the "map" item).

For example, in the following example Domain Name Object, `foo.bar` is resolved to `foo.bar.@`.

    {
      "map": {
        "www": {
          "alias": "foo.bar"
        }
      }
    }

In the following Domain Name Object, `foo.bar` is resolved to `foo.bar.baz.@`.

    {
      "map": {
        "baz": {
          "map": {
            "www": {
              "alias": "foo.bar"
            }
          }
        }
      }
    }

In all cases, the name must constitute a valid DNS name after it is resolved
and thus becomes fully qualified. In particular, this means that the resulting name must
not exceed 255 characters.

The WHOIS Entity Schema
-----------------------
A value complies with the WHOIS Entity Schema if it takes one of the following forms:

  - An object complying with the rules for an object as would be encoded into the
    value of a name in the `id` namespace. The object must comply with the rules
    for values in that namespace.

  - A string beginning with "id/" and thereby identifying the Namecoin name of
    an identity in the `id` namespace. The identifier ends at the first
    whitespace character, if any, and anything that follows is a freeform
    string which may be used to provide additional information.

    The string up to and excluding the first whitespace character (or the
    entire string, if there is no whitespace) must be a valid name in the `id`
    namespace as per the rules of that namespace. The rules of that namespace
    shall be used in representing the entity.

  - A freeform string describing the entity arbitrarily. The string must not
    begin with "id/".

Definitions of Valid Names
--------------------------
This specification refers to "DNS names" and "hostnames". The set of valid
hostnames is a subset of the set of valid DNS names.

A name is a valid DNS name if:

  - it consists of a sequence of zero or more valid DNS labels separated by
    ASCII '.' and optionally terminated by a single '.' which denotes that it
    is fully qualified, and;

  - it does not exceed 255 octets in length.

What constitutes a valid DNS label is beyond the scope of this specification.
(After all, the rules have changed over time; see the now deprecated RFC 2874).
However, the set of valid DNS labels MUST be a superset of the DNS labels which
comply with the following:

  - it matches the POSIX regexp `^[a-z0-9_-]+$`, and;

  - it does not exceed 63 octets in length.

A hostname is a particular kind of DNS name following stricter rules. Namely,
every label in a hostname must be a valid host label. A label is a valid host
label if it complies with the following:

  - it matches the POSIX regexp `^(xn--)?([a-z0-9]+-)*[a-z0-9]+$`, and;

  - it is a valid DNS label.

Names of all kinds SHOULD always be specified in lowercase.

Notes on DNS Subtleties
-----------------------
This document does not bother to reiterate the various subtleties of DNS which
apply with regard to the items defined herein which are mappable to DNS
resource records. For example, NS records normally preclude any other records
at or below that name, but DS records at that name and IPv4 or IPv6 glue
records below it are an exception. Since domains may rely on glue records
correct processing of these cases is critical. In general a good understanding
of DNS and its edge cases is necessary in implementing this specification, and
there are doubtless other potential issues not listed here.

Error Recovery Considerations
-----------------------------
Any domain name system has the capacity to operate as critical infrastructure.
It is important that implementations not penalise names excessively for partial
invalidity in their values. Where an error is encountered, it should be silently
ignored and an attempt to process the remainder of the object should be made.

For example, in the following example an invalid IP address is specified:

    {
      "ip": ["site", "192.0.2.1"]
    }

The "ip" item is not validly constructed, and so the Domain Name Object itself
is not validly constructed. However, a resilient implementation will consider
the domain to at least have the IP address (A record) of 192.0.2.1. This demonstrates
a general principle in processing Domain Name Objects: semantic or form errors
in an item should not penalize the processing of other items, and items which
express multiple conceptual values should have as many of those values processed
as possible.

Because errors should not cause processing to stop, the outcome of processing
should not vary based on whether the erroneous values are leading or trailing
with regard to the valid data.

Definition of Base64
--------------------
All references to base64 encoding in this document refer to the base64 encoding
scheme described in RFC 4648. Base64 encoding and decoding MUST be performed
as specified therein. Encoded base64 strings MUST be in canonical form as specified
in RFC 4648 s. 3.5. The standard alphabet shall be used, as described in RFC 4648 s. 4. The "URL-safe" encoding MUST NOT be used.

Previously Deprecated Item Types
--------------------------------

The following item types were already deprecated before this document. They
remain deprecated.

  - "fingerprint": This was used to express a certificate fingerprint for a
    service provided at a domain. It is deprecated in favour of the "tls" item type.

Newly Deprecated Item Types
---------------------------

The following item types are deprecated by this document.

  - "tls": A previous format for the "tls" item type was proposed which strips out
    some of the fields incorporated in the TLSA DNS record type. Since this precludes
    parity with DNS, use of this format is highly undesirable.

    The proposed format was also inconsistent with the "service" (SRV) specification
    in its denotation of subdomains. ("service" uses `[[appName, transportName, ...]]`
    whereas the proposed "tls" format used an object mapping transport protocols to
    objects mapping port numbers to arrays of incomplete TLSA-ish records. This
    scheme has further problems as JSON does not support non-string keys, which
    means the port numbers had to be denoted as strings.)

    As of writing, just 14 domains are attempting to make use of the (old)
    "tls" format. Of those domains, only six do so correctly, one of which has
    other configuration issues; two erroneously specify integers and not
    strings for the port number, and six appear to use an even older format:

        "tls": { "sha1": ["12:34:56:AB:CD:EF:..."], "enforce": "self" }

  - "email": This is nominally intended to specify an e. mail address for insertion
    into the hostmaster field of a SOA record. However, this assumes that each
    .bit domain will constitute its own zone and thus have a SOA record. This is
    unworkable since many such domains may simply delegate to another nameserver
    via the "ns" item type and two zone apexes cannot occur at the same name.

    ICANN TLDs are operated as a single zone containing NS, DS and glue
    records. This is likely to be a preferable mode of operation. Thus the
    "email" item type has little use.

    Even if zone-per-domain operation is used, the SOA hostmaster field is of such
    obscure and minimal utility that it is unlikely to be of much interest. Superior
    information can be obtained by WHOIS information provided by the "info" item type,
    or via the standard e. mail address of hostmaster@, which should always be supported
    anyway.

  - "spf": Attempts to transition to the use of a special SPF DNS resource
    record type for the purposes of publishing SPF information have mostly
    failed. SPF TXT records outnumber SPF data published in SPF records. Since
    all implementations must support SPF in TXT records, there is little
    advantage to the use of the SPF resource record type. In fact, further
    standards developments such as DomainKeys use TXT records to store their
    information.

    As of writing, no domain name registered in the name database uses this item type.

Possible Future Directions
--------------------------

  - msgpack encoding: Aside from the more compact encoding, an advantage of msgpack
    is its support for binary strings, both in terms of efficient encoding (so no base64)
    and the semantic distinction. (Since JSON supports only strings, there is no schema-free
    way to distinguish between a logical text string and a logical binary string represented
    as a base64-encoded JSON string.)

  - Compression: Since values are not long, anything heavier than zlib is likely to be
    counterproductive. Thus if compression were supported, the choice would be between
    zlib and lighter formats such as Snappy or LZ4.


