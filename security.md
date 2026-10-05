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

#### Impact

The consequence for the business if the threat is realised:

| Rating | Meaning |
|---|---|
| Very High | Threatens the survival of the business, or breaches client data |
| High | Serious financial, legal or reputational damage |
| Moderate | Real disruption to operations, recoverable |
| Low | Brief disruption, no lasting damage |
| Very Low | Absorbed in normal operation |

#### Risk determination

Risk is not calculated arithmetically. The spreadsheet looks the pair up in the following matrix, held on its `RiskValues` sheet:

| Likelihood ↓ &nbsp; Impact → | Very Low | Low | Moderate | High | Very High |
|---|---|---|---|---|---|
| **Very High** | Very Low | Low | Moderate | High | **Very High** |
| **High** | Very Low | Low | Moderate | High | **Very High** |
| **Moderate** | Very Low | Low | Moderate | Moderate | High |
| **Low** | Very Low | Low | Low | Low | Moderate |
| **Very Low** | Very Low | Very Low | Very Low | Low | Low |

The matrix is deliberately weighted towards impact. A threat that is almost certain to occur but causes trivial damage still resolves to Very Low, while a Very High risk requires both a Very High impact and at least a High likelihood. This means the assessment prioritises what would genuinely hurt the business rather than what merely happens often.

**Ranking.** The 35 entries are ordered by risk rating, then by impact, then by likelihood, and numbered 1 to 35. Ranks are unique, so there is a single unambiguous priority order.

**Worked example — rank 1.** `T2V1A1` pairs the threat *software attacks* with the asset *client administrative credentials*. The vulnerability is that phishing or malware on a staff workstation can harvest stored client admin credentials. We rated the likelihood **High**, because phishing against IT service providers is constant and automated, and the impact **Very High**, because those credentials unlock every client network, not just Westline's own. High combined with Very High gives **Very High** risk, and the highest impact places it at rank 1.

Across our 35 entries we used likelihoods of High, Moderate and Low, and impacts of Very High, High and Moderate. We did not rate anything Very High likelihood or Very Low impact, because for a business of this size neither extreme was defensible for any of the pairings we identified.

The assessment is scoped to the business described in our assumptions and to the network we designed and built in `network.md` and hardened in `harden.md`.

### 1.2 Assets

We identified **26 assets across all six asset types**, drawn directly from our own network design and assumptions:

| Type | Count | Examples |
|---|---|---|
| **Data** | 6 | Client administrative credentials; client network documentation; client remote access details; client backup data; support tickets and client records; router root password and SSH private key |
| Hardware | 4 | Router/firewall (OpenWRT); web server (51.1.20.10); five staff and sysadmin workstations; network printer (51.1.10.20) |
| Software | 4 | OpenWRT firmware; uhttpd; Dropbear SSH server; dnsmasq |
| Network | 4 | WAN link (53.10.1.0/24); staff LAN (51.1.10.0/24); server network (51.1.20.0/24); management network (51.1.30.0/24) |
| People | 4 | Owner/principal consultant; two IT support technicians; administrative staff member; part-time systems administrator |
| Processes | 4 | Handling client support requests; administering client systems; managing client backups; router administration over SSH |

The hardware, network and software assets correspond to the devices, subnets and services documented in `network.md` Sections 2, 3 and 5 and `harden.md` Section 1.4. The people correspond to the five staff in our assumptions (`network.md` Section 1.3).

The data assets reflect what an IT services provider actually holds. As argued in `network.md` Section 1.2, Westline stores administrative credentials, network documentation, remote access details and backup data belonging to its **client** businesses. This is what makes the business a more attractive target than an ordinary firm of its size.
