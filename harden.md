# Security Hardening and Traffic Analysis

COIT20246 Cyber Security and Networking — Term 2, 2026
SYD Group 6 — Md Arman Joarder (12312653), Atikur Rahman Mimmoy (12327451)

All work in this section was carried out on the OpenWRT VM provided in this unit (OpenWrt 22.03.3, r20028-43d71ad93e), the same VM documented in `network.md`.

---

## 1. Hardening the OpenWRT System

### 1.1 Change the Default Root Password



**Before — the default password is still in place.**

We logged in over SSH using the unit's default root password and viewed the stored hash:

```sh
cat /etc/shadow
```

![Password hash before the change](images/harden1-shadow-before.png)

The root entry was:

```
root:$1$3a5XsGay$jC88WNh...:19398:0:99999:7:::
```

The third field, `19398`, is the number of days since 1 January 1970 on which the password was last changed. That corresponds to **10 February 2023** — the date the VM image was built. The password had never been changed in the life of the image.

**The change.**

```sh
passwd
```

![Changing the root password](images/harden1-passwd.png)

**After — a new hash is stored.**

```sh
cat /etc/shadow | head -1
```

![Password hash after the change](images/harden1-shadow-after.png)

```
root:$1$.uLexRAq$hHOIu6d...:20723:0:99999:7:::
```

Three things changed and one did not:

| Field | Before | After |
|---|---|---|
| Algorithm prefix | `$1$` | `$1$` — **unchanged** |
| Salt | `3a5XsGay` | `.uLexRAq` |
| Hash | `jC88WNh...` | `hHOIu6d...` |
| Last changed (days since epoch) | 19398 — 10 Feb 2023 | 20723 — 27 Sep 2026 |

The salt and hash are both completely different, and the date field updated to the day we made the change. We discuss the algorithm, which did not change, in Section 1.2 below.

*Note: the password hashes shown in our screenshots are truncated. Publishing a complete hash in a repository would allow it to be attacked offline, which is the very risk this section is about.*

---

### 1.2 Examine How Passwords Are Stored

**What we found.** Passwords on OpenWRT are stored in `/etc/shadow`, one account per line, with fields separated by colons. The password field itself has three parts separated by `$`:

```
$1$wnfSwEHe$jqsqgLYsGmURGIMkAGzs0.
 │     │              │
 │     │              └─ the hash itself
 │     └──────────────── the salt
 └────────────────────── the algorithm identifier
```

**The hashing algorithm.** The prefix `$1$` identifies the algorithm:

| Prefix | Algorithm | Assessment |
|---|---|---|
| `$1$` | MD5-crypt | Weak — what this VM uses |
| `$5$` | SHA-256-crypt | Acceptable |
| `$6$` | SHA-512-crypt | Current common default on Linux |
| `$y$` | yescrypt | Modern, memory-hard |

Our VM uses **MD5-crypt**, and notably it still used MD5-crypt for the *new* password we set in Section 1.1. Changing the password improved the secret but did nothing to improve how that secret is stored.

**Why passwords are stored as hashes rather than plaintext.** A hash function is one-way: it is easy to compute the hash from a password, but computationally infeasible to recover the password from the hash. When a user logs in, the system hashes what they typed and compares it to the stored hash — the system never needs to know the actual password.

This matters because files get stolen. If an attacker obtains `/etc/shadow` — through a backup, a misconfigured file share, or a vulnerability that allows file reading — plaintext passwords would hand them every account immediately, including any passwords the staff had reused on other systems. With hashes, the attacker has to crack each one.

**What the salt does.** The salt, `wnfSwEHe` in our case, is a random value stored alongside the hash and mixed into the hashing. It means two accounts with the same password produce completely different hashes, so an attacker cannot tell which users share a password. More importantly, it defeats precomputed rainbow tables: an attacker cannot use a table of pre-hashed common passwords, because they would need a separate table for every possible salt.

