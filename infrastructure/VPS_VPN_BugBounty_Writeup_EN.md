# The Ultimate Setup: Offensive Architecture with VPS and VPNs for Bug Bounty

> **TL;DR:** If you've ever woken up unable to access Netflix, Infobae, or your scope because a WAF blacklisted your IP for raw-dogging ffuf without protection, this article is for you. Here's the real blueprint of how top hunters from the Bug Bounty Argentina community build distributed, cheap, and lethal offensive infrastructure.

---

## Prologue: The Day I Got Banned From Half The Internet

A while back, a community member shared their experience:

> *"Bug Bounty Tip: If you're gonna fuzz from home, double-check that your VPN is active. A friend of a friend woke up with half the internet blocked because they didn't turn on their VPN 🤣🤣😭😭😭"*

And it's no joke. When you start doing serious recon, you realize that throwing tools like `amass`, `ffuf`, or `nuclei` from your physical machine is like going to war with a BB gun. You expose yourself to:

- **Massive IP bans:** Akamai, Cloudflare, and other CDNs will blacklist you and you'll lose access not only to the target, but to all sites using that infrastructure.
- **Bandwidth limitations:** Your home ISP gives you 10 Mbps upload if you're lucky.
- **Instability:** Power goes out, WiFi drops, Windows restarts for updates... and you just lost 8 hours of scanning.

The solution isn't "being more careful." The solution is **building a distributed architecture**.

---

## The Professional Hunter's Architecture

Top-earning hunters don't have a magic setup. They have **separation layers** between manual analysis and automated brute force.

### Layer 1: Local Command Center (Your Machine)

This is where you do the fine work:

- **OS:** Mac, Windows, Linux (whatever feels comfy).
- **Main Tool:** Burp Suite Professional. Your scalpel.
- **VPN Client:** To switch regions when you get manually banned.
- **Browser with Proxy:** Firefox/Chrome with FoxyProxy configured to `127.0.0.1:8080`.

Your local machine is your **command center**. This is where you do spidering, craft surgical payloads, analyze responses in Burp's Repeater and Intruder.

### Layer 2: Cloud Execution Layer (VPS in the Cloud)

This is where you delegate everything heavy, noisy, and time-consuming:

- **OS:** Debian/Ubuntu Server (headless, no GUI).
- **Tools:** `subfinder`, `httpx`, `nuclei`, `ffuf`, `amass`, `gau`, `waybackurls`.
- **Orchestration:** `tmux` or `screen` to keep processes alive even when you disconnect.
- **Connectivity:** 100-500 Mbps symmetrical. Real bandwidth.

The workflow is simple:

1. SSH into the VPS.
2. Launch a `tmux` with 3 panes: one for passive recon, another for fuzzing, another for monitoring.
3. Fire commands and disconnect. The VPS keeps working 24/7.
4. When you come back, attach to the `tmux` and review results.

---

## The Harsh Reality of Costs

One of the biggest myths in the community is that you need a millionaire budget to do Bug Bounty. Here's the truth:

### Option 1: The Minimalist Setup ($0 - $6 USD/month)

**Providers:**
- **Oracle Cloud Free Tier:** 4 vCPU + 24GB RAM (free forever). Great to start.
- **AWS Free Tier:** 1 Core, 1GB RAM, free for 12 months.
- **Digital Ocean / Linode (with referral):** $200 USD initial credit.

**What it's good for:**
- Running `subfinder`, `httpx`, and `nuclei` on medium targets.
- Having a clean external IP to avoid bans.

**Limitation:**
- 1GB RAM falls short if you want to do massive subdomain permutations or run `amass` in brute force mode.

### Option 2: The Intermediate Setup ($6 - $12 USD/month)

**Providers:**
- **Contabo:** 4GB RAM, 4 vCore, 200GB SSD for $6.99 USD/month.
- **Digital Ocean:** 2GB RAM, 1 CPU for $12 USD/month (more stable for production).
- **Hetzner:** 2GB RAM, 2 vCore for $4.51 EUR/month (~$5 USD).

