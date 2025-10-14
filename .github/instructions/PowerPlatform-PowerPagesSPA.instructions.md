# Power Pages SPA Development Guide

This guide covers Power Pages SPA-specific setup and integration. For general React development, Fluent UI 9 patterns, routing, and UI best practices, refer to **React-SPA.instructions.md**.

**Power Pages SPA Workflow:**
1. Create React SPA project (standard Vite + React + TypeScript)
2. Configure Power Pages deployment settings
3. Upload and deploy to Power Pages
4. Configure authentication and authorization
5. Use Power Pages Web API for data access

**For React Development:** See React-SPA.instructions.md for:
- Complete folder structure and organization
- Fluent UI 9 components and theming
- React Router implementation
- Performance optimization
- Responsive design patterns
- Accessibility guidelines

---

## Important Notes

> **Preview Feature**: Power Pages SPA sites are currently in preview and subject to change.

> **Client-Side Rendering Only**: Power Pages SPA sites run entirely in the user's browser, unlike traditional Power Pages sites.

> **Source Code Management**: SPA sites are managed only through source code and CLI tools (no visual designer).

---

# 1. Prerequisites

Before you begin, make sure you have:

- A Power Pages environment with [admin privileges](https://learn.microsoft.com/en-us/power-pages/getting-started/create-manage#roles-and-permissions)
- [Power Platform CLI (PAC CLI)](https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction) version 1.44.x or later installed and authenticated
- A Power Pages site on version 9.7.4.x or later
- [Node.js](https://nodejs.org/) (LTS version)
- [Visual Studio Code](https://code.visualstudio.com/)

## Verify PAC CLI Authentication

```powershell
pac org who
```

If you are not authenticated, authenticate first:

```powershell
pac auth create --name "MyProfile" --environment <environment-url>
```

---

# 2. Create Power Pages SPA Project

## Step 1: Initialize React Project

Create a standard React + TypeScript project using Vite:

```powershell
npm create vite@latest your-project-name -- --template react-ts
cd your-project-name
npm install
```

## Step 2: Install Dependencies

Install React Router and Fluent UI 9 (as per React-SPA.instructions.md):

```powershell
npm install react-router-dom
npm install @fluentui/react-components @fluentui/react-icons
npm install react@18 react-dom@18
```

**Note:** React 18 is required for Fluent UI 9 compatibility.

## Step 3: Project Structure

Follow the folder structure defined in **React-SPA.instructions.md**. Your Power Pages SPA project should have:

```
/your-project
│
├─ src/                       ← Your source code (React components, pages, etc.)
├─ build/                     ← Compiled assets (output of `npm run build`)
├─ powerpages.config.json     ← Power Pages configuration file
├─ public/                    ← Static assets
├─ package.json
├─ vite.config.ts
└─ README.md                  ← Project documentation
```

---

# 3. Power Pages Configuration

## Create powerpages.config.json

Create a `powerpages.config.json` file in your project root to customize deployment behavior:

```json
{
  "siteName": "Your SPA Site Name",
  "defaultLandingPage": "index.html",
  "compiledPath": "./build"
}
```

### Configuration Options

| Property | Description | Required |
|----------|-------------|----------|
| `siteName` | Display name for your Power Pages site | No |
| `defaultLandingPage` | Entry point HTML file (usually `index.html`) | No |
| `compiledPath` | Path to compiled assets (relative to root) | No |

**Note:** If you provide command-line arguments, they take precedence over config file values.

---

# 4. Build and Deploy to Power Pages

## Build Your Application

Build your React application for production:

```powershell
npm run build
```

This creates optimized production files in the `build/` or `dist/` folder (depending on your Vite configuration).

## Upload to Power Pages

### Using Configuration File

If you have `powerpages.config.json` configured:

```powershell
pac pages upload-code-site --rootPath "."
```

### Using Command-Line Parameters

```powershell
pac pages upload-code-site `
  --rootPath "." `
  --compiledPath "./build" `
  --siteName "Your SPA Site Name"
```

### Parameters

| Parameter | Alias | Required | Description |
|-----------|-------|----------|-------------|
| `--rootPath` | `-rp` | Yes | Local folder with your site's source files |
| `--compiledPath` | `-cp` | No | Path to compiled assets (e.g., `./build` or `./dist`) |
| `--siteName` | `-sn` | No | Display name for your Power Pages site |

## Activate Your Site

1. Go to [Power Pages](https://make.powerpages.microsoft.com/)
2. Select **Inactive sites**
3. Find your site and select **Reactivate**
4. When the site is active, navigate to your site's URL to verify deployment

> **Tip:** Any subsequent `upload-code-site` command automatically updates the active site without requiring reactivation.

## Download an Existing Site

To download an existing Power Pages SPA site for local editing:

```powershell
pac pages download-code-site `
  --environment "https://your-org.crm.dynamics.com" `
  --path "./downloaded-site" `
  --webSiteId "<site-guid>" `
  --overwrite
```

### Parameters

| Parameter | Alias | Required | Description |
|-----------|-------|----------|-------------|
| `--environment` | `-env` | No | Dataverse environment (GUID or full URL). Defaults to active auth profile |
| `--path` | `-p` | Yes | Local directory to download the site code |
| `--webSiteId` | `-id` | Yes | Website record GUID of the Power Pages SPA site |
| `--overwrite` | `-o` | No | Overwrite existing files in target directory |

---

# 5. Authentication and Authorization

Power Pages SPA sites use the same [security model](https://learn.microsoft.com/en-us/power-pages/security/power-pages-security) as traditional Power Pages sites.

## Configure Identity Providers

### Step 1: Access Security Settings

1. Go to [Power Pages](https://make.powerpages.microsoft.com/)
2. Find your site and select **Edit**
3. Select **Security** > **Identity providers**

### Step 2: Configure Identity Provider

4. Add or set up [identity providers](https://learn.microsoft.com/en-us/power-pages/security/authentication/), such as:
   - Microsoft Entra ID (Azure AD) - automatically configured for new sites
   - Azure AD B2C
   - Other OAuth 2.0 providers

5. Note the **Authority URL** for your identity provider (needed for client-side authentication)

## Access User Context in Code

Power Pages provides a global user object accessible from client-side code:

```typescript
// Access user information
const userInfo = (window as any)["Microsoft"]?.Dynamic365?.Portal?.User;

// User properties
const username = userInfo?.userName ?? "";
const firstName = userInfo?.firstName ?? "";
const lastName = userInfo?.lastName ?? "";
const email = userInfo?.emailAddress ?? "";
const isAuthenticated = username !== "";
```

## Authority URL for Authentication

### Microsoft Entra ID

```typescript
const authorityUrl = `https://login.windows.net/${tenantId}/`;
```

### Other Identity Providers

Find the Authority URL by going to:
- [Power Pages](https://make.powerpages.microsoft.com/) > Your Site > **Security** > **Identity providers** > Configuration settings

## Sample React Authentication Component

```typescript
import React from 'react';
import { Button } from '@fluentui/react-components';

export const AuthButton = () => {
  const userInfo = (window as any)["Microsoft"]?.Dynamic365?.Portal?.User;
  const username = userInfo?.userName ?? "";
  const firstName = userInfo?.firstName ?? "";
  const lastName = userInfo?.lastName ?? "";
  const isAuthenticated = username !== "";
  const [token, setToken] = React.useState<string>("");
  
  // Get tenant ID from environment variables
  const tenantId = import.meta.env.VITE_TENANT_ID;

  React.useEffect(() => {
    const getToken = async () => {
      try {
        const token = await (window as any).shell.getTokenDeferred();
        setToken(token);
      } catch (error) {
        console.error('Error fetching token:', error);
      }
    };
    
    if (isAuthenticated) {
      getToken();
    }
  }, [isAuthenticated]);

  return (
    <div style={{ display: 'flex', alignItems: 'center', gap: '1rem' }}>
      {isAuthenticated ? (
        <>
          <span>Welcome {firstName} {lastName}</span>
          <Button 
            appearance="primary"
            onClick={() => window.location.href = "/Account/Login/LogOff?returnUrl=%2F"}
          >
            Logout
          </Button>
        </>
      ) : (
        <form action="/Account/Login/ExternalLogin" method="post">
          <input 
            name="__RequestVerificationToken" 
            type="hidden" 
            value={token} 
          />
          <Button 
            name="provider" 
            type="submit" 
            appearance="primary"
            value={`https://login.windows.net/${tenantId}/`}
          >
            Login
          </Button>
        </form>
      )}
    </div>
  );
};
```

## Conditional Rendering Based on Authentication

```typescript
import React from 'react';
import { Navigate } from 'react-router-dom';

interface ProtectedRouteProps {
  children: React.ReactNode;
}

export const ProtectedRoute: React.FC<ProtectedRouteProps> = ({ children }) => {
  const userInfo = (window as any)["Microsoft"]?.Dynamic365?.Portal?.User;
  const isAuthenticated = userInfo?.userName !== "";

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return <>{children}</>;
};
```

---

# 6. Use Power Pages Web API

Power Pages SPA sites can leverage [Power Pages Web APIs](https://learn.microsoft.com/en-us/power-pages/configure/web-api-overview) to interact with Dataverse data.

> **⚠️ CRITICAL WARNING: Anonymous Website Limitations**
>
> **Portal Web API in anonymous mode (unauthenticated users) ONLY supports READ operations.**
>
> **You CANNOT perform Write operations (Create, Update, Delete) via Portal Web API for anonymous users.**
>
> **Implications:**
> - ❌ Anonymous contact forms cannot be submitted directly using Portal Web API
> - ❌ Anonymous users cannot create or update records via Web API
> - ❌ Any Create/Update/Delete operations will fail for unauthenticated users
>
> **Alternative Solution for Anonymous Write Operations:**
> - ✅ Use **HTTP Request-triggered Power Automate flows** for anonymous submissions
> - ✅ Call Power Automate from your React SPA instead of Portal Web API
> - ✅ Power Automate can write to Dataverse on behalf of anonymous users
>
> **Pattern for authenticating Power Automate calls will be defined in future updates.**
>
> **For authenticated users:** All CRUD operations (Create, Read, Update, Delete) are supported based on table permissions and web roles.

## Prerequisites for Web API Access

Before using Web APIs, ensure:

1. **Web APIs are enabled** for your Power Pages site
2. **Table permissions** are properly configured
3. **Web roles** are assigned to users appropriately (for authenticated scenarios)

## Anonymous vs Authenticated Web API Usage

### Anonymous (Unauthenticated) Users

**Supported Operations:**
- ✅ **READ only** - Fetch and display data from Dataverse tables

**NOT Supported:**
- ❌ CREATE - Cannot create new records
- ❌ UPDATE - Cannot modify existing records
- ❌ DELETE - Cannot delete records

**Use Cases:**
- Display product catalogs
- Show public information
- List events or news articles
- Display FAQ data

**For Write Operations:** Use Power Automate HTTP-triggered flows (see section below)

### Authenticated Users

**Supported Operations:**
- ✅ CREATE - Create new records (based on table permissions)
- ✅ READ - Fetch data (based on table permissions)
- ✅ UPDATE - Modify records (based on table permissions)
- ✅ DELETE - Remove records (based on table permissions)

**Access Control:**
- Determined by **table permissions** and **web roles**
- Users must be assigned to appropriate web roles
- Scope can be: Global, Contact, Account, Parent, Self

## Enable Web API

1. Go to [Power Pages](https://make.powerpages.microsoft.com/)
2. Select your site > **Set up** > **Table permissions**
3. Configure permissions for the tables you want to access via API

## Fetch Data from Dataverse

```typescript
// Example: Fetch all records from a custom table
const fetchRecords = async () => {
  try {
    const response = await fetch("/_api/cr7ae_creditcardses");
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const data = await response.json();
    const records = data.value;
    
    return records;
  } catch (error) {
    console.error('Error fetching records:', error);
    throw error;
  }
};
```

## Query with Filters

```typescript
// Example: Fetch filtered records
const fetchFilteredRecords = async (category: string) => {
  try {
    const query = `/_api/cr7ae_products?$filter=cr7ae_category eq '${category}'`;
    const response = await fetch(query);
    const data = await response.json();
    
    return data.value;
  } catch (error) {
    console.error('Error fetching filtered records:', error);
    throw error;
  }
};
```

## Create a Record

```typescript
// Example: Create a new record
const createRecord = async (recordData: any) => {
  try {
    const response = await fetch("/_api/cr7ae_products", {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(recordData),
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const newRecord = await response.json();
    return newRecord;
  } catch (error) {
    console.error('Error creating record:', error);
    throw error;
  }
};
```

## Update a Record

```typescript
// Example: Update an existing record
const updateRecord = async (recordId: string, updates: any) => {
  try {
    const response = await fetch(`/_api/cr7ae_products(${recordId})`, {
      method: 'PATCH',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(updates),
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return true;
  } catch (error) {
    console.error('Error updating record:', error);
    throw error;
  }
};
```

## Delete a Record

```typescript
// Example: Delete a record
const deleteRecord = async (recordId: string) => {
  try {
    const response = await fetch(`/_api/cr7ae_products(${recordId})`, {
      method: 'DELETE',
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    return true;
  } catch (error) {
    console.error('Error deleting record:', error);
    throw error;
  }
};
```

## React Hook for Web API

```typescript
import { useState, useEffect } from 'react';

interface UseWebApiOptions {
  endpoint: string;
  autoFetch?: boolean;
}

export const useWebApi = <T,>({ endpoint, autoFetch = true }: UseWebApiOptions) => {
  const [data, setData] = useState<T[]>([]);
  const [isLoading, setIsLoading] = useState<boolean>(false);
  const [error, setError] = useState<string | null>(null);

  const fetchData = async () => {
    setIsLoading(true);
    setError(null);
    
    try {
      const response = await fetch(endpoint);
      
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      
      const result = await response.json();
      setData(result.value);
    } catch (err) {
      const errorMsg = err instanceof Error ? err.message : 'Unknown error';
      setError(errorMsg);
      console.error('Error fetching data:', err);
    } finally {
      setIsLoading(false);
    }
  };

  useEffect(() => {
    if (autoFetch) {
      fetchData();
    }
  }, [endpoint, autoFetch]);

  return { data, isLoading, error, refetch: fetchData };
};

// Usage
const MyComponent = () => {
  const { data, isLoading, error, refetch } = useWebApi<Product>({
    endpoint: '/_api/cr7ae_products',
  });

  if (isLoading) return <Spinner label="Loading..." />;
  if (error) return <div>Error: {error}</div>;

  return (
    <div>
      {data.map(product => (
        <div key={product.cr7ae_productid}>{product.cr7ae_name}</div>
      ))}
    </div>
  );
};
```

## Anonymous Write Operations via Power Automate

Since Portal Web API does not support write operations for anonymous users, use Power Automate HTTP-triggered flows as an alternative.

### Pattern Overview

```
React SPA (Anonymous User)
    ↓
HTTP Request to Power Automate
    ↓
Power Automate Flow
    ↓
Write to Dataverse
```

### Step 1: Create Power Automate Flow

1. Go to [Power Automate](https://make.powerautomate.com/)
2. Create a new **Instant cloud flow**
3. Choose trigger: **When an HTTP request is received**
4. Define the request body JSON schema (example for contact form):

```json
{
  "type": "object",
  "properties": {
    "firstName": {
      "type": "string"
    },
    "lastName": {
      "type": "string"
    },
    "email": {
      "type": "string"
    },
    "message": {
      "type": "string"
    }
  }
}
```

5. Add action: **Dataverse** > **Add a new row**
6. Select your table and map fields from the HTTP request
7. Add action: **Response** to return success/error message
8. Save the flow and copy the **HTTP POST URL**

### Step 2: Call Power Automate from React

```typescript
// Example: Anonymous contact form submission
const submitContactForm = async (formData: {
  firstName: string;
  lastName: string;
  email: string;
  message: string;
}) => {
  try {
    const flowUrl = 'https://prod-xx.eastus.logic.azure.com:443/workflows/.../triggers/manual/paths/invoke?...';
    
    const response = await fetch(flowUrl, {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json',
      },
      body: JSON.stringify(formData),
    });
    
    if (!response.ok) {
      throw new Error(`HTTP error! status: ${response.status}`);
    }
    
    const result = await response.json();
    console.log('Form submitted successfully:', result);
    return result;
  } catch (error) {
    console.error('Error submitting form:', error);
    throw error;
  }
};
```

### Step 3: React Component Example

```typescript
import React, { useState } from 'react';
import { Button, Input, Textarea, MessageBar } from '@fluentui/react-components';

export const AnonymousContactForm = () => {
  const [formData, setFormData] = useState({
    firstName: '',
    lastName: '',
    email: '',
    message: '',
  });
  const [isSubmitting, setIsSubmitting] = useState(false);
  const [submitStatus, setSubmitStatus] = useState<'success' | 'error' | null>(null);

  const handleSubmit = async (e: React.FormEvent) => {
    e.preventDefault();
    setIsSubmitting(true);
    setSubmitStatus(null);

    try {
      await submitContactForm(formData);
      setSubmitStatus('success');
      // Reset form
      setFormData({ firstName: '', lastName: '', email: '', message: '' });
    } catch (error) {
      setSubmitStatus('error');
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      {submitStatus === 'success' && (
        <MessageBar intent="success">
          Thank you! Your message has been submitted successfully.
        </MessageBar>
      )}
      {submitStatus === 'error' && (
        <MessageBar intent="error">
          An error occurred. Please try again later.
        </MessageBar>
      )}
      
      <Input
        placeholder="First Name"
        value={formData.firstName}
        onChange={(e, data) => setFormData({ ...formData, firstName: data.value })}
        required
      />
      <Input
        placeholder="Last Name"
        value={formData.lastName}
        onChange={(e, data) => setFormData({ ...formData, lastName: data.value })}
        required
      />
      <Input
        type="email"
        placeholder="Email"
        value={formData.email}
        onChange={(e, data) => setFormData({ ...formData, email: data.value })}
        required
      />
      <Textarea
        placeholder="Message"
        value={formData.message}
        onChange={(e, data) => setFormData({ ...formData, message: data.value })}
        required
      />
      
      <Button
        appearance="primary"
        type="submit"
        disabled={isSubmitting}
      >
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </Button>
    </form>
  );
};
```

### Security Considerations

> **⚠️ Important:** Power Automate HTTP-triggered flows with anonymous access should be carefully secured.

**Best Practices:**
- Implement rate limiting in Power Automate
- Add data validation in the flow
- Consider using Azure API Management for additional security
- Log all anonymous submissions for audit purposes
- Implement CAPTCHA or similar anti-spam measures in your React form

> **Note:** Authentication patterns for Power Automate calls will be defined in future updates to enhance security.

---

# 7. Local Development with Power Pages Web API

## Enable Local Development

To enable Web API calls from `localhost` during development:

### Step 1: Configure Microsoft Entra ID App

1. Go to [Azure Portal](https://portal.azure.com/)
2. Open the Microsoft Entra app registered for your Power Pages site
3. Enable **Single Page Application (SPA)** authentication
4. Add `http://localhost:<port>/` as a redirect URI using the SPA platform configuration

### Step 2: Add Site Settings in Power Pages

Add these [site settings](https://learn.microsoft.com/en-us/power-pages/configure/configure-site-settings) in your Power Pages site:

1. Go to [Power Pages](https://make.powerpages.microsoft.com/)
2. Select your site > **Set up** > **Site settings**
3. Add the following settings:

```
Authentication/BearerAuthentication/Enabled = true
Authentication/BearerAuthentication/Protocol = OpenIdConnect
Authentication/BearerAuthentication/Provider = AzureAD
```

### Step 3: Configure Vite Proxy (Avoid CORS)

Update your `vite.config.ts` to proxy Web API requests:

```typescript
import { defineConfig } from 'vite';
import react from '@vitejs/plugin-react';
import path from 'path';

export default defineConfig({
  base: "./",
  plugins: [react()],
  resolve: {
    alias: {
      "@": path.resolve(__dirname, "./src"),
    },
  },
  server: {
    proxy: {
      '/_api': {
        target: 'https://your-site.powerappsportals.com',
        changeOrigin: true,
        secure: true,
      },
    },
  },
});
```

### Step 4: Use ADAL.js for Authentication

Install ADAL.js for client-side login:

```powershell
npm install adal-angular
```

> **Important:** Use ADAL.js (not MSAL.js) because Power Pages uses Microsoft Entra v1 endpoints.

### Step 5: Add Authorization Header

Include the Bearer token in all Web API requests:

```typescript
const fetchWithAuth = async (endpoint: string, token: string) => {
  const response = await fetch(endpoint, {
    headers: {
      'Authorization': `Bearer ${token}`,
      'Content-Type': 'application/json',
    },
  });
  
  return response.json();
};
```

### Step 6: Set Site Visibility to Public

Set your site visibility to **Public** to allow `localhost` access during development and testing.

> **Important:** Apply these settings only in development environments. Bearer authentication requires portal version 9.7.6.6 or later.

---

# 8. Power Pages Management Configuration

## Configure Table Permissions

1. Go to [Power Pages](https://make.powerpages.microsoft.com/)
2. Select your site > **Set up** > **Table permissions**
3. Create table permissions for each table you want to expose via Web API:
   - **Table**: Select the Dataverse table
   - **Access type**: Read, Write, Create, Delete, Append, Append To
   - **Scope**: Global, Contact, Account, Parent, Self
4. Associate permissions with **Web Roles**

## Configure Web Roles

1. Go to **Set up** > **Web roles**
2. Create or edit web roles
3. Assign users to appropriate web roles
4. Link table permissions to web roles

## Enable Web API for Tables

1. Go to **Set up** > **Site settings**
2. Add or modify site settings to enable Web API:
   - `Webapi/<table-logical-name>/enabled = true`
   - Example: `Webapi/cr7ae_product/enabled = true`

## Site Visibility Settings

Configure site visibility:
- **Public**: Accessible to everyone (useful for local development)
- **Private**: Requires authentication

## Configure Authentication Settings

1. Go to **Security** > **Identity providers**
2. Configure identity providers (Microsoft Entra ID, Azure AD B2C, etc.)
3. Set up registration, invitation, and login settings
4. Configure external identity provider settings

---

# 9. Key Differences from Traditional Power Pages Sites

| Feature | Traditional Power Pages | Power Pages SPA |
|---------|------------------------|-----------------|
| **Server-side refresh** | Returns specific pages | Always returns the site's root page; client-side router handles sub-routes |
| **Route conflicts** | Server-side routes take precedence | Client-side routes take precedence; hard refresh falls back to root |
| **Page workspace** | Visual designer available | Not supported; use client routing and components |
| **Page-level security** | Configured in portal | Check assigned web roles with global user object; conditionally render UI |
| **Style workspace** | Visual styling editor | Not supported; use framework's styling (CSS, CSS-in-JS, Fluent UI) |
| **Localization** | Multi-language support | Single-language support; implement client-side resource loading |
| **Liquid templating** | Liquid code and templates supported | Not supported; use framework's template engine and Web APIs |
| **SEO support** | Server-rendered pages | Limited (client-side rendering) |
| **Component library** | Out-of-the-box lists, forms, etc. | Build custom components using React and Web APIs |

---

# 10. Best Practices

## Development Workflow

1. **Start with mock data** - Build and test UI locally with mock data
2. **Match Dataverse schema** - Ensure mock data matches your Dataverse table structure
3. **Test locally** - Use Vite dev server with proxy for Web API calls
4. **Build and deploy** - Run `npm run build` and `pac pages upload-code-site`
5. **Test in Power Pages** - Verify functionality with live data in deployed site

## Security Best Practices

- Always check user authentication before displaying sensitive data
- Use table permissions to control data access at the API level
- Implement proper web roles and assign users appropriately
- Validate user permissions on the client side for better UX (server-side validation is enforced)

## Performance Optimization

- Follow React best practices (see React-SPA.instructions.md)
- Implement code splitting with lazy loading
- Use React.memo for expensive components
- Optimize Web API queries with filters and pagination

## Error Handling

```typescript
const handleWebApiError = (error: any) => {
  if (error.status === 401) {
    // Unauthorized - redirect to login
    window.location.href = "/Account/Login";
  } else if (error.status === 403) {
    // Forbidden - user doesn't have permission
    console.error("Access denied. Check table permissions and web roles.");
  } else {
    console.error("Web API error:", error);
  }
};
```

---

# 11. Troubleshooting

## Common Issues

### Site Not Appearing After Upload

**Solution:**
1. Go to [Power Pages](https://make.powerpages.microsoft.com/)
2. Check **Inactive sites**
3. Reactivate your site

### Web API Returns 401 Unauthorized

**Causes:**
- User not authenticated
- Bearer authentication not enabled
- Incorrect identity provider configuration

**Solution:**
- Ensure user is logged in
- Verify site settings for Bearer authentication
- Check identity provider configuration

### Web API Returns 403 Forbidden

**Causes:**
- Missing table permissions
- User not assigned to appropriate web role

**Solution:**
- Configure table permissions for the table
- Assign user to a web role with appropriate permissions

### CORS Errors During Local Development

**Solution:**
- Configure Vite proxy in `vite.config.ts` (see section 7)
- Ensure site visibility is set to Public for development

### Changes Not Reflected After Upload

**Solution:**
- Clear browser cache
- Perform a hard refresh (Ctrl+F5)
- Verify the build folder contains updated files

### Anonymous Write Operations Failing

**Cause:**
- Portal Web API does not support Create/Update/Delete for anonymous users

**Solution:**
- Use Power Automate HTTP-triggered flows for write operations
- See section "Anonymous Write Operations via Power Automate"
- Ensure flow has proper permissions to write to Dataverse

### Power Automate Flow Returns 401 or 403

**Causes:**
- Flow connection doesn't have permissions
- Anonymous access not properly configured

**Solution:**
- Check the Dataverse connection in Power Automate
- Ensure the connection has appropriate permissions
- Verify the flow is enabled and published

---

# 12. Summary

## Power Pages SPA-Specific Content (This File)

- Power Pages prerequisites and environment setup
- `powerpages.config.json` configuration
- PAC CLI commands (`pac pages upload-code-site`, `pac pages download-code-site`)
- Power Pages authentication and authorization
- Accessing user context via `window["Microsoft"].Dynamic365.Portal.User`
- Power Pages Web API integration and CRUD operations
- **Anonymous vs Authenticated Web API usage (READ-only for anonymous)**
- **Anonymous write operations via Power Automate HTTP-triggered flows**
- **Critical limitations: Anonymous users cannot Create/Update/Delete via Portal Web API**
- Local development with Entra ID authentication
- Power Pages management configuration (table permissions, web roles, site settings)
- Key differences from traditional Power Pages sites

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

---

## Additional Resources

- [Create and deploy a single-page application in Power Pages](https://learn.microsoft.com/en-us/power-pages/configure/create-code-sites)
- [Power Pages Web API Overview](https://learn.microsoft.com/en-us/power-pages/configure/web-api-overview)
- [Power Pages Security](https://learn.microsoft.com/en-us/power-pages/security/power-pages-security)
- [Configure authentication identity providers](https://learn.microsoft.com/en-us/power-pages/security/authentication/)
- [Power Platform CLI Reference](https://learn.microsoft.com/en-us/power-platform/developer/cli/introduction)
