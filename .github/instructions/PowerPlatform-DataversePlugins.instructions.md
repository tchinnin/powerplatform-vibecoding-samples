---
applyTo: "Plugins/**"
---

# GitHub Copilot Instructions for Dataverse Plugin Development

These are special instructions for GitHub Copilot (Agent Mode) to follow when helping with Dataverse plugin development.

---

## 📌 Project Context
- This repository is for **Dataverse plugin development**.
- The code is written in **C# (.NET Framework 4.6.2)**.
- Plugins are deployed as **NuGet packages** using **Power Platform CLI (PAC)**, *not* Plugin Registration Tool (PRT).
- The project uses **early bound classes** generated with `pac modelbuilder build`.
- We use **Visual Studio Code** as the main IDE.
- **CRITICAL:** Always use `pac plugin init` to create plugin projects and ensure all plugins extend the generated `PluginBase.cs` class.

---

## 📁 Project Structure

Each plugin project should follow this standard folder structure:

```
Plugins/
└── {ProjectName}/
    ├── {ProjectName}.csproj          # Project file with NuGet package configuration
    ├── {ProjectName}.snk             # Strong name key file for assembly signing
    ├── PluginBase.cs                 # Generated base class from pac plugin init
    ├── Models/                       # Early-bound entity classes
    │   ├── Entities/                 # Generated entity classes
    │   ├── OptionSets/               # Generated option set classes
    │   └── EntityOptionSetEnum.cs    # Option set enumeration
    ├── Helpers/                      # Reusable helper classes
    │   ├── EnvironmentVariableHelper.cs
    │   ├── BusinessUnitHelper.cs
    │   ├── TeamHelper.cs
    │   ├── SecurityRoleHelper.cs
    │   └── CommonHelper.cs
    └── Plugins/                      # Plugin implementations
        ├── EntityNameCreatePlugin.cs
        ├── EntityNameUpdatePlugin.cs
        └── ...
```

### Project File (.csproj) Configuration

The `.csproj` file must include:
- **Target Framework**: `net462`
- **Assembly Signing**: Strong name key file
- **NuGet Package Configuration**: PackageId, Version, Authors, Company, Description
- **Required NuGet Packages**:
  - `Microsoft.CrmSdk.CoreAssemblies` (version 9.0.2.*)
  - `Microsoft.PowerApps.MSBuild.Plugin` (version 1.*)
  - `Microsoft.NETFramework.ReferenceAssemblies` (version 1.0.*)

Example:
```xml
<Project Sdk="Microsoft.NET.Sdk">
  <PropertyGroup>
    <TargetFramework>net462</TargetFramework>
    <SignAssembly>true</SignAssembly>
    <AssemblyOriginatorKeyFile>ProjectName.snk</AssemblyOriginatorKeyFile>
    <AssemblyVersion>1.0.0.0</AssemblyVersion>
    <FileVersion>1.0.0.0</FileVersion>
    <RootNamespace>Company.Product.Feature</RootNamespace>
    <!-- Plugin Assembly ID in Dataverse - used for pac plugin push --pluginId -->
    <PluginId></PluginId>
  </PropertyGroup>

  <PropertyGroup>
    <PackageId>ProjectName</PackageId>
    <Version>$(FileVersion)</Version>
    <Authors>YourTeam</Authors>
    <Company>YourCompany</Company>
    <Description>Project description</Description>
  </PropertyGroup>

  <ItemGroup>
    <PackageReference Include="Microsoft.CrmSdk.CoreAssemblies" Version="9.0.2.*" PrivateAssets="All" />
    <PackageReference Include="Microsoft.PowerApps.MSBuild.Plugin" Version="1.*" PrivateAssets="All" />
    <PackageReference Include="Microsoft.NETFramework.ReferenceAssemblies" Version="1.0.*" PrivateAssets="All" />
  </ItemGroup>
</Project>
```

---

## 🚀 Step-by-Step Development Workflow