**What it's good for:**
- Running multiple tools in parallel.
- Heavy crawling with `gospider` or `katana`.
- Leaving `nuclei` running with all templates for days.

**Community comment:**
> *"OVH has cheap VPS, idk why everyone goes to DigitalOcean which is more expensive. Hetzner is super solid too."*

### Option 3: The Distributed Farm (Advanced Users)

Here we enter hardcore territory. An advanced strategy is renting multiple cheap servers to distribute scanning and avoid bans.

**The concept:**
- Rent VPS for $2-5 USD/month from providers like **Racknerd**, **OVH**, or **Hetzner**.
- Each VPS has its own public IP.
- Split the subdomain or path dictionary among nodes.
- Use **pssh** (Parallel SSH) or **Axiom** for distributed orchestration.

**Example with pssh:**

```bash
# hosts.txt file with VPS IPs
vps1.example.com
vps2.example.com
vps3.example.com

# Execute subfinder in parallel on all VPS
pssh -h hosts.txt -i "subfinder -d target.com -silent"
```

**Result:**
- Multiple different IPs hitting the target.
- If one gets banned, you have the others working.
- Massive combined throughput.

**Total cost:** ~$50-200 USD/month depending on scale.

---

## VPNs: Not Paranoia, It's Ammunition

A VPN isn't for "hiding from the NSA." It's for **rotating IPs** when you get blocked and keep moving forward.

### The VPN Workflow

1. Install an OpenVPN client on your VPS or local machine.
2. Contract a provider that allows **multiple simultaneous connections**:
   - **NordVPN:** Unlimited devices (~$3-4 USD/month with long plan).
   - **Surfshark:** Unlimited devices (~$2.50 USD/month with long plan).
   - **Mullvad:** 5 devices (€5/month, ~$5.50 USD).
   - **Private Internet Access (PIA):** Unlimited devices (~$2 USD/month with long plan).

3. Launch dockers or tmux panes that route their traffic through different VPN tunnels.

**Example with Docker:**

```bash
# Launch 10 containers, each connected to a different VPN
for i in {1..10}; do
  docker run -d --name vpn-worker-$i \
    -v /path/to/openvpn-config-$i.ovpn:/vpn/config.ovpn \
    my-vpn-image
done
```

Each worker exits with a different IP. If one gets banned, the other 9 keep operating.

### The Squid Proxy Trick

An advanced community configuration:

> *"I use multiple VPN connections with Squid Proxy, so I have IP1 on port 8081, IP2 on 8082, etc."*

**What does this achieve?**
- Can configure Burp Suite to rotate between proxies.
- If fuzzing with `ffuf`, can use `--proxy http://localhost:8081` and switch ports at will.

---

## X11 Forwarding Magic: Remote GUI Without VNC

One of the most common pain points is: *"How do I run Burp Suite on a VPS if it has no GUI?"*

The amateur answer is installing VNC or RDP. The professional answer is **X11 Forwarding**.

### What is X11 Forwarding?

It's a protocol that allows the **GUI of a program to run on the server**, but **render on your local screen**.

**Advantages:**
- Low latency (50-100 ms if the VPS is nearby).
- Don't need to install a complete Desktop Environment on the VPS.
- The heavy process (Burp making HTTP requests) runs on the VPS with its 100 Mbps bandwidth.

**Example:**

```bash
# Connect to VPS with X11 Forwarding enabled
ssh -X root@vps_ip

# Once inside, execute Burp
java -jar burpsuite_pro.jar
```

The Burp window opens on your screen, but **HTTP requests exit from the VPS IP**.

### Modern Alternative: Xpra

If X11 latency is high, you can use **Xpra**, which is like "screen for GUI applications":

```bash
# On the VPS
xpra start :100 --start-child="java -jar burpsuite_pro.jar"

# From your local PC
xpra attach ssh:root@vps_ip:100
```

---

## VPS Providers: Tier List for Bug Bounty (Updated 2026)

Not all providers are equal. Some will close your account at the first Abuse complaint. Here's the community's experience:

