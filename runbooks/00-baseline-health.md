# 00 — Baseline health

**Goal:** establish what a healthy, idle Spark looks like on *your* machine, and
write it down. Ten minutes, once.

Every other runbook assumes you have done this. When runbook 40 says "if the
model fails to load, check GPU memory," it is useless unless you already know
what your GPU memory reads when nothing is wrong.

There is a second reason, less obvious and more important. Most of what goes
wrong on this machine does not announce itself. Nothing crashes. A service quietly
stops being enabled, a disk quietly fills, a thermal ceiling quietly costs you 20%
throughput. You cannot detect drift without a baseline, and the baseline has to be
captured while things work. That is now.

---

## The check

Run these six. Read the next section before worrying about anything.

```bash
uname -a                  # kernel and architecture
nvidia-smi                # GPU, driver, CUDA, what is using the GPU
free -h                   # memory
df -h /                   # disk
systemctl --failed        # anything broken
uptime                    # how long since the last reboot, and current load
```

None of these change anything. All are safe to run at any time, including while
a training job is running.

---

## Reading the output

### `uname -a`

**Status: VERIFIED** — 2026-08-24, driver 580.159.03, CUDA 13.0.

```
Linux <hostname> 6.17.0-1026-nvidia #26-Ubuntu SMP PREEMPT_DYNAMIC ... aarch64 aarch64 aarch64 GNU/Linux
```

Two things matter here.

**`aarch64`** is the one that will bite you. This machine is ARM64, not x86-64.
A large fraction of the Python and Docker ecosystem publishes x86-64 builds only.
When a `pip install` fails with "no matching distribution found" or a container
dies instantly with `exec format error`, this is why — not a broken install.
Runbook 70 deals with this properly. For now, just know that `aarch64` is the
root cause of an entire category of failure you have not met yet.

**`-nvidia`** in the kernel version means you are on NVIDIA's kernel, not
Ubuntu's generic one. Do not "upgrade" to a generic kernel to fix something. The
GPU driver is built against this one.

> **Substrate note.** `uname` prints kernel identity. `-a` means "all fields."
> The three repeated `aarch64`s are machine hardware, processor, and OS platform —
> they agree here, and if they ever disagree you have a much stranger problem than
> this manual covers.

### `nvidia-smi`

**Status: VERIFIED** — 2026-08-24.

```
+-----------------------------------------------------------------------------------------+
| NVIDIA-SMI 580.159.03             Driver Version: 580.159.03     CUDA Version: 13.0     |
+-----------------------------------------+------------------------+----------------------+
| GPU  Name                 Persistence-M | Bus-Id          Disp.A | Volatile Uncorr. ECC |
| Fan  Temp   Perf          Pwr:Usage/Cap |           Memory-Usage | GPU-Util  Compute M. |
|=========================================+========================+======================|
|   0  NVIDIA GB10                    On  |   <gpu-bus-id>    Off |                  N/A |
| N/A   45C    P8              3W /  N/A  | Not Supported          |      0%      Default |
+-----------------------------------------+------------------------+----------------------+
```

**Four fields on this screen look like faults and are not.**

| Reads | Status |
|---|---|
| `Memory-Usage: Not Supported` | Expected. There is no discrete VRAM to report. |
| `Pwr: 3W / N/A` | Expected. No cap is exposed; the 3W draw is real idle. |
| `Fan: N/A` | Expected. No fan telemetry is exposed here. |
| `Volatile Uncorr. ECC: N/A` | Expected. No ECC counters are exposed here. |

What is **VERIFIED** is that all four read this way on a healthy machine, so none
of them is a symptom. *Why* NVIDIA exposes no fan or ECC telemetry on this part is
not something this manual has established — it is left unexplained rather than
guessed at.

`Memory-Usage: Not Supported` is the single most alarming line a new Spark owner
sees, and it is the most important thing on the page to understand. The GB10 uses
**unified memory** — one 121 GiB pool shared by CPU and GPU. There is no separate
VRAM figure because there is no separate VRAM. Every habit you have built around
"will this fit in 24 GB" needs re-forming, mostly in your favor. Runbook 10 is
entirely about this.

The practical consequence right now: **the GPU-level memory total is unavailable,
but per-process GPU memory is not.** This still works:

```bash
nvidia-smi --query-compute-apps=pid,process_name,used_memory --format=csv
```

**Status: VERIFIED** — 2026-08-24, with a model resident:

```
pid, process_name, used_gpu_memory [MiB]
152615, /usr/local/bin/ollama, 5303 MiB
```

So: use `free -h` for the whole-machine picture and `--query-compute-apps` for
per-process attribution. Runbook 10 explains why these two disagree, and why
that is not an error.

`P8` is the idle power state (**VERIFIED** idle). NVIDIA performance states run
`P0` (maximum) through `P8`, so a loaded GPU should not sit at `P8` — seeing `P8`
while you believe a job is running is a strong hint the job is not on the GPU at
all. The under-load behaviour of this specific part is **UNVERIFIED**; it is
confirmed in runbook 20, which puts the machine under real load.

The `Processes` block is the useful part of this command day to day. Idle, it
shows Xorg holding a couple hundred MiB — normal on a machine with a desktop:

