# Using TLS to Secure QUIC(draft-ietf-quic-tls-19)

```
QUIC                                                     M. Thomson, Ed.
Internet-Draft                                                   Mozilla
Intended status: Standards Track                          S. Turner, Ed.
Expires: September 12, 2019                                        sn3rd
                                                          March 11, 2019


                        Using TLS to Secure QUIC
                         draft-ietf-quic-tls-19

```
## Abstract

   This document describes how Transport Layer Security (TLS) is used to
   secure QUIC.

## Note to Readers

   Discussion of this draft takes place on the QUIC working group
   mailing list (quic@ietf.org), which is archived at
   https://mailarchive.ietf.org/arch/search/?email_list=quic [1].

   Working Group information can be found at https://github.com/quicwg
   [2]; source code and issues list for this draft can be found at
   https://github.com/quicwg/base-drafts/labels/-tls [3].

## Status of This Memo

   This Internet-Draft is submitted in full conformance with the
   provisions of BCP 78 and BCP 79.

   Internet-Drafts are working documents of the Internet Engineering
   Task Force (IETF).  Note that other groups may also distribute
   working documents as Internet-Drafts.  The list of current Internet-
   Drafts is at https://datatracker.ietf.org/drafts/current/.

   Internet-Drafts are draft documents valid for a maximum of six months
   and may be updated, replaced, or obsoleted by other documents at any
   time.  It is inappropriate to use Internet-Drafts as reference
   material or to cite them other than as "work in progress."

   This Internet-Draft will expire on September 12, 2019.