### Step 1: Initialize Plugin Project with PAC CLI

**CRITICAL:** Always use `pac plugin init` to create the plugin project structure.

```sh
cd Plugins
pac plugin init --outputDirectory {ProjectName} --author "{AuthorName}"
```

This command generates:
- Project structure with `.csproj` file
- **`PluginBase.cs`** - Essential base class for all plugins
- Strong name key file (`.snk`)
- Sample plugin file
- `.gitignore` and `.vscode` configurations

**After initialization:**
1. Update the `.csproj` file with the correct `RootNamespace`
2. Update NuGet package metadata (Company, Description)
3. Add the `<PluginId>` property (leave empty initially, will be filled after first deployment)

### Step 2: Generate Early-Bound Classes

**WITHOUT SDK Messages (Recommended for cleaner code):**
```sh
pac modelbuilder build \
  --outdirectory {ProjectName}/Models \
  --namespace {Company}.{Product}.{Feature}.Models \
  --environment https://{org}.crm.dynamics.com/ \
  --emitfieldsclasses \
  --generateGlobalOptionSets \
  --entitynamesfilter "table1;table2;table3"
```

**Key Parameters:**
- `--outdirectory`: Where to generate classes (e.g., `./Models`)
- `--namespace`: Root namespace for generated classes
- `--emitfieldsclasses`: Generates Fields helper class for each entity
- `--generateGlobalOptionSets`: Includes option sets
- `--entitynamesfilter`: Semicolon-separated list of tables (quoted)
- **DO NOT use** `--generatesdkmessages` to avoid noise

**Result:**
- `Models/Entities/` - Entity classes (strongly typed)
- `Models/OptionSets/` - Global option sets
- `Models/EntityOptionSetEnum.cs` - Option set enumeration helper

### Step 3: Implement Helper Classes

**Principle: Factorize common functionality into reusable helper classes.**

Create helper classes in the `Helpers/` folder to avoid code duplication across plugins. Organize helpers by **domain/responsibility** to maintain clean architecture.

#### Organizational Guidelines

**Group helpers by functional domain:**
- **BusinessUnit operations** - BU creation, hierarchy navigation, ownership management
- **Team operations** - Team creation, member management, role associations
- **SecurityRole operations** - Role retrieval, permission management, BU-specific role resolution
- **Common utilities** - Shared functions used across multiple domains (ownership, cleanup, validation)
- **Configuration** - Environment variables, settings retrieval

**Key Principles:**
1. **One helper class per domain** - Keep related operations together
2. **Static helper methods** - For stateless utility functions
3. **Consistent method signatures** - Always include `IOrganizationService`, `ITracingService`, and domain-specific parameters
4. **Comprehensive tracing** - Log all major operations for debugging
5. **Error handling** - Let exceptions bubble up to the plugin's catch block

#### Example Helper Organization

```
Helpers/
├── BusinessUnitHelper.cs      # BU-related operations
├── TeamHelper.cs              # Team-related operations  
├── SecurityRoleHelper.cs      # Security role operations
├── CommonHelper.cs            # Shared utilities (ownership, cleanup)
└── EnvironmentVariableHelper.cs  # Configuration retrieval
```

#### Important Pattern: Security Role Resolution

When working with security roles across Business Units, remember:
- **Use `ParentRootRoleId`** to find BU-specific role instances, not role name matching
- Security roles are BU-specific; each BU has its own instance of a role
- Never hardcode role GUIDs; always retrieve dynamically

### Step 4: Implement Plugins

**CRITICAL:** All plugin classes MUST extend the `PluginBase` class generated by `pac plugin init`.

