# Cerberus-bootkit
TripleBind Systemd Backdoor for persistence (Post-Exploitation)

# Cerberus

> Lightweight Linux post-exploitation implant with multi-channel remote access and systemd persistence.

## Overview

Cerberus is a lightweight Linux implant designed for post-exploitation environments.

Its primary objective is to provide reliable remote command execution through multiple independent communication channels while maintaining a minimal dependency footprint.

The implant is entirely Bash-based and relies only on utilities commonly available on Linux distributions.

---

## Features

- Multi-channel bind shell deployment
- systemd persistence
- Multiple fallback execution methods
- Lightweight Bash implementation
- Minimal runtime dependencies
- Payload obfuscation
- Process isolation
- Automatic service restart

---

## Architecture

```
                +----------------------+
                |  systemd Service     |
                +----------+-----------+
                           |
                           v
                +----------------------+
                |   Builder Script     |
                +----------+-----------+
                           |
        +------------------+------------------+
        |                  |                  |
        v                  v                  v
   Netcat Shell      Socat Shell       Python Shell
     :4242              :4444             :9090
```

Each communication channel operates independently.

Failure of one listener does not affect the remaining execution paths.

---

## Why multiple shells?

Different Linux distributions provide different toolsets.

Some systems may include:

- netcat
- busybox nc
- socat
- python3

Launching multiple listeners increases execution reliability across heterogeneous environments.

---

## Base64 payloads

Payloads are encoded in Base64 to:

- reduce signature-based detection
- avoid shell quoting issues
- simplify payload replacement
- improve portability

The encoding is not intended as cryptographic protection.

---

## Why the large amount of "useless" runtime code?

The runtime layer serves several purposes.

### Static analysis noise

Large quantities of harmless code reduce the signal-to-noise ratio during manual inspection.

Analysts spend additional time distinguishing operational code from placeholders.

### Signature disruption

Many antivirus engines and YARA rules rely on compact, recognizable patterns.

Padding changes file size, entropy distribution and instruction layout.

### Visual camouflage

At first glance the script resembles a legitimate runtime component instead of a short backdoor.

The majority of functions intentionally perform no operations.

---

## Masquerading

The implant intentionally adopts filenames resembling Linux and Snap runtime components.

Examples include:

```

sys-devices-platform-\*
snap-sys-devices-\*

```

These names blend into legitimate system directories and reduce suspicion during casual administrative inspection.

---

## Persistence

Persistence is implemented through a systemd service.

Characteristics:

- Restart=always
- Automatic recovery after crashes
- Root execution
- Boot-time execution
- Service-based lifecycle

The service terminates independently from spawned listeners.

Background processes remain isolated from the launcher whenever the service configuration permits it.

Some environments require additional service options (such as KillMode) depending on the target distribution and systemd behavior.

---

## Runtime Flow

```

systemd
│
└── builder.sh
│
├── disable firewall
├── launch listener #1
├── launch listener #2
├── launch listener #3
└── exit

```

---

## Operational Notes

The implant intentionally avoids:

- kernel modules
- rootkit functionality
- process injection
- memory patching
- filesystem hiding

The objective is reliability rather than stealth at the kernel level.

---

## Design Goals

- Small footprint
- Easy deployment
- Distribution compatibility
- Low dependency count
- Fast execution
- Simple persistence
- Multiple remote access paths
- Easy payload replacement

---

## Disclaimer

For authorized security research and adversary emulation only. DO NOT USE IN UNAUTHORIZED ENVIRONMENT
