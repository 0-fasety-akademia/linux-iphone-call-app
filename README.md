# Linux iPhone Calls

Answer and make **iPhone calls on Ubuntu** over Bluetooth. Your laptop acts as a hands-free
kit (like a car): see who is calling, answer on the laptop, keep the call on the iPhone or
decline it, and call anyone from your iPhone contacts.

**Created by: JGu**  
**Supported by Claude**

Part of the series: linux-icloud-photos · linux-icloud-mail · linux-apple-music-player ·
linux-imessage · **linux-iphone-call-app**.

> 🇵🇱 Odbieranie i wykonywanie połączeń z iPhone'a na Ubuntu przez Bluetooth. Interfejs
> PL/EN według języka systemu. Opis po polsku na dole.

## Features

- **Incoming call** popup + notification with three choices:
  **Answer** (on the laptop) · **On iPhone** (stop ringing here, answer on the phone) · **Decline**
- Caller name from your iPhone contacts (matched on the last 9 digits, so `+48`, `0048`
  and local numbers all match)
- In-call: mute, keypad (DTMF), **move audio to iPhone ⇄ laptop**, end call, call timer
- **Keypad** with contact autocomplete, **Contacts** with search, **Recents** (iPhone call
  history incl. missed calls)
- Missed-call notification with **Call back**
- **Tray** icon with status colors (idle / ringing / in call / missed / offline) and menu;
  closing the window keeps it in the tray, quit from the tray menu
- **Light/dark theme** following the system, with matching app icon
- **PL / EN** UI following the system language
- Option **“Keep iPhone app audio on the iPhone”**: games, YouTube, music stay on the
  phone; only calls come to the laptop (enabled by default)
- Optional ringtone on the laptop, start at login (enabled on first run)
- Version and build date shown bottom-left; **About** (Created by: JGu · Supported by Claude) and **Diagnostics** dialogs

## Privacy

Everything stays on your machine. Contacts and call history are pulled from the iPhone over
Bluetooth (PBAP), cached in `~/.local/share/linux-iphone-call-app/` with `0600`
permissions, and never sent anywhere. No network access is used at all.

## Requirements

- Ubuntu with **PipeWire ≥ 1.4** (Bluetooth telephony service `org.pipewire.Telephony`),
  e.g. Ubuntu 25.10. Ubuntu 24.04 ships PipeWire 1.0, which does not have it — the app
  shows a warning and the Diagnostics dialog explains it.
- WirePlumber 0.4 or 0.5, BlueZ with `bluez-obexd`
- iPhone paired with the laptop. For contacts/history on the iPhone:
  **Settings → Bluetooth → (i) next to the laptop → Sync Contacts**

Check your version:

```bash
pipewire --version
```

## Install

```bash
sudo apt install ./linux-iphone-call-app_0.1.3_all.deb
linux-iphone-call-app            # or from the app grid: “Linux iPhone Calls”
```

Update: quit from the tray (or `pkill -f linux_iphone_call_app`), install the new `.deb`,
start again.

## Run from source

```bash
sudo apt install python3-gi gir1.2-gtk-3.0 gir1.2-ayatanaappindicator3-0.1 \
                 bluez-obexd pipewire-bin wireplumber librsvg2-common
PYTHONPATH=src python3 -m linux_iphone_call_app.app
```

Build the package: `packaging/build-deb.sh` → `dist/linux-iphone-call-app_<ver>_all.deb`

UI test against mocked BlueZ / PipeWire telephony (screenshots in `/tmp/lica-shots`):

```bash
dbus-run-session -- xvfb-run -a -s "-screen 0 1000x780x24" tests/run_ui_test.sh
```

## How it works

| Part | Technology |
|---|---|
| Call control (ring, answer, decline, dial, DTMF) | PipeWire Bluetooth telephony D-Bus API (`org.pipewire.Telephony`, HFP hands-free role) |
| Call audio | HFP/SCO via PipeWire; bridged to speakers/mic with `pw-loopback` if WirePlumber hasn't linked it |
| Audio to iPhone / laptop | `RejectSCO` + `Activate` on the transport; fallbacks: card profile off, then re-connect HFP |
| Contacts, call history | Bluetooth PBAP via `bluez-obexd` |
| Connect / disconnect | BlueZ `Device1.ConnectProfile` for the Hands-Free Audio Gateway only (no media) |
| App audio stays on iPhone | WirePlumber drop-in removing the `a2dp_sink` role, then a WirePlumber restart (postponed until no call is active) |
| UI | Python 3 + GTK 3, Ayatana AppIndicator tray, freedesktop notifications |

Files written to your config:

- `~/.config/linux-iphone-call-app/config.json`
- `~/.config/autostart/linux-iphone-call-app.desktop` (toggle in the menu)
- `~/.config/wireplumber/wireplumber.conf.d/51-linux-iphone-call-app-no-phone-media.conf`
  (WirePlumber 0.5) or `~/.config/wireplumber/bluetooth.lua.d/51-linux-iphone-call-app-no-phone-media.lua`
  (0.4), only while the option is on

## Limitations

- Regular cellular calls only (no FaceTime / WhatsApp audio)
- Hands-free audio quality (mSBC 16 kHz, like a car kit)
- One phone at a time (choose it in the menu if several are paired)
- “Keep iPhone app audio on the iPhone” applies to every phone: no phone can use the
  laptop as a Bluetooth speaker while it is on

## Troubleshooting

- **“Bluetooth telephony service missing”** → PipeWire is older than 1.4.
- **No contacts** → enable *Sync Contacts* on the iPhone for this laptop, then menu →
  *Refresh contacts and history*.
- **App audio still on the laptop** → toggle Bluetooth on the iPhone once; check the
  Diagnostics line *App audio stays on iPhone*.
- Run `linux-iphone-call-app` from a terminal to see logs, and include the Diagnostics text
  in bug reports.

---

## Po polsku

**Linux iPhone Calls** – laptop jako zestaw głośnomówiący dla iPhone'a. Gdy ktoś dzwoni,
widzisz kto, i wybierasz: **Odbierz** (na laptopie), **Na iPhonie** albo **Odrzuć**. W trakcie
rozmowy: wyciszenie, klawiatura, przenoszenie dźwięku iPhone ⇄ laptop. Dzwonisz z klawiatury
(z podpowiadaniem kontaktów), z listy kontaktów albo z historii połączeń. Tray, powiadomienia,
motyw i ikona jasna/ciemna, interfejs PL/EN. Opcja „Dźwięk aplikacji iPhone'a zostaje na
iPhonie” sprawia, że gry i YouTube grają na telefonie, a na laptopa trafiają tylko rozmowy.

Wymaga PipeWire 1.4+. Wszystkie dane zostają lokalnie na laptopie.

## License

MIT — see [LICENSE](LICENSE). Created by: JGu · Supported by Claude. The phone handset glyph in the icons is based on Material
Icons “call” (Apache License 2.0, © Google).
