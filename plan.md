# Project Plan

COIT20246 Cyber Security and Networking — Term 2, 2026
SYD Group 6 — Atikur Rahman Mimmoy (12327451), Md Arman Joarder (12312653)

## Communication Plan

We communicate face-to-face at the SYD campus. We book a group study room in the library so we can run the OpenWRT VM together and look at each other's work on screen. One of us books the room at the start of each week and tells the other. If no room is free, we meet in the open study area on campus. Between meetings we use our phone group chat for short messages and reply the same day.

**Frequency.** Twice a week, on the two days we are both on campus for class:

- **Thursday, after class** — about two hours. We do the practical work for that week.
- **Friday, after class** — about two hours. We write up the week's section, check the screenshots and commit our work before we leave.

Because our classes are on Thursday and Friday, we are both already on campus on those days, so we book the study room straight after class and neither of us has to make an extra trip. We also work together in the tutorial and ask the tutor about anything we could not solve.



## Schedule

| Week | Task | Who | Committed by end of week |
|---|---|---|---|
| 5 | Set up the repository, read the specification, agree the plan, import the OpenWRT VM | Both | `README.md`, `plan.md` |
| 6 | Write the assumptions; record the OpenWRT interfaces, IP addresses and VirtualBox adapter types | Arman (assumptions), Atikur (network) | `network.md` — assumptions and setup |
| 7 | Draw the lab network diagram in draw.io; set up the test web server | Atikur (diagram), Arman (web server) | `network.md` — diagram and web server |
| 8 | Configure and test all four firewall rules, with before and after screenshots | Arman (HTTP, ICMP), Atikur (SSH, management) | `network.md` — firewall rules |
| 9 | Draw the production network diagram; complete the four hardening steps | Arman (diagram, SSH keys, services), Atikur (password, `/etc/shadow`) | `network.md` — production design, `harden.md` |
| 10 | Capture and analyse the HTTP and SSH traffic; complete the risk assessment and security controls | Arman (HTTP, controls), Atikur (SSH, risk assessment) | `harden.md`, both `.pcap` files, `security.md`, `risk-assessment.xlsx` |
| 11 | Record the video, write the reflection, check everything against the specification, submit on Moodle and Echo360 | Both | `reflection.md` and submission |
