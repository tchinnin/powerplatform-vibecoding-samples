# Copilot Instructions for Power Platform Projects

Business Application Project using Microsoft Power Platform.
This project will use Microsoft Dataverse as a data platform.
This folder wil store :
- Context
    - Project context
    - Business requirements in agile form
    - Functional Flow diagrams
- Architecture & Design principles documentation
    - Dataverse data modelisation
    - Dataverse security modelisation
- Technical artefacts used in Power Platform Developments
    - Unpacked Power Platform solutions
    - Power Platform ALM artefacts
    - Dataverse Plugins code
    - Power Apps PCF code
    - Power Apps Code Apps
    - Power Pages SPA Websites


## Setup

First step when initializing a new project **is to fill in the CONTEXT.md file.**
Use this instructions to help you fill in the CONTEXT.md file.
The file structure is already provided and will not be changed.
All data will be provided by the user. The agent will ask the user for missing data.

-----

# 1. Global Agent Behavior

## General Behavior

- You are an agent: continue working until the user's request is fully resolved. 
  Only end your turn when you're confident the problem is solved and no further 
  action is required.

- Your thinking should be thorough—it's absolutely fine (and encouraged) if your 
  reasoning is long. Think step by step before and after each action you take.

- Plan extensively before making any function calls. Reflect critically after 
  each one. Avoid chaining function calls without introspection between them, as 
  that can impair insight and decision-making.

- If you're unsure about file contents or the codebase structure, use tools to 
  inspect and read relevant files. Never guess or make assumptions.

- Only make necessary, intentional changes that are either directly requested or 
  clearly required for task completion. Avoid editing unrelated or unclear areas.

## Code Quality and Style

- Prefer simple solutions that are easy to understand and maintain.

- Avoid code duplication: before writing new logic, check if similar 
  functionality already exists in the codebase.

- Only introduce a new pattern or technology if all options for improving the 
  current implementation have been exhausted. If you do introduce something new, 
  make sure to fully remove the old implementation to avoid duplication or 
  confusion.

- Keep the codebase clean and organized. Use consistent patterns and naming 
  conventions where applicable.

- Avoid writing one-off scripts in the main codebase—especially if they are 
  only intended to run once.

- Refactor files when they exceed 200–300 lines of code to preserve modularity 
  and clarity.

- Never overwrite the .env file without asking for and receiving explicit 
  confirmation.

- Follow best practices around formatting and consistency. Use linters, 
  formatters, and style guides where appropriate.

## Coding Workflow

- Stay focused on the parts of the code directly relevant to the current task.

- Do not touch unrelated code, even if it could be improved, unless explicitly 
  instructed to do so.

- Avoid major architectural changes or large refactors unless they are 
  structured, justified, and approved.

- Before making a change, always consider its impact on other parts of the 
  system—downstream dependencies, shared services, and global logic should be 
  reviewed.

- Summarize your reasoning and decision-making if a change affects 
  multiple components.

-----

# 2. Copilot Instructions
Depending on the task requested, you will use one of the following instruction files to get instructions on how to proceed :

| Instruction File | Purpose |
|------------------|---------|
| copilot-instructions.md | For general Copilot usage instructions and CONTEXT.md file |
| Requirements.instructions.md | For documenting and normalizing business requirements in agile form & functional diagrams |
| PowerPlatform-DataverseDesign.instructions.md | For Dataverse data modelisation and security modelisation |
| PowerPlatform-MDAClientSideScript.instructions.md | For Model Driven App Client Side JavaScript development |
| PowerPlatform-DataversePlugins.instructions.md | For Dataverse Plugins code, used as custom steps or custom APIs |
| PowerPlatform-PCF.instructions.md | For Power Apps PCF code |
| PowerPlatform-CodeApps.instructions.md | For Power Apps Code Apps (uses React-SPA.instructions.md for development practices) |
| PowerPlatform-PowerPagesSPA.instructions.md | For Power Pages SPA Websites (uses React-SPA.instructions.md for development practices) |
| React-SPA.instructions.md | For React-based SPA development with Fluent UI 9 (shared by Code Apps and Power Pages SPA) |

# CONTEXT.md

You will help fill in the CONTEXT.md file. If the file is not present. Do not create it. Ask the user to create it.
This file is responsible for explaining the context of the project, with technical and functional aspects.
**This file has to be fully completed before starting any other operation.**
It will be used copilot agent interactions.
File structure is already provided and agent will not change it.
All data will be provided by the user. The agent will ask the user for missing data.

# Diagrams
For all diagrams, you will use mermaid syntax.