#### Plugin Template
```csharp
using System;
using Microsoft.Xrm.Sdk;
using {GeneratedNamespace}; // Namespace where PluginBase was generated
using {Namespace}.Helpers;
using {Namespace}.Models;

namespace {Namespace}.Plugins
{
    /// <summary>
    /// Plugin description
    /// Execution: PreOperation on Create of entity_name
    /// </summary>
    public class EntityNameCreatePlugin : PluginBase  // MUST extend PluginBase
    {
        public EntityNameCreatePlugin() : base(typeof(EntityNameCreatePlugin))
        {
        }

        protected override void ExecuteDataversePlugin(ILocalPluginContext localPluginContext)
        {
            if (localPluginContext == null)
            {
                throw new ArgumentNullException(nameof(localPluginContext));
            }

            var context = localPluginContext.PluginExecutionContext;
            var service = localPluginContext.PluginUserService;
            var tracingService = localPluginContext.TracingService;

            tracingService.Trace("Plugin execution started");

            // Validate message and entity
            if (context.MessageName != "Create" || 
                context.PrimaryEntityName != EntityName.EntityLogicalName)
            {
                return;
            }

            // Validate input parameters
            if (!context.InputParameters.Contains("Target") || 
                !(context.InputParameters["Target"] is Entity))
            {
                return;
            }

            var targetEntity = (Entity)context.InputParameters["Target"];
            var entity = targetEntity.ToEntity<EntityName>();

            // Track created resources for cleanup
            Guid businessUnitId = Guid.Empty;
            Guid teamId = Guid.Empty;

            try
            {
                // Validate required fields
                if (string.IsNullOrWhiteSpace(entity.RequiredField))
                {
                    throw new InvalidPluginExecutionException("Required field is missing");
                }

                // 1. Retrieve environment variables
                var rootBUId = EnvironmentVariableHelper.GetEnvironmentVariableValue(
                    service, "env_RootBUId", tracingService);

                // 2. Create Business Unit
                businessUnitId = BusinessUnitHelper.CreateBusinessUnit(
                    service, $"BU - {entity.RequiredField}", rootBUId, tracingService);

                // 3. Create Team
                teamId = TeamHelper.CreateOwnerTeam(
                    service, $"Team - {entity.RequiredField}", businessUnitId, tracingService);

                // 4. Get and associate security role
                var rootRoleId = EnvironmentVariableHelper.GetEnvironmentVariableValue(
                    service, "env_RootRoleId", tracingService);
                    
                var targetRoleId = SecurityRoleHelper.GetRoleForBusinessUnit(
                    service, rootRoleId, businessUnitId, tracingService);

                TeamHelper.AssociateSecurityRoleToTeam(
                    service, teamId, targetRoleId, tracingService);

                // 5. Set record ownership
                CommonHelper.SetRecordOwnership(
                    service, targetEntity, businessUnitId, teamId, tracingService);

                tracingService.Trace("Plugin execution completed successfully");
            }
            catch (Exception ex)
            {
                tracingService.Trace($"Error occurred: {ex.Message}");
                tracingService.Trace($"Stack Trace: {ex.StackTrace}");
                
                // Cleanup created resources on error
                CommonHelper.CleanupResources(service, teamId, businessUnitId, tracingService);
                
                throw new InvalidPluginExecutionException(
                    $"An error occurred: {ex.Message}", ex);
            }
        }
    }
}
```

**Key Points:**
- Extend `PluginBase` (generated by pac plugin init)
- Override `ExecuteDataversePlugin` method
- Use `ILocalPluginContext` for services
- Always include cleanup logic in catch block

### Step 5: Increment Package Version

**CRITICAL:** Before building for deployment, **ALWAYS increment the version** in the `.csproj` file.

1. Open the `.csproj` file
2. Update the version numbers:
   ```xml
   <AssemblyVersion>1.0.1.0</AssemblyVersion>
   <FileVersion>1.0.1.0</FileVersion>
   ```
3. The `<Version>` property will automatically use `$(FileVersion)`

**Why increment the version?**
- Ensures the NuGet package is regenerated
- Tracks plugin changes in the environment
- Required for proper deployment and updates
- Prevents deployment conflicts

**Version Naming Convention:**
- **Major.Minor.Patch.Build** (e.g., 1.0.1.0)
- Increment **Patch** for bug fixes
- Increment **Minor** for new features
- Increment **Major** for breaking changes

