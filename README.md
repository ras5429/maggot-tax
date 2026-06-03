# Quarterly Estimated Tax Estimator — Federal + Colorado

A single-file web app for estimating **quarterly estimated tax payments** using the
**annualized income installment method** — IRS Worksheet 2‑9 for the federal side and
Colorado Form DR 0204 for the state side. Built for a married‑filing‑jointly household
with one W‑2 earner and one self‑employed spouse, where income varies through the year
and safe‑harbor methods aren't practical.

> ⚠️ **This is an estimate, not tax advice.** It implements the standard deduction, the
> QBI deduction, and self‑employment tax, but it does not cover every situation
> (itemized deductions, investment income / NIIT, credits beyond the basics, multi‑state
> income, etc.). Verify against the official worksheets or a tax professional before
> relying on the numbers.

---

## Features

- **Annualized method, done right.** Self‑employment tax (including the `× 0.9235` net‑earnings
  adjustment and the prorated Social Security wage‑base cap), the QBI deduction, the standard
  deduction, and the 2026 MFJ brackets are all applied per IRS Worksheet 2‑9.
- **Federal *and* Colorado** in one pass. Colorado uses federal taxable income as its starting
  point at the flat 4.40% rate, with the DR 0204 annualized percentages (17.5 / 35 / 52.5 / 70).
- **Stateless, quarter‑by‑quarter.** Pick the quarter you're filing; enter cumulative
  year‑to‑date figures for it and every prior quarter.
- **Underpaid / overpaid flags.** For each completed quarter it shows what you *should* have
  paid versus what you *did*, and the "pay now" amount automatically rolls any shortfall forward.
- **File‑based persistence.** Export a small JSON file at the end of a session and load it next
  quarter — your inputs are the source of truth, and results are always recomputed.
- **Externalized tax‑year config.** Every value that changes from year to year lives in one place.
- **Private by design.** 100% client‑side. No backend, no analytics, no data leaves the browser.

---

## Quick start

### Run locally
Just open `index.html` in any modern browser — no build step, no server.


## Using the app

1. **Choose the quarter** you're filing at the top.
2. **Enter cumulative year‑to‑date numbers** for that quarter and each earlier one:
   - Self‑employment net profit (after business expenses)
   - W‑2 wages
   - Federal tax withheld (YTD)
   - Colorado tax withheld (YTD)
   - The estimated payment you made (or plan to make) for each quarter
3. Read the **federal** and **Colorado** amounts to pay, plus the per‑quarter status table.
4. Expand **"Show the worksheet math"** to trace every line.
5. **Export this year's file** to save your inputs; **Load saved file** next quarter to resume.

All amounts are **cumulative** (total so far this year), not just that quarter's slice —
that's how the annualized worksheet works.

---

## Updating for a new tax year

Everything that changes year to year is isolated in a single config block — you never touch
the calculation logic. Two ways to do it:

**Option A — edit in place (permanent).**
Open the HTML and edit the `TAX_CONFIG` block at the top of the `<script>` (it's inside a
labeled banner; the lines marked ★ are the ones that move each year). Save.

**Option B — per‑year config files.**
Copy `tax-config-2026.json` to `tax-config-2027.json`, update the values, and use the
**Load config file** button in the app to run with it. Keep one file per year in the repo.

### What to update each year

| Field | Notes |
|---|---|
| `taxYear` | Display year. |
| `standardDeduction.MFJ` | ★ Annual inflation adjustment. |
| `federalBrackets_MFJ` | ★ `[upperBound, rate]` pairs; use `null` for the top bracket. |
| `socialSecurityWageBase` | ★ SSA announces this each fall. |
| `qbiThreshold_MFJ` | ★ Taxable‑income threshold for the full 20% QBI deduction. |
| `coloradoRate` | ★ Moves occasionally (TABOR). |
| `periods[].due` | ★ Due dates shift for weekends/holidays. |
| `rates` block | Statutory (SS 12.4%, Medicare 2.9%, etc.) — rarely changes. |
| `periods[].months`, `federalPct`, `coloradoPct` | Statutory schedule — rarely changes. |

Values that are mechanically derived — annualization factors, the prorated Social Security
caps, and the SE‑tax factors — are computed from the fields above, so you don't edit them.

> Note: the in‑app **Load config file** applies for that browser session only. To change the
> permanent default the page ships with, edit the `TAX_CONFIG` block (Option A).

---

## How the calculation works

For each annualization period (cutoffs at Mar 31, May 31, Aug 31, Dec 31):

1. **SE tax** — net profit `× 0.9235` → net earnings; Social Security portion (capped at the
   prorated wage base) `+` Medicare portion, annualized. Half of it becomes a deduction.
2. **Income tax** — annualized AGI `−` standard deduction `−` QBI deduction → taxable income →
   federal MFJ brackets. (Additional Medicare Tax applies above $250k MFJ.)
3. **Required payment** — total tax `×` the period's applicable percentage (fed builds to 90%,
   Colorado to 70%), minus withholding to date and prior payments.
4. **Colorado** — federal taxable income `×` 4.40%, then the DR 0204 percentages.

### A note on self‑employment tax
SE tax is **individual**. Only the self‑employed person's own income — and their own W‑2 wages,
if any — affect it. A spouse's W‑2 wages do **not** belong in the SE‑tax calculation. The
"self‑employed person's own W‑2 wages" field defaults to `$0` for exactly this reason.

### Sources (2026 values)
- IRS **Publication 505**, Worksheet 2‑9 (Annualized Estimated Tax)
- 2026 MFJ standard deduction **$32,200**; 2026 MFJ brackets
- SSA 2026 Social Security wage base **$184,500**
- Colorado **DR 0204** / **DR 0104EP**; flat rate **4.40%**; required annual amount = lesser of
  70% of current‑year liability or 100%/110% of prior year

---

## Limitations

- Married filing jointly only.
- Standard deduction only (no itemizing).
- No investment income / Net Investment Income Tax, and only the basic credits.
- No state other than Colorado; no multi‑state allocation.
- Federal bracket boundaries above the 24% rate are best‑read 2026 figures — fine for typical
  income, but worth a check if income is high.

---

## File structure

```
index.html            # the app (rename from quarterly-tax-estimator.html)
tax-config-2026.json  # editable tax-year config (also embedded in the app as the default)
README.md             # this file
```

---

## License

Personal project — use and adapt freely. No warranty; see the disclaimer above.
