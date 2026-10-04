# Risk Assessment and Security Controls

COIT20246 Cyber Security and Networking — Term 2, 2026
SYD Group 6 — Md Arman Joarder (12312653), Atikur Rahman Mimmoy (12327451)

This section assesses the cyber security risks facing Westline IT Solutions, the managed IT services provider described in `network.md` Section 1, and recommends controls for the asset that our assessment identifies as carrying the highest risk.

---

## 1. Cyber Security Risk Assessment

### 1.1 Method

We used the TVAMatrix risk assessment spreadsheet provided in this unit, which follows the qualitative approach described in NIST SP 800-30. The completed spreadsheet is in this repository as [`risk-assessment.xlsx`](risk-assessment.xlsx).

The method works in four stages:

1. **Identify assets.** Every asset is listed with its type — Data, Hardware, Software, Network, People or Processes.
2. **Identify threats and vulnerabilities.** For each meaningful pairing of a threat with an asset, we describe the specific vulnerability — the way that threat could actually be realised against that asset in this business. Each pairing is given a TVA identifier such as `T2V1A1`, meaning threat 2 exploiting vulnerability 1 against asset 1.
3. **Rate each TVA.** Each triple is given a Likelihood and an Impact on a five-point scale.
4. **Determine and rank risk.** The spreadsheet looks the Likelihood and Impact pair up in its risk matrix to produce a risk rating, and every TVA is ranked in order so the highest priorities are clear.

#### Likelihood

How probable it is that the threat successfully exploits the vulnerability against this asset, judged over a twelve-month period:

| Rating | Meaning |
|---|---|
| Very High | Expected to occur repeatedly |
| High | Expected to occur at least once |
| Moderate | Could reasonably occur |
| Low | Unlikely, but possible |
| Very Low | Would require unusual circumstances |