**Why MD5-crypt is a weakness.** The problem with MD5 here is not that it is broken for collisions — that matters for signatures, not password storage. The problem is that it is **fast**. MD5-crypt was designed in the 1990s and performs a fixed 1000 iterations. Modern hardware can compute millions of MD5 hashes per second, so an attacker with a stolen `/etc/shadow` can test enormous numbers of candidate passwords very quickly. SHA-512-crypt is slower by design and supports a configurable iteration count, and modern algorithms such as yescrypt are also memory-hard, meaning they resist acceleration on GPUs.

The practical consequence for Westline IT Solutions is that the strength of the root password is doing all of the work. Because the stored hash provides little resistance to offline cracking, a short or predictable password would fall quickly if the file were ever obtained.

We therefore chose a long password for the root account, since password length is the main defence available to us against offline cracking.

**A related weakness.** The BusyBox `passwd` implementation used by OpenWRT only *warns* about a weak password rather than rejecting it, and the device offers no password policy configuration — no enforced minimum length, complexity requirement or dictionary check. Nothing on the router prevents an administrator from setting a very short root password.

Combined with the MD5-crypt storage described above, this means the system neither stores passwords in a form that resists cracking, nor prevents a weak one being chosen in the first place. For a business deployment, password strength for this device would have to be enforced by organisational policy rather than by the device. It also strengthens the case for the SSH key-based authentication we configure in Section 1.3, which removes password strength from the equation for remote access entirely.

**Accounts that cannot be logged into.** The remaining entries in `/etc/shadow` are service accounts, and none of them has a usable password:

```
daemon:*:0:0:99999:7:::
ftp:*:0:0:99999:7:::
network:*:0:0:99999:7:::
nobody:*:0:0:99999:7:::
ntp:x:0:0:99999:7:::
dnsmasq:x:0:0:99999:7:::
logd:x:0:0:99999:7:::
ubus:x:0:0:99999:7:::
```

A `*` or `x` in the password field is not a valid hash, so no input can ever match it and the account cannot be used to log in. These accounts exist so that services can run under their own restricted identity rather than as root, which limits the damage if one of those services is exploited. `root` is the only account on this system with a real password.

---

### 1.3 Set Up SSH Key-Based Authentication

**The risk this addresses.** With password authentication, anyone who can reach the SSH port can attempt to log in, and the only thing standing between them and root access is a secret that can be guessed. Automated brute-force tools try thousands of common passwords per minute against exposed SSH services. Passwords can also be reused across systems, shoulder-surfed, phished, or captured by a keylogger on the administrator's workstation.

Key-based authentication removes the guessable secret entirely.

**Generating the key pair.** We generated the key pair on the Windows host, not on the router, because the private key must stay on the client machine:

```
ssh-keygen -t ed25519 -C "syd-group-6"
```

![Generating the SSH key pair](images/harden3-keygen.png)

The key fingerprint is `SHA256:Bp9A21zUHRjRDgGwRBF3O9iuf9/TDmiL2siaakBCU7Q`.

We chose **ed25519** rather than the older RSA. Ed25519 is the current recommended algorithm for SSH keys: it provides security comparable to a 3072-bit RSA key in a 256-bit key, it is fast, and its implementation avoids several classes of mistake that have caused problems with RSA in the past. This is a deliberate contrast with the MD5-crypt password storage we found in Section 1.2 — where we could not choose the algorithm, here we could, and we chose a modern one.

Two files were created:

| File | Contents | Where it belongs |
|---|---|---|
| `id_ed25519` | **Private** key | Stays on the Windows workstation, never copied anywhere |
| `id_ed25519.pub` | **Public** key | Copied to the router |

**Installing the public key on OpenWRT.**

![The public key](images/harden3-pubkey.png)

OpenWRT uses dropbear as its SSH server, which reads authorised keys from `/etc/dropbear/authorized_keys`:

```sh
mkdir -p /etc/dropbear
echo 'ssh-ed25519 AAAAC3NzaC1lZDI1NTE5AAAAIFLihl2UfQC1lqQ3qetrUuHlREbc3Y3dQJVp2+cELX7h syd-group-6' >> /etc/dropbear/authorized_keys
chmod 600 /etc/dropbear/authorized_keys
/etc/init.d/dropbear restart
```

![Authorized keys file installed](images/harden3-authkeys.png)

```
-rw-------    1 root     root            93 Sep 27 13:28 /etc/dropbear/authorized_keys
```

