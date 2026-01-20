# YourFrontDesk: Case Study

**How we built an AI-powered phone system for hotels in Gibraltar**

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

### Chapter 5: What's Next - Adding AI (Coming Soon)

**Current state:**
Right now, when you call the number, you hear "Hello World" and then it hangs up. This was just a test to prove the phone line works.

**Next step:**
Connect to Vapi, an AI voice platform that will:
1. Answer calls with a natural voice
2. Understand what the caller says
3. Have a real conversation
4. Take actions (book appointments, answer questions, transfer calls)

**What we need:**
1. Vapi account and API credentials
2. Configure what the AI should say and do
3. Update our server to route calls to Vapi

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
| Vapi | AI voice platform | Coming soon |

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
| Next | Connect Vapi AI to answer calls |

---

## Current Status

| Component | Status |
|-----------|--------|
| Server | ✅ Running |
| Remote Access | ✅ Working (Tailscale) |
| Phone System (Asterisk) | ✅ Running |
| Phone Line (SIP Trunk) | ✅ Connected & tested |
| AI Assistant (Vapi) | ⏳ Next step |

**Bottom line:** The phone infrastructure is complete and tested. When you call the DDI number, the call reaches our server. The next step is connecting Vapi so an AI answers instead of "Hello World".

---

## What We Learned

1. **IP whitelisting doesn't work for mobile teams** - Use Tailscale or similar for secure access from anywhere.

2. **Start simple** - We used Asterisk without a GUI. It's more work to configure, but uses less resources and we have full control.

3. **Test incrementally** - We tested "Hello World" before connecting AI. Always verify each step works.

4. **Document everything** - This guide exists because we wrote things down as we went.

5. **Internal networks are simpler** - Because our server is inside the provider's network, we didn't need username/password for the SIP trunk. It's a direct trusted connection.

---

## Next Steps

1. **Set up Vapi account** at https://vapi.ai
2. **Create an AI assistant** in Vapi dashboard
3. **Get the assistant ID**
4. **Connect Asterisk to Vapi** (5 minute configuration)
5. **Test end-to-end** - call the number, AI should answer

---

*Document created: January 20, 2026*
*Last updated: January 20, 2026*
