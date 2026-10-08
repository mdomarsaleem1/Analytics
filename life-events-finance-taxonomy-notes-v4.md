# Life Events Finance Taxonomy v4: Companion Notes

> **SUPERSEDES v3.** This is the current version. v3 notes are retained for reference.

Companion to `life-events-finance-taxonomy-v4.csv`. Built October 8, 2026 by adding a `transcript_keywords` column to every row of the v3 taxonomy (813 rows, 41 L1 domains, 769 unique L3 events). No other field was altered: all 12 original columns are byte-identical to v3.

## Column definitions

- **L1**: The broad event domain, framed around the customer's life or business rather than the bank's products. There are 41 domains: 14 carried over from v2, 27 new.
- **L2**: The event category within a domain (for example, Job Loss and Income Interruption). Domains researched without an L2 layer (cards, investment banking, auto lending) use the L1 name as the L2.
- **L3**: The specific labelable event (for example, Involuntary job loss or layoff). There are 769 unique L3 names.
- **impact_order**: `1` for first-order impact, meaning the event directly changes cash flow, savings, borrowing need, risk exposure, or credit behavior. `2` for second-order impact, meaning the effect acts through a triggered first-order event or a second-hop consequence.
- **impact_description**: One sentence on what changes financially and why it matters.
- **affected_products**: The product lines touched, semicolon separated. New-domain research used code tokens (BCHK, SBA7, RCF); these are mapped to readable names consistent with v2's style.
- **cross_sell_opportunities**: One to three concrete, tasteful plays an agent or banker could raise, semicolon separated.
- **servicing_actions**: One to two proactive servicing actions, semicolon separated.
- **example_utterance**: One realistic caller or chat sentence that would map to this L3, usable as training or few-shot data for the labeling model.
- **conversation_signal**: High, Medium, or Low. How likely the event is to be volunteered directly in a service call or chat. High means customers routinely say it ("I was laid off", "we're buying a house"). Medium means it surfaces in specific servicing contexts or indirectly. Low means it is rarely or never mentioned directly because of sensitivity, stigma, privacy, or irrelevance to the banking interaction (miscarriage, infertility treatment, domestic violence exit, addiction treatment, elder exploitation). Low-signal labels are still valid taxonomy entries, but they are weak targets for conversation labeling.
- **system_capture**: Direct, Proxy, or None. Whether the event can be detected reactively from bank systems and operational data. Direct means a system event maps nearly one to one to the life event (a 529 distribution, a mortgage payoff, a loan application). Proxy means systems see a correlated footprint but not the cause. None means no reliable system footprint exists.
- **signal_notes**: One sentence stating the combination strategy: what the system sees, what is missing (the why), and how the two signals work together. For Proxy cases it names the disambiguating question. For wealth-transfer rows it names proactive leading indicators (client age plus balances plus missing beneficiaries, RMD age approaching).
- **transcript_keywords**: Five to ten lowercase keywords or short phrases per row, semicolon separated, for matching against call center transcripts and chat logs. Written in natural caller language (first person, informal, as actually spoken), with synonyms and common variants. Distinctive multi-word phrases are preferred over single generic words. For Low conversation_signal rows the keywords lean toward the proxy language callers actually use rather than clinical terms. For business, commercial, and investment-banking rows they use the speaker's professional register. See "Using transcript_keywords" below.

## The signal model: conversation versus system capture

The two new flags exist because neither channel sees the whole event. The canonical example is an address change. The core system records the new address (system_capture = Proxy), which proves a move happened but does not tell why. The why could be a new job, a divorce, a first home purchase, or downsizing, and each why carries a different cross-sell and servicing playbook. The taxonomy is the disambiguation layer between the system trigger and the agent's one question: when the system fires on an address change, the agent asks which kind of move it was, labels the life event, and applies the matching playbook.

The four useful combinations:

