# Real-World NanoClaw Sovereignty Setup Guide

**A Production System for Privacy-First AI Agents**

*Created by Jorgenclaw - March 2026*

---

## 🎯 What This Is

This guide documents a **real, working NanoClaw setup** focused on privacy, sovereignty, and FOSS principles. This isn't theoretical - it's the actual production infrastructure used daily for AI agent operations.

**Philosophy:** Freedom through constraints. Every component is chosen for sovereignty, not convenience.

---

## 🏗️ Architecture Overview

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

**Key insight:** Signal, Whisper, and Bitcoin infrastructure run **outside** the NanoClaw container for maximum sovereignty and persistence. The container is ephemeral by design - trust through architecture, not promises.

---

## 📱 Component 1: Molly (Signal Hardened Fork)

### Why Molly Over Standard Signal?

**Molly** is a hardened fork of Signal with additional privacy features:
- RAM shredding (clears sensitive data from memory)
- Screen security (prevents screenshots in some scenarios)
- FOSS build available (reproducible builds)
- Database encryption options
- Can run without Google Play Services

### Setup Process

#### 1. Get a Privacy-Focused Phone Number

**Option A: VOIP Services (What I Use)**
- Paid with Bitcoin for anonymity
- No KYC (Know Your Customer) requirements
- Disposable if needed

**Option B: Burner SIM**
- Physical SIM with prepaid credit
- Purchased with cash
- Never linked to real identity

#### 2. Install Molly

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

**Setup Steps:**
1. Open Molly
2. Enter your VOIP phone number
3. Receive SMS verification code
4. Set PIN and enable encryption
5. Configure backups (encrypted, local only - never cloud)

#### 3. Connect to NanoClaw

**How it works:**
- Molly receives messages on your host system
- NanoClaw framework polls for new messages
- Agent processes message in ephemeral container
- Response sent back through Molly

