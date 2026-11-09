# Ready-to-send proposals — 3 live jobs

Portfolio link for all three: **https://github.com/ExinX/rea-windows-verified**

Ranked by odds × value. Send in this order.

---

# 1. Port Legacy Windows Tool Logic to Python (UPX/Delphi) — $150

**Best odds.** Client active yesterday, 14 prior hires, only 10–15 proposals,
already interviewing. Pure REA work. Asks 3 questions — answer them in order.

---

Yes — I've dumped modified UPX by hand, and this is exactly the shape of job my
toolchain is built for.

**1. Manually dumping a modified UPX from memory with x32dbg + Scylla?**

Yes. Modified UPX usually means the original OEP and section names are gone, so
`upx -d` fails or produces garbage. The reliable path: run to the packer's tail
jump, set a hardware breakpoint on the stack pointer at the `jmp`/`pushad`-pop
sequence, catch the OEP, then dump with Scylla and fix the IAT — Scylla's
auto-search plus manual validation of each thunk. Then verify the dump by
re-running it and checking imports resolve.

**2. Tools for the Delphi binary?**

- **Ghidra 12.1.4** for the static pass — I have it running headless with REA,
  so I can decompile and cross-reference quickly
- **IDR (Interactive Delphi Reconstructor)** for the VMT/RTTI recovery, since
  Delphi's class metadata and `published` method tables survive in the binary and
  give you real method names instead of `FUN_0040a1c0`
- **x32dbg + Scylla** for the unpack
- **Python** to reimplement and diff against your sample output

The key advantage: you're giving me a known input and its exact output. That
turns this from "understand the algorithm" into "match the algorithm", which is
far more reliable — I can iterate until the bytes match, and the 5-file
acceptance test becomes a regression suite rather than a gamble.

**3. Timeline and fixed price?**

**$150 fixed, 3–4 days.** Day 1 unpack and verify the dump; days 2–3 recover the
conversion routine and reimplement in Python; day 4 validate against your sample
and the 5 acceptance files. If the algorithm turns out to be trivial (plain
XOR/Base64 with a fixed key), I'll tell you and you can pay less.

Deliverable: `generate_cert.py`, no Windows dependencies, plus a short note on
what the algorithm actually is so you're not dependent on me later.

Portfolio: https://github.com/ExinX/rea-windows-verified

---

# 2. Fix NO_LICENSE Error in Native x86 DLL — $100

**Good odds.** Only 5–10 proposals, 2 invites unanswered. They already used
Ghidra 12.1.4 — the exact version I run.

---

The detail that makes me a strong fit: your DLL was analyzed with **Ghidra
12.1.4**, which is the exact version I have running headless with REA. I can pick
up your existing decompiled C and the VS2022 Win32 reconstruction rather than
starting over.

**How I'd approach `License open failed: NO_LICENSE`:**

That error string is the fastest lead — I'd find it in the binary, get its
cross-references, and walk backwards to the branch that produces it. In a licence
path there are usually only a handful of failure exits, and each maps to a
distinct cause:

1. **Credential Manager** — `CredRead`/`CredEnumerate` returning
   `ERROR_NOT_FOUND` because the credential was written under a different
   target name, a different user context, or a different bitness. A 32-bit DLL
   reading a credential written by a 64-bit process is a classic.
2. **Registry / machine binding** — a licence keyed to a machine GUID or volume
   serial that changed.
3. **File or environment dependency** — an encrypted dataset path resolved
   relative to the working directory, so it works when launched from Explorer
   and fails when launched from a service or a different cwd.
4. **Session/impersonation context** — the process running as a service or
   elevated when the licence was provisioned unelevated.

The `NO_LICENSE` vs `LICENSE_EXPIRED` vs `LICENSE_CORRUPT` distinction usually
tells you which of these it is, so I'd start by confirming exactly which branch
fires and under what conditions.

**Deliverable:** the root cause written down, a working fix, and confirmation
that ECU selection and encrypted dataset compatibility still work. I'd test
against the original DLL side by side so we know the fix didn't change behaviour
elsewhere.

**$100 fixed, 2–3 days**, assuming the reconstruction project builds. If the
cause turns out to be environmental rather than a code defect, I'll tell you
before charging — sometimes the answer is "your credential is under the wrong
account" and that's worth knowing.

Portfolio: https://github.com/ExinX/rea-windows-verified

---

# 3. Proprietary Software Source Recovery — $20,000

**Lowest odds** (20–50 proposals, client new since Nov 5) but the biggest number.
Worth one shot.

---

I'd approach this as a specification recovery first, code second — because
"re-create the original codebase faithfully" is where these projects usually go
wrong.

**The honest constraint:** decompiled output is compiler-shaped, not
human-shaped. Ghidra gives you correct control flow and correct data structures,
but the variable names, the original class boundaries, and the intent are gone.
Anyone who promises a byte-faithful source reconstruction is guessing. What you
can get reliably is: exact behaviour, exact data layouts, exact algorithms, and
a clean codebase that mirrors them and that your team can actually maintain.

**My approach:**

1. **Inventory** — map modules, entry points, imports, and the string table.
   Establish what the program actually does before touching code.
2. **Behavioural spec** — for each module, write down inputs, outputs, side
   effects, and error paths, with the address and function name as evidence for
   every claim. This is the deliverable that survives even if the code doesn't.
3. **Data structures** — recover struct layouts from access patterns and
   cross-reference them against the decompiler's types. This is the part that
   determines whether the rebuild is maintainable.
4. **Reimplementation** — clean code in a modern language, written against the
   spec, not transliterated from decompiler output.
5. **Differential testing** — run original and rebuild side by side on the same
   inputs and diff the outputs. This is the only real proof of fidelity.

**Toolchain:** Ghidra 12.1.4 headless (I have it wired into an agent via REA, so
I can query the binary in plain English and get evidence back — function names,
addresses, call graphs — rather than clicking through a GUI), plus IDA if the
binary has structures Ghidra handles poorly. I also have jadx, apktool, WABT and
packet capture available if any component turns out to be managed, Android, WASM
or network-facing.

**Timeline:** I'd want to see the binary before committing to a number, because
"proprietary desktop application" ranges from a 2 MB utility to a 200 MB suite.
Rough shape: 1 week inventory + spec, then 2–4 weeks reimplementation depending
on module count, with progress updates at each milestone as you asked.

**Suggested start:** a paid 1-week discovery ($1,500) that delivers the module
inventory and behavioural spec. You then own a document that's useful regardless
of who finishes the build — and you can decide whether to continue with me on
evidence rather than a promise.

Portfolio: https://github.com/ExinX/rea-windows-verified

---

## Sending checklist

- [ ] Log into Upwork (I can't — no credentials, and I won't touch them)
- [ ] Profile headline: "REA / MCP agent engineer — reverse engineering with
      Ghidra, jadx, apktool"
- [ ] Add portfolio item: repo URL + a screenshot of `rea-audit.mjs` output
- [ ] Send #1 first (best odds), then #2, then #3
- [ ] Connects: ~6–16 each ($0.90–$2.40); #3 may cost more
- [ ] Apply to #1 within hours — it's the freshest and most winnable

## What I could not do

- Log into Upwork (needs your credentials)
- Verify whether you already have an Upwork account
- Send the proposals — Upwork has no public API for submissions

Everything else is done: repo is public, proposals are written, links are live.
