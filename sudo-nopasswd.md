Understood. Your current `sudo -l` output shows **you have normal sudo**, but **no `NOPASSWD` rule is being applied** (otherwise the “may run… (ALL : ALL) ALL” line would typically reflect `NOPASSWD:` for the matching rule, and `sudo -l` / `sudo -n` tests would behave differently).

Here is the fastest, reliable way to fix it and verify.

---

## 1) Add an explicit NOPASSWD rule via `/etc/sudoers.d/` (recommended)

Run:

```bash
sudo visudo -f /etc/sudoers.d/90-wli-nopasswd
```

Put **exactly** this line in the file:

```text
wli ALL=(ALL:ALL) NOPASSWD: ALL
```

Save and exit.

Then fix permissions (sudo will ignore the file if perms are wrong):

```bash
sudo chown root:root /etc/sudoers.d/90-wli-nopasswd
sudo chmod 440 /etc/sudoers.d/90-wli-nopasswd
```

Validate sudoers syntax:

```bash
sudo visudo -c
```

---

## 2) Test correctly (use `-n` to avoid password prompts during testing)

This should return your sudo privileges **without prompting**:

```bash
sudo -n -l
```

And this should succeed without a password prompt:

```bash
sudo -n whoami
```

Expected output:

```text
root
```

If `sudo -n whoami` fails, it will print an error like “a password is required” — which means the `NOPASSWD` rule still isn’t matching.

---

## 3) If it still doesn’t work: identify what’s overriding or preventing the rule

Run these diagnostics:

### A) Confirm the file is being included

Ubuntu normally includes `/etc/sudoers.d/*` via `/etc/sudoers`. Check:

```bash
sudo grep -nE '^(#includedir|#include)\b' /etc/sudoers
```

You should see something like:

```text
#includedir /etc/sudoers.d
```

If you do **not** see it, your drop-in file will never apply.

### B) Confirm the rule is present and clean

```bash
sudo cat -n /etc/sudoers.d/90-wli-nopasswd
```

Make sure there are **no typos**, no weird characters, and the line is exactly the one above.

### C) Check for a later conflicting rule (order matters)

If you previously added another rule for `wli` elsewhere, a later rule can change behavior. Search all sudoers sources:

```bash
sudo grep -RIn --color=always '^\s*wli\s\+|^\s*%sudo\s\+' /etc/sudoers /etc/sudoers.d/*
```

If there are multiple `wli` entries, ensure the `NOPASSWD` one is **last** (or remove conflicting ones).

---

## 4) Common “gotcha” that matches your symptom

* You may have edited the wrong file, or the file permissions are not `0440`, so sudo ignores it silently.
* Or `/etc/sudoers` on that machine does not include `/etc/sudoers.d`.

---

If you paste the output of these three commands, I can tell you exactly what is wrong in one pass:

```bash
sudo grep -nE '^(#includedir|#include)\b' /etc/sudoers
ls -l /etc/sudoers.d/90-wli-nopasswd
sudo -n -l
```

Understood. If `sudo -n whoami` still says “a password is required,” then **your `NOPASSWD` rule is not being loaded or not being applied**. At this point the two most common root causes are:

1. `/etc/sudoers` is **not including** `/etc/sudoers.d`, or
2. your drop-in file is being **ignored** (wrong owner/permissions, bad filename, or invalid syntax/encoding).

Here is the shortest path to isolate and fix.

---

## 1) Confirm `/etc/sudoers.d` is actually included

Run:

```bash
sudo grep -nE '^(#includedir|#include)\b' /etc/sudoers
```

### Expected

You should see something like:

```text
#includedir /etc/sudoers.d
```

### If you see nothing

Then your `/etc/sudoers.d/wli` file will never be read.

**Fix (safe):**

```bash
sudo visudo
```

Add (or ensure you have) this line near the end:

```text
#includedir /etc/sudoers.d
```

Save/exit, then continue to Step 2.

---

## 2) Ensure the drop-in file is not being ignored

Run:

```bash
sudo ls -l /etc/sudoers.d/wli
sudo cat -n /etc/sudoers.d/wli
```

It must be:

* Owned by `root:root`
* Mode `0440` (i.e., `-r--r-----`)
* Contain exactly one valid line, e.g.:

```text
wli ALL=(ALL:ALL) NOPASSWD: ALL
```

If needed, force-correct ownership/permissions:

```bash
sudo chown root:root /etc/sudoers.d/wli
sudo chmod 440 /etc/sudoers.d/wli
```

---

## 3) Validate the sudoers configuration (this is decisive)

Run:

```bash
sudo visudo -c
```

* If it reports **any error**, sudo may ignore parts of the configuration.
* If it reports OK, proceed.

---

## 4) Re-test passwordless sudo (non-interactive)

```bash
sudo -n whoami
sudo -n -l
```

If configured correctly, `sudo -n whoami` should output:

```text
root
```

---

## 5) If it *still* fails, check for “silent ignore” due to file name/format

### A) Ensure no invalid files in `/etc/sudoers.d/`

Sudo ignores the entire directory if it finds problematic entries (e.g., wrong perms, editor backup files).

```bash
sudo ls -la /etc/sudoers.d
```

Look for files like `wli~`, `.swp`, `README` with wrong perms, etc. Anything not `root:root` and `0440` can cause issues.

### B) Fix by isolating to a clean file name

Create a new clean file:

```bash
sudo visudo -f /etc/sudoers.d/90-wli-nopasswd
```

Add:

```text
wli ALL=(ALL:ALL) NOPASSWD: ALL
```

Then:

```bash
sudo chown root:root /etc/sudoers.d/90-wli-nopasswd
sudo chmod 440 /etc/sudoers.d/90-wli-nopasswd
```

---

## What I need from you to pinpoint the exact cause (copy/paste output)

Run these three commands and paste the output:

```bash
sudo grep -nE '^(#includedir|#include)\b' /etc/sudoers
sudo ls -l /etc/sudoers.d/wli
sudo visudo -c
```

With those three, I can tell you precisely whether it’s an include problem, an ignored file problem, or a global sudoers parsing issue.
