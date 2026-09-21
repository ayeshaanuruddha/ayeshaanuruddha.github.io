---
layout: case-study
title: POS Webhook Ingestion & Balanced Journal Entries
date: 2025-03-01
code: FIN-INT-101
status: UNDER CONSTRUCTION
status_tone: gold
client: Omnichannel Retail Enterprise
project-date: March 2025
category: Financial Systems & ERP Integration
card_summary: >-
  Integration contract and double-entry accounting engine between an edge POS
  gateway and a cloud ERP general ledger, built under strict SOX 404 compliance.
card_quote: >-
  GIVEN a $5.40 sale WHEN the payment.captured webhook fires THEN ΣDebits −
  ΣCredits = $0.00 before ERP commit.
full_spec: /businessone.html
full_spec_label: View full spec
ticket:
  type: Story
  priority: High
  labels: [Functional Requirement, Finance-Integration, POS-Engine, General-Ledger, ERP]
  owner: Business Analyst
  signoff: Product Owner (Finance), Lead Integration Architect, Lead QE, Lead Dev, Principal Controller, SME/Dev
---

<h3>Executive Summary</h3>
<p>This project automates the reconciliation of Point of Sale (POS) transactions into a cloud General Ledger ERP, saving the finance team over 18 hours per month. By unbundling fees and taxes in real-time while strictly adhering to SOX 404 compliance, it completely eliminates manual data entry errors. The following specification bridges complex accounting requirements with robust technical API integrations to ensure secure and balanced financial reporting.</p>

<h3>[FIN-INT-101] Functional Specification: POS Webhook Ingestion &amp; Balanced Journal Entry Creation</h3>

<h4>📋 Ticket Completion Status</h4>
<div class="table-wrap"><table>
  <tbody>
    <tr><td>🐛 <strong>Type</strong></td><td>Story</td></tr>
    <tr><td>⏫ <strong>Priority</strong></td><td>High</td></tr>
    <tr><td>🏷️ <strong>Labels</strong></td><td><code>Functional Requirement</code> <code>Finance-Integration</code> <code>POS-Engine</code> <code>General-Ledger</code> <code>ERP</code></td></tr>
    <tr><td>⭐ <strong>Ticket Status</strong></td><td><code>UNDER CONSTRUCTION</code></td></tr>
    <tr><td>👥 <strong>Ticket Owner</strong></td><td>Business Analyst</td></tr>
    <tr><td>✅ <strong>Reviewed and Signed off</strong></td><td>Product Owner (Finance), Lead Integration Architect, Lead QE, Lead Dev, Principal Controller, SME/Dev</td></tr>
  </tbody>
</table></div>

<hr>

<h3>(1) Business Context &amp; User Story</h3>

<h4>(1.1) Summary</h4>
<div class="table-wrap"><table>
  <tbody>
    <tr><td><strong>As a</strong></td><td>Corporate Revenue Accountant</td></tr>
    <tr><td><strong>I want</strong></td><td>inbound POS card transaction webhooks to automatically unbundle gross sales into merchandise revenue, sales tax liabilities, and merchant processing fees, posting balanced double-entry journals into the cloud ERP</td></tr>
    <tr>
      <td><strong>So that I</strong></td>
      <td>
        • Recognize real-time revenue and operational fee expenses immediately at the point of sale<br>
        • Eliminate manual spreadsheet journal uploads during month-end close<br>
        • Mathematically prevent unbalanced financial postings from reaching the general ledger
      </td>
    </tr>
  </tbody>
</table></div>

