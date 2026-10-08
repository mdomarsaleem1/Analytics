# Life Events Finance Taxonomy

A structured taxonomy of life events with first-order and second-order financial impacts, built for labeling call center calls, chat transcripts, and brand conversations to improve cross-sell and servicing.

## Files

- `life-events-finance-taxonomy-v4.csv` — the taxonomy (813 rows, 41 L1 domains, 769 unique L3 events)
- `life-events-finance-taxonomy-notes-v4.md` — companion notes: column definitions, the signal model, duplication conventions, worked labeling examples, and caveats

## Structure

Three levels, life-event-first framing (domains describe the customer's life, not the bank's products):

- **L1** — broad life-event domain (e.g. Family and Household, Business Formation, Wealth Transfer and Intergenerational Planning)
- **L2** — event category within a domain (e.g. Job Loss and Income Interruption)
- **L3** — the specific labelable event (e.g. Involuntary job loss or layoff). An L3 may appear under multiple L2s when the playbook genuinely differs by context; each mapping is its own row.

Coverage spans consumer life events from birth to death and estate settlement, plus business banking and SMB lending, cards lifecycle, investment banking, auto lending, travel and lifestyle, commercial lending, and wealth transfer.

## Columns

| Column | Meaning |
|---|---|
| L1, L2, L3 | Taxonomy levels |
| impact_order | 1 = directly changes cash flow, savings, borrowing need, risk, or credit behavior; 2 = indirect, via a triggered first-order event |
| impact_description | One-sentence financial impact |
| affected_products | Product lines touched |
| cross_sell_opportunities | Concrete plays an agent or banker could raise |
| servicing_actions | Proactive servicing moves |
| example_utterance | Realistic caller/chat sentence mapping to this event |
| conversation_signal | High / Medium / Low — how likely the event is volunteered on a call or chat |
| system_capture | Direct / Proxy / None — whether bank systems can detect it reactively |
| signal_notes | The observable proxy or leading indicator, and how system plus conversation signals combine |
| transcript_keywords | Caller-language keywords and phrases for transcript matching (weak-supervision seeds, not exact-match rules) |

## The signal model

Two independent ratings per event:

- **Conversation signal** answers: will they say it? Many high-value labels are low-signal here (sensitivity, stigma, privacy).
- **System capture** answers: can systems see it reactively? Roughly half the taxonomy is Proxy: the system sees a correlated footprint but not the cause. An address change proves a move but not why (new job, divorce, first home purchase). The system event should trigger the one disambiguating question, because each possible why carries a different playbook.

## Caveats

- Cross-sell and servicing plays are constructed for labelability. They are not validated against any institution's product shelf or compliance constraints; have product and legal bless them before use.
- Dollar and tax figures are research-time values. Verify before external use.
- Transcript keywords are seed lists for weak supervision. Stem and normalize at implementation, pair with the conversation-signal flag, and re-validate precision on real transcripts before production use.
