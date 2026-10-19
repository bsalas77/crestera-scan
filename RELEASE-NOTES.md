# Release notes

## v1.0.1: current libraries

A maintenance release. Scan is built with the latest Go libraries for
cryptography and the operating system (`golang.org/x/crypto` 0.58,
`x/sys` 0.49, `x/sync` 0.24, `x/text` 0.43). No advisories lie behind these
updates; nothing else changes. Upgrade by running
`sudo ./crestera-scan-linux-amd64 service install` with the new binary.

## v1.0.0: production release, Linux first

The first production release of the Free edition. Everything in v0.1.0 to
v0.3.0 (scanning, search, scheduled scans, change detection, alerts, remote
sensors, CISA KEV, Crestera Watch inventory, Pulse export, two-factor
sign-in) with these additions:

**Linux service.** `sudo ./crestera-scan-linux-amd64 service install` sets
up a hardened systemd service: its own system user with no login, the
program in `/usr/local/bin`, settings in `/etc/crestera-scan`, data in
`/var/lib/crestera-scan`, a read-only view of the rest of the system, and no
capabilities (`--raw` grants `CAP_NET_RAW` alone, for SYN scans and ARP).
`systemd-analyze security` rates it 1.7, "OK". Run it again to upgrade.
`--sensor` installs a remote sensor the same way. Checked on Ubuntu 26.04
and AlmaLinux 10.2.

**The Crestera suite's look.** The console now uses the shell shared by
Crestera GRC, ITSM, Vuln and Control: a navy sidebar with grouped sections,
a white top bar, light pages and white cards, and a navy sign-in page.
Scan's accent is cyan. Pro views carry a Pro badge in the Free edition. Your
name, for the password and two-factor settings, is now at the bottom left.

**Verifying downloads with OpenSSL.** Releases add `SHA256SUMS.ed25519`, and
SECURITY.md gives the signing key as PEM: `openssl pkeyutl -verify` and
`sha256sum -c` check a download on any current Linux.

**License.** The Free edition now ships with its license (LICENSE): free for
your organization's internal use, including commercial use; not for resale
or hosting for others; authorized scanning only.

**Upgrading from v0.3.0.** Run the new binary on the old data folder; the
database is upgraded on start. To move a Linux install into the service,
see "Moving a portable install into the service" in QUICKSTART.md.

**Pro features in the same binary.** NVD version matching, findings with
owners and deadlines, FFIEC/NCUA/GLBA examiner reports, Entra ID single
sign-on and external attack surface discovery are part of the program and
stay locked without a license key: their views explain what they do, and
the Free features are unaffected.

**Fixed.** The Windows service commands in QUICKSTART.md were garbled in
v0.3.0 (`.estera-scan-windows-amd64.exe`); they read
`.\crestera-scan-windows-amd64.exe service install` and so on.

## v0.3.0 (M3): remote sensors, KEV, Watch, Pulse, two-factor

**Remote sensors.** `crestera-scan sensor enroll|run` puts the scanner on a
machine inside a network the server cannot reach. The sensor only connects
out, to a separate endpoint (`CRESTERASCAN_SENSOR_ADDR`, off by default), and
listens on no port. It enrolls once with a 24-hour token from **Sensors >
New sensor**, gets a client certificate from the server's own sensor CA,
which it pins by fingerprint, and from then on every request is mutual TLS.
Each sensor keeps its own `--allow` ranges: a scope outside them fails on the
sensor with nothing sent. The server checks every address a sensor reports
against the scope again. Scopes get **Scanned by**; revoking a sensor shuts
it out at once; a sensor silent for 10 minutes has its scan failed.

**CISA KEV.** Services whose product is in the Known Exploited
Vulnerabilities catalog get a KEV tag with the CVE ids, are searchable
(`kev:true`, `kev.cve:CVE-2021-41773`) and are counted per host. The catalog
lists no versions, so this is a product match and says so: the version still
has to be checked. A snapshot is built in; **Settings** downloads the latest
from cisa.gov (once or daily, `HTTPS_PROXY` honoured) or takes the file as an
upload. New matches, from a scan or a catalog update, are changes
(*CISA KEV match*, medium) that alert rules receive.

