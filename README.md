<p align="center">
  <h1 align="center">🔍 SubScope</h1>
  <p align="center">
    <b>A modular, multi-technique subdomain discovery & attack surface mapping toolkit</b>
  </p>
  <p align="center">
    <img src="https://img.shields.io/badge/python-3.8+-blue?logo=python&logoColor=white" alt="Python">
    <img src="https://img.shields.io/badge/license-MIT-green" alt="License">
    <img src="https://img.shields.io/badge/platform-macOS%20%7C%20Linux-lightgrey" alt="Platform">
  </p>
</p>

---

SubScope combines **6 enumeration engines**, **50+ subdomain takeover signatures**, **ASN intelligence**, and **hierarchy visualization** into a single, threadable CLI. Think of it as your personal recon pipeline — passive OSINT when you want stealth, active bruteforce when you need depth, and post-discovery analysis all in one place.

## ⚡ Features at a Glance

| Capability | Technique | Type |
|---|---|---|
| **Wordlist Bruteforce** | DNS resolution against a 5k+ wordlist | 🔴 Active |
| **Certificate Transparency** | crt.sh log search | 🟢 Passive |
| **Search Engine Dorking** | DuckDuckGo `site:` queries | 🟢 Passive |
| **DNS Aggregators** | HackerTarget · DNSDumpster · AnubisDB | 🟢 Passive |
| **SAN Extraction** | TLS certificate Subject Alternative Names | 🟢 Passive |
| **Favicon Hashing** | mmh3-based fingerprinting via CriminalIP | 🟢 Passive |
| **Subdomain Takeover** | 50+ service-specific fingerprint matching | 🔴 Active |
| **ASN & Reverse DNS** | WHOIS → BGP range → reverse lookup sweep | 🔴 Active |
| **Hierarchy Mapping** | Tree visualization & Maltego-ready CSV export | 🔵 Analysis |
| **Alive Filtering** | DNS-resolve to filter live hosts | 🔴 Active |
| **Bulk SAN Scan** | Mass TLS certificate SAN extraction | 🟢 Passive |
| **Recursive Enumeration** | Chain discovered subdomains as new targets | 🔴 Active |

## 📦 Installation

```bash
git clone https://github.com/xV4L3x/sub-scope.git
cd sub-scope
python -m venv .venv && source .venv/bin/activate
pip install -r requirements.txt
```

Create a `.env` file for API-dependent modules:

```env
SHODAN_API_KEY=your_shodan_key
CRIMINAL_IP_API_KEY=your_criminalip_key
```

## 🚀 Quick Start

```bash
# Bruteforce subdomains with default wordlist
python main.py dns enum -d example.com -w wordlists/subdomains.txt

# Run ALL enumeration methods at once
python main.py dns enum -d example.com -w wordlists/subdomains.txt -m all

# Save results to file
python main.py dns enum -d example.com -w wordlists/subdomains.txt -m all -o results.txt

# Turbo mode — 50 threads
python main.py dns enum -d example.com -w wordlists/subdomains.txt -m all -mT 50

# Check for subdomain takeover vulnerabilities
python main.py dns takeover -i subdomains.txt -mT 30
```

## 📖 Usage Guide

### Global Flags

| Flag | Description |
|---|---|
| `-o <file>` | Write results to an output file |
| `-oF <format>` | Custom output format (variables: `$d` = domain, `$i` = IP) |
| `-mT <n>` | Maximum concurrent threads |

### DNS Mode

All DNS commands follow the pattern:

```
python main.py dns <subcommand> [flags]
```

---

### 🔎 `enum` — Subdomain Enumeration

The core discovery engine. Combine multiple techniques in a single run.

```bash
python main.py dns enum -d <domain> [flags]
```

| Flag | Description |
|---|---|
| `-d <domain>` | Target domain **(required)** |
| `-w <wordlist>` | Path to subdomain wordlist (required for bruteforce) |
| `-m <modes>` | Enumeration modes — comma-separated IDs or `all` |
| `-s <sources>` | Certificate transparency sources (default: `crt.sh`) |
| `-sE <engines>` | Dorking search engines (default: `duckduckgo`) |
| `-dSE <engines>` | DNS aggregator sources (default: `hackertarget`) |
| `-r <depth>` | Enable recursive enumeration with max depth |

**Enumeration Modes:**

