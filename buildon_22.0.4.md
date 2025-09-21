Got you. Let’s keep `installErizo.sh` and simply stop it from building the `tests` target.

# Minimal patch (only skip tests; keep everything else the same)

### 1) Change `install_erizo*()` to build only `erizo`

`installErizo.sh` currently calls:

```
./buildProject.sh $FAST_MAKE
```

inside both `install_erizo()` and `install_erizo_release()`. Replace those with:

```
./buildProject.sh erizo $FAST_MAKE
```

This makes the build step compile only the `erizo` target, so the `tests` executable (where `pthread_yield` blows up) is never linked.&#x20;

One-liner to patch both spots:

```bash
cd ~/licode
cp scripts/installErizo.sh scripts/installErizo.sh.bak
sed -i 's#\./buildProject\.sh \$FAST_MAKE#./buildProject.sh erizo $FAST_MAKE#g' scripts/installErizo.sh
```

### 2) (Optional) Make `buildProject.sh` default to the `erizo` target

`buildProject.sh` iterates each build dir and runs `make $*`. If you ever call it without a target, it will try to build everything (including tests). Force it to always build only `erizo` by default: change `make $*` → `make erizo $*`.&#x20;

One-liner:

```bash
cd ~/licode/erizo
cp buildProject.sh buildProject.sh.bak
sed -i 's/make \$*/make erizo $*/' buildProject.sh
```

### 3) Run your usual installer

```bash
cd ~/licode
./scripts/installErizo.sh
```

That’s it—no tests will be built, so the `pthread_yield` link error won’t appear. If you later *do* want to run the tests, either revert the backups or just export the flags before building to fix the link:

```bash
export CXXFLAGS="$CXXFLAGS -pthread"
export LDFLAGS="$LDFLAGS -pthread"
./scripts/installErizo.sh -t
```

If anything still tries to build `tests`, paste the last \~20 lines of the script around the build call and I’ll tailor the exact sed for your copy.

https://chatgpt.com/s/t_68cfc84a200c8191abf1aa86b1e539f3
