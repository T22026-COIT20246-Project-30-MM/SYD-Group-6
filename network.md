# Network Setup

## Assumptions

Write your answer here.

## OpenWRT and VirtualBox Setup

Write your answer here. Ensure you embed images of the network design, and link to the drawio files. The drawio files must be in your repository. 

## Firewall Configuration

Write your answere here.

## Production Network Design

Write your answer here.

# Network Setup

COIT20246 Cyber Security and Networking — Term 2, 2026
SYD Group 6 — Atikur Rahman Mimmoy (12327451), Md Arman Joarder (12312653)

## 1. Assumptions

The scenario does not state every detail about the business, so we have made the following assumptions. Our network design, firewall rules and risk assessment are all based on them.

### 1.1 Location

The business is located in Sydney, New South Wales, in a leased office on Level 2 of a small commercial building in Parramatta.

We assume a single office in one location. This means the business needs only one local network, one internet connection and one router/firewall, with no links to branch offices. All staff work from this office, so we do not design for remote access or VPN connections.

### 1.2 Type of Professional Services

The business is an accounting and taxation practice. It prepares individual and small business tax returns, lodges BAS statements, provides bookkeeping and payroll services, and gives financial planning advice to local clients.

This assumption matters for security because the business stores tax file numbers, bank account details, payroll records and financial statements belonging to its clients. This data is attractive to attackers because it can be used for identity theft and fraud, and the business is legally required to protect it under Australian privacy law. For this reason confidentiality is the highest priority in our risk assessment.

### 1.3 Number of Staff and Their Roles

The business has 5 staff:

| Role | Number | Network access needed |
|---|---|---|
| Owner / principal accountant | 1 | Full access to client records and financial systems |
| Accountants | 2 | Access to client records for their own clients |
| Administrative staff | 1 | Appointments, invoicing and general correspondence |
| IT support contractor (part-time, on site weekly) | 1 | Router and server administration |

This gives 5 Windows workstations on the internal network, plus a shared network printer. Only the IT support contractor needs to reach the router management interface, which is why we restrict management access rather than allowing it from every workstation on the network.

### 1.4 Website Content

The website is a public information site only. It shows the business name, a description of the services offered, the office address and opening hours, and a contact phone number and email address.

The website does not have client logins, file uploads, online payments, or a database, and it does not collect or store any personal information. We assume clients send documents by other means and do not upload them through the site.

This assumption keeps the web server simple, since no database or user authentication is required. The website still needs protection: if it were defaced or taken offline, the business would lose credibility with existing clients and would appear untrustworthy to potential new clients, which is a reputational risk rather than a data breach risk.

