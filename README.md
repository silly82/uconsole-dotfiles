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
