# uConsole Dotfiles — ClockworkPi uConsole CM4

Meine persönliche Sway/Wayland-Konfiguration für den ClockworkPi uConsole
(Raspberry Pi CM4, Debian 13 trixie).

## Inhalt

| Verzeichnis | Inhalt |
|---|---|
| `sway/` | Sway-Compositor Config mit uConsole-Anpassungen |
| `waybar/` | Waybar Statusleiste mit Icons und Klick-Aktionen |
| `greetd/` | Login-Screen (greetd + gtkgreet + cage) |

## Features

- **Panel-Rotation** — DSI-1 via `transform 270` (cage) / `transform 90` (Sway), scale 1.5 → ~853×480
- **Alt-Taste als Super** — linke Alt = `$mod` via `altwin:swap_lalt_lwin` (uConsole hat keine Win-Taste)
- **Waybar-Icons** — Nerd Font Icons für WLAN, Lautstärke, Helligkeit, Akku
- **WLAN-GUI** — Klick aufs WLAN-Symbol () öffnet `iwgtk` (Netzwerkauswahl)
- **Logout-Button** — Klick auf  öffnet Bestätigung, dann `swaymsg exit`
- **Lautstärke** — Klick auf Lautstärke-Icon öffnet `pavucontrol`
- **fuzzel Launcher** — `$mod+d` öffnet fuzzel (schnelles App-Finden)
- **Login-Screen** — greetd + gtkgreet + cage mit Session-Auswahl (sway, labwc, bash)
- **Hardware-Tasten** — uConsole Lautstärke/Helligkeit via PipeWire + brightnessctl
- **Sway-Reload inkl. Waybar** — `$mod+Shift+c` startet waybar automatisch neu

## Installation

### Abhängigkeiten
```bash
sudo apt-get install -y sway waybar mako fuzzel foot brightnessctl pipewire \
                        greetd gtkgreet cage iwgtk network-manager

# Nerd Font Symbols für Waybar-Icons
mkdir -p ~/.local/share/fonts
cd ~/.local/share/fonts
curl -sL https://github.com/ryanoasis/nerd-fonts/releases/download/v3.3.0/NerdFontsSymbolsOnly.zip -o symbols.zip
unzip -qo symbols.zip && fc-cache -fv
```

### Configs übernehmen
```bash
cp -r sway/config ~/.config/sway/config
cp -r waybar/* ~/.config/waybar/
sudo cp greetd/greetd-gtkgreet-run /usr/local/bin/
sudo cp greetd/config.toml /etc/greetd/
sudo cp greetd/environments /etc/greetd/
```

### Benutzer _greetd berechtigen (wichtig!)
```bash
sudo usermod -aG video,input,render _greetd
```

### Display-Manager umstellen
```bash
sudo systemctl stop lightdm
sudo systemctl disable lightdm
sudo ln -sf /lib/systemd/system/greetd.service /etc/systemd/system/display-manager.service
sudo systemctl daemon-reload
sudo systemctl enable --now greetd
```

## Keybindings

| Tasten | Aktion |
|---|---|
| `Alt + Return` | Terminal (foot) |
| `Alt + d` | App-Launcher (fuzzel) |
| `Alt + Shift + c` | Sway + Waybar neuladen |
| `Alt + Shift + e` | Sway beenden |
| `Alt + Shift + v` | Clipboard-Verlauf |
| `Alt + Tab` | Nächster Workspace |
| `Alt + Shift + Tab` | Vorheriger Workspace |
| `Alt + 1–0` | Workspace 1–10 |
| `Alt + Shift + 1–0` | Fenster auf Workspace verschieben |
| `Alt + r` | Resize-Modus |
| `Alt + f` | Vollbild |

## Waybar-Klick-Aktionen

| Symbol | Klick | Aktion |
|---|---|---|
|  | Linksklick | Sway beenden (mit Bestätigung) |
|  | Linksklick | nmtui (NetworkManager TUI) |
| 🔊 | Linksklick | pavucontrol (Audio-Einstellungen) |

