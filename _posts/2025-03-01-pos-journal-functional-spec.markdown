---
#layout: default
modal-id: 4
date: 2025-03-01
img: pos_journal_integration.png
alt: POS Webhook Ingestion & Balanced Journal Entry Creation
project-date: March 2025
client: Omnichannel Retail Enterprise
category: Financial Systems & ERP Integration
---

<h3>Executive Summary</h3>
<p style="text-align: justify;">This project automates the reconciliation of Point of Sale (POS) transactions into a cloud General Ledger ERP, saving the finance team over 18 hours per month. By unbundling fees and taxes in real-time while strictly adhering to SOX 404 compliance, it completely eliminates manual data entry errors. The following specification bridges complex accounting requirements with robust technical API integrations to ensure secure and balanced financial reporting.</p>

<h3 style="text-align: left;">[FIN-INT-101] Functional Specification: POS Webhook Ingestion &amp; Balanced Journal Entry Creation</h3>

<h4 style="text-align: left;">📋 Ticket Completion Status</h4>
<div class="table-responsive">
<table class="table table-bordered">
  <tbody>
    <tr><td style="width: 25%; text-align: left;">🐛 <strong>Type</strong></td><td style="text-align: justify;">Story</td></tr>
    <tr><td style="text-align: left;">⏫ <strong>Priority</strong></td><td style="text-align: justify;">High</td></tr>
    <tr><td style="text-align: left;">🏷️ <strong>Labels</strong></td><td style="text-align: justify;"><code>Functional Requirement</code> <code>Finance-Integration</code> <code>POS-Engine</code> <code>General-Ledger</code> <code>ERP</code></td></tr>
    <tr><td style="text-align: left;">⭐ <strong>Ticket Status</strong></td><td style="text-align: justify;"><code>UNDER CONSTRUCTION</code></td></tr>
    <tr><td style="text-align: left;">👥 <strong>Ticket Owner</strong></td><td style="text-align: justify;">Business Analyst</td></tr>
    <tr><td style="text-align: left;">✅ <strong>Reviewed and Signed off</strong></td><td style="text-align: justify;">Product Owner (Finance), Lead Integration Architect, Lead QE, Lead Dev, Principal Controller, SME/Dev</td></tr>
  </tbody>
</table>
</div>

<hr>

<h3 style="text-align: left;">(1) Business Context &amp; User Story</h3>

<h4 style="text-align: left;">(1.1) Summary</h4>
<div class="table-responsive">
<table class="table table-bordered">
  <tbody>
    <tr><td style="width: 20%; text-align: left;"><strong>As a</strong></td><td style="text-align: justify;">Corporate Revenue Accountant</td></tr>
    <tr><td style="text-align: left;"><strong>I want</strong></td><td style="text-align: justify;">inbound POS card transaction webhooks to automatically unbundle gross sales into merchandise revenue, sales tax liabilities, and merchant processing fees, posting balanced double-entry journals into the cloud ERP</td></tr>
    <tr>
      <td style="text-align: left;"><strong>So that I</strong></td>
      <td style="text-align: justify;">
        • Recognize real-time revenue and operational fee expenses immediately at the point of sale<br>
        • Eliminate manual spreadsheet journal uploads during month-end close<br>
        • Mathematically prevent unbalanced financial postings from reaching the general ledger
      </td>
    </tr>
  </tbody>
</table>
</div>

