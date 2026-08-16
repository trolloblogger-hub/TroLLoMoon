# 🌙 TroLLoMoon

**A detailed, beautiful and genuinely useful moon calendar for Windows 11.**
Free, open source, and fully offline.

> ⚠️ Early development. The astronomy core and a first "Today" screen are being built.

---

## What it does

TroLLoMoon shows you what the Moon is doing — right now and on any date you pick.

- **Moon phase** — phase name and phase angle for any date
- **Illumination** — the illuminated fraction of the disc, in percent
- **Lunar age and lunar day** — days since the last new moon, and the lunar day (1–30)
- **Phase events** — next and previous new moon, first quarter, full moon, last quarter
- **Zodiac sign of the Moon** — from the Moon's apparent ecliptic longitude (tropical zodiac)
- **A real Moon disc** — drawn on the GPU with Win2D from the actual illumination,
  not a stock image
- **Moonrise, culmination and moonset** for your location, with the altitude the Moon
  reaches at culmination and how long it stays up

Planned for later steps: monthly calendar view, eclipses, super- and micromoons, a
gardening moon calendar, and Bulgarian folk traditions.

### Location

Pick a city from the built-in list or type coordinates by hand. The choice is remembered in
`%LOCALAPPDATA%\TroLLoMoon\settings.json`.

TroLLoMoon deliberately does **not** use the Windows geolocation API: on desktop it resolves
position over WiFi or IP, which means a network request, and it needs an extra capability in
the app manifest. The app asks for `runFullTrust` and nothing else.

**Languages:** Bulgarian and English.

---

## Design principles

These are constraints, not preferences:

| | |
|---|---|
| 🪟 **Windows 11 only** | No cross-platform code, no speculative abstractions |
| 🆓 **Free and open source** | No paid libraries, no paid or key-requiring APIs |
| ✈️ **Fully offline** | Zero network requests for the astronomy. No API keys, no accounts |
| 🎯 **Accuracy first** | The correctness of the numbers matters more than shipping features fast |
| 🎨 **Native Fluent look** | Mica, rounded corners, system accent colour, automatic light/dark |

---

## Accuracy

Accuracy is the point of this project, so it is worth being precise about what that means.

The astronomy is computed with [**AASharp**](https://github.com/jsauve/AASharp), a C# port
of PJ Naughter's AA+ library, which implements the algorithms from Jean Meeus,
*Astronomical Algorithms* (2nd ed.).

Two things that are easy to get wrong and that this project handles deliberately:

**Time scales.** Meeus's algorithms return results in **TT** (Terrestrial Dynamical Time),
not UTC. The difference, ΔT, is currently around 70 seconds. Code that ignores this
produces phase times that are wrong by more than a minute. All time-scale conversion in
TroLLoMoon lives in a single file, `Astronomy/TimeScales.cs`, and nothing else in the
codebase converts between scales.

**Coordinate conventions.** Apparent vs. true coordinates, and geocentric vs. topocentric
positions, are not interchangeable. Each call into AASharp is checked for which one it
expects.

Where a simplification is made, the code carries a `// ТОЧНОСТ:` comment explaining what
the trade-off is and when it stops being acceptable.

### Test tolerances

The unit tests check computed values against reference values from published astronomical
sources, hard-coded as constants (this keeps the test suite offline). **All reference
values are stored in UTC**, which is stated explicitly at each one — published tables mix
UT and TT, and comparing them carelessly manufactures a ~70-second error that isn't there.

| Date range | Tolerance for phase moments | Why |
|---|---|---|
| 1900 – 2025 | **< 30 seconds** | ΔT is measured, not predicted |
| 2026 onward | **< 2 minutes** | ΔT is a *forecast* and grows uncertain |

Meeus's phase algorithm itself is good to roughly ±3–4 seconds for modern dates; the
tolerances above are dominated by the uncertainty in ΔT, not by the algorithm.

---

## Architecture

```
TroLLoMoon.sln
├── src/
│   ├── TroLLoMoon.Core/        class library — no UI dependencies whatsoever
│   │   ├── Astronomy/          wrappers over AASharp, pure functions
│   │   ├── Models/             MoonSnapshot, MoonPhaseEvent, ZodiacSign, ...
│   │   └── Services/           IMoonCalculator, ...
│   └── TroLLoMoon.App/         WinUI 3 — presentation only
│       ├── Views/  ViewModels/  Controls/  Strings/  Assets/
└── tests/
    └── TroLLoMoon.Core.Tests/  xUnit
```

`TroLLoMoon.Core` references nothing from WinUI or the Windows App SDK and builds and
tests standalone. If that ever stops being true, the architecture has gone wrong.

Times are held in **UTC** internally (and Julian Day where AASharp wants it); conversion to
local time happens only at the UI boundary.

---

## Building

**Requirements**

- [.NET SDK 9.0](https://dotnet.microsoft.com/download/dotnet/9.0) (pinned via `global.json`)
- Windows 11 SDK 26100
- Visual Studio 2022/2026 or Build Tools with the Windows App SDK components

**Commands**

```bash
# build everything
dotnet build TroLLoMoon.sln -c Debug

# build just the astronomy core (must succeed on its own, without WinUI)
dotnet build src/TroLLoMoon.Core/TroLLoMoon.Core.csproj

# run the tests
dotnet test tests/TroLLoMoon.Core.Tests/TroLLoMoon.Core.Tests.csproj

# run the app
dotnet run --project src/TroLLoMoon.App/TroLLoMoon.App.csproj
```

The app builds unpackaged and framework-dependent by default, which keeps debugging simple.
It needs the Windows App Runtime 1.8 installed. Note that `WindowsAppSDKSelfContained` +
unpackaged crashes inside `Microsoft.UI.Xaml.dll` before managed code runs — don't enable it
for the normal build.

### Packaging

A portable, fully self-contained build — no prerequisites on the target machine:

```bash
dotnet publish src/TroLLoMoon.App/TroLLoMoon.App.csproj -c Release -r win-x64 --self-contained true -p:WindowsAppSDKSelfContained=true -o artifacts/selfcontained
```

A signed MSIX. `BuildMsix=true` switches the project from unpackaged to packaged; the
thumbprint must belong to a code-signing certificate in your own certificate store:

```bash
dotnet build src/TroLLoMoon.App/TroLLoMoon.App.csproj -c Release -r win-x64 -p:BuildMsix=true -p:GenerateAppxPackageOnBuild=true -p:AppxPackageSigningEnabled=true -p:PackageCertificateThumbprint=YOUR_THUMBPRINT -p:AppxBundle=Never -p:UapAppxPackageBuildMode=SideloadOnly
```

> Never add `PublishSingleFile`, `PublishTrimmed`, or `PublishAot` to either command — see
> the licence note below.

---

## Credits

- **Jean Meeus**, *Astronomical Algorithms* — the algorithms behind everything here
- **PJ Naughter** — the AA+ library
- **AASharp** — the C# port that this project builds on
