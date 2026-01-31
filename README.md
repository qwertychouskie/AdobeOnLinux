# Adobe Creative Cloud on Linux (extremely experimental!)

1. Install Wine Staging 11.1 or later
3. Download the Winetricks script below
4. Run with: `WINEPREFIX=~/.local/share/wineprefixes/adobe-creative-cloud /path/to/downloaded/winetricks -q adobe_cc`

## Known issues

- Wine Staging 11.1 contains some, but not all, of the patches from [@PhialsBasement](https://github.com/PhialsBasement) (https://github.com/ValveSoftware/wine/compare/bleeding-edge...PhialsBasement:wine-adobe-installers:bleeding-edge))
  - Patches included: https://gitlab.winehq.org/wine/wine-staging/-/commit/8090aa9cbe1939b2ab9c44098ea51c8abc913a59
  - All patches available: https://github.com/ValveSoftware/wine/compare/bleeding-edge...PhialsBasement:wine-adobe-installers:bleeding-edge
  - TODO: Make custom build based off Staging 11.1 but including all the Adobe patches?  Though perhaps Staging 11.2 will include all of them.
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
| Audition 2020           | ❌ [Crashes on launch](https://bugs.winehq.org/show_bug.cgi?id=50814), but [an applicable PR was just opened](https://gitlab.winehq.org/wine/wine/-/merge_requests/9961) |

More testing needed/welcomed!
