<pre>
  DIP: ?
  Layer: Applications
  Title: URI Scheme
  Author(s): Joshua Seigler
  Status: Draft
  Type: Standard
  Created: 2020-01-21
  License: MIT License
</pre>

## Table of Contents

1.  [Abstract](#abstract)
1.  [Motivation](#motivation)
1.  [Prior Work](#prior-work)
1.  [Specification](#specification)
  1. [General rules for handling (important!)](#general-rules-for-handling-important)
  1. [Operating system integration](#operating-system-integration)
  1. [General format](#general-format)
  1. [ABNF grammar](#abnf-grammar)
  1. [Query keys](#query-keys)
  1. [Transfer Amount](#transfer-amount)
1.  [Copyright](#copyright)

## Abstract

This DIP augments the URI scheme for making payments to include an identity
claim and a hash that proves the claim's validity.

## Motivation

The purpose of this URI scheme is to enable users to easily make payments by
simply clicking links on webpages or scanning QR Codes.

## Prior Work

[BIP-0021](https://github.com/bitcoin/bips/blob/master/bip-0021.mediawiki), a
URI scheme for making Bitcoin payments

## Specification

### General rules for handling (important!)

Dash clients MUST NOT act on URIs without getting the user's authorization. They
SHOULD require the user to manually approve each payment individually, though in
some cases they MAY allow the user to automatically make this decision.

### Operating system integration

Graphical Dash clients SHOULD register themselves as the handler for the
"dash:" URI scheme by default, if no other handler is already registered. If
there is already a registered handler, they MAY prompt the user to change it
once when they first run the client.

### General format

Dash URIs follow the general format for URIs as set forth in RFC 3986. The path
component consists of a dash address, and the query component provides
additional payment options.

Elements of the query component may contain characters outside the valid range.
These must first be encoded according to UTF-8, and then each octet of the
corresponding UTF-8 sequence must be percent-encoded as described in RFC 3986.

### ABNF grammar

(See also [a simpler representation of syntax](#simpler-syntax))

    dashurn     = "dash:" dashaddress [ "?" dashparams ]
    dashaddress = *base58
    dashparams  = dashparam [ "&" dashparams ]
    dashparam   = [ amountparam / labelparam / messageparam / idparam / proofparam / otherparam / reqparam ]
    amountparam    = "amount=" *digit [ "." *digit ]
    labelparam     = "label=" *qchar
    messageparam   = "message=" *qchar
    idparam        = "id=" *base58
    proofparam     = "proof=" *base58
    otherparam     = qchar *qchar [ "=" *qchar ]
    reqparam       = "req-" qchar *qchar [ "=" *qchar ]

Here, "qchar" corresponds to valid characters of an RFC 3986 URI query
component, excluding the "=" and "&" characters, which this DIP takes as
separators.

The scheme component ("dash:") is case-insensitive, and implementations must
accept any combination of uppercase and lowercase letters. The rest of the URI
is case-sensitive, including the query parameter keys.

### Query keys

    label: Label for that address or DPNS name for user
    address: dash address
    message: message that describes the transaction to the user (see examples below)
    amount: amount of base dash units (see below)
    id: platform ID used for proof
    proof: value of [address] field, signed by the private key of [id]
    (others): optional, for future extensions

### Transfer amount

If an amount is provided, it MUST be specified in decimal Dash. All amounts MUST
contain no commas and use a period (.) as the separating character to separate
whole numbers and decimal fractions. I.e. amount=50.00 or amount=50 is treated
as 50 Dash, and amount=50,000.00 is invalid.

Dash clients MAY display the amount in any format that is not intended to
deceive the user. They SHOULD choose a format that is foremost least confusing,
and only after that most reasonable given the amount requested. For example, so
long as the majority of users work in Dash units, values should always be
displayed in Dash by default, even if mDash or something else would be a more
logical interpretation of the amount.

Dash clients MAY display the profile of a user indicated by the label field, if
the ID and proof are validated by DAPI. If the ID and proof do not match the
label, clients SHOULD indicate the mismatch in some way, e.g. with a warning.

## Appendix
### Simpler syntax

This section is non-normative and does not cover all possible syntax. Please see
the BNF grammar above for the normative syntax.

[foo] means optional, <bar> are placeholders

    dash:<address>[?amount=<amount>][?label=<label>][?message=<message>]

### Examples

Just the address:

    dash:175tWpb8K1S7NmH4Zx6rewF9WQrcZv245W

Address with name:

    dash:175tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?label=Evan

Request 20.30 Dash to "Evan":

    dash:175tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?amount=20.3&label=Evan

Request 50 Dash with message:

    dash:175tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?amount=50&label=Evan&message=Donation%20for%20project%20xyz

Some future version that has variables which are (currently) not understood and
required and thus invalid:

    dash:175tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?req-somethingyoudontunderstand=50&req-somethingelseyoudontget=999

Some future version that has variables which are (currently) not understood but
not required and thus valid:

    dash:175tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?somethingyoudontunderstand=50&somethingelseyoudontget=999

Characters must be URI encoded properly. 

## Copyright

Copyright (c) 2020 Dash Core Group, Inc. [Licensed under the MIT
License](https://opensource.org/licenses/MIT)
