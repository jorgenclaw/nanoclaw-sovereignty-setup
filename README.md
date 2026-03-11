# Real-World NanoClaw Sovereignty Setup Guide

*Synthesized by Jorgenclaw (AI agent) and Claude Code (host AI), with direct feedback and verification from Scott Jorgensen*

**A Production System for Privacy-First AI Agents**

*March 2026*

---

## What You're Setting Up

This guide walks you through building a **real, working privacy-first infrastructure** for running an AI agent (NanoClaw) on your own terms. This is not theoretical — it is the actual production system used daily.

Here is what the components are in plain language:

- **Molly** — A hardened version of the Signal messaging app. This is how you talk to your AI agent securely.
- **Local Whisper** — Speech-to-text software that runs entirely on your machine, so your voice recordings never leave your computer.
- **Parmanode** — Software that runs a Bitcoin node (a full copy of the Bitcoin transaction history) on your own hardware, so you can verify transactions yourself instead of trusting a company.
- **Zeus and Cake Wallet** — Self-custodial wallets (meaning you hold the keys, not a company) for Bitcoin, Lightning, and other cryptocurrencies on your phone.
- **NanoClaw** — The AI agent framework that ties it all together, running in ephemeral containers (temporary sandboxes that get cleaned up after each use).

**Philosophy:** Freedom through constraints. Every component is chosen for sovereignty, not convenience.

> **Feeling stuck?** Don't be afraid to ask Claude or any AI assistant directly where you are in the process and what to do next. You can paste error messages, describe what you see on screen, or just say "I'm lost" — and get help.

---

## How the Pieces Fit Together

```
┌─────────────────────────────────────────────────────────────┐
│                    HOST SYSTEM (Your Machine)                │
│                                                              │
│  ┌──────────────┐  ┌──────────────┐  ┌──────────────────┐  │
│  │ Molly/Signal │  │   Whisper    │  │   Parmanode      │  │
│  │  (Hardened)  │  │  (Local STT) │  │ (Bitcoin Node)   │  │
│  └──────┬───────┘  └──────┬───────┘  └────────┬─────────┘  │
│         │                 │                    │            │
│         │                 │                    │            │
│  ┌──────┴─────────────────┴────────────────────┴─────────┐  │
│  │                                                        │  │
│  │              NanoClaw Container                        │  │
│  │  ┌──────────────────────────────────────────────┐     │  │
│  │  │  • Processes messages from Signal/Molly      │     │  │
│  │  │  • Receives transcribed voice via Whisper    │     │  │
│  │  │  • Session-based, ephemeral containers       │     │  │
│  │  │  • Skills: MoltBook, Clawstr, agent-browser  │     │  │
│  │  └──────────────────────────────────────────────┘     │  │
│  │                                                        │  │
│  └────────────────────────────────────────────────────────┘  │
│                                                              │
└─────────────────────────────────────────────────────────────┘
           │                        │
           ▼                        ▼
    Zeus Embedded Node      Cake Wallet → Parmanode
    (Phone - Lightning)     (Phone - Multi-coin)
```

**Key insight:** Signal, Whisper, and Bitcoin infrastructure run **outside** the NanoClaw container for maximum sovereignty and persistence. The container is ephemeral by design — trust through architecture, not promises.

---

## Setting Up Secure Messaging with Molly

### Why Molly instead of standard Signal?

**Molly** is a hardened fork (a modified version built from the same source code) of Signal that adds extra privacy protections:

- RAM shredding — clears sensitive data from memory after use
- Screen security — prevents screenshots in some scenarios
- FOSS (free and open-source software) build available with reproducible builds, so anyone can verify the code
- Database encryption options
- Can run without Google Play Services

### How to get a privacy-focused phone number

**Option A: VOIP Services (What I Use)**
- Paid with Bitcoin for anonymity
- No KYC (Know Your Customer) requirements — meaning you do not have to prove your identity
- Disposable if needed

**Option B: Burner SIM**
- Physical SIM with prepaid credit
- Purchased with cash
- Never linked to real identity

### How to install Molly on your phone

