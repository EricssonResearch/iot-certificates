---
title: CBOR IoT Profile of X.509 Certificates
# abbrev: CBOR-IoT-Certificates
docname: draft-raza-ace-cbor-iot-certificates-latest

ipr: trust200902
wg: ACE Working Group
cat: std

coding: utf-8
pi: # can use array (if all yes) or hash here
  toc: yes
  sortrefs: yes
  symrefs: yes
  tocdepth: 2

author:
      -
        ins: S. Raza
        name: Shahid Raza
        org: RISE SICS
        email: shahid.raza@ri.se        
      -
        ins: J. Höglund
        name: Joel Höglund
        org: RISE SICS
        email: joel.hoglund@ri.se   
      -
        ins: G. Selander
        name: Göran Selander
        org: Ericsson AB
        email: goran.selander@ericsson.com
      -
        ins: J. Mattsson
        name: John Mattsson
        org: Ericsson AB
        email: john.mattsson@ericsson.com



normative:

  RFC2119:
  RFC7049:
  RFC7228:
  RFC8174:


informative:

  RFC6347:
  I-D.selander-ace-cose-ecdhe:



--- abstract

This document specifies a CBOR encoding and profiling of X.509 public key certificate suitable for Internet of Things (IoT) deployments. The full X.509 public key certificate format and commonly used ASN.1 encoding is overly verbose for constrained IoT environments. Profiling together with CBOR encoding can reduces certificate sizes by XX%.

--- middle

# Introduction  {#intro}

One of the challenges with deploying a Public Key Infrastructure (PKI) for the Internet of Things (IoT) is the size and encoding of public key certificates, since those are not optimized for constrained environments {{RFC7228}}. More compact certificate representations are desireable. Due to the current PKI usage of X.509 certificates, keeping X.509 compatibility is ___

CBOR {{RFC7049}}

DTLS {{RFC6347}}

EDHOC {{I-D.selander-ace-cose-ecdhe}}


* Terminology   {#terminology}

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "NOT RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in BCP 14 {{RFC2119}} {{RFC8174}} when, and only when, they appear in all capitals, as shown here.

This specification makes use of the following terminology:  

IoT device: A resource constrained device with Internet connectivity.

# IoT certificate profile

* Version number  
The X.509 standard has not moved beyond version 3 since 2008. With the introduction of certificate extensions new certificate fields can be added without breaking the format, making version changes less likely. Therefore this IoT profile fixes the version number to 3.
* Serial number  
The serial number together with the identity of the CA is the unique identifier of a certificate. The X.509 standard does not specify the signedness of the serial number, but the IoT profile requires an unsigned integer.
* Issuer  
Used to identify the issuing CA through a sequence of name-value pairs. The IoT profile is restricting this to one pair, common name and associated string value. The requirement is that the common name must uniquely identify the CA.
* Validity  
Requiring the use of UTCTime-format, YYMMDDhhmmss, gives the most compact representation.
* Subject  
The subject section has the same format as the issuer, identifying the receiver of the public key through a sequence of name-value pairs. This sequence is in the profile restricted to a single pair, subject name and associated (unique) value. For an IoT-device, the MAC-derived EUI-64 serves this purpose well.
* Subject public key info  
For the IoT devices, elliptic curve cryptography based algorithms have clear advantages. For the IoT profile the public key algorithm is fixed to _prime256v1_.
* Extension  
To maintain forward compatibility, the IoT profile does not restrict the use of extensions. By the X.509-standard, any device must be able to process eight extensions types. Since only four of them are critical for IoT, this profile is making the other four optional. Still mandatory to be understood are:
  * Key Usage
  * Subject Alternative Name
  * Basic Constraints
  * Extended Key Usage
* Signature algorithm  
Fixed to ECDSA with SHA256.
* Signature  
The field corresponding to the signature done by by the CA private key. For the IoT-profile, this is restricted to ECDSA signatures.

# Profile encoding and compression
By combining the use of the CBOR encoding instead of ASN.1 and base64 encoding together with the knowledge of the IoT profile restrictions, the resulting IoT certificates size can be reduced with more than 50%.


* Version number  
Assuming a fixed version 3 flag, the field is omitted in the encoding.
* Serial number  
With no known structure, the only savings come from cbor encoding.
* Issuer  
With the restriction of only allowing common name, the common name can be omitted. The overhead goes from 13 bytes to one byte.
* Validity  
The time is encoded as UnixTime. This reduces the size from 32 to 11 bytes for a 'not before'-'not after' validity pair.
* Subject  
A subject identified by a EUI-64, based on a 48bit unique MAC id, can be encoded with only 7 bytes using CBOR. This is a reduction down from 36 bytes for a corresponding ASN.1 encoding.

* Subject public key info  
The algorithm identifier is known from the profile restrictions and can be omitted. One of the public key ECC curve point elements can be calculated from the other, hence only one the needed. These actions together reduce size from 91 to 35 bytes.

* Extension  
Some minor savings can be achieved by the more compact CBOR encoding.

* Signature algorithm  
Since this is fixed by the profile restrictions, it can be omitted, saving 12 bytes.

* Signature  
By omitting unneeded ASN.1 information, the overhead for sending the two 32-bit values is reduced from 11 to two bytes.

# Expected certificate sizes


See {{fig-table}}. 

~~~~~~~~~~~
[//]: # (WIP Savings for profile: mainly extra issuer and subject info: -11 -18 -11 -18)
[//]: # (WIP Savings for compression: id 5, alg 12, subj 27, val. 21, issuer 12, publ key 56, s-alg 12, sig 9)

+---------------------------------------------------------------+
|                 |   X.509    | X.509 profiled | X.509 encoded |
+---------------------------------------------------------------+
| IoT certificate |    450     |      392       |      238      |
+---------------------------------------------------------------+
|                 |            |                |               |
+---------------------------------------------------------------+
~~~~~~~~~~~
{: #fig-table title="Table"}
{: artwork-align="center"}

# Certificate decoding architecture
For the currently used DTLS v1.2 protocol, where the handshake is sent unencrypted, the actual encoding and compression can be done at a 6lowpan border gateway which allows the server side to stay unmodified. For DTLS v1.3 the encoding needs to be done fully end-to-end, through adding the endcoding/decoding functionality to the server.


# Security Considerations  {#sec-cons}

TBD

# Privacy Considerations 

TBD

# IANA Considerations  {#iana}

TBD

# Acknowledgments 
{: numbered="no"}

The following people have contributed to this document: 


--- back

# Appendix {#app} Certificate Field Size Calculations

Breakdown of field savings

--- fluff