<h4 style="text-align: left;">(1.2) Operational Elicitation and System Requirements</h4>
<div class="table-responsive">
<table class="table table-bordered" style="font-size: 0.9em;">
  <thead style="background: #f1f5f9;">
    <tr>
      <th style="width: 20%; text-align: left;">Dimension</th>
      <th style="width: 40%; text-align: left;">(i) Operational Elicitation (As-Is Finding)</th>
      <th style="width: 40%; text-align: left;">(ii) System Requirements (To-Be Contract)</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left;"><strong>Discovery Source</strong></td>
      <td style="text-align: justify;"><strong>Month-End Close Observation &amp; Incident Logs:</strong> Finance manually downloads weekly CSV dumps from payment processors and POS back-offices, consuming 18+ hours per close cycle.</td>
      <td style="text-align: justify;"><strong>Event-Driven Architecture:</strong> Ingestion microservice exposes a secure public endpoint accepting real-time HTTPS JSON webhooks as card transactions settle (T+0).</td>
    </tr>
    <tr>
      <td style="text-align: left;"><strong>Fee Recognition</strong></td>
      <td style="text-align: justify;"><strong>Lump-Sum Opacity:</strong> Merchant fees (2.6% + $0.10) are deducted as a net lump-sum payout days later, forcing accounting to estimate processing expenses via weekly approximations.</td>
      <td style="text-align: justify;"><strong>Transaction-Level Fee Unbundling:</strong> Middleware parses <code>fee_breakdown.amount</code> directly from the payload and immediately debits <code>GL 6200 - Payment Processing Fees</code>.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><strong>Tax Segregation</strong></td>
      <td style="text-align: justify;"><strong>Tax Liability Variance:</strong> Store managers report gross terminal totals; staff accountants manually calculate tax splits in Excel, causing periodic cent-level transposition errors.</td>
      <td style="text-align: justify;"><strong>Statutory Liability Segregation:</strong> Middleware extracts <code>tax_breakdown.total</code> and credits <code>GL 2200 - Sales Tax Payable</code> as a dedicated current liability balance.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><strong>Ledger Integrity</strong></td>
      <td style="text-align: justify;"><strong>Unbalanced Manual Commits:</strong> Manual CSV journal entry uploads fail 3.8% of the time due to human entry rounding differences where Debits &ne; Credits.</td>
      <td style="text-align: justify;"><strong>Mathematical Equilibrium Assertion:</strong> Integration engine enforces a pre-commit assertion: <code>&Sigma; Debits - &Sigma; Credits = $0.00</code> prior to invoking ERP journal APIs.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><strong>Cost Center Tagging</strong></td>
      <td style="text-align: justify;"><strong>Centralized Dumping:</strong> Sales book to a generic corporate clearing account because register operators do not format store numbers consistently.</td>
      <td style="text-align: justify;"><strong>Automated Segment Tagging:</strong> Middleware maps <code>payload.store_id</code> against the ERP Cost Center master directory, tagging every ledger line to the exact store branch.</td>
    </tr>
  </tbody>
</table>
</div>

<hr>

<h3 style="text-align: left;">(2) Acceptance Criteria (Behavioral &amp; Data Engine)</h3>

<h4 style="text-align: left;">(2.1) Environmental &amp; Operational Preconditions</h4>
<ul style="text-align: justify; padding-left: 20px; line-height: 1.7;">
  <li style="margin-bottom: 6px;"><strong>Security &amp; Auth:</strong> Webhooks dispatched over TLS 1.3 with a valid HMAC-SHA256 signature matching the shared secret in the key vault.</li>
  <li style="margin-bottom: 6px;"><strong>Entity Availability:</strong> Target General Ledger accounts (<code>1050</code>, <code>6200</code>, <code>4100</code>, <code>2200</code>) and store <code>Cost_Center</code> entities must reside in an <code>ACTIVE</code> status within the ERP Chart of Accounts.</li>
  <li style="margin-bottom: 6px;"><strong>Fiscal Calendar:</strong> Transaction timestamp must fall within an open accounting period in the corporate ERP general ledger.</li>
</ul>

<h4 style="text-align: left;">(2.2) Common Non-Functional Baselines</h4>
<ul style="text-align: justify; padding-left: 20px; line-height: 1.7;">
  <li style="margin-bottom: 6px;"><strong>Payload Ingestion SLA:</strong> Webhook acknowledgment (<code>HTTP 200 OK</code>) must return within &le; 300ms of socket receipt.</li>
  <li style="margin-bottom: 6px;"><strong>Transactional Atomicity:</strong> All ledger legs must commit within an atomic ACID boundary; partial or single-leg journal entries must never post.</li>
  <li style="margin-bottom: 6px;"><strong>Audit Lineage:</strong> Every transaction must preserve raw ingress JSON payloads and the returned ERP Journal Document ID within an immutable audit table.</li>
</ul>

<h4 style="text-align: left;">(2.3) Functional Scenarios (Gherkin Syntax)</h4>

