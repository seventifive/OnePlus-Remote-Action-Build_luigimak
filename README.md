# OnePlus-Remote-Action-Build
Build OnePlus latest kernel for your device with KSUN/WildKSU/KSU + SusFS anytime without knowing or self-patching. Just clone, edit config as per requirement and build.

## How to Start?
1. Click on Fork button.
![Click on Fork button.](images/img1.png)

2. Clone all branches!
![Untick copy the main branch only](images/img2.png)

3. Voila. All Done here :-)
4. Then clone the repo to your local system. [Optional if you want fine control]

## How to edit configs?
1. Change Branch from main to devices.
<p align="center">
  <img src="images/change-branch.png" alt="Click on main button next to branches and click devices"/>
</p>

2. Edit the config for your device using web edit option.

Note: Not gonna explain for cloned user as you already have some idea on how to use git or use AI to learn.

## How to Build Kernel now that everything is setup?
1. Click on Actions Tab.
2. Accept the Enable Workflow Caution message from GitHub.
![Click 'I understand my workflows, go ahead and Enable them' button](images/accept-workflow.png)

3. Click "Build OnePlus Kernel" under All Workflows
<p align="center">
  <img src="images/workflow.png" alt="Click 'Build OnePlus Kernel'"/>
</p>

4. Click Run Workflow to get build options. Click Second Run Workflow button to build.
![Click Run Workflow to get build options. Click Second Run Workflow button to build](images/run-workflow.png)

## How and What to configure in Build Options?
1. **Branch**: Always keep it main.
2. **Select the OnePlus kernels to build**: Select your device and OOS/COS version. Eg: OP11_oos16
3. **Enter KernelSU build json**: This input needs a valid single line json. Eg: `[{"type":"ksun","hash":"dev"}]`
	a. **type**: Valid options are :- `ksun` and `ksu`
	b. **hash**: This is the branch name or commit hash of selected KSU variant you want to build. Some valid values are:
		i.   `[{"type":"ksun","hash":"dev"}]`
		ii.  `[{"type":"ksu","hash":"main"}]`
		iii. `[{"type":"ksun","hash":"8de61e2305d02b927f32c41b91c05437c8a92425"}]`
4. **Compiler optimization level**: Valid options are `O2` and `O3`. Recomended to always use `O2`.
5. **Clean build (no ccache)**: Enabling this builds without previous caches. First build is always clean and take a long time. Subsequent builds are faster when cache is used. If a cache is not used for a long time, the cache is invalidated and will be considered a fresh build.

Below Options all have same properties:

6. **Enter SusFS Branch or commit hash for android12-5.10**: SusFS hash for android12-5.10

7. **Enter SusFS Branch or commit hash for android13-5.15**: SusFS hash for android13-5.15

8. **Enter SusFS Branch or commit hash for android14-6.1**: SusFS hash for android14-6.1

9. **Enter SusFS Branch or commit hash for android15-6.6**: SusFS hash for android15-6.6

10.**Enter SusFS Branch or commit hash for android16-6.12**: SusFS hash for android16-6.12

	a. Leave blank for latest.
	b. Take hash from SusFS repo and correct branch based on your kernel.
	c. You need to fill hash only for your kernel version if you need a particular version. 

## FAQs
### 1. Why are the builds failing?
Builds can fail for various reasons:
1. Fix Patches developed for KSUN + SusFS might be outdated.
2. SusFS might not be in sync with latest KSUN or KSU.
3. KSUN might not be in sync with latest SusFS.
4. Issue in remote build logic during maintainance or update.

### 2. What should I do if builds are failing?
1. Wait for Fix Patches to get updated if KSUN + SusFS build is failing.
2. Wait for All Projects [SusFS, KSUN, KSU, Remote build(WildKernels OnePlus Repo)] to get update.
3. Check if both branches are in sync with upstream action build.
4. Build with old working hashes. [You can ask for build hashes in WildKernels TG group]

