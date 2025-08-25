# GitHub Copilot (Agent mode) — Power Apps PCF (Virtual Field Control)

Purpose

Make the agent proactive for building and iterating on a Power Apps PCF Field control that is virtual, written in TypeScript + TSX, and aligned with the Model-driven App (MDA) theme. The agent must prompt for required inputs, preview the exact pac commands it will run in the VS Code integrated PowerShell terminal, and require explicit confirmation before making environment changes.

Inspiration and references

- Agent working style: https://github.com/scottdurow/PowerAppsCodeAppsDemos/blob/main/ProjectHub/.github/copilot-instructions.md
- Virtual PCFs with Fluent UI 9: https://dianabirkelbach.wordpress.com/2024/12/06/virtual-pcfs-with-fluent-ui-9-after-ga/
- Style Fluent UI 9 PCFs for Power Apps (Model-driven theme): https://dianabirkelbach.wordpress.com/2025/02/25/style-your-fluent-ui-9-pcfs-for-power-apps/
- PCF learning roadmap: https://dianabirkelbach.wordpress.com/2024/04/03/pcf-learning-roadmap/

Agent operating mode

- Be proactive: ask concise questions, propose sensible defaults, and confirm before executing.
- Use the VS Code integrated terminal (PowerShell on Windows) to run pac commands. Prefer the "Power Platform: Run pac command" action when available; otherwise run directly.
- Always show the command list first; only run after the user types an explicit approval: I CONFIRM.
- Stop on the first failing command and summarize the error with a short remediation tip.

What “virtual” means here

- The PCF renders using the host Model-driven App’s React runtime and theme, not its own independent UI library and theme. Keep the bundle lean and avoid shipping a competing design system.

Framework flag (-fw) values

- The pac pcf init -fw option accepts only: react or none.
- For a virtual PCF that uses host MDA React and theme, choose: -fw react.
- "Virtual" is an architectural pattern, not a framework flag.

Interactive flow (questions to ask)

1) Action to perform: init-solution, create-pcf, build, pack, import, push, publish.
2) Solution name (default: MySolution if none detected).
3) Publisher prefix (default: dev).
4) PCF control name (e.g., SampleFieldControl).
5) Namespace (e.g., samples or com.example).
6) Virtual control? (default: yes) — if yes, use -fw react (framework accepts react|none); “virtual” is a pattern, not a flag.
7) Target environment (connection name or org URL) if importing/pushing.
8) Show commands and run now? (yes/no). If org changes: require I CONFIRM.

Project scaffolding and safe local steps (PowerShell)

```powershell
# Initialize a solution folder (safe, local)
pac solution init --publisher-prefix $publisherPrefix --name $solutionName

# Create a virtual field PCF using TSX/React (safe, local)
# Use -fw react for virtual PCFs
pac pcf init --namespace $namespace --name $controlName --template field -fw react

# Install deps and build (safe, local)
cd $controlName
npm install
pac pcf build
```

Org-modifying steps (require explicit confirmation)

```powershell
# Pack a solution zip
pac solution pack --zipFile ${solutionName}.zip

# Import solution into an environment (org-modifying)
pac solution import --path ${solutionName}.zip --target $orgUrl

# Optionally publish changes after import (org-modifying)
pac solution publish
```

TSX / React usage for virtual field controls

- Use TypeScript + TSX. Configure tsconfig (jsx: react or react-jsx). Add react, react-dom, @types/react, @types/react-dom.
- Render a React root in updateView and unmount it in destroy. Pass the PCF context into your root component.
- Prefer consuming host-provided styles/tokens. If you use Fluent UI v9, configure it to inherit the MDA theme (see posts above); avoid bundling theme JSON.
- Keep CSS scoped to the control container; avoid global overrides. Respect light/dark and high-contrast modes.

Manifest guidance (virtual Model-driven field)

- Use the field template; do not declare a dataset for a virtual field control.
- Supported platforms must include ModelDriven.
- Expose only required configuration properties (<property> entries). Provide clear display-name/description.
- Reference the single JS (and optional CSS) bundle in <resources>.

Conceptual control sketch (TSX)

```ts
// index.ts
import * as React from "react";
import { createRoot, Root } from "react-dom/client";
import { App } from "./components/App";

export class FieldControl implements ComponentFramework.StandardControl<IInputs, IOutputs> {
  private container!: HTMLDivElement;
  private root!: Root;

  public init(context) {
    this.container = document.createElement("div");
    context.container.appendChild(this.container);
    this.root = createRoot(this.container);
  }

  public updateView(context: ComponentFramework.Context<IInputs>) {
    this.root.render(React.createElement(App, { context }));
  }

  public getOutputs(): IOutputs { return {}; }

  public destroy(): void { this.root?.unmount(); }
}
```

Agent execution rules

- Show commands before execution; require I CONFIRM for org-modifying actions (import, publish, push).
- If pac is missing, guide the user to install Power Platform CLI and re-check pac --version.
- If auth fails, offer pac auth create/select help and do not proceed with org changes until a profile is selected.
- On failure, stop and summarize the last command, exit code, and a short next step.

Troubleshooting quick tips

- Missing Node/npm: install Node LTS, then rerun npm install.
- TSX build errors: verify tsconfig "jsx" and React versions; ensure bundler outputs the file referenced in manifest.
- Theme mismatch: ensure you use MDA theme tokens/variables; avoid hard-coded colors; test in light/dark/HC.

Quality gates (green-before-done)

- Build passes: pac pcf build returns success.
- Lint/typecheck passes (if present).
- Manifest validates (ModelDriven support, field template, no dataset for virtual).
- Optional smoke: list output artifacts and bundle size; warn if unusually large.

Agent checklist (what to collect from the user)

- Solution name, publisher prefix
- Control name, namespace
- Virtual control? (default yes)
- Target environment (for org changes)
- Actions to run now; final confirmation for org changes

Requirements coverage

- Proactive agent with prompts and confirmations — covered.
- Run pac in VS Code terminal — covered with PowerShell examples.
- Use pac pcf init -fw react for virtual PCF — covered and clarified (valid values: react|none).
- TSX/React usage with virtual MDA React — covered.
- Model-driven theme alignment — covered with references and tips.

-- End of instructions


