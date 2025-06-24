---
title: '为 UE5 配置 VSCode + Clangd 代码补全'
published: 2025-06-24T08:07:20.369Z
description: ''
updated: ''
tags:
  - Note
  - VSCode
  - Clangd
  - UE5
draft: false
pin: 0
toc: true
lang: ''
abbrlink: ''
---

> 主要参考了以下内容：
>
> * [Windows 下使用 Vscode + Clangd 搭建 UE4 开发环境](https://zhuanlan.zhihu.com/p/507625365)
> * [boocs/unreal-clangd](https://github.com/boocs/unreal-clangd)
> * [.code-workspace template file](https://gist.github.com/MrWen33/7b20ee520f28d73280c27309c1fcf23d)
>

原理：UE 官方支持 Clang，可以使用 Clang 生成 `compile_commands.json` 文件，喂给 clangd

## 使用到的 VS Code 插件

* C/C++：C++ 基础插件
* Clangd：用来代码补全和格式化
* C#

## UE 5.4 版本环境依赖

* **Windows**

  * Visual Studio 2022 v17.4 or newer
  * Windows SDK 10.0.18362 or newer
  * **LLVM clang**

    * **Minimum: 15.0.0**
    * **Preferred: 16.0.6**
  * .NET 4.6.2 Targeting Pack
  * .NET 6.0

LLVM 安装过程省略，注意 **UBT 只识别 clang 默认路径**，可以直接安装到默认路径或创建软链接

## UBT 生成 compile_commands.json

在引擎目录下执行：

```powershell
Engine\\Build\\BatchFiles\\Build.bat -help
```

输出内容：

```plaintext
Using bundled DotNet SDK version: 6.0.302
Running UnrealBuildTool: dotnet "..\..\Engine\Binaries\DotNET\UnrealBuildTool\UnrealBuildTool.dll" -help
Global options:
  -Help                :  Display this help.
  -Verbose             :  Increase output verbosity
  -VeryVerbose         :  Increase output verbosity more
  -Log                 :  Specify a log file location instead of the default Engine/Programs/UnrealBuildTool/Log.txt
  -TraceWrites         :  Trace writes requested to the specified file
  -Timestamps          :  Include timestamps in the log
  -FromMsBuild         :  Format messages for msbuild
  -SuppressSDKWarnings :  Missing SDKs error verbosity level will be reduced from warning to log
  -Progress            :  Write progress messages in a format that can be parsed by other programs
  -NoMutex             :  Allow more than one instance of the program to run at once
  -WaitMutex           :  Wait for another instance to finish and then start, rather than aborting immediately
  -RemoteIni           :  Remote tool ini directory
  -Mode=               :  Select tool mode. One of the following (default tool mode is "Build"):
                            AggregateClangTimingInfo, AggregateParsedTimingInfo, Analyze, ApplePostBuildSync, Build,
                            ClRepro, Clean, Deploy, Execute, FixIncludePaths, GenerateClangDatabase, GenerateProjectFiles,
                            IOSPostBuildSync, IWYU, InlineGeneratedCpps, JsonExport, PVSGather, ParseMsvcTimingInfo,
                            PipInstall, PrintBuildGraphInfo, ProfileUnitySizes, Query, QueryTargets, Server, SetupPlatforms,
                            Test, UnrealHeaderTool, ValidatePlatforms, WriteDocumentation, WriteMetadata
  -Clean               :  Clean build products. Equivalent to -Mode=Clean
  -ProjectFiles        :  Generate project files based on IDE preference. Equivalent to -Mode=GenerateProjectFiles
  -ProjectFileFormat=  :  Generate project files in specified format. May be used multiple times.
  -Makefile            :  Generate Linux Makefile
  -CMakefile           :  Generate project files for CMake
  -QMakefile           :  Generate project files for QMake
  -KDevelopfile        :  Generate project files for KDevelop
  -CodeliteFiles       :  Generate project files for Codelite
  -XCodeProjectFiles   :  Generate project files for XCode
  -EddieProjectFiles   :  Generate project files for Eddie
  -VSCode              :  Generate project files for Visual Studio Code
  -VSMac               :  Generate project files for Visual Studio Mac
  -CLion               :  Generate project files for CLion
  -Rider               :  Generate project files for Rider
```

注意参数 `-Mode=GenerateClangDatabase` 即为生成 compile_commands.json

## 配置 VS Code 生成任务

可以放在 `.vscode` 路径下

替换其中 `{ProjectName}` 和 `{UnrealRootPath}` 为具体值，可以参考 `{ProjectName}.code-workspace`

```json
{
  "version": "2.0.0",
  "tasks": [
    {
      "label": "Subtask:GenClangDatabase",
      "group": "none",
      "command": "Engine\\Build\\BatchFiles\\Build.bat",
      "args": [
        "{ProjectName}",
        "Win64",
        "DebugGame",
        "-project=${workspaceFolder}\\VehicleDemo.uproject",
        "-mode=GenerateClangDatabase",
        "-waitmutex"
      ],
      "problemMatcher": "$msCompile",
      "type": "shell",
      "options": {
        "cwd": "{UnrealRootPath}"
      }
    },
    {
      "label": "Subtask:MoveCompileCommands",
      "group": "none",
      "command": "move",
      "args": [
        "compile_commands.json",
        "${workspaceFolder}\\.vscode\\compile_commands.json",
        "-Force"
      ],
      "problemMatcher": "$msCompile",
      "type": "shell",
      "options": {
        "cwd": "{UnrealRootPath}"
      }
    },
    {
      "label": "Generate Compile Commands",
      "group": "build",
      "dependsOn": [
        "Subtask:GenClangDatabase",
        "Subtask:MoveCompileCommands"
      ],
      "dependsOrder": "sequence",
      "problemMatcher": []
    }
  ]
}
```

## 配置 Clangd 和 C/C++ 插件

同样可以放在 `.vscode` 路径下

```json
{
  "[cpp]": {
    "editor.defaultFormatter": "llvm-vs-code-extensions.vscode-clangd"
  },
  "C_Cpp.autocomplete": "disabled",
  "C_Cpp.errorSquiggles": "disabled",
  "C_Cpp.intelliSenseEngine": "disabled",
  "C_Cpp.intelliSenseEngineFallback": "disabled",
  "C_Cpp.formatting": "disabled",
  "clangd.arguments": [
    "--all-scopes-completion=true",
    "--background-index",
    "--clang-tidy",
    "--clang-tidy-checks=performance-*, bugprone-*, misc-*, google-*, modernize-*, readability-*, portability-*",
    "--compile-commands-dir=build",
    "--completion-style=detailed",
    "--enable-config=true",
    "--function-arg-placeholders=false",
    "--header-insertion=never",
    "--log=info",
    "--pch-storage=memory",
    "--pretty",
    "-j=6"
  ],
  "clangd.checkUpdates": false,
  "clangd.path": "clangd"
}
```

## 其他配置

clang-format：

```yaml
---
BasedOnStyle: LLVM
UseTab: Never
BreakBeforeBraces: Allman
ColumnLimit: 120
IndentWidth: 4
SortIncludes: 'false'
AccessModifierOffset: -4
NamespaceIndentation: All
```

gitignore

```yaml
# Unreal Engine 5 gitignore

# Built data and folders
/Binaries/
/DerivedDataCache/
/Intermediate/
/Saved/
/Plugins/*/Binaries/
/Plugins/*/Intermediate/

# Build output
*.exe
*.dll
*.dylib
*.so
*.app

# Crashlytics generated files
/Crashes/
/CrashReportClient/
/Saved/Crashes/
/Saved/Logs/
/Saved/Sandboxes/
/Saved/SaveGames/
/Saved/Config/Windows/

# Cache files for macOS
.DS_Store

# Cache files for Visual Studio
.vs/
*.opensdf
*.sdf
*.suo
*.user
*.userprefs
*.VC.db

# Cache files for Rider
.idea/

# Cache files for Visual Studio Code
.vscode/

# User-specific umaps
*.umap*user

# Other files to ignore
*.swp
*.bak
*.tmp
*.log
/Content/Thumbnails/
/Content/DerivedDataCache/
/Web/
/Build/
/Manifest_NonUFSFiles_Win64.txt
/Manifest_NonUFSFiles_Win32.txt
/Debug/
/*.opensdf
/*.sdf
/*.opendb
/*.VC.db
```
