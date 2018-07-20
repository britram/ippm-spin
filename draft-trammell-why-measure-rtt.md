---
title: Why do we need passive measurement of round trip time?
abbrev: Why measure RTT?
docname: draft-trammell-why-measure-rtt
date:
category: info

ipr: trust200902
workgroup: QUIC
keyword: Internet-Draft

stand_alone: yes
pi: [toc, sortrefs, symrefs]

author:
  -
    ins: B. Trammell
    name: Brian Trammell
    org: ETH Zurich
    email: ietf@trammell.ch

normative:
  QUIC-SPIN-EXP:
    title: "The QUIC Latency Spin Bit"
    date: {DATE}
    seriesinfo:
      Internet-Draft: draft-ietf-quic-spin-exp
    author:
      -
        ins: B. Trammell
      -
        ins: M. Kuehlewind

    author:
      -
        ins: B. Trammell
      -
        ins: M. Kuehlewind


informative:
  TRILAT:
    title: On the Suitability of RTT Measurements for Geolocation (https://github.com/britram/trilateration/blob/paper-rev-1/paper.ipynb)
    author:
      -
        ins: B. Trammell
    date: 2017-08-30
  TOKYO-PING:
    title: From Paris to Tokyo - On the Suitability of ping to Measure Latency (In Proc. ACM IMC 2014)
    author:
      -
        ins: C. Pelsser
      -
        ins: L. Cittadini
      -
        ins: S. Vissicchio
      -
        ins: R. Bush
    date: 2014-10-23
  CARRA-RTT:
    title: Passive Online RTT Estimation for Flow-Aware Routers Using One-Way Traffic (NETWORKING 2010, LNCS 6091, pp. 109–121)
    author:
      -
        ins: D. Carra
      -
        ins: K. Avrachenkov
      -
        ins: S. Alouf
      -
        ins: A. Blanc
      -
        ins: P. Nain
      -
        ins: G. Post
    date: 2010
  CONUS:
    title: Comparison of Backbone Node RTT and Great Circle Distances (https://github.com/acmacm/CONUS-RTT)
    author:
      -
        ins: A. Morton
    date: 2017-09-01
  NOSPIN:
    title: Description of a tool chain to evaluate Unidirectional Passive RTT measurement (and results) (https://github.com/acmacm/PassiveRTT)
    author:
      -
        ins: A. Morton
    date: 2017-10-05
  SPINBIT-REPORT:
    title: Latency Spinbit Implementation Experience (https://devae.re/f/eth/quic/spinbit_report/)
    author:
      -
        ins: P. De Vaere
    date: 2017-11-28
  MINQ:
    title: MINQ, a simple Go implementation of QUIC (https://github.com/ekr/minq)
    author:
      -
        ins: E. Rescorla
    date: 2017-11-28
  MOKUMOKUREN:
    title: Mokumokuren, a lightweight flow meter using gopacket (https://github.com/britram/mokumokuren)
    author:
      -
        ins: B. Trammell
    date: 2017-11-12
  IMC-CONGESTION:
    title: Challenges in Inferring Internet Interdomain Congestion (in Proc. ACM IMC 2014)
    author:
      -
        ins: M. Luckie
      -
        ins: A. Dhamdhere
      -
        ins: D. Clark
      -
        ins: B. Huffaker
      -
        ins: k. claffy
    date: 2014-11
  IMC-TCPSIG:
    title: TCP Congestion Signatures (in Proc. ACM IMC 2017)
    author:
      -
        ins: S. Sundaresan
      -
        ins: A. Dhamdhere
      -
        ins: M. Allman
      -
        ins: k claffy
  CACM-TCP:
    title: Passively Measuring TCP Round-Trip Times (in Communications of the ACM)
    author:
      -
        ins: S. Strowes
    date: 2013-10
  SHBAIR:
    title: A multi-level framework to identify HTTPS services (in Proc. IEEE/IFIP NOMS)
    author:
      -
        ins: W. M. Shbair
      -
        ins: T. Cholez
      -
        ins: J. Francois
      -
        ins: I. Chrisment
    date: 2016-04
  TMA-QOF:
    title: Inline Data Integrity Signals for Passive Measurement (in Proc. TMA 2014)
    author:
      -
        ins: B. Trammell
      -
        ins: D. Gugelmann
      -
        ins: N. Brownlee
    date: 2014-04
  WWMM-BLOAT:
    title: Impact of TCP Congestion Control on Bufferbloat in Cellular Networks (in Proc. IEEE WoWMoM 2013)
    author:
      -
        ins: S. Alfredsson
      -
        ins: G. Giudice
      -
        ins: J. Garcia
      -
        ins: A. Brunstrom
      -
        ins: L. Cicco
      -
        ins: S. Mascolo
    date: 2013-06

--- abstract

This document describes the utility of passive latency measurement, both for
the generation of latency metrics, as well as for other measurement tasks,
when passive latency measurement is the only facility available for
measurement. It additionally discusses other metrics derivable from the
transport-independent latency spin signal defined in
{{!TSVWG-SPIN=I-D.trammell-tsvwg-spin}}.

--- middle

# Introduction

\[frontmatter goes here]


## About This Document

This document is maintained in the GitHub repository
https://github.com/britram/draft-trammell-tsvwg-spin, and the editor's copy is
available online at https://britram.github.io/draft-trammell-tsvwg-spin.
Current open issues on the document can be seen at
https://github.com/britram/draft-trammell-tsvwg-spin/issues. Comments and
suggestions on this document can be made by filing an issue there, or by
contacting the editor.

This document is based in part on [?QUIC-SPIN=I-D.trammell-quic-spin],
however, aside from {{more-spin}}, it is not specific to the spin bit
proposal.

# Direct Utility of Passive RTT Measurement {#use-cases}

RTT measurement generates two-way latency metric samples; these samples are
useful in many measurement tasks which directly require latency data. The
measurement methodologies using two-way latency measurement samples follow
one of a few basic variants:

- The RTT evolution of a flow or a set of flows can be compared to baseline or
  expected RTT measurements for flows with the same characteristics in order
  to detect or localize latency issues in a specific network.

- The RTT evolution of a single flow can also be examined in detail to
  diagnose performance issues with that flow.

- Samples of RTT for a flow aggregate (e.g., all flows between two given
  networks) can be used without regard to temporal evolution of the RTT, in
  order to examine the distribution of RTTs for a group of flows that should
  have similar RTT (e.g., because they should share the same path(s)).

## Inter-domain Troubleshooting

Network access providers are often the first point of contact by their
customers when network problems impact the performance of bandwidth-intensive
and latency-sensitive applications such as video, regardless of whether the
root cause lies within the access provider's network, the service provider's
network, on the Internet paths between them, or within the customer's own
network.

Points on path can extract spatial delay metric samples {{?RFC6049}} from
fields of the transport layer (e.g. TCP) or application layer (e.g. RTP). The
information is captured in the upper layer because neither the IP header nor
the UDP layer includes fields allowing the measurement of upstream and
downstream delay.

Local network performance problems are detected with monitoring tools which
observe the variation of upstream latency and downstream latency.

Inter-domain troubleshooting relies on the same metrics but is not a proactive
task; instead, it is a recursive process which hones in on the domain and link
responsible for the failure. In practice, inter-domain troubleshooting is a
communication process between the Network Operations Center (NOC) teams of the
networks on the path, because the root cause of a problem is rarely located on
a single network, and requires cooperation and exchange of data between the
NOCs.

One example is the troubleshooting performance degradation resulting from a
change of routing policy on one side of the path which increases queueing on
the other side of the path.

## Bufferbloat Mitigation in Cellular Networks

Cellular networks consist of multiple Radio Access Networks (RAN) where
mobile devices are attached to base stations. It is common that base stations
from different vendors and different generations are deployed in the same
cellular network.

Due to the dynamic nature of RANs, base stations have typically been
provisioned with large buffers to maximize throughput despite rapid changes in
capacity. As a side effect, bufferbloat has become a common issue in such
networks {{WWMM-BLOAT}}.

An effective way of mitigating bufferbloat without sacrificing too much
throughput is to deploy Active Queue Management (AQM) in bottleneck routers
and base stations. However, due to the variation in deployed base-stations it
is not always possible to enable AQM at the bottlenecks, without massive
infrastructure investments.

An alternative approach is to deploy AQM as a network function in a more
centralized location than the traditional bottleneck nodes. Such an AQM
monitors the RTT progression of flows and drops or marks packets when the
measured latency is indicative of congestion. Such a function also has the
possibility to detect misbehaving flows and reduce the negative impact they
have on the network.

## Locating WiFi Problems in Home Networks

Many residential networks use WiFi (802.11) on the last segment, and WiFi
signal strength degradation manifests in high first-hop delay, due to the fact
that the MAC layer will retransmit packets lost at that layer. Measuring the
RTT between endpoints on the customer network and parts of the service
provider's own infrastructure (which have predictable delay characteristics)
can be used to isolate this cause of performance problems.

The network provider can measure the RTT at the home gateway, or at an
upstream point if there is no access to home gateway. A problem in the WiFi
network is identified by seeing high delay and low packet loss.

These measurements are particularly useful for traffic which is latency
sensitive, such as interactive video applications. However, since high latency
is often correlated with other network-layer issues such as chronic
interconnect congestion {{IMC-CONGESTION}}, it is useful for general
troubleshooting of network layer issues in an interdomain setting.

In this case, multiple RTT samples per flow are useful less for observing
intraflow behavior, and more for generating sufficient samples for a given
aggregate to make a high-quality measurement.

## Internet Measurement Research

As a large, distributed, engineered system with no centralized control, the
Internet has emergent properties of interest to the research community not
just for purely scientific curiosity, but also to provide applicable guidance
to Internet engineering, Internet protocol design and development, network
operations, and policy development. Latency measurements in particular are
both an active area of research as well as an important tool for certain
measurement studies (see, e.g. {{IMC-TCPSIG}}, from the most recent Internet
Measurement Conference). While much of this work is currently done with active
measurements, the ability to generate latency samples passively or using a
hybrid measurement approach (i.e., through passive observation of
purpose-generated active measurement traffic; see {{?RFC7799}}) can
drastically increase the efficiency and scalability of these studies.

# Indirect Utility of RTT Measurements {#wireshark}

In addition to the direct generation of RTT metric samples, RTT measurement
can also be used for indirect generation of other metrics when more direct
means are not available.

## Intraflow performance troubleshooting

A variety of tools are used for detailed troubleshooting of the performance of
single flows, both for debugging transport- and application-layer protocol
implementations, as well as to determine whether a particular end-to-end
performance issue is related to particular network conditions. One common type
of visualization  used for TCP (implemented, for example, in the TCP Stream
Graphs feature of Wireshark, https://www.wireshark.org/) shows the development
over time of the sequence and acknowledgment numbers, including
retransmissions, and the evolution of the inflight and receiver flow control
windows over time. By analyzing the relationship among loss, latency, and
throughput, the precise cause of an observed performance on a given flow can
be determined.

While RTT measurements on their own are not enough to drive such a
visualization, many similar techniques can be built on high-resolution time
series RTT data. Here we exploit two properties of transport protocols:

- The size of the inflight window is equal to the number of bytes/packets sent
  per RTT, so inflight window evolution can be generated at each RTT sample r
  at t, and summing the number of bytes/packets sent between t - r and t.

- Changes in the inflight window are can be related to sender reactions to
  congestion. For common loss- and ECN-based congestion control protocols such
  as NewReno {{?RFC6582}} and Cubic {{?RFC8312}}, inflight window reductions
  are correlated with sender-experienced congestion or loss.

Inflight window evolution over time, together with heuristic assumptions about
server behavior, can go a long way toward replacing direct visibility of
transport protocol dynamics (sequence and acknowledgment number seqence over
time) for encrypted transports; the exact details of this are a subject of
present and future research.


# Additional Metrics Derivable from the Spin Bit {#more-spin}

The latency spin signal mechanism itself {{TSVWG-SPIN}} has additional
measurement utility; these observations do not apply to other methodologies
for measuring RTT.

## Derived Loss and Reordering

When used alone (as a one-bit signal), measurement systems using the latency
spin bit must use heuristics to reject samples which are potentially-lost,
potentially-reordered, or potentially-delayed. When these heuristics are
instrumented to note their sample rejection rate, this rate itself is a
potentially-useful proxy metric for "difficulty" (vaguely defined) experienced
by a flow.

When the latency signal is used with the Valid Edge Counter (VEC), additional
information is available in the wire image to reject samples due to loss,
delay, or reordering. Analysis of the VEC together with the series of spin bit
values can be used to recognize single loss and reordering events, which can
be used to generate loss and reordering metrics at the resolution of the
flow's round trip time. Optimal use of the VEC signal to generate loss and
reordering metric signals is a subject of ongoing research.

## Two-Point Intradomain Measurement

The spin bit is also useful as a basic signal for instantaneous measurement of
the treatment of traffic carrying the latency spin signal within a single
network. Though the primary design goal of the spin bit signal is to enable
single-observer on-path measurement of end-to-end RTT, the spin bit can also
be used by two cooperating observers with access to traffic flowing in the
same direction as an alternate marking signal, as described in
{{?ALT-MARK=I-D.ietf-ippm-alt-mark}}. The only difference from alternate
marking with a generated signal is that the size of the alternation will
change with the flight size each RTT. However, these changes do not affect the
applicability of the method that works for each marking batch separately
applied between two measurement points on the same direction. This two point
measurement is an additional feature enabled "for free" by the spin bit
signal.

So, with more than one observer on the same direction, it can be useful to
segment the RTT and deduce the contribution to the RTT of the portion of the
network between two on-path observers. This can be easily performed by
calculating the delay between two or more measurement points on a single
direction by applying {{ALT-MARK}}. In this way, packet loss, delay and delay
variation can be measured for each segment of the network depending on the
number and distribution of the available on-path observation points. When
these observation points are applied at network borders, the alternate-marking
signal can be used to measure the performance of QUIC traffic within a network
operator's own domain of responsibility. own portion of the network.

# Contributors

This document contains text from [QUIC-SPIN], which is the work of the
following authors in addition to the editor of this document:

- Piet De Vaere, ETH Zurich
- Roni Even, Huawei
- Giuseppe Fioccola, Telecom Italia
- Thomas Fossati, Nokia
- Marcus Ihlar, Ericsson
- Al Morton, AT&T Labs
- Emile Stephan, Orange

# Acknowledgments

Thanks to Mark Nottingham for suggesting that this document should exist.

This work is partially supported by the European Commission under Horizon 2020
grant agreement no. 688421 Measurement and Architecture for a Middleboxed
Internet (MAMI), and by the Swiss State Secretariat for Education, Research,
and Innovation under contract no. 15.0268. This support does not imply
endorsement.