<h4>(1.2) Operational Elicitation and System Requirements</h4>
<div class="table-wrap"><table>
  <thead>
    <tr>
      <th>Dimension</th>
      <th>(i) Operational Elicitation (As-Is Finding)</th>
      <th>(ii) System Requirements (To-Be Contract)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>Discovery Source</strong></td>
      <td><strong>Month-End Close Observation &amp; Incident Logs:</strong> Finance manually downloads weekly CSV dumps from payment processors and POS back-offices, consuming 18+ hours per close cycle.</td>
      <td><strong>Event-Driven Architecture:</strong> Ingestion microservice exposes a secure public endpoint accepting real-time HTTPS JSON webhooks as card transactions settle (T+0).</td>
    </tr>
    <tr>
      <td><strong>Fee Recognition</strong></td>
      <td><strong>Lump-Sum Opacity:</strong> Merchant fees (2.6% + $0.10) are deducted as a net lump-sum payout days later, forcing accounting to estimate processing expenses via weekly approximations.</td>
      <td><strong>Transaction-Level Fee Unbundling:</strong> Middleware parses <code>fee_breakdown.amount</code> directly from the payload and immediately debits <code>GL 6200 - Payment Processing Fees</code>.</td>
    </tr>
    <tr>
      <td><strong>Tax Segregation</strong></td>
      <td><strong>Tax Liability Variance:</strong> Store managers report gross terminal totals; staff accountants manually calculate tax splits in Excel, causing periodic cent-level transposition errors.</td>
      <td><strong>Statutory Liability Segregation:</strong> Middleware extracts <code>tax_breakdown.total</code> and credits <code>GL 2200 - Sales Tax Payable</code> as a dedicated current liability balance.</td>
    </tr>
    <tr>
      <td><strong>Ledger Integrity</strong></td>
      <td><strong>Unbalanced Manual Commits:</strong> Manual CSV journal entry uploads fail 3.8% of the time due to human entry rounding differences where Debits &ne; Credits.</td>
      <td><strong>Mathematical Equilibrium Assertion:</strong> Integration engine enforces a pre-commit assertion: <code>&Sigma; Debits - &Sigma; Credits = $0.00</code> prior to invoking ERP journal APIs.</td>
    </tr>
    <tr>
      <td><strong>Cost Center Tagging</strong></td>
      <td><strong>Centralized Dumping:</strong> Sales book to a generic corporate clearing account because register operators do not format store numbers consistently.</td>
      <td><strong>Automated Segment Tagging:</strong> Middleware maps <code>payload.store_id</code> against the ERP Cost Center master directory, tagging every ledger line to the exact store branch.</td>
    </tr>
  </tbody>
</table></div>

<hr>

<h3>(2) Acceptance Criteria (Behavioral &amp; Data Engine)</h3>

<h4>(2.1) Environmental &amp; Operational Preconditions</h4>
<ul>
  <li><strong>Security &amp; Auth:</strong> Webhooks dispatched over TLS 1.3 with a valid HMAC-SHA256 signature matching the shared secret in the key vault.</li>
  <li><strong>Entity Availability:</strong> Target General Ledger accounts (<code>1050</code>, <code>6200</code>, <code>4100</code>, <code>2200</code>) and store <code>Cost_Center</code> entities must reside in an <code>ACTIVE</code> status within the ERP Chart of Accounts.</li>
  <li><strong>Fiscal Calendar:</strong> Transaction timestamp must fall within an open accounting period in the corporate ERP general ledger.</li>
</ul>

<h4>(2.2) Common Non-Functional Baselines</h4>
<ul>
  <li><strong>Payload Ingestion SLA:</strong> Webhook acknowledgment (<code>HTTP 200 OK</code>) must return within &le; 300ms of socket receipt.</li>
  <li><strong>Transactional Atomicity:</strong> All ledger legs must commit within an atomic ACID boundary; partial or single-leg journal entries must never post.</li>
  <li><strong>Audit Lineage:</strong> Every transaction must preserve raw ingress JSON payloads and the returned ERP Journal Document ID within an immutable audit table.</li>
</ul>

<h4>(2.3) Functional Scenarios (Gherkin Syntax)</h4>

