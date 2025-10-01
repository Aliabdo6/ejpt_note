# eJPT Study Notes

Personal study repository for INE's **Junior Penetration Tester (eJPT)** certification. Contains organized notes and practical scripts.

---

## 📂 Repository Structure

```
ejpt_note/
│
├── 1 Information_Gathering/
│   ├── 1 Passive Information Gathering/
│   │   ├── 1 Reconnaissance & Footprinting.md
│   │   ├── 2 WHOIS_Enumeration.md
│   │   ├── 3 Netcraft Website Footprinting.md
│   │   ├── 4 DNS Recon.md
│   │   ├── 5 WAF Detection.md
│   │   ├── 6 Subdomain Enumeration.md
│   │   ├── 7 Google Dorking.md
│   │   ├── 8 Email Harvesting.md
│   │   └── 9 Leaked password databases.md
│   │
│   └── 2 Active Information Gathering/
│       ├── 1 DNS Zone Transfers.md
│       ├── 2 Host Discovery with Nmap.md
│       └── 3 Port Scanning with Nmap.md
│
├── 2 Scanning_Enumeration/         (coming soon)
├── 3 Vulnerability_Assessment/      (coming soon)
├── 4 Exploitation/                  (coming soon)
├── 5 Post_Exploitation/             (coming soon)
├── 6 Web_Application_Testing/       (coming soon)
│
└── scripts/                         (automation helpers)
    ├── recon.sh
    ├── enum.py
    └── ...
```

---

## 🎯 What's Inside

**Study Notes**: Each topic has its own markdown file with:

- Core concepts and definitions
- Practical commands and examples
- Tool usage and syntax
- Tips and common pitfalls

**Scripts**: Automation tools to speed up repetitive tasks during labs and practice.

---

## 🚀 Quick Start

Clone the repository:

```bash
git clone https://github.com/Aliabdo6/ejpt_note.git
cd ejpt_note
```

Browse the notes:

```bash
cd "1 Information_Gathering/1 Passive Information Gathering"
cat "2 WHOIS_Enumeration.md"
```

Or open in your editor:

```bash
code .
```

---

## 📚 Study Progress

- [x] **Module 1: Information Gathering**
  - [x] Passive Information Gathering (9 topics)
  - [x] Active Information Gathering (3 topics)
- [ ] **Module 2: Scanning & Enumeration**
- [ ] **Module 3: Vulnerability Assessment**
- [ ] **Module 4: Exploitation**
- [ ] **Module 5: Post-Exploitation**
- [ ] **Module 6: Web Application Testing**

---

## 🛠️ Scripts Usage

All scripts accept command-line arguments (no hardcoded values):

```bash
# Example: Reconnaissance script
./scripts/recon.sh --target 10.10.10.5 --output ./results

# Example: Enumeration helper
python3 scripts/enum.py --target 10.10.10.5 --service http
```

Each script includes a `--help` flag for usage instructions.

---

## ⚠️ Legal Notice

**For educational purposes only.**

Only use these materials on systems you own or have explicit permission to test. Unauthorized access is illegal.

---

## 📝 License

MIT License - See LICENSE file for details.

---

## 🤝 Contributing

Suggestions and improvements are welcome! Keep notes concise and scripts dynamic (no hardcoded targets).

---

**Status**: 🟢 Active Development | **Last Updated**: October 2025
