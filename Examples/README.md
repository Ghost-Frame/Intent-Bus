# Intent Bus Examples (v7.6)

This directory contains **reference worker implementations** for the Intent Bus v7.6 protocol.

Workers are **active execution agents**, not passive clients.

Incorrect worker behavior breaks:
- delivery guarantees
- at-least-once execution safety
- routing consistency

---

## Core v7.6 Model

Intent Bus uses:

- `goal` → intent type (what to do)
- `namespace` → routing isolation boundary
- `capabilities` → worker eligibility filter
- `worker-id` → identity binding

Workers must:
- claim → execute → fulfill/fail
- never silently drop jobs

---

## Prerequisites

### API Key (required)

```bash
echo "your_api_key_here" > ~/.apikey
chmod 600 ~/.apikey
```

---

## Bash Workers (.sh)

**Auth Mode:** API Key Header  
**Runtime:** POSIX / BusyBox compatible  
**Dependencies:** curl, jq

### Install

#### Termux
```bash
pkg install curl jq
```

#### Linux
```bash
sudo apt install curl jq
```

---

## Available Workers

### 1. Notification Worker
- Executes mobile notifications
- Uses Termux API
- Safe UI truncation enforced

### 2. Logging Worker
- Writes structured logs to file
- Supports persistent audit trails

### 3. Integration Workers
- Discord / external webhook workers
- Extendable execution adapters

---

## Running Workers

Before running, configure:

- `GOAL`
- `NAMESPACE`
- `WORKER_ID`
- `CAPABILITIES`

```bash
chmod +x worker.sh
./worker.sh
```

---

## v7.6 Protocol Rules

### 1. Completion Requirement
Workers MUST call:
- `/fulfill/<id>` on success
- `/fail/<id>` on execution error

Silent drop = protocol violation

---

### 2. At-Least-Once Delivery
Jobs may be retried.

Workers MUST ensure:
- idempotent execution logic
- safe re-runs without side effects

---

### 3. Idle Handling
Server may respond:

```text
204 No Content
```

Optional header:

```text
Retry-After: <seconds>
```

Workers MUST respect this when present.

---

### 4. Routing Model (v7.6)

Workers are selected using:

- `goal`
- `namespace`
- `X-Worker-Capabilities`

Workers are selected by: goal, namespace, X-Worker-Capabilities, X-Worker-ID, and priority (higher values claimed first).

---

### 5. Security Model

Workers MUST treat payloads as:
- untrusted input
- execution-sensitive data

Refer to:
- `WORKER_SECURITY.md`

---

## Python Workers

**Auth Mode:** HMAC (strict validation)

```bash
pip install intent-bus
```

Python workers enforce:
- signed intents
- validated execution schema
- stricter safety guarantees than Bash workers

---

## Important Design Principle

Intent Bus workers are:
> "edge execution nodes in a distributed intent system"

They are not scripts — they are **stateful execution participants**.

---

## License

MIT