```
|    0   N/A  N/A            2432      G   /usr/lib/xorg/Xorg                      223MiB |
```

An empty process list while you expect inference to be running is a real finding.

### `free -h`

**Status: VERIFIED** — 2026-08-24, idle.

```
               total        used        free      shared  buff/cache   available
Mem:           121Gi       3.8Gi       113Gi        32Mi       5.3Gi       117Gi
Swap:           15Gi          0B        15Gi
```

Because memory is unified, **this is your GPU memory readout.** Watch
`available`, not `free`.

> **Substrate note — `free` vs `available`.** `free` is memory nobody has touched.
> `available` is what a new process could actually get, including cache the kernel
> will drop on demand. `buff/cache` is not lost memory; it is disk cache that gets
> evicted the moment something needs the space. On a machine that has just read a
> 40 GB model off disk, `free` looks frightening and `available` is fine. Read
> `available`.

Swap at `0B` used is what you want. Swap in active use on this machine means
something has overcommitted the unified pool, and performance will have already
collapsed — you will notice before this number does.

### `df -h /`

**Status: VERIFIED** — 2026-08-24.

```
Filesystem      Size  Used Avail Use% Mounted on
/dev/nvme0n1p2  3.7T  1.6T  2.0T  45% /
```

Model weights are large and accumulate invisibly. This machine is at 45% with no
deliberate effort, and the two biggest consumers were both in places their owner
had forgotten about. Runbook 30 covers where the space actually goes; it is not
where you think.

Check this before any large download, not after.

### `systemctl --failed`

**Status: VERIFIED** — 2026-08-24.

```
  UNIT LOAD ACTIVE SUB DESCRIPTION
0 loaded units listed.
```

Zero is the answer you want. This is the fastest whole-system health check on
Linux and it has no Windows equivalent that is quite this direct.

> **Substrate note — systemd.** `systemd` is the thing that starts and supervises
> background programs at boot. If you are coming from Windows: it is Services,
> plus Task Scheduler, plus a fair bit of Event Viewer, in one tool. `systemctl`
> is how you talk to it. A "unit" is one managed thing — usually a service.
>
> Two words that look synonymous and are not:
> - **active** — running right now.
> - **enabled** — will start on its own at boot.
>
> A service can be active but not enabled. It works today and is gone after the
> next reboot, with no error anywhere. This is the most common way a Spark
> silently stops doing what you set it up to do, and it is the subject of
> runbook 90.

Check the two that matter:

```bash
systemctl is-active ollama   # running now?
systemctl is-enabled ollama  # survives reboot?
```

**Status: VERIFIED** — both return `active` / `enabled` on the reference machine.

### `uptime`

**Status: VERIFIED** — 2026-08-24.

```
 16:23:55 up 11 days, 22:11,  2 users,  load average: 0.18, 0.11, 0.05
```

The three load figures are 1-, 5-, and 15-minute averages of how many processes
wanted CPU. This box has 20 cores, so 0.18 is essentially nothing.

Worth sitting with: **eleven days of uptime at near-zero load.** That is a
correctly functioning machine doing nothing at all, and it is the exact condition
this manual exists to fix. Healthy and idle are not the same thing.

---

## Capture the baseline

Copy the template and fill it in from what you just ran:

```bash
cp MACHINE.example.md MACHINE.md
```

`MACHINE.md` is gitignored — it holds your hostname, addresses, and paths, and
never leaves your machine. `MACHINE.example.md` is the committed template.

Record at minimum: kernel, driver, CUDA, total memory, disk used, enabled
services, and today's date. Two lines of effort now; the only way to answer
"did something change?" later.

Re-run this check and update the file after any driver update, any distribution
upgrade, and any time something starts behaving differently. The drift log at the
bottom of the template is where you note what changed and when.

---

## When it looks wrong

| Symptom | First thing to check |
|---|---|
| `nvidia-smi` — command not found | `$PATH`, or driver not installed. → 00 troubleshooting |
| `nvidia-smi` hangs or reports no devices | Driver/kernel mismatch after an update. → 00 troubleshooting |
| `Memory-Usage: Not Supported` | Not a fault. See above, and runbook 10. |
| Disk above ~85% | Runbook 30 before anything else. |
| `systemctl --failed` lists units | Read `journalctl -u <unit> -n 50` first. |
| Service works, gone after reboot | `enabled` vs `active`. → runbook 90. |

Symptom-indexed detail lives in `troubleshooting/`. This table is the triage
step, not the fix.

---

## Status

| Claim | Status | Verified against |
|---|---|---|
| All six commands, outputs as shown | **VERIFIED** | 2026-08-24 · driver 580.159.03 · CUDA 13.0 · Ubuntu 24.04.4 · kernel 6.17.0-1026-nvidia |
| `Memory-Usage: Not Supported` is expected on GB10 | **VERIFIED** | same |
| Ollama `active` + `enabled` | **VERIFIED** | same |
| Failure-mode table entries | **UNVERIFIED** | Not reproduced on this machine — these are anticipated, not observed. Each is confirmed or removed as it is encountered. |

Outputs are sanitized per the placeholder convention in `CONTRIBUTING.md`:
hostname, GPU bus ID, and user paths are replaced. Nothing else is altered.
