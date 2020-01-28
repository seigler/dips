<pre>
  BIP: ?
  Layer: Applications
  Title: URI Scheme
  Author: Joshua Seigler <dip-uri-identity@joshua.seigler.net>
  Status: Draft
  Dash-Status: Draft
  Type: Standards Track
  Created: 2020-01-21
  License: MIT License
</pre>

This DIP extends [Dash's adaptation of BIP-0021](https://github.com/dashevo/bips/blob/master/bip-0021.mediawiki).

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
    1. [Dash Platform Name](#dash-platform-name)
1.  [Copyright](#copyright)

## Abstract
This DIP proposes an extension to the URI scheme for making payments.

## Motivation
The purpose of this extension is to enable a payment URI to include a Dash platform name, and an assurance that the person who controls the name endorses the payment address in the URI.

## Prior Work

Dash version of [BIP-0021](https://github.com/dashevo/bips/blob/master/bip-0021.mediawiki), a URI scheme for making Bitcoin payments  
[BIP-0020](https://github.com/dashevo/bips/blob/master/bip-0020.mediawiki)

## Specification

### General rules for handling (important!)

Clients MUST NOT act on URIs without getting the user's authorization.
They SHOULD require the user to manually approve each payment individually, though in some cases they MAY allow the user to automatically make this decision.

### Operating system integration

Graphical clients SHOULD register themselves as the handler for the "dash:" URI scheme by default, if no other handler is already registered. If there is already a registered handler, they MAY prompt the user to change it once when they first run the client.

### General format

Payment URIs follow the general format for URIs as set forth in RFC 3986. The path component consists of a valid address, and the query component provides additional payment options.

Elements of the query component may contain characters outside the valid range. These must first be encoded according to UTF-8, and then each octet of the corresponding UTF-8 sequence must be percent-encoded as described in RFC 3986.

### ABNF grammar

(See also [a simpler representation of syntax](#simpler-syntax))

    dashurn     = "dash:" dashaddress [ "?" dashparams ]
    dashaddress = *base58
    dashparams  = dashparam [ "&" dashparams ]
    dashparam   = [ amountparam / labelparam / messageparam / instantsendparam / dpnparam / otherparam / reqparam ]
    amountparam    = "amount=" *digit [ "." *digit ]
    labelparam     = "label=" *qchar
    messageparam   = "message=" *qchar
    instantsendparam     = "IS=" *bool
    dpnparam       = "dpn=" *qchar "," *base58
    otherparam     = qchar *qchar [ "=" *qchar ]
    reqparam       = "req-" qchar *qchar [ "=" *qchar ]

Here, "qchar" corresponds to valid characters of an RFC 3986 URI query component, excluding the "=" and "&" characters, which this DIP takes as separators.

The scheme component ("dash:") is case-insensitive, and implementations must accept any combination of uppercase and lowercase letters. The rest of the URI is case-sensitive, including the query parameter keys.

### Query Keys

- **label**: Label for that address
- **address**: dash address
- **message**: message that describes the transaction to the user (see examples below)
- **amount**: amount of base dash units (see below)
- **dpn**: A Dash Platform name and a signature approving this dash address ([see below](#dash-platform-name))
- **(others)**: optional, for future extensions

### Transfer amount

If an amount is provided, it MUST be specified in decimal.
All amounts MUST contain no commas and use a period (.) as the separating character to separate whole numbers and decimal fractions.
I.e. amount=50.00 or amount=50 is treated as 50, and amount=50,000.00 is invalid.

Clients MAY display the amount in any format that is not intended to deceive the user.
They SHOULD choose a format that is foremost least confusing, and only after that most reasonable given the amount requested.
For example, so long as the majority of users work in DASH units, values should always be displayed in DASH by default, even if mDASH would otherwise be a more logical interpretation of the amount.

### Dash Platform Name

Dash clients MAY display the profile of a user indicated by the `dpn` field, if the ID and signature are validated by DAPI.
If the ID and proof do not match the Dash Platform name, clients SHOULD indicate the mismatch in some way, e.g. with a warning.
The message being signed is just the selected address, and the message is signed by the private key of the latest identity associated with the Dash platform name.
The signature field is the base58 encoding of that signature.

### Payment identifiers, not person identifiers

Current best practices are that a unique address should be used for every transaction.
Therefore, a URI scheme should not represent an exchange of personal information, but a one-time payment.

### Accessibility (URI scheme name)

Should someone from the outside happen to see such a URI, the URI scheme name already gives a description.
A quick search should then do the rest to help them find the resources needed to make their payment.
Other proposed names sound much more cryptic; the chance that someone googles that out of curiosity are much slimmer.
Also, very likely, what he will find are mostly technical specifications - not the best introduction to Dash.

## Forward compatibility

Variables which are prefixed with a req- are considered required.  If a client does not implement any variables which are prefixed with req-, it MUST consider the entire URI invalid.  Any other variables which are not implemented, but which are not prefixed with a req-, can be safely ignored.

## Backward compatibility

The parameter added here is optional, so older clients will ignore it.

## Appendix
### Simpler syntax

This section is non-normative and does not cover all possible syntax.
Please see the BNF grammar above for the normative syntax.

[foo] means optional, &lt;bar&gt; are placeholders

    dash:<address>[?amount=<amount>][?label=<label>][?message=<message>][?IS=<instantsend>][?dpn=<dash platform name>,<signature>]

## Examples

Just the address:

    dash:X75tWpb8K1S7NmH4Zx6rewF9WQrcZv245W

Address with name:

    dash:X75tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?label=Luke-Jr

Request 20.30 DASH to "Luke-Jr":

    dash:X75tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?amount=20.3&label=Luke-Jr

Request 11 DASH with Dash Platform name:

    dash:X75tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?amount=11&dpn=evan.dash,eaXxEtfFFkE5DsY5MvsT7HFoJTPiZkRZyvLTjHx7kvFGHJgqSuFaZUhdx94QG47BhjrCJHkGqoPcyDWh3bVkYmp

Request 50 DASH with message:

    dash:X75tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?amount=50&label=Luke-Jr&message=Donation%20for%20project%20xyz

Request 50 DASH (via InstantSend) with message:

    dash:X75tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?amount=50&label=Luke-Jr&message=Donation%20for%20project%20xyz&IS=1

Some future version that has variables which are (currently) not understood and required and thus invalid:

    dash:X75tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?req-somethingyoudontunderstand=50&req-somethingelseyoudontget=999

Some future version that has variables which are (currently) not understood but not required and thus valid:

    dash:X75tWpb8K1S7NmH4Zx6rewF9WQrcZv245W?somethingyoudontunderstand=50&somethingelseyoudontget=999

Characters must be URI encoded properly.

## Reference Implementations

### Dash clients
- Dash-Core supports the old version of Dash URIs (i.e. without the req- prefix), with Windows and KDE integration as of commit [https://github.com/dashpay/dash/commit/70f55355e29c8e45b607e782c5d76609d23cc858  70f55355e29c8e45b607e782c5d76609d23cc858].

### Libraries
- Javascript - https://github.com/bitcoinjs/bip21
- Java - https://github.com/SandroMachado/BitcoinPaymentURI
- Swift - https://github.com/SandroMachado/BitcoinPaymentURISwift

## Dash Modifications
- Remove coin specific references where possible
- Change handler from "bitcoin:" to "dash:"
- Add the InstantSend URI parameter ("IS")
- Add the Dash platform name parameter ("dpn")