### 3. How often is fix patches updated?
1. Depending on how important the changes are:
	1. 1-3 days if new detection fixes are added.
	2. If unimportant and standard fixes, then 1 week to 2 weeks.

### 4. I'm a pro dev. I can build my own patches. Why should I clone this repo and use this?
You can clone my OnePlus build repo or Wild Kernels repo and do the necessary changes there and then point remote action build to your own fork so that you can build for your changes and your devices only. This repo should help you build and test your patches before PRs.

### 5. How do I know if my device is supported?
Check the devices branch for list of devices and supported OOS/COS version.

### 6. It says OOS everywhere. Does that mean COS is not supported?
Nope. COS is supported. Its just a naming scheme used. COS and OOS both use same kernel. If there is any difference, it is denoted using model numbers like `PJZ` or `CPH`.

### 7. I have Realme Device. Can I use the action build to build kernels?
Check the compatibity list at Wild Kernels. You can build and test the kernel yourself and report us at WildKernels if its working for you, so that we can update the list.

---

# 🖥️ Self-Hosted Runner Setup (Ubuntu 24.04 / WSL2)

This section covers the complete setup required to run the CI pipeline on self-hosted GitHub Actions runners.

> **Tested environment:** Ubuntu 24.04 on WSL2 (Windows 11), 4 concurrent runners, ext4 filesystem, 1 TB+ disk.

---

## 1. System Packages

```bash
sudo apt-get update
sudo DEBIAN_FRONTEND=noninteractive apt-get install -y --no-install-recommends \
  git curl ca-certificates build-essential clang lld flex bison \
  libelf-dev libssl-dev libncurses-dev zlib1g-dev liblz4-tool \
  libxml2-utils rsync unzip dwarves file python3 ccache jq bc \
  dos2unix kmod libdw-dev elfutils gcc-aarch64-linux-gnu
```

---

## 2. Node.js >= 20

Required by `actions/checkout@v4` and other actions:

```bash
curl -fsSL https://deb.nodesource.com/setup_20.x | sudo -E bash -
sudo apt-get install -y nodejs
node --version   # must be v20.x or higher
```

---

## 3. GitHub CLI

Required by the `trigger-release` job:

```bash
(type -p wget >/dev/null || sudo apt-get install wget -y) && \
sudo mkdir -p -m 755 /etc/apt/keyrings && \
wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null && \
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg && \
echo "deb [arch=$(dpkg --print-architecture) \
  signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] \
  https://cli.github.com/packages stable main" \
  | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null && \
sudo apt-get update && sudo apt-get install gh -y

gh auth login
```

---

## 4. sudo Without Password

The workflow uses `sudo` in non-interactive steps:

```bash
echo "$USER ALL=(ALL) NOPASSWD:ALL" | sudo tee /etc/sudoers.d/$USER
```

---

## 5. bindgen (for Rust kernel builds)

Required for devices with `rust_build: true` (e.g. OP15, OP15r, ACE6T).

> The standard `curl ... | sh` method fails on WSL2 because the rustup installer writes to `/tmp` before the `chmod` step. Downloading directly to `$HOME` avoids this issue.

```bash
curl --proto '=https' --tlsv1.2 -sSf \
  "https://static.rust-lang.org/rustup/dist/x86_64-unknown-linux-gnu/rustup-init" \
  -o "$HOME/rustup-init"
chmod +x "$HOME/rustup-init"
"$HOME/rustup-init" -y --default-toolchain none --no-modify-path
rm -f "$HOME/rustup-init"

source "$HOME/.cargo/env"
cargo install bindgen-cli --version 0.69.5
bindgen --version   # must print: bindgen 0.69.5

# Add cargo to shell profile
echo 'source "$HOME/.cargo/env"' >> "$HOME/.bashrc"
```

---

## 6. ccache

> **Critical:** `hard_link = false` is required. With `hard_link = true`, ccache creates read-only hard links for `.o` files, causing `objcopy` to fail during kernel builds on ext4.