The `chmod 600` is required, not cosmetic. Dropbear refuses to use an `authorized_keys` file that is writable by anyone other than its owner, because any user able to write to that file could append their own public key and grant themselves root access without needing a password at all.

**Demonstrating a successful key-based login.**

![Passwordless SSH login using the key](images/harden3-after.png)

The session opens directly at the OpenWrt banner with no password prompt, confirming that the router authenticated us using the key.

**Why key-based authentication is more secure than password-only authentication.**

The fundamental difference is that **no reusable secret is ever sent to the server, and no reusable secret is stored on it.**

With a password, the client sends the actual password to the server, which hashes it and compares. The server must therefore store something derived from the password (the `/etc/shadow` hash from Section 1.2), and that stored value is a target — as we showed, an MD5-crypt hash offers limited resistance to offline cracking.

With key-based authentication the exchange works differently. The server sends the client a random challenge. The client signs that challenge with its private key and returns the signature. The server verifies the signature using the public key it holds. The private key itself never travels across the network, and the server only ever holds the **public** key — which is useless to an attacker. Someone who steals `/etc/dropbear/authorized_keys` from the router gains nothing: they cannot derive the private key from the public key, and they cannot replay a previous signature, because each login uses a fresh random challenge.

The practical consequences are:

- **Brute force becomes infeasible.** Guessing a 256-bit ed25519 key is not comparable to guessing a password — there is no dictionary of likely keys, and the keyspace is far beyond exhaustive search.
- **Nothing worth stealing sits on the server.** Compromising the router does not yield credentials that work elsewhere.
- **Nothing is transmitted that could be captured.** Even if the connection were somehow downgraded or observed, no password crosses the wire.
- **Access is revocable per device.** Removing one line from `authorized_keys` revokes one workstation without affecting any other, and without changing a shared password that everyone then has to be told.

For Westline IT Solutions, where the systems administrator needs remote access to the router and the business holds client credentials, this is a substantially stronger position than password authentication alone.

**A trade-off we made.** We created the key without a passphrase so that the demonstration login is unattended. A passphrase encrypts the private key file on disk, so a stolen or compromised laptop would not immediately grant router access. In a real deployment for this business we would use a passphrase together with an SSH agent, which prompts once per session rather than on every connection, giving the protection without the friction.

**Further hardening we would recommend.** Password authentication is still enabled on the router alongside the key. The logical next step is to disable it, so that keys become the only accepted method:

```sh
uci set dropbear.@dropbear[0].PasswordAuth='off'
uci set dropbear.@dropbear[0].RootPasswordAuth='off'
uci commit dropbear
/etc/init.d/dropbear restart
```

We have documented rather than applied this change, because with only one authorised key installed, any problem with that key would leave no way in over the network. In a production deployment it would be applied once a second administrator key had been installed and tested.

---

### 1.4 Disable an Unnecessary Service

**The risk this addresses.** Every service running on a device is code that can contain vulnerabilities, and every service that listens on the network is a way in. A service that is not needed provides no benefit but carries the same risk as one that is. Reducing the number of running services is called reducing the attack surface, and it is one of the cheapest hardening measures available: there is no trade-off in functionality if the service genuinely is not used.

Unused services also create a monitoring problem. An administrator who does not know what is supposed to be running cannot easily recognise something that should not be.

**Identifying what is running.** We listed the services configured to start at boot:

```sh
ls /etc/rc.d/ | grep '^S'
```

```
S00sysfixtime   S12log      S19firewall   S50cron      S95done
S00urngd        S12rpcd     S20network    S50uhttpd    S96led
S10boot         S19dnsmasq  S35odhcpd     S80ucitrack  S98sysntpd
S10system       S19dropbear S94gpio_switch S99urandom_seed
S11sysctl
```

![Services configured to start at boot](images/harden4-services.png)

We reviewed these against what our network actually needs:

| Service | Needed? | Reason |
|---|---|---|
| `dropbear` | Yes | SSH administration (Section 1.3) |
| `uhttpd` | Yes | The business website and LuCI |
| `firewall` | Yes | All four rules in `network.md` |
| `network` | Yes | Core networking |
| `dnsmasq` | Yes | DNS and DHCP for the LAN |
| `log`, `sysntpd` | Yes | Logging, and accurate timestamps for those logs |
| `rpcd` | Yes | Required by the LuCI management interface |
| **`odhcpd`** | **No** | IPv6 DHCP and router advertisements — our network is IPv4-only |
| `gpio_switch`, `led` | No | Control physical hardware that does not exist on a VM |

**The service we disabled: `odhcpd`.**

We chose odhcpd because it is the clearest case of a service that is both unnecessary and network-facing. It is the IPv6 DHCP server and router advertisement daemon. Our lab network and our production design in `network.md` are IPv4 throughout — the production addressing uses 51.x and 53.x IPv4 subnets, and no part of the design uses IPv6. The service therefore provides nothing, while still running as root and processing network input.

The `gpio_switch` and `led` services are equally unnecessary on a virtual machine, since there is no physical hardware for them to control, but they do not accept network traffic and so removing them would not reduce exposure in the same way.

**Before — odhcpd is running and set to start at boot.**

```sh
ps w | grep odhcpd
ls /etc/rc.d/ | grep odhcpd
```

```
 1887 root      1092 S    /usr/sbin/odhcpd
K85odhcpd
S35odhcpd
```

![odhcpd running before](images/harden4-before.png)

**Disabling it.**

```sh
/etc/init.d/odhcpd stop
/etc/init.d/odhcpd disable
```

Both commands are needed and they do different things. `stop` terminates the process that is running now. `disable` removes the `S35odhcpd` startup symlink from `/etc/rc.d/`, so the service does not start again when the router reboots. Using `stop` alone would give the appearance of success until the next restart.

**After — the process is gone and it will not return.**

```sh
ps w | grep odhcpd
ls /etc/rc.d/ | grep odhcpd
```

![odhcpd disabled after](images/harden4-after.png)

The only remaining line in the `ps` output is the `grep` command itself, and `ls /etc/rc.d/` returns nothing for odhcpd — both the start and stop symlinks have been removed.

**Why disabling unnecessary services improves security.** The principle is that code which is not running cannot be exploited. A vulnerability discovered in odhcpd in the future would not affect this router, because the service is not there to attack. This also means one less component to patch and monitor.

For Westline IT Solutions there is a second, more specific benefit. Router advertisements are how devices on a network learn their IPv6 configuration, including the default gateway. On a network where IPv6 is not being managed or monitored, an attacker who gains a foothold can use rogue router advertisements to make themselves the default IPv6 gateway and intercept traffic, while the administrators continue watching IPv4 and see nothing wrong. Turning off IPv6 services that are not in use removes an entire parallel network stack that nobody at the business is paying attention to.

---

## 2. Capture and Analyse Network Traffic

We captured traffic on the OpenWRT router using `tcpdump` and analysed it on the Windows host using Wireshark. Two captures were taken over the host-only network between the Windows host (192.168.56.1) and the router (192.168.56.2): one of unencrypted HTTP traffic to our business website, and one of an encrypted SSH administration session.

Both capture files are included in this repository:

| File | Size | Packets | Contents |
|---|---|---|---|
| [`captures/http-capture.pcap`](captures/http-capture.pcap) | 12,612 bytes | 40 | HTTP traffic to the test website |
| [`captures/ssh-capture.pcap`](captures/ssh-capture.pcap) | 15,453 bytes | 100 | An SSH session on port 2222 |

---

### 2.1 Capture 1 — HTTP Traffic

#### Taking the capture

We ran tcpdump on the router, filtering for TCP port 80 — the port serving our website, as identified in `network.md` Section 3.1:

```sh
tcpdump -i br-mng -s 0 -w /tmp/http-capture.pcap 'tcp port 80'
```

The `-i br-mng` selects the interface facing the Windows host. The `-s 0` sets an unlimited snapshot length so that full packets are captured rather than just their headers — without this the HTML payload would be truncated and the analysis below would not be possible.

While the capture was running we loaded `http://192.168.56.2/?v=1` from Chrome on the Windows host.

![tcpdump HTTP capture](images/capture1-tcpdump.png)