| Provider | Price (RAM/CPU) | Bandwidth | TOS Policy | Recommended Use |
|----------|-----------------|-----------|------------|-----------------|
| **Digital Ocean** | $6 (1GB/1CPU) | 1000 Mbps | Permissive with pentesting | Burn & Destroy: spin up, scan, destroy |
| **Linode (Akamai)** | $5 (1GB/1CPU) | 1000 Mbps | Very permissive | Main recon server 24/7 |
| **Contabo** | $6.99 (4GB/4vCore) | 200 Mbps | Permissive | Heavy tasks (Amass, Nuclei, Dalfox) |
| **Hetzner** | $4.51 EUR (2GB/2vCore) | Up to 20 TB included | Very permissive | Distributed farms |
| **Vultr** | $6 (1GB/1CPU) | 2000 Mbps | Permissive | High-speed recon |
| **Oracle Cloud Free** | $0 (24GB/4vCore) | Variable | Restrictive | Testing and learning |
| **AWS Lightsail** | $3.50 (512MB/1CPU) | 1 TB included | Moderate | Microservices and automation |

### Community Comments

> *"Some providers accept crypto for payment, which is useful if you want more anonymity in your tests."*

> *"Linode never failed me, and only once they contacted me because they thought I got pwned. One ticket and half an hour later everything was OK."*

> *"DigitalOcean knows what Bug Bounty is. They have their own program on HackerOne."*

> *"Hetzner has incredible price/performance ratio. For Europe it's unbeatable."*

---

## Axiom: Distributed Cloud Automation

**Axiom** is a dynamic automation framework that allows you to create, manage, and destroy VPS fleets on demand specifically for recon and bug bounty.

### What is Axiom?

Developed by **@pry0cc**, Axiom allows you to:
- Deploy multiple VPS in seconds on Digital Ocean, Linode, Azure, AWS.
- Automatically distribute recon tasks among all nodes.
- Scale horizontally: need more power, create more instances.
- Destroy everything when done to minimize costs.

### Axiom Advantages

1. **Cost-effective:** Only pay for the time you use the VPS.
2. **Scalable:** Can go from 1 to 50 instances in minutes.
3. **Modular:** Pre-built integrations with popular tools.
4. **IP Rotation:** Each instance has its own public IP.

### Basic Installation

```bash
# Clone Axiom
git clone https://github.com/pry0cc/axiom
cd axiom

# Run installer
./interact/axiom-configure

# Configure provider (example: Digital Ocean)
# You need your provider's API token
```

### Typical Workflow with Axiom

```bash
# 1. Create a fleet of 10 instances
axiom-fleet my-recon-fleet -i 10

# 2. Distribute subdomain enumeration
axiom-scan domains.txt -m subfinder -o subdomains.txt

# 3. Validate active hosts in parallel
axiom-scan subdomains.txt -m httpx -o active-hosts.txt

# 4. Destroy the fleet to stop paying
axiom-rm my-recon-fleet -f
```

### Real Use Cases

**Scenario 1: Massive Recon in Live Hacking Event**
```bash
# Create 20 instances
axiom-fleet lhe-2026 -i 20

# Scan 10,000 subdomains in parallel
axiom-scan massive-scope.txt -m nuclei -T lhe-2026 -o results/
```

**Scenario 2: Distributed Fuzzing**
```bash
# Distribute directory fuzzing across 5 VPS
axiom-scan endpoints.txt -m ffuf -w /path/to/wordlist.txt -T my-fleet
```

**Scenario 3: Screenshots of Thousands of Hosts**
```bash
# Capture screenshots of all active hosts
axiom-scan active-hosts.txt -m gowitness -T fleet
```

### Popular Axiom Modules

| Module | Tool | Use |
|--------|------|-----|
| `subfinder` | ProjectDiscovery | Subdomain enumeration |
| `httpx` | ProjectDiscovery | Active host validation |
| `nuclei` | ProjectDiscovery | Vulnerability scanning |
| `ffuf` | ffuf | Directory/Parameter fuzzing |
| `gowitness` | GoWitness | Web application screenshots |
| `amass` | OWASP Amass | Passive/active recon |

