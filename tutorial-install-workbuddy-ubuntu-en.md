# Tutorial: Install WorkBuddy AI on Ubuntu

This guide uses the **international build** ([workbuddy.ai](https://www.workbuddy.ai/)), not the China build (`workbuddy.cn`). Verified on Ubuntu 26.04 LTS (x86_64).

> **Why international?** The China build defaults to Chinese, login often goes through an enterprise/WeChat flow, and `.cn` accounts do not work with `.ai`.

Indonesian version: [tutorial-install-workbuddy-ubuntu-id.md](./tutorial-install-workbuddy-ubuntu-id.md)

---

## Prerequisites

- Ubuntu (Debian-based), **amd64 / x64** architecture
- `sudo` access
- Internet connection
- Commands: `curl`, `python3`, `apt`

Check architecture:

```bash
uname -m
# Expected: x86_64
```

---

## 1. Download the official `.deb`

International update API:

```bash
curl -fsS "https://www.workbuddy.ai/v2/update?platform=workbuddy-linux-x64-deb"
```

The API URL often has a wrong path (`linux-x64-deb` / `WorkBuddy-linux-x64-deb-...`). The correct CDN path is:

`https://download.codebuddy.ai/workbuddy/saas/linux-x64/WorkBuddy-linux-x64-<version>.deb`

### One-shot download

```bash
cd /tmp

meta=$(curl -fsS "https://www.workbuddy.ai/v2/update?platform=workbuddy-linux-x64-deb")
version=$(python3 -c "import json,sys; print(json.loads(sys.argv[1])['version'])" "$meta")
url=$(python3 -c "
import json,sys
u=json.loads(sys.argv[1])['url']
u=u.replace('/linux-x64-deb/','/linux-x64/')
u=u.replace('WorkBuddy-linux-x64-deb-','WorkBuddy-linux-x64-')
print(u)
" "$meta")

echo "Version: $version"
echo "URL:     $url"

# If the API is stale, try a known-good version (optional)
# url="https://download.codebuddy.ai/workbuddy/saas/linux-x64/WorkBuddy-linux-x64-5.7.6.40488862-3e43260f.deb"

curl -fL --progress-bar -o WorkBuddy-intl-linux-x64.deb "$url"
ls -lh WorkBuddy-intl-linux-x64.deb
```

Debian package name: **`workbuddyai`** (not `workbuddy`). Homepage: `https://www.workbuddy.ai`.

---

## 2. Install

```bash
sudo apt install -y /tmp/WorkBuddy-intl-linux-x64.deb
```

The vendor installs into a path **with a space**: `/opt/WorkBuddy AI`. Electron on Linux **fails to launch** from that path (`execvp: /opt/WorkBuddy`).

### Fix the install path (required)

```bash
# Move to a path without spaces
sudo mv "/opt/WorkBuddy AI" /opt/WorkBuddyAI
sudo ln -sfn /opt/WorkBuddyAI "/opt/WorkBuddy AI"

# Fix chrome-sandbox (Electron)
sudo chown root:root /opt/WorkBuddyAI/chrome-sandbox
sudo chmod 4755 /opt/WorkBuddyAI/chrome-sandbox
```

### PATH wrapper

```bash
sudo tee /usr/local/bin/workbuddyai >/dev/null <<'EOF'
#!/usr/bin/env bash
exec /opt/WorkBuddyAI/workbuddyai "$@"
EOF
sudo chmod +x /usr/local/bin/workbuddyai
sudo ln -sfn /usr/local/bin/workbuddyai /usr/local/bin/workbuddy
```

### Fix the `.desktop` file (app menu + dock icon)

The real window class is `"workbuddy ai", "WorkBuddy AI"`. `StartupWMClass` must match; otherwise the Ubuntu dock shows a blank icon.

```bash
sudo tee /usr/share/applications/workbuddyai.desktop >/dev/null <<'EOF'
[Desktop Entry]
Name=WorkBuddy AI
Exec=/opt/WorkBuddyAI/workbuddyai %U
TryExec=/opt/WorkBuddyAI/workbuddyai
Terminal=false
Type=Application
Icon=workbuddyai
StartupWMClass=WorkBuddy AI
Comment=WorkBuddy Desktop - AI Agent Desktop Application
MimeType=x-scheme-handler/workbuddy-ai;
Categories=Development;
EOF

sudo cp /usr/share/icons/hicolor/256x256/apps/workbuddyai.png /usr/share/pixmaps/workbuddyai.png
sudo gtk-update-icon-cache -f /usr/share/icons/hicolor
sudo update-desktop-database /usr/share/applications
```

---

## 3. Launch & sign in

- App menu → **WorkBuddy AI**, or
- Terminal: `workbuddyai`

Sign in with a **[workbuddy.ai](https://www.workbuddy.ai/)** account, not `workbuddy.cn`.

UI language: avatar (bottom left) → **Settings** → **System Settings** → **Language** → **English**.

---

## 4. Update script (`update-workbuddy`)

Save as `~/.local/bin/update-workbuddy` (ensure `~/.local/bin` is on your `PATH`):

```bash
mkdir -p ~/.local/bin
```

What the script should do:

1. Check version via `https://www.workbuddy.ai/v2/update?platform=workbuddy-linux-x64-deb`
2. Rewrite the CDN URL (`linux-x64` / drop `-deb` in the filename)
3. `sudo apt install` the `workbuddyai` package
4. Re-run path normalization to `/opt/WorkBuddyAI`, sandbox fix, `.desktop`, and wrapper

Usage:

```bash
update-workbuddy          # update when a newer version is available
update-workbuddy --force  # force download & reinstall
```

> Ubuntu has **no** official WorkBuddy apt repo. Updates mean downloading a new `.deb` (in-app update may appear, but the Linux channel is often incomplete).

On this machine the script usually lives at:

`~/.local/bin/update-workbuddy`

---

## 5. Behavior after closing the window

**Expected:** closing the window ≠ fully quitting. Background processes (daemon/sandbox) stay running so agent tasks can continue and a phone can stay connected.

Fully quit: tray → **Quit**, or:

```bash
pkill -f '/opt/WorkBuddyAI/workbuddyai'
```

---

## 6. Android (optional)

There is a mobile app; it is often a **direct APK** from the official site (not always on the Play Store).

- Safe **only** if downloaded from an official WorkBuddy/Tencent domain
- Do not use mirrors or “mod” APKs
- Official docs often mention Mac/Windows for **Connect to Computer**; remote control from a phone on **Linux** is not guaranteed

On desktop: enable **Allow Mobile Connections**, then on the phone sign in with the same account → Connect to Computer.

---

## 7. Uninstall

```bash
pkill -f '/opt/WorkBuddyAI/workbuddyai' 2>/dev/null || true
sudo apt remove -y workbuddyai
sudo rm -f /usr/local/bin/workbuddyai /usr/local/bin/workbuddy
sudo rm -f /usr/share/applications/workbuddyai.desktop
sudo rm -rf /opt/WorkBuddyAI "/opt/WorkBuddy AI"
```

Local data (optional to remove):

```bash
rm -rf ~/.workbuddy-ai
# Old China-build backup (if you installed it before): ~/.workbuddy.cn.bak.*
```

---

## Troubleshooting

| Symptom | Cause / fix |
| --- | --- |
| Menu click does nothing; log shows `execvp: /opt/WorkBuddy` | Path contains a space → move to `/opt/WorkBuddyAI` (step 2) |
| `chrome-sandbox` / zygote fatal | `chown root` + `chmod 4755` on `chrome-sandbox` |
| Blank dock icon | Set `StartupWMClass=WorkBuddy AI` + `Icon=workbuddyai`, refresh icon cache |
| Enterprise login / Chinese UI | Wrong build (China). Remove `workbuddy`, install `workbuddyai` |
| API download 404 | Rewrite URL to `.../linux-x64/WorkBuddy-linux-x64-....deb` |
| `update-workbuddy` says API is older than installed | Use `--force` only if you really want to reinstall the API build |

---

## Command checklist

```bash
# 1) Download + install
cd /tmp
# ... download WorkBuddy-intl-linux-x64.deb (see section 1) ...
sudo apt install -y ./WorkBuddy-intl-linux-x64.deb

# 2) Fix path + sandbox + launcher
sudo mv "/opt/WorkBuddy AI" /opt/WorkBuddyAI
sudo ln -sfn /opt/WorkBuddyAI "/opt/WorkBuddy AI"
sudo chown root:root /opt/WorkBuddyAI/chrome-sandbox
sudo chmod 4755 /opt/WorkBuddyAI/chrome-sandbox
# ... wrapper + .desktop (see section 2) ...

# 3) Launch
workbuddyai
```

---

## References

- International site: <https://www.workbuddy.ai/>
- Linux x64 update API: `https://www.workbuddy.ai/v2/update?platform=workbuddy-linux-x64-deb`
- CDN: `https://download.codebuddy.ai/workbuddy/saas/linux-x64/`
- Cross-device docs: <https://www.workbuddy.ai/document/cross-device-tasks>