<div class="table-responsive">
<table class="table table-bordered table-striped">
  <thead>
    <tr><th colspan="2" style="text-align: left;">Scenario 01: Nominal Path &mdash; Valid Card Sale &amp; Balanced Multi-Leg Journal Commit</th></tr>
  </thead>
  <tbody>
    <tr><td style="width: 15%; text-align: left;"><strong>GIVEN</strong></td><td style="text-align: justify;">a retail customer purchases an item ($5.00 base + $0.40 sales tax = $5.40 total charge) and POS captures payment</td></tr>
    <tr><td style="text-align: left;"><strong>WHEN</strong></td><td style="text-align: justify;">the payment gateway dispatches the <code>payment.captured</code> webhook containing gross $5.40, tax $0.40, fee $0.24, and net settlement $5.16</td></tr>
    <tr><td style="text-align: left;"><strong>THEN</strong></td><td style="text-align: justify;">the middleware balance assertion verifies: <code>&Sigma; Debits ($5.16 + $0.24) - &Sigma; Credits ($5.00 + $0.40) = $0.00</code></td></tr>
    <tr><td style="text-align: left;"><strong>AND</strong></td><td style="text-align: justify;">submits an atomic journal entry to ERP: Debit GL 1050 ($5.16), Debit GL 6200 ($0.24), Credit GL 4100 ($5.00), Credit GL 2200 ($0.40)</td></tr>
    <tr><td style="text-align: left;"><strong>AND</strong></td><td style="text-align: justify;">returns <code>HTTP 200 OK</code> to the gateway, updating internal state to <code>ERP_POSTED</code></td></tr>
  </tbody>
</table>
</div>

<div class="table-responsive">
<table class="table table-bordered table-striped">
  <thead>
    <tr><th colspan="2" style="text-align: left;">Scenario 02: Validation Exception &mdash; Double-Entry Imbalance Interception</th></tr>
  </thead>
  <tbody>
    <tr><td style="width: 15%; text-align: left;"><strong>GIVEN</strong></td><td style="text-align: justify;">an incoming transaction payload where line items sum incorrectly (e.g., Gross = $5.40, but Line Items + Tax = $5.39, |&Delta;| = $0.01)</td></tr>
    <tr><td style="text-align: left;"><strong>WHEN</strong></td><td style="text-align: justify;">the pre-commit balance assertion executes prior to ERP dispatch</td></tr>
    <tr><td style="text-align: left;"><strong>THEN</strong></td><td style="text-align: justify;">the engine must block ERP API execution, set status to <code>PAYLOAD_ERROR</code>, and route the message to the Dead-Letter Queue (DLQ)</td></tr>
    <tr><td style="text-align: left;"><strong>AND</strong></td><td style="text-align: justify;">dispatch an alert notification with error code <code>ERR-FIN-101</code> without blocking remaining queue processing</td></tr>
  </tbody>
</table>
</div>

<div class="table-responsive">
<table class="table table-bordered table-striped">
  <thead>
    <tr><th colspan="2" style="text-align: left;">Scenario 03: Infrastructure Exception &mdash; Transient ERP Downtime &amp; Idempotent Retry</th></tr>
  </thead>
  <tbody>
    <tr><td style="width: 15%; text-align: left;"><strong>GIVEN</strong></td><td style="text-align: justify;">a mathematically balanced transaction payload ready for ledger commit</td></tr>
    <tr><td style="text-align: left;"><strong>WHEN</strong></td><td style="text-align: justify;">the ERP API endpoint returns a transient error (<code>HTTP 429</code> or <code>HTTP 503</code>)</td></tr>
    <tr><td style="text-align: left;"><strong>THEN</strong></td><td style="text-align: justify;">the engine records the payload into a persistent retry table with state <code>NETWORK_TIMEOUT</code></td></tr>
    <tr><td style="text-align: left;"><strong>AND</strong></td><td style="text-align: justify;">executes exponential backoff retries (10s, 30s, 2m, 10m) supplying the original <code>transaction_id</code> as the <code>Idempotency-Key</code> header to prevent duplicate ledger postings</td></tr>
  </tbody>
</table>
</div>

