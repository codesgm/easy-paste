# Easy Paste — histórico de área de transferência

<img src="icon.png" alt="" width="112" align="right">

A GNOME Shell extension that brings a Windows-style clipboard history panel to GNOME. Open a searchable list of everything you have copied — text and images alike — then pick an entry and have it pasted straight into the application you were working in.

### Local fork: open at the mouse pointer

The panel opens next to the mouse pointer, both from the configured shortcut
and from the top-bar icon. Its position is captured once per opening; moving
the mouse afterwards does not move the panel. GNOME's native popup positioning
keeps it within the monitor work area when space permits and flips it above
the pointer near the bottom edge. The height limit uses the pointer's monitor.

The extension reads the system clipboard whenever its contents change, and keeps that history locally in `~/.cache/clipboard-history@manouchehr/`. Nothing is sent over the network.

**Atalho padrão: Ctrl + ' (apóstrofo).** Você pode alterá-lo nas preferências.

Written against the modern ESM extension API for **GNOME Shell 46–50**, and tested on **Wayland** under **Arch Linux**.

## Instalação no Ubuntu

Este fork abre o histórico na posição do mouse. Baseado no projeto
[Clipboard History, de Manouchehr Ghalevand](https://github.com/M-ghalevand/gnome-shell-extension-clipboard-history), sob licença GPL-2.0-or-later.

### 1. Verifique a compatibilidade e instale as ferramentas

Compatível com GNOME Shell **46 a 50**. Confira sua versão:

```bash
gnome-shell --version
sudo apt update
sudo apt install git gnome-shell-extension-prefs libglib2.0-bin
```

### 2. Baixe, empacote e instale

```bash
git clone https://github.com/codesgm/easy-paste.git
cd easy-paste
gnome-extensions pack . --force --extra-source=LICENSE --extra-source=icon.svg
gnome-extensions install --force clipboard-history@manouchehr.shell-extension.zip
glib-compile-schemas "$HOME/.local/share/gnome-shell/extensions/clipboard-history@manouchehr/schemas/"
```

O fork mantém o identificador original, portanto substitui a extensão Clipboard
History se ela já estiver instalada. Os comandos de instalação normalmente não
exibem mensagem quando terminam com sucesso.

### 3. Recarregue e ative

No **Wayland**, saia da sessão do Ubuntu e entre novamente. No **X11**, pressione
**Alt + F2**, digite `r` e pressione Enter. Para descobrir sua sessão, execute
`echo "$XDG_SESSION_TYPE"`.

Depois de recarregar:

```bash
gnome-extensions enable clipboard-history@manouchehr
gnome-extensions prefs clipboard-history@manouchehr
```

### 4. Use o atalho Ctrl + '

Copie alguns textos, mova o mouse e pressione **Ctrl + '** (apóstrofo).
Escolha um item para colar. O painel também abre pelo ícone superior e permanece
no lugar enquanto estiver aberto. **Esc** ou um clique fora fecha o painel.

Você pode escolher outro atalho nas preferências. Configurações anteriores são
preservadas ao atualizar. Para aplicar o novo padrão caso já tenha personalizado
o atalho, execute:

```bash
gsettings --schemadir "$HOME/.local/share/gnome-shell/extensions/clipboard-history@manouchehr/schemas" reset org.gnome.shell.extensions.clipboard-history toggle-shortcut
```

### Atualizar este fork

Dentro da pasta do projeto, execute `git pull` e repita os comandos de
empacotamento, instalação e compilação do passo 2. Recarregue a sessão conforme
o passo 3 para carregar o código novo.

---

## Features

- **Text and image history** — every clipboard entry is captured and kept in a scrollable list
- **Inline image previews** — copied images are shown as thumbnails rather than opaque placeholders
- **Pinning** — pin the entries you reach for often so they are never evicted from history
- **Search** — filter the entire history as you type
- **Auto-paste** — selecting an entry inserts it directly into the focused application
- **Emoji, Kaomoji and Symbols tabs** — insert characters that are awkward to type, the same way the Windows Win+. panel does
- **Persistent storage** — history survives logout, and pinned entries are kept indefinitely
- **Password-manager aware** — clipboard content marked sensitive by KeePassXC, Bitwarden or KWallet is never recorded
- **Full RTL support** — Persian and Arabic text renders and aligns correctly

## Requirements

| | |
|---|---|
| GNOME Shell | 46 – 50 |
| Session type | Wayland (X11 also supported) |
| Build dependency | `glib-compile-schemas` (ships with `glib2`) |

## Project layout

The repository checks out as `gnome-shell-extension-clipboard-history`, but the *installed* directory must be named after the UUID, `clipboard-history@manouchehr`.

```
gnome-shell-extension-clipboard-history/
├── extension.js       # Core logic: clipboard monitoring, menu, auto-paste
├── prefs.js           # Preferences page: shortcut, history size, image capture
├── metadata.json
├── stylesheet.css
├── LICENSE
└── schemas/
    └── org.gnome.shell.extensions.clipboard-history.gschema.xml
```

The repository also holds `icon.svg`, the source artwork, and `icon.png`, a 256×256 render of it for the extensions.gnome.org listing. Neither is needed at runtime. `gnome-extensions pack` picks up neither on its own; `icon.svg` is added to the bundle explicitly with `--extra-source`, as is `LICENSE`, while `icon.png` stays out of it.

## Installation

The installed directory name must match the UUID exactly: `clipboard-history@manouchehr`.

### Manual install (recommended for development)

```bash
# Copy the checkout in under its UUID
cp -r easy-paste ~/.local/share/gnome-shell/extensions/clipboard-history@manouchehr

# Compile the GSettings schema (required for preferences to work)
glib-compile-schemas ~/.local/share/gnome-shell/extensions/clipboard-history@manouchehr/schemas/

# Enable it
gnome-extensions enable clipboard-history@manouchehr
```

### Using `gnome-extensions pack`

The official tooling produces the distributable bundle to upload to extensions.gnome.org:

```bash
gnome-extensions pack . --force \
  --extra-source=icon.svg \
  --extra-source=LICENSE
gnome-extensions install --force clipboard-history@manouchehr.shell-extension.zip
gnome-extensions enable clipboard-history@manouchehr
```

`pack` picks up `stylesheet.css`, `prefs.js` and `schemas/` on its own; `icon.svg` and `LICENSE` are not among the files it knows about, hence the two `--extra-source` flags.

The bundle ships the schema as XML and does **not** contain a compiled `gschemas.compiled` — `pack` never compiles schemas. That is what extensions.gnome.org expects, since it compiles them on upload, but it means a locally installed bundle still needs the schema compiled by hand before the preferences will open:

```bash
glib-compile-schemas ~/.local/share/gnome-shell/extensions/clipboard-history@manouchehr/schemas/
```

### Loading the new code

- **X11** — press `Alt+F2`, type `restart`, and press Enter.
- **Wayland** — GNOME Shell cannot be restarted in place. Log out and back in, or use the nested session described under [Development](#development).

Once enabled, a small clipboard icon appears in the top panel and toggles the history popup.

## The Super+V shortcut

This fork defaults to Ctrl + apostrophe. If you prefer Super+V, follow the optional steps below.

GNOME binds both **Super+V** and **Super+M** to `toggle-message-tray` out of the box, so first free Super+V:

```bash
gsettings set org.gnome.shell.keybindings toggle-message-tray "['<Super>m']"

# Revert to the GNOME default
gsettings reset org.gnome.shell.keybindings toggle-message-tray
```

Then open the extension preferences, click **Change** next to *Toggle clipboard panel*, and press Super+V.

## Preferences

```bash
gnome-extensions prefs clipboard-history@manouchehr
```

The preferences window lets you rebind the shortcut (click **Change**, then press the new combination), set the maximum number of history entries, and toggle whether copied images are captured.

## Development

### Testing in a nested session

GNOME Shell cannot be restarted within a running Wayland session, but a nested instance can be launched in its own window. It runs independently and picks up extensions from your user directory.

The correct flag depends on your Shell version — check `gnome-shell --version`:

```bash
# GNOME 49 and later
dbus-run-session gnome-shell --devkit --wayland

# GNOME 48 and earlier
dbus-run-session gnome-shell --nested --wayland
```

Or use this script, which picks the right flag automatically:

```bash
#!/bin/sh -e
export G_MESSAGES_DEBUG=all
export SHELL_DEBUG=all

if [ "$(gnome-shell --version | awk '{print int($3)}')" -ge 49 ]; then
    dbus-run-session gnome-shell --devkit --wayland
else
    dbus-run-session gnome-shell --nested --wayland
fi
```

Enable the extension inside the nested session and open the panel from its top-bar icon. JavaScript cannot be hot-reloaded, so after editing `extension.js` you need to close the nested window and reopen it.

> The nested instance is not perfectly isolated from the host session, but it is safe and more than adequate for day-to-day development.

### Troubleshooting

```bash
# Live GNOME Shell log, including JavaScript errors from extensions
journalctl -f -o cat /usr/bin/gnome-shell

# Preferences run in a separate process and log separately
journalctl -f -o cat /usr/bin/gjs

# Watch settings as they are written
dconf watch /org/gnome/shell/extensions/clipboard-history/
```

**Looking Glass** (`Alt+F2`, then `lg`) has an Extensions tab showing extension state and any errors raised during load.

If a change to `extension.js` does not take effect, a disable/enable cycle is usually enough:

```bash
gnome-extensions disable clipboard-history@manouchehr
gnome-extensions enable clipboard-history@manouchehr
```

Changes to `metadata.json` or the schema require a full logout.

## Known limitations

- Images are placed back on the clipboard under the single MIME type they were captured as (`image/png`, `image/jpeg`, `image/tiff` or `image/bmp`). Most applications — browsers, GIMP, LibreOffice — accept this, but a few expect `text/uri-list` or another format and will not receive the paste.
- A 250 ms delay (`PASTE_DELAY_MS` in `extension.js`) precedes the synthetic Ctrl+V. This suits most applications; adjust it if paste events land too early or too late on your system.
- History and cached images are stored in `~/.cache/clipboard-history@manouchehr/` and persist across enable/disable and logout. Pinned entries are kept indefinitely; the rest are trimmed to the configured maximum.

## Contributing

Bug reports and pull requests are welcome. When reporting a problem, please include your GNOME Shell version (`gnome-shell --version`), your session type, and any relevant output from `journalctl -f -o cat /usr/bin/gnome-shell`.

## License

Copyright © 2025 Manouchehr Ghalevand.

This program is free software; you can redistribute it and/or modify it under the terms of the **GNU General Public License, version 2 or (at your option) any later version**, as published by the Free Software Foundation. It is distributed in the hope that it will be useful, but WITHOUT ANY WARRANTY; without even the implied warranty of MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE. See [LICENSE](LICENSE) for the full text.

SPDX identifier: `GPL-2.0-or-later`
