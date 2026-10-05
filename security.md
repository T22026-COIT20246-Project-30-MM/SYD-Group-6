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

### 1.3 Threats and Vulnerabilities

We recorded **35 threat/vulnerability/asset entries covering all 12 information security threats** — the specification requires at least 8.

| Threats covered | TVA entries |
|---|---|
| 1 People errors | 9 |
| 2 Software attacks | 12 |
| 3 Information extortion | 1 |
| 4 Espionage and trespass | 4 |
| 5 Theft | 1 |
| 6 Technological obsolescence | 1 |
| 7 Forces of nature | 1 |
| 8 Technical hardware failures | 2 |
| 9 Technical software failures | 1 |
| 10 Changing quality of services | 3 |
| 11 Sabotage and vandalism | 1 |
| 12 IP compromises | 1 |

The distribution is deliberately uneven. People errors and software attacks dominate because those are the threats that most plausibly affect a five-person professional services business working over the internet. Forces of nature and IP compromises are present and assessed, but a single well-chosen vulnerability reflects their real weight for this business better than padding them out.

Vulnerability descriptions are specific to our own network rather than generic. For example, the highest-ranked entry is:

> **T2V1A1** — *Phishing or malware on a staff workstation harvests stored client admin credentials, giving access to every client network*

This references the staff LAN and the workstation compromise scenario we discussed when justifying firewall rule 4 in `network.md` Section 4.4.

### 1.4 TVA Matrix

The TVAMatrix sheet is generated automatically from the Vulnerabilities sheet and shows which threats have been considered against which assets. We used it to check coverage before rating.

![TVA Matrix — assets 1 to 9](images/tvamatrix-1.png)

![TVA Matrix — assets 10 to 18](images/tvamatrix-2.png)

![TVA Matrix — assets 19 to 26](images/tvamatrix-3.png)

Every asset has at least one assessed vulnerability, and the data assets — the first six rows — carry the densest coverage, which reflects their importance to this business.

### 1.5 Results

Applying the risk matrix in Section 1.1 to our 35 entries produced:

| Risk | Count |
|---|---|
| Very High | 4 |
| High | 12 |
| Moderate | 15 |
| Low | 4 |

The four Very High risks are:

| Rank | TVA | Asset | Type | Vulnerability |
|---|---|---|---|---|
| **1** | T2V1A1 | Client administrative credentials | Data | Phishing or malware on a staff workstation harvests stored client admin credentials, giving access to every client network |
| 2 | T2V1A3 | Client remote access details | Data | Ransomware on a technician workstation encrypts or steals saved client remote access details |
| 3 | T2V1A9 | Staff and sysadmin workstations | Hardware | Phishing email or malware compromises a staff workstation, the most common entry point |
| 4 | T1V1A21 | Administrative staff member | People | Admin staff open a malicious invoice attachment; their workstation can reach the stored client credentials |

All four were rated High likelihood with Very High impact. They also describe the same underlying attack path: a staff workstation is compromised through phishing or malware, and from that foothold the attacker reaches credentials that unlock client networks. The assessment converges on one problem rather than four unrelated ones, which is what makes the control recommendations in Section 2 straightforward to prioritise.

---

## 2. Recommended Security Controls

### 2.1 The Selected Data Asset

Our risk assessment ranks **Asset 1 — Client administrative credentials** as the highest risk, at **Very High** (Likelihood: High, Impact: Very High). It is both the highest-ranked data asset and the highest-ranked risk overall.

**Why the impact is Very High.** These are the usernames, passwords and administrative accounts that Westline holds for its clients' servers, firewalls, and Microsoft 365 tenancies. Losing them does not expose one network — it exposes every client network the business administers. This is the supply chain attack pattern seen repeatedly against managed service providers, where compromising one provider gives access to dozens of downstream businesses. For Westline the consequences would include client data breaches, mandatory notification under Australian privacy law, immediate loss of client trust, and plausibly the end of the business.

**Why the likelihood is High.** The credentials are currently stored on staff workstations, which are the most exposed devices in the business: they read email, browse the web, and are operated by non-specialists. Phishing is the most common initial access method against small businesses, and every one of our four Very High risks runs through a compromised workstation.

The three controls below are selected from NIST SP 800-53 and address this asset at three different points: **at rest**, **in use**, and **in transit**.

---
### 2.2 Control 1 — Authenticator Management (NIST SP 800-53: IA-5)

**Implement a centralised, encrypted credential vault.**

#### How it reduces the risk

The highest-ranked vulnerability, T2V1A1, exists because credentials are stored on the workstations themselves — in browser password stores, spreadsheets, or documents. Malware that reaches a workstation can read all of them at once.

A credential vault changes what a workstation compromise yields. Credentials are held encrypted in a dedicated store and decrypted only in memory when actually used. Malware on the workstation no longer finds a readable file of every client's passwords; it finds an encrypted vault requiring a master secret it does not have.