<div class="table-wrap"><table>
  <thead>
    <tr><th colspan="2">Scenario 01: Nominal Path &mdash; Valid Card Sale &amp; Balanced Multi-Leg Journal Commit</th></tr>
  </thead>
  <tbody>
    <tr><td><strong>GIVEN</strong></td><td>a retail customer purchases an item ($5.00 base + $0.40 sales tax = $5.40 total charge) and POS captures payment</td></tr>
    <tr><td><strong>WHEN</strong></td><td>the payment gateway dispatches the <code>payment.captured</code> webhook containing gross $5.40, tax $0.40, fee $0.24, and net settlement $5.16</td></tr>
    <tr><td><strong>THEN</strong></td><td>the middleware balance assertion verifies: <code>&Sigma; Debits ($5.16 + $0.24) - &Sigma; Credits ($5.00 + $0.40) = $0.00</code></td></tr>
    <tr><td><strong>AND</strong></td><td>submits an atomic journal entry to ERP: Debit GL 1050 ($5.16), Debit GL 6200 ($0.24), Credit GL 4100 ($5.00), Credit GL 2200 ($0.40)</td></tr>
    <tr><td><strong>AND</strong></td><td>returns <code>HTTP 200 OK</code> to the gateway, updating internal state to <code>ERP_POSTED</code></td></tr>
  </tbody>
</table></div>

<div class="table-wrap"><table>
  <thead>
    <tr><th colspan="2">Scenario 02: Validation Exception &mdash; Double-Entry Imbalance Interception</th></tr>
  </thead>
  <tbody>
    <tr><td><strong>GIVEN</strong></td><td>an incoming transaction payload where line items sum incorrectly (e.g., Gross = $5.40, but Line Items + Tax = $5.39, |&Delta;| = $0.01)</td></tr>
    <tr><td><strong>WHEN</strong></td><td>the pre-commit balance assertion executes prior to ERP dispatch</td></tr>
    <tr><td><strong>THEN</strong></td><td>the engine must block ERP API execution, set status to <code>PAYLOAD_ERROR</code>, and route the message to the Dead-Letter Queue (DLQ)</td></tr>
    <tr><td><strong>AND</strong></td><td>dispatch an alert notification with error code <code>ERR-FIN-101</code> without blocking remaining queue processing</td></tr>
  </tbody>
</table></div>

<div class="table-wrap"><table>
  <thead>
    <tr><th colspan="2">Scenario 03: Infrastructure Exception &mdash; Transient ERP Downtime &amp; Idempotent Retry</th></tr>
  </thead>
  <tbody>
    <tr><td><strong>GIVEN</strong></td><td>a mathematically balanced transaction payload ready for ledger commit</td></tr>
    <tr><td><strong>WHEN</strong></td><td>the ERP API endpoint returns a transient error (<code>HTTP 429</code> or <code>HTTP 503</code>)</td></tr>
    <tr><td><strong>THEN</strong></td><td>the engine records the payload into a persistent retry table with state <code>NETWORK_TIMEOUT</code></td></tr>
    <tr><td><strong>AND</strong></td><td>executes exponential backoff retries (10s, 30s, 2m, 10m) supplying the original <code>transaction_id</code> as the <code>Idempotency-Key</code> header to prevent duplicate ledger postings</td></tr>
  </tbody>
</table></div>