<h4 style="text-align: left;">(2.4) Field-Level Input Specifications &amp; Data Dictionary</h4>
<div class="table-responsive">
<table class="table table-bordered" style="font-size: 0.9em;">
  <thead style="background: #f1f5f9;">
    <tr>
      <th style="width: 18%; text-align: left;">Field Label</th>
      <th style="width: 14%; text-align: left;">Ingress Method</th>
      <th style="width: 20%; text-align: left;">Data Format &amp; Mask</th>
      <th style="width: 10%; text-align: left;">Length</th>
      <th style="width: 8%; text-align: left;">Req?</th>
      <th style="width: 30%; text-align: left;">Validation Rules &amp; Error Handling</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left;"><code>transaction_id</code></td>
      <td style="text-align: left;">Webhook JSON</td>
      <td style="text-align: left;">Alphanumeric<br><code>^[a-zA-Z0-9_-]+$</code></td>
      <td style="text-align: left;">64 Chars</td>
      <td style="text-align: left;">Yes</td>
      <td style="text-align: justify;">Evaluated against Redis cache. If ID exists within 24h window, suppress duplicate ERP call.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><code>store_id</code></td>
      <td style="text-align: left;">Webhook JSON</td>
      <td style="text-align: left;">Alphanumeric<br><code>^STORE-[A-Z0-9]+$</code></td>
      <td style="text-align: left;">32 Chars</td>
      <td style="text-align: left;">Yes</td>
      <td style="text-align: justify;">Cross-referenced against ERP cost center master. If unmapped, halt with <code>ERR-FIN-102</code>.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><code>timestamp_utc</code></td>
      <td style="text-align: left;">Webhook JSON</td>
      <td style="text-align: left;">ISO-8601 UTC<br><code>YYYY-MM-DDTHH:mm:ss.sssZ</code></td>
      <td style="text-align: left;">24 Chars</td>
      <td style="text-align: left;">Yes</td>
      <td style="text-align: justify;">Evaluated against ERP fiscal calendar. Must fall within an open accounting period.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><code>subtotal</code></td>
      <td style="text-align: left;">Webhook JSON</td>
      <td style="text-align: left;">Decimal (0.00)</td>
      <td style="text-align: left;">12 Digits</td>
      <td style="text-align: left;">Yes</td>
      <td style="text-align: justify;">Merchandise net sales before tax. Must be &gt; 0.00. Mapped to Credit <code>GL 4100</code>.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><code>tax_total</code></td>
      <td style="text-align: left;">Webhook JSON</td>
      <td style="text-align: left;">Decimal (0.00)</td>
      <td style="text-align: left;">10 Digits</td>
      <td style="text-align: left;">Yes</td>
      <td style="text-align: justify;">Jurisdiction sales tax. Must match configured rate (&plusmn; $0.01). Mapped to Credit <code>GL 2200</code>.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><code>fee_amount</code></td>
      <td style="text-align: left;">Webhook JSON</td>
      <td style="text-align: left;">Decimal (0.00)</td>
      <td style="text-align: left;">10 Digits</td>
      <td style="text-align: left;">Yes</td>
      <td style="text-align: justify;">Processing fee assessed by gateway. Must be &ge; 0.00. Mapped to Debit <code>GL 6200</code>.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><code>net_settlement</code></td>
      <td style="text-align: left;">Webhook JSON</td>
      <td style="text-align: left;">Decimal (0.00)</td>
      <td style="text-align: left;">12 Digits</td>
      <td style="text-align: left;">Yes</td>
      <td style="text-align: justify;">Expected cash deposit: <code>(subtotal + tax_total) - fee_amount</code>. Mapped to Debit <code>GL 1050</code>.</td>
    </tr>
  </tbody>
</table>
</div>

<h4 style="text-align: left;">(2.5) Alerts, Feedback &amp; Error Messages</h4>
<div class="table-responsive">
<table class="table table-bordered" style="font-size: 0.9em;">
  <thead style="background: #f1f5f9;">
    <tr>
      <th style="width: 15%; text-align: left;">Message Code</th>
      <th style="width: 18%; text-align: left;">Trigger Condition</th>
      <th style="width: 16%; text-align: left;">Severity / Channel</th>
      <th style="width: 26%; text-align: left;">System Event Log Content</th>
      <th style="width: 25%; text-align: left;">Remediation Action</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left;"><code>ERR-FIN-101</code></td>
      <td style="text-align: left;">&Sigma; Debits &ne; &Sigma; Credits</td>
      <td style="text-align: left;">Critical / PagerDuty</td>
      <td style="text-align: left;"><code>UNBALANCED_PAYLOAD_REJECTED: Debit ($5.40) != Credit ($5.39). DLQ Enqueued.</code></td>
      <td style="text-align: justify;">Route payload to DLQ; alert on-call Systems BA to investigate line-item rounding variance.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><code>ERR-FIN-102</code></td>
      <td style="text-align: left;">Store ID unmapped in ERP</td>
      <td style="text-align: left;">High / Slack-Finance</td>
      <td style="text-align: left;"><code>INVALID_COST_CENTER: Store ID [STORE-99] not active in Chart of Accounts.</code></td>
      <td style="text-align: justify;">Quarantine message; prompt Finance Admin to map store node in ERP Cost Center master.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><code>WARN-FIN-105</code></td>
      <td style="text-align: left;">Ingestion retry count &gt; 3</td>
      <td style="text-align: left;">Medium / CloudWatch</td>
      <td style="text-align: left;"><code>RETRY_THRESHOLD_WARNING: TXN [txn_choc_98124] backoff interval 2m reached.</code></td>
      <td style="text-align: justify;">System retries automatically; escalates to ERP integration team if attempts exceed 5.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><code>SUCC-FIN-001</code></td>
      <td style="text-align: left;">Balanced journal committed</td>
      <td style="text-align: left;">Info / Audit Log</td>
      <td style="text-align: left;"><code>ERP_POSTED_SUCCESS: Journal Entry [JE-882190] created for TXN [txn_choc_98124].</code></td>
      <td style="text-align: justify;">Advance record state to <code>ERP_POSTED</code>; register in daily settlement pool.</td>
    </tr>
  </tbody>
