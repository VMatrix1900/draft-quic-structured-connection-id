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

{{arch}} shows the architecture of the optimization intermediary. The sender, which can be either the client or server based on the direction of communication, incorporates metadata into the connection ID field as outlined in the referenced section (See {{format}}). This metadata allows the intermediary to execute optimizations tailored to the information provided. Given that various applications may require the disclosure of distinct metadata to the intermediary, a standardized template is adopted to specify the metadata's content and structure. There are two primary methods for obtaining this template:

1. For each category of application, a specific template is crafted and cataloged within a new IANA registry. This approach leverages the global accessibility of the template definition, eliminating the need for its distribution by the configuration agent. The responsibility for developing these templates falls to the respective working groups or documents, which is beyond the scope of this document.
2. The configuration agent, operating within its domain, defines and disseminates the template. This strategy ensures the template's relevance and effectiveness is confined to the domain under the agent's control, tailored according to the capabilities of the network devices present.

If the network between the intermediary and endpoints is not trusted, the metadata MUST be encrypted. In such scenarios, the encryption parameters must be exclusively shared with authenticated intermediaries, potentially via the configuration agent. A viable encryption strategy might involve adopting the algorithm proposed in {{?I-D.ietf-quic-load-balancers}}, ensuring the security of the metadata.

# Structured Connection ID {#format}
~~~
Structured Connection ID {
  Config Parameters (8),
  Metadata (40...152),
}
~~~
{: #cid-format title="Format of structured CID"}

The format of the structured connection ID is shown in {{cid-format}}. The content and the format of the metadata field are defined by a template, carrying the information such as media characteristics in {{Section 3.1 of ?I-D.ietf-avtext-framemarking}}, the service requirement such as delay and importance in {{Section 3 of ?I-D.kaippallimalil-tsvwg-media-hdr-wireless-04}}.

# Coexistence with QUIC Load Balancer

If an intermediary serves dual roles as both the load balancer and the optimization node, and if both entities are underpinned by a unified trust relationship, then it is feasible to consolidate the Metadata and the Server ID specified in {{?I-D.ietf-quic-load-balancers}}. This consolidation allows for the utilization of a singular Config Parameter and a shared encryption/decryption methodology. 

Conversely, if the load balancer and the optimization node are separated, the Server ID and the Metadata needs to be segregated too. One option is to split the CID into two segments: one for the Server ID and the other for the metadata. Each segment would be governed by its own set of Config Parameters and subjected to individual encryption protocols, ensuring the integrity and segregation of the transmitted information.


# Security Considerations

TBD
