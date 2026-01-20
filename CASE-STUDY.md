# YourFrontDesk: Case Study

> **How we built an AI-powered phone system for hotels in Gibraltar**

---

## What is YourFrontDesk?

YourFrontDesk is an AI receptionist that answers phone calls for hotels and property management companies. Instead of a human sitting at a desk 24/7, an AI assistant answers calls, helps guests, and handles bookings.

**The problem we solved:** Hotels need someone to answer phones around the clock, but staffing is expensive. An AI can answer instantly, never sleeps, and costs a fraction of a human receptionist.

**What we built:** A phone system that:
1. Receives real phone calls from a Gibraltar phone number
2. Routes those calls to an AI voice assistant
3. The AI has natural conversations with callers
4. Can book appointments, answer questions, or transfer to a human

---

## The Simple Version

```
Guest calls hotel number
        ↓
Call goes to our server in Gibraltar
        ↓
Server sends call to AI (Vapi)
        ↓
AI answers: "Hello, thank you for calling. How can I help you?"
        ↓
Guest has a conversation with AI
```

---

## The Story: How We Built This

### Chapter 1: Getting a Server (January 2026)

**The challenge:** We needed a computer that could receive phone calls 24/7.

**The solution:** We rented a virtual server from Gibtelecom in Gibraltar. Think of it like renting a computer that lives in a data center instead of your office.

**What we got:**
- A server running Linux (Debian 12)
- A permanent internet address (static IP)
- A phone line connected directly to the server
- A Gibraltar phone number (DDI)

**Cost:** Part of the Gibtelecom Cloud SIP package

---

### Chapter 2: The Access Problem (January 6, 2026)

**The challenge:** We needed to control the server remotely, but securely.

When you rent a server, you control it by typing commands from your laptop. This is called "SSH" - it's like a remote keyboard for the server.

The problem: If anyone on the internet can try to log in, hackers will try to break in. They run automated programs that guess passwords all day long.

**First attempt - IP Whitelisting:**
Our hosting contact tried to only allow our specific internet address. But our address changes when we move locations or our router restarts. This created an endless loop:
1. Provider allows our IP
2. We move to a coffee shop
3. Our IP changes
4. We're locked out
5. Contact provider again
6. Repeat forever

**The solution - Tailscale:**
We installed Tailscale, which creates a private tunnel between our laptop and the server. It's like having a secret passageway that works from anywhere.

Now we can:
- Access the server from home, office, or anywhere
- No need to keep asking the provider to update settings
- The public "door" stays locked to hackers

**Result:** Secure access from anywhere in the world.

---

### Chapter 3: Installing the Phone System (January 17, 2026)

**The challenge:** The server was just a blank computer. We needed phone system software.

**What is a PBX?**
PBX stands for "Private Branch Exchange" - it's the software that manages phone calls. It decides:
- What happens when someone calls
- Where to route the call
- What message to play
- How to connect to AI

**Our choice: Asterisk**
We chose Asterisk because:
- It's free and open-source
- It's lightweight (works on our small server)
- It's been around for 20+ years - very reliable
- Used by millions of phone systems worldwide

**Why not FreePBX?**
FreePBX has a nice web interface (point and click), but it needs 4x more memory. Our server only has 1GB of RAM, so Asterisk alone was the better choice.

**The installation:**
Since Asterisk isn't included in the standard software library, we had to build it from scratch:
1. Downloaded the source code
2. Installed all the required tools
3. Compiled (built) the software - took about 15 minutes
4. Configured it to start automatically

**Result:** Phone system running, listening for calls on port 5060.

---

### Chapter 4: Connecting the Phone Line (January 20, 2026)

**The challenge:** The phone system was running, but not connected to real phone lines.

**How phone calls work on the internet:**
Traditional phones use copper wires. Modern systems use "SIP" (Session Initiation Protocol) - this sends voice calls over the internet, just like video calls.

Gibtelecom has a "SIP trunk" - this is the bridge between the traditional phone network and our server.

