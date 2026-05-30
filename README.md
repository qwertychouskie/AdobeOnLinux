# Adobe Creative Cloud on Linux (extremely experimental!)

A commmunity effort to run the Adobe Creative Cloud suite of applications on Linux systems.  Everything here is **extremely experimental**, expect things to be rather broken for the time being.

> [!CAUTION]
> The goal of this project is to run unmodified, unhacked Adobe software on Linux, via industry-standard compatibility layers (Wine/Proton/etc).  We will **not** support anyone looking to run pirated versions of Adobe software.  Yes, we know people have been able to run modified versions of some of the Creative Cloud applications on Linux for years; that is **not** what this project is about.

## Basic installation steps

1. Install Wine Staging (11.10 or later)
2. Download the [`winetricks`](winetricks) script from this repository
4. Run with: `WINEPREFIX=~/.local/share/wineprefixes/adobe-creative-cloud /path/to/downloaded/winetricks -q adobe_cc`

## Known issues

- Wine Devel contains some, but not all, of the patches needed.  Please use Wine Staging (11.10 or later), which includes all of the current patches.
- Mouse cursor is invisible in initial installer window: https://bugs.winehq.org/show_bug.cgi?id=58922
- Sometimes you need to open the Creative Cloud application twice for it to launch without crashing/closing

## Individual app test results

| Application                                 | Installs?                 | Runs? |
| ------------------------------------------- | ------------------------- | ----- |
| UXP Developer Tools                         | ✅ Yes                    | ✅ Yes, appears to run as expected |
| Character Animator 2026                     | ✅ Yes                    | ❌ No, [Crashes on launch](https://bugs.winehq.org/show_bug.cgi?id=59311) |
| [Express Photos](EXPRESS_PHOTOS.md)         | ⚠️ No, but has workaround | ❌ No, requires WinUI support, see linked page for details |
| Audition 2020                               | ⚠️ Untested               | ⚠️ Needs re-testing, now that https://bugs.winehq.org/show_bug.cgi?id=50814 is closed |

More testing needed/welcomed!

## Wine upstream PRs/fixes

- [x] [mshtml: Update element event handlers when the corresponding attribute value changes](https://gitlab.winehq.org/wine/wine/-/merge_requests/9976) (released in Wine Devel 11.2)
- [x] [jscript: Fix DISPATCH_METHOD | DISPATCH_PROPERTYGET in ES5+ modes](https://gitlab.winehq.org/wine/wine/-/merge_requests/10004) (released in Wine Devel 11.2)
- [ ] [mshtml/msxml3: Add XMLSerializer, embedded XML declaration handling](https://gitlab.winehq.org/wine/wine/-/merge_requests/10025) (included in Wine Staging 11.2)
  - [x] [mshtml: Add XMLSerializer implementation](https://gitlab.winehq.org/wine/wine/-/merge_requests/10063) (split off code from above PR, merged and released in Wine Devel 11.3)
- [ ] [Human-written patches](https://github.com/sander110419/lightroom-cc-on-linux/issues/2#issuecomment-4476933248) inspired by the vibe-coded patches from https://github.com/sander110419/lightroom-cc-on-linux:
  - [x] https://gitlab.winehq.org/wine/wine/-/merge_requests/10940
  - [x] https://gitlab.winehq.org/wine/wine/-/merge_requests/10941
  - [x] https://gitlab.winehq.org/wine/wine/-/merge_requests/10957
  - [x] https://gitlab.winehq.org/wine/wine/-/merge_requests/10958
  - [x] https://gitlab.winehq.org/wine/wine/-/merge_requests/10969
  - [x] https://gitlab.winehq.org/wine/wine/-/merge_requests/10970
  - [x] https://gitlab.winehq.org/wine/wine/-/merge_requests/10991
- [ ] [Implement SetThreadpoolTimerEx](https://bugs.winehq.org/show_bug.cgi?id=57980#c14) (Included in Wine Staging 11.10)

## Winetricks upstream PRs

- [x] [w_set_app_winver: Fix issue with app names starting with "n" (#2466)](https://github.com/Winetricks/winetricks/pull/2466)
- [x] [webview2: New verb (#2467)](https://github.com/Winetricks/winetricks/pull/2467)
- [ ] TODO: Open draft `adobe_cc` verb PR (perhaps after SetThreadpoolTimerEx is implemented?  Would avoid a ton of the weirdness needed to make things semi-functional)
