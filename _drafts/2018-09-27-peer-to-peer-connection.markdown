---
layout: post
title:  Peer to peer connection
date: '2018-09-27 21:39:00'
tags:
- p2p
---
Before ipv6, due to limit of number of ipv4, network segmented into public and private network. Public network are computers visible to any connected computers. Private network are computers behind gate keeper (NAT server, can be from ISP, corporate network), to which computers outside network can't initiate connection to. 

Consider 2 computers behind 2 different NAT, for hole punching to work:
Key points:
- NAT assign a temporary public endpoint (nat_public_ip:public_port) to an computer that initiate outgoing connection to a rendevous server R
- rendevous server returns public endpoint of one peer to another and vice versa.
- each client then init a connection to the other peer public endpoint. The counter party NAT may drop incoming as unsolicited request. However, if other party before that send out a request to the sender public endpoint as well, counter party NAT allows that ( request is solicited by a prior request). And a hole is punched.

How bittorrent client connect to peers which are not publicly available.


References
This is a highly recommended read: http://www.brynosaurus.com/pub/net/p2pnat/