1. **High conversation, Direct system** (for example, mortgage application; business loan application): the highest-confidence labels. Use the conversation to confirm details and sequence the cross-sell.
2. **High conversation, Proxy system** (for example, job loss: direct deposit stops; new baby: 529 inquiry): customers volunteer it, systems see only the footprint. Ask the disambiguating question to confirm the trigger and size the need.
3. **Low conversation, Direct or Proxy system** (for example, minimum-payment revolving pattern; MCA entrapment via daily debits; minimum-payment behavior): the bank should trigger proactive outreach rather than wait for the call. Several of these rows say so explicitly in signal_notes.
4. **Low conversation, no system footprint** (for example, miscarriage, domestic violence exit, addiction treatment): keep as labels for banker-led discovery and sensitive handling, not for detection. Do not build triggers on them.

A note on commercial and investment-banking rows: their conversation_signal ratings assume banker, RM, and treasury channels. If the taxonomy is applied only to recorded call-center calls, those rows skew Low and Medium and need recalibration for that channel. The IB rows say this explicitly: most are coverage-trigger labels (mandate pursuit), not conversation labels.

## Using transcript_keywords

The keywords are seed lists for weak supervision, not exact-match rules. Recommended implementation: normalize transcripts the same way (lowercase, stem, strip punctuation), then use the keyword sets as labeling functions or as seed phrases for embedding-based retrieval, and always pair them with the conversation_signal flag. High-signal rows can carry more weight in a rules pass; Low-signal rows should be used for recall-oriented candidate generation (surface the conversation for human review) rather than auto-labeling, since the event is rarely stated outright.

Three examples from the file:

**1. Consumer, High signal: Involuntary job loss or layoff.** Keywords: `i was laid off; lost my job; got laid off; position was eliminated; company downsizing; let go from my job; my role was eliminated; laid off yesterday; worried about making; making my mortgage`. Note the synonym coverage (laid off, lost my job, let go, position eliminated) and the spillover into the financial consequence (worried about making the mortgage), which is how job loss actually surfaces on servicing calls.

**2. Business, High signal: Cash-flow crunch or at risk of missing payroll.** Keywords: `running payroll; first employee; hiring my first employee; payroll taxes; w-2 employees; payroll service; cash crunch; can't make payroll; making payroll; running out of cash`. The register shifts to the owner's vocabulary: payroll, cash crunch, w-2. On business calls the event arrives as an operational problem ("payroll is due Friday"), not a life-event announcement.

