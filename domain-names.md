Namecoin: Domain Names 3.0
==========================

This is a draft specification under development. It has no official endorsement
by the Namecoin project. Do not use it; instead, use the Domain Names 2.0
specification published on the Namecoin wiki.

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

#### Abstract Constructs

  - "map": Used to express a Domain Name Object for a subdomain of the current
    name.

    The value for this item shall be of the following form:

    - An object mapping zero or more subdomain names to objects.

      Each key shall be of one of the following forms:

        - A string containing a valid and ordinary DNS label; the key MUST be a
          single unqualified label and so MUST NOT contain any dots (ASCII
          '.').

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

        The textual expression of this field in RFC 4034 uses hex encoding. Therefore this
        field must be converted to the correct form by decoding it and reencoding it 
        using base64. The base64 encoding shall be performed as described in RFC 4648.
        The encoded string shall be in canonical form as specified in RFC 4648 s. 3.5.
        The standard alphabet shall be used, as described in RFC 4648 s. 4. The "URL-safe"
        encoding shall NOT be used.

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

  - "service": TODO

  - "mx": TODO

  - "txt": TODO

  - "tls": TODO

### Non-DNS Item Types

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

### Administrative Constructs

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

    - A freeform string. This is necessary for compatibility purposes.

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

  - it matches the POSIX regexp `^([a-z0-9]+-)*[a-z0-9]+$`, and;

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

Deprecated Item Types
---------------------

  - "tls":

  - "loc":

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

  - "fingerprint": This was used to express a certificate fingerprint for a
    service provided at a domain. It is deprecated in favour of the "tls" item type.