</table>
</div>

<hr>

<h3 style="text-align: left;">(3) Security &amp; Access Control (RBAC Matrix)</h3>
<div class="table-responsive">
<table class="table table-bordered">
  <thead style="background: #f1f5f9;">
    <tr>
      <th style="width: 25%; text-align: left;">Enterprise Role Profile</th>
      <th style="width: 12%; text-align: center;">Execute / Ingest</th>
      <th style="width: 12%; text-align: center;">Query / Lookup</th>
      <th style="width: 15%; text-align: center;">Manual Override / DLQ Replay</th>
      <th style="width: 36%; text-align: left;">Functional Scope &amp; Governance Boundary</th>
    </tr>
  </thead>
  <tbody>
    <tr>
      <td style="text-align: left;"><strong>POS Gateway Service Account</strong></td>
      <td style="text-align: center; color: #16a34a; font-weight: bold;">Y</td>
      <td style="text-align: center; color: #dc2626; font-weight: bold;">N</td>
      <td style="text-align: center; color: #dc2626; font-weight: bold;">N</td>
      <td style="text-align: justify;">Automated programmatic account authorized strictly to post webhooks via mTLS.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><strong>Staff Revenue Accountant</strong></td>
      <td style="text-align: center; color: #dc2626; font-weight: bold;">N</td>
      <td style="text-align: center; color: #16a34a; font-weight: bold;">Y</td>
      <td style="text-align: center; color: #dc2626; font-weight: bold;">N</td>
      <td style="text-align: justify;">Read-only inspection of processed transaction logs, journal IDs, and daily reconciliation views.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><strong>Senior Finance Operations Lead</strong></td>
      <td style="text-align: center; color: #dc2626; font-weight: bold;">N</td>
      <td style="text-align: center; color: #16a34a; font-weight: bold;">Y</td>
      <td style="text-align: center; color: #16a34a; font-weight: bold;">Y</td>
      <td style="text-align: justify;">Authorized to inspect DLQ payloads, update cost-center lookup mappings, and trigger payload re-syncs.</td>
    </tr>
    <tr>
      <td style="text-align: left;"><strong>Internal / SOX Auditor</strong></td>
      <td style="text-align: center; color: #dc2626; font-weight: bold;">N</td>
      <td style="text-align: center; color: #16a34a; font-weight: bold;">Y</td>
      <td style="text-align: center; color: #dc2626; font-weight: bold;">N</td>
      <td style="text-align: justify;">Immutable read-only view across transaction audit tables, payload hashes, and timestamp logs.</td>
    </tr>
  </tbody>
</table>
</div>

<hr>

<h3 style="text-align: left;">(4) Scope Boundaries &amp; Operational Exclusions</h3>
<ul style="text-align: justify; padding-left: 20px; line-height: 1.7;">
  <li style="margin-bottom: 8px;"><strong>PCI-DSS Card Tokenization:</strong> Point-to-Point Encryption (P2PE), PAN handling, and card brand authorization handshakes are fully executed by the physical pin-pad and gateway; this service ingests post-authorization webhooks only.</li>
  <li style="margin-bottom: 8px;"><strong>Bank Payout Statement Clearing:</strong> Matching posted clearing balances in GL <code>1050</code> against electronic bank statements (BAI2 / CAMT.053) is governed separately under the 3-Way Bank Reconciliation specification.</li>
  <li style="margin-bottom: 8px;"><strong>Manual Journal Corrections:</strong> Direct ledger adjustments inside the ERP interface remain subject to native accounting controls and are outside the scope of this middleware service.</li>