**Crestera Watch.** Connect Watch with a Viewer account and Hosts shows which
live hosts have an agent, a stale one, none, or are unknown (no name to match
on). Search `agent:none`. A host that appears without an agent, or loses its
agent, is a change (*No Watch agent*); the first sync is the baseline.

**Host names.** Reverse DNS (the server's resolver, only for addresses in
scope) and, on Standard scans, a NetBIOS name query. `CRESTERASCAN_REVERSE_DNS=0`
or `--no-rdns` turns DNS lookups off.

**Crestera Pulse export.** **Export for Pulse** on Hosts: `assets.csv` and
`metrics.csv` in Pulse's own import format.

**Two-factor sign-in.** TOTP per account (QR setup, ten one-time recovery
codes, each code usable once), which an administrator can require for
everyone. Admins can remove a user's second factor for a lost phone.

**Also**
- Secrets the server reads back (TOTP seeds, alert passwords and signing
  keys, the Watch password) are encrypted with `data/secrets.key`. Back it
  up, and `sensor-ca.key`, with the database (SECURITY.md).

**Upgrading from v0.2.0**: replace the binary and restart. The database is
migrated (schema versions 7 to 10), and alert secrets saved by v0.2.0 in
plain text are encrypted on first start.

**Verified**
- `test/lab/sensor/run.sh`: the real binary as server and sensor in Docker,
  the server with no route to the site network. The sensor listens on no
  port; a reused token is refused; results match each target's socket table
  exactly; a scope outside the sensor's allowed ranges is refused with 0
  probes; a console stop reached the sensor in 2 seconds; a revoked sensor's
  next request was refused.
- KEV: the variety lab's httpd 2.4.49 is flagged with CVE-2021-41773; a
  download from cisa.gov through the server's own client; an uploaded newer
  catalog produced exactly one new match on a live server.
- Watch: a real Crestera Watch server over TLS (CA pinned) and a real
  enrolled agent: its host matched, an unnamed host stayed unknown, the
  first sync raised nothing, and a new unmanaged host raised its change.
- Pulse: Pulse's own `import-csv.js` loaded the export into its Postgres.
- Two-factor: zbarimg decodes the setup QR to the shown key; codes from
  oathtool are accepted at setup and at sign-in.
- Host names from Docker's resolver for 17 lab containers, and over NetBIOS
  from Samba's nmbd and from a real Windows 11 machine (its computer name).
- A sensor on a separate real Linux host (WSL), enrolled with a server in
  Docker across NAT, scanned the VirtualBox host-only network (Windows
  Server 2022, Ubuntu, AlmaLinux) and reported every host.
- Regression: the M1 /24 lab (Safe, Standard, SYN, canaries), the variety lab
  (nothing reported that is not listening) and the M2 lab (now 8 changes,
  the new host also matching KEV) all pass.
- Upgrade from v0.2.0: scopes, jobs, runs, services and alert rules
  survive; the v0.2.0 webhook secret is encrypted on first start; the first
  v0.3.0 run diffs against the v0.2.0 run.

## v0.2.0 (M2): schedules, changes and alerts

**Search language**
- AND (or a space), OR, NOT (or a leading `-`) and parentheses; numbers and
  dates compare (`port:<1024`, `tls.key_bits:<2048`, `cert.expires:<30d`,
  `last_seen:<7d`); 15 more fields (banner, hostname, mac, scope,
  http.status, tls.issuer, tls.san, tls.version, tls.key_bits,
  tls.self_signed, ssh.key_type, suspect, first_seen, last_seen,
  service.seen).
- In Hosts, each port or service condition asks whether any of the host's
  services matches: `port:22 AND port:80` is a host with both, `NOT port:23`
  a host without Telnet.
- Still parameterized SQL only, held by a fuzz test of about 3,000 random and
  hostile queries.

**Schedules**: five-field cron or `@hourly` / `@daily` / `@weekly` /
`@monthly` on any job. A scheduled minute fires once, even across a
restart; a firing outside the scope's scan window is skipped and audited.
Runs missed while the server was down are not caught up.

