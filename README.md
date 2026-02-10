# Adobe Creative Cloud on Linux (extremely experimental!)

A commmunity effort to run the Adobe Creative Cloud suite of applications on Linux systems.  Everything here is **extremely experimental**, expect things to be rather broken for the time being.

> [!CAUTION]
> The goal of this project is to run unmodified, unhacked Adobe software on Linux, via industry-standard compatibility layers (Wine/Proton/etc).  We will **not** support anyone looking to run pirated versions of Adobe software.  Yes, we know people have been able to run modified versions of some of the Creative Cloud applications on Linux for years; that is **not** what this project is about.

## Basic installation steps

1. Install Wine Staging 11.2 or later
3. Download the Winetricks script from this repository
4. Run with: `WINEPREFIX=~/.local/share/wineprefixes/adobe-creative-cloud /path/to/downloaded/winetricks -q adobe_cc`

## Known issues

- Wine Devel 11.2 contains some, but not all, of the patches from [@PhialsBasement](https://github.com/PhialsBasement).  Please use Wine Staging 11.2, which includes all of the patches.
- Mouse cursor is invisible in initial installer window: https://bugs.winehq.org/show_bug.cgi?id=58922
- Sometimes you need to open the Creative Cloud application twice for it to launch without crashing/closing
- The installation script installes a partially-broken native msxml3.  The partially-brokenness is actually needed for the Creative Cloud app to launch without hitting https://bugs.winehq.org/show_bug.cgi?id=57980 but will cause some Creative Cloud applications to fail to launch/operate correctly.
  - A potential workaround is to launch the Creative Cloud app, then once it loads, run `WINEPREFIX=~/.local/share/wineprefixes/adobe-creative-cloud winecfg` and set `msxml3` to `builtin`.  Horrible hack, but until the wine bug is fixed, I don't know of a better way.

## Individual app test results

| Application             | Test result |
| ----------------------- | ----------- |
| UXP Developer Tools     | ✅ Appears to run as expected |
| Character Animator 2026 | ❌ [Crashes on launch](https://bugs.winehq.org/show_bug.cgi?id=59311) |
| Express Photos          | ❌ [Installation fails](https://bugs.winehq.org/show_bug.cgi?id=59278) |
| Audition 2020           | ❌ [Crashes on launch](https://bugs.winehq.org/show_bug.cgi?id=50814), but [an applicable PR was merged](https://gitlab.winehq.org/wine/wine/-/merge_requests/9961) and should be in the Wine 11.3 release |

More testing needed/welcomed!

## Wine upstream PRs/fixes

- [x] [mshtml: Update element event handlers when the corresponding attribute value changes](https://gitlab.winehq.org/wine/wine/-/merge_requests/9976) (released in Wine Devel 11.2)
- [x] [jscript: Fix DISPATCH_METHOD | DISPATCH_PROPERTYGET in ES5+ modes](https://gitlab.winehq.org/wine/wine/-/merge_requests/10004) (released in Wine Devel 11.2)
- [ ] [mshtml/msxml3: Add XMLSerializer, embedded XML declaration handling](https://gitlab.winehq.org/wine/wine/-/merge_requests/10025) (included in Wine Staging 11.2)
  - [ ] [mshtml: Add XMLSerializer implementation](https://gitlab.winehq.org/wine/wine/-/merge_requests/10063) (split off code from above PR)

## Winetricks upstream PRs

- [ ] [w_set_app_winver: Fix issue with app names starting with "n" (#2466)](https://github.com/Winetricks/winetricks/pull/2466)
- [ ] [webview2: New verb (#2467)](https://github.com/Winetricks/winetricks/pull/2467)
- [ ] TODO: Open draft `dobbe_cc` verb PR (perhaps after SetThreadpoolTimerEx is implemented?  Would avoid a ton of the weirdness needed to make things semi-functional)
