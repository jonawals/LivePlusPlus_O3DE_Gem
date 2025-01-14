# LivePlusPlus_O3DE_Gem

An integration of [Live++](https://liveplusplus.tech/) for [Open 3D Engine](https://o3de.org/) ([GitHub](https://github.com/o3de/o3de)) (Windows only).

*Important*: Live++ is not open source. See its website for details: [Pricing](https://liveplusplus.tech/pricing.html) and [Legal](https://liveplusplus.tech/legal_notice.html).

## Installation

1. Let's assume O3DE engine is at `C:\git\o3de` and your project is at `C:\git\YourProject`.
1. Pull down this repository, for example to `C:\git\Gems\LivePlusPlus`.
1. Register it as a gem for your O3DE project (see [Register Gems](https://o3de.org/docs/user-guide/project-config/register-gems/) for details). For example:
    - `C:\git\o3de> .\scripts\o3de.bat register --gem-path C:\git\Gems\LivePlusPlus --external-subdirectory-project-path C:\git\YourProject`.
1. Enable the `LivePlusPlus` gem in your project
    - This can be achieved by adding `LivePlusPlus` to `enabled_gems.json` in the `C:\git\YourProject` directory.
1. Get a copy of [Live++](https://liveplusplus.tech/).
1. Unzip it into `C:\git\Gems\LivePlusPlus\3rdParty\LivePP`.
1. Compile your project and run.

> Note: It may be necessary to adjust the path in `LivePlusPlusWrapper_Windows.cpp` that refers to the `Gems\LivePlusPlus\3rdParty\LivePP` folder (this will depend on what the name and location of the `LivePlusPlus` gem is). For initial setup try using an absolute path for the `fullWidePath` variable.

## Modifying Compiler/Linker Settings for Hotpatching C++ code

1. Go to `o3de\cmake\Platform\Common\MSVC\Configurations_msvc.cmake`
1. Make the following changes:

    ```bash
    PS C:\git\o3de> git diff C:\git\o3de\cmake\Platform\Common\MSVC\Configurations_msvc.cmake
    diff --git a/cmake/Platform/Common/MSVC/Configurations_msvc.cmake b/cmake/Platform/Common/MSVC/Configurations_msvc.cmake
    index 9353e4eb1c..3ae8796b3c 100644
    --- a/cmake/Platform/Common/MSVC/Configurations_msvc.cmake
    +++ b/cmake/Platform/Common/MSVC/Configurations_msvc.cmake
    @@ -68,6 +68,8 @@ ly_append_configurations_options(
            /Zc:inline      # Removes unreferenced functions or data that are COMDATs or only have internal linkage
            /Zc:wchar_t     # Use compiler native wchar_t
            /Zi             # Generate debugging information (no Edit/Continue)
    +        /Z7             # for hot patching C++ code
    +        /Gw             # for hot patching C++ code
        COMPILATION_RELEASE
            /Ox             # Full optimization
            /Ob2            # Inline any suitable function
    @@ -78,8 +80,9 @@ ly_append_configurations_options(
            /NOLOGO             # Suppress Copyright and version number message
            /IGNORE:4099        # 3rdParty linking produces noise with LNK4099
        LINK_NON_STATIC_PROFILE
    -        /OPT:REF            # Eliminates functions and data that are never referenced
    -        /OPT:ICF            # Perform identical COMDAT folding. Redundant COMDATs can be removed from the linker output
    +        /OPT:NOREF            # for hot patching C++ code
    +        /OPT:NOICF            # for hot patching C++ code
    +        /FUNCTIONPADMIN       # for hot patching C++ code
            /INCREMENTAL:NO
            /DEBUG              # Generate pdbs
        LINK_NON_STATIC_RELEASE
    ```

## Configuration

1. Refer to [Live++ documentation](https://liveplusplus.tech/docs/documentation.html).
1. By default, all gems registered for your O3DE project will be hot patched.
1. Optionally, you can set a regex filter to choose which O3DE gems to hot patch in the following file.

    ```c++
    C:\git\Gems\LivePlusPlus\Code\Platform\Windows\LivePlusPlusSystemComponent_Windows.h

    std::regex m_approvedRegex{ ".*" }; // Matches any path
    std::regex m_approvedRegex{".*ImGui.*"}; // Match ImGui gems
    std::regex m_approvedRegex{".*ImGui.*|.*ScriptCanvas.*"}; // Match ImGui and ScriptCanvas gem
    ```

    > Note: This can significantly improve performance of Live++ and his highly encouraged to improve iteration times.

1. By default only AZ Modules are loaded for use with Live++. There are some dynamic libraries that are themselves not AZ Modules. It is possible to load these explicitly in `LivePlusPlusSystemComponent_Windows.cpp` with:

    ```c++
    const char* library = <full-path-to-dll>; // e.g. <path/to>/EditorLib.dll
    wchar_t widePath[1024] = { 0 };
    AZStd::to_wstring(widePath, strlen(library), library);
    lpp::lppEnableModuleAsync(m_livePlusPlus.Get(), widePath);
    ```

## Troubleshooting

1. You can modify `Code\Platform\Windows\platform_windows.cmake` to enable a trace line to see where this gem expects to find Live++.

    ```cmake
    set(livepp_path ${CMAKE_CURRENT_LIST_DIR}/../../../3rdParty/LivePP/API/LPP_API.h)`
    #message(livepp_path="${livepp_path}")
    ```

Enjoy!
