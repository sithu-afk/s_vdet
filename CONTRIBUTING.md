# Contributing to s_vdet

Thanks for considering a contribution. This project is small and
intentionally simple - please keep that spirit in mind.

## Getting set up

```bash
git clone https://github.com/sithu-afk/s_vdet.git
cd s_vdet
pip install -e ".[dev]" --break-system-packages   # or drop the flag outside Kali/Debian
```

## Running tests

```bash
pip install pytest --break-system-packages
pytest tests/
```

## Adding a misconfiguration check

Checks live in `vdet/rules.py`. Each one is a plain function:

```python
def check_something(ip, open_ports, banners):
    if SOME_PORT in open_ports:
        return [finding(
            "RULE-ID", "Short title", "high",  # info|low|medium|high|critical
            "Full description of the risk and remediation.",
            port=SOME_PORT,
        )]
    return []
```

Register it in `ALL_CHECKS` at the bottom of the file. If the check does
extra network I/O beyond reading the already-collected banner (e.g. an
FTP login attempt, a TLS handshake), add it to `deep_checks` instead of
`fast_checks` so default scans stay fast.

## Adding to the CVE seed database

`vdet/data/cve_seed.py` is a small curated offline fallback, not meant to
be exhaustive. Good additions are well-known, high-impact CVEs with clear
OS/software version mapping (the kind that show up repeatedly in CTFs and
security training - EternalBlue, BlueKeep, etc.). Bulk/comprehensive CVE
data belongs in the local SQLite cache via `--update-cve-db`, not here.

## Adding an OS fingerprint signature

Banner-based signatures live in `vdet/fingerprint.py`
(`BANNER_OS_PATTERNS`). For anything Windows-specific and precise, prefer
extending `vdet/smb_fingerprint.py`'s `BUILD_TO_NAME` table instead - it's
authoritative where banner guessing is ambiguous.

## Pull requests

- Keep changes focused - one feature or fix per PR
- Add a test if you're touching parsing/matching logic (`targets.py`,
  `rules.py`, `cve.py`, `fingerprint.py`, `smb_fingerprint.py`)
- Update `README.md` if you add a flag or change default behavior

## Code of conduct

Be respectful. This tool touches security/offensive tooling territory -
keep discussion focused on legitimate defensive and authorized-testing use.
