# kanata on macOS

`macos.kbd` expects the macOS input source to be **Colemak** (kanata passes letters through).

Versions: kanata **v1.12.x** needs Karabiner-DriverKit-VirtualHIDDevice **v6.2.0** exactly.
kanata v1.13+ needs driver **v8.0.0** — upgrade both together. Don't run Karabiner-Elements
alongside it (it grabs the keyboard and bundles its own driver version).

## Install

1. Driver: install the v6.2.0 `.pkg` from
   https://github.com/pqrs-org/Karabiner-DriverKit-VirtualHIDDevice/releases/tag/v6.2.0, then
   ```sh
   sudo /Applications/.Karabiner-VirtualHIDDevice-Manager.app/Contents/MacOS/Karabiner-VirtualHIDDevice-Manager forceActivate
   ```
   System Settings > General > Login Items & Extensions > Driver Extensions: enable
   `org.pqrs.Karabiner-DriverKit-VirtualHIDDevice`.
2. kanata: download `kanata-macos-arm64` from the v1.12.x release (not Homebrew — upgrades
   change the path and macOS drops the permission grant).
   ```sh
   chmod +x kanata_macos_arm64 && sudo mv kanata_macos_arm64 /usr/local/bin/kanata
   xattr -d com.apple.quarantine /usr/local/bin/kanata 2>/dev/null
   ```
3. Config (from this repo's root):
   ```sh
   mkdir -p ~/.config/kanata && ln -sf "$PWD/kanata/macos.kbd" ~/.config/kanata/kanata.kbd
   kanata --cfg ~/.config/kanata/kanata.kbd --check
   ```
4. Permissions: System Settings > Privacy & Security > **Input Monitoring** and
   **Accessibility** → `+` → Cmd+Shift+G → `/usr/local/bin/kanata`.
5. Smoke test (exit with physical Control + Space + Esc):
   ```sh
   sudo "/Library/Application Support/org.pqrs/Karabiner-DriverKit-VirtualHIDDevice/Applications/Karabiner-VirtualHIDDevice-Daemon.app/Contents/MacOS/Karabiner-VirtualHIDDevice-Daemon" &
   sudo kanata --cfg ~/.config/kanata/kanata.kbd
   ```
   Then stop that test daemon (`sudo pkill -f Karabiner-VirtualHIDDevice-Daemon`).
6. Run at boot. `dev.kanata.kanata.plist` assumes the config at `/Users/dylan/.config/kanata/`.
   ```sh
   for p in org.pqrs.Karabiner-VirtualHIDDevice-Daemon dev.kanata.kanata; do
     sudo cp "kanata/$p.plist" /Library/LaunchDaemons/
     sudo chown root:wheel "/Library/LaunchDaemons/$p.plist"
     sudo launchctl enable "system/$p"
     sudo launchctl bootstrap system "/Library/LaunchDaemons/$p.plist"
   done
   ```

## Day to day

- Reload after editing: `sudo launchctl kickstart -k system/dev.kanata.kanata`
- Stop (the panic chord just gets restarted by launchd): `sudo launchctl bootout system/dev.kanata.kanata`
- Logs: `tail -f /var/log/kanata.log`

## Troubleshooting

- `Bootstrap failed: 5: Input/output error`: plist not owned by `root:wheel`, already
  bootstrapped, or disabled (`launchctl enable` first).
- `connect_failed asio.system:2` looping: the VirtualHIDDevice daemon isn't running.
- Keys pass through unmapped after replacing the binary: toggle kanata off/on in Input Monitoring.
- Don't also swap modifiers in System Settings > Keyboard > Modifier Keys.