</ul>

<hr>

<h3 style="text-align: left;">(5) Integration Contracts &amp; Technical Architecture</h3>

<h4 style="text-align: left;">(5.1) Inbound POS Webhook Payload Schema</h4>
<p style="text-align: justify;">Inbound payment events are accepted via HTTPS POST with mutual TLS (mTLS) and payload HMAC verification:</p>

<!-- Dark Mode Code Container -->
<div style="background: #0f172a; color: #f8fafc; border-radius: 6px; padding: 18px; margin: 18px 0; box-shadow: 0 4px 10px rgba(0,0,0,0.15); overflow-x: auto; text-align: left;">
<pre style="background: transparent; border: none; color: inherit; margin: 0; padding: 0; font-family: 'Courier New', Courier, monospace; font-size: 0.88em; line-height: 1.5; white-space: pre;"><span style="color: #94a3b8;">{</span>
  <span style="color: #38bdf8;">"transaction_id"</span>: <span style="color: #fde047;">"txn_choc_98124"</span>,
  <span style="color: #38bdf8;">"client_timestamp_utc"</span>: <span style="color: #fde047;">"2026-09-15T14:40:10.102Z"</span>,
  <span style="color: #38bdf8;">"store_id"</span>: <span style="color: #fde047;">"STORE-COLOMBO-01"</span>,
  <span style="color: #38bdf8;">"currency"</span>: <span style="color: #fde047;">"USD"</span>,
  <span style="color: #38bdf8;">"line_items"</span>: <span style="color: #94a3b8;">[</span>
    <span style="color: #94a3b8;">{</span>
      <span style="color: #38bdf8;">"item_sku"</span>: <span style="color: #fde047;">"CHOC-DARK-85"</span>,
      <span style="color: #38bdf8;">"quantity"</span>: <span style="color: #4ade80;">1</span>,
      <span style="color: #38bdf8;">"unit_price"</span>: <span style="color: #4ade80;">5.00</span>,
      <span style="color: #38bdf8;">"subtotal"</span>: <span style="color: #4ade80;">5.00</span>
    <span style="color: #94a3b8;">}</span>
  <span style="color: #94a3b8;">]</span>,
  <span style="color: #38bdf8;">"tax_breakdown"</span>: <span style="color: #94a3b8;">{</span>
    <span style="color: #38bdf8;">"jurisdiction"</span>: <span style="color: #fde047;">"LK-WESTERN"</span>,
    <span style="color: #38bdf8;">"effective_rate_pct"</span>: <span style="color: #4ade80;">8.00</span>,
    <span style="color: #38bdf8;">"total"</span>: <span style="color: #4ade80;">0.40</span>
  <span style="color: #94a3b8;">}</span>,
  <span style="color: #38bdf8;">"payment_capture"</span>: <span style="color: #94a3b8;">{</span>
    <span style="color: #38bdf8;">"payment_method"</span>: <span style="color: #fde047;">"EMV_CONTACTLESS"</span>,
    <span style="color: #38bdf8;">"card_brand"</span>: <span style="color: #fde047;">"VISA"</span>,
    <span style="color: #38bdf8;">"gross_amount"</span>: <span style="color: #4ade80;">5.40</span>,
    <span style="color: #38bdf8;">"fee_breakdown"</span>: <span style="color: #94a3b8;">{</span>
      <span style="color: #38bdf8;">"interchange_pct"</span>: <span style="color: #4ade80;">2.60</span>,
      <span style="color: #38bdf8;">"fixed_cut"</span>: <span style="color: #4ade80;">0.10</span>,
      <span style="color: #38bdf8;">"amount"</span>: <span style="color: #4ade80;">0.24</span>
    <span style="color: #94a3b8;">}</span>,
    <span style="color: #38bdf8;">"net_settlement"</span>: <span style="color: #4ade80;">5.16</span>
  <span style="color: #94a3b8;">}</span>
<span style="color: #94a3b8;">}</span></pre>
</div>

<h4 style="text-align: left;">(5.2) Persistence &amp; Database State Transitions</h4>

