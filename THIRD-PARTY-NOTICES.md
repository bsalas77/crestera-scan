# Third-party notices

Crestera Scan is written in Go and links the open-source libraries below into
its binary. All are permissively licensed (MIT or BSD); none requires
Crestera Scan itself to be open-sourced, and none is modified from its
published form. This list is generated from the dependency graph of the
shipped binary (`cmd/crestera-scan`) with
[`go-licenses`](https://github.com/google/go-licenses):

```
go run github.com/google/go-licenses@latest report ./cmd/crestera-scan
```

CI runs `go-licenses check` on every change and fails if a dependency with any
other license appears.

Crestera Scan does not bundle, vendor, link or invoke nmap, masscan, Npcap or
libpcap. Its port lists are its own.

## Go modules

| Module | Version | License |
|---|---|---|
| github.com/dustin/go-humanize | v1.0.1 | MIT |
| github.com/google/uuid | v1.6.0 | BSD-3-Clause |
| github.com/jackc/pgpassfile | v1.0.0 | MIT |
| github.com/jackc/pgservicefile | 5a60cdf6a761 | MIT |
| github.com/jackc/pgx/v5 | v5.11.0 | MIT |
| github.com/jackc/puddle/v2 | v2.2.2 | MIT |
| github.com/mattn/go-isatty | v0.0.24 | MIT |
| github.com/ncruces/go-strftime | v1.0.0 | MIT |
| github.com/remyoudompheng/bigfft | 24d4a6f8daec | BSD-3-Clause |
| golang.org/x/crypto | v0.58.0 | BSD-3-Clause |
| golang.org/x/sync | v0.24.0 | BSD-3-Clause |
| golang.org/x/sys | v0.49.0 | BSD-3-Clause |
| golang.org/x/text | v0.43.0 | BSD-3-Clause |
| modernc.org/libc | v1.77.1 | BSD-3-Clause, with MIT and BSD components listed in its LICENSE-3RD-PARTY.md |
| modernc.org/mathutil | v1.7.1 | BSD-3-Clause (go-licenses reports "Unknown"; the module's LICENSE file is the 3-clause BSD text) |
| modernc.org/memory | v1.12.1 | BSD-3-Clause |
| modernc.org/sqlite | v1.60.1 | BSD-3-Clause; embeds SQLite, which is in the public domain |
| rsc.io/qr | v0.2.0 | BSD-3-Clause (draws the two-factor setup QR code) |

The Go standard library and runtime are BSD-3-Clause
(Copyright The Go Authors).

## Data

| Data | Source | License |
|---|---|---|
| Service fingerprints in `internal/engine/probe/recog/xml/` | [Rapid7 Recog](https://github.com/rapid7/recog), commit 2d19677 (2026-09-29): `ssh_banners`, `http_servers`, `ftp_banners`, `smtp_banners`, `mysql_banners`, `pop_banners`, `imap_banners` | BSD-2-Clause, Copyright 2014 Rapid7, Inc. Full text under License texts below. Files are unmodified. |
| Navigation icon shapes in `web/app.js` (`ICONS`) | [Lucide](https://lucide.dev): server, layers, activity, scan-line, crosshair, radio, bell, list-checks, globe, file-text, users, clipboard-list, sliders-horizontal | ISC License. Copyright (c) for portions of Lucide are held by Cole Bemis 2013-2022 as part of Feather (MIT); all other copyright (c) for Lucide are held by Lucide Contributors 2022. Permission to use, copy, modify, and/or distribute this software for any purpose with or without fee is hereby granted, provided that the above copyright notice and this permission notice appear in all copies. THE SOFTWARE IS PROVIDED "AS IS" AND THE AUTHOR DISCLAIMS ALL WARRANTIES WITH REGARD TO THIS SOFTWARE. |
| CISA Known Exploited Vulnerabilities catalog snapshot in `internal/kev/data/kev.json.gz` | [CISA KEV](https://www.cisa.gov/known-exploited-vulnerabilities-catalog), catalog 2026.10.15, reduced to the fields Crestera Scan uses by `tools/kevsnap` | A work of the U.S. Government (CISA), in the public domain in the United States. |

## License texts

The full license text of each module is in its source distribution, at the
version listed, and is reproduced in the release SBOM (`sbom.cdx.json`).

### MIT (go-humanize, pgpassfile, pgservicefile, pgx, puddle, go-isatty, go-strftime)

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
SOFTWARE.

Copyright holders: Dustin Sallings (go-humanize); Jack Christensen
(pgpassfile, pgservicefile, pgx, puddle); Yasuhiro Matsumoto (go-isatty);
Nuno Cruces (go-strftime).

### BSD-3-Clause (golang.org/x/*, google/uuid, bigfft, modernc.org/*, rsc.io/qr)

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

- Redistributions of source code must retain the above copyright notice, this
  list of conditions and the following disclaimer.
- Redistributions in binary form must reproduce the above copyright notice,
  this list of conditions and the following disclaimer in the documentation
  and/or other materials provided with the distribution.
- Neither the name of the copyright holder nor the names of its contributors
  may be used to endorse or promote products derived from this software
  without specific prior written permission.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.

Copyright holders: The Go Authors (golang.org/x/*, rsc.io/qr); Google Inc.
(google/uuid); Rémy Oudompheng and The Go
Authors (bigfft); The Libc, Mathutil, Memory and Sqlite Authors (modernc.org).

### BSD-2-Clause (Rapid7 Recog fingerprints)

Copyright (c) 2014, Rapid7, Inc.
All rights reserved.

Redistribution and use in source and binary forms, with or without
modification, are permitted provided that the following conditions are met:

1. Redistributions of source code must retain the above copyright notice, this
   list of conditions and the following disclaimer.
2. Redistributions in binary form must reproduce the above copyright notice,
   this list of conditions and the following disclaimer in the documentation
   and/or other materials provided with the distribution.

THIS SOFTWARE IS PROVIDED BY THE COPYRIGHT HOLDERS AND CONTRIBUTORS "AS IS"
AND ANY EXPRESS OR IMPLIED WARRANTIES, INCLUDING, BUT NOT LIMITED TO, THE
IMPLIED WARRANTIES OF MERCHANTABILITY AND FITNESS FOR A PARTICULAR PURPOSE ARE
DISCLAIMED. IN NO EVENT SHALL THE COPYRIGHT HOLDER OR CONTRIBUTORS BE LIABLE
FOR ANY DIRECT, INDIRECT, INCIDENTAL, SPECIAL, EXEMPLARY, OR CONSEQUENTIAL
DAMAGES (INCLUDING, BUT NOT LIMITED TO, PROCUREMENT OF SUBSTITUTE GOODS OR
SERVICES; LOSS OF USE, DATA, OR PROFITS; OR BUSINESS INTERRUPTION) HOWEVER
CAUSED AND ON ANY THEORY OF LIABILITY, WHETHER IN CONTRACT, STRICT LIABILITY,
OR TORT (INCLUDING NEGLIGENCE OR OTHERWISE) ARISING IN ANY WAY OUT OF THE USE
OF THIS SOFTWARE, EVEN IF ADVISED OF THE POSSIBILITY OF SUCH DAMAGE.