## Copyright Notice

   Copyright (c) 2019 IETF Trust and the persons identified as the
   document authors.  All rights reserved.


   This document is subject to BCP 78 and the IETF Trust's Legal
   Provisions Relating to IETF Documents
   (https://trustee.ietf.org/license-info) in effect on the date of
   publication of this document.  Please review these documents
   carefully, as they describe your rights and restrictions with respect
   to this document.  Code Components extracted from this document must
   include Simplified BSD License text as described in Section 4.e of
   the Trust Legal Provisions and are provided without warranty as
   described in the Simplified BSD License.

## Table of Contents
```
   1.  Introduction  . . . . . . . . . . . . . . . . . . . . . . . .   3
   2.  Notational Conventions  . . . . . . . . . . . . . . . . . . .   4
     2.1.  TLS Overview  . . . . . . . . . . . . . . . . . . . . . .   4
   3.  Protocol Overview . . . . . . . . . . . . . . . . . . . . . .   6
   4.  Carrying TLS Messages . . . . . . . . . . . . . . . . . . . .   7
     4.1.  Interface to TLS  . . . . . . . . . . . . . . . . . . . .   9
       4.1.1.  Sending and Receiving Handshake Messages  . . . . . .   9
       4.1.2.  Encryption Level Changes  . . . . . . . . . . . . . .  11
       4.1.3.  TLS Interface Summary . . . . . . . . . . . . . . . .  12
     4.2.  TLS Version . . . . . . . . . . . . . . . . . . . . . . .  13
     4.3.  ClientHello Size  . . . . . . . . . . . . . . . . . . . .  14
     4.4.  Peer Authentication . . . . . . . . . . . . . . . . . . .  14
     4.5.  Enabling 0-RTT  . . . . . . . . . . . . . . . . . . . . .  15
     4.6.  Rejecting 0-RTT . . . . . . . . . . . . . . . . . . . . .  15
     4.7.  HelloRetryRequest . . . . . . . . . . . . . . . . . . . .  15
     4.8.  TLS Errors  . . . . . . . . . . . . . . . . . . . . . . .  16
     4.9.  Discarding Unused Keys  . . . . . . . . . . . . . . . . .  16
     4.10. Discarding Initial Keys . . . . . . . . . . . . . . . . .  17
   5.  Packet Protection . . . . . . . . . . . . . . . . . . . . . .  18
     5.1.  Packet Protection Keys  . . . . . . . . . . . . . . . . .  18
     5.2.  Initial Secrets . . . . . . . . . . . . . . . . . . . . .  18
     5.3.  AEAD Usage  . . . . . . . . . . . . . . . . . . . . . . .  19
     5.4.  Header Protection . . . . . . . . . . . . . . . . . . . .  20
       5.4.1.  Header Protection Application . . . . . . . . . . . .  21
       5.4.2.  Header Protection Sample  . . . . . . . . . . . . . .  22
       5.4.3.  AES-Based Header Protection . . . . . . . . . . . . .  23
       5.4.4.  ChaCha20-Based Header Protection  . . . . . . . . . .  24
     5.5.  Receiving Protected Packets . . . . . . . . . . . . . . .  24
     5.6.  Use of 0-RTT Keys . . . . . . . . . . . . . . . . . . . .  24
     5.7.  Receiving Out-of-Order Protected Frames . . . . . . . . .  25
   6.  Key Update  . . . . . . . . . . . . . . . . . . . . . . . . .  25
   7.  Security of Initial Messages  . . . . . . . . . . . . . . . .  27
   8.  QUIC-Specific Additions to the TLS Handshake  . . . . . . . .  28
     8.1.  Protocol and Version Negotiation  . . . . . . . . . . . .  28
     8.2.  QUIC Transport Parameters Extension . . . . . . . . . . .  28
     8.3.  Removing the EndOfEarlyData Message . . . . . . . . . . .  29

   9.  Security Considerations . . . . . . . . . . . . . . . . . . .  29
     9.1.  Replay Attacks with 0-RTT . . . . . . . . . . . . . . . .  29
     9.2.  Packet Reflection Attack Mitigation . . . . . . . . . . .  30
     9.3.  Peer Denial of Service  . . . . . . . . . . . . . . . . .  31
     9.4.  Header Protection Analysis  . . . . . . . . . . . . . . .  31
     9.5.  Key Diversity . . . . . . . . . . . . . . . . . . . . . .  32
   10. IANA Considerations . . . . . . . . . . . . . . . . . . . . .  33
   11. References  . . . . . . . . . . . . . . . . . . . . . . . . .  33
     11.1.  Normative References . . . . . . . . . . . . . . . . . .  33
     11.2.  Informative References . . . . . . . . . . . . . . . . .  34
     11.3.  URIs . . . . . . . . . . . . . . . . . . . . . . . . . .  35
   Appendix A.  Sample Initial Packet Protection . . . . . . . . . .  35
     A.1.  Keys  . . . . . . . . . . . . . . . . . . . . . . . . . .  35
     A.2.  Client Initial  . . . . . . . . . . . . . . . . . . . . .  36
     A.3.  Server Initial  . . . . . . . . . . . . . . . . . . . . .  38
   Appendix B.  Change Log . . . . . . . . . . . . . . . . . . . . .  39
     B.1.  Since draft-ietf-quic-tls-18  . . . . . . . . . . . . . .  39
     B.2.  Since draft-ietf-quic-tls-17  . . . . . . . . . . . . . .  39
     B.3.  Since draft-ietf-quic-tls-14  . . . . . . . . . . . . . .  39
     B.4.  Since draft-ietf-quic-tls-13  . . . . . . . . . . . . . .  40
     B.5.  Since draft-ietf-quic-tls-12  . . . . . . . . . . . . . .  40
     B.6.  Since draft-ietf-quic-tls-11  . . . . . . . . . . . . . .  40
     B.7.  Since draft-ietf-quic-tls-10  . . . . . . . . . . . . . .  40
     B.8.  Since draft-ietf-quic-tls-09  . . . . . . . . . . . . . .  41
     B.9.  Since draft-ietf-quic-tls-08  . . . . . . . . . . . . . .  41
     B.10. Since draft-ietf-quic-tls-07  . . . . . . . . . . . . . .  41
     B.11. Since draft-ietf-quic-tls-05  . . . . . . . . . . . . . .  41
     B.12. Since draft-ietf-quic-tls-04  . . . . . . . . . . . . . .  41
     B.13. Since draft-ietf-quic-tls-03  . . . . . . . . . . . . . .  41
     B.14. Since draft-ietf-quic-tls-02  . . . . . . . . . . . . . .  41
     B.15. Since draft-ietf-quic-tls-01  . . . . . . . . . . . . . .  41
     B.16. Since draft-ietf-quic-tls-00  . . . . . . . . . . . . . .  42
     B.17. Since draft-thomson-quic-tls-01 . . . . . . . . . . . . .  42
   Acknowledgments . . . . . . . . . . . . . . . . . . . . . . . . .  42
   Contributors  . . . . . . . . . . . . . . . . . . . . . . . . . .  42
   Authors' Addresses  . . . . . . . . . . . . . . . . . . . . . . .  42
```
## 1.  Introduction
本文描述QUIC[QUIC传输协议](https://tools.ietf.org/html/draft-ietf-quic-transport-19)如何通过TSL[TSL13](https://tools.ietf.org/html/rfc8446)保证安全。    
相比之前的版本，TLS 1.3为连接建立提供了关键的延迟改进。不考虑丢包的情况下，大部分连接可以在一个来回（1-RTT）内建立和保护。在同一客户机和服务器之间的后续连接上，客户机通常可以立即发送应用程序数据，也即使用0-RTT。  
本文档描述了tls如何充当quic的安全组件。  
## 2.  符号约定
本文件使用了[QUIC传输协议](https://tools.ietf.org/html/draft-ietf-quic-transport-19)中规定的术语。  
为了简洁起见，缩写tls用于引用tls 1.3，尽管可以使用较新版本（见第4.2节)。  
### 2.1.  TLS 概述
TLS为两个端点在不可信媒介（互联网）上建立通信提供了方法，确保信息交换无法被观察、修改或伪造。  
在内部，tls是一个分层协议，其结构如图所示以下：  
```
   +--------------+--------------+--------------+
   |  Handshake   |    Alerts    |  Application |
   |    Layer     |              |     Data     |
   |              |              |              |
   +--------------+--------------+--------------+
   |                                            |
   |               Record Layer                 |
   |                                            |
   +--------------------------------------------+
```
每个上层（握手、警报和应用程序数据）都被当作一个类型化的TLS记录序列携带。记录是单独加密保护的，然后通过可靠传输（典型的比如TCP）保证有序和可靠。  

不能在quic中发送Change Cipher Spec（更改密码规格）记录。
　>Change Cipher Spec协议独立于握手协议，单独属于一类，也是其中最简单的一个。协议由单个消息组成 , 该消息只包含一个值为 1 的单个字节。该消息由客户端和服务器端各自发出用来通知对方，从这个消息以后要开始使用之前协商好的密钥套件了，这个消息一般是在握手到发出 Finish 消息之前发出。  

TLS身份认证的密钥交换在客户端和服务器两个实体中发生。client启动Exchange，server响应。如果密钥交换成功完成，client和server会同意一个密码。TLS支持两个预共享密钥（PSK）和diffie-hellman（DH）密钥交换。PSK是0-RTT的基础，在密钥被销毁后DH保证完全的前向保密（perfect forward secrecy，PFS） 
 >完全的前向保密（perfect forward secrecy，PFS），简单来说就是密钥泄漏出去，不会造成之前通讯时使用的会话密钥泄漏，不会暴露之前的通信内容。PFS不能保证泄漏后的通信安全，但是要保证泄漏前的通信安全。的安全要求一个密钥只能访问由它所保护的数据；用来产生密钥的元素一次一换，不能再产生其他的密钥；一个密钥被破解，并不影响其他密钥的安全性。  

   
