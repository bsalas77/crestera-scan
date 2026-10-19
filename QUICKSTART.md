# Crestera Scan quickstart

From download to first scan in under ten minutes. One file, no installer, no
other software: no Npcap, no nmap, no database server.

Only scan networks you are authorised to scan.

Download the binary for your system with `SHA256SUMS` and
`SHA256SUMS.ed25519` from the same release, and check them before running
anything: see [Verifying a release](SECURITY.md#verifying-a-release) (two
commands on Linux).

## 1. Start it

**Linux (recommended): Ubuntu, Debian, RHEL, AlmaLinux, Rocky or any other
systemd distribution, amd64 or arm64**

Install it as a service (use `crestera-scan-linux-arm64` on ARM):

```bash
chmod +x crestera-scan-linux-amd64
sudo ./crestera-scan-linux-amd64 service install
sudo cat /var/lib/crestera-scan/initial-admin-password.txt
```

That creates a `crestera-scan` system user, copies the binary to
`/usr/local/bin/crestera-scan`, and starts a hardened systemd service that
starts at boot. Settings are in `/etc/crestera-scan/crestera-scan.env`, data
in `/var/lib/crestera-scan`, the log in `journalctl -u crestera-scan`. More
under [Run as a Linux service](#run-as-a-linux-service). Skip to step 2.

To try it without installing anything, run it in a folder of its own:
`./crestera-scan-linux-amd64 serve` (it keeps a `data` folder beside itself,
as in the example output below).

**Windows:** a code-signed Windows build will follow.

The first run creates a `data` folder next to the binary and prints three
things worth reading:

```
FIRST RUN: created administrator "admin" with password ....................
certificate SHA-256 fingerprint e883b2e1...
listening on https://127.0.0.1:8443  data folder .../data
```

The password is also saved in `data/initial-admin-password.txt`.

Antivirus products that screen unfamiliar programs may hold the first launch
for 15 to 30 seconds. Wait for the `listening on` line.

## 2. Sign in

Open <https://127.0.0.1:8443> on the same machine. The browser will warn
about the certificate, because it is self-signed; compare its SHA-256
fingerprint with the one step 1 printed, then continue. Sign in as `admin`
with the printed password. (A server without a desktop: set
`CRESTERASCAN_HTTP_ADDR=0.0.0.0:8443` first, see
[Run as a Linux service](#run-as-a-linux-service), and open
`https://<server>:8443` from your computer.)

Click your username (bottom left) to change the password, then delete
`initial-admin-password.txt` (`/var/lib/crestera-scan/` for the service,
`data/` beside the binary otherwise). The same menu turns on two-factor sign-in:
scan the QR code with an authenticator app (Microsoft Authenticator, Google
Authenticator, 1Password and others), enter the code it shows, and keep the
ten recovery codes somewhere safe. To make it compulsory for everyone, set
**Settings > Two-factor sign-in** to *Required* once your own account has it.
If someone loses their phone, an administrator removes their second factor
from **Users > Edit**.

## 3. Define what may be scanned

**Scopes → New scope.** Give it a name and list the targets, one per line:

```
10.20.0.0/24
10.20.1.10-10.20.1.50
```

Leave the profile on **Safe**. Add anything that must never be touched under
**Exclusions** (a core banking host, a vendor appliance). A scope can cover at
most 1,048,576 addresses.

## 4. Scan

**Scans → New job**, choose the scope, keep `top100`, **Create**, then
**Start**. The run appears below and updates every few seconds. **Stop**
halts one run; **Stop all scans** halts everything immediately.

A fully populated /24 on Safe with `top100` takes about eight and a half
minutes; one with 30 live hosts takes about three and a half minutes, mostly spent
confirming that the empty addresses are empty.

For a short list of hosts you cannot afford to miss, such as a server VLAN,
set **Discovery** on the job to "treat every address as live": every
address is port-scanned whether or not it answers discovery.

## 5. Look at the results

**Hosts** and **Services** list what was found. Search with `field:value`
terms, combined with `AND` (or just a space), `OR`, `NOT` (or a leading `-`)
and parentheses. Numbers and dates compare with `<`, `<=`, `>`, `>=`.

| Search | Finds |
|---|---|
| `net:10.20.0.0/24 (port:3389 OR port:5900)` | remote desktops in that subnet |
| `product:openssh version:7.` | old OpenSSH |
| `cert.expires:<30d` | certificates expiring within 30 days |
| `tls.key_bits:<2048 OR tls.self_signed:true` | weak or self-signed certificates |
| `http.title:"sign in" -net:10.99.0.0/16` | login pages outside the lab range |
| `port:22 AND port:80` (Hosts) | hosts with both SSH and web |
| `net:10.20.0.0/24 NOT port:3389` (Hosts) | hosts there without RDP |
| `last_seen:<7d` | hosts not seen for over a week |

Fields: `ip`, `net`, `port`, `service`, `product`, `version`, `cpe`,
`banner`, `hostname`, `mac`, `scope`, `http.title`, `http.server`,
`http.status`, `tls.subject`, `tls.issuer`, `tls.san`, `tls.version`,
`tls.key_bits`, `tls.self_signed`, `cert.expires`, `ssh.fingerprint`,
`ssh.key_type`, `suspect`, `first_seen`, `last_seen`. For `cert.expires`,
`30d` means 30 days from now; for `first_seen` and `last_seen` it means 30
days ago. In **Hosts**, each port or service condition asks whether any of
the host's services matches; in **Services** it applies to that one row.
**Export CSV** and **Export JSON** download the current search.

Host names come from reverse DNS (the server's own resolver, only for
addresses in scope) and, on Standard scans, from a NetBIOS name query that
Windows answers with its computer name. Set `CRESTERASCAN_REVERSE_DNS=0` to
turn the DNS lookups off.

**CISA KEV.** Services whose product is in CISA's Known Exploited
Vulnerabilities catalog get a red **KEV** tag with the CVE ids; search with
`kev:true` or `kev.cve:CVE-2021-41773`. The match is by product: the
catalog lists no versions, so a match says this product has vulnerabilities
being exploited and the version still has to be checked (an up-to-date
Apache is flagged like an old one). A catalog snapshot is built in;
**Settings > CISA Known Exploited Vulnerabilities** downloads the latest
(once, or daily) from cisa.gov, or takes the file as an upload for servers
without internet access. New matches, from a scan or a catalog update, appear
under **Changes** as *CISA KEV match* and can be alerted on.

**Hosts without an agent.** If you run Crestera Watch, connect it under
**Settings > Crestera Watch**: its address, and a Watch *Viewer* account made
for Crestera Scan (with its two-factor key if it has one; for Watch's own
certificate, paste the CA that signed it). Every 15 minutes, and after each
scan, Hosts gets an **Agent** column: *agent*, *stale agent* (silent for a
week), *no agent*, or *unknown*. Watch agents report a name, not an address,
so hosts are matched on the first label of their name; a host Crestera Scan
has no name for stays *unknown* rather than being called unmanaged. Search
`agent:none` for live hosts with no agent. The first sync is the baseline;
after it, a host that appears without an agent, or loses its agent, is listed
under **Changes** as *No Watch agent*.

**Export for Pulse** (Hosts) downloads a zip with `assets.csv` and
`metrics.csv` in Crestera Pulse's import format: every host in the current
search as an asset, and counts of open services, expired and expiring
certificates, weak RSA keys, cleartext remote-access services, services
matching the KEV catalog and, with Watch connected, live hosts without an
agent. The `README.txt` inside says how to load them
alongside your own Pulse data.

## 6. Schedule scans and watch for changes

Give a job a **Schedule** to run it unattended: five-field cron in the
server's local time (`0 2 * * 1-5` is 02:00 on weekdays), or `@hourly`,
`@daily` (02:00), `@weekly`, `@monthly`. A scheduled run that would start
outside the scope's scan window is skipped and noted in the audit log.

From a scope's second finished run on, **Changes** lists what differs from
the run before: hosts appearing or gone, ports opened or closed, a service
or version changing, a TLS certificate replaced, and an SSH host key
changing (high severity: the classic sign of a replaced or impersonated
server). A port only counts as opened or closed if both runs scanned it,
and a host only counts as gone if the newer run finished.

## 7. Get alerted

Under **Alerts** (admins), add a **channel**, press **Send test**, then add
a **rule** that says which changes go to it:

- **E-mail**: any SMTP relay, over STARTTLS or implicit TLS (there is no
  plaintext mode). One message per run.
- **Syslog**: RFC 5424 to your SIEM over TLS (RFC 5425, port 6514), TCP or
  UDP. One message per change, with the details as structured data
  (`[crestera@32473 kind="port_opened" ip="10.20.0.15" port="3389" ...]`).
- **Webhook**: a signed JSON POST to an HTTPS endpoint. The signing secret is
  shown once, when the channel is created; see SECURITY.md for how a
  receiver checks the signature.

A rule can narrow by change kind, minimum severity, scope, and a search
(`port:3389`, `net:10.20.0.0/24 -product:openssh`). Undelivered alerts are
retried after 1, 5, 15 and 60 minutes, then marked failed with the error
shown in **Recent alerts**.

## 8. Scan other networks with a remote sensor

A sensor is this same program on a machine inside a network the server cannot
reach (a branch, a DMZ). It connects out to the server, asks for work, scans
and reports back; nothing ever connects to it, and it listens on no port.

1. On the server, turn on the sensor endpoint and restart:
   `CRESTERASCAN_SENSOR_ADDR=0.0.0.0:8444`. Sensors must be able to reach
   this port; the console can stay on 127.0.0.1.
2. **Sensors > New sensor**, give it a name. The console shows the command to
   run on the sensor machine, with a one-time token (valid 24 hours) and the
   fingerprint of the server's sensor CA:

   ```bash
   crestera-scan sensor enroll --server https://scan.example.org:8444 --token cst_... --ca-sha256 3f9a... --allow 10.20.0.0/16
   crestera-scan sensor run
   ```

   `--allow` lists the only ranges this sensor will ever scan. It is the
   sensor operator's limit, not the server's: a scope that asks the sensor
   for anything outside it fails on the sensor before a probe is sent.
3. In **Scopes**, set **Scanned by** to the sensor. Its jobs then wait for
   the sensor to collect them; results appear like any other, marked with
   the sensor.

The sensor trusts the server by the pinned CA, not by its name, so `--server`
can be any address that reaches it, through NAT or `HTTPS_PROXY`. The server
checks every address a sensor reports against the scope again and refuses
the whole report if one is outside it. A sensor that stops reporting for 10
minutes has its scan failed; **Revoke** shuts a sensor out at once. Its
certificate renews itself before it expires. Run it as a service as shown
under *Run as a service on Ubuntu*, with `ExecStart=... sensor run`.

## 9. Pro features

Pro features need a license. Paste the token under **Settings > Edition**,
or set `CRESTERASCAN_LICENSE`; the console says which edition is running
and until when. Everything in sections 1 to 8 works without one.

**NVD vulnerability matching.** Every product Crestera Scan identifies is
looked up in the NIST National Vulnerability Database once a day (new ones
after each scan), and each service's version is checked against the
vulnerable version ranges there. **Services** gains a *Vulnerabilities*
column: how many CVEs and the worst CVSS score. Search with `vuln:true`,
`vuln.cve:CVE-2021-41773` or `cvss:>=9`. A service whose version moves into
a vulnerable range, or which a newly published CVE covers, is listed under
**Changes** as *Vulnerable version (NVD)*, graded by score.

The KEV tag becomes version-aware too: *KEV n (this version)* when NVD puts
the version in the range of a known-exploited CVE, *KEV: not this version*
when it does not.

Matches are by the version a service announces. Linux distributions backport
fixes without changing that version (Ubuntu's OpenSSH 8.9p1 is patched where
upstream 8.9p1 is not), so a match on a distribution's package needs
checking against the package itself. NVD entries that name a product for
every version with no bound are not counted: they cannot be decided by
version. Lookups go to services.nvd.nist.gov, through `HTTPS_PROXY` if set;
an optional free NVD API key (**Settings > NVD**) makes them about ten times
faster.

**Findings.** **Findings** lists what to fix, worked out from every scan:
a known-exploited vulnerability in the version running (critical; on a
product match without NVD, high), a version in vulnerable ranges with a CVSS
score of 7 or more, an expired or expiring certificate, a weak RSA key, a
cleartext remote-login service, and a live host with no Crestera Watch agent.
A finding stays one finding across scans, resolves itself when the condition
is gone, and reopens if it comes back. Each is due a number of days after it
is first seen, by severity (critical 15, high 30, medium 90, low 180; change
them under **Settings**); the list shows what is overdue. Operators assign
findings, start work and comment; administrators accept a risk, with a
reason, until a date within a year (the finding reopens then if it is still
there), or mark a false positive. Every step is kept in the finding's
history, which the database refuses to alter.

**Examiner report.** **Reports** builds a page for an FFIEC, NCUA or GLBA
review, over the last 30 days to 12 months, for every scope or one: scan
coverage and cadence (a scope with no completed scan in the period is
called out), inventory, exposure, open critical findings, remediation
against due dates (share resolved on time, mean time to resolve), risk
acceptances with who approved them and until when, and the changes
detected. A table maps each section to the requirement it supports (for
GLBA, 16 CFR 314.4(c)(2), (d)(1) and (d)(2); for NCUA, 12 CFR 748.0 and
Part 748 Appendix A; for FFIEC, the IT Examination Handbook's Information
Security booklet). Print it or save it as PDF from the browser. It is
evidence for the examiner, not a finding of compliance. Set the
institution's name for the report once, in the same view.

**External attack surface.** List the institution's domains under
**External** (an administrator) and Crestera Scan reads, once a day, every
host name under them that public Certificate Transparency logs record with
an unexpired, publicly trusted certificate, including ones nobody remembers
(a test portal, a vendor-hosted service). Names stay listed once found. Each is
resolved with the server's DNS and checked against the scopes. The view
lists by default the names that point at a public address no scope covers,
which is exposure nobody is scanning; **Add to scope** puts an address into
a scope, and scanning it is then up to that scope's jobs. **Ignore** hides a
name you have dealt with (a third party's service, a parked domain). A name
that resolves to a private address is marked: internal addressing published
to the world. After the first look-up, each new name is a change (*New
external name*), medium when it is exposed, so alert rules can send it.

It is passive: the requests go to crt.sh and api.certspotter.com and name
only your domains, and nothing is sent to the hosts found. Names that never
had an HTTPS certificate do not appear; for those, scan your public ranges.

**Single sign-on (Entra ID).** Staff sign in with their Microsoft account,
and their role comes from Entra. In the Entra admin center:

1. **App registrations > New registration**, platform *Web*, redirect URI
   `https://<this server>/api/sso/callback` (the address people use).
2. **Certificates & secrets > New client secret**; copy its *value*.
3. **App roles**: create `Scan.Admin`, `Scan.Operator` and `Scan.Viewer`
   (allowed member types: users/groups), then assign them to users or groups
   under **Enterprise applications > (the app) > Users and groups**.

Then under **Settings > Single sign-on** enter the issuer
`https://login.microsoftonline.com/<tenant id>/v2.0`, the application
(client) ID, the secret and the same redirect URL, and save. The sign-in page
offers **Sign in with** your label. Any OpenID Connect provider works the
same way (Okta, Keycloak, AD FS); to map groups instead of app roles, set
the roles claim to `groups` and enter the group object IDs.

An account is created at its first sign-in, named by `preferred_username`
(the UPN), and its role is set from the provider's claims on every sign-in,
so removing a role in Entra takes effect at the next sign-in; disable the
account here to cut off a session already open. Someone with no matching
role is refused, unless you choose *Viewer* for anyone else. These accounts
have no password here, and two-factor sign-in is the provider's (conditional
access). A provider user can never take over a local account of the same
name: keep the local administrator as the way in if the provider is down.

## Profiles

| | Safe (default) | Standard |
|---|---|---|
| New connections per second | 50 | 500 |
| Connections per host at once | 2 | 10 |
| What is sent to an open port | nothing; it connects and listens | an HTTP `GET /`, a TLS handshake, an SSH key exchange |
| Identifies | services that announce themselves (SSH, FTP, SMTP, MySQL...) | also web servers, page titles, TLS certificates, SSH host keys |

On Safe, use `top100`. `top1000` on Safe is about 254,000 connections for a
full /24, which is well over an hour at 50 per second; use Standard for that.

Both profiles stop scanning a host that was answering and then goes quiet
(the circuit breaker), honour the scope's scan window, and stop the moment
you press Stop.

## Command line

One-off scan, no server, results as JSON:

```bash
crestera-scan scan --target 10.20.0.0/24 --profile safe --out results.json
crestera-scan scan --target 10.20.0.0/24 --exclude 10.20.0.5 --profile standard --ports 22,80,443,3389
crestera-scan scan --target 10.20.0.0/24 --remember results.json --out results2.json   # try ports seen last time first
crestera-scan scan --target 10.20.0.0/24 --no-rdns --out results.json               # no DNS name lookups
```

Account recovery, run on the server itself:

```bash
crestera-scan user list
crestera-scan user passwd admin      # reads the new password from standard input
crestera-scan user unlock admin
```

Remote sensor (section 8):

```bash
crestera-scan sensor enroll --server https://SERVER:8444 --token TOKEN --ca-sha256 HEX --allow 10.20.0.0/16
crestera-scan sensor run
crestera-scan sensor status
```

## Roles

| Role | Can |
|---|---|
| Viewer | see hosts, services, scopes and runs |
| Operator | also create and start jobs, stop scans, export |
| Admin | also manage scopes, exclusions and users, and read the audit log |

Five wrong passwords (or two-factor codes) lock an account for fifteen
minutes. Sign-ins, scope and
exclusion changes, job starts, stops and exports are written to an audit log
that the database itself refuses to alter.

## Configuration

Environment variables, or `KEY=value` lines in a settings file. None is
required. The settings file is the first of: the file named by
`CRESTERASCAN_ENV_FILE`; `crestera-scan.env` next to the binary;
`/etc/crestera-scan/crestera-scan.env` (written by `service install` on
Linux; a sensor's is `crestera-scan-sensor.env` there). Variables already set
in the environment win over the file.

| Variable | Default | |
|---|---|---|
| `CRESTERASCAN_HTTP_ADDR` | `127.0.0.1:8443` | Listen address. Set to `0.0.0.0:8443` to reach the console from other machines. |
| `CRESTERASCAN_DATA_DIR` | `data` beside the binary | Database, certificate, first-run password, and `secrets.key`, which encrypts stored secrets: back it up with the database (see SECURITY.md). |
| `CRESTERASCAN_TLS_CERT_FILE`, `CRESTERASCAN_TLS_KEY_FILE` | self-signed | Your own certificate. Set both. |
| `CRESTERASCAN_STORE` | `sqlite` | `postgres` to use Postgres. |
| `CRESTERASCAN_POSTGRES_DSN` | | `postgres://user:pass@host:5432/db` |
| `CRESTERASCAN_RAW` | off | `1` enables SYN scanning and ARP discovery for Standard scans. Linux only. |
| `CRESTERASCAN_REVERSE_DNS` | on | `0` stops looking up host names in DNS. |
| `CRESTERASCAN_SENSOR_ADDR` | off | Listen address for remote sensors, such as `0.0.0.0:8444` (mutual TLS). |
| `CRESTERASCAN_LICENSE` | | A Pro license token (or paste it under **Settings > Edition**). |

There is no plaintext HTTP mode.

## SYN and ARP (Linux only)

By default every scan is a TCP connect scan, on every platform. On Linux,
Standard scans can instead use half-open SYN probes and ARP discovery on
directly attached networks. It needs the `CAP_NET_RAW` capability. Installed
as a service, that is `sudo ./crestera-scan-linux-amd64 service install --raw`;
run by hand:

```bash
sudo setcap cap_net_raw+ep ./crestera-scan-linux-amd64
CRESTERASCAN_RAW=1 ./crestera-scan-linux-amd64 serve
# or one-off:
./crestera-scan-linux-amd64 scan --target 10.20.0.0/24 --profile standard --syn --arp
```

Without the capability the server logs the reason and uses connect scanning.
Safe scans are always connect-only. Windows builds never use raw sockets.

## Docker

```bash
docker build -t crestera-scan .
docker run -d --name crestera-scan -p 127.0.0.1:8443:8443 -v crestera-scan-data:/data crestera-scan
docker logs crestera-scan      # shows the first-run password
```

## Run as a Linux service

`sudo ./crestera-scan-linux-amd64 service install` sets up everything:

| | |
|---|---|
| Program | `/usr/local/bin/crestera-scan` |
| Runs as | the `crestera-scan` system user (no login shell), with no capabilities |
| Settings | `/etc/crestera-scan/crestera-scan.env` (written once, never overwritten) |
| Data | `/var/lib/crestera-scan` (mode 0700) |
| Log | `journalctl -u crestera-scan` |
| Unit | `/etc/systemd/system/crestera-scan.service` |

The unit is locked down: the system is read-only to the service apart from
its data folder, home folders and devices are hidden, and it gets no
capabilities unless you install it with `--raw` (`CAP_NET_RAW` only, for SYN
scans and ARP discovery). `systemd-analyze security crestera-scan` rates it
1.7, "OK".

Change a setting by editing the settings file, then
`sudo systemctl restart crestera-scan`. To open the console to other
machines, set `CRESTERASCAN_HTTP_ADDR=0.0.0.0:8443` there.

**Upgrade:** download the new binary and run `sudo ./crestera-scan-linux-amd64
service install` again. It replaces the program, keeps the data and
settings, keeps the `--raw` choice, and restarts the service.

**Moving a portable install into the service** (the `data` folder beside
the binary, from running `serve` by hand or from a release before 1.0.0):
stop the old process, then, in its folder:

```bash
sudo ./crestera-scan-linux-amd64 service install
sudo systemctl stop crestera-scan
sudo rm -rf /var/lib/crestera-scan/*
sudo cp -a data/. /var/lib/crestera-scan/
sudo chown -R crestera-scan:crestera-scan /var/lib/crestera-scan
sudo systemctl start crestera-scan
```

Accounts, scopes, results and the certificate come along; the database is
upgraded on start. Copy any settings from the old `crestera-scan.env` into
`/etc/crestera-scan/crestera-scan.env`, except `CRESTERASCAN_DATA_DIR`.

**Command-line tools** work on the service's data when run as its user, for
example to reset a password:

```bash
sudo -u crestera-scan crestera-scan user passwd admin
```

Run as root, they refuse: files root created in the data folder would lock
the service out of its own database.

**A remote sensor** on a Linux machine (step 8) installs the
same way, as its own service and user:

```bash
sudo ./crestera-scan-linux-amd64 service install --sensor
sudo -u crestera-scan-sensor crestera-scan sensor enroll --server https://SERVER:8444 --token TOKEN --ca-sha256 FINGERPRINT --allow 10.20.0.0/16
sudo systemctl start crestera-scan-sensor
```

`service status`, `service stop`, `service start` and `service uninstall`
(add `--sensor` for the sensor) do what they say. Uninstalling removes the
unit and the program and leaves the data, settings and user in place.

Without systemd, run `crestera-scan serve` under your own supervisor as an
unprivileged user, with `CRESTERASCAN_DATA_DIR` set.

## Security software on the scanning machine

Some antivirus products on Windows run a local proxy that answers connections
on a target's behalf. Avast's and AVG's Mail Shield do this for the mail ports
(25, 110, 143, 465, 587, 993, 995): every live host then appears to run a mail
server. Found on the first real-network test, where 70 of 80 "open" ports on a
home LAN came from the scanner's own antivirus.

Crestera Scan detects the pattern (mail ports open, and not one of them sends
the greeting a real SMTP, POP3 or IMAP server always sends) and marks those
ports **suspect**. The scan prints a warning, the run shows it, and the
Services view tags each one. They are kept, not deleted, so nothing real is
ever hidden.

For clean results, run the scanner on a machine without traffic-intercepting
security software (a Linux host or VM is best), or ask whoever manages the
antivirus to exclude the scanner from its mail and web shields.
