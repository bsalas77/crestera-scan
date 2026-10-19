# Security

## Reporting a vulnerability

Do not open a public issue for a suspected vulnerability. Report it privately
so it can be fixed before it is disclosed:

- through GitHub: open the [crestera-scan](https://github.com/bsalas77/crestera-scan)
  repository's **Security** tab and choose **Report a vulnerability**; the
  report stays private between you and the maintainer;
- or by e-mail to **barry.salas@gmail.com**.

Include the version (`crestera-scan version`), the platform and steps to
reproduce.

A way to make Crestera Scan contact an address outside its configured scope is
treated as the most serious class of bug in this product, whatever its impact.

## What the product does to protect itself

- HTTPS only, TLS 1.2 or later. No plaintext listener exists.
- Listens on 127.0.0.1 by default.
- Local accounts with Argon2id password hashes (64 MiB, 2 passes).
- Five failed sign-ins lock an account for 15 minutes; sign-in attempts are
  also limited per client address. A wrong two-factor code counts as a
  failed sign-in.
- Optional two-factor sign-in (TOTP, RFC 6238) per account, which an
  administrator can make compulsory. Each code works once, even within its
  time window; ten one-time recovery codes are stored only as hashes.
- Optional single sign-on (Pro) with OpenID Connect: authorization code flow
  with PKCE, state bound to the browser by a cookie, nonce, and ID tokens
  verified here (RS256 only, keys of 2048 bits or more, issuer, audience,
  authorised party, expiry). Implemented with the standard library alone.
  Single sign-on accounts have no local password and never match a local
  account's name; their role is re-read from the provider at every sign-in.
- Session cookies are `Secure`, `HttpOnly`, `SameSite=Strict`; state-changing
  requests need a custom header, so cross-site requests are refused.
- The console is served with `default-src 'self'` and contains no inline
  script. Scanned text (banners, page titles, certificate fields) is only ever
  inserted as text.
- Search queries compile to parameterized SQL; no user input is concatenated
  into a statement.
- CSV exports neutralise cells that a spreadsheet would run as formulas.
- The audit log, the observations table and (Pro) each finding's history
  are append-only, enforced by database triggers.
- Apart from scans, alert deliveries and reverse-DNS lookups, the server
  connects out for these, each off until an administrator turns it on:
  - downloading the CISA KEV catalog, when an administrator clicks
    **Update now** or turns on daily updates. It goes to the configured https
    address (cisa.gov by default), through `HTTPS_PROXY` if set. Without it,
    the catalog built into the release is used;
  - signing in to Crestera Watch to read its agent list, every 15 minutes
    once connected, with a Watch account whose password and two-factor key
    are stored encrypted. Plain http needs an explicit setting.
  - (Pro, from when a license is applied) looking up the products it has
    identified in the NVD CVE API at services.nvd.nist.gov. A request
    names a vendor and product (for example apache:http_server), never an
    address, host name or version from your network.
  - (Pro, once an administrator lists the institution's domains) searching
    the public Certificate Transparency services crt.sh and
    api.certspotter.com for names under those domains, daily, and resolving
    the names found with the server's own DNS resolver. The requests name
    only the domains entered. Nothing is sent to the hosts found; they are
    scanned only if an administrator adds them to a scope.
  - (Pro, once single sign-on is set up) the identity provider's discovery,
    key and token endpoints during sign-ins.

  All of them refuse the built-in deny list, like alert deliveries.
- Alert deliveries (e-mail, syslog, webhook) refuse the built-in deny list
  (cloud metadata, multicast, broadcast). They are not limited to scan
  scopes: an administrator chooses the destinations, and the "Send test"
  button connects to whatever host and port a channel names. Only
  administrators can create channels.
- Text from scanned banners is stripped of control characters before it
  goes into an alert, so a hostile banner cannot forge extra syslog lines or
  break an e-mail.

## Verifying a release

Each release publishes `SHA256SUMS`, its ed25519 signature
(`SHA256SUMS.ed25519`, raw; `SHA256SUMS.sig`, the same in hex) and a
CycloneDX SBOM (`sbom.cdx.json`). Save the release signing public key as
`crestera-scan-release.pem`:

```
-----BEGIN PUBLIC KEY-----
MCowBQYDK2VwAyEAZtjJr7UfsF5dyKFAhIHv+E10/6aGHKozjXvMhGRGxt4=
-----END PUBLIC KEY-----
```

Then, in the folder with the downloads (OpenSSL 3, as on any current Linux):

```bash
openssl pkeyutl -verify -pubin -inkey crestera-scan-release.pem -rawin -in SHA256SUMS -sigfile SHA256SUMS.ed25519
sha256sum -c --ignore-missing SHA256SUMS
```

The first prints `Signature Verified Successfully`; the second `OK` for each
file you downloaded. The same key in hex, for `tools/relsign verify`:
`66d8c9afb51fb05e5dc8a1408481eff84d74ffa6861caa338d7bcc846446c6de`.

The Windows binary is not yet Authenticode-signed, so Windows SmartScreen and
some antivirus products may hold it on first launch.

## Verifying webhook deliveries

Every webhook POST carries

```
X-Crestera-Signature: t=<unix seconds>,v1=<hex HMAC-SHA256>
X-Crestera-Delivery: <id, the same on every retry of one delivery>
```

where the MAC is computed with the channel's signing secret over the bytes
`<t>.<raw request body>`. A receiver should recompute it, compare in constant
time, reject a `t` more than five minutes from its own clock, and use the
delivery id to ignore a retry it has already processed. In Python:

```python
import hashlib, hmac, time

def verified(secret: str, header: str, body: bytes) -> bool:
    parts = dict(p.split("=", 1) for p in header.split(","))
    mac = hmac.new(secret.encode(), parts["t"].encode() + b"." + body, hashlib.sha256).hexdigest()
    return hmac.compare_digest(mac, parts["v1"]) and abs(time.time() - int(parts["t"])) < 300
```

The signing secret is shown once, when the channel is created or its secret
is rotated. It is never returned by the API afterwards.

## Secrets at rest and backups

Secrets the server has to read back (TOTP seeds, SMTP passwords, webhook
signing secrets) are encrypted with AES-256-GCM before they reach the
database. The key is `secrets.key` in the data folder, created on first
start. With Postgres, a copy of the database alone therefore does not hand
over working credentials; with SQLite the key sits next to the database and
this protects a copied database file, not a copied data folder.

**Back up `secrets.key` with the database.** Restored without it, the
server still starts and every scan result is intact, but alert channels
need their passwords and secrets entered again and two-factor users sign in
with a recovery code, or have an administrator reset their second factor.
Secrets saved by v0.2.0 in plaintext are encrypted the first time v0.3.0
starts.

## Remote sensors

- A sensor only ever connects out, to the server's sensor endpoint
  (`CRESTERASCAN_SENSOR_ADDR`, off by default, a listener separate from the
  console). It listens on no port.
- Enrollment uses a one-time token from the console, valid for 24 hours and
  stored only as a hash. The sensor makes its own key, which never leaves it,
  and gets a client certificate from the server's private sensor CA
  (`sensor-ca.key` in the data folder). It trusts the server by that CA's
  fingerprint, given with the token, not by a host name.
- Every request after enrollment must present the sensor's current
  certificate (mutual TLS). Revoking a sensor, or the sensor renewing its
  certificate, makes the previous certificate useless at once.
- The sensor keeps its own list of ranges it may scan (`--allow`). A scope
  outside it fails on the sensor with nothing sent, so control of the server
  is not control of what a sensor's network is scanned with. On the sensor
  every probe still goes through the scope guard, as on the server.
- The server checks every address a sensor reports against the scope again
  and refuses the whole report, and fails the run, if one is outside it.

Back up `sensor-ca.key` and `sensor-ca.crt` with the data folder. Without
them the server makes a new CA and every sensor has to be enrolled again.
