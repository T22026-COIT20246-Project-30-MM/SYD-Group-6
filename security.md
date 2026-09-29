# Risk Assessment and Security Controls

COIT20246 Cyber Security and Networking — Term 2, 2026
SYD Group 6 — Md Arman Joarder (12312653), Atikur Rahman Mimmoy (12327451)

This section assesses the cyber security risks facing Westline IT Solutions, the managed IT services provider described in `network.md` Section 1, and recommends controls for the asset that our assessment identifies as carrying the highest risk.

---

## 1. Cyber Security Risk Assessment

### 1.1 Method

We used the TVAMatrix risk assessment spreadsheet provided in this unit, following the simplified process covered in the unit material. The completed spreadsheet is in this repository as [`risk-assessment.xlsx`](risk-assessment.xlsx).

The method works in four stages:

1. **Identify assets.** Every asset is listed with its type — Data, Hardware, Software, Network, People or Processes.
2. **Identify threats and vulnerabilities.** For each meaningful pairing of a threat with an asset, we describe the specific vulnerability — the way that threat could actually be realised against that asset in this business.
3. **Rate each TVA.** Each threat/vulnerability/asset triple is given a Likelihood and an Impact, and the spreadsheet calculates the resulting Risk from the unit's risk matrix.
4. **Rank.** Every TVA is ranked in order of risk, so the highest priorities are clear.

The assessment is scoped to the business described in our assumptions and to the network we designed and built in `network.md` and hardened in `harden.md`.
