# Jellyfin Desktop with Picture in Picture support

I use Jellyfin desktop a lot on Mac and I missed PIP support, so I implemented it. 
[Video](https://drive.google.com/file/d/1qt8gFE0JDluJfdaFFixgj-qo4L7fI1Po/view?usp=sharing).

I implemented this feature while the Jellyfin-Desktop team was migrating from QT to CEF and disallowed changes to the old code base, which is now archived.
The new [CEF-based client](https://github.com/jellyfin/jellyfin-desktop) is still in active development and not yet released to the public.

This version is based on the final Jellyfin-Desktop-QT release [v1.12.0](https://github.com/jellyfin-archive/jellyfin-desktop-qt/releases/tag/v1.12.0).

Picture in Picture was tested macOS Sequoia and Windows 11. It is intentionally disabled on Linux and Wayland as I didn't have the time to test it.

## Features
- Add PiP support to the native mpv player integration
- Add keyboard shortcuts:
  - `Ctrl+Shift+P` to toggle PiP
  - `Escape` to exit PiP
- Expose PiP as a supported native-shell feature so PiP toggle appears in video player OSD
- Minimal screen real estate usage:
  - PiP window matches video aspect ratio
  - PiP window is frameless (no title bar)
  - PiP window can be resized down to 160x90
- Persist PiP size and position across sessions, separately from normal window geometry
- Add drag-anywhere move support for the frameless PiP windows
- Keep PiP always-on-top
- Respect Kiosk mode: if `forceAlwaysFS` is enabled, PiP mode is blocked
- Add `WindowManager` APIs and state for entering, exiting, toggling, and querying PiP mode
- Add custom edge-resize handling for Windows and Linux (MacOs has native support)
- Automatically exit PiP when playback stops
- Support fullscreen <-> PiP transitions and restore window state correctly

## Notes
The [upstream PR](https://github.com/jellyfin-archive/jellyfin-desktop-qt/pull/1193) was closed because Jellyfin-desktop migrated from QT to CEF and the maintainers didn't want to review and roll out such a significant change into the abandoned version. I fully understand their POV. Once the new CEF version is in a stable state, I may re-implement this feature there.

- PiP only activates in video player screen (a video is loaded)
- PiP is blocked on Wayland
- PiP is blocked when `forceAlwaysFS` is enabled (Kiosk mode)
- The interaction between PiP and fullscreen is complicated. It would probably be simpler to disallow PiP <-> fullscreen transitions entirely, but that would require WebView changes and careful handling of shortcuts, including video double-click behavior. The current implementation handles all transitions well enough.
- On macOS, transitioning from PiP back to normal mode can break WebView mouse tracking if the cursor is already inside the restored window bounds. This prevents mouse cursor and webview OSD from reappearing if `mouseIdle` transitioned to `true`. WebView needs a native mouse-enter event to re-establish tracking. The most reliable fix I found was to place the window off-screen, then restore it on the next event-loop tick. This may be avoidable by keeping the PiP window framed, but the frameless PiP behavior is worth the tradeoff in my opinion.

---
# Original Jellyfin Desktop README
> [!WARNING]
> **Deprecated:** Development of this Qt-based desktop client has stopped in favor of a completely rewritten client built on SDL and CEF. The new client can be found at [jellyfin/jellyfin-desktop](https://github.com/jellyfin/jellyfin-desktop).

Jellyfin desktop client built with Qt WebEngine and [libmpv](https://github.com/mpv-player/mpv). Supports audio passthrough, hardware decoding, and playback of more formats without transcoding.

![Screenshot of Jellyfin Desktop](screenshots/video_player.png)

## Downloads
- [Flathub (Linux)](https://flathub.org/apps/details/org.jellyfin.JellyfinDesktop)

### Development Builds
Built from the latest commit on `master`.

#### macOS
- [Apple Silicon](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-macos/master/macos-arm64.zip)
- [Intel](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-macos/master/macos-x86_64.zip)

#### Windows
- [x64 Installer](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-windows/master/windows-x64-installer.zip)
- [x64 Portable](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-windows/master/windows-x64-portable.zip)

#### Linux
- [AppImage (x86_64)](https://nightly.link/jellyfin/jellyfin-desktop/workflows/build-appimage/master/linux-appimage-x86_64.zip)

## Building
See [dev/](dev/) for platform-specific build instructions.

## File Locations
Data is stored per-profile in a `profiles/<profile-id>/` subdirectory. The main configuration file is `jellyfin-desktop.conf`. You can also add `mpv.conf` to configure MPV directly.

**Windows:**
- Config: `%LOCALAPPDATA%\Jellyfin Desktop\profiles\<profile-id>\`
- Cache: `%LOCALAPPDATA%\Jellyfin Desktop\profiles\<profile-id>\`
- Logs: `%LOCALAPPDATA%\Jellyfin Desktop\profiles\<profile-id>\logs\`

**Linux:**
- Config: `~/.local/share/jellyfin-desktop/profiles/<profile-id>/`
- Cache: `~/.cache/jellyfin-desktop/profiles/<profile-id>/`
- Logs: `~/.local/share/jellyfin-desktop/profiles/<profile-id>/logs/`

**Linux (Flatpak):**
- Config: `~/.var/app/org.jellyfin.JellyfinDesktop/data/jellyfin-desktop/profiles/<profile-id>/`
- Cache: `~/.var/app/org.jellyfin.JellyfinDesktop/cache/jellyfin-desktop/profiles/<profile-id>/`
- Logs: `~/.var/app/org.jellyfin.JellyfinDesktop/data/jellyfin-desktop/profiles/<profile-id>/logs/`

**macOS:**
- Config: `~/Library/Application Support/Jellyfin Desktop/profiles/<profile-id>/`
- Cache: `~/Library/Caches/Jellyfin Desktop/profiles/<profile-id>/`
- Logs: `~/Library/Logs/Jellyfin Desktop/<profile-id>/`
