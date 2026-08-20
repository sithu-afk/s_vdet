<p align="center">
  <img src="https://raw.githubusercontent.com/sithu-afk/s_vdet/main/logo.png" alt="s_vdet logo" width="180">
</p>

# s_vdet — Network & Host Vulnerability Detector

A CLI tool for network reconnaissance and host vulnerability detection:
TCP port scanning, service/OS fingerprinting, misconfiguration checks,
and CVE matching. Command name: **`vdet`**.


> **⚠ LEGAL NOTICE:** Only scan systems you own or have explicit written
> authorization to test. Unauthorized scanning may violate computer misuse
> laws in your jurisdiction. `vdet` shows a confirmation prompt before every
> scan (skip with `--yes` for automation you've already cleared).

---

## Install

```bash
git clone https://github.com/sithu-afk/s_vdet.git
cd s_vdet
unzip vdet 
pip install -e . --break-system-packages
```

This installs the `vdet` command. No third-party dependencies required —
pure Python standard library, so it also runs under **Termux on Android**.

Platform support:
| Runs on (as the tool) | Linux | macOS | Windows | Termux (Android) |
|---|---|---|---|---|
| Supported | ✅ | ✅ | ✅ | ✅ |

| Scans as a target | Linux hosts | Windows hosts | Network devices | Android (ADB exposed) | iOS |
|---|---|---|---|---|---|
| Supported | ✅ | ✅ | ✅ | ⚠️ partial | ❌ (limited) |

---

## Quick start

```bash
# Default scan — top 100 ports, human-readable report
vdet 192.168.1.10

# Full deep scan (banners, OS fingerprint, CVE match, TLS/FTP checks) -> JSON
vdet 192.168.1.0/24 --deep -oJ report.json

# Fast recon, top 100 ports, verbose, aggressive timing
vdet 10.0.0.5 --top-ports 100 -v -T4

# Targeted scan for a specific host/services + CVE matching
vdet 10.0.0.5 -p 445,3389,135 --deep --severity high

# Scan a list of targets from a file
vdet -iL targets.txt -o results.txt
```

## Target formats

```bash
vdet 192.168.1.10                # single IP
vdet 192.168.1.0/24               # CIDR
vdet 192.168.1.10-50              # short range (last octet)
vdet 192.168.1.10-192.168.1.50    # full range
vdet example.com                  # hostname (DNS resolved)
vdet -iL targets.txt              # one target spec per line, '#' = comment
```

## Flags

```
Port options:
  -p, --ports RANGE      Ports to scan, e.g. 1-1000 or 22,80,443 (default: top 100)
  --top-ports N          Scan only the top N most common ports

Scan behavior:
  -sV                    Enable service/banner version detection
  --deep                 Full checks: banners, OS detect, CVE scan, TLS/FTP checks
  -T 0-5                 Timing template: 0=paranoid .. 5=insane (default 3)
  --os-detect            Run OS fingerprinting (implied by --deep)
  --cve-scan             Match fingerprint against CVE database (implies --os-detect)
  --cve-source {auto,local,nvd,seed}   CVE data source (default: auto)
  --min-cvss SCORE       Only report CVEs with CVSS >= SCORE

CVE database management:
  --update-cve-db        Refresh local CVE cache from the NVD API and exit
  --cve-target CPE/KEYWORD   What to fetch (repeatable), e.g.
                          --cve-target "windows_server_2012_r2"
  --nvd-api-key KEY      NVD API key for higher rate limits

Output:
  -o, --output FILE      Write human-readable report to FILE
  -oJ FILE               Write JSON report to FILE
  --severity LEVEL       Minimum severity to display (info/low/medium/high/critical)
  -v, --verbose          Verbose progress output
  --no-color             Disable colored output

Authorization:
  --yes, --no-consent-check   Skip the authorization confirmation prompt
```

## Exit codes (for CI/CD gating)

| Code | Meaning |
|---|---|
| 0 | No findings above "low" |
| 1 | Medium-severity findings present |
| 2 | High-severity findings present |
| 3 | Critical-severity findings present |

---

## How it works

```
targets.py     -> parse target specs (IP/CIDR/range/hostname/file) into IP list
scanner.py     -> threaded TCP connect scan (no root required)
banner.py      -> connect to open ports, grab service banners / HTTP headers / TLS info
fingerprint.py -> combine banners + TTL + open-port patterns -> best-guess OS/software
rules.py       -> static misconfiguration checks (Telnet, anon FTP, exposed DBs, weak TLS...)
cve.py         -> match fingerprint against CVEs: local SQLite cache -> NVD API -> seed DB
report.py      -> terminal / JSON / text report generation, severity-ranked
consent.py     -> authorization confirmation gate
cli.py         -> `vdet` argument parsing and pipeline orchestration
```

### CVE matching tiers

1. **Local SQLite cache** — built via `vdet --update-cve-db --cve-target ...`,
   pulls from the live NVD API once and caches results for instant, rate-limit-free
   lookups on every future scan.
2. **Live NVD API** — used directly with `--cve-source nvd` (needs network access
   to `services.nvd.nist.gov`; get a free API key for higher rate limits).
3. **Built-in seed DB** (`vdet/data/cve_seed.py`) — a small curated set of
   well-known, high-impact CVEs (EternalBlue, BlueKeep, SMBGhost, PrintNightmare,
   vsftpd backdoor, etc.) mapped to common OS/software fingerprints. Always
   available offline, guarantees non-empty results for common lab/CTF targets
   (e.g. a Windows Server 2012 R2 box) even with zero network access.

`--cve-source auto` (default) checks the local cache first, then always layers
in the seed DB for coverage.


## Roadmap ideas
- Async scanning (`asyncio`) for larger subnets
- SMB/RPC-based precise OS/build fingerprinting via `impacket` (optional extra)
- Default-credential checks for common admin panels
- HTML report output
- GitHub Action wrapper for CI-gated infra scanning

## License
MIT

