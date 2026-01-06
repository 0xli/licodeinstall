1. cpp version

2. python error

This is **PEP 668** behavior: on newer Ubuntu/Python (commonly Ubuntu 24.04 + Python 3.12), the system Python environment is marked **“externally managed”**, so `sudo pip3 install ...` is blocked to prevent breaking OS-managed Python packages.

Your script fails specifically at:

```bash
sudo pip3 install conan==1.62.0
sudo pip3 install cpplint==1.5.4
```

Below are three practical fixes. I recommend **Option A** (virtualenv) for Licode builds.

---

## Option A (recommended): Install conan + cpplint into a project venv (no system breakage)

Run this once from the Licode repo root:

```bash
sudo apt update
sudo apt install -y python3-venv python3-full

python3 -m venv .venv
source .venv/bin/activate

pip install --upgrade pip
pip install conan==1.62.0 cpplint==1.5.4

# verify
conan --version
cpplint --version
```

Then **rerun the Licode dependency script**, but you must ensure it **does not use `sudo pip3`** anymore. Patch it like this:

### Patch `scripts/installUbuntuDeps.sh`

Replace:

```bash
install_conan(){
  sudo pip3 install conan==1.62.0
}

install_cpplint(){
   sudo pip3 install cpplint==1.5.4
}
```

With:

```bash
install_conan(){
  python3 -m venv .venv || true
  source .venv/bin/activate
  pip install --upgrade pip
  pip install conan==1.62.0
}

install_cpplint(){
  source .venv/bin/activate
  pip install cpplint==1.5.4
}
```

Important: run the script from the repo root so `.venv` ends up in the expected place.

---

## Option B: Use `pipx` (clean, also safe)

If you just want those tools installed as standalone commands:

```bash
sudo apt update
sudo apt install -y pipx
pipx ensurepath

pipx install conan==1.62.0
pipx install cpplint==1.5.4
```

Then modify the script to **skip** the `sudo pip3 install ...` steps (or comment them out), since the commands will already exist in PATH.

---

## Option C (fastest, least safe): Override with `--break-system-packages`

If you want minimal changes and accept the risk:

```bash
sudo pip3 install --break-system-packages conan==1.62.0
sudo pip3 install --break-system-packages cpplint==1.5.4
```

You can patch the script to:

```bash
sudo pip3 install --break-system-packages conan==1.62.0
sudo pip3 install --break-system-packages cpplint==1.5.4
```

I do **not** recommend this on servers you want to keep stable long-term.

---

## What I would do for Licode on Ubuntu today

* Use **Option A (venv)** so Licode’s build tooling is reproducible and doesn’t fight the OS.
* Keep `sudo apt install ...` for system libs, but keep Python tooling inside `.venv`.

If you paste the top part of `scripts/installUbuntuDeps.sh` (especially any `set -e`, working directory assumptions, or helper functions), I can give you a precise patch that won’t break the rest of the script flow.
