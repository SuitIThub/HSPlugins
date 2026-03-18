---
name: build-hsplugins
description: Build any BepInEx plugin in the HSPlugins repository. Automatically detects the target project from context (open files, recent edits, folder structure) or asks the user to select one.
---

# Build HSPlugins

Dynamic build skill for the HSPlugins multi-project repository containing BepInEx plugins for various Illusion games.

## When to Use

Apply this skill when:
- User asks to "build", "compile", or "deploy" a plugin
- After code changes that need to be tested
- User mentions a specific plugin name (e.g., "build Timeline", "compile HSPE")
- When working in any project folder within this repository
- When not prompted by the user, ask first if the project should be built

## Repository Structure

This repository contains multiple BepInEx plugins organized by game platform:

### Game Platforms
| Code | Game | Application |
|------|------|-------------|
| HS | Honey Select | StudioNEO |
| HS2 | Honey Select 2 | StudioNEOV2 |
| AI | AI Shoujo | StudioNEOV2 |
| KK/KOI | Koikatsu | CharaStudio |
| KKS | Koikatsu Sunshine | CharaStudio |
| PH | PlayHome | PlayHomeStudio |

### Plugin Families

| Family | Projects | Description |
|--------|----------|-------------|
| PoseEditor | HSPE, KKPE, KKSPE, AIPE, HS2PE | Pose editing tools |
| Timeline | Timeline.KK, Timeline.KKS, Timeline.AI, Timeline.HS2 | Animation timeline |
| NodesConstraints | NodesConstraints.KK, NodesConstraints.KKS, NodesConstraints.AI, NodesConstraints.HS2 | IK constraints |
| VideoExport | VideoExport.KK, VideoExport.KKS, VideoExport.AI, VideoExport.HS2 | Video recording |
| UsefulStuff | KKUS, KKSUS, AIUS, HS2US | Utility features |
| CollidersDebug | CollidersDebug.KK, CollidersDebug.KKS | Collider visualization |
| MoreAccessories | MoreAccessories, MoreAccessoriesKOI, MoreAccessoriesAI, MoreAccessoriesPH | Extra accessory slots |

### Standalone Plugins (HS only)
- HSIBL - Image-Based Lighting
- HSLRE - Lighting & Render Effects
- HSStandard - Standard shader tools
- RendererEditor - Material/Renderer editing
- LightingEditor - Light editing
- FogEditor - Fog editing
- CameraEditor - Camera editing
- PoseViewer - Pose preview
- BonesFramework - Custom bone support
- HSExtSave - Extended save format
- HSSuimono - Water effects
- Instrumentation - Debug tools

## Project Detection Logic

### Step 1: Check Context

Determine the target project by analyzing (in order of priority):

1. **User's explicit request**: If user names a specific plugin, use that
2. **Currently open/focused file**: Check which project folder the file belongs to
3. **Recently modified files**: Look at the project folder of recent edits
4. **Conversation context**: Analyze any mentioned file paths or plugin names

### Step 2: Map to Build Target

Use this mapping to determine the correct build command:

```
Project Folder → Build Configuration → Output DLL Name
─────────────────────────────────────────────────────────
HSPE           → HS|HS2|KOI|AI|PH   → HSPENeo/HS2PE/KKPE/AIPE/PHPE
KKPE           → Release            → KKPE
KKSPE          → Release            → KKSPE  
AIPE           → Release            → AIPE
HS2PE          → Release            → HS2PE
Timeline.KK    → Release            → Timeline
Timeline.KKS   → Release            → Timeline
Timeline.AI    → Release            → Timeline
Timeline.HS2   → Release            → Timeline
NodesConstraints.* → Release        → NodesConstraints
VideoExport.*  → Release            → VideoExport
KKUS/KKSUS/... → Release            → UsefulStuff
(most others)  → Release/HS         → (varies)
```

### Step 3: Determine Configuration

For platform-specific projects (suffix indicates platform):
- `.HS2` suffix → Configuration: `Release` or `HS2`
- `.AI` suffix → Configuration: `Release` or `AI`
- `.KK` suffix → Configuration: `Release` or `KOI`
- `.KKS` suffix → Configuration: `Release`
- No suffix (HS-only) → Configuration: `HS`

For multi-platform projects (like HSPE):
- Build with the specific platform configuration: `HS`, `HS2`, `AI`, `KOI`, `PH`

## Build Process

### Step 1: Identify Target Project

If the target project cannot be determined from context:

Use the AskQuestion tool with options organized by category:

```
Categories to present:
- Timeline: Timeline.KK, Timeline.KKS, Timeline.AI, Timeline.HS2
- PoseEditor: HSPE (multi-config), KKPE, KKSPE, AIPE, HS2PE
- NodesConstraints: NodesConstraints.KK, NodesConstraints.KKS, NodesConstraints.AI, NodesConstraints.HS2
- VideoExport: VideoExport.KK, VideoExport.KKS, VideoExport.AI, VideoExport.HS2
- UsefulStuff: KKUS, KKSUS, AIUS, HS2US
- CollidersDebug: CollidersDebug.KK, CollidersDebug.KKS
- MoreAccessories: MoreAccessories (HS), MoreAccessoriesKOI, MoreAccessoriesAI, MoreAccessoriesPH
- HS Only: HSIBL, HSLRE, HSStandard, RendererEditor, LightingEditor, etc.
```

### Step 2: Determine Build Configuration

For multi-config projects (like HSPE), ask which platform to target if not clear from context.

### Step 3: Run the Build

Build using MSBuild (recommended for this solution):

```powershell
# For projects with specific .csproj (most common)
dotnet build "ProjectFolder/Project.csproj" -c Release

# For multi-config projects, specify configuration
dotnet build "HSPE/HSPE.csproj" -c HS2

# Or build from solution with specific project
dotnet build HSPlugins.sln -c Release -t:ProjectName
```

### Step 4: Verify Output

Check the build output location based on configuration:
- `bin/HS/` - Honey Select builds
- `bin/HS2/` - Honey Select 2 builds
- `bin/AI/` - AI Shoujo builds
- `bin/KOI/` - Koikatsu builds
- `bin/Release/` - Generic release builds

## Deployment Configuration

Default deployment targets (can be customized):

| Platform | Deploy Path | Application |
|----------|-------------|-------------|
| HS | `D:/Honey Select/BepInEx/plugins/HS_Plugins/` | `D:/Honey Select/StudioNEOV2.exe` |
| HS2 | `D:/Honey Select 2/BepInEx/plugins/HS2_plugins/` | `D:/Honey Select 2/StudioNEOV2.exe` |
| AI | `D:/AI-Syoujyo/BepInEx/plugins/AI_Plugins/` | `D:/AI-Syoujyo/StudioNEOV2.exe` |
| KK | `D:/Koikatu/BepInEx/plugins/KK_Plugins/` | `D:/Koikatu/CharaStudio.exe` |

### Deployment Steps

1. **Check for running process**
   - Before copying, check if the target application is running
   - If running, ask user if it should be closed or wait

2. **Handle existing files**
   - When running in background mode: backup original DLL automatically
   - When running interactively: use AskQuestion to ask if existing DLL should be:
     - Overwritten directly
     - Deactivated (rename to `.dl_`)
   - If a deactivated file exists, offer to rename with incremental number or overwrite

3. **Copy the DLL**
   - Copy from build output to deployment target

4. **Launch application** (if requested)
   - Start the appropriate game/studio application

5. **Monitor log** (if requested)
   - Log location: `{GamePath}/output_log.txt`
   - Success indicator: `[Info   :Advanced Map Search] Background scan finished`
   - Report any errors related to the built plugin

## Example Workflows

### Example 1: Context-Based Detection
User is editing `Timeline.Core/AssemblyInfo.cs` and says "build this"
→ Detect Timeline family → Ask which platform (KK, KKS, AI, HS2)
→ Build `Timeline.{Platform}` with Release configuration

### Example 2: Explicit Request
User says "build HS2PE"
→ Build `HS2PE/HS2PE.csproj` with Release configuration
→ Output: `bin/HS2/HS2PE.dll`

### Example 3: Family Request
User says "build Timeline for HS2"
→ Build `Timeline.HS2/Timeline.HS2.csproj` with Release configuration
→ Output: `bin/HS2/Timeline.dll`

### Example 4: Ambiguous Request
User says "build the pose editor"
→ Multiple options exist (HSPE, KKPE, KKSPE, AIPE, HS2PE)
→ Use AskQuestion to let user select platform

## Troubleshooting

**Build fails with missing references:**
- Run `nuget restore HSPlugins.sln` first
- Check if referenced assembly paths in .csproj are correct

**Build fails with SDK not found:**
- Ensure .NET Framework SDK is installed
- Some projects require .NET Framework 3.5, 4.6, or 4.8

**Platform-specific build errors:**
- Ensure the correct build configuration is selected
- Some projects only build for specific platforms

## Tool Usage

Always use the AskQuestion tool when:
- Multiple projects match the user's request
- The target platform is ambiguous
- Deployment options need user confirmation

When running in background mode:
- Do not ask for feedback during the build and deployment process
- Make reasonable default choices (backup existing files, use Release config)
