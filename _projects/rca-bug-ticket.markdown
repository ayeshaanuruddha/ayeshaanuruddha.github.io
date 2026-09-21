---
layout: case-study
title: Mobile Sync Latency — Root Cause Analysis
date: 2025-02-15
code: WMS-BUG-1042
status: READY FOR REVIEW
status_tone: green
client: Enterprise Logistics Firm
project-date: February 2025
category: Business Analysis & RCA
card_summary: >-
  RCA for a mobile sync latency bug, tracing UI freezes during
  high-volume batch scanning to database row-locking via the 5 Whys
  method.
card_quote: >-
  Why 5 (root cause): the DB lacks optimized indexing for parallel batch
  processing, forcing synchronous waits.
full_spec: /warehouse.html
full_spec_label: View full spec
ticket:
  type: Bug
  priority: High
  labels: [Mobile OS, LiveInventory, RCA, Performance]
  owner: Business Analyst
  signoff: PO, SME/Dev, QE
---

<h2>Executive Summary</h2>
<p>This Root Cause Analysis (RCA) document details the investigation and resolution of a critical UI latency issue during high-volume warehouse batch processing. By applying the "5 Whys" methodology, a database locking bottleneck was identified and resolved through an asynchronous synchronization architecture. The resulting action plan ensures system stability and resolves bottlenecks that directly impact warehouse throughput and worker productivity.</p>

<hr>

<h2>(1) User Story Details</h2>

<h3>(1.1) Summary</h3>
<div class="table-wrap"><table>
  <tbody>
    <tr><td><strong>As a</strong></td><td>warehouse selector</td></tr>
    <tr><td><strong>I</strong></td><td>want my mobile application to immediately sync allocated inventory quantities upon scanning</td></tr>
    <tr><td><strong>So that I</strong></td><td>am not blocked by loading screens or latency during high-volume batch processing</td></tr>
  </tbody>
</table></div>

<h2>(2) User Experience &amp; Preconditions</h2>
<div class="table-wrap"><table>
  <tbody>
    <tr><td><strong>Preconditions</strong></td><td>The mobile client device is operating on a stable warehouse Wi-Fi network and processing a multi-item batch selection.</td></tr>
    <tr><td><strong>Current Process (Bug Behavior)</strong></td><td>When a user scans an item during peak hours, the application experiences a 3 to 5-second delay before the UI updates and the next scan is permitted.</td></tr>
    <tr><td><strong>New Process (Expected Behavior)</strong></td><td>The mobile application should process the payload and update the local UI within 200ms, syncing with the backend asynchronously.</td></tr>
  </tbody>
</table></div>

<h2>(3) Root Cause Analysis (5 Whys Method)</h2>
<p><strong>Problem Statement:</strong> Mobile application experiences UI freezing and high latency during inventory allocation scans.</p>
<div class="table-wrap"><table>
  <tbody>
    <tr><td><strong>Why 1</strong></td><td>The mobile application is waiting for a synchronous response from the central database before allowing the next action.</td></tr>
    <tr><td><strong>Why 2</strong></td><td>The backend API endpoint is taking over 3 seconds to process the JSON payload.</td></tr>
    <tr><td><strong>Why 3</strong></td><td>The database forensic logs show a bottleneck when querying the current Quantity on Hand (QOH) during the transaction.</td></tr>
    <tr><td><strong>Why 4</strong></td><td>The allocation table is experiencing row-level locking because hundreds of concurrent users are attempting to read/write to the same high-volume SKU index.</td></tr>
    <tr><td><strong>Why 5 (Root Cause)</strong></td><td>The database lacks an optimized indexing strategy for parallel batch processing, and the mobile app architecture forces a synchronous wait state instead of an asynchronous background sync.</td></tr>
  </tbody>
</table></div>

<h2>(4) Acceptance Criteria</h2>
<div class="table-wrap"><table>
  <tbody>
    <tr><td><strong>GIVEN</strong></td><td>the user is processing a multi-item batch</td></tr>
    <tr><td><strong>WHEN</strong></td><td>an item barcode is successfully scanned</td></tr>
    <tr><td><strong>THEN</strong></td><td>the UI must update instantly to allow the next scan without locking</td></tr>
  </tbody>
</table></div>
<div class="table-wrap"><table>
  <tbody>
    <tr><td><strong>GIVEN</strong></td><td>a batch scan event has occurred</td></tr>
    <tr><td><strong>WHEN</strong></td><td>the API payload is transmitted</td></tr>
    <tr><td><strong>THEN</strong></td><td>the synchronization must happen asynchronously in the background</td></tr>
  </tbody>
</table></div>
<div class="table-wrap"><table>
  <tbody>
    <tr><td><strong>GIVEN</strong></td><td>a network interruption occurs during asynchronous sync</td></tr>
    <tr><td><strong>WHEN</strong></td><td>connectivity is restored</td></tr>
    <tr><td><strong>THEN</strong></td><td>the application must automatically retry the queued payloads without data loss</td></tr>
  </tbody>
</table></div>

<h2>(5) References</h2>

<h3>(5.1) UAT Requirement</h3>
<div class="table-wrap"><table>
  <tbody>
    <tr><td><strong>Tested Version</strong></td><td><code>[Sanitized Build Version ANDROID]</code></td></tr>
    <tr><td><strong>WMS Host</strong></td><td><code>Production</code></td></tr>
    <tr><td><strong>Area</strong></td><td>Core Inventory Module</td></tr>
    <tr><td><strong>WMS Process</strong></td><td>Batch Allocation &amp; Sync</td></tr>
    <tr><td><strong>User Experience</strong></td><td>UI freezing during multi-item scan</td></tr>
    <tr><td><strong>Issue received via/from</strong></td><td>Warehouse Operations Team</td></tr>
    <tr><td><strong>UAT #ID</strong></td><td><code>WMS-BUG-1042</code></td></tr>
  </tbody>
</table></div>

<h3>(5.2) Developer Notes</h3>
<div class="table-wrap"><table>
  <tbody>
    <tr><td><strong>Tech Design</strong></td><td>API Payload Analysis &amp; Asynchronous Sync Architecture</td></tr>
    <tr><td><strong>Database Forensics</strong></td><td>Transaction Log Analysis (Shold be Attached)</td></tr>
  </tbody>
</table></div>