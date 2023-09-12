---
title: "Structured Connection ID Carrying Metadata"
abbrev: "Structured Connection ID"
category: std

docname: draft-shi-quic-structured-connection-id-latest
submissiontype: IETF  # also: "independent", "IAB", or "IRTF"
number:
date:
consensus: true
v: 3
area: "Transport"
workgroup: "QUIC"
keyword:
 - Internet-Draft
venue:
  group: "QUIC"
  type: "Working Group"
  arch: "https://mailarchive.ietf.org/arch/browse/quic/"
  github: "VMatrix1900/draft-quic-structured-connection-id"
  latest: "https://VMatrix1900.github.io/draft-quic-structured-connection-id/draft-shi-quic-structured-connection-id.html"

author:
   -
     ins: H. Shi
     name: Hang Shi
     organization: Huawei Technologies
     email: shihang9@huawei.com
     country: China

normative:

informative:

...

--- abstract

This document describes a mechanism to carry the metadata in the QUIC connection ID so that the intermediary can perform optimization.

--- middle

# Introduction

Nowadays, media applications are usually able to dynamically adjust the size and quality of the stream to adapt to fluctuating network conditions. However, for the high throughput and low latency media traffic, adaptation only by the endpoint is not good enough, especially when the network condition is challenging, such as the wireless networks discussed in {{?I-D.kaippallimalil-tsvwg-media-hdr-wireless}} and {{?I-D.defoy-moq-relay-network-handling}}. To this end, it is desirable to have the intermediary performing optimization for the endpoint. For example, low-priority packets can be dropped to save the resource when the network is congested.

One example of such an intermediary is the relay in the Media over QUIC working group. To quote the charter from the MoQ working group. "Media over QUIC (moq) will develop a simple low-latency media delivery solution for ingest and distribution of media. This solution addresses use cases including live streaming, gaming, and media conferencing and will scale efficiently." "Even when media content is end-to-end encrypted, the relays can access metadata needed for caching (such as timestamp), making media forwarding decisions (such as drop or delay under congestion), and so on."

Due to the end-to-end encryption of the QUIC, the intermediary does not have the necessary metadata to perform optimization. A similar problem exists when the media is encrypted and transferred using SRTP {{!RFC3711}}. To solve the problem, {{?I-D.ietf-avtext-framemarking}} defines an extension of the RTP header containing the video frame information. This document defines an extension of the QUIC header, using the connection ID to carry the necessary metadata. To mitigate the linkability between the multiple connection IDs of the same connection and protect privacy, the metadata MAY be encrypted and only decrypted by an authenticated intermediary. Similar to {{?I-D.ietf-quic-load-balancers}}, a configuration agent is used to distribute the encryption parameters and the template of the metadata.

# Terminology

This document uses terms in the {{?I-D.ietf-quic-load-balancers}}:

- "client" and "server" refer to the QUIC endpoint.
- Intermediary refers to a network element that forwards QUIC packets and does not possess the QUIC connection keys. Such an intermediary can be QUIC proxy defined in the MASQUE working group, wireless node described in the {{?I-D.kaippallimalil-tsvwg-media-hdr-wireless}}, and relay defined in the Media over QUIC working group.
- CID: Connection ID in the QUIC header.
- Configuration agent: An entity that distributes the encryption parameter and the template of the metadata field.

All wire formats will be depicted using the notation defined in {{Section 1.3 of !RFC9000}}.

## Requirements Language

{::boilerplate bcp14-tagged}

# Architecture

~~~
                             + --------------+
                             | Configuration |
         +-------------------+     agent     +-------------------+
        /                    +------+--------+                    \
       /Config Parameters and template of the Metadata field in CID\
      /                             |                               \
     /          _______             |              _______           \
+---V----+     (       )     +------v-------+     (       )     +-----v----+
| Client +----( Network )----+ Intermediary +----( Network )----+  Server  |
+--------+     (_______)     +--------------+     (_______)     +----------+

~~~
{: #arch title="Architecture of the intermediary"}

{{arch}} shows the architecture of the optimization intermediary. The sender endpoint encodes the metadata into the connection ID field (See {{format}}). The intermediary performs the related optimization based on the metadata. Since different applications may need to expose different metadata to the intermediary, a template is used to define the content and the format of metadata. The template is determined and distributed by a configuration agent. If the network between the intermediary and endpoints is not trusted by endpoints, the metadata MAY be encrypted. In this case, the parameter for encryption MUST be shared only with the authenticated intermediary through the configuration agent. The means of authentication and the distribution of these parameters and template is not in the scope of this document.

# Structured Connection ID {#format}
~~~
Structured Connection ID {
  Config Parameters (8),
  Metadata (40...152),
}
~~~
{: #cid-format title="Format of structured CID"}

The format of the structured connection ID is shown in {{cid-format}}. The content and the format of the metadata field are defined by a template, carrying the information such as media characteristics in {{Section 3.1 of ?I-D.ietf-avtext-framemarking}}, the service requirement such as delay and importance in {{Section 5 of ?I-D.kaippallimalil-tsvwg-media-hdr-wireless-01}}.

If an intermediary acts as both the load balancer and the optimization point and they share the same trust relationship, the Metadata and the Server ID defined in {{?I-D.ietf-quic-load-balancers}} can be put together and share the same Config Parameter. Otherwise, if a QUIC connection goes through both the load balancer and optimization point, an additional mechanism is needed for the coexistence of the metadata and the Server ID. The details will be worked out in the later version.

# Security Considerations

TBD
