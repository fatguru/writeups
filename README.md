# Offensive Security Writeups

> Real-world writeups on Red Team operations, Penetration Testing, Bug Bounty hunting, and Offensive Security techniques.

[![License: CC BY-SA 4.0](https://img.shields.io/badge/License-CC%20BY--SA%204.0-lightgrey.svg)](https://creativecommons.org/licenses/by-sa/4.0/)
[![Writeups](https://img.shields.io/badge/Writeups-1-blue.svg)](#)
[![Languages](https://img.shields.io/badge/Languages-EN%20|%20ES-green.svg)](#)

Content includes infrastructure setup, automation workflows, exploit techniques, and lessons learned from actual engagements. No corporate fluff. Just technical content that works.

---

## 📚 Table of Contents

### Infrastructure & Automation
- [**VPS & VPN Setup for Bug Bounty**](infrastructure/VPS_VPN_BugBounty_Writeup.md) 🇪🇸
  - [English Version](infrastructure/VPS_VPN_BugBounty_Writeup_EN.md) 🇬🇧
  - Distributed architecture, IP rotation, and automation
  - Cost analysis: $0 to $200/month setups
  - Axiom framework for dynamic cloud deployment
  - Based on 825+ real community messages

### Coming Soon
- [ ] Red Team Infrastructure Setup
- [ ] Post-Exploitation Techniques
- [ ] Active Directory Attack Paths
- [ ] Web Application Exploitation
- [ ] Mobile Security Testing
- [ ] Cloud Penetration Testing

---

## 🎯 What You'll Find Here

### Content Standards
- ✅ **Technical and Practical:** Zero fluff, pure actionable content
- ✅ **Real-World Experience:** From actual engagements and research
- ✅ **Code-Heavy:** Copy-paste ready commands and scripts
- ✅ **Bilingual:** Most content in English and Spanish

### Topics Covered
- **Red Team Operations:** Infrastructure, C2 setup, persistence
- **Penetration Testing:** Methodologies, workflows, tooling
- **Bug Bounty:** Automation, recon, exploitation techniques
- **Exploit Development:** Binary exploitation, shellcoding
- **Post-Exploitation:** Privilege escalation, lateral movement
- **Cloud Security:** AWS/Azure/GCP misconfigurations

---

## 📂 Repository Structure

```
offensive-security-writeups/
├── README.md
├── infrastructure/          # VPS, VPN, tooling setup
│   ├── VPS_VPN_BugBounty_Writeup.md
│   └── VPS_VPN_BugBounty_Writeup_EN.md
├── red-team/               # Red Team operations
│   └── [coming soon]
├── pentesting/             # Penetration testing
│   └── [coming soon]
├── exploitation/           # Exploit development
│   └── [coming soon]
└── resources/              # Scripts, tools, configs
    └── [coming soon]
```

---

## 🚀 Getting Started

### For Readers
Browse the writeups by category. Each includes:
- Context and motivation
- Step-by-step instructions
- Working code and commands
- Tools and resources references
- Lessons learned

### For Contributors
Want to contribute? Check out the contribution guidelines:

1. **Content must be**:
   - Based on real experience (no pure theory)
   - Technically accurate and verifiable
   - Well-formatted Markdown
   - Original or properly attributed

2. **Format**:
   - Clear structure with headers
   - Code blocks with language specified
   - English for technical terms, narrative in your language
   - Both English and Spanish versions (if possible)

3. **Submission**:
   - Fork this repo
   - Create a branch: `git checkout -b writeup/topic-name`
   - Submit a Pull Request with a clear description

---

## 🛠️ Tools & Frameworks Covered

**Recon & Automation:**
- ProjectDiscovery suite (subfinder, httpx, nuclei)
- Amass, ffuf, gobuster
- Axiom (dynamic cloud deployment)
- ReconFTW, Osmedeus

**Exploitation:**
- Burp Suite Professional
- Metasploit Framework
- SQLmap, XSStrike
- Custom exploit scripts

**Infrastructure:**
- Digital Ocean, Linode, Hetzner, Contabo
- Docker, Terraform, Ansible
- OpenVPN, Wireguard
- Cobalt Strike, Mythic C2

**Cloud:**
- AWS CLI, Azure CLI, gcloud
- ScoutSuite, Prowler, CloudMapper
- Pacu, ROADtools

---

## 📊 Stats

- **Writeups Published:** 1
- **Languages:** English, Spanish
- **Community Messages Analyzed:** 825+
- **Tools Covered:** 50+
- **Providers Reviewed:** 10+

---

## 🌟 Featured Content

### Infrastructure & Automation
**[The Ultimate VPS & VPN Setup for Bug Bounty](infrastructure/VPS_VPN_BugBounty_Writeup.md)**

Learn how top hunters build distributed, cost-effective infrastructure:
- 3 setup tiers: from $0 to $200/month
- VPN provider comparison (2026 prices)
- Axiom framework for dynamic scaling
- X11 Forwarding for remote GUI
- Real use cases from the community

*Available in: 🇪🇸 Spanish | 🇬🇧 English*

---

## 🔗 Useful Resources

**Communities:**
- [Bug Bounty Argentina (Telegram)](https://t.me/bugbountyargentina)
- [r/bugbounty](https://reddit.com/r/bugbounty)
- [r/netsec](https://reddit.com/r/netsec)
- [HackerOne Community](https://hackerone.com/directory)

**Platforms:**
- [HackerOne](https://www.hackerone.com/)
- [Bugcrowd](https://www.bugcrowd.com/)
- [Intigriti](https://www.intigriti.com/)
- [YesWeHack](https://www.yeswehack.com/)

**Learning:**
- [PortSwigger Web Security Academy](https://portswigger.net/web-security)
- [PentesterLab](https://pentesterlab.com/)
- [HackTheBox](https://www.hackthebox.eu/)
- [TryHackMe](https://tryhackme.com/)

---

## 📄 License

This content is licensed under **Creative Commons Attribution-ShareAlike 4.0 International (CC BY-SA 4.0)**.

You are free to:
- ✅ Share and adapt the content
- ✅ Use it commercially

As long as you:
- ✅ Give appropriate credit
- ✅ Share under the same license

See [LICENSE](LICENSE) for details.

---

## 💬 Contact & Contribution

**Found a bug in the content?** Open an [Issue](../../issues)

**Want to contribute?** Submit a [Pull Request](../../pulls)

**Questions?** Join the [Bug Bounty Argentina](https://t.me/bugbountyargentina) community

---

## 🙏 Acknowledgments

This content is possible thanks to:
- **Bug Bounty Argentina Community:** For sharing real-world experiences
- **Open Source Tooling:** ProjectDiscovery, Tomnomnom, PortSwigger, and the entire infosec community
- **Individual Hunters:** Who share their workflows and lessons learned

---

*"Knowledge grows when shared. Hack responsibly."*

**#OffensiveSecurity #RedTeam #Pentesting #BugBounty #InfoSec #Hacking**