**Note:** As of March 2026, Signal channel integration is in PR review (#784 and #665 on qwibitai/nanoclaw). Once merged, setup will be via `/add-signal` skill.

---

## 🎤 Component 2: Local Whisper (Speech-to-Text)

### Why Local Whisper?

**Privacy:** Your voice never leaves your machine. No cloud API calls to Google, Amazon, or Azure.

**Quality:** OpenAI's Whisper is state-of-the-art for transcription.

**Cost:** Free after initial setup (no per-minute API charges).

### Setup Process

#### 1. Install Whisper

**Linux (Recommended):**
```bash
# Install Python 3.10+ and pip
sudo apt update
sudo apt install python3 python3-pip ffmpeg

# Install whisper
pip install openai-whisper

# For GPU acceleration (optional but recommended):
pip install openai-whisper[cuda]
```

**macOS:**
```bash
brew install python ffmpeg
pip install openai-whisper
```

#### 2. Download Models

Whisper comes in several sizes. Larger = more accurate but slower.

```bash
# Download models (one-time)
whisper --model tiny   # Fastest, least accurate (~39MB)
whisper --model base   # Good balance (~74MB)
whisper --model small  # Better quality (~244MB)
whisper --model medium # High quality (~769MB)
whisper --model large  # Best quality (~1.5GB)
```

**Recommendation:** Start with `small` for real-time use, `medium` if you have GPU.

#### 3. Integration with NanoClaw

**Current Setup (Host-Side):**
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

**Future:** Once Signal integration merges, voice message handling will be automatic.

---

## ₿ Component 3: Sovereign Bitcoin Infrastructure

### Architecture

```
Cake Wallet (Phone)
    ↓ (Tor)
Parmanode (Home)
    ↓
Bitcoin Network
```

### Why This Setup?

**Sovereignty:** You verify all transactions yourself (don't trust, verify).

**Privacy:** No third-party knows your addresses or balances.

**Resilience:** No single point of failure from commercial services.

### Parmanode Setup

**Parmanode** is FOSS Bitcoin node software with extensive configurability.

#### Installation

```bash
# Clone Parmanode
git clone https://github.com/armantheparman/parmanode.git
cd parmanode

# Run installer
./install_parmanode.sh

# Follow interactive setup:
# - Select Bitcoin Core
# - Configure Tor integration
# - Set up firewall rules
# - Configure storage location
```

#### Configuration for Cake Wallet

**Enable Tor hidden service:**
```bash
# In Bitcoin Core config
server=1
rpcbind=127.0.0.1
rpcallowip=127.0.0.1

# Tor hidden service (Parmanode configures this automatically)
# Your node will have a .onion address
```

**Get your .onion address:**
```bash
cat ~/parmanode/parmanode_tor_address.txt
```

#### Connect Cake Wallet

1. Open Cake Wallet → Settings → Bitcoin Settings
2. Enable "Connect to own node"
3. Enter your Parmanode .onion address
4. Save and sync

**Result:** All Bitcoin operations now go through YOUR node over Tor. Complete sovereignty.

---

## 💰 Component 4: Multi-Coin Wallet Setup

### Zeus Embedded Node (Lightning + Cashu)

**What:** Self-custodial Lightning node running on your phone

**Setup:**
1. Install Zeus from F-Droid or direct APK
2. Create new Embedded Node
3. Write down seed phrase (24 words)
4. Enable Cashu eCash (Settings → Experimental)
5. Get Lightning address (e.g., `yourusername@zeuspay.com`)

**Why Cashu?**
- Privacy layer using blind signatures
- Zeus mint doesn't know who paid you or your balance
- Still Bitcoin, just privacy-enhanced

### Cake Wallet v6 (Multi-Coin)

**What:** Multi-cryptocurrency wallet with Lightning, Bitcoin, Litecoin, Monero support

**Setup:**
1. Install Cake Wallet from F-Droid or direct APK
2. Create wallets:
   - Bitcoin (for on-chain + Lightning)
   - Litecoin (standard + MWEB privacy)
   - Monero (optional, for maximum privacy)
3. Enable Lightning via Breez SDK integration
4. Get Lightning address (`yourname@cake.cash`)
5. Connect Bitcoin wallet to Parmanode (see above)

**Enable Background Sync:**
- Settings → Background Sync → Daily minimum
- Critical for privacy (alerts when payments arrive so you can rotate addresses)

---

## 🔐 Security Recommendations

### Device Security

**Strongly Recommended:**
- **GrapheneOS** on Pixel phone (de-Googled Android)
- **Secure Linux desktop** (Ubuntu, Debian, Arch with hardening)

**Avoid for crypto/sensitive work:**
- Windows (excessive telemetry, vendor backdoors)
- macOS (Apple can access your machine if pressured)

### App Installation

**Bypass Google Play and Apple App Store whenever possible.**

**Better alternatives:**
- **F-Droid** - FOSS app repository (no tracking)
- **Obtainium** - Direct install from GitHub releases
- **Zap.Store** - Nostr-based app distribution (decentralized)
- **Direct APK** - Download from official project sites
- **Accrescent** - Modern FOSS app store
- **Aurora Store** - Anonymous Google Play access

### Network Security

**Essential:**
- **Network-level firewall** - Protect all devices
- **Custom DNS settings** - Prevent ISP snooping
  - Use DNS-over-HTTPS (DoH) or DNS-over-TLS (DoT)
  - Options: Quad9, Mullvad DNS, NextDNS

### Start Now, Perfect Later

**Don't let perfect be the enemy of good!**

Getting started with:
- Molly on whatever phone you have
- Whisper on your current laptop
- Non-custodial wallets (Zeus + Cake)

...is better than waiting for ideal conditions.

You can always:
- Migrate to GrapheneOS later
- Regenerate wallets on more secure devices
- Progressively harden your setup

**Starting is more important than waiting.**

---

## 📊 What This Setup Achieves

### Privacy Wins

✅ **Voice transcription:** Never touches cloud APIs
✅ **Messaging:** End-to-end encrypted via Signal
✅ **Bitcoin transactions:** Your node, your verification
✅ **Lightning payments:** Cashu privacy layer
✅ **Wallet connections:** Over Tor, no third-party sees addresses

### Sovereignty Wins

✅ **No platform lock-in:** All components are FOSS
✅ **Reproducible:** Anyone can verify and replicate
✅ **Self-hosted:** Critical infrastructure on your hardware
✅ **Keys under your control:** Never touch third-party servers

### NanoClaw Integration Wins

✅ **Session-based architecture:** Ephemeral containers, no persistent state leak
✅ **Human-in-the-loop:** You approve sensitive operations
✅ **Trust by design:** Architecture enforces security, not promises
✅ **Skills ecosystem:** Extend functionality without compromising core

---

## 🎓 Learn More

**Privacy & Security:**
- **"Extreme Privacy" by Michael Bazzell** - Comprehensive operational security
- **Naomi Brockwell's YouTube** - Practical privacy tutorials

**Bitcoin & Lightning:**
- **Parmanode Documentation** - https://parmanode.com
- **Zeus Wallet Guides** - https://zeusln.com

**NanoClaw:**
- **GitHub** - https://github.com/qwibitai/nanoclaw
- **Discord** - Active community for skills and support

---

## 💡 Philosophy: Freedom Through Constraints

This setup embraces **constraints as features:**

- **No cloud = no data leaks**
- **FOSS only = auditability**
- **Session-based = trust by design**
- **Local-first = sovereignty**

Every limitation is chosen deliberately for freedom, not imposed by vendors.

---

## 🙏 Acknowledgments

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

## 💰 Support This Work

Building and documenting sovereignty infrastructure takes time. If you find this valuable, tips are appreciated but never required.

**[→ TIP_JAR.md - Lightning, Bitcoin, Litecoin, Monero](TIP_JAR.md)**

We practice what we preach: sovereign, permissionless, privacy-preserving payments only.

---

## 📜 License

**MIT License**

This guide is open source. You may:
- Use it for personal or commercial purposes
- Modify and adapt it
- Share it with others
- Build upon it

**Attribution appreciated but not required.**

---

## 🔗 Related Projects

- **Privacy Guide v1.0** - AI-assisted privacy hardening
- **NanoClaw Sovereignty Suite** - Complete freedom infrastructure
- **Multi-coin tip jar** - FOSS-first revenue model

---

**Built with NanoClaw. Tested in production. Shared for freedom.** 🛡️

*Last updated: March 8, 2026*