### Step 6: Build Project

#### Debug Build
```sh
dotnet build
```

#### Release Build (for deployment)

**CRITICAL:** After incrementing the version, use this workflow:
```sh
dotnet clean && dotnet build -c Release
```

**Why use `dotnet clean`?**
- Removes all previously generated files (`bin/`, `obj/`, `.nupkg`)
- Forces complete recompilation
- Ensures the NuGet package is regenerated with latest code
- Guarantees version update is applied

**Output:**
- `bin/Release/net462/{ProjectName}.dll` - Plugin assembly
- `bin/Release/{ProjectName}.{Version}.nupkg` - **NuGet package for deployment**

### Step 7: Deploy Plugin Package

**CRITICAL:** Use `--pluginId` and `--pluginFile` parameters (NOT `--solution-name`):

#### First Deployment (no PluginId yet)

**CRITICAL:** The `--pluginId` parameter is mandatory for `pac plugin push`. For the first deployment, the plugin must be registered manually.

1. **Ask the user to deploy the plugin manually** (using Plugin Registration Tool or Power Platform admin center)
2. **Ask the user for the Plugin ID (GUID)** once the plugin is registered
3. **Update the `.csproj` file automatically** by adding the GUID to the `<PluginId>` property
4. Confirm to the user that the `.csproj` has been updated and future deployments will use `pac plugin push` automatically

#### Subsequent Deployments (with PluginId configured)

**Read the PluginId from .csproj:**

**macOS / Linux (bash/zsh):**
```sh
# Extract PluginId from .csproj
PLUGIN_ID=$(grep -o '<PluginId>[^<]*' {ProjectName}.csproj | sed 's/<PluginId>//')

# Deploy using the extracted ID
pac plugin push \
  --pluginId "$PLUGIN_ID" \
  --pluginFile ./bin/Release/net462/{ProjectName}.dll
```

**Windows (PowerShell):**
```powershell
# Extract PluginId from .csproj
$PLUGIN_ID = Select-String -Path "{ProjectName}.csproj" -Pattern '<PluginId>([^<]+)</PluginId>' | ForEach-Object { $_.Matches.Groups[1].Value }

# Deploy using the extracted ID
pac plugin push `
  --pluginId "$PLUGIN_ID" `
  --pluginFile ./bin/Release/net462/{ProjectName}.dll
```

**Alternative: Cross-platform with dotnet CLI:**
```sh
# Works on all platforms
PLUGIN_ID=$(dotnet msbuild {ProjectName}.csproj -getProperty:PluginId -nologo)
pac plugin push --pluginId "$PLUGIN_ID" -pf ./bin/Release/net462/{ProjectName}.dll
```

**Key Parameters:**
- `--pluginId`: The GUID of the plugin assembly in Dataverse (stored in `.csproj`)
- `--pluginFile` or `-pf`: Path to the compiled DLL file
- **DO NOT use `--solution-name`** - This parameter is not needed for plugin updates

**Benefits of this approach:**
- Plugin ID stored in version control with the code
- No need to ask user for Plugin ID on each deployment
- Direct plugin assembly update
- No solution dependency
- Faster deployment
- Works across all environments

---

## 📝 Coding Standards and Best Practices

### General Principles
1. **Always use `pac plugin init`** - Ensures PluginBase.cs is generated
2. **All plugins extend PluginBase** - Use generated scaffolding
3. **Use strongly typed, early-bound classes** - Never use late binding
4. **Organize code into helpers** - Keep plugins thin, logic in helpers
5. **Use meaningful tracing** - Log every major step
6. **Handle errors gracefully** - Include cleanup/rollback logic
7. **Follow SOLID principles** - Single responsibility per class
8. **Use `nameof()`** - Instead of hardcoded strings

### Specific Patterns

