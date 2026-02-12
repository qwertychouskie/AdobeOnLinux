# Adobe Express Photos

- Installs? ⚠️ No, but has workaround (manually extract msixbundle)
- Runs? No (requires WinUI support in Wine)

## Installation - blocking issues

- [ ] [[Bug 59278] Adobe Express Photos installer crashes on unimplemented function KERNEL32.dll.PackageFullNameFromId](https://bugs.winehq.org/show_bug.cgi?id=59278)
- [ ] [[Bug 59407] Multiple apps require appx/appxbundle support to install](https://bugs.winehq.org/show_bug.cgi?id=59407)

## Running - blocking issues

- [ ] [[Bug 56916] DevExpress WinUI FeatureDemo crashes on start due to missing Microsoft.ui.xaml.dll](https://bugs.winehq.org/show_bug.cgi?id=56916)

# Claude Code investigation (here be vibes, accuracy of anything and everything here on out not guaranteed!)

## Current Status: ❌ Not Compatible

Adobe Express Photos cannot currently run on Wine due to missing Windows App SDK support.

## Package Information

- **Package Type:** MSIX bundle (UWP/Desktop Bridge hybrid)
- **Version:** 3.26.0.0
- **Architecture:** x64
- **Package Size:** 219 MB (compressed), 411 MB (extracted)
- **Runtime:** .NET 8.0.16 (CoreCLR 8.0.1625.21506)

## Technical Details

### Application Architecture

Express Photos is a **Desktop Bridge application** - a Win32/.NET application packaged in a UWP container with modern Windows App SDK dependencies.

**Key Components:**
- `PSExpressBroker.exe` - Full trust broker process (Windows.FullTrustApplication)
- `PSExpressCore.exe` - Sandboxed UI process (windows.partialTrustApplication)
- `PSExpressStartUpService.exe` - Startup service

### Dependencies

Required Windows components:
1. **Microsoft.WindowsAppRuntime.1.6** (Windows App SDK)
   - MinVersion: 6000.373.1641.0
2. **Microsoft.VCLibs.140.00.UWPDesktop**
   - MinVersion: 14.0.33728.0
3. **.NET 8 Runtime** ✅ (Available in Wine via winetricks)

### Manifest Capabilities

```xml
<Capability Name="internetClient" />
<rescap:Capability Name="runFullTrust" />
```

The `runFullTrust` capability indicates this is a full Win32 application, not a sandboxed UWP app.

## Installation Process

### MSIX Extraction (Successful ✅)

Since Wine cannot install MSIX packages, we manually extracted the bundle:

```bash
# 1. Capture MSIX bundle during failed installation
# (Located in C:\adobeTemp\<random>\1\PSExpress\x64_x86.msixbundle)

# 2. Extract the bundle
7z x x64_x86.msixbundle -o./bundle-extracted

# 3. Extract the MSIX package
7z x bundle-extracted/PSExpress_3.26.0.0.msix -o./app-contents

# 4. Copy to Wine prefix
cp -r ./app-contents/* "$WINEPREFIX/drive_c/Program Files/AdobeExpressPhotos/"
```

## Execution Attempts

### Attempt 1: PSExpressBroker.exe

**Error:**
```
0x80040154 (REGDB_E_CLASSNOTREG)
Failed to find library for L"Microsoft.Windows.AppLifecycle.AppInstance"
```

**Stack Trace:**
```
at WinRT.ActivationFactory.Get(String, Guid)
at Microsoft.Windows.AppLifecycle.AppInstance.get__objRef_global__Microsoft_Windows_AppLifecycle_IAppInstanceStatics()
at Microsoft.Windows.AppLifecycle.AppInstance.FindOrRegisterForKey(String)
at PSExpressBroker.Program.Main(String[] args)
```

**Root Cause:** Missing Windows App SDK AppLifecycle component

### Attempt 2: PSExpressCore.exe

**Error:**
```
0x80040154 (REGDB_E_CLASSNOTREG)
Failed to find library for L"Microsoft.UI.Xaml.Application"
```

**Stack Trace:**
```
at WinRT.ActivationFactory.Get(String, Guid)
at Microsoft.UI.Xaml.Application.get__objRef_global__Microsoft_UI_Xaml_IApplicationStatics()
at Microsoft.UI.Xaml.Application.Start(ApplicationInitializationCallback callback)
at PSExpress.Program.Main(String[])
```

**Root Cause:** Missing WinUI 3 (Microsoft.UI.Xaml) component

## Wine Limitations

### Missing WinRT Support

Express Photos uses `RoGetActivationFactory` to activate WinRT components:

```
fixme:combase:RoGetActivationFactory (L"Microsoft.Windows.AppLifecycle.AppInstance", {...}, ...): semi-stub
err:combase:RoGetActivationFactory Failed to find library for L"Microsoft.Windows.AppLifecycle.AppInstance"
```

Wine's implementation is incomplete and cannot locate or activate Windows App SDK components.

### Missing Windows App SDK

The Windows App SDK (formerly Project Reunion) is a modern Windows development framework that includes:
- **AppLifecycle:** Application instance management, activation
- **WinUI 3:** Modern UI framework (Microsoft.UI.Xaml)
- **WindowsAppSDK:** Core runtime libraries
- **WinRT:** Windows Runtime type system

**None of these are implemented in Wine.**

### Related Wine Limitations

1. **MSIX Installation:** Wine cannot install MSIX/AppX packages ([Issue #2225](https://github.com/Winetricks/winetricks/issues/2225))
2. **WinRT Activation:** `RoGetActivationFactory` is a stub
3. **UWP Runtime:** No support for UWP app containers
4. **Windows App SDK:** No implementation exists

## What Works

✅ **.NET 8 Runtime** - Successfully runs via winetricks (`winetricks dotnet8`)
✅ **MSIX Extraction** - Can extract packages with 7zip
✅ **Basic Execution** - The .exe files launch and reach the WinRT activation step
✅ **CoreCLR** - .NET Core runtime initializes correctly

## What Doesn't Work

❌ **WinRT Component Activation** - Cannot load Windows App SDK libraries
❌ **AppLifecycle APIs** - Instance management fails
❌ **WinUI 3** - Modern UI framework not available
❌ **MSIX Installation** - Cannot use native Windows installer

## Comparison: Traditional Win32 vs Windows App SDK

| Feature | Traditional Win32 | Windows App SDK (Express Photos) |
|---------|------------------|----------------------------------|
| API Type | Win32 API | WinRT + Win32 |
| UI Framework | WPF/WinForms | WinUI 3 (XAML) |
| Packaging | MSI/EXE | MSIX/AppX |
| Runtime | .NET Framework | .NET 6+ / Windows App SDK |
| Wine Support | ⚠️ Partial | ❌ None |

## Alternative Approaches Investigated

### 1. MSIX-Packaging Toolkit ❌
- **Tool:** [microsoft/msix-packaging](https://github.com/microsoft/msix-packaging)
- **Result:** Build requires CMake 3.29+ (system has 3.28.3)
- **Alternative:** Used 7zip instead (successful)

### 2. Direct Execution ❌
- Copied extracted files to Wine prefix
- Attempted to run executables directly
- Failed due to Windows App SDK dependency

### 3. .NET Runtime Installation ⚠️
- Installed dotnet8 via winetricks
- .NET runtime works, but doesn't help with WinRT/Windows App SDK

## Recommendations

### For Users
**Do not attempt to install Adobe Express Photos on Wine.** It fundamentally requires Windows App SDK which is not available in Wine.

### For Developers
To make Express Photos work on Wine would require:

1. **Implement WinRT activation in Wine**
   - Full `RoGetActivationFactory` support
   - WinRT type system
   - COM/WinRT interop

2. **Implement Windows App SDK**
   - AppLifecycle APIs
   - WinUI 3 framework
   - WindowsAppSDK runtime

3. **Implement MSIX support**
   - Package deployment
   - AppX registration
   - UWP container management

**Estimated effort:** Months to years of Wine development work.

### Better Alternatives
Focus on Adobe applications that use:
- Traditional Win32 APIs
- WPF/WinForms (not WinUI 3)
- MSI/EXE installers (not MSIX)
- .NET Framework or standalone .NET (not Windows App SDK)

## Existing Wine Forks with WinRT Support

### wine-uwp (Rosentti)

**Repository:** [Rosentti/wine-uwp](https://github.com/Rosentti/wine-uwp)

**What it implements:**
- Various UWP (Universal Windows Platform) APIs
- Multiple `Windows.*` namespaces:
  - `windows.ui.xaml` (UWP XAML - **not** WinUI 3)
  - `windows.applicationmodel`
  - `windows.gaming.*`
  - `windows.storage`
  - `windows.media.*`
  - `windows.devices.*`
  - `windows.globalization`
  - And many more...
- WinRT activation infrastructure
- Custom DXVK build for UWP apps

**Current state:**
- Early development
- Can run some UWP-based games with controller input
- Incomplete input handling (mouse/keyboard)
- XAML implementation incomplete

**Tested applications:**
- Marble Maze demo (works with controller, crashes on targets)
- Minecraft for Windows 10 (hangs after AnalyticsInfo)
- Asphalt 8 (missing TouchCapabilities)
- RetroArch-UWP (missing ICoreApplication2)

**Does it help Express Photos?** ❌ No
- Implements `Windows.UI.Xaml` (UWP XAML)
- Express Photos needs `Microsoft.UI.Xaml` (WinUI 3)
- Different platform generation entirely
- No Windows App SDK (Microsoft.*) namespaces

**Reference value:** ✅ High
- Shows WinRT activation patterns
- Demonstrates Windows.* namespace implementation
- Provides DllGetActivationFactory examples
- Could guide future Windows App SDK work

### WineGDK (Weather-OS)

**Repository:** [Weather-OS/WineGDK](https://github.com/Weather-OS/WineGDK)

**What it implements:**
- Xbox Game Development Kit (GDK) APIs
- Some WinRT components
- XGameRuntime support
- Gaming-focused infrastructure

**Does it help Express Photos?** ❌ No
- Focused on Xbox gaming APIs
- Not general Windows App SDK support

**Reference value:** ⚠️ Limited
- Gaming-specific implementation
- Less relevant for productivity apps

## Open Source Windows App SDK Components

Microsoft is actively working to open source the Windows App SDK:

**Already Open Source:**
- [WinUI 3 (microsoft-ui-xaml)](https://github.com/microsoft/microsoft-ui-xaml) - UI controls library
- [Windows App SDK](https://github.com/microsoft/WindowsAppSDK) - Core framework (partial source)
- [C++/WinRT](https://github.com/microsoft/cppwinrt) - C++ language projection
- [C#/WinRT](https://github.com/microsoft/CsWinRT) - C# language projection

**The Challenge:**
These components are **fundamentally dependent on Windows platform APIs**. The Windows App SDK:
- Cannot be built independently of Windows
- Requires Windows-specific runtime infrastructure
- Depends on DirectComposition, modern COM, and platform services

**Future Potential:**
With Microsoft's [commitment to fully open source Windows App SDK](https://www.thurrott.com/dev/324106/microsoft-to-finally-improve-and-then-open-source-the-windows-app-sdk), a future path exists for Wine to:
1. Use open source SDK as reference implementation
2. Implement WinRT activation layer incrementally
3. Build compatibility layer for modern Windows apps

**Timeline:** 12-24+ months of focused Wine development work

## Wine's Current WinRT Support

**RoGetActivationFactory:**
- ✅ Implemented in Wine mainline (moved from Wine Staging)
- ⚠️ Semi-functional - handles basic activation
- ❌ Missing Windows App SDK library resolution
- Can activate WinRT classes if DLL is registered

**Status in Wine Staging 11.2:**
Wine now includes basic WinRT activation support, but lacks:
- Windows App SDK runtime libraries
- Microsoft.* namespace implementations
- WinUI 3 framework
- Modern composition APIs

## Key Differences: UWP vs Windows App SDK

| Aspect | UWP (wine-uwp implements) | Windows App SDK (Express Photos needs) |
|--------|---------------------------|----------------------------------------|
| Namespace | `Windows.*` | `Microsoft.*` + `Windows.*` |
| UI Framework | Windows.UI.Xaml | Microsoft.UI.Xaml (WinUI 3) |
| Platform | Windows 10+ Store apps | Modern desktop apps |
| Packaging | AppX/MSIX (required) | MSIX (optional, can use EXE) |
| Runtime | UWP container | Win32 + App SDK runtime |
| Wine Support | ⚠️ Partial (wine-uwp) | ❌ None |

**Critical distinction:**
```
wine-uwp implements:  Windows.UI.Xaml     (UWP - Generation 1)
Express Photos needs: Microsoft.UI.Xaml   (WinUI 3 - Generation 2)
                      ↑
                   Different platforms!
```

## Potential Implementation Path

For someone interested in implementing Windows App SDK support in Wine:

1. **Study wine-uwp codebase**
   - Understand WinRT activation patterns
   - Learn Windows.* namespace implementation
   - Study DllGetActivationFactory usage

2. **Implement Microsoft.* namespaces**
   - Start with Microsoft.Windows.AppLifecycle
   - Add Microsoft.UI.Xaml (WinUI 3)
   - Build on existing Wine WinRT support

3. **Add platform dependencies**
   - AppWindow APIs
   - Modern composition
   - Platform integration

4. **Test incrementally**
   - Simple Windows App SDK apps first
   - Gradually add features
   - Test with Express Photos

**Estimated effort:** Many months of development

## Resources

### Wine Projects
- [wine-uwp](https://github.com/Rosentti/wine-uwp) - Wine fork with UWP API implementations
- [WineGDK](https://github.com/Weather-OS/WineGDK) - Wine fork with Xbox GDK support
- [Wine MSIX/AppX Support Issue](https://github.com/Winetricks/winetricks/issues/2225)
- [Bottles MSIX Support Request](https://github.com/bottlesdevs/Bottles/issues/3350)
- [WineHQ: UWP Support Discussion](https://forum.winehq.org/viewtopic.php?t=39125)

### Microsoft Open Source
- [WinUI 3 (microsoft-ui-xaml)](https://github.com/microsoft/microsoft-ui-xaml) - UI controls library
- [Windows App SDK](https://github.com/microsoft/WindowsAppSDK) - Core framework
- [C++/WinRT](https://github.com/microsoft/cppwinrt) - C++ language projection
- [C#/WinRT](https://github.com/microsoft/CsWinRT) - C# language projection
- [Microsoft MSIX Packaging](https://github.com/microsoft/msix-packaging)

### Documentation
- [Windows App SDK Overview](https://learn.microsoft.com/en-us/windows/apps/windows-app-sdk/)
- [Microsoft Open Sourcing Windows App SDK](https://www.thurrott.com/dev/324106/microsoft-to-finally-improve-and-then-open-source-the-windows-app-sdk)
- [Display WinRT UI Objects](https://github.com/MicrosoftDocs/windows-dev-docs/blob/docs/hub/apps/develop/ui-input/display-ui-objects.md)
- [AppWindow Discussion](https://github.com/microsoft/WindowsAppSDK/discussions/1990)

## Investigation Date
February 11-12, 2026

## Tested With
- Wine Staging 11.2 (with Valve's PackageFullNameFromId patch)
- Custom Wine build with WoW64 mode
- .NET 8 Runtime (via winetricks)
- Ubuntu/Debian-based system

## Conclusion

While Adobe Express Photos is technically a Desktop Bridge application (Win32 in UWP container) rather than a pure UWP app, its dependency on Windows App SDK and WinUI 3 makes it incompatible with Wine. The application successfully demonstrates that MSIX packages can be extracted and analyzed, but execution requires WinRT infrastructure that Wine does not provide.

**Status: Not feasible on Wine at this time.**
