---
applyTo: "Plugins/**"
---

# GitHub Copilot Instructions

These are special instructions for GitHub Copilot (Agent Mode) to follow when helping me in this repository.

---

## 📌 Project Context
- This repository is for **Dataverse plugin development**.
- The code is written in **C# (.NET)**.
- Plugins are deployed using **Power Platform CLI (PAC)**, *not* Plugin Registration Tool (PRT).
- The project uses **early bound classes** generated with `pac modelbuilder build`.
- We use **Visual Studio Code** as the main IDE.

---

## 🤖 Copilot Agent Goals

When assisting me, Copilot should:
1. **Suggest strongly typed, early bound code** whenever interacting with Dataverse (no late bound).
2. **Use async/await patterns** where possible.
3. **Follow clean coding practices** (SOLID, dependency injection if needed, minimal business logic in `Execute`).
4. **Use meaningful logging** via `ITracingService`.
5. **Guide me with PAC CLI commands** for build, push, and model generation.
6. **When a PAC CLI command requires information from the user** (such as environment URL or ID, solution name, plugin ID, publisher name, etc.), **prompt the user to provide it, and remember it for subsequent PAC CLI commands** in the session.
7. **For the very first plugin deployment, create a new solution for the plugin and publish it before deploying the plugin assembly.**
8. **When initializing a new project, collect all required information from the developer** to populate the `.github/settings.md` file, including:
   - Development Environment URL
   - Solution Name and Unique Name
   - Publisher Name and Prefix
   - Target Tables/Entities for plugins
   - Assembly Name and Namespace
   - Project structure preferences

---

## 🚀 Development Workflow
Copilot should help me follow this workflow:

0. **Project Setup (first time):**
   - Collect project information to populate `.github/settings.md`
   - Ask for: Environment URL, Solution details, Publisher info, Target entities, Assembly/Namespace names
   - Update settings file with provided information

1. **Initialize project (once):**
   ~~~sh
   pac plugin init --outputDirectory src/Plugins
   ~~~

2. **Generate early bound classes:**
   ~~~sh
   pac modelbuilder build --outputDirectory src/Models --url https://<org>.crm.dynamics.com --authType OAuth --username <user> --entitynamesfilter <table1>;<table2>
   ~~~

3. **Build project:**
   ~~~sh
   dotnet build
   ~~~

4. **Deploy plugin assembly:**
   ~~~sh
   pac plugin push --assembly ./bin/Debug/MyPlugin.dll --pluginSolutionUniqueName MySolution
   ~~~

5. **Test & Debug:** use Dataverse Plugin Trace Logs.

---

## 📝 Coding Standards
- Use `nameof()` instead of hardcoded strings for parameters.
- Always check input parameters for null values.
- Prefer `TryGetValue` when retrieving attributes from the context.
- Keep plugins focused on one responsibility; extract helper services if logic grows.

---

## ✅ Example Snippets

### Example: Plugin Skeleton
~~~csharp
public class AccountCreatePlugin : IPlugin
{
    public void Execute(IServiceProvider serviceProvider)
    {
        var context = (IPluginExecutionContext)serviceProvider.GetService(typeof(IPluginExecutionContext));
        var tracing = (ITracingService)serviceProvider.GetService(typeof(ITracingService));
        var factory = (IOrganizationServiceFactory)serviceProvider.GetService(typeof(IOrganizationServiceFactory));
        var service = factory.CreateOrganizationService(context.UserId);

        try
        {
            if (context.InputParameters.Contains("Target") && context.InputParameters["Target"] is Account account)
            {
                // Example: auto-fill field
                if (string.IsNullOrWhiteSpace(account.AccountNumber))
                {
                    account.AccountNumber = Guid.NewGuid().ToString("N").Substring(0, 8).ToUpper();
                }
            }
        }
        catch (Exception ex)
        {
            tracing.Trace($"Plugin Error: {ex}");
            throw;
        }
    }
}
~~~

### Example: CLI Command to Deploy
~~~sh
pac plugin push --assembly ./bin/Debug/MyPlugin.dll --pluginSolutionUniqueName MySolution
~~~

---

## 🙌 How Copilot Should Behave
- If I ask for Dataverse code, **always suggest early bound with proper types**.
- If I ask about deployment, suggest **PAC CLI commands** instead of PRT.
- If unsure, ask me for clarification instead of guessing.