### Best Practices with Axiom

1. **Use Snapshots:** Create a snapshot with all your tools installed for quick deploy.
2. **Cost Monitoring:** Destroy fleets when not in use.
3. **Rate Limiting:** Configure `-rate` in your scans to not saturate targets.
4. **Storage:** Sync results to your local machine frequently.

**Official documentation:** [https://github.com/pry0cc/axiom](https://github.com/pry0cc/axiom)

---

## Automation Scripts: Bootstrap in 5 Minutes

When you spin up a new VPS, you can't waste 30 minutes installing tools manually. You need a **bootstrap script**.

### Base Script (Debian/Ubuntu)

```bash
#!/bin/bash
set -e

echo "[+] Updating system..."
apt update && apt upgrade -y

echo "[+] Installing base dependencies..."
apt install -y git wget curl tmux jq libpcap-dev build-essential python3-pip

echo "[+] Installing Go..."
wget https://go.dev/dl/go1.22.1.linux-amd64.tar.gz
tar -C /usr/local -xzf go1.22.1.linux-amd64.tar.gz
echo 'export PATH=$PATH:/usr/local/go/bin:~/go/bin' >> ~/.bashrc
source ~/.bashrc

echo "[+] Installing ProjectDiscovery tools..."
go install -v github.com/projectdiscovery/subfinder/v2/cmd/subfinder@latest
go install -v github.com/projectdiscovery/httpx/cmd/httpx@latest
go install -v github.com/projectdiscovery/nuclei/v3/cmd/nuclei@latest
go install -v github.com/projectdiscovery/katana/cmd/katana@latest

echo "[+] Installing Tomnomnom tools..."
go install -v github.com/tomnomnom/httprobe@latest
go install -v github.com/tomnomnom/waybackurls@latest
go install -v github.com/tomnomnom/gf@latest
go install -v github.com/tomnomnom/qsreplace@latest

echo "[+] Installing ffuf..."
go install -v github.com/ffuf/ffuf/v2@latest

echo "[+] Installing Amass..."
go install -v github.com/owasp-amass/amass/v4/...@master

echo "[+] Cloning wordlists..."
git clone https://github.com/danielmiessler/SecLists.git ~/wordlists

echo "[+] Setup complete. Welcome to your VPS."
```

### Usage

```bash
# Copy script to VPS
scp bootstrap.sh root@vps_ip:/root/

# Connect and execute
ssh root@vps_ip
chmod +x bootstrap.sh
./bootstrap.sh
```

In 5 minutes you have a VPS ready for recon.

---

## Real Use Cases: How The Community Uses It

### Case 1: Solo Hunter with 1 VPS

**Setup:**
- 1 Contabo VPS ($6.99 USD/month, 4GB RAM).
- Surfshark VPN ($2.50 USD/month).
- Burp Suite Pro on local machine.

**Workflow:**
1. Does passive recon from VPS: `subfinder`, `httpx`, `waybackurls`.
2. Analyzes results on local machine with Burp.
3. If needs to fuzz, activates VPN on machine and throws `ffuf` from there.

**Total cost:** ~$10 USD/month.

### Case 2: Hunter with Distributed Architecture

**Setup:**
- 1 main VPS on Hetzner (4GB RAM) for orchestration.
- Axiom configured with Digital Ocean for dynamic deploy.
- VPN with multiple connections.

**Workflow:**
1. Fleet deployment with Axiom when there's a LHE or large scope.
2. Automatic task distribution with Axiom modules.
3. Results centralized in main VPS.
4. Fleet destruction post-event.

**Total cost:** $5 USD/month (fixed VPS) + variable Axiom costs (only when used).

### Case 3: Advanced Setup with Permanent Farm

**Setup:**
- 10 cheap VPS on Hetzner/Racknerd ($3-5 USD each).
- 1 control server with database for orchestration.
- Custom script with `pssh` for task distribution.

**Workflow:**
1. Loads domains to scan in a queue/database.
2. Each VPS queries, takes work, executes, and reports.
3. Centralized system detects duplicates.

**Total cost:** ~$40-60 USD/month.

**Lesson learned:**
> *"Massive automation works for recon, but big bounties come from manual work and creativity."*

---

## Common Mistakes and How to Avoid Them

### Mistake 1: Running Amass Without VPS

> *"If you don't run Amass on a VPS, it'll kill your connection on your local PC. It's one of the best tools for finding subdomains."*

**Solution:** Always run heavy tools on VPS.

### Mistake 2: Forgetting to Activate VPN

> *"Got banned by Akamai, now I can't access several important sites. Now using VPN until my IP changes."*

**Solution:** Create an alias that validates if VPN is active before running tools:

```bash
# ~/.bashrc
function safe_ffuf() {
  if ! curl -s https://api.ipify.org | grep -q "YOUR_VPN_IP"; then
    echo "[!] VPN isn't active. Turn it on first."
    return 1
  fi
  ffuf "$@"
}
```

### Mistake 3: Using Ubuntu on VPS with 1GB RAM

> *"Ubuntu eats 600 MB RAM by default. On a 1GB VPS it kills you. Debian uses half, I think 200-300 MB."*

**Solution:** Always use Debian on low-resource VPS.

### Mistake 4: Not Limiting Request Rate

> *"Waited weeks with my home IP blocked... not even the PlayStation worked. Since then, never again did I throw scans from anywhere other than a VPS."*

**Solution:** Always use `--rate-limit` in tools like `ffuf` and `nuclei`:

```bash
ffuf -u https://target.com/FUZZ -w wordlist.txt -rate 50
nuclei -l targets.txt -rl 150
```

---

## Conclusion: Infrastructure Is Your Competitive Edge

Bug Bounty isn't just about finding vulnerabilities. It's about **optimizing your workflow** to cover more scope, faster, without burning your infrastructure.

**Ultimate Setup Summary:**

1. **Main VPS:** Contabo/Hetzner (4-8GB RAM) for 24/7 recon.
2. **Commercial VPN:** NordVPN/Surfshark for IP rotation.
3. **Local Burp Suite Pro:** For surgical manual analysis.
4. **Axiom (optional):** For dynamic scaling in events.
5. **Bootstrap script:** To spin up new VPS in 5 minutes.
6. **tmux/screen:** To keep processes alive.

---

## Bonus: Resources Mentioned by The Community

**VPS Providers:**
- [Digital Ocean](https://www.digitalocean.com/)
- [Linode (Akamai)](https://www.linode.com/)
- [Contabo](https://contabo.com/)
- [Hetzner](https://www.hetzner.com/)
- [Vultr](https://www.vultr.com/)
- [Oracle Cloud](https://www.oracle.com/cloud/free/) (Permanent free tier)

**Recommended VPNs:**
- [Mullvad VPN](https://mullvad.net/) (Privacy focus)
- [NordVPN](https://nordvpn.com/) (Unlimited devices)
- [Surfshark](https://surfshark.com/) (Unlimited devices)
- [Private Internet Access](https://www.privateinternetaccess.com/)
- [ProtonVPN](https://protonvpn.com/) (Free tier available)

**Automation Frameworks:**
- [Axiom](https://github.com/pry0cc/axiom) - Dynamic distributed automation
- [ReconFTW](https://github.com/six2dez/reconftw) - All-in-one framework
- [Osmedeus](https://github.com/j3ssie/osmedeus) - Automated Recon & Scanning
- [VPS Docker for Pentest](https://github.com/aaaguirrep/vps-docker-for-pentest)

**Provisioning Scripts:**
- [Terraform + Ansible for VPS](https://github.com/aaaguirrep/vps-docker-for-pentest)

---

## Final Words

This writeup is based on **real messages** from the **Bug Bounty Argentina** community, extracted from years of conversations about infrastructure, automation, and tooling.

It's not theory. It's the raw experience of hunters who cash out and share their real workflows.

If this helped you, share it. If you have a different setup, tell us in the comments.

*Happy Hunting*

---

**#BugBounty #VPS #VPN #Automation #Recon #OffensiveInfrastructure #InfoSec**
