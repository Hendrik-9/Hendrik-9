<div align="center">

<img src="https://readme-typing-svg.demolab.com?font=JetBrains+Mono&weight=700&size=28&duration=2000&pause=1000&color=00E57A&center=true&vCenter=true&width=750&lines=Building+on-chain+systems.;//Hours wasted here: 534H;it+WORKED+on+Localhost!!;measure+first%2C+optimize+later.;ship+small%2C+then+make+it+less+embarrassing." alt="Typing SVG" />

<br/>

[![Rust](https://img.shields.io/badge/Rust-000000?style=for-the-badge&logo=rust&logoColor=white)](https://rust-lang.org)
![Solidity](https://img.shields.io/badge/Solidity-363636?style=for-the-badge&logo=solidity&logoColor=white)
[![TypeScript](https://img.shields.io/badge/TypeScript-3178C6?style=for-the-badge&logo=typescript&logoColor=white)](https://typescriptlang.org)
[![Python](https://img.shields.io/badge/Python-3776AB?style=for-the-badge&logo=python&logoColor=white)](https://python.org)
[![Linux](https://img.shields.io/badge/Linux-FCC624?style=for-the-badge&logo=linux&logoColor=black)](https://kernel.org)

</div>

---

## `whoami`

```rust
struct Builder {
    name:     "Hendrik",
    location: "Germany",
    focus:    ["backend", "infrastructure", "blockchain", "automation"],
    status:   "learning more than shipping — on purpose",
    motto:    "understand the system before writing much code",
}
```

I build backend systems and infrastructure, mostly around blockchain. I like the
parts other people find annoying: latency, observability, the silent failure at
2am that has no log line. I learn by building the whole thing end to end —
node, service, dashboard, alerts — and then writing down what bit me.

---

## What I'm working on right now

> A liquidation bot on Base, watching Aave V3 and Morpho Blue from a self-hosted node.
> Live since May 2026. Liquidations executed so far: zero.
> Either my filters are too strict, or the market is suspiciously well-behaved.
> But on it!. The bot is having a very relaxing time regardless.

---

## Projects

<details open>
<summary><b>⚡ Liquidation bot (Base)</b> &nbsp;<img src="https://img.shields.io/badge/LIVE-00E57A?style=flat-square" /></summary>

<br/>

A real-time Rust service that monitors lending positions and reacts when one
becomes liquidatable. For me this was mostly an exercise in low-latency async
Rust, batching RPC calls sensibly, and being able to *see* what a running system
is doing instead of guessing.

**Architecture**

```
position monitor
    └── multicall3 batch scanner ......... 500 positions per eth_call
            └── filter engine ............ health-factor threshold
                    └── route optimizer .. QuoterV2, 4 fee tiers
                            └── execution  flash-loan, atomic
```

**Observability layer**

```
running service
    ├── Prometheus metrics ...... :9091
    ├── structured logs ......... tracing-appender, 3 async streams
    └── Telegram alerts ......... crash · error spike · node disconnect · low wallet
```

**Latency — measured vs. goal**

```
state scan       37ms
route calc       48ms
gas estimate     13ms
tx submission   113ms
─────────────────────
total           213ms   ← what I actually measured
target          8–15ms  ← where I want to get (WS subs + in-memory cache)
```

The 8–15ms is a goal, not a result. I'm leaving it in so future-me has something
specific to be disappointed about.

**The honest part:** it runs, it's stable, it has caught nothing. That's either a
filter bug or a quiet market, but still on it....

`Rust` · `alloy 2.0` · `Tokio` · `Solidity` · `Foundry` · `TypeScript` · `Prometheus`

</details>

<details>
<summary><b>🖥️ Self-hosted Base node (op-node + reth)</b> &nbsp;<img src="https://img.shields.io/badge/OPS-0052FF?style=flat-square" /></summary>

<br/>

A full Base L2 node on a rented VM. This is the project where I actually learned
how an L2 fits together — a consensus client and an execution client talking over
an authenticated engine API.

**How the layers stack**

```
op-node ........ consensus — derives blocks from L1
   │ JWT auth (engine API)
reth ........... execution — runs the EVM, holds state
   │ alloy (HTTP + WebSocket)
application .... my services read from here
```

**Things that bit me — and what they taught me**

```
JWT format mismatch       silent auth failure, no error    → trust no log that isn't there
SIDECARS flag = false     sync dies, nothing logged        → read the source, not the docs
firewall after container  peer discovery never recovers    → order of operations matters
exec API left open        easy to forget on fresh deploy   → lock to app IP, every time
```

That third evening — reading op-node source to find an env flag with no log line —
was not the evening I had planned. It is, however, the thing I learned the most from.

**Why self-host:** for latency-sensitive work, local sub-millisecond calls beat a
shared, rate-limited managed RPC. For a normal app, a managed RPC is completely
fine and I would not put myself through this without a reason. I had a reason.

`reth` · `op-node` · `Debian 13` · `systemd` · `Docker Compose` · `JWT` · `Linux`

</details>

<details>
<summary><b>🔺 Cross-DEX bot</b> &nbsp;<img src="https://img.shields.io/badge/LIVE-00E57A?style=flat-square" /></summary>

<br/>

An on-chain price scanner across 37 pools and 4 DEXs. Every candidate trade is
simulated before submission, so I don't pay gas to find out I was wrong. This was
the first thing I ever shipped on-chain, so it has the structural elegance of a
first pancake — it works, and I can see exactly where I'd do it better now.

**Flow**

```
new block event
    └── multicall batch price refresh ... 37 pools, 4 exchanges
            ├── direct spread detection
            └── triangular route detection
                    └── simulate → filter → execute atomically
```

The simulation step kills routes that don't profit after fees. The honest result:
the spreads it finds are small once gas is paid. Turns out plenty of people had
this idea before... i did, who could see that one coming? — a genuinely useful lesson, mildly bruising.

`TypeScript` · `Solidity` · `Hardhat` · `React` · `Vite` · `Multicall3`

</details>

<details>
<summary><b>🔒 Truth Protocol</b> &nbsp;<img src="https://img.shields.io/badge/HACKATHON%20IDEA-b478ff?style=flat-square" /></summary>

<br/>

Built as a concept during a weekend hackathon-style sprint. Status: idea.
Will it be finished? Almost certainly not. Is it on here anyway? Obviously.

The concept was a mechanism-design puzzle: an on-chain market for information
bounties, where a commitment is staked, contributors fund it, and threshold
encryption releases the content when a target is met. Fun to think about for a
weekend.

**Open questions I cheerfully did not solve**

```
verification signal   where does it come from without being gameable?
anonymity             how to not de-anonymize the people it's meant to protect?
legal standpoint      didn't do research on this, but doesnt really matter cause it just an concept           
```

It's an outline, not a product. I keep it here because half-finished hackathon
ideas are the most honest thing on any GitHub profile.

`Solidity` · `Foundry` · `Base L2` *(aspirational)*

</details>

---




## The rite-of-passage projects

Every developer has these. The ones from the start, before the ambition outran
the skill. I'm not deleting them — they're the receipts.

<details>
<summary><b>🧮 The Calculator</b></summary>

<br/>

Yes. That calculator. Add, subtract, multiply, divide. The first one where I
discovered that dividing by zero is a *whole thing* and not just my problem.
Faithfully reinventing a device that has existed since 1642.

`Python` · `humble beginnings`

</details>

<details>
<summary><b>✅ The To-Do list</b></summary>

<br/>

The legally-required second project of every programming journey. Add a task,
check it off, delete it. Mine even saved to a file — which felt, at the time,
like industrial-grade persistence engineering. It was a JSON dump. It was fine.

`Python` · `JSON` · `character development`

</details>

<details>
<summary><b>🔢 Number-guessing game</b></summary>

<br/>

The computer picks a number, you guess, it says "higher" or "lower." This is
where I met the `while` loop and we have been close ever since.

`Python` · `the while loop and I`

</details>

> None of these are impressive. All of them taught me something I still use.
> That's the actual point of a starter project — it's not the code, it's the
> first time a thing you typed *did something*.

---

## Security

I take part in a public bug bounty program. I found a vulnerability, reported it
through the proper channel, and it's currently in triage. Out of respect for
responsible-disclosure rules I'm not naming the target — the boring answer is
also the correct one.

`bug bounty` · `responsible disclosure` · `HTTP` · `JWT`

---

## How I work

```
1. pick something with real consequence — latency, uptime, money
2. understand the system before writing much code
3. measure first, optimize after
4. ship something small, then make it less embarrassing
5. write down the bug that cost an evening, so it only costs one
```

---

## Skills

Honestly levelled. **Solid** = shipped real work with it. **Working** = used it,
would reach for docs. **Learning** = started, not there yet.

**Languages**
- *Solid:* TypeScript, Bash
- *Working:* Rust, Solidity, Python, JavaScript
- *Learning:* C# *(started — drawn to it for backend work)*, SQL

**Backend & APIs**
- *Solid:* REST / HTTP, building my own endpoints
- *Working:* FastAPI, PostgreSQL / Supabase, WebSockets

**Infrastructure & ops**
- *Solid:* Linux (Debian, systemd), Docker & Compose, self-hosted L2 node
- *Working:* AWS / GCP VMs, networking (VPC, firewall, DNS)
- *Learning:* CI/CD (GitHub Actions)

**Blockchain**
- *Solid:* EVM mechanics, L2 architecture (op-node / reth), Aave / Morpho lending
- *Working:* Uniswap V3 / DEX mechanics, Foundry, Hardhat, flash loans

**Observability**
- *Solid:* Telegram alerting / alert design
- *Working:* structured logging

**CLI & workflow**
- *Solid:* curl, jq, grep/sed/awk, tmux, ssh, reading other people's code
- *Working:* Git / GitHub (branches, PRs)

---

## Learning right now

- Lower-latency Rust — in-memory caches, WS subscriptions, lock-free patterns
- Auditing my own Solidity *before* deploy, not after
- C# and a more conventional backend stack, from the ground up
- Doing bug bounty methodically instead of by vibes

## Honest gaps

- Frontend taste — my dashboards are functional, not beautiful
- Solidity assembly / Yul
- Distributed systems theory — haven't needed it yet, so I don't claim it
- CI/CD beyond the basics
- Explaining my own work without a 200-line README *(see: this README)*

---

<div align="center">
<sub>Everyone has to start somewhere, right?

Best ;)</sub>
</div>
