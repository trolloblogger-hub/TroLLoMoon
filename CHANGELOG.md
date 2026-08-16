# Changelog

All notable changes to TroLLoMoon are recorded here.

The format follows [Keep a Changelog](https://keepachangelog.com/en/1.1.0/), and this
project uses [semantic versioning](https://semver.org/spec/v2.0.0.html).

---

## 0.2.0 — 2026-08-16

The first public release.

### Added

**Monthly calendar.** A month at a time, with a small Moon disc drawn for every day, the
lunar day and the exact moment it changes, and an accent bar marking days on which a
principal phase falls. Selecting a day shows its details.

**Moonrise, culmination and moonset** for a chosen location, with the altitude the Moon
reaches at culmination and how long it stays above the horizon. Location is picked from a
list of Bulgarian regional centres or entered by hand, and is remembered between runs.

**Moonrise-based lunar days.** The lunar day now follows the definition printed lunar
calendars use — the first lunar day runs from the new moon to the first moonrise, and each
following day begins at the next moonrise. The earlier calendar definition
(`floor(age) + 1`) remains only as a fallback when no location is set.

**Gardening calendar.** The Moon's sign gives the kind of day — root in earth signs, leaf
in water, flower in air, fruit in fire — with the exact time each day changes over, what
tradition holds suitable for each kind, and separate advice for the waxing and waning Moon.
New and full moon days are marked as rest days.

**Traditional descriptions** of all 30 lunar days and of the Moon in each of the 12 signs:
symbol, character of the day, what tradition holds favourable and unfavourable, and notes on
the body, food and dreams. Written for this project, not copied from any lunar-calendar
website.

**Language switcher.** Bulgarian and English, switchable at runtime, including date formats
and decimal and thousands separators.

**Packaging.** A signed MSIX and a fully self-contained portable build.

### Fixed

**Time scale beyond 2017.** `AASDynamicalTime.UTC2TT` stops using the leap-second table
after about 2017 and silently falls back to ΔT, which is a different quantity and, for
current dates, a forecast that reality has diverged from. For 2026 it returns 73.1 s instead
of 69.184 s — a roughly 4 second error in every computed phase moment, growing to 8 s by
2030. TroLLoMoon now computes the difference itself as `(TAI − UTC) + 32.184`.

**Moonrise and moonset.** `AASRiseTransitSet2.CalculateMoon` counts parallax twice: it
applies a topocentric correction to the coordinates and then compares against the geocentric
standard altitude from Meeus chapter 15. The result is about 5.5 minutes late. TroLLoMoon
implements rise and set itself; three algebraically equivalent formulations agree to the
second, and the library's helper matches the incorrect one.

**Language selection no longer prevents startup.** `ApplicationLanguages.PrimaryLanguageOverride`
throws in a self-contained unpackaged build; the exception was not caught broadly enough and
the app crashed before any XAML loaded. Resource resolution follows the thread culture, so
the language still applies.

### Notes

The **Garden** and **Traditions** screens are folk and astrological tradition, not
astronomy, and the app states this on both screens. Everything else is calculated and can be
checked against published sources.

The gardening calendar uses the **tropical** zodiac, as the rest of the app does and as
printed Bulgarian calendars do. The biodynamic school of Maria Thun uses the **sidereal**
zodiac, which currently differs by about 24° and therefore gives different days.

### Verified against

- Meeus, *Astronomical Algorithms* — worked examples 47.a, 48.a, 49.a and 49.b. The phase
  angle agrees to 0.2 arcseconds.
- An independently implemented algorithm — root-finding on chapter 47 and 25 positions
  against the chapter 49 series. Maximum disagreement across 40 phase moments from 1977 to
  2064: 13.7 seconds.
- `astrohoroscope.info` for August 2026 — all four principal phases agree to the minute, and
  the lunar days and zodiac positions match.

328 unit tests.

---

## 0.1.0 — not published

Internal milestone. Solution scaffolding, the astronomy core (phase, illumination, moon age,
lunar day, zodiac sign, principal phase events), the reference-value test suite, and a first
"Today" screen with a Win2D Moon disc.
