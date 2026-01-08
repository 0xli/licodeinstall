# 0. Changes
## 0.1 two changes in scripts/installUbuntuDeps.sh
```
# gcc
gcc_version=0

check_version(){
  if [[ $(lsb_release -rs) == "18.04" ]] || [[ $(lsb_release -rs) == "20.04" ]]
  then
     gcc_version=10
  else
     gcc_version=13
  fi
}
# python
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
## 0.2 in order to run mangodb for 20.04 on 24.04
echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
sudo apt update
sudo apt install -y libssl1.1

# 1. cpp version need be 13
gcc_version=0

check_version(){
  if [[ $(lsb_release -rs) == "18.04" ]] || [[ $(lsb_release -rs) == "20.04" ]]
  then
     gcc_version=10
  else
     gcc_version=13
  fi
}

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

# 2. python error

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

# install setuptools
. .venv/bin/activate && python -m pip install -U setuptools
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

# 3. mongodb can not start
Your `mongod` is **not starting because the MongoDB binary Licode downloads (MongoDB 4.4.13 for Ubuntu 20.04)** is linked against **OpenSSL 1.1** (`libcrypto.so.1.1`). On newer Ubuntu (22.04/24.04), OpenSSL 1.1 is not installed by default (and often not available in the standard repos), so you see:

`mongod: error while loading shared libraries: libcrypto.so.1.1: cannot open shared object file`

This is a known incompatibility: Ubuntu 22.04 moved to OpenSSL 3 (`libssl3`), and legacy binaries needing `libssl1.1`/`libcrypto.so.1.1` fail. ([Ask Ubuntu][1])

Below are the clean fixes, in recommended order.

---

## Fix 1 (fastest, works): Install `libssl1.1` compatibility package

### Step 0: confirm your Ubuntu version

```bash
lsb_release -a
```

### If you are on Ubuntu 20.04

Just install it normally:

```bash
sudo apt update
sudo apt install -y libssl1.1
```

### If you are on Ubuntu 22.04 (or newer)

Ubuntu doesn’t ship `libssl1.1` by default; you can pull it from Ubuntu 20.04 “focal-security” (common workaround). ([Ask Ubuntu][1])

```bash
echo "deb http://security.ubuntu.com/ubuntu focal-security main" | sudo tee /etc/apt/sources.list.d/focal-security.list
sudo apt update
sudo apt install -y libssl1.1
```

Then retry:

```bash
/data/sfu/licode/build/libdeps/mongodb-linux-x86_64-ubuntu2004-4.4.13/bin/mongod --version
```

If `apt install libssl1.1` still fails (rare, but possible depending on your environment), use the direct `.deb` approach:

```bash
cd /tmp
wget http://archive.ubuntu.com/ubuntu/pool/main/o/openssl/libssl1.1_1.1.1f-1ubuntu2_amd64.deb
sudo dpkg -i libssl1.1_1.1.1f-1ubuntu2_amd64.deb
sudo apt -f install -y
```

This “download the deb and dpkg -i” method is widely used to resolve the exact dependency issue. ([Stack Overflow][2])

---

## Fix 2 (cleaner long-term): Don’t use Licode’s bundled MongoDB; use a supported system MongoDB

The Licode script is pinning an old MongoDB tarball (4.4.13). If you install a MongoDB version built for your Ubuntu release, you avoid the OpenSSL mismatch entirely.

Practical approach:

1. Install MongoDB from MongoDB’s official repo for your Ubuntu version (6.0/7.0 depending on release).
2. Modify Licode’s startup to point to `/usr/bin/mongod` (system binary) instead of the bundled one under `build/libdeps/...`.

This avoids mixing old OpenSSL 1.1 libs into a newer OS (which many consider risky). ([Ask Ubuntu][3])

If you want, paste the portion of Licode’s script that starts MongoDB (often `scripts/startMongo.sh` or similar), and I’ll give you an exact patch.

---

## Notes on the other messages you saw

* `mkdir: cannot create directory '../dist': File exists` is harmless (script should ideally use `mkdir -p`).
* `JSC_PARSE_ERROR ... unknown JSDoc tag "contributor"` is a warning from the JS compiler tooling and is not the blocker.
* `npm audit` vulnerabilities are not what’s preventing the install. The blocker is **MongoDB failing to run**.

---

## Quick verification checklist (after Fix 1)

Run:

```bash
ldd /data/sfu/licode/build/libdeps/mongodb-linux-x86_64-ubuntu2004-4.4.13/bin/mongod | grep -E "ssl|crypto"
```

You want to see the `libcrypto.so.1.1` / `libssl.so.1.1` lines **resolved to actual paths**, not “not found”.

---

If you tell me the output of `lsb_release -a` (20.04 vs 22.04 vs 24.04) and the exact script/file Licode uses to start MongoDB (or paste that small section), I’ll give you the most minimal patch for your setup.

[1]: https://askubuntu.com/questions/1403619/mongodb-install-fails-on-ubuntu-22-04-depends-on-libssl1-1-but-it-is-not-insta?utm_source=chatgpt.com "Depends on libssl1.1 but it is not installable"
[2]: https://stackoverflow.com/questions/73656873/unable-to-install-mongodb-in-ubuntu-22-04-mongodb-org-libssl1-1?utm_source=chatgpt.com "Unable to install mongodb in ubuntu 22.04 (mongodb-org, ..."
[3]: https://askubuntu.com/questions/1408000/unable-to-locate-package-libssl1-1?utm_source=chatgpt.com "Unable to locate package libssl1.1"

# 3. lib not found: AVUTIL-NOTFOUND when install Erizo
You are now past the Python/Conan issues. The Erizo build is failing at CMake time with:

`lib not found: AVUTIL-NOTFOUND`

`avutil` is part of **FFmpeg** (library name typically `libavutil`). CMake cannot find the FFmpeg development libraries/headers on your system, so Licode’s `test_lib()` check aborts.

### What to do (Ubuntu)

1. Install FFmpeg dev packages (plus the ones Licode commonly needs):

```bash
sudo apt update
sudo apt install -y \
  pkg-config \
  ffmpeg \
  libavutil-dev libavcodec-dev libavformat-dev libswresample-dev libswscale-dev \
  libvpx-dev
