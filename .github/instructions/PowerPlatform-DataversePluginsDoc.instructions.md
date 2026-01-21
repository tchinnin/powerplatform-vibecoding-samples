---
applyTo: "Plugins/**/*.md"
---

# GitHub Copilot Instructions for Dataverse Plugin Documentation

These instructions guide the agent in documenting Dataverse .NET plugins developed in the codebase.

---

## 📌 Documentation Context

- This documentation is for **Dataverse C# Plugin projects**
- Each plugin project is located under `Plugins/{ProjectName}/`
- Projects contain:
  - `Helpers/` - Reusable helper classes
  - `Plugins/` - Plugin implementations (standard messages or custom APIs)
  - `{ProjectName}.csproj` - Project file with NuGet configuration

---

## 📁 Documentation Structure

Each plugin project should have **three markdown documentation files** in its root folder:

```
Plugins/
└── {ProjectName}/
    ├── README.md                 # Global project documentation with table of contents
    ├── HELPERS.md                # Helper classes documentation
    ├── PLUGINS.md                # Plugin implementations documentation
    ├── Helpers/                  # Helper source code
    ├── Plugins/                  # Plugin source code
    └── {ProjectName}.csproj      # Project file
```

---

## 📝 Documentation Files Structure

### 1. README.md - Global Project Documentation

This is the main entry point for the plugin project documentation.

#### Structure:

```markdown
# {Project Name} - Dataverse Plugin Package

## Overview
Brief description of the project purpose and scope.

## Package Information

| Property | Value |
|----------|-------|
| **Package ID** | {PackageId from .csproj} |
| **Version** | {Version from .csproj} |
| **Authors** | {Authors from .csproj} |
| **Company** | {Company from .csproj} |
| **Target Framework** | net462 |
| **Assembly Signing** | Yes (.snk) |

## Dependencies

### NuGet Packages
- `Microsoft.CrmSdk.CoreAssemblies` - Version X.X.X
- `Microsoft.PowerApps.MSBuild.Plugin` - Version X.X.X
- `Microsoft.NETFramework.ReferenceAssemblies` - Version X.X.X

### Dependent Assemblies
List any additional assemblies included in the package (if applicable).

## Documentation

### [Helper Classes](HELPERS.md)
- [HelperName1](HELPERS.md#helpername1)
- [HelperName2](HELPERS.md#helpername2)
- ...

### [Plugin Implementations](PLUGINS.md)
- [PluginName1](PLUGINS.md#pluginname1)
- [PluginName2](PLUGINS.md#pluginname2)
- ...

## Deployment

This package is deployed using Power Platform CLI:
\`\`\`sh
pac plugin push --pluginId {PluginPackageId} --pluginFile ./bin/Release/{ProjectName}.{Version}.nupkg
\`\`\`

**Note:** The plugin package ID is provided by the user after initial import into the environment.

## Environment Variables

List of required environment variables (if applicable):
- `env_VariableName1` - Description
- `env_VariableName2` - Description

## Prerequisites

- Dataverse environment with required tables
- Security roles configured
- Environment variables configured
```

---

### 2. HELPERS.md - Helper Classes Documentation

One paragraph per helper class with functional description, algorithm, inputs, and outputs.

#### Structure:

```markdown
# Helper Classes Documentation

This document describes the reusable helper classes used by the plugins in this project.

---

## {HelperClassName}

**File:** `Helpers/{HelperClassName}.cs`

### Functional Description
Clear description of what this helper class does and its purpose in the business context.

### Methods

#### {MethodName1}

**Purpose:** Brief description of what this method does.

**Algorithm:**
1. Step 1 description
2. Step 2 description
3. Step 3 description
...

**Inputs:**
- `IOrganizationService service` - Dataverse service connection
- `ITracingService tracingService` - Tracing service for logging
- `{ParamType} {paramName}` - Parameter description
- ...

**Outputs:**
- **Return Type:** `{ReturnType}`
- **Description:** Description of what is returned

**Exceptions:**
- `InvalidPluginExecutionException` - When/why this is thrown
- `{OtherException}` - When/why this is thrown

**Example Usage:**
\`\`\`csharp
var result = HelperClassName.MethodName(service, param1, param2, tracingService);
\`\`\`

---

[Repeat for each method in the helper class]

---

[Repeat entire section for each helper class]
```