It also addresses **T1V1A1** (rank 6) — technicians reusing, writing down or emailing passwords. A vault removes the reason to do any of those: credentials are retrievable on demand, so there is no incentive to keep a personal copy. Its generator also makes every client credential long, random and unique, so one compromised password cannot be reused elsewhere.

#### How it would be implemented here

- Deploy a team password manager (for example Bitwarden or 1Password for Business) with a vault shared across the five staff.
- Organise the vault into collections per client, so each technician sees only the clients they support. The administrative staff member gets no access to client credentials at all — consistent with the least privilege point in `network.md` Section 1.3.
- Migrate every existing credential into the vault, then **delete the originals** from workstations, spreadsheets and browser stores. Migration without deletion leaves the original exposure intact.
- Rotate every credential during migration, since any that were previously stored in plaintext must be assumed exposed.
- Enable audit logging so credential access is recorded — which also improves detection for `harden.md` Section 1.4's point about knowing what normal looks like.

#### Relationship to our network setup and hardening

This control reinforces the principle demonstrated in `harden.md` Section 1.2, where we found root passwords stored as MD5-crypt hashes. The lesson there was that **how a secret is stored determines how much protection it offers when the storage is stolen**. That finding applies directly here: client credentials in a plaintext file on a workstation have no protection at all, while credentials in an encrypted vault remain protected even after the file is exfiltrated.

The vault also complements firewall rule 4 (`network.md` Section 4.4). That rule prevents a compromised workstation from reaching the router's management interface; the vault prevents the same workstation from yielding the credentials for client systems. Together they contain a workstation compromise to that one machine.

#### Disadvantages

- **It creates a single high-value target.** Every credential now sits in one system. If the master password or vault account is compromised, the attacker gets everything at once. This makes protecting the vault itself critical — which is why Control 2 matters.
- **Cost.** A business plan is roughly AU$5–10 per user per month. For five staff that is a real, recurring expense for a small business.
- **Availability risk.** If the vault service is unreachable — during the NBN outage we assessed as T10V1A15 — staff cannot retrieve credentials and cannot support clients. Offline caching mitigates this but must be configured deliberately.
- **Adoption friction.** Technicians used to a saved browser password will find the vault slower. If it is inconvenient enough, staff will work around it by keeping local copies, which reintroduces exactly the vulnerability the control was meant to remove. Success depends as much on training and enforcement as on the software.

---
### 2.3 Control 2 — Multi-Factor Authentication (NIST SP 800-53: IA-2(1))

**Require a second authentication factor for the credential vault and for all administrative access to client systems.**

#### How it reduces the risk

Control 1 protects credentials while they are stored. Multi-factor authentication changes what a stolen credential is *worth*.

With single-factor authentication, a captured password is immediately usable. With MFA, the attacker also needs something the user physically has — an authenticator app code or a hardware token. A password harvested by malware, phished through a fake login page, or captured from the network is no longer sufficient on its own.

This directly reduces the likelihood of T2V1A1 being successfully exploited. It also mitigates **T2V2A1** (rank 5), where credentials could be intercepted in transit: an intercepted password without the second factor does not grant access.

Critically, MFA on the vault itself answers the single-point-of-failure objection raised against Control 1.

#### How it would be implemented here

- Enforce MFA on the password vault for all five staff, with no exemptions — an exempt account becomes the attacker's target.
- Enforce MFA on every client system that supports it: Microsoft 365 tenancies, remote access gateways, and client firewall administration interfaces.
- Issue **hardware security keys** (FIDO2, such as YubiKeys) to the owner and the systems administrator, whose accounts carry the widest access. Hardware keys resist phishing in a way that app-generated codes do not, because the key verifies the site's identity cryptographically and simply will not authenticate to a fraudulent domain.
- Use an authenticator app for the two technicians and the administrative staff member as a lower-cost option.
- Avoid SMS as a second factor. It is vulnerable to SIM-swap attacks and is no longer considered adequate for privileged access.
- Document break-glass recovery: recovery codes printed and stored in the office safe, so a lost token does not lock the business out of its own client systems.

#### Relationship to our network setup and hardening

This extends a principle we already applied. In `harden.md` Section 1.3 we replaced password authentication on the router with SSH key-based authentication, precisely because possession of a key is stronger than knowledge of a password, and because no reusable secret is stored on the server. MFA applies that same reasoning to the client systems and cloud services the router itself cannot protect.

It also compensates for a limitation we documented. In `network.md` Section 4.2 we noted that moving SSH to port 2222 is obscurity rather than real protection, and in `harden.md` Section 1.2 that OpenWRT neither stores passwords robustly nor enforces password strength. MFA reduces the consequences of exactly those weaknesses: even a weak or exposed password is not enough by itself.

#### Disadvantages