```bash
mkdir -p ~/.ccache
cat > ~/.ccache/ccache.conf << EOF
max_size = 40G
compression = true
compression_level = 1
direct_mode = true
hash_dir = false
file_clone = false
inode_cache = true
umask = 002
sloppiness = file_macro,time_macros,include_file_mtime,include_file_ctime,pch_defines,system_headers,locale
cache_dir = $HOME/.ccache
depend_mode = false
hard_link = false
stats_log = $HOME/.ccache/stats.log
compiler_check = content
EOF

ccache -p | grep -E "max_size|hard_link|compression"   # verify
```

---

## 7. git Configuration

```bash
git config --global user.name "your-github-username"
git config --global user.email "your@email.com"

# Required on WSL2 to prevent "dubious ownership" errors
# when the runner agent creates _work directories
git config --global safe.directory '*'
```

---

## 8. Runner Registration

Create the runner directory and register it with GitHub.

The example below sets up a **single runner**. To add more runners for parallel builds, repeat the steps in a new directory (e.g. `$HOME/self-host/runner2/`, `runner3/`, etc.) using a fresh registration token each time.

```bash
RUNNER_DIR="$HOME/self-host/runner"
REPO_URL="https://github.com/YOUR_USERNAME/YOUR_REPO"

mkdir -p "$RUNNER_DIR"
cd "$RUNNER_DIR"

# Check https://github.com/actions/runner/releases for the latest version
curl -o actions-runner-linux-x64.tar.gz -L \
  "https://github.com/actions/runner/releases/download/v2.331.0/actions-runner-linux-x64-2.331.0.tar.gz"
tar xzf ./actions-runner-linux-x64.tar.gz
rm actions-runner-linux-x64.tar.gz

# Get token from: Repository -> Settings -> Actions -> Runners -> New self-hosted runner
./config.sh --url "$REPO_URL" \
            --token YOUR_REGISTRATION_TOKEN \
            --name "runner" \
            --labels "self-hosted,ubuntu-24.04" \
            --work "_work" \
            --unattended
```

---

## 9. Runner PATH (`.env`)

The runner needs `bindgen` and `cargo` in its PATH. The `.env` file is read only at `run.sh` startup — restart the runner after any change.

```bash
echo "PATH=$HOME/.cargo/bin:$PATH" >> "$HOME/self-host/runner/.env"

# Verify
grep PATH "$HOME/self-host/runner/.env"
```

---

## 10. Starting the Runner

Start the runner in a terminal. The `.env` file is read only at startup.

```bash
wsl -d Ubuntu-24.04 bash -c "/home/YOUR_USERNAME/self-host/runner/run.sh"
```

The runner prints `Listening for Jobs` when ready.

---

## Directory Structure

```
$HOME/
├── .ccache/                        # ccache storage (up to 40 GB)
│   └── ccache.conf
├── .cargo/
│   └── bin/
│       └── bindgen                 # bindgen 0.69.5
└── self-host/
    └── runner/
        ├── run.sh
        ├── .env                    # PATH=$HOME/.cargo/bin:$PATH
        └── _work/
            └── REPO/REPO/          # Workspace (auto-managed by runner)
```

---

## Important Notes

- **Manual startup required:** The runner must be started manually with `run.sh`. If WSL2 restarts, restart it before triggering any workflow.
- **Multiple runners:** Each additional runner is an independent directory registered separately. All runners share `~/.ccache`, which is safe and beneficial — concurrent builds of different devices populate a shared cache with no risk of corruption.
- **Disk usage:** Kernel sources (~20–80 GB per device) remain on disk after each run. To free space: `rm -rf ~/self-host/runner/_work/REPO/REPO/DEVICE_NAME/`
- **`hard_link = false` is mandatory:** With `hard_link = true`, `.o` files become read-only and `objcopy` fails during kernel builds on ext4.