## Sway Shortcuts Wallpaper

Ein Hintergrundbild (`sway/wallpapers/wallpaper.png`) mit allen wichtigen Sway-Shortcuts in zwei Spalten, erstellt mit Pillow. Aktiviert in der Sway-Config via `output * bg ... fill`.

## Installierte Apps

| App | Zweck |
|---|---|
| **Tuba** (0.9.2) | Mastodon/Fediverse-Client (GTK4/libadwaita, für kleines Display optimiert) |
| **Raspberry Pi Connect** (2.12.0) | Remote-Zugriff via connect.raspberrypi.com (aktiv, signiert) |

### Raspberry Pi Connect einrichten
```bash
rpi-connect signin   # öffnet OAuth-Link → im Browser anmelden
```

### Netzwerk-GUI später nachrüsten
```bash
sudo apt install network-manager-gnome   # nm-connection-editor (GTK-GUI)
# Danach in waybar/config.jsonc: "on-click" ändern auf "nm-connection-editor"
```

## Netzwerk-Status (aktuell)

| Interface | IP | Verbindung |
|---|---|---|
| `eth1` | 192.168.1.43 | Ethernet |
| `wlan0` | 192.168.1.44 | WiFi — figaro |

---

## 4G/LTE Modul (SIMCOM SIM7600G-H)

### Einschalten
```bash
sudo uconsole-4g enable
sleep 20
mmcli -L
mmcli -m 0
```

### Ausschalten (Akku sparen)
```bash
sudo uconsole-4g disable
```

### SIM einlegen
SIM-Slot auf der Rückseite des 4G-Moduls (unter der Abdeckung).
```bash
mmcli -m 0 --set-primary-sim-slot=1
```

### Netzwerk-Verbindung
```bash
sudo nmcli connection add type gsm ifname cdc-wdm0 con-name "4G" apn "internet"
sudo nmcli connection up "4G"
```
(APN je nach Provider: `internet.de`, `web.vodafone`, `telekom`, etc.)

### Technisch
- Chip: SIMCOM SIM7600G-H (LTE Cat-4, bis 150 Mbps DL)
- Interface: QMI via `qmi_wwan` → `wwan0`
- Firmware: LE20B04SIM7600G22
- Status: `mmcli -m 0`
- Power via GPIO 15/24 (uconsole-4g Skript)

## WLAN — Signalverbesserung

Das CM4-Modul im Gehäuse hat oft schwachen 5-GHz-Empfang.
Auf 2.4 GHz zwingen (deutlich bessere Reichweite):

```bash
sudo nmcli connection modify <SSID> 802-11-wireless.band bg
sudo nmcli connection down <SSID>
sudo nmcli connection up <SSID>
```

### Power Save deaktivieren
```bash
sudo iw dev wlan0 set power_save off
```

## CM5 Upgrade

- **WLAN** — CM5 hat WiFi 5 (BCM4375) mit besserer Antennenabstimmung → 5 GHz könnte dann brauchbar sein
- **GPU** — VideoCore 7 → Wayfire/niri für Carousel-Workspaces testbar
- **Config** — Overlay in config.txt auf `clockworkpi-uconsole-cm5` ändern; Antenne testen (ant1/ant2)

## SD-Karte (Datenträger)

Der interne microSD-Slot wird als `/dev/mmcblk1` erkannt.

### Automount
SD-Karte einlegen → wird automatisch nach `/mnt/sd` gemountet.
Bei Entfernung → automatisch unmountet.

### Manuell
Manuell einbinden per udisks2:
```bash
udisksctl mount -b /dev/mmcblk1p1
``

```bash
lsblk  # prüfen ob SD erkannt wird
mount | grep mmcblk1  # Mount-Point anzeigen
```

### Verwendung
- Daten auslagern (Musik, Downloads, Docker)
- Home-Verzeichnis auf SD auslagern für mehr eMMC-Platz
- Backup-Ziel