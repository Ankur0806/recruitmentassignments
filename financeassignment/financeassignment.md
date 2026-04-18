# Bank Reconciliation Assignment

**A skills assessment for prospective Fellows**

---

## 1. Situation

You have been given two files representing the same three-month period (01-Jan-2026 to 31-Mar-2026) for **Arka Tea Traders Pvt Ltd**, a Kochi-based wholesale tea supplier:

1. **`Arka_Tea_Traders_Zoho_Ledger.csv`** — an export from the company's Zoho Books system, showing the internal ledger of its HDFC current account.
2. **`Arka_Tea_Traders_HDFC_Statement.pdf`** — the bank's own statement for the same account and period.

The Finance head has noticed that Zoho's closing balance does not match the HDFC closing balance, and has asked you to investigate.

She wants three things from you:

- A clear list of what's causing the gap.
- A formal Bank Reconciliation Statement.
- A memo recommending what to change so this doesn't keep happening.

## 2. Why this matters

A bank reconciliation is one of the oldest accounting controls in existence. Its whole point is to make sure the company's records of a bank account match what the bank itself says — and when they don't, to explain *why* in terms that distinguish genuine errors from legitimate timing.

In real organisations, a stale reconciliation is how frauds hide, how misclassifications compound, and how founders discover ₹X lakh of "missing money" that was never actually missing — just misunderstood.

How you approach this task tells us a lot about how you will approach reality.

## 3. Your Task

### Part A — Discrepancy Register

Compare the two files and identify every item where they do not match. For each one, produce a row in a spreadsheet (CSV or XLSX) with the following columns:

| Column | What goes here |
|---|---|
| S.No. | Serial number |
| Date | Date of the item (use the bank's date where they differ) |
| Description | Short description of the transaction |
| Zoho Amount | What Zoho shows (blank if not in Zoho) |
| Bank Amount | What the bank shows (blank if not in bank) |
| Difference | Numeric gap |
| Classification | Type A / B / C / D (see below) |
| Root Cause | One-line explanation of why this happened |
| Correction Required | What needs to change (in Zoho, not in bank) |

**Classification codes:**

- **Type A** — Error in Zoho (missing entry, wrong amount, duplicate, wrong date)
- **Type B** — Error in Bank (rare, but possible — e.g., bank charge wrongly levied)
- **Type C** — Timing difference (cheque issued but not presented; deposit-in-transit; etc.) — no error, just a legitimate lag
- **Type D** — Prior-period / opening balance issue

### Part B — Bank Reconciliation Statement

Produce a formal BRS that starts with the Zoho closing balance and, through a series of documented adjustments, arrives at the HDFC closing balance. Standard BRS format. Format doesn't need to be fancy — clarity matters more than aesthetics.

### Part C — Correcting Journal Entries

For each Type A (error in Zoho) item, write the journal entry that would correct the books. Use standard double-entry format:

```
Dr [Account] ........ ₹X,XXX
    Cr [Account] .... ₹X,XXX
(Narration: what this entry does and why)
```

You do not need to post journal entries for Type C items — timing differences don't require correction, only disclosure.

### Part D — Memo to Finance Head

Write a short memo (max 500 words, plain prose) addressed to the Finance head. It should answer:

1. **What** you found — summarised, not exhaustively listed.
2. **Why** this happened — root-cause analysis. Don't just list symptoms; go up one level. Why did six Type A errors accumulate in a single quarter?
3. **What** should change — specific, actionable process recommendations.

The memo is the part we read most carefully. It tells us how you think.

## 4. Deliverables

Submit four files:

1. `Part_A_Discrepancy_Register.xlsx` (or `.csv`)
2. `Part_B_Bank_Reconciliation_Statement.pdf` (or `.md` / `.docx`)
3. `Part_C_Correcting_Journal_Entries.md` (or inside the Part B document — your choice)
4. `Part_D_Memo.md` (or `.pdf` / `.docx`)

Compress all four into a single ZIP file named `<YourName>_Arka_BankRecon.zip`.

## 5. Rules of Engagement

- **Time budget: 3 hours** from the moment you open the files. We assess pace as well as accuracy. If you spend 8 hours on this, your output should be proportionally better — but we will ask you how long you took.
- **You may use any AI tool** — Claude, ChatGPT, Gemini, whatever works for you. We encourage it. But your final output must reflect your thinking, not the tool's. Show working, make the reasoning visible, and be prepared to defend every number.
- **Cite your AI usage honestly.** In the Part D memo, add a one-line postscript: *"AI tools used: [names]. Approximate share of task: [%]."* We care about honesty, not about whether you used AI.
- **No partial submissions.** Submit all four parts or none.
- **Do not ask us clarifying questions** about the data. Ambiguity is part of the test — handle it with documented assumptions.

## 6. What We Are Evaluating

| Skill | Signal |
|---|---|
| **Attention to detail** | Did you find all the discrepancies, or did you stop at the easy ones? |
| **Accounting literacy** | Are your journal entries correct, with the right debit/credit sides? |
| **Classification judgement** | Can you tell a timing difference from an error? |
| **Root-cause reasoning** | Do you explain *why*, not just *what*? |
| **Written communication** | Is your memo clear, specific, and executive-appropriate? |
| **Effective AI usage** | Did the tool make you faster without making you lazy? |
| **Intellectual honesty** | Did you own what you don't know, or bluff? |

## 7. A Few Hints

- There are **more than five** material discrepancies. Do not stop after the first two you find.
- **The opening balance gap is real** and is the largest single item. Any reconciliation that does not address it is incomplete.
- **Some "mismatches" are not errors.** A cheque issued on 28th March that the vendor banks on 2nd April is a timing item, not a missing entry. Your classification needs to reflect this.
- **Narrations are not identical between the two files.** Bank narrations follow SWIFT/NEFT/UPI coding conventions; Zoho narrations are human-readable. Matching by description alone will fail — use date + amount + context.
- **Not every discrepancy is large.** Two small bank-originated items (one credit, one debit) tend to get missed because they look insignificant individually. They're not insignificant collectively.
- **Assume no GST implications** for this exercise. Focus on the bank reconciliation itself. The "Charges — QTR Maintenance + GST" line in the bank statement should be treated as a single composite debit, not split into base + GST.

## 8. Format Conventions

- All amounts in **INR**
- All dates in **DD/MM/YYYY** format
- Debit / Credit interpreted in the context of the bank account as an asset:
  - Money **IN** = Debit (increases balance)
  - Money **OUT** = Credit (decreases balance)
- If you disagree with any of these conventions, note your alternative clearly and use it consistently.

---

*Good luck. We read every submission carefully — this is not a throwaway filter.*