**Changes**: each finished run is compared with the previous one of the
same scope. Host new / gone, port opened / closed, service or version
changed, certificate replaced, SSH host key changed (high severity). A port
only opens or closes if both runs scanned it; a host is only gone if the
newer run finished; antivirus artefacts are ignored. New Changes view and
`GET /api/changes`.

**Alerts**: channels for e-mail (STARTTLS or implicit TLS only), syslog
(RFC 5424 over UDP, TCP or TLS with an optional pinned CA) and webhooks
(signed with HMAC-SHA256; see SECURITY.md). Rules by change kind, minimum
severity, scope and search. Retries after 1, 5, 15 and 60 minutes, then
failed with the error shown. Send test button. Secrets are write-only.

**Also**: changes are saved before a run is marked finished; scanned text
is stripped of control characters before it reaches an alert; the server
binds its port before saying it is listening.

**Upgrading from v0.1.0**: replace the binary and restart. The database is
migrated automatically (schema versions 2 to 6).

**Verified**
- Upgrade from v0.1.0: data, jobs, runs and services made by the v0.1.0
  binary survive; the first v0.2.0 run diffs against the run v0.1.0 made.
- Fresh install at the release commit: Windows 11 (clone to first results
  in 2m38s) and a clean Ubuntu 24.04, following QUICKSTART steps 1 to 7.
- `test/lab/m2/run.sh`, real software in Docker: a job scheduled every
  three minutes; between two runs one change of each kind (host removed and
  added, SSH keys regenerated, certificate replaced, nginx 1.26.3 to
  1.27.5, a port removed and another added). Exactly the seven changes were
  found, and delivered to Mailpit (STARTTLS with login), syslog-ng (strict
  RFC 5424 over UDP, TCP and TLS) and a webhook receiver that verified the
  signatures and refused the first call to exercise the retry.

## v0.1.0 (M1): standalone network discovery

First release. One binary for Windows and Linux; web console, REST API,
scheduler and scanner; SQLite or Postgres.

**Scanning**
- TCP connect discovery and port scans (`top100`, `top1000`, `all`, custom);
  SYN scan and ARP discovery on Linux with CAP_NET_RAW.
- Discovery that finds hosts which drop most probes: 33 TCP ports, the
  operating system's neighbour table on the local segment, ports remembered
  from earlier scans, and a per-job "every address is live" switch.
- Safe (default: 50 connections/s, 2 per host, sends nothing to services)
  and Standard (500/s, 10 per host, full detection) profiles; circuit
  breaker that pauses rather than abandons; scan windows; hard stop.
- Banners, HTTP title and server, TLS certificate details, SSH host keys;
  product, version and CPE from Rapid7 Recog fingerprints.
- Ports answered by antivirus mail proxies on the scanning machine are
  flagged "suspect" rather than reported as open.

**Scope guarantee**: every packet goes through `internal/scope`, which
checks it immediately before sending; a built-in deny list covers cloud
metadata, multicast and broadcast; tests and a packet-level witness prove
nothing outside scope is contacted.

**Server**: HTTPS only (self-signed on first run), local accounts with
Argon2id, Admin / Operator / Viewer, lockout, append-only audit log,
`field:value` search, CSV and JSON export, Windows service.

**Verified**
- A fully populated /24 on Safe: 8m28s, 660 of 660 open ports matching the
  kernel's listener list. A /24 with 30 live hosts: 3m28s.
- 17 real server packages in Docker: 33 of 33 listening ports found.
- VirtualBox VMs (Windows Server, Ubuntu, AlmaLinux) and a home LAN.
- Fresh install from the repository to first results: 2m48s on Windows 11;
  a clean Ubuntu 24.04 with nothing else installed also passes.

**Known limits**
- The Windows binary is not Authenticode-signed yet.
- Neighbour-table discovery only sees the scanner's own network segment;
  devices asleep at scan time (phones) answer nothing.
- Scheduling, diffs and alerts arrive in M2; remote sensors in M3.

Verify a download with `sha256sum -c SHA256SUMS` and the signature as
described in SECURITY.md.
