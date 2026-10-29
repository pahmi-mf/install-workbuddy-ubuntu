# Tutorial: Install WorkBuddy AI di Ubuntu

Panduan ini memakai **build internasional** ([workbuddy.ai](https://www.workbuddy.ai/)), bukan build China (`workbuddy.cn`). Diuji di Ubuntu 26.04 LTS (x86_64).

> **Kenapa internasional?** Build China default-nya Mandarin, login sering mengarah ke alur enterprise/WeChat, dan akun `.cn` tidak cocok dengan `.ai`.

English version: [tutorial-install-workbuddy-ubuntu-en.md](./tutorial-install-workbuddy-ubuntu-en.md)

---

## Prasyarat

- Ubuntu (Debian-based), arsitektur **amd64 / x64**
- Akses `sudo`
- Koneksi internet
- Perintah: `curl`, `python3`, `apt`

Cek arsitektur:

```bash
uname -m
# Harus: x86_64
```

---

## 1. Unduh paket `.deb` resmi

API update internasional:

```bash
curl -fsS "https://www.workbuddy.ai/v2/update?platform=workbuddy-linux-x64-deb"
```

URL dari API sering salah path (`linux-x64-deb` / `WorkBuddy-linux-x64-deb-...`). Path CDN yang benar:

`https://download.codebuddy.ai/workbuddy/saas/linux-x64/WorkBuddy-linux-x64-<versi>.deb`

### Cara cepat (satu blok)

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

# Jika API stale, coba versi yang diketahui tersedia (opsional)
# url="https://download.codebuddy.ai/workbuddy/saas/linux-x64/WorkBuddy-linux-x64-5.7.6.40488862-3e43260f.deb"

curl -fL --progress-bar -o WorkBuddy-intl-linux-x64.deb "$url"
ls -lh WorkBuddy-intl-linux-x64.deb
```

Nama paket Debian: **`workbuddyai`** (bukan `workbuddy`). Homepage: `https://www.workbuddy.ai`.

---

## 2. Install

```bash
sudo apt install -y /tmp/WorkBuddy-intl-linux-x64.deb
```

Vendor menginstall ke folder **dengan spasi**: `/opt/WorkBuddy AI`. Electron di Linux **gagal launch** dari path itu (error `execvp: /opt/WorkBuddy`).

### Perbaiki path (wajib)

```bash
# Pindahkan ke path tanpa spasi
sudo mv "/opt/WorkBuddy AI" /opt/WorkBuddyAI
sudo ln -sfn /opt/WorkBuddyAI "/opt/WorkBuddy AI"

# Perbaiki chrome-sandbox (Electron)
sudo chown root:root /opt/WorkBuddyAI/chrome-sandbox
sudo chmod 4755 /opt/WorkBuddyAI/chrome-sandbox
```

### Wrapper di PATH

```bash
sudo tee /usr/local/bin/workbuddyai >/dev/null <<'EOF'
#!/usr/bin/env bash
exec /opt/WorkBuddyAI/workbuddyai "$@"
EOF
sudo chmod +x /usr/local/bin/workbuddyai
sudo ln -sfn /usr/local/bin/workbuddyai /usr/local/bin/workbuddy
```

### Perbaiki `.desktop` (menu + dock icon)

Window class aslinya: `"workbuddy ai", "WorkBuddy AI"`. `StartupWMClass` harus cocok, kalau tidak icon di dock Ubuntu kosong.

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

## 3. Jalankan & login

- App menu → **WorkBuddy AI**, atau
- Terminal: `workbuddyai`

Login dengan akun **[workbuddy.ai](https://www.workbuddy.ai/)**, bukan `workbuddy.cn`.

Bahasa UI: avatar (kiri bawah) → **Settings** → **System Settings** → **Language** → **English**.

---

## 4. Script update (`update-workbuddy`)

Simpan sebagai `~/.local/bin/update-workbuddy` (pastikan `~/.local/bin` ada di `PATH`):

```bash
mkdir -p ~/.local/bin
```

Isi script (inti perilaku):

1. Cek versi via `https://www.workbuddy.ai/v2/update?platform=workbuddy-linux-x64-deb`
2. Perbaiki URL CDN (`linux-x64` / tanpa `-deb` di nama file)
3. `sudo apt install` paket `workbuddyai`
4. Jalankan ulang normalisasi path `/opt/WorkBuddyAI`, sandbox, `.desktop`, dan wrapper

Pemakaian:

```bash
update-workbuddy          # update jika ada versi baru
update-workbuddy --force  # paksa unduh & reinstall
```

> Ubuntu **tidak** punya repo apt resmi WorkBuddy. Update = unduh `.deb` baru (in-app update bisa muncul, tapi channel Linux sering tidak lengkap).

Salinan script yang dipakai di mesin ini biasanya ada di:

`~/.local/bin/update-workbuddy`

---

## 5. Perilaku setelah close

**Normal:** menutup jendela ≠ keluar penuh. Proses background (daemon/sandbox) tetap jalan agar task agent bisa lanjut dan HP bisa connect.

Keluar penuh: tray → **Quit**, atau:

```bash
pkill -f '/opt/WorkBuddyAI/workbuddyai'
```

---

## 6. Android (opsional)

Ada app mobile; sering unduhan **APK langsung** dari situs resmi (bukan selalu di Play Store).

- Aman **hanya** jika dari domain resmi WorkBuddy/Tencent
- Jangan unduh dari mirror / APK “mod”
- Docs resmi sering menyebut Mac/Windows untuk **Connect to Computer**; di Linux fitur remote dari HP belum dijamin

Di desktop: aktifkan **Allow Mobile Connections**, lalu di HP login akun yang sama → Connect to Computer.

---

## 7. Uninstall

```bash
pkill -f '/opt/WorkBuddyAI/workbuddyai' 2>/dev/null || true
sudo apt remove -y workbuddyai
sudo rm -f /usr/local/bin/workbuddyai /usr/local/bin/workbuddy
sudo rm -f /usr/share/applications/workbuddyai.desktop
sudo rm -rf /opt/WorkBuddyAI "/opt/WorkBuddy AI"
```

Data lokal (opsional dihapus):

```bash
rm -rf ~/.workbuddy-ai
# Backup build China lama (jika pernah install): ~/.workbuddy.cn.bak.*
```

---

## Troubleshooting

| Gejala | Penyebab / solusi |
| --- | --- |
| Klik menu tidak launch; log `execvp: /opt/WorkBuddy` | Path ada spasi → pindah ke `/opt/WorkBuddyAI` (langkah 2) |
| `chrome-sandbox` / zygote fatal | `chown root` + `chmod 4755` pada `chrome-sandbox` |
| Icon di dock kosong | Set `StartupWMClass=WorkBuddy AI` + `Icon=workbuddyai`, refresh icon cache |
| Login enterprise / UI Mandarin | Salah build (China). Hapus `workbuddy`, pasang `workbuddyai` |
| Unduh API 404 | Rewrite URL ke `.../linux-x64/WorkBuddy-linux-x64-....deb` |
| `update-workbuddy` bilang API lebih lama dari yang terpasang | Pakai `--force` hanya jika memang ingin reinstall build API |

---

## Ringkasan perintah (checklist)

```bash
# 1) Unduh + install
cd /tmp
# ... unduh WorkBuddy-intl-linux-x64.deb (lihat bagian 1) ...
sudo apt install -y ./WorkBuddy-intl-linux-x64.deb

# 2) Fix path + sandbox + launcher
sudo mv "/opt/WorkBuddy AI" /opt/WorkBuddyAI
sudo ln -sfn /opt/WorkBuddyAI "/opt/WorkBuddy AI"
sudo chown root:root /opt/WorkBuddyAI/chrome-sandbox
sudo chmod 4755 /opt/WorkBuddyAI/chrome-sandbox
# ... wrapper + .desktop (lihat bagian 2) ...

# 3) Jalankan
workbuddyai
```

---

## Referensi

- Situs internasional: <https://www.workbuddy.ai/>
- API update Linux x64: `https://www.workbuddy.ai/v2/update?platform=workbuddy-linux-x64-deb`
- CDN: `https://download.codebuddy.ai/workbuddy/saas/linux-x64/`
- Docs cross-device: <https://www.workbuddy.ai/document/cross-device-tasks>
