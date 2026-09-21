---
layout: case-study
title: Edge-Based E-Voting System with MPC
date: 2025-08-20
code: FYP · 2025
status: FEATURED
status_tone: purple
client: University of Kelaniya
project-date: August 2025
category: Cybersecurity & Web Development
description: This project introduces a privacy-preserving, edge-based e-voting system integrating Multi-Party Computation (MPC) specifically architected for Sri Lanka. It leverages edge computing to decentralize ballot processing at polling sites, reducing network dependence, while MPC protocols ensure ballot secrecy and verifiable aggregation. The developed local prototype successfully demonstrates fingerprint-based voter authentication, secure ballot encryption, and distributed MPC tallying to make it technologically impossible for any single authority to tamper with individual votes.
card_summary: >-
  An edge-computing e-voting framework combining ESP32 biometric
  authentication with Multi-Party Computation to guarantee CIAAN across
  every vote.
card_quote: >-
  Objective: to establish a verifiable, trust-building e-voting system
  tailored for Sri Lanka's infrastructural constraints.
full_spec: https://github.com/ayeshaanuruddha/e-vote-main
full_spec_label: View source
---

### Executive Summary
{: .text-justify }

This project tackles the challenges of manual, paper-based electoral systems in Sri Lanka by proposing a modern, cost-effective, privacy-preserving electronic voting framework. It leverages edge computing and Multi-Party Computation (MPC) to guarantee the Confidentiality, Integrity, Availability, Authentication, and Non-repudiation (CIAAN) of every vote. By demonstrating successful biometric authentication and secure distributed tallying, this prototype establishes a verifiable and trust-building architecture for national elections.

### Edge-Based E-Voting System with MPC

#### 📋 Project Overview

| | |
|---|---|
| 🎯 **Objective** | To establish a verifiable, trust-building e-voting system tailored for Sri Lanka's infrastructural constraints. |
| 🛠️ **Technology Stack** | Remix (Frontend), FastAPI (Backend), MySQL (Database), ESP32 Microcontrollers (Edge). |
| 🔒 **Security Model** | CIAAN Architecture, Biometric Authentication, Secret-Sharing Algorithms. |
| 🎓 **Context** | BSc Honours in Computer Science Final Year Project, Faculty of Computing and Technology, University of Kelaniya. |
{: .table .table-bordered }

---

### (1) Core System Architecture

| | |
|---|---|
| **Tier 1: Edge Nodes** | ESP32 devices handle local biometric voter authentication and ballot encryption at polling stations to reduce network dependence. |
| **Tier 2: MPC Overlay** | A secure peer-to-peer network distributes secret shares of the votes to enable collective tallying without revealing individual selections. |
| **Tier 3: Central Services** | A high-performance FastAPI backend manages the voter registry, API routing, and final encrypted result storage via MySQL. |
{: .table .table-bordered }

### (2) Key Implementation Features

| | |
|---|---|
| **Biometric Authentication** | Voters are authenticated using fingerprint scanners integrated with ESP32 edge nodes, ensuring the one-person-one-vote principle. |
| **Secure Vote Casting** | Votes are cryptographically encapsulated using advanced protocols before storage to ensure absolute integrity and secrecy. |
| **Distributed Tallying** | MPC protocols guarantee that it is technologically impossible for any single authority to tamper with or reconstruct individual votes. |
{: .table .table-bordered .table-striped }

### (3) System Evaluation & Future Work
{: .text-justify }

A developed local prototype successfully demonstrated the complete pipeline: fingerprint-based voter authentication, secure ballot encryption, and distributed MPC tallying. While testing successfully proved the feasibility of preventing duplicate voting and protecting against tampering, it was conducted in a controlled environment limited to a single ESP32 device, meaning high-load scalability remains to be fully evaluated.

**Future Enhancements:** The roadmap for a nationwide rollout includes extending the prototype to higher-performance edge devices (like the Raspberry Pi), integrating post-quantum cryptography (PQC) for long-term security, and employing multi-biometric sensors for enhanced authentication.

<div class="text-center" style="margin: 2rem 0;">
  <a href="https://github.com/ayeshaanuruddha/e-vote-main" target="_blank" rel="noopener noreferrer" class="btn btn-lg" style="background-color: #24292e; color: #fff; width: 100%; max-width: 420px;">
    <i class="fa-brands fa-github"></i> View Source Code on GitHub
  </a>
</div>