#### Environment Variables
- Store configuration values in Dataverse environment variables
- Retrieve as Guid values, not text
- Use descriptive naming: `{prefix}_{project}_{purpose}`

#### Business Units
- Always create BUs with parent hierarchy
- Use descriptive naming conventions
- Track created BUs for cleanup on error

#### Security Roles
- **Use `ParentRootRoleId`** to find roles in target BUs
- Never hardcode role Guids
- Never match by role name (BU-specific roles have same names)
- Always verify role exists before association

#### Error Handling
- Wrap plugin logic in try-catch
- Log detailed error messages
- Cleanup created resources on error (reverse order)
- Throw `InvalidPluginExecutionException` with user-friendly messages

### Code Organization
- **1 plugin class per file**
- **1 helper class per responsibility**
- **Keep files under 200-300 lines** - Refactor if needed
- **No business logic in ExecuteDataversePlugin()** - Delegate to helpers

---

## ✅ Pre-Deployment Checklist

Before deploying plugins:

- [ ] Project initialized with `pac plugin init`
- [ ] All plugins extend PluginBase.cs
- [ ] Early-bound classes generated without SDK messages
- [ ] All required fields are validated
- [ ] Environment variables are documented
- [ ] Error handling includes cleanup logic
- [ ] Tracing is comprehensive
- [ ] Helper functions are reusable
- [ ] Assembly is signed with strong name key
- [ ] **Version incremented in .csproj file (CRITICAL)**
- [ ] `dotnet clean && dotnet build -c Release` executed
- [ ] `PluginId` configured in .csproj (after first deployment)
- [ ] Deployment command reads `PluginId` from .csproj
- [ ] Deployment uses `--pluginId` and `--pluginFile` (NOT `--solution-name`)

---

## 🙌 How Copilot Should Behave

- **Always use `pac plugin init`** - Never manually create plugin projects
- **Ensure plugins extend PluginBase** - Generated by pac plugin init
- **Always use early-bound classes** - Never suggest late binding
- **Use ParentRootRoleId for security roles** - Not role name matching
- **Suggest helper functions first** - Before writing plugin logic
- **Guide with PAC CLI commands** - Not Plugin Registration Tool
- **ALWAYS remind to increment version** - Before build and deployment
- **Check for `PluginId` in .csproj** - If present, use it; if not, guide first deployment
- **After first deployment, ask user for Plugin ID** - Then update .csproj automatically with the value
- **Update `<PluginId>` in .csproj automatically** - Don't ask user to do it manually
- **Use `--pluginId` and `--pluginFile` for deployment** - Never use `--solution-name`
- **Always run `dotnet clean` before release build** - To ensure package regeneration
- **Ask for missing information** - Environment URL, etc. (but not Plugin ID if in .csproj)
- **Follow the step-by-step workflow** - Don't skip steps
- **Promote clean code** - Suggest refactoring when needed

---

## 🔧 Troubleshooting

### Common Issues

**Issue:** Plugin doesn't have PluginBase.cs  
**Solution:** Reinitialize project with `pac plugin init`

**Issue:** SDK Messages cluttering the project  
**Solution:** Regenerate without `--generatesdkmessages` flag

**Issue:** Security role not found in target BU  
**Solution:** Use `ParentRootRoleId` to find the role, not role name

**Issue:** Plugin fails silently  
**Solution:** Add comprehensive tracing at every step

**Issue:** Deployment fails - solution not found  
**Solution:** Use `--pluginId` and `--pluginFile` instead of `--solution-name`

**Issue:** Plugin not updating despite new deployment  
**Solution:** Increment version in `.csproj` and run `dotnet clean` before build

**Issue:** Cannot find Plugin ID  
**Solution:** Check `.csproj` for `<PluginId>` property first. If empty, check Power Platform admin center > Plugin assemblies, or deploy first time without `--pluginId`

**Issue:** PluginId is empty in .csproj  
**Solution:** This is normal for first deployment. Deploy without `--pluginId`, then retrieve the ID from admin center and add it to .csproj
