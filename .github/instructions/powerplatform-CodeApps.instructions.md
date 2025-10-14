# Power Apps Code Apps Development Guide

This guide covers Power Apps Code Apps-specific setup and integration. For general React development, Fluent UI 9 patterns, routing, and UI best practices, refer to **React-SPA.instructions.md**.

**Power Apps Code Apps Workflow:**
1. Code App project setup (Power Platform-specific)
2. Power Platform SDK integration
3. Data management (mock first, then Power Platform connectors)
4. Build and deployment to Power Platform

**For React Development:** See React-SPA.instructions.md for:
- Folder structure and organization
- Fluent UI 9 components and theming
- React Router implementation
- Performance optimization
- Responsive design patterns
- Accessibility guidelines

---

# 1. Code App Project Setup (Official)

Follow these steps to create a complete Power Apps Code App from scratch.

## Prerequisites

- Power Platform environment with Code Apps enabled
- [Visual Studio Code](https://code.visualstudio.com/)
- [Node.js](https://nodejs.org/) (LTS version)
- [Power Platform Tools for VS Code](https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction)

## Step 0: Create folders

1. Create a folder named `CodeApps` in your development directory
2. Inside `CodeApps`, create a folder named after your app, e.g. `AppName`
   - `AppName` will be replaced by the name of the app. Ask the user to provide it.
3. Inside `AppName`, create a `README.md` with the detail of the CodeApp context.
   - Follow the src/README.md template from React-SPA.instructions.md

## Step 1: Initialize your Vite App

1. Open Visual Studio Code and open a new PowerShell terminal and enter:
   ```powershell
   npm create vite@latest AppName -- --template react-ts
   cd .\CodeApps\AppName
   npm install
   ```

2. If you are asked, agree to install `create-vite`

3. Accept the default package name `AppName` by pressing **Enter**.

4. If you are asked to select a framework, select **React**.

5. If you are asked to select a variant, select **TypeScript**.

6. At this time, the Power SDK requires the port to be 3000 in the default configuration. 

   Install the node type definitions using:
   ```powershell
   npm i --save-dev @types/node
   ```

7. Configure Vite for Power Apps**
  Update `vite.config.ts`:
  ```typescript
  import { defineConfig } from 'vite'
  import react from '@vitejs/plugin-react'
  import path from "path";

  export default defineConfig({
    base: "./",
    server: {
      host: "::",
      port: 3000,
    },
    plugins: [react()],
    resolve: {
      alias: {
        "@": path.resolve(__dirname, "./src"),
      },
    },
  });
  ```

8. Update `tsconfig.json`:
  ```json
  {
    "compilerOptions": {
      "verbatimModuleSyntax": false
    }
  }
  ```

10. Install Dependencies, update `package.json`:
  ```json
  {
    "@fluentui/react-components": "^9.54.0",
    "@fluentui/react-icons": "^2.0.0",
    "@microsoft/power-apps": "^0.3.1",
    "react": "^18.2.0",
    "react-dom": "^18.2.0",
    "typescript": "^5.8.3"
  }
  ```
  **Force to React 18 for Fluent v9 compatibility**

11. Install dependencies using:
   ```powershell
   cd .\CodeApps\AppName
   npm install
   ```

12. Press `Ctrl + C` to stop the local server when you have checked it runs ok.

## Step 2: Initialize your Code App

1. **Check** PAC CLI auth using:
   ```powershell
   pac org who
   ```
   If you are not authenticated, get authenticated first.

3. **Select** your environment using:
   ```powershell
   pac env select -env <URL of your environment>
   ```
   You can also use the Power Platform Tools VS Code Extension to select the Environment

4. **Initialize** your code app using:
   ```powershell
   pac code init --displayName "AppName"
   ```
   Notice that there is now a `power.config.json` file in your project.


6. **Open** the `package.json`, and update the existing line:
   ```json
   "dev": "vite"
   ```
   to be:
   ```json
   "dev": "start pac code run && vite",
   ```
   Save the updated `package.json`.

7. **Add a new file** under the `src` folder named `PowerProvider.tsx` with the following content:

   ```typescript
  import { initialize } from "@microsoft/power-apps/app";
  import { useEffect, type ReactNode } from "react";

  interface PowerProviderProps {
      children: ReactNode;
  }

  export default function PowerProvider({ children }: PowerProviderProps) {
      useEffect(() => {
          const initApp = async () => {
              try {
                  await initialize();
                  console.log('Power Platform SDK initialized successfully');
              } catch (error) {
                  console.error('Failed to initialize Power Platform SDK:', error);
              }
          };
          
          initApp();
      }, []);

      return <>{children}</>;
  }
   ```

8. **Save** the file.

9. **Open** `main.tsx` and add the following import under the existing imports:
   ```typescript
   import PowerProvider from './PowerProvider.tsx'
   ```

10. **Update** `main.tsx` from:
    ```typescript
    <StrictMode>
      <App />
    </StrictMode>,
    ```
    to be:
    ```typescript
    <StrictMode>
      <PowerProvider>
        <App />
      </PowerProvider>
    </StrictMode>,
    ```

11. **Save** the file

12. You can now test the Code App by using:
    ```powershell
    npm run dev
    ```
    This will run the vite server, but also start the Power SDK server.

13. Open the URL provided by the Power SDK Server.
    > **IMPORTANT**: Open the url in the same browser profile as your power platform tenant.

14. You should see the app open and running in Power Apps.

## Step 3: Build and Deploy

### Build your app
```powershell
npm run build
```

### Push your app to Power Platform
```powershell
pac code push
```

## Common Issues and Troubleshooting

- **Port 3000 Required**: Power Apps Code Apps require port 3000
- **PowerProvider Issues**: Ensure PowerProvider.tsx is properly configured
- **Build Errors**: Run `npm run build` before deploying
- **Authentication**: Use same browser profile as Power Platform tenant
- **verbatimModuleSyntax**: Must be set to `false` in `tsconfig.app.json`

## Recommended Folder Architecture

Follow the folder structure defined in **React-SPA.instructions.md** with these Power Apps Code Apps-specific additions:

```
src/
│
├─ README.md             # Project documentation (see React-SPA.instructions.md template)
├─ PowerProvider.tsx     # Power Platform SDK initialization (Code Apps specific)
│
├─ services/             # Data access & API clients
│   ├─ generated/       # Auto-generated by `pac code add-datasource` (Code Apps specific)
│   ├─ apiClient.ts     # Wrapper for fetch/axios for consistency
│   └─ dataService.ts   # Example of manual service abstraction
│
├─ models/               # TypeScript interfaces & types
│   ├─ generated/       # Auto-generated by `pac code add-datasource` (Code Apps specific)
│   └─ domainModels.ts  # Custom domain models, enums, types
│
├─ mockData/             # Local mock data for testing or prototyping
│   ├─ users.mock.json  # JSON mock data files
│   └─ orders.mock.ts   # TypeScript mock data with functions
│
├─ config/               # Environment & app configuration
│   └─ power.config.json # Power Platform configuration (Code Apps specific)
│
└─ ... (other folders from React-SPA.instructions.md)
```

### Power Apps Code Apps-Specific Considerations

#### **Generated vs. Manual Code**
- Keep auto-generated code (from `pac code add-datasource`) in `generated/` subdirectories
- Never manually edit generated files
- Create manual abstractions in parent folders (e.g., `services/dataService.ts`)

#### **PowerProvider.tsx**
- Required at root level for Power Platform SDK initialization
- Must wrap the application in main.tsx

#### **Mock Data Strategy**
- Store all mock data in the `mockData/` folder
- Use the same interfaces/types as live data
- Structure mock data to match Power Platform connector responses exactly
- Include realistic data for comprehensive testing before connecting to live connectors

---

# 3. Power Platform-Specific UI Patterns

> **Note:** For general Fluent UI 9 setup, theming, components, and React patterns, see **React-SPA.instructions.md**.

## Status Indicators for Data Connection

Use badges to indicate whether the app is using mock data or connected to live Power Platform data:
```typescript
import { Badge } from '@fluentui/react-components';

// Mock data mode
<Badge appearance="tint" color="important">
  📋 Demo Mode - Using Mock Data
</Badge>

// Live data mode
<Badge appearance="tint" color="success">
  ✅ Connected to Live Data
</Badge>
```

---

# 4. Data Management Strategy (Power Platform-Specific)

> **Note:** For general React hooks, component patterns, UI components, and performance optimization, see **React-SPA.instructions.md**.

## Development Workflow

1. **Start with mock data** - Build and test all UI functionality
2. **Match API structures** - Ensure mock data matches Power Platform connector responses
3. **Implement side-by-side code** - Keep both mock and live code with clear TODOs
4. **Test thoroughly** - Validate with mock data before switching to live
5. **Gradual migration** - Convert one feature/connector at a time
6. **Maintain fallbacks** - Keep error handling for live data failures

## Mock Data Structure

### Create Mock Data Matching Live APIs
```typescript
// mockData/userData.ts
export interface User {
  Id: string;
  DisplayName: string;
  JobTitle: string;
  Mail: string;
  BusinessPhones: string[];
  OfficeLocation: string;
  Department: string;
}

export const mockUsers: User[] = [
  {
    Id: "user1",
    DisplayName: "John Doe",
    JobTitle: "Software Engineer",
    Mail: "john.doe@contoso.com",
    BusinessPhones: ["+1 (555) 0100"],
    OfficeLocation: "Building 1/1234",
    Department: "Engineering"
  },
  // More mock users...
];

export const searchUsers = (searchTerm: string, pageSize: number = 50): User[] => {
  if (!searchTerm) return mockUsers.slice(0, pageSize);
  
  const term = searchTerm.toLowerCase();
  return mockUsers
    .filter(user => 
      user.DisplayName.toLowerCase().includes(term) ||
      user.JobTitle.toLowerCase().includes(term) ||
      user.Department.toLowerCase().includes(term)
    )
    .slice(0, pageSize);
};
```

## Code Pattern for Mock-to-Live Conversion

### Step 1: Side-by-Side Implementation
```typescript
// Import both mock data and service (one commented)
// TODO: Replace with live Service when connecting to real data
// import { UserService } from '../Services/UserService';
import * as mockData from '../mockData/userData';
```

### Step 2: Function Implementation with TODOs
```typescript
const loadUsers = async (searchTerm: string) => {
  setIsLoading(true);
  try {
    // TODO: Replace with live API call
    // const result = await UserService.SearchUser(searchTerm, 50);
    // if (result.success && result.data) {
    //   setUsers(result.data);
    // }

    // Using mock data for demonstration
    const mockResults = mockData.searchUsers(searchTerm, 50);
    setUsers(mockResults);
    console.log('Using mock data:', mockResults.length, 'users');
    
  } catch (error) {
    console.error('Error loading users:', error);
    setUsers([]);
  } finally {
    setIsLoading(false);
  }
};
```

### Step 3: Conversion to Live Data
```typescript
// Import live service (mock data commented out)
import { UserService } from '../Services/UserService';
// import * as mockData from '../mockData/userData';

const loadUsers = async (searchTerm: string) => {
  setIsLoading(true);
  try {
    // Live API call
    const result = await UserService.SearchUser(searchTerm, 50);
    if (result.success && result.data) {
      setUsers(result.data);
      console.log('Live data loaded:', result.data.length, 'users');
    } else {
      console.error('API error:', result.errorMessage);
      setUsers([]);
    }
  } catch (error) {
    console.error('Error loading users:', error);
    setUsers([]);
  } finally {
    setIsLoading(false);
  }
};
```

## TypeScript Interfaces for Consistency

```typescript
// Shared interface for both mock and live data
export interface UserProfile {
  Id: string;
  DisplayName: string;
  JobTitle: string;
  Mail: string;
  BusinessPhones: string[];
  OfficeLocation: string;
  Department: string;
}

// Service response wrapper
export interface ServiceResponse<T> {
  success: boolean;
  data?: T;
  errorMessage?: string;
}
```

## Error Handling for Live Data

```typescript
const loadLiveData = async () => {
  try {
    setIsLoading(true);
    setError(null);
    
    const result = await Service.FetchData();
    
    if (result.success && result.data) {
      setData(result.data);
    } else {
      setError(result.errorMessage || 'Failed to load data');
      console.error('API error:', result.errorMessage);
    }
  } catch (error) {
    const errorMsg = error instanceof Error ? error.message : 'Unknown error';
    setError(`Network error: ${errorMsg}`);
    console.error('Exception loading data:', error);
  } finally {
    setIsLoading(false);
  }
};
```

## Data Access Service Pattern

### Service Architecture
- Define TypeScript interfaces for all data operations (CRUD, search, pagination)
- Create contracts that both mock and real services will implement
- Ensure strongly-typed parameters and return types for all operations
- Implement singleton factory to manage service instances
- Allow runtime switching between mock and real services

### Mock Services (Initial Implementation)
- Create mock implementations that simulate real data operations
- Include realistic test data and proper pagination/filtering simulation
- Provide comprehensive logging for development debugging
- **Start here - implement mock services first**

### Real Services (Future Implementation)
- **Do not implement initially - focus on mock services**
- **Connector names will be determined later**
- Will integrate with Power Apps generated service classes
- Must maintain same interface as mock services

## Connecting to Power Platform

### Get Connection IDs

List available connections to find the connection ID:

```powershell
pac connection list
```

This will show all available connections with their IDs and names.

### Add Data Source

Once ready to connect to live data:

```powershell
pac code add-data-source -a "connector_name" -c <connectionId>
```

---

# GitHub Copilot Optimization

## Copilot-Friendly Code Patterns

### 1. Clear TODO Comments
```typescript
// TODO: Replace with live UserService when connecting to real data
// const result = await UserService.SearchUser(searchTerm, pageSize);

// Using mock data for demonstration
const mockResults = mockData.searchUsers(searchTerm, 50);
```

### 2. Side-by-Side Examples
```typescript
// Live version (commented during mock phase)
// const result = await Service.FetchData();
// if (result.success && result.data) { setData(result.data); }

// Mock version (active during development)
const mockResults = mockData.fetchData();
setData(mockResults);
```

### 3. Consistent Function Signatures
```typescript
// Mock service
export const searchUsers = (searchTerm: string, pageSize: number): User[] => {...}

// Live service
export const SearchUser = async (searchTerm: string, pageSize: number): Promise<ServiceResponse<User[]>> => {...}
```

### 4. TypeScript Interfaces
```typescript
export interface User {
  Id: string;
  DisplayName: string;
  JobTitle: string;
}

// Works with both mock and live
const user: User = mockData.getUser();
const liveUser: User = await Service.GetUser();
```

### 5. Clear Separation of Concerns
```typescript
// ✅ Good: Clear separation
const DataDisplay = ({ data }) => <div>{/* UI only */}</div>;
const DataContainer = () => {
  const data = useDataService(); // Service logic separate
  return <DataDisplay data={data} />;
};

// ❌ Avoid: Mixed concerns
const Component = () => {
  // UI and data logic mixed together
};
```

## Effective Copilot Prompts

### Converting Mock to Live Data
```
@workspace Convert the mock data in UserPage.tsx to live UserService calls. 
Replace mockData.searchUsers with UserService.SearchUser, update error handling 
for live API failures, and change the status badge to show live connection.
```

### Adding New Features
```
@workspace Add a new page component for displaying team information. Follow the 
pattern with mock data first, include search functionality, responsive card 
layout, and proper loading states.
```

### Implementing Fluent UI Patterns
```
@workspace Create a DataGrid component using Fluent UI v9 with server-side sorting, 
column resizing, and pagination. Follow the patterns in the existing codebase and 
include proper TypeScript interfaces.
```

### Error Handling Updates
```
@workspace Update error handling across all service calls to handle network errors, 
authentication failures, and API rate limiting. Add user-friendly error messages 
using Fluent UI MessageBar component.
```

---

# 5. Local Development & Deployment

## Local Development

Run the development server:

```powershell
npm run dev
```

This starts both the Vite development server and Power SDK server.

Open the URL provided by the Power SDK Server in the same browser profile as your Power Platform tenant.

## Build and Deploy

### Build your app
```powershell
npm run build
```

### Push your app to Power Platform
```powershell
pac code push
```

---

# Summary

## Power Apps Code Apps-Specific Content (This File)
- Power Platform CLI setup and authentication
- Code App project initialization with `pac code init`
- PowerProvider.tsx configuration
- Power Platform SDK integration
- Mock-to-live data conversion patterns for Power Platform connectors
- `pac code add-data-source` for connecting to Power Platform
- Power Platform-specific deployment (`pac code push`)

## React/Fluent UI Content (See React-SPA.instructions.md)
- Complete folder structure and organization
- Fluent UI 9 installation, theming, and components
- React hooks, component patterns, and best practices
- React Router implementation and navigation
- Performance optimization techniques
- Responsive design patterns
- Accessibility guidelines (ARIA, keyboard navigation)
- Loading states, error handling UI patterns
- DataGrid, Card layouts, Search/Filter patterns

**Always refer to React-SPA.instructions.md for general React and Fluent UI development practices.**