---

### 3. PLUGINS.md - Plugin Implementations Documentation

One paragraph per plugin with functional description, algorithm, registration details, and dependencies.

#### Structure:

```markdown
# Plugin Implementations Documentation

This document describes all plugin implementations in this project.

---

## {PluginClassName}

**File:** `Plugins/{PluginClassName}.cs`

### Functional Description
Clear business-level description of what this plugin accomplishes and why it exists.

### Algorithm
1. **Validation Phase**
   - Validate message name and entity
   - Validate input parameters
   - Validate required fields

2. **Main Logic**
   - Step 1: Description
   - Step 2: Description
   - Step 3: Description
   ...

3. **Error Handling**
   - Cleanup logic if errors occur
   - What resources are rolled back

### Registration Details

#### For Standard Message Plugins:

| Property | Value |
|----------|-------|
| **Message** | Create / Update / Delete / etc. |
| **Primary Entity** | {logical_name} |
| **Execution Mode** | Synchronous / Asynchronous |
| **Execution Order** | {number} |

**Registered Steps:**

| Stage | Image | Image Type | Attributes |
|-------|-------|------------|------------|
| PreValidation / PreOperation / PostOperation | {ImageAlias} | PreImage / PostImage / Both | attribute1, attribute2, ... |

#### For Custom API Plugins:

| Property | Value |
|----------|-------|
| **Custom API Name** | {api_name} |
| **Binding Type** | Entity / Entity Collection / Global |
| **Bound Entity** | {logical_name} (if applicable) |
| **Is Function** | Yes / No |
| **Is Private** | Yes / No |

**Input Parameters:**

| Parameter Name | Type | Required | Description |
|----------------|------|----------|-------------|
| {param1} | String | Yes | Description |
| {param2} | EntityReference | No | Description |

**Output Parameters:**

| Parameter Name | Type | Description |
|----------------|------|-------------|
| {output1} | Boolean | Description |
| {output2} | String | Description |

### Dependencies

**Helper Classes:**
- `{HelperClassName1}` - Used for {purpose}
- `{HelperClassName2}` - Used for {purpose}

**External Libraries:**
- None / List any non-standard libraries

**Environment Variables:**
- `env_VariableName1` - Used for {purpose}
- `env_VariableName2` - Used for {purpose}

**Related Tables:**
- `{table_name1}` - {Read/Write/Create}
- `{table_name2}` - {Read/Write/Create}

### Error Handling

**Common Exceptions:**
- `InvalidPluginExecutionException` - Thrown when {condition}
- Custom validation errors and their messages

**Cleanup Logic:**
- What gets cleaned up on error
- Order of cleanup operations

### Notes

Any additional important information, edge cases, or known limitations.

---

[Repeat for each plugin class]
```

---

## 🚀 Documentation Generation Workflow

### Step 1: Analyze the Project Structure

```sh
# Navigate to the plugin project
cd Plugins/{ProjectName}
```

**Collect information:**
1. Read the `.csproj` file for package metadata
2. List all files in `Helpers/` folder
3. List all files in `Plugins/` folder
4. Identify dependencies from `.csproj`

### Step 2: Generate HELPERS.md

For each helper class in `Helpers/`:
1. Read the source code file
2. Extract class name and purpose from comments/code
3. For each public method:
   - Extract method signature
   - Identify parameters and return type
   - Document the algorithm from code logic
   - Note any exceptions thrown
4. Create method documentation following the template

### Step 3: Generate PLUGINS.md

For each plugin class in `Plugins/`:
1. Read the source code file
2. Extract plugin name and purpose from comments
3. Identify the execution pipeline:
   - Check `MessageName` property
   - Check `PrimaryEntityName` property
   - Determine execution stage from `ExecuteDataversePlugin` logic
   - Identify if it's a Custom API by naming convention or code structure