| ID | Method | Description |
|---|---|---|
| `0` | Bruteforce | Resolve subdomains from wordlist (active) |
| `1` | Cert Transparency | Query crt.sh certificate logs (passive) |
| `2` | Dorking | DuckDuckGo `site:` search (passive) |
| `3` | DNS Aggregators | HackerTarget, DNSDumpster, AnubisDB (passive) |
| `4` | SAN | Extract names from TLS certificates (passive) |
| `5` | Favicon | Favicon hash lookup via CriminalIP (passive) |

**Examples:**

```bash
# Passive-only recon (cert transparency + aggregators + SAN)
python main.py dns enum -d target.com -m 1,3,4

# Full-spectrum with recursion (depth 3)
python main.py dns enum -d target.com -w wordlists/subdomains.txt -m all -r 3

# All DNS aggregators
python main.py dns enum -d target.com -m 3 -dSE all

# Custom output format
python main.py dns enum -d target.com -m all -w wordlists/subdomains.txt -o scan.csv -oF "$d,$i"
```

---

### 🚩 `takeover` — Subdomain Takeover Detection

Checks subdomains against **50+ service fingerprints** including AWS S3, GitHub Pages, Heroku, Shopify, Netlify, and many more.

```bash
python main.py dns takeover -i <subdomains_file> [-mT <threads>]
```

**Supported services include:**
`AWS/S3` · `GitHub` · `Heroku` · `Shopify` · `Fastly` · `Netlify` · `Wordpress` · `Tumblr` · `Ghost` · `Pantheon` · `Bitbucket` · `Surge.sh` · `Webflow` · `Zendesk` · `Fly.io` · `Ngrok` · `Kinsta` · `Strikingly` · `Unbounce` · and 30+ more

---

### 🌐 `asn` — ASN & Reverse DNS Intelligence

Maps input domains to their ASN ranges via WHOIS/RADB, then sweeps the entire IP block using Hurricane Electric BGP data and PTR reverse lookups to uncover related infrastructure.

```bash
python main.py dns asn -i <domains_file> [-mT <threads>]
```

---

### 🌳 `hierarchy` — Subdomain Hierarchy Tree

Visualize the parent-child relationships between discovered subdomains in an indented tree format.

```bash
python main.py dns hierarchy -d <domain> -i <subdomains_file> [flags]
```

| Flag | Description |
|---|---|
| `--show-ip` | Resolve and display IP addresses |
| `--maltego` | Export as Maltego-compatible CSV (A/CNAME records) |

**Example output:**
```
example.com
\__api.example.com (93.184.216.34)
   \__v1.api.example.com (93.184.216.35)
   \__v2.api.example.com (93.184.216.36)
\__mail.example.com (93.184.216.40)
\__dev.example.com (93.184.216.50)
   \__staging.dev.example.com (93.184.216.51)
```

---

### ✅ `alive` — Live Host Filtering

Filter a list of subdomains down to only those that resolve to an IP address.

```bash
python main.py dns alive -i <subdomains_file> [-o <output_file>]
```

---

### 🔐 `san` — Bulk SAN Certificate Scan

Connect to each domain over TLS and extract all Subject Alternative Names from the certificate. Useful for discovering related domains hosted on shared infrastructure.

```bash
python main.py dns san -i <domains_file> [-mT <threads>]
```


## 🧩 Extending SubScope

SubScope's architecture is designed for easy extension:

- **Add a new enumeration technique** — Create a module in `modes/dns/enumeration/`, implement a function with the signature `(args, domain, max_threads)`, register it in `enumeration/main.py`
- **Add a new DNS aggregator** — Drop a module in `dns_aggregators/`, implement `(domain, args, url)`, register in `dns_aggregators/main.py`
- **Add a new search engine** — Add to `dorking/`, implement `(dork, args)`, register in `dorking/main.py`
- **Add takeover signatures** — Append to the `services` dict in `takeover.py` with an error regex pattern

## 📋 Requirements

- Python 3.8+
- Dependencies: `requests`, `dnsdumpster`, `duckduckgo_search`, `IPy`, `python-whois`, `certifi`, `cryptography`, `python-dotenv`, `mmh3`, `shodan`
- Optional: Shodan API key, CriminalIP API key (for favicon module)

## ⚠️ Disclaimer

This tool is intended for **authorized security testing and educational purposes only**. Always obtain proper authorization before scanning targets you do not own. The authors are not responsible for any misuse of this software.

## 📝 License

MIT License - see [LICENSE](LICENSE) for details.
