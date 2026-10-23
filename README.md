# Highway Rush — Android

The web game (HTML + Canvas + WebAudio) wrapped as a native Android app.
The game runs in a fullscreen, landscape-locked WebView. **No network permission is
required** — everything ships inside the APK.

## Install

```
adb install -r HighwayRush-1.0.apk
```

Or copy `HighwayRush-1.0.apk` to the phone and tap it (you'll need "install from unknown
sources" enabled).

| File | Notes |
| --- | --- |
| `HighwayRush-1.0.apk` | Release build, ~150 KB, signed with the bundled keystore. **Use this one.** |
| `HighwayRush-1.0-debug.apk` | Debug build, larger, for debugging |

| | |
| --- | --- |
| Package | `com.highwayrush.game` |
| Min Android | 7.0 (API 24) |
| Target Android | 15 (API 35) |
| Orientation | Landscape (sensor) |

## Controls

Touch buttons appear automatically on touch devices:

| Where | Control |
| --- | --- |
| Bottom-left / bottom-right corners | **LEFT / RIGHT** arrows — change lane |
| Left / right screen edges | **tap to steer** one lane (hold to repeat); the whole third of the screen is a tap zone |
| Left / right edge, mid height | **BRAKE** and **NITRO** |
| Bottom centre | **Pause** |

Gas is automatic. Keyboard: `A`/`D` or arrows to change lane, `W` gas, `S` brake, `Space`
nitro, `P` or `Esc` pause. Mouse: click the left or right third of the screen to steer.

Settings has a **WEATHER** toggle (clear / rain / storm) — rain darkens the asphalt, adds
reflective lane sheen and drops the headlight visibility.

## Build from source

Requires JDK 17 and the Android SDK (platform 35, build-tools 35). Point the SDK location
at `android/local.properties` via `sdk.dir`.

```powershell
cd android
.\gradlew.bat assembleRelease     # -> app/build/outputs/apk/release/app-release.apk
```

### Editing the game

The playable copy lives at `android/app/src/main/assets/www/`. The root `index.html`, the
three stylesheets (`style.css`, `ui.css`, `game.css`), `js/` and `assets/` are the web
originals — after changing them, copy the whole set across and rebuild:

```powershell
Copy-Item index.html, style.css, ui.css, game.css android\app\src\main\assets\www\
Copy-Item js android\app\src\main\assets\www\ -Recurse
Copy-Item assets android\app\src\main\assets\www\ -Recurse
```

`js/data.gen.js` is generated from `data/*.json`:

```powershell
node tools\build-data.js
```

### Previewing changes

Open `index.html` in a browser. Hash flags jump straight to a state:

| Hash | Effect |
| --- | --- |
| `#play` | start a run immediately |
| `#play&<level>` | start that campaign level (`coast`, `quarry`, `downtown`, …) |
| `#play&<mode>` | start a mode (`timeattack`, `boss`, `endless`) |
| `#play&sim=900` | step 900 fixed frames without waiting for them to render |
| `#garage`, `#campaign`, `#missions`, `#settings`, `#controls`, `#credits`, `#achievements` | open that screen |

Syntax-check every module and grab headless screenshots without leaving the shell:

```powershell
Get-ChildItem js -Filter *.js | ForEach-Object { node --check $_.FullName }
.\tools\screenshot.ps1 -Name menu
.\tools\screenshot.ps1 -Name play -Hash '#play&sim=30000' -Wait 14000
```

The screenshot script also reports anything the page logged to the console, so a broken
screen shows up as `PAGE ERRORS`. `-Touch` renders the on-screen touch pad on desktop for
layout checks, and `-Width`/`-Height` let you check a landscape phone (e.g. `844x390`).
Run the screenshots one at a time — the script reuses a fixed Chrome profile directory.

Keyboard access: <kbd>Tab</kbd> and the arrow keys move through menus, <kbd>Esc</kbd> closes
the confirmation dialog. Screen changes, results and settings are announced to screen
readers, and `prefers-reduced-motion` disables camera shake, hit flashes and the speed-line
overlay.

## Signing

`android/keystore/highwayrush.jks` is a self-signed key generated locally so the release
build is installable straight away (`highwayrush` / `highwayrush`). It's excluded by
`.gitignore`. **Replace it before publishing anywhere** and keep the replacement private.

## App icon

* Adaptive icon — `android/app/src/main/res/mipmap-anydpi-v26/ic_launcher.xml`
  (vector layers in `res/drawable/ic_launcher_{background,foreground}.xml`,
  plus `ic_launcher_monochrome.xml` for Android 13+ themed icons)
* Legacy launcher PNGs — `res/mipmap-*/ic_launcher{,_round}.png`
* Source artwork — `android/tools/icon.svg`, rendered at 1024px and downscaled

Regenerate after editing the artwork:

```powershell
cd android\tools
& "$env:ProgramFiles\Google\Chrome\Application\chrome.exe" --headless --disable-gpu `
    --hide-scrollbars --force-device-scale-factor=1 --default-background-color=00000000 `
    --window-size=1024,1024 --screenshot="$PWD\icon-1024.png" "$PWD/icon-render.html"
.\make-icons.ps1
```

## Layout

```
android/
  app/src/main/
    assets/www/          the whole web game (index.html, *.css, js/, assets/)
    java/.../MainActivity.java
    res/                 icons, themes, strings
  keystore/              signing key
  tools/                 icon source + generator
store/icon-512.png       512x512 Play Store asset
```