**The configuration:**
We learned from our provider that:
- The SIP server address is on an internal network
- No username/password needed (it's a direct private connection)
- Our server just needs to point at that address

**The first test:**
We configured Asterisk to play "Hello World" when someone calls. Then we called the number...

**Problem:** The call arrived but was rejected! The phone number came with a "+" symbol, and our system wasn't expecting that format.

**The fix:** Updated the configuration to accept numbers starting with "+".

**Second test:** Called again. SUCCESS! We heard "Hello World" through the phone.

**Result:** Phone line working. Calls arrive at our server.

---

### Chapter 5: Connecting the AI Voice Assistant (January 20, 2026)

**The challenge:** We had a working phone system, but calls just played "Hello World". We needed to connect to Vapi, an AI platform that would have actual conversations with callers.

**What is Vapi?**
Vapi is an AI voice platform. When we send a call to Vapi:
1. It listens to what the caller says
2. Converts speech to text
3. An AI (like ChatGPT) generates a response
4. Text is converted back to natural speech
5. The caller hears the AI respond

All this happens in under a second - the conversation feels natural.

**The problem: SIP configuration**
Connecting two phone systems (Asterisk and Vapi) requires precise settings. Both systems need to "speak the same language" and trust each other.

Our first attempts failed with cryptic errors:
- "403 Forbidden" - Vapi was rejecting our calls
- "401 Unauthorized" - authentication wasn't working

**The breakthrough:**
After researching Vapi's documentation, we discovered the correct format for connecting. The key was using a specific address format that Vapi expects - without the right format, Vapi didn't know where to route our calls.

**The test:**
We made a test call. The server received it, forwarded to Vapi, and... the AI answered! We had our first conversation with the AI receptionist.

**Result:** Calls to the Gibraltar number now connect to an AI that can have natural conversations.

---

### Chapter 6: Under Attack - Securing the Server (January 20, 2026)

**The surprise discovery:**
While debugging the Vapi connection, we noticed something alarming in the server logs. Thousands of login attempts from IP addresses around the world.

**What was happening?**
Automated bots constantly scan the internet looking for phone systems. When they find one, they try to:
- Register fake phone extensions
- Make free international calls through your system
- Use your server for fraud

In just a few hours, we saw:
- 87,000+ attempts from one attacker
- 3,000+ from another
- Multiple attackers trying different techniques

**The risk:**
If attackers succeed, they could:
- Run up massive phone bills
- Use the system for scam calls
- Damage the hotel's reputation

**Our response:**

**Step 1: Identify the attackers**
We analyzed the server logs to find which IP addresses were attacking us and blocked the worst offenders immediately.

**Step 2: Install automatic protection**
We set up fail2ban - software that monitors login attempts. When it sees too many failures from one address, it automatically blocks that attacker for 24 hours.

**Step 3: Create an allow-list**
We configured the firewall to explicitly trust:
- The phone company (Gibtelecom)
- The AI platform (Vapi)
- Our secure remote access (Tailscale)

Everyone else has to pass through the automatic protection system.

**Step 4: Make it permanent**
We saved all the security rules so they survive server restarts.

**The result:**
Within minutes of setup, the security system had already blocked 2 new attackers automatically. The system now protects itself 24/7.

**What we learned:**
Any server exposed to the internet will be attacked. It's not a question of "if" but "when". Building security from day one is essential.

---

### Chapter 7: The AI Receptionist is Live (January 20, 2026)

**Current state:**
When you call the Gibraltar number:
1. The call arrives at our server
2. Our server authenticates with Vapi
3. Vapi's AI assistant answers
4. You have a natural conversation
5. The call ends cleanly

**What the AI can do:**
- Greet callers professionally
- Answer questions about the hotel
- Provide information from a knowledge base
- Transfer to a human if needed

**Next steps:**
- Train the AI with specific hotel information
- Set up business hours handling
- Add booking capabilities
- Configure voicemail for after-hours

---

## Technical Architecture (For the Curious)

```
┌─────────────────────────────────────────────────────────────────────┐
│                        THE COMPLETE SYSTEM                          │
└─────────────────────────────────────────────────────────────────────┘

┌──────────────┐     ┌──────────────┐     ┌──────────────┐
│   CALLER     │     │   TELECOM    │     │  OUR SERVER  │
│              │     │   PROVIDER   │     │              │
│  Guest dials │────▶│  Phone       │────▶│  Asterisk    │
│  the hotel   │     │  Network     │     │  PBX         │
│  number      │     │              │     │              │
└──────────────┘     └──────────────┘     └──────────────┘
                                                 │
                                                 ▼
                                          ┌──────────────┐
                                          │   VAPI AI    │
                                          │              │
                                          │  AI talks    │
                                          │  to guest    │
                                          │              │
                                          └──────────────┘
```

### The Components Explained

| Component | What it is | Our setup |
|-----------|------------|-----------|
| Server | A computer in a data center | Debian 12 Linux |
| Public IP | The server's address on the internet | Static IP from provider |
| Tailscale | Secure remote access tunnel | Lets us control the server from anywhere |
| Asterisk | Phone system software | Routes calls, plays messages |
| SIP Trunk | Connection to phone network | Direct internal connection |
| DDI | The actual phone number | Gibraltar number |
| Vapi | AI voice platform | Answers calls with AI assistant |
| fail2ban | Automatic attack blocker | Bans attackers for 24 hours |

---

## Key Information at a Glance

### Server Access
```
ssh -i [keyfile].pem [user]@[tailscale-ip]
```
(Requires Tailscale running)

### Server Details
| Property | Value |
|----------|-------|
| Operating System | Debian 12 Linux |
| Phone Software | Asterisk 20.17.0 |
| RAM | 1 GB |
| Storage | 30 GB |

### SIP Trunk (Phone Line)
| Property | Value |
|----------|-------|
| Provider | Gibtelecom |
| Port | 5060 |
| Authentication | None (direct internal link) |
| Status | ✅ Working |

---

## Glossary: What All These Terms Mean

| Term | Plain English |
|------|---------------|
| **Server** | A computer that runs 24/7 in a data center |
| **VM (Virtual Machine)** | A "virtual" server - we rent part of a bigger computer |
| **Linux / Debian** | The operating system (like Windows, but for servers) |
| **SSH** | Remote control for servers - like a keyboard over the internet |
| **Tailscale** | Software that creates a secure private tunnel to the server |
| **IP Address** | A computer's address on the internet (like a phone number for computers) |
| **Port** | A numbered "door" for different types of traffic (5060 = phone calls) |
| **PBX** | Phone system software that routes calls |
| **Asterisk** | The specific PBX software we use (free, open-source) |
| **SIP** | The protocol (language) that internet phone calls use |
| **SIP Trunk** | A phone line delivered over the internet |
| **DDI** | Direct Dial In - the actual phone number people call |
| **RTP** | The protocol that carries the actual voice audio |
| **Vapi** | An AI platform that can answer phone calls with a virtual assistant |
| **fail2ban** | Security software that automatically blocks attackers |
| **Firewall** | Rules that control what network traffic is allowed |

---

## Timeline: What Happened When

| Date | Milestone |
|------|-----------|
| Jan 6, 2026 | Server access configured via Tailscale |
| Jan 6, 2026 | Closed public port 22 (more secure) |
| Jan 17, 2026 | Asterisk PBX installed from source |
| Jan 17, 2026 | Basic configuration completed |
| Jan 20, 2026 | SIP trunk connected (no auth needed) |
| Jan 20, 2026 | **First successful test call!** |
| Jan 20, 2026 | Discovered SIP scanner attacks (87,000+ attempts) |
| Jan 20, 2026 | **Vapi AI connected and working!** |
| Jan 20, 2026 | Security hardening completed (fail2ban + firewall) |
| Next | Train AI with hotel-specific knowledge |

---

## Current Status

| Component | Status |
|-----------|--------|
| Server | ✅ Running |
| Remote Access | ✅ Working (Tailscale) |
| Phone System (Asterisk) | ✅ Running |
| Phone Line (SIP Trunk) | ✅ Connected & tested |
| AI Assistant (Vapi) | ✅ Connected & working |
| Security (fail2ban) | ✅ Active, auto-blocking attackers |
| Firewall | ✅ Configured with whitelist |

**Bottom line:** The complete AI phone system is live. When you call the Gibraltar number, an AI receptionist answers and has a natural conversation with you. The system is protected against attacks and runs 24/7.

---

## What We Learned

1. **IP whitelisting doesn't work for mobile teams** - Use Tailscale or similar for secure access from anywhere.

2. **Start simple** - We used Asterisk without a GUI. It's more work to configure, but uses less resources and we have full control.

3. **Test incrementally** - We tested "Hello World" before connecting AI. Always verify each step works.

4. **Document everything** - This guide exists because we wrote things down as we went.

5. **Internal networks are simpler** - Because our server is inside the provider's network, we didn't need username/password for the SIP trunk. It's a direct trusted connection.

6. **Read the documentation carefully** - The Vapi connection took several attempts because we used the wrong address format. The official docs had the answer.

7. **Security can't wait** - Within hours of going live, we had 90,000+ attack attempts. Always build security from day one.

8. **Automate security** - Manual IP blocking doesn't scale. Automated tools like fail2ban protect you while you sleep.

---

## Next Steps

1. **Train the AI** - Add hotel-specific knowledge and FAQs
2. **Set up business hours** - Different behaviour for day/night
3. **Add booking capabilities** - Let the AI take reservations
4. **Configure voicemail** - For after-hours or when AI can't help
5. **Monitor and improve** - Review calls and refine AI responses

---

*Document created: January 20, 2026*
*Last updated: January 20, 2026 - Added Vapi integration and security chapters*
