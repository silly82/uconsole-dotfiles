# pifates — ClockworkPi uConsole CM4 config

Meine persönliche Sway/Wayland-Konfiguration für den ClockworkPi uConsole
(Raspberry Pi CM4, Debian 13 trixie).

## Inhalt

| Verzeichnis | Inhalt |
|---|---|
| `sway/` | Sway-Compositor Config mit uConsole-Anpassungen |
| `waybar/` | Waybar Statusleiste mit Icons (benötigt Nerd Font) |
| `greetd/` | Login-Screen (greetd + gtkgreet + cage) |

## Features

- DSI-Panel-Rotation (transform 90, scale 1.5) für Querformat ~853x480
- Linke Alt-Taste als `$mod` (Super) via `altwin:swap_lalt_lwin`
- Logout-Button in Waybar
- fuzzel als App-Launcher
- uConsole Hardware-Tasten (Lautstärke, Helligkeit)
- gtk-greeter-labwc als Wayland-native Login-Session-Auswahl

## Installation

```bash
# Configs übernehmen
cp -r sway/config ~/.config/sway/config
cp -r waybar/* ~/.config/waybar/
sudo cp greetd/* /etc/greetd/
```

Notwendige Pakete:
```bash
sudo apt-get install -y sway waybar mako fuzzel foot brightnessctl pipewire
# Nerd Font Symbols für Icons:
mkdir -p ~/.local/share/fonts
cd ~/.local/share/fonts
curl -sL https://github.com/ryanoasis/nerd-fonts/releases/download/v3.3.0/NerdFontsSymbolsOnly.zip -o symbols.zip
unzip -qo symbols.zip && fc-cache -fv
```

## 4G/LTE Modul (SIMCOM SIM7600G-H)

### Einschalten
```bash
sudo uconsole-4g enable
# ~20s warten, dann:
mmcli -L
mmcli -m 0
```

### Ausschalten (Akku sparen)
```bash
sudo uconsole-4g disable
```

### SIM einlegen
SIM-Slot auf der Rückseite des 4G-Moduls (unter der Abdeckung).
Nach Einlegen: Modem einschalten, SIM-Slot aktivieren:
```bash
mmcli -m 0 --set-primary-sim-slot=1
```

### Netzwerk-Verbindung (QMI)
```bash
sudo nmcli connection add type gsm ifname cdc-wdm0 con-name "4G" apn "internet"
sudo nmcli connection up "4G"
```
(APN je nach Provider anpassen, z.B. `internet`, `web.vodafone`, `telekom`, etc.)

### Technisch
- Chip: SIMCOM SIM7600G-H (LTE Cat-4, bis 150 Mbps DL)
- Interface: QMI via `qmi_wwan` → `wwan0`
- Ports: `ttyUSB0-4` (AT, GPS, Audio)
- Status via ModemManager: `mmcli -m 0`
- Power via GPIO 15/24 (uconsole-4g Skript)