4. Document the algorithm step-by-step from code
5. List all helper classes used (from `using` statements and method calls)
6. Extract environment variables referenced
7. Identify related tables from entity operations
8. Document input/output parameters for Custom APIs

### Step 4: Generate README.md

1. Extract package information from `.csproj`:
   - PackageId
   - Version
   - Authors
   - Company
2. List NuGet dependencies
3. Create table of contents with links to:
   - Each helper in HELPERS.md
   - Each plugin in PLUGINS.md
4. List required environment variables (aggregate from all plugins)
5. Add deployment instructions

---

## 📋 Best Practices for Documentation

### Writing Style
- Use clear, concise language
- Write in present tense
- Focus on **what** and **why**, not just **how**
- Include business context, not just technical details

### Code Analysis
- Read the actual code - don't guess
- Include all public methods from helpers
- Document exceptions that are thrown
- Note any TODOs or known limitations

### Links and Navigation
- Use relative links between documentation files
- Create anchors for each section (lowercase, hyphens for spaces)
- Ensure table of contents links work

### Maintenance
- Update documentation when code changes
- Keep version numbers in sync with `.csproj`
- Document breaking changes

---

## ✅ Documentation Checklist

Before considering documentation complete:

- [ ] README.md exists with complete package information
- [ ] README.md table of contents links to all helpers and plugins
- [ ] HELPERS.md documents all helper classes
- [ ] Each helper method has inputs, outputs, and algorithm documented
- [ ] PLUGINS.md documents all plugin implementations
- [ ] Each plugin has registration details (steps/images or Custom API details)
- [ ] All dependencies are documented (helpers, environment variables, tables)
- [ ] Error handling and cleanup logic is documented
- [ ] All links between documents work correctly
- [ ] Code examples are syntactically correct
- [ ] Version information matches .csproj file

---

## 🙌 How Copilot Should Behave

When asked to document plugins:

1. **Read the actual code** - Never assume or guess implementation details
2. **Start with README.md** - Create the overview and table of contents first
3. **Document helpers before plugins** - Plugins depend on helpers
4. **Follow the templates exactly** - Consistency is important
5. **Extract from .csproj** - Get accurate version and dependency info
6. **Be thorough** - Document all public methods and plugins
7. **Link everything** - Create navigable documentation
8. **Ask for clarification** - If business context is unclear from code
9. **Update existing docs** - If documentation files already exist
10. **Validate links** - Ensure all internal links work

---

## 📝 Example Command Flow

When user asks to document a plugin project:

1. **Confirm project location:**
   ```
   "I'll document the plugin project at Plugins/{ProjectName}/"
   ```

2. **Read the .csproj file** to get package information

3. **List and read all helper files** in Helpers/

4. **List and read all plugin files** in Plugins/

5. **Create HELPERS.md** with all helper documentation

6. **Create PLUGINS.md** with all plugin documentation

7. **Create README.md** with overview and table of contents

8. **Confirm completion:**
   ```
   "Documentation complete:
   - README.md (project overview)
   - HELPERS.md (X helper classes)
   - PLUGINS.md (Y plugin implementations)"
   ```

---

## 🔧 Troubleshooting

### Issue: Can't determine plugin registration details
**Solution:** Look for comments in the plugin class header, check `MessageName` and stage in code

### Issue: Helper algorithm is complex
**Solution:** Break it down into logical steps, focus on the key operations

### Issue: Custom API parameters unclear
**Solution:** Look for `context.InputParameters` and `context.OutputParameters` usage in code

### Issue: Dependencies not obvious
**Solution:** Check `using` statements and method calls throughout the class

---

## 📚 Additional Notes

- Documentation should be written in the same language as the main project (English or French based on CONTEXT.md)
- Keep business descriptions high-level and technical details in algorithm sections
- When in doubt about business purpose, extract it from code comments or ask the user
- Always include the file path in documentation headers for easy reference