<!-- State Diagram Graphic -->
<div style="text-align: center; margin: 25px 0;">
  <img src="img/portfolio/pos_state_flowchart.png" 
       alt="POS Ingestion and Journal Lifecycle State Machine" 
       class="img-responsive img-centered" 
       style="width: 100%; max-width: 780px; border-radius: 8px; border: 1px solid #cbd5e1; box-shadow: 0 4px 14px rgba(15, 23, 42, 0.08); margin: 0 auto;">
</div>

<ul style="text-align: justify; padding-left: 20px; line-height: 1.7;">
  <li style="margin-bottom: 6px;"><strong>Target Database Entities:</strong> <code>INTEGRATION_TXN_LOG</code>, <code>ERP_JOURNAL_STAGING</code>, <code>DLQ_EXCEPTION_LOG</code>, <code>IDEMPOTENCY_CACHE</code>.</li>
  <li style="margin-bottom: 6px;"><strong>State Transition Engine:</strong> <code>QUEUED</code> &rarr; <code>VALIDATED</code> &rarr; <code>ERP_POSTED</code> processed inside an atomic transactional boundary.</li>
  <li style="margin-bottom: 6px;"><strong>Concurrency &amp; Idempotency:</strong> External UUID cached in Redis with a 24-hour TTL; duplicate calls with identical transaction IDs are suppressed prior to ERP API dispatch.</li>
</ul>

<ul style="text-align: justify; padding-left: 20px; line-height: 1.7;">
  <li style="margin-bottom: 6px;"><strong>Target Database Entities:</strong> <code>INTEGRATION_TXN_LOG</code>, <code>ERP_JOURNAL_STAGING</code>, <code>DLQ_EXCEPTION_LOG</code>, <code>IDEMPOTENCY_CACHE</code>.</li>
  <li style="margin-bottom: 6px;"><strong>State Transition Engine:</strong> <code>QUEUED</code> &rarr; <code>VALIDATED</code> &rarr; <code>ERP_POSTED</code> processed inside an atomic transactional boundary.</li>
  <li style="margin-bottom: 6px;"><strong>Concurrency &amp; Idempotency:</strong> External UUID cached in Redis with a 24-hour TTL; duplicate calls with identical transaction IDs are suppressed prior to ERP API dispatch.</li>
</ul>

<hr>

<h3 style="text-align: left;">(6) Traceability &amp; Definition of Done (DoD)</h3>

<h4 style="text-align: left;">(6.1) Upstream Traceability</h4>
<div class="table-responsive">
<table class="table table-bordered">
  <tbody>
    <tr><td style="width: 25%; text-align: left;"><strong>Jira Epic Link</strong></td><td style="text-align: justify;"><code>[FIN-EPIC-4400] Omnichannel Retail POS to Cloud ERP General Ledger Modernization</code></td></tr>
    <tr><td style="text-align: left;"><strong>Regulatory Anchor</strong></td><td style="text-align: justify;">Sarbanes-Oxley (SOX) Section 404 (Internal Controls) &amp; ASC 606 (Revenue Recognition Standards)</td></tr>
    <tr><td style="text-align: left;"><strong>Target Observability SLI</strong></td><td style="text-align: justify;">Webhook ingestion to ERP journal creation latency: &le; 1200ms at p95</td></tr>
  </tbody>
</table>
</div>

<h4 style="text-align: left;">(6.2) Definition of Done Checklist</h4>
<div class="table-responsive">
<table class="table table-bordered">
  <tbody>
    <tr>
      <td>
        <ul style="text-align: justify; padding-left: 20px; margin-bottom: 0; line-height: 1.8;">
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
</table>
</div>

<!-- Link to Retail POS-to-Ledger Whitepaper -->
<div style="text-align: center; margin: 30px auto 20px; width: 100%;">
  <a href="businessone.html" class="btn btn-lg" style="
    background-color: #1971b3;
    border-color: #1971b3;
    color: #ffffff;
    font-weight: 600;
    padding: 12px 16px;
    border-radius: 4px;
    text-decoration: none;
    display: inline-block;
    width: 100%;
    max-width: 420px;
    box-sizing: border-box;
    box-shadow: 0 3px 8px rgba(0,0,0,0.12);
  ">
    <i class="fa-solid fa-cash-register" style="margin-right: 8px;"></i>Explore POS-to-Journal Integration
  </a>
</div>