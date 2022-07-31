---
layout: post
title: 'Tutorial: A scan chain attack on an implementation of DES'
subtitle: A primer to hardware security
gh-repo: learn-hardware-security/py-des-scan
gh-badge: [star, fork, follow]
tags: [reverse-engineering, hardware, python]
---

Scan chains, a Design  for Test (DFT) technique, are implemented in integrated circuits (ICs) in order to test their correct functionality. They provide high fault coverage and do not need complex hardware for test pattern generation or signature analysis. They function by converting all internal flip-flops into scannable flip-flops, pairing each element with an additional multiplexer such that they may be disconnected from the main circuit and daisy-chained to instead feed their data amongst themselves and out of the circuit. This architecture of a scan chain is depicted here.

![scan chain arch.]({{ 'assets/img/hw-scan-attack/scan_chain.png' | relative_url }}){: .mx-auto.d-block :}

# The issue

Scan chains are intented to allow an IC designer or manufacturer to test a given design post-fabrication by checking the flow of bits as they move through the circuit. 
However, as they cannot be removed from the design once constructed, they also represent for malicious third parties a significant source of information about the internal functionality and implementation of a given device. 
Although designers can seek to prevent access to the scan chain (e.g. by using access control, either digital or physical) these can be overcome given time and access to the product.

In this tutorial we will explore this by providing an implementation of the aged [DES](https://en.wikipedia.org/wiki/Data_Encryption_Standard) algorithm.
Although no longer widely used (indeed, it is recommended _not_ to be used), DES was used for decades to protect digital secrets.
Further, the steps that we outline in this tutorial may be utilized in the general sense to attack other algorithms, notably [AES](https://ieeexplore.ieee.org/abstract/document/6847798).

# The tutorial

This tutorial is presented in PDF format [on Arxiv](https://arxiv.org/pdf/2207.10466.pdf). Thanks also to my co-authors Benjamin Tan (University of Calgary) and Ramesh Karri (NYU).
