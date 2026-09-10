---
#layout: default
modal-id: 2
date: 2025-02-15
img: rca-analysis.png
alt: Root Cause Analysis Bug Ticket
project-date: February 2025
client: Enterprise Logistics Firm
category: Business Analysis & RCA
---

<p>This sanitized bug ticket and Root Cause Analysis document uses standard business analysis structures, including Gherkin syntax and the 5 Whys method, allowing you to showcase your problem-solving process without exposing proprietary data.</p>

<h3>[BUG-1042] [Performance] Mobile App Synchronization Latency During High-Volume Batch Processing</h3>

<h4>📋 Ticket Completion Status</h4>
<table class="table table-bordered" style="text-align: left;">
  <tbody>
    <tr><td>🐛 <strong>Type</strong></td><td>Bug</td></tr>
    <tr><td>⏫ <strong>Priority</strong></td><td>High</td></tr>
    <tr><td>🏷️ <strong>Labels</strong></td><td><code>Android</code> <code>LiveInventory</code> <code>RCA</code> <code>Performance</code></td></tr>
    <tr><td>⭐ <strong>Ticket status</strong></td><td><code>READY FOR REVIEW</code></td></tr>
    <tr><td>👥 <strong>Ticket owner</strong></td><td>Business Analyst</td></tr>
    <tr><td>✅ <strong>Reviewed and Signed off</strong></td><td>PO, SME/Dev, QE</td></tr>
  </tbody>
</table>

<hr>

<h3>(1) User Story Details</h3>

<h4>(1.1) Summary</h4>
<table class="table table-bordered" style="text-align: left;">
  <tbody>
    <tr><td><strong>As a</strong></td><td>warehouse selector</td></tr>
    <tr><td><strong>I</strong></td><td>want my mobile application to immediately sync allocated inventory quantities upon scanning</td></tr>
    <tr><td><strong>So that I</strong></td><td>am not blocked by loading screens or latency during high-volume batch processing</td></tr>
  </tbody>
</table>

<h3>(2) User Experience &amp; Preconditions</h3>
<table class="table table-bordered" style="text-align: left;">
  <tbody>
    <tr><td><strong>Preconditions</strong></td><td>The mobile client device is operating on a stable warehouse Wi-Fi network and processing a multi-item batch selection.</td></tr>
    <tr><td><strong>Current Process (Bug Behavior)</strong></td><td>When a user scans an item during peak hours, the application experiences a 3 to 5-second delay before the UI updates and the next scan is permitted.</td></tr>
    <tr><td><strong>New Process (Expected Behavior)</strong></td><td>The mobile application should process the payload and update the local UI within 200ms, syncing with the backend asynchronously.</td></tr>
  </tbody>
</table>

<h3>(3) Root Cause Analysis (5 Whys Method)</h3>
<p><strong>Problem Statement:</strong> Mobile application experiences UI freezing and high latency during inventory allocation scans.</p>
<table class="table table-bordered" style="text-align: left;">
  <tbody>
    <tr><td><strong>Why 1</strong></td><td>The mobile application is waiting for a synchronous response from the central database before allowing the next action.</td></tr>
    <tr><td><strong>Why 2</strong></td><td>The backend API endpoint is taking over 3 seconds to process the JSON payload.</td></tr>
    <tr><td><strong>Why 3</strong></td><td>The database forensic logs show a bottleneck when querying the current Quantity on Hand (QOH) during the transaction.</td></tr>
    <tr><td><strong>Why 4</strong></td><td>The allocation table is experiencing row-level locking because hundreds of concurrent users are attempting to read/write to the same high-volume SKU index.</td></tr>
    <tr><td><strong>Why 5 (Root Cause)</strong></td><td>The database lacks an optimized indexing strategy for parallel batch processing, and the mobile app architecture forces a synchronous wait state instead of an asynchronous background sync.</td></tr>
  </tbody>
</table>

<h3>(4) Acceptance Criteria</h3>
<table class="table table-bordered table-striped" style="text-align: left;">
  <tbody>
    <tr><td><strong>GIVEN</strong></td><td>the user is processing a multi-item batch</td></tr>
    <tr><td><strong>WHEN</strong></td><td>an item barcode is successfully scanned</td></tr>
    <tr><td><strong>THEN</strong></td><td>the UI must update instantly to allow the next scan without locking</td></tr>
  </tbody>
</table>
<table class="table table-bordered table-striped" style="text-align: left;">
  <tbody>
    <tr><td><strong>GIVEN</strong></td><td>a batch scan event has occurred</td></tr>
    <tr><td><strong>WHEN</strong></td><td>the API payload is transmitted</td></tr>
    <tr><td><strong>THEN</strong></td><td>the synchronization must happen asynchronously in the background</td></tr>
  </tbody>
</table>
<table class="table table-bordered table-striped" style="text-align: left;">
  <tbody>
    <tr><td><strong>GIVEN</strong></td><td>a network interruption occurs during asynchronous sync</td></tr>
    <tr><td><strong>WHEN</strong></td><td>connectivity is restored</td></tr>
    <tr><td><strong>THEN</strong></td><td>the application must automatically retry the queued payloads without data loss</td></tr>
  </tbody>
</table>

<h3>(5) References</h3>

<h4>(5.1) UAT Requirement</h4>
<table class="table table-bordered" style="text-align: left;">
  <tbody>
    <tr><td><strong>Tested Version</strong></td><td><code>[Sanitized Build Version ANDROID]</code></td></tr>
    <tr><td><strong>WMS Host</strong></td><td><code>Production</code></td></tr>
    <tr><td><strong>Area</strong></td><td>Core Inventory Module</td></tr>
    <tr><td><strong>WMS Process</strong></td><td>Batch Allocation &amp; Sync</td></tr>
    <tr><td><strong>User Experience</strong></td><td>UI freezing during multi-item scan</td></tr>
    <tr><td><strong>Issue received via/from</strong></td><td>Warehouse Operations Team</td></tr>
    <tr><td><strong>UAT #ID</strong></td><td><code>BUG-1042-ANON</code></td></tr>
  </tbody>
</table>

<h4>(5.2) Developer Notes</h4>
<table class="table table-bordered" style="text-align: left;">
  <tbody>
    <tr><td><strong>Tech Design</strong></td><td>API Payload Analysis &amp; Asynchronous Sync Architecture</td></tr>
    <tr><td><strong>Database Forensics</strong></td><td>Transaction Log Analysis (Attached)</td></tr>
  </tbody>
</table>