<h4>(2.4) Field-Level Input Specifications &amp; Data Dictionary</h4>
<div class="table-wrap"><table>
  <thead>
    <tr>
      <th>Field Label</th>
      <th>Ingress Method</th>
      <th>Data Format &amp; Mask</th>
      <th>Length</th>
      <th>Req?</th>
      <th>Validation Rules &amp; Error Handling</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>transaction_id</code></td>
      <td>Webhook JSON</td>
      <td>Alphanumeric<br><code>^[a-zA-Z0-9_-]+$</code></td>
      <td>64 Chars</td>
      <td>Yes</td>
      <td>Evaluated against Redis cache. If ID exists within 24h window, suppress duplicate ERP call.</td>
    </tr>
    <tr>
      <td><code>store_id</code></td>
      <td>Webhook JSON</td>
      <td>Alphanumeric<br><code>^STORE-[A-Z0-9]+$</code></td>
      <td>32 Chars</td>
      <td>Yes</td>
      <td>Cross-referenced against ERP cost center master. If unmapped, halt with <code>ERR-FIN-102</code>.</td>
    </tr>
    <tr>
      <td><code>timestamp_utc</code></td>
      <td>Webhook JSON</td>
      <td>ISO-8601 UTC<br><code>YYYY-MM-DDTHH:mm:ss.sssZ</code></td>
      <td>24 Chars</td>
      <td>Yes</td>
      <td>Evaluated against ERP fiscal calendar. Must fall within an open accounting period.</td>
    </tr>
    <tr>
      <td><code>subtotal</code></td>
      <td>Webhook JSON</td>
      <td>Decimal (0.00)</td>
      <td>12 Digits</td>
      <td>Yes</td>
      <td>Merchandise net sales before tax. Must be &gt; 0.00. Mapped to Credit <code>GL 4100</code>.</td>
    </tr>
    <tr>
      <td><code>tax_total</code></td>
      <td>Webhook JSON</td>
      <td>Decimal (0.00)</td>
      <td>10 Digits</td>
      <td>Yes</td>
      <td>Jurisdiction sales tax. Must match configured rate (&plusmn; $0.01). Mapped to Credit <code>GL 2200</code>.</td>
    </tr>
    <tr>
      <td><code>fee_amount</code></td>
      <td>Webhook JSON</td>
      <td>Decimal (0.00)</td>
      <td>10 Digits</td>
      <td>Yes</td>
      <td>Processing fee assessed by gateway. Must be &ge; 0.00. Mapped to Debit <code>GL 6200</code>.</td>
    </tr>
    <tr>
      <td><code>net_settlement</code></td>
      <td>Webhook JSON</td>
      <td>Decimal (0.00)</td>
      <td>12 Digits</td>
      <td>Yes</td>
      <td>Expected cash deposit: <code>(subtotal + tax_total) - fee_amount</code>. Mapped to Debit <code>GL 1050</code>.</td>
    </tr>
  </tbody>
</table></div>

<h4>(2.5) Alerts, Feedback &amp; Error Messages</h4>
<div class="table-wrap"><table>
  <thead>
    <tr>
      <th>Message Code</th>
      <th>Trigger Condition</th>
      <th>Severity / Channel</th>
      <th>System Event Log Content</th>
      <th>Remediation Action</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><code>ERR-FIN-101</code></td>
      <td>&Sigma; Debits &ne; &Sigma; Credits</td>
      <td>Critical / PagerDuty</td>
      <td><code>UNBALANCED_PAYLOAD_REJECTED: Debit ($5.40) != Credit ($5.39). DLQ Enqueued.</code></td>
      <td>Route payload to DLQ; alert on-call Systems BA to investigate line-item rounding variance.</td>
    </tr>
    <tr>
      <td><code>ERR-FIN-102</code></td>
      <td>Store ID unmapped in ERP</td>
      <td>High / Slack-Finance</td>
      <td><code>INVALID_COST_CENTER: Store ID [STORE-99] not active in Chart of Accounts.</code></td>
      <td>Quarantine message; prompt Finance Admin to map store node in ERP Cost Center master.</td>
    </tr>
    <tr>
      <td><code>WARN-FIN-105</code></td>
      <td>Ingestion retry count &gt; 3</td>
      <td>Medium / CloudWatch</td>
      <td><code>RETRY_THRESHOLD_WARNING: TXN [txn_choc_98124] backoff interval 2m reached.</code></td>
      <td>System retries automatically; escalates to ERP integration team if attempts exceed 5.</td>
    </tr>
    <tr>
      <td><code>SUCC-FIN-001</code></td>
      <td>Balanced journal committed</td>
      <td>Info / Audit Log</td>
      <td><code>ERP_POSTED_SUCCESS: Journal Entry [JE-882190] created for TXN [txn_choc_98124].</code></td>
      <td>Advance record state to <code>ERP_POSTED</code>; register in daily settlement pool.</td>
    </tr>
  </tbody>
</table></div>

<hr>

<h3>(3) Security &amp; Access Control (RBAC Matrix)</h3>
<div class="table-wrap"><table>
  <thead>
    <tr>
      <th>Enterprise Role Profile</th>
      <th>Execute / Ingest</th>
      <th>Query / Lookup</th>
      <th>Manual Override / DLQ Replay</th>
      <th>Functional Scope &amp; Governance Boundary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td><strong>POS Gateway Service Account</strong></td>
      <td>Y</td>
      <td>N</td>
      <td>N</td>
      <td>Automated programmatic account authorized strictly to post webhooks via mTLS.</td>
    </tr>
    <tr>
      <td><strong>Staff Revenue Accountant</strong></td>
      <td>N</td>
      <td>Y</td>
      <td>N</td>
      <td>Read-only inspection of processed transaction logs, journal IDs, and daily reconciliation views.</td>
    </tr>
    <tr>
      <td><strong>Senior Finance Operations Lead</strong></td>
      <td>N</td>
      <td>Y</td>
      <td>Y</td>
      <td>Authorized to inspect DLQ payloads, update cost-center lookup mappings, and trigger payload re-syncs.</td>
    </tr>
    <tr>
      <td><strong>Internal / SOX Auditor</strong></td>
      <td>N</td>
      <td>Y</td>
      <td>N</td>
      <td>Immutable read-only view across transaction audit tables, payload hashes, and timestamp logs.</td>
    </tr>
  </tbody>
</table></div>

<hr>

<h3>(4) Scope Boundaries &amp; Operational Exclusions</h3>
<ul>
  <li><strong>PCI-DSS Card Tokenization:</strong> Point-to-Point Encryption (P2PE), PAN handling, and card brand authorization handshakes are fully executed by the physical pin-pad and gateway; this service ingests post-authorization webhooks only.</li>
  <li><strong>Bank Payout Statement Clearing:</strong> Matching posted clearing balances in GL <code>1050</code> against electronic bank statements (BAI2 / CAMT.053) is governed separately under the 3-Way Bank Reconciliation specification.</li>
  <li><strong>Manual Journal Corrections:</strong> Direct ledger adjustments inside the ERP interface remain subject to native accounting controls and are outside the scope of this middleware service.</li>
</ul>

<hr>

<h3>(5) Integration Contracts &amp; Technical Architecture</h3>

<h4>(5.1) Inbound POS Webhook Payload Schema</h4>
<p>Inbound payment events are accepted via HTTPS POST with mutual TLS (mTLS) and payload HMAC verification:</p>

<!-- Dark Mode Code Container -->
<div class="codeblock">
<pre>{
  "transaction_id": "txn_choc_98124",
  "client_timestamp_utc": "2026-09-15T14:40:10.102Z",
  "store_id": "STORE-COLOMBO-01",
  "currency": "USD",
  "line_items": [
    {
      "item_sku": "CHOC-DARK-85",
      "quantity": 1,
      "unit_price": 5.00,
      "subtotal": 5.00
    }
  ],
  "tax_breakdown": {
    "jurisdiction": "LK-WESTERN",
    "effective_rate_pct": 8.00,
    "total": 0.40
  },
  "payment_capture": {
    "payment_method": "EMV_CONTACTLESS",
    "card_brand": "VISA",
    "gross_amount": 5.40,
    "fee_breakdown": {
      "interchange_pct": 2.60,
      "fixed_cut": 0.10,
      "amount": 0.24
    },
    "net_settlement": 5.16
  }
}</pre>
</div>

<h4>(5.2) Persistence &amp; Database State Transitions</h4>

<!-- State Diagram Graphic -->
<div>
  <img src="img/portfolio/pos_state_flowchart.png" 
       alt="POS Ingestion and Journal Lifecycle State Machine" 
       class="img-responsive img-centered">
</div>

<ul>
  <li><strong>Target Database Entities:</strong> <code>INTEGRATION_TXN_LOG</code>, <code>ERP_JOURNAL_STAGING</code>, <code>DLQ_EXCEPTION_LOG</code>, <code>IDEMPOTENCY_CACHE</code>.</li>
  <li><strong>State Transition Engine:</strong> <code>QUEUED</code> &rarr; <code>VALIDATED</code> &rarr; <code>ERP_POSTED</code> processed inside an atomic transactional boundary.</li>
  <li><strong>Concurrency &amp; Idempotency:</strong> External UUID cached in Redis with a 24-hour TTL; duplicate calls with identical transaction IDs are suppressed prior to ERP API dispatch.</li>
</ul>

<ul>
  <li><strong>Target Database Entities:</strong> <code>INTEGRATION_TXN_LOG</code>, <code>ERP_JOURNAL_STAGING</code>, <code>DLQ_EXCEPTION_LOG</code>, <code>IDEMPOTENCY_CACHE</code>.</li>
  <li><strong>State Transition Engine:</strong> <code>QUEUED</code> &rarr; <code>VALIDATED</code> &rarr; <code>ERP_POSTED</code> processed inside an atomic transactional boundary.</li>
  <li><strong>Concurrency &amp; Idempotency:</strong> External UUID cached in Redis with a 24-hour TTL; duplicate calls with identical transaction IDs are suppressed prior to ERP API dispatch.</li>
</ul>

<hr>

<h3>(6) Traceability &amp; Definition of Done (DoD)</h3>

<h4>(6.1) Upstream Traceability</h4>
<div class="table-wrap"><table>
  <tbody>
    <tr><td><strong>Jira Epic Link</strong></td><td><code>[FIN-EPIC-4400] Omnichannel Retail POS to Cloud ERP General Ledger Modernization</code></td></tr>
    <tr><td><strong>Regulatory Anchor</strong></td><td>Sarbanes-Oxley (SOX) Section 404 (Internal Controls) &amp; ASC 606 (Revenue Recognition Standards)</td></tr>
    <tr><td><strong>Target Observability SLI</strong></td><td>Webhook ingestion to ERP journal creation latency: &le; 1200ms at p95</td></tr>
  </tbody>
</table></div>

<h4>(6.2) Definition of Done Checklist</h4>
<div class="table-wrap"><table>
  <tbody>
    <tr>
      <td>
        <ul>
          <li>⬜ Specification reviewed, groomed, and signed off by Finance Product Owner, Lead QE, and Systems Architect.</li>
          <li>⬜ Automated Gherkin test scenarios implemented covering Nominal, Math Imbalance, and Retry routines.</li>
          <li>⬜ Mathematical assertion unit tests pass: zero journal entries committed where &Sigma; Debits &ne; &Sigma; Credits.</li>
          <li>⬜ Idempotency cache tests verified: submitting identical payload 10 times produces exactly 1 ERP journal entry.</li>
          <li>⬜ Dead-Letter Queue alerting verified: malformed payloads trigger notification channels and quarantine records.</li>
          <li>⬜ User Acceptance Testing (UAT) completed with Senior Revenue Accountant validating general ledger account mappings.</li>
        </ul>
      </td>
    </tr>
  </tbody>
</table></div>

<!-- Link to Retail POS-to-Ledger Whitepaper -->
<div>
  <a href="businessone.html" class="btn btn-lg">
    <i class="fa-solid fa-cash-register"></i>Explore POS-to-Journal Integration
  </a>
</div>