```

2. Confirm the library is visible to the linker:

```bash
pkg-config --modversion libavutil
pkg-config --libs libavutil
ldconfig -p | grep libavutil
```

If `pkg-config` can’t find `libavutil`, it means the `*-dev` packages did not install correctly or your `PKG_CONFIG_PATH` is unusual.

3. Clean and rerun Erizo build:

```bash
./scripts/installErizo.sh
```

---

## If you are on Ubuntu 24.04 and apt can’t provide the right FFmpeg dev packages

On some minimal images or constrained repos, you may have `ffmpeg` runtime but missing `libavutil-dev`. If `apt` says “unable to locate package libavutil-dev”, enable the Ubuntu “universe” repo:

```bash
sudo add-apt-repository universe
sudo apt update
sudo apt install -y libavutil-dev libavcodec-dev libavformat-dev libswresample-dev libswscale-dev
```

---

## One more likely blocker after this: OpenSSL 1.1 again (MongoDB / maybe others)

Even after Erizo compiles, Licode may still fail later if the bundled MongoDB requires `libcrypto.so.1.1` (you already hit that earlier). So after you fix FFmpeg, if you see OpenSSL 1.1 errors again, you’ll need the `libssl1.1` compatibility fix we discussed earlier.

---

### Quick diagnostic I want you to run (no guessing)

Run these and paste output:

```bash
lsb_release -a
apt-cache policy libavutil-dev
pkg-config --modversion libavutil || true
```

With those three outputs, I can tell you exactly whether you’re missing packages, missing `universe`, or dealing with a repo/version mismatch.