```
40 packets captured
40 packets received by filter
0 packets dropped by kernel
```

**A problem we had to solve.** Our first attempt produced a 5,475-byte file that contained no page content at all. The browser had the page cached, so it sent a conditional request and the server replied `304 Not Modified` with an empty body — the headers were captured but the HTML never crossed the network. We solved this by requesting a URL the browser had never seen, `?v=1`, which forced a complete fetch. The file grew to 12,612 bytes and the page content appeared.

We verified the payload was present before moving to Wireshark, by searching the capture file directly on the router:

```sh
grep -a "12312653" /tmp/http-capture.pcap
```

```
<tr><th>Student 1</th><td>Md Arman Joarder &mdash; 12312653</td></tr>
```

A student ID typed into an HTML file was recoverable from the raw capture with a single text search, with no analysis tool involved at all.

#### Analysis in Wireshark

We transferred the capture to the Windows host and opened it in Wireshark with the display filter `http`.

![Transferring the captures](images/capture-transfer.png)

![HTTP packet list in Wireshark](images/capture1-wireshark-list.png)

Twelve of the forty packets are HTTP; the remainder are TCP handshake and acknowledgement packets. The exchange is:

| Packet | Direction | Content |
|---|---|---|
| 4 | .1 → .2 | `GET /?v=1 HTTP/1.1` |
| 14 | .2 → .1 | `HTTP/1.1 200 OK (text/html)` — the full page |
| 16 / 19 | both | `GET /favicon.ico` → `404 Not Found` |
| 21–39 | both | Later `GET`s → `304 Not Modified` |

The `304 Not Modified` responses are the caching behaviour that defeated our first attempt, now visible in the data.

#### The request

![HTTP request in Wireshark](images/capture1-wireshark-request.png)

Expanding packet 4 shows the full protocol stack, and every layer leaks something:

| Layer | Field | Value |
|---|---|---|
| Ethernet | Source / destination MAC | `0a:00:27:00:00:13` → `08:00:27:e4:b4:9d` |
| IPv4 | Source / destination IP | 192.168.56.1 → 192.168.56.2 |
| TCP | Source / destination port | 9406 → 80 |
| HTTP | Request URI | `http://192.168.56.2/?v=1` |
| HTTP | Host header | `192.168.56.2` |
| HTTP | User-Agent | `Mozilla/5.0 (Windows NT 10.0; Win64; x64) ... Chrome/153.0.0.0` |

The destination MAC `08:00:27:e4:b4:9d` is the same address we identified as `br-mng` in `network.md` Section 2.2, confirming the traffic is reaching the router over the host-only bridge.

#### The response

![HTTP response in Wireshark](images/capture1-wireshark-response.png)

Packet 14 carries the reply. Wireshark reports `[5 Reassembled TCP Segments (5723 bytes)]` — the page was larger than one packet, so it was split across five TCP segments which Wireshark reassembled. The headers show:

```
HTTP/1.1 200 OK
Content-Type: text/html
Content-Length: 5499
ETag: "11d-157b-6ab66263"
Last-Modified: Fri, 25 Sep 2026 12:00:35 GMT
Date: Sun, 27 Sep 2026 13:49:46 GMT
```

Even in the raw hex pane, the ASCII column is directly readable — `span>Router, firewall and secure Wi-Fi installation and configuration.</span>` — with no decoding required.

#### Following the stream

Right-clicking the packet and selecting **Follow → HTTP Stream** reassembles the entire conversation: 6 client packets, 6 server packets, 9,651 bytes.

![Following the HTTP stream](images/capture1-followstream.png)

The complete page is readable, including the personalised project details:

```html
<div class="card project-card">
  <h2>COIT20246 Project &mdash; Test Website</h2>
  <table>
    <tr><th>Student 1</th><td>Md Arman Joarder &mdash; 12312653</td></tr>
    <tr><th>Student 2</th><td>Atikur Rahman Mimmoy &mdash; 12327451</td></tr>
    <tr><th>Group</th><td>SYD Group 6</td></tr>
    <tr><th>Campus</th><td>Sydney (SYD)</td></tr>
    <tr><th>Date created</th><td>25 September 2026</td></tr>
  </table>
</div>
```