**Android Installation:**
```bash
# Option 1: F-Droid (recommended)
# Add Molly F-Droid repo: https://molly.im/fdroid/
# Install "Molly-FOSS" (Google-free version)

# Option 2: Direct APK
# Download from https://github.com/mollyim/mollyim-android/releases
# Verify signature
# Install APK
```

**After installation, set up your account:**
1. Open Molly
2. Enter your VOIP phone number
3. Receive SMS verification code
4. Set PIN and enable encryption
5. Configure backups — keep them encrypted and local only, never cloud

### How Molly connects to NanoClaw

Here is the flow:
- Molly receives messages on your host system
- NanoClaw polls for new messages automatically
- Your AI agent processes each message in an ephemeral container
- The response is sent back through Molly

**Note:** As of March 2026, Signal channel integration is in PR review (#784 and #665 on qwibitai/nanoclaw). Once merged, setup will be via `/add-signal` skill.

---

## Setting Up Local Voice Transcription with Whisper

### Why run speech-to-text locally?

**Privacy:** Your voice never leaves your machine. There are no cloud API calls to Google, Amazon, or Azure.

**Quality:** OpenAI's Whisper is state-of-the-art for transcription, and it runs entirely on your hardware.

**Cost:** Free after initial setup — no per-minute API charges.

### How to install Whisper

**On Linux (recommended):**
```bash
# Install Python (the programming language Whisper is built with),
# pip (a tool for installing Python packages), and ffmpeg (an audio converter)
sudo apt update
sudo apt install python3 python3-pip ffmpeg

# Install the Whisper transcription software
pip install openai-whisper

# If you have an NVIDIA GPU and want faster transcription (optional):
pip install openai-whisper[cuda]
```

**On macOS:**
```bash
# Install Python and ffmpeg using Homebrew (a package manager for macOS)
brew install python ffmpeg

# Install Whisper
pip install openai-whisper
```

### How to download transcription models

Whisper offers several model sizes. Larger models are more accurate but slower to run.

```bash
# Download the model you want (this is a one-time download)
whisper --model tiny   # Fastest, least accurate (~39MB)
whisper --model base   # Good balance (~74MB)
whisper --model small  # Better quality (~244MB)
whisper --model medium # High quality (~769MB)
whisper --model large  # Best quality (~1.5GB)
```

**Recommendation:** Start with `small` for everyday use. If you have a dedicated GPU, try `medium` for better accuracy.

### How to connect Whisper to NanoClaw

**Current Setup (runs on the host machine):**
```bash
#!/bin/bash
# save as: ~/scripts/transcribe-signal-voice.sh

AUDIO_FILE=$1
OUTPUT_DIR=~/signal-transcriptions

whisper "$AUDIO_FILE" \
  --model small \
  --language en \
  --output_dir "$OUTPUT_DIR" \
  --output_format txt

# Send transcription to NanoClaw
# (Implementation depends on your NanoClaw setup)
```

**Future:** Once Signal integration merges, voice message handling will be automatic — you will not need this script.

---

## Setting Up Your Own Bitcoin Node

### How the pieces connect

```
Cake Wallet (Phone)
    ↓ (Tor)
Parmanode (Home)
    ↓
Bitcoin Network
```

### Why run your own node?

**Sovereignty:** You verify all transactions yourself. The Bitcoin community says "don't trust, verify" — this is how you do it.

**Privacy:** No third-party company knows your addresses or balances.

**Resilience:** No single point of failure from commercial services going down or changing their terms.

### How to install Parmanode

**Parmanode** is FOSS (free and open-source) Bitcoin node software that walks you through setup with interactive prompts.

```bash
# Download the Parmanode installer from GitHub
git clone https://github.com/armantheparman/parmanode.git
cd parmanode

# Run the installer — it will ask you questions along the way
./install_parmanode.sh

# The installer will guide you through:
# - Selecting Bitcoin Core (the main Bitcoin software)
# - Configuring Tor integration (for anonymous network connections)
# - Setting up firewall rules (to protect your machine)
# - Choosing where to store the blockchain data
```

### How to configure your node for Cake Wallet

**Enable a Tor hidden service** so your phone wallet can reach your node privately:
```bash
# These settings go in the Bitcoin Core configuration file.
# Parmanode sets most of this up for you automatically.
server=1
rpcbind=127.0.0.1
rpcallowip=127.0.0.1

# Tor hidden service (Parmanode configures this automatically)
# Your node will get a .onion address — a special Tor-only URL
```

**Find your .onion address** (you will need this for your wallet):
```bash
# This prints the address your wallet will connect to
cat ~/parmanode/parmanode_tor_address.txt
```

### How to connect Cake Wallet to your node

1. Open Cake Wallet on your phone, then go to Settings, then Bitcoin Settings
2. Enable "Connect to own node"
3. Enter the .onion address you found above
4. Save and sync

**What success looks like:** Your wallet will begin syncing with your own node. All Bitcoin operations now go through YOUR hardware over Tor (an anonymizing network). You have complete sovereignty over your transaction verification.

---

## Setting Up Your Wallets

### Zeus Embedded Node — Lightning payments and Cashu privacy

**What this gives you:** A self-custodial (you hold the keys, not a company) Lightning node running directly on your phone for fast, low-fee Bitcoin payments.

**How to set it up:**
1. Install Zeus from F-Droid or download the APK directly from the project site
2. Create a new Embedded Node
3. Write down your seed phrase (24 words) — this is your backup if you lose your phone. Store it offline and keep it safe.
4. Enable Cashu eCash by going to Settings, then Experimental
5. Get your Lightning address (for example, `yourusername@zeuspay.com`)

**What is Cashu?** Cashu is a privacy layer that uses blind signatures — a cryptographic technique where the Zeus mint (the service that holds funds temporarily) does not know who paid you or what your balance is. Your Bitcoin is still Bitcoin, just with an extra layer of privacy.

### Cake Wallet v6 — Multi-cryptocurrency wallet

**What this gives you:** A single wallet app that handles Bitcoin (on-chain and Lightning), Litecoin, and Monero.

**How to set it up:**
1. Install Cake Wallet from F-Droid or download the APK directly
2. Create your wallets:
   - Bitcoin — for on-chain transactions and Lightning payments
   - Litecoin — with standard and MWEB privacy (a Litecoin feature that hides transaction amounts)
   - Monero — optional, for maximum transaction privacy
3. Enable Lightning via the Breez SDK integration (found in settings)
4. Get your Lightning address (for example, `yourname@cake.cash`)
5. Connect the Bitcoin wallet to your Parmanode using the instructions above

**Turn on background sync** — this is important for privacy:
- Go to Settings, then Background Sync, then set it to Daily minimum
- This alerts you when payments arrive so you can rotate addresses, which prevents others from linking your transactions together

---

## Security Recommendations

### Choosing secure devices

**Strongly recommended:**
- **GrapheneOS** on a Pixel phone — a de-Googled version of Android built for privacy
- **Secure Linux desktop** — Ubuntu, Debian, or Arch with hardening applied

**Avoid for crypto and sensitive work:**
- Windows — sends extensive telemetry (usage data) to Microsoft, and vendor backdoors are a known risk
- macOS — Apple retains the ability to access your machine if pressured by authorities

### Where to get your apps

**Bypass Google Play and Apple App Store whenever possible.** Those stores track what you install and can remove apps at any time.

**Better alternatives:**

| Source | What it is |
|--------|-----------|
| **F-Droid** | A FOSS app repository with no tracking |
| **Obtainium** | Installs apps directly from GitHub releases |
| **Zap.Store** | Nostr-based app distribution — fully decentralized |
| **Direct APK** | Download from official project websites |
| **Accrescent** | A modern FOSS app store |
| **Aurora Store** | Access Google Play anonymously, without a Google account |

### Protecting your network

**Essential steps:**
- **Network-level firewall** — protects all devices on your network
- **Custom DNS settings** — prevents your ISP (internet service provider) from seeing which websites you visit
  - Use DoH (DNS-over-HTTPS) or DoT (DNS-over-TLS) — these encrypt your DNS queries (the lookups your computer does to find website addresses)
  - Good options: Quad9, Mullvad DNS, NextDNS

### Start now, improve later

You do not need to get everything perfect before you begin. Getting started with:

- Molly on whatever phone you already have
- Whisper on your current laptop
- Non-custodial wallets like Zeus and Cake

...is far better than waiting for ideal conditions.

You can always:
- Migrate to GrapheneOS later
- Regenerate wallets on more secure devices
- Progressively harden your setup over time

**Starting is more important than waiting.**

---

## What This Setup Achieves

### Privacy wins

- **Voice transcription** never touches cloud APIs — it stays on your machine
- **Messaging** is end-to-end encrypted via Signal/Molly
- **Bitcoin transactions** are verified by your own node, not a third party
- **Lightning payments** have Cashu's privacy layer protecting your balances
- **Wallet connections** run over Tor, so no third party sees your addresses

### Sovereignty wins

- **No platform lock-in** — all components are FOSS (free and open-source software)
- **Reproducible** — anyone can verify and replicate this setup
- **Self-hosted** — critical infrastructure runs on your own hardware
- **Keys under your control** — your private keys never touch third-party servers

### NanoClaw integration wins

- **Session-based architecture** — ephemeral containers mean no persistent state can leak between sessions
- **Human-in-the-loop** — you approve sensitive operations before they happen
- **Trust by design** — architecture enforces security, not promises or policies
- **Skills ecosystem** — you can extend functionality without compromising the core system

---

## Learn More

**Privacy and security:**
- **"Extreme Privacy" by Michael Bazzell** — a comprehensive guide to operational security
- **Naomi Brockwell's YouTube** — practical privacy tutorials you can follow along with

**Bitcoin and Lightning:**
- **Parmanode Documentation** — https://parmanode.com
- **Zeus Wallet Guides** — https://zeusln.com

**NanoClaw:**
- **GitHub** — https://github.com/qwibitai/nanoclaw
- **Discord** — an active community for skills and support

---

## Philosophy: Freedom Through Constraints

This setup embraces **constraints as features:**

- **No cloud = no data leaks**
- **FOSS only = auditability** (anyone can read the source code)
- **Session-based = trust by design**
- **Local-first = sovereignty**

Every limitation is chosen deliberately for freedom, not imposed by vendors.

---

## Acknowledgments

**Influenced by:**
- Michael Bazzell ("Extreme Privacy")
- Naomi Brockwell (NBTV)
- NanoClaw community
- FOSS philosophy

**Built on:**
- Signal/Molly protocol
- OpenAI Whisper
- Bitcoin Core
- Parmanode
- Zeus Lightning
- Cake Wallet
- NanoClaw framework

---

## Support This Work

Building and documenting sovereignty infrastructure takes time. If you find this valuable, tips are appreciated but never required.

**[→ TIP_JAR.md - Lightning, Bitcoin, Litecoin, Monero](TIP_JAR.md)**

We practice what we preach: sovereign, permissionless, privacy-preserving payments only.

---

## Troubleshooting

If you run into problems, here are some common issues:

| Problem | What it means | What to do |
|---------|---------------|------------|
| Molly won't receive verification SMS | Your VOIP number may not support SMS from Signal's verification service | Try a different VOIP provider, or use a burner SIM instead |
| `whisper` command not found | Python did not install Whisper to your system PATH | Run `pip show openai-whisper` to verify it is installed, then try `python3 -m whisper` instead |
| Parmanode sync is very slow | The initial blockchain download is roughly 600GB and can take days | This is normal — let it run. Using an SSD and wired internet connection will help |
| Cake Wallet won't connect to your node | The .onion address may be wrong, or Tor may not be running | Double-check the address in `~/parmanode/parmanode_tor_address.txt` and verify the Tor service is active |
| NanoClaw container fails to start | Dependencies may be missing or the container image may need rebuilding | Check the logs and try rebuilding with `./container/build.sh` |

---

## License

**MIT License**

This guide is open source. You may:
- Use it for personal or commercial purposes
- Modify and adapt it
- Share it with others
- Build upon it

**Attribution appreciated but not required.**

---

## Related Projects

- **Privacy Guide v1.0** — AI-assisted privacy hardening
- **NanoClaw Sovereignty Suite** — Complete freedom infrastructure
- **Multi-coin tip jar** — FOSS-first revenue model

---

**Built with NanoClaw. Tested in production. Shared for freedom.**

*Last updated: March 8, 2026*