- **Friction on every login.** Five staff authenticating to multiple client systems daily will each lose seconds to minutes per login. For a business billing by the hour that is a measurable cost, and it is the most common reason MFA rollouts get quietly disabled.
- **Token loss and lockout.** A lost phone or key locks a technician out mid-support-call. Recovery processes must be in place and tested, and they themselves become a target for social engineering — an attacker calling the helpdesk claiming to have lost their token.
- **Not all client systems support it.** Older client hardware and legacy applications may offer no MFA option, so coverage will be incomplete and the business must track which systems remain single-factor.
- **Hardware cost.** FIDO2 keys are roughly AU$50–80 each, and spares are needed, so realistically four to six keys.
- **MFA fatigue attacks.** Where push-approval prompts are used, attackers repeatedly trigger prompts until a tired user approves one. Number matching or hardware keys avoid this, but it must be configured for rather than assumed.

---
### 2.4 Control 3 — Transmission Confidentiality and Integrity (NIST SP 800-53: SC-8)

**Require encrypted channels for every connection that carries client credentials or client data.**

#### How it reduces the risk

Controls 1 and 2 protect credentials at rest and at the point of use. This control protects them while they travel across a network.

Vulnerability **T2V2A1** (rank 5, High) records that credentials sent over unencrypted protocols can be intercepted by anyone on the network path. We did not assert this — we demonstrated it. In `harden.md` Section 2.1 we captured HTTP traffic to our own website and recovered the complete page content, including names and student IDs, with a single text search of the raw capture file. Anything transmitted over HTTP, including a password typed into a login form, is exposed the same way.

The counter-demonstration is in Section 2.2 of the same file: the identical test against an SSH session returned nothing. Encryption is what made the difference, using the same network and the same tools.

Encryption also provides **integrity**, which is easy to overlook. As noted in `harden.md` Section 2.1, an attacker on the path of an unencrypted connection can alter it in flight — injecting a fake login prompt to harvest credentials directly, for instance. An encrypted, authenticated channel makes tampering detectable.

#### How it would be implemented here

- **Migrate the business website to HTTPS.** Obtain a certificate from Let's Encrypt and configure uhttpd to serve TLS on port 443, redirecting port 80 to it. This replaces the plain-HTTP configuration documented in `network.md` Section 3.1.
- **Add a firewall rule for HTTPS.** Following the same pattern as our existing rules in `network.md` Section 4:
  ```sh
  uci set firewall.httpsrule=rule
  uci set firewall.httpsrule.name='Allow-HTTPS'
  uci set firewall.httpsrule.src='lan'
  uci set firewall.httpsrule.proto='tcp'
  uci set firewall.httpsrule.dest_port='443'
  uci set firewall.httpsrule.target='ACCEPT'
  uci commit firewall
  /etc/init.d/firewall restart
  ```
- **Restrict the LuCI management interface to HTTPS.** OpenWRT's uhttpd supports `listen_https`; port 81 currently serves plain HTTP, so administrative sessions to the router are presently unencrypted.
- **Mandate encrypted protocols for all client administration.** SSH rather than telnet, HTTPS rather than HTTP, and a VPN for any remote access to client networks. No client credential should ever be typed into an unencrypted session.
- **Restrict the SSH algorithms offered.** Our capture analysis in `harden.md` Section 2.2 showed dropbear still offering `hmac-sha1` and `diffie-hellman-group14-sha1`, both SHA-1 based and obsolete. Removing them from the offered set closes the possibility of a downgrade against an older client.

#### Relationship to our network setup and hardening

This control is the direct remedy for what our own packet captures revealed. The HTTP capture in `harden.md` Section 2.1 is the evidence that plain HTTP offers no confidentiality; the SSH capture in Section 2.2 is the evidence that encryption works. The recommendation follows from our own measurements rather than from general principle.

It also completes the hardening work. We secured *access* to the router in `harden.md` Sections 1.1 to 1.4 and controlled *which services are reachable* in `network.md` Section 4. Encrypting the channels themselves addresses the remaining gap: an attacker who cannot log in and cannot reach the management port may still be able to read traffic in transit.

#### Disadvantages

- **Certificate management overhead.** Certificates expire — Let's Encrypt certificates every 90 days. If renewal fails, the website breaks with a browser security warning, which for a firm selling security services is worse than the original problem. Automated renewal must be configured and monitored.
- **Internal certificates are awkward.** Public certificate authorities will not issue certificates for internal addresses such as 51.1.30.1, so the management interface needs either an internal certificate authority or self-signed certificates, both of which produce browser warnings that train staff to click through security prompts — a habit with its own risk.
- **Performance and hardware limits.** TLS adds computational overhead. A consumer-grade router handling encryption for the whole office may become a bottleneck, and the business may need better hardware.
- **Configuration is easy to get wrong.** HTTPS with an expired certificate, a weak cipher suite, or mixed HTTP content provides less protection than it appears to, while giving staff and clients false confidence.
- **It does not protect endpoints.** Encryption secures data in transit only. If a workstation is compromised, credentials are captured as they are typed, before encryption applies. This is precisely why the three controls are recommended together rather than individually.

---
