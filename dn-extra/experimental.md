  - "ptr": Used to identify zero or more name pointer records. This item shall
    map to zero or more DNS resource records of type "PTR", as defined in RFC
    2915, and is semantically equivalent to that set of resource records.

    The value for this item shall be of one of the the following forms:

    - An array of zero or more items. Each such item shall be a string
      representing a PTR record and containing a valid domain name, which
      constitutes the value of that record.

    - A string. Where this form is encountered, it shall be substituted with an
      array containing that string and be processed as though that was what as
      encountered, as per the above form.

    The conventional use of PTR records is in reverse DNS, which is not applicable
    to Namecoin. However, PTR records may be used for other purposes.

  - "naptr": Used to identify zero or more naming authority pointer records.
    This item shall map to zero or more DNS resource records of type "NAPTR",
    as defined in RFC 2915, and is semantically equivalent to that set of
    resource records.

    The value for this item shall be of the following form:

    - An array of zero or more items. Each such item shall represent a NAPTR
      record, and shall be of the following form:

      - An array of at least six items.

        The first item shall be a non-negative integer expressible in 16 bits
        expressing the Order of the NAPTR record.

        The second item shall be a non-negative integer expressible in 16 bits
        expressing the Preference of the NAPTR record.

        The third item shall be a string the UTF-8 representation of which shall
        not exceed 255 bytes, expressing the Flags field of the NAPTR record.

        The fourth item shall be a string the UTF-8 representation of which shall
        not exceed 255 bytes, expressing the Service field of the NAPTR record.

        The fifth item shall be a string the UTF-8 representation of which shall
        not exceed 255 bytes, expressing the Regexp field of the NAPTR record.

        The sixth item shall be a string containing a valid domain name,
        expressing the Replacement field of the NAPTR record.

        Any additional items in the array beyond the first six shall be ignored.

    EXPERIMENTAL. No custom merge rule is currently specified for this item type,
    but one is probably necessary for proper use.

  - "a6": Used to identify zero or more IPv6 Anchor records. This item shall
    map to zero or more DNS resource records of type "A6", as defined in RFC 2874,
    and is semantically equivalent to that set of resource records.

    The value for this item shall be of the following form:

    - An array of zero or more items. Each such item shall represent an A6 record,
      and shall be of the following form:

      - An array of at least two items.

        The first item shall be an integer in the range 0 <= x <= 128
        expressing the Prefix Length of the A6 record.

        The second item shall be a string representing an IPv6 address
        expressing the Address Suffix of the A6 record. If the Prefix Length is
        128, this may be specified as `null`.

        If the Prefix Length is not zero, there shall be a third item which shall
        be a string containing a valid domain name expressing the Prefix Name of
        the A6 record. If the Prefix Length is zero, this field shall be absent
        or shall be specified as `null`.

        Any additional items in the array shall be ignored.

    HISTORIC. The A6 record is no longer recommended for use.

  - "sshfp": Used to identify zero or more SSH server key fingerprints. This item
    shall map to zero or more DNS resource records of type "SSHFP", as defined in
    RFC 4255, and is semantically equivalent to that set of resource records.

    The value for this item shall be of the following form:

    - An array of zero or more items. Each such item shall represent a SSHFP record,
      and shall be of the following form:

      - An array of at least three items.

        The first item shall be a non-negative integer expressible in 8 bits
        expressing the Algorithm Number of the corresponding SSHFP record (RFC
        4255, s. 3.1.1).

        The second item shall be a non-negative integer expressible in 8 bits
        expressing the Fingerprint Type of the corresponding SSHFP record (RFC
        4255, s. 3.1.2).

        The third item shall be a string containing the base64 encoding of the
        logical content of the Fingerprint field of the corresponding SSHFP
        record (RFC 4255, s. 3.1.3).

        The textual expression of this field in RFC 4255 uses hex encoding. There
        this field must be converted to the correct form by decoding it and
        reencoding it using base64.

        Any additional items in the array shall be ignored.


