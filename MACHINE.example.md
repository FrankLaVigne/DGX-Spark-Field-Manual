# Machine Inventory

Copy this to `MACHINE.md` and fill in your own values. **`MACHINE.md` is
gitignored** — it is the one place real hostnames, addresses, and paths belong,
so that everything else in the repo stays machine-agnostic.

Keeping it also gives you a drift log: when something breaks after an update,
the first useful question is "what changed since this was last known good?"

## Identity

| Property | Value |
|---|---|
| Hostname | |
| LAN address | |
| Wired interface | |
| Reachable via | (SSH / RDP / Tailscale / …) |

## Hardware

| Property | Value | How to check |
|---|---|---|
| SoC | | `nvidia-smi --query-gpu=name --format=csv` |
| CPU | | `lscpu` |
| Memory | | `free -h` |
| Storage | | `df -h /` |

## Software versions

| Component | Version | How to check |
|---|---|---|
| OS | | `cat /etc/os-release` |
| Kernel | | `uname -r` |
| NVIDIA driver | | `nvidia-smi --query-gpu=driver_version --format=csv` |
| CUDA | | `nvcc --version` |
| Docker | | `docker --version` |

## Enabled services

What starts on boot, and why. See `runbooks/90-keeping-it-running.md`.

| Service | Purpose |
|---|---|

## Drift log

Append-only. Date, what changed, what broke or improved.

| Date | Change | Effect |
|---|---|---|