**3. Low signal, proxy phrasing: Discovering elder financial exploitation.** Keywords: `someone's taking mom's money; elder exploitation; caregiver stealing; unusual withdrawals from dad's account; tricked my elderly mother into wiring`. Nobody calls the bank to report "elder financial exploitation" in those words; the caller describes the proxy (unusual withdrawals, a caregiver, mom's money). The keywords follow the caller's language, and the conversation_signal = Low flag tells the labeling pipeline to treat matches as candidates for review, not confident labels.

Guidance that applies to all rows: stem and normalize at implementation (plurals, contractions, tense); do not use the keywords as exact-match rules, since phrasing varies and several L3s share vocabulary (for example, "divorce" appears across divorce filing, QDRO, and gray divorce rows); when two rows match, prefer the more specific L3 and use impact_order and signal_notes to break ties; re-validate keyword precision on a sample of real transcripts before production use, especially for Low-signal rows.

## What changed from v2 to v3

- **27 new L1 domains** across seven research domains: business banking and SMB and B2B (7 L1s: Business Formation, Growth and Expansion, Business Borrowing, Cash Management and Operations, B2B Trade and Commerce, Business Distress, Succession and Exit); cards (7 L1s across the cardholder lifecycle plus Business Cards); investment banking (5 L1s); auto lending (5 L1s); Travel, Lifestyle, and Deals (1 L1, 59 rows); Commercial Lending (1 L1, 53 rows); Wealth Transfer and Intergenerational Planning (1 L1, 64 rows).
- **3 new columns** on every row: conversation_signal, system_capture, signal_notes.
- **Backfill**: all 416 v2 rows were scored on the two new flags using the rubric above, with exact-name overrides for sensitive events (miscarriage, domestic violence exit, addiction treatment, elder exploitation, and others are all rated Low conversation signal). No other v2 field was altered.
- **Product vocabulary**: research code tokens mapped to readable names (for example, BCHK to Business Checking, SBA7 to SBA 7(a) Loan, RCF to Revolving Credit Facility, MA to M&A Advisory). One generic "Insurance" token was avoided; business contexts use Business Insurance and wealth contexts use Life Insurance.

## Counts by L1

813 total rows. 769 unique L3 events (626 first-order, 187 second-order). Signal distributions: conversation High 402, Medium 306, Low 105; system Direct 390, Proxy 396, None 27.

New L1s and row counts: Business Formation 18, Growth and Expansion 20, Business Borrowing 20, Cash Management and Operations 22, B2B Trade and Commerce 18, Business Distress 19, Succession and Exit 18, Card Acquisition and Onboarding 5, Account Management 9, Rewards, Fees and Benefits 6, Borrowing on the Card 5, Fraud, Disputes and Security 5, Delinquency, Hardship and Closure 6, Business Cards 4, M&A and Strategic Advisory 6, Equity Capital Markets 5, Debt Capital Markets and Leveraged Finance 6, Restructuring and Special Situations 3, Event-Driven Treasury and Risk 3, Shopping and Origination 7, Funding and Early Loan Life 4, Lease Lifecycle 4, Servicing and Refinancing 5, Distress (auto) 3, Travel, Lifestyle, and Deals 59, Commercial Lending 53, Wealth Transfer and Intergenerational Planning 64.

Carried-over v2 L1s: Family and Household 45, Employment and Income 40, Money Milestones and Financial Events 36, Retirement and Aging 36, Housing and Real Estate 35, Death, Estate, and Legacy 30, Legal and Identity 29, Health and Medical 29, Debt and Credit Management 28, Business and Self-Employment 27, Education and Training 26, Relocation and Mobility 22, Military Service and Veterans 17, Loss, Disaster, and Victimization 16.

## Cross-domain overlaps adjudicated

The researchers flagged possible duplicates across domains. Each was checked against the actual sibling content. The rule applied: duplicate a row only when the playbook genuinely differs by context; otherwise keep one canonical row and record the relationship here.

- **Kept as single canonical rows with cross-references** (same event, "also relevant in" another L2): seasonal inventory build-up, second-location opening, CRE purchase, equipment purchase, covenant waiver, invoice factoring, MCA refinance, partner capital contribution, late-paying customers, key-person insurance, partner dispute, covenant breach, missed loan payment, tax lien, partner buyout, buy-sell funding, buyer acquisition financing, owner death and disability continuity events. These live under their primary L1; the related L2s are noted as alternate label contexts, not separate rows.
- **Cards versus travel overlaps** (travel-triggered card events: rewards optimization, premium upgrade inquiries, retention calls, foreign-transaction questions): kept in both domains with distinct L3 names, because the trigger differs. Labeling rule: label the trigger (travel) in the travel domain and the product mechanics (upgrade, retention, fee) in the cards domain; a single conversation can carry both labels.
- **Investment banking versus commercial lending adjacency** (bridge financing, dividend recap, corporate distress, out-of-court restructuring): kept in both. Boundary: IB rows are the advisory and deal-execution lens; commercial rows are the borrower-side facility and relationship lens. A bridge loan commitment and a bridge-financed acquisition are two labels for related but distinct banking moments.
- **IB-003 leveraged buyout versus Business Distress L3-120 management buyout**: kept both; sponsor LBO and management buyout are the same financing shape at different scales with different counterparties.
- **Auto delinquency and repossession versus cards delinquency**: kept in both domains; cross-product distress is itself a signal, and the notes flag the correlation rather than merging.
- **One exact-name duplicate kept intentionally**: "Separating personal and business finances" appears under both Business and Self-Employment (v2, consumer lens) and Business Formation (v3, business-banking lens) with genuinely different playbooks, following the taxonomy's standing duplication convention.
- **Near-duplicates kept as distinct rows** (checked, playbooks differ): "Opening first business checking account" versus "First business bank account"; "Hiring first W-2 employee" versus "Hiring first employees"; "Closing or dissolving the business" versus "Closing or winding down a business"; "Inherited IRA management (10-year rule)" versus "Inherited IRA (10-year rule)"; "Family business succession planning" versus "Family business succession"; "Long-term care insurance purchase decision" versus "Long term care insurance purchase (midlife)"; "Dispute or chargeback filing" versus "Card dispute or chargeback". In each pair the v2 row is the consumer lens and the v3 row is the specialist lens (business banking, wealth, or cards lifecycle).

## Worked labeling examples

**1. Consumer: the address-change disambiguation.** The core system posts an address change (system_capture = Proxy). No call comes in. The playbook: trigger a servicing outreach, and the agent's one question is "did you move for a new job, or did you buy a place?" If the answer is a job move, the labels are Relocation and Mobility / Domestic Moves plus Employment and Income / New Employment and Advancement; plays are direct-deposit switch and relocation-expense coverage. If the answer is a first home purchase, the labels are Housing and Real Estate / Buying a Home; plays are homeowner insurance, HELOC education, and equity-building content. Same system event, two different label sets, two different revenue paths. This is the pattern the signal_notes column encodes.

**2. Business: the MCA trap.** Daily debits from a merchant cash advance funder appear in the business checking account (conversation_signal = Low, system_capture = Proxy; owners rarely volunteer this because of stigma). The label is Business Borrowing / Asset-based and alternative financing / Taking a merchant cash advance. The playbook is detection-led, not conversation-led: when the pattern appears, the agent's question is "are you trapped in a cash advance? We may be able to help you out of it," leading to the companion label Refinancing out of an MCA / high-cost debt and a term-loan consolidation play. Waiting for the owner to call about it misses the event entirely.

**3. Wealth transfer: proactive beneficiary-gap outreach.** The system finds a 68-year-old client with $900K across IRAs and a taxable account and no beneficiary on file (conversation_signal = Medium, system_capture = Direct; null beneficiary fields are directly queryable). The label is Wealth Transfer and Intergenerational Planning / Beneficiary designations / Beneficiary designation gap discovered. The playbook is fully proactive: the advisor calls before any life event forces the issue, combining the leading indicators (age plus balances plus missing beneficiaries) named in signal_notes. The cross-sell is the estate-planning attorney referral and trust review; the servicing action is the annual beneficiary health check. With roughly $36T expected to move to heirs over the next two decades and about half of inheritors not planning to keep the parents' advisor, this is the highest-ROI proactive motion in the taxonomy.

## Caveats

- Cross-sell plays and servicing actions are constructed for labelability. They are NOT validated against any bank's product shelf, pricing, or compliance constraints. Product and legal teams must bless them before they reach agents or models.
- Dollar figures and tax figures in impact descriptions (IVF costs, surrogacy costs, 2026 annual gift-tax exclusion, QCD limits, RMD ages and penalties) are research-time values from general sources. Verify before any external or customer-facing use.
- Commercial-lending signal ratings assume banker, RM, and treasury channels. Recalibrate if labeling only recorded call-center calls.
- State estate-tax thresholds change frequently; per-state validation is needed at implementation.
- Pilot labeling with inter-rater measurement is recommended for adjacent L3s (for example, separation versus divorce finalized; the two child-support rows; travel-triggered versus generic card events).
- IB rows are coverage triggers for mandate pursuit, not conversation labels; see the signal model section.