The business contact details are equally exposed:

```html
<tr><th>Address</th><td>Level 2, 15 Macquarie Street, Parramatta NSW 2150</td></tr>
<tr><th>Phone</th><td>(02) 9000 1234</td></tr>
<tr><th>Email</th><td>support@westlineit.example.com</td></tr>
```

#### Security implications — what could an attacker learn?

Anyone positioned to observe this traffic obtains the following without any cracking, guessing or specialist equipment:

**The complete content of every page viewed.** Our page is public information, so the content itself is not confidential. But the principle generalises: HTTP provides no confidentiality whatsoever, so whatever a user views is visible to anyone on the path. If Westline IT Solutions later added a client portal over HTTP, every client record displayed would be exposed the same way.

**Credentials, if any were used.** Our site has no login, but had it, the username and password would appear in the capture exactly as the HTML did. The same applies to session cookies — an attacker who captures a session cookie can impersonate that user without ever knowing their password.

**Who is talking to whom, and about what.** The IP and MAC addresses identify both machines, the URI shows precisely which pages were requested, and the timing shows when.

**What software the client is running.** The User-Agent header volunteers Windows 10, 64-bit, Chrome 153. An attacker can look up known vulnerabilities for that exact browser version and select a matching exploit, rather than guessing.

**The ability to modify traffic, not just read it.** This is the consequence that is easy to overlook. HTTP provides no integrity protection, so an attacker positioned between the client and server can alter the response in transit — injecting a fake login form, adding malicious JavaScript, or changing the displayed bank details on an invoice. The browser has no way to detect the change, because there is nothing to verify against.

For a business that sells cyber security services, hosting its public site over plain HTTP is also a credibility problem in itself: modern browsers display a "Not secure" warning in the address bar, which is visible in our own screenshots.

---

### 2.2 Capture 2 — SSH Traffic

#### Taking the capture

We captured an SSH administration session, filtering on port 2222 — the non-standard port we moved SSH to in `network.md` Section 4.2:

```sh
tcpdump -i br-mng -s 0 -w /tmp/ssh-capture.pcap 'tcp port 2222'
```

We ran this from the VirtualBox console rather than over SSH, so that the capture would record only the session being studied and not the session doing the capturing. While it ran, we connected from the Windows host and executed `uname -a`, `ls /etc` and `cat /etc/openwrt_release`.

![tcpdump SSH capture](images/capture2-tcpdump.png)

```
100 packets captured
101 packets received by filter
0 packets dropped by kernel
```

We then applied exactly the same test we used on the HTTP capture, searching for text we knew had crossed the connection:

```sh
grep -a "openwrt_release" /tmp/ssh-capture.pcap
grep -a "DISTRIB_RELEASE" /tmp/ssh-capture.pcap
```

Both searches returned nothing. We had typed `cat /etc/openwrt_release` and watched `DISTRIB_RELEASE='22.03.3'` print on screen, yet neither string appears anywhere in 15,453 bytes of captured traffic.

#### Analysis in Wireshark

![SSH packet list in Wireshark](images/capture2-wireshark-list.png)

Wireshark identified the traffic as **SSHv2** despite the non-standard port, using protocol heuristics rather than the port number. The session divides cleanly into two phases:

| Packets | Phase | Readable? |
|---|---|---|
| 5–7 | TCP three-way handshake | Headers only |
| 8–9 | Version banner exchange | **Yes — plaintext** |
| 11–15 | Key Exchange Init, Elliptic Curve Diffie-Hellman | **Yes — plaintext** |
| 16 | New Keys | Changeover point |
| 18 onward | `Encrypted packet (len=…)` | **No** |

Note that Wireshark labels 192.168.56.2 as "Client" and 192.168.56.1 as "Server", which is the reverse of reality — 192.168.56.2 is the router running the SSH server. We refer to them by address to avoid repeating that mistake.

#### What is visible before encryption begins

Reading the capture file directly shows exactly what crosses the wire in the clear:

```
SSH-2.0-dropbear
SSH-2.0-OpenSSH_for_Windows_9.5
```

Each side announces its software and version. The algorithm negotiation is also plaintext — the router offers:

```
kex:     curve25519-sha256, diffie-hellman-group14-sha256, diffie-hellman-group14-sha1
hostkey: ssh-ed25519, rsa-sha2-256, ssh-rsa
cipher:  chacha20-poly1305@openssh.com, aes128-ctr, aes256-ctr
mac:     hmac-sha1, hmac-sha2-256
```

The host key type in use is `ssh-ed25519`, matching the key algorithm we chose in Section 1.3.

#### What is not visible

![Following the SSH stream](images/capture2-followstream.png)

Following the TCP stream shows the two version banners as readable text, and then nothing but binary data for the remainder of the session. The commands we typed, their output, the directory listing of `/etc`, and the authentication exchange itself are all unrecoverable.

This is the direct counterpart to the HTTP stream in Section 2.1: the same tool, the same view, the same network — and no readable content.

#### Comparison and why encryption matters

The two captures were taken minutes apart, on the same interface, between the same two machines, using the same tools. The only difference is the protocol.

| | HTTP (port 80) | SSH (port 2222) |
|---|---|---|
| `grep` for known content | Returned the full HTML | Returned nothing |
| Follow Stream | Complete page, readable | Two banners, then binary |
| Content confidentiality | None | Protected |
| Content integrity | None — modifiable in transit | Protected — tampering detected |
| Credentials | Would be readable | Never transmitted (key-based) |
| Software versions disclosed | Yes, via User-Agent | Yes, via version banner |
| Endpoints, ports, timing | Visible | Visible |

**Encryption protects content, not metadata.** The SSH capture still reveals a great deal. An observer can see that 192.168.56.1 connected to 192.168.56.2 on port 2222, at what time, for how long, and how many bytes flowed in each direction. Packet sizes and timing can leak more than is obvious: interactive SSH sends a packet per keystroke, so the pattern of small packets reveals typing rhythm and command lengths even though the characters themselves are protected.

**The negotiation phase is a real disclosure.** An attacker learns the router runs dropbear and the client runs OpenSSH 9.5 on Windows, then searches for known vulnerabilities in those exact versions. They also learn which algorithms are supported — and here that reveals something worth acting on. The router still offers `hmac-sha1` and `diffie-hellman-group14-sha1`, both of which rely on SHA-1, an algorithm now considered obsolete. Modern clients will negotiate something stronger, but continuing to offer legacy options creates the possibility of a downgrade attack against an older or manipulated client. For Westline IT Solutions we would recommend restricting the offered algorithms to the modern set.

**Why this matters for the business.** Westline's systems administrator uses SSH to manage the router, and the traffic crosses the same office network as everything else. If that administration were done over an unencrypted protocol such as telnet, an attacker with a foothold on any staff workstation could read the entire session — including the credentials used to authenticate — and gain full control of the router. Because SSH is used instead, and because we configured key-based authentication in Section 1.3, there is no password in the traffic to capture and no content to read.

The practical recommendation that follows from these two captures is straightforward: the business website should be served over HTTPS rather than HTTP. The mechanism is the same one demonstrated by the SSH capture — encrypt the channel so that content is confidential and tamper-evident, rather than relying on the network itself being trustworthy.

---

## 3. References

OpenWrt Project. *Dropbear SSH Server Configuration*. https://openwrt.org/docs/guide-user/base-system/dropbear

OpenWrt Project. *OpenWrt Firewall Configuration*. https://openwrt.org/docs/guide-user/firewall/firewall_configuration

The Tcpdump Group. *tcpdump(8) man page*. https://www.tcpdump.org/manpages/tcpdump.1.html

Wireshark Foundation. *Wireshark User's Guide*. https://www.wireshark.org/docs/wsug_html_chunked/

Bernstein, D. J., Duif, N., Lange, T., Schwabe, P., and Yang, B.-Y. *High-speed high-security signatures* (Ed25519). https://ed25519.cr.yp.to/

COIT20246 Cyber Security and Networking, Term 2 2026, unit lecture material and lab practicals, CQUniversity.

*Generative AI (Claude) was used to help improve the wording of explanations in this report and to check command syntax. All configuration, testing, captures and screenshots are our own work, produced on the OpenWRT VM provided in this unit.*
