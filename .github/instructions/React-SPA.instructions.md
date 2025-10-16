---
applyTo: "CodeApps/**,PowerPagesSPA/**,PCFs/**"
---

# Fluent UI 9 + React SPA Development Guidelines

This document outlines the standard practices and conventions for React-based development projects using Fluent UI 9 and modern React patterns.

---

## 1. Folder Structure

Maintain a clear and consistent folder structure across all projects:

```
src/
├── README.md             # Global React project documentation
├── assets/               # Static assets (images, fonts, icons)
├── components/           # Reusable UI components
│   ├── ComponentName/
│   │   ├── ComponentName.tsx
│   │   ├── ComponentName.style.ts
│   │   ├── ComponentName.type.ts
│   │   └── index.ts      # Barrel export
│   ├── ProtectedRoute/   # Route guard component
│   │   ├── ProtectedRoute.tsx
│   │   ├── ProtectedRoute.type.ts
│   │   └── index.ts
│   └── MainLayout/       # Layout wrapper for routes
│       ├── MainLayout.tsx
│       ├── MainLayout.style.ts
│       └── index.ts
├── hooks/                # Custom React hooks
│   ├── useAuth.ts        # Authentication hook (for protected routes)
│   ├── useNavigate.ts    # Re-export or custom navigation hook
│   ├── useRouteParams.ts # Type-safe route params hook
│   ├── useHookName.ts
│   └── index.ts          # Barrel export
├── contexts/             # React Context providers
│   ├── AuthContext.tsx   # Authentication context
│   ├── ContextNameContext.tsx
│   └── index.ts          # Barrel export
├── pages/                # Page-level components
│   └── PageName/
│       ├── PageName.tsx
│       ├── PageName.style.ts
│       ├── index.ts      # Barrel export
│       └── README.md     # Page documentation (overview, components used, features)
├── theme/                # Theming configuration
│   └── customTheme.ts
├── routes/               # React Router configuration
│   ├── index.tsx         # Main router configuration with createBrowserRouter
│   ├── constants.ts      # Route path constants (ROUTES object)
│   └── types.ts          # Route parameter types
├── utils/                # Utility functions
├── services/             # API and service integrations
├── models/               # TypeScript type definitions and interfaces
└── mockData/             # Mock data for testing and development
```

### Folder Structure Guidelines

- **Global Documentation**: A `README.md` at the `/src` level provides a high-level overview of the entire React project
- **Components**: Each component must have its own folder with separate files for logic, styles, and types
- **Barrel Exports**: Always use `index.ts` for clean imports (e.g., `import { Button } from '@/components/Button'`)
- **Page Documentation**: Each page should have a `README.md` documenting:
  - Global overview of the page design and purpose
  - List of components used within the page
  - Key features and functionality
  - User interactions and flows
  - Data requirements and state management
- **Separation of Concerns**: Keep styles, types, and logic in separate files for maintainability

#### Routing-Specific Files

- **routes/index.tsx**: Main router configuration using `createBrowserRouter` with all route definitions, lazy-loaded pages, and nested routes
- **routes/constants.ts**: Centralized route path constants to avoid magic strings (e.g., `ROUTES.DASHBOARD`, `ROUTES.PROFILE(id)`)
- **routes/types.ts**: TypeScript types for route parameters and navigation state

#### Component Files for Routing

- **components/ProtectedRoute**: Reusable component for authentication guards on protected routes
- **components/MainLayout**: Layout wrapper component using `<Outlet />` for nested routes with shared UI (header, sidebar, footer)

#### Hook Files for Routing

- **hooks/useAuth.ts**: Authentication hook for checking user login status (used by ProtectedRoute)
- **hooks/useRouteParams.ts**: Type-safe wrapper for `useParams()` with proper TypeScript typing
- **hooks/useNavigate.ts**: Optional custom navigation hook with additional logic or analytics tracking

#### Context for Routing

- **contexts/AuthContext.tsx**: Authentication context provider for managing user state across routes

---

### Global Project Documentation (src/README.md)

The `src/README.md` file serves as the central documentation for the entire React application. It provides a high-level overview to help developers and GitHub Copilot agents understand the project architecture without going into excessive detail.

#### Required Sections

1. **Project Overview**
   - Brief description of the application purpose
   - Technology stack summary (React, TypeScript, Fluent UI 9, React Router, etc.)
   - Key architectural decisions

2. **Navigation Structure**
   - Overview of routing approach (React Router with `createBrowserRouter`)
   - Main routes and their purposes
   - Authentication flow and protected routes
   - Layout hierarchy (nested routes, shared layouts)

3. **Pages**
   - List of all pages with brief descriptions
   - Page hierarchy and relationships
   - Link to individual page README files for detailed documentation

4. **Components Architecture**
   - Overview of component organization
   - Key reusable components and their purposes
   - Component categorization (UI components, layout components, utility components)
   - Shared component patterns (ProtectedRoute, MainLayout, etc.)

5. **State Management**
   - Context providers used in the application
   - Global state management approach
   - Data flow patterns

6. **Data Sources**
   - API integrations overview
   - Service layer architecture
   - Mock data strategy (if applicable)
   - Data fetching patterns

7. **Theming & Styling**
   - Theme configuration approach
   - Design system (Fluent UI 9)
   - Responsive design strategy
   - Custom styling patterns

8. **Project Structure Quick Reference**
   - Folder structure summary
   - Import path conventions
   - File naming conventions

**Example Template:**

```markdown
# React Application Documentation

## Project Overview
Brief description of what this application does and who it's for.

**Tech Stack:** React 18, TypeScript, Fluent UI 9, React Router v6, Vite

## Navigation Structure
- **Routing:** Client-side routing using React Router's `createBrowserRouter`
- **Main Routes:** Home, Dashboard, Profile, Settings
- **Protected Routes:** Dashboard and Profile require authentication
- **Layouts:** MainLayout provides shared header/footer for all authenticated pages

## Pages
| Page | Route | Description | Protected |
|------|-------|-------------|-----------|
| Home | `/` | Landing page | No |
| Dashboard | `/dashboard` | User dashboard with analytics | Yes |
| Profile | `/profile/:userId` | User profile management | Yes |
| Settings | `/settings` | Application settings | Yes |

See individual page folders for detailed documentation.

## Components Architecture
- **Layout Components:** MainLayout, Header, Footer, Sidebar
- **Route Guards:** ProtectedRoute (authentication check)
- **UI Components:** Organized by feature/domain in `/components`
- **Shared Patterns:** All components follow consistent structure (component.tsx, styles.ts, types.ts, index.ts)

## State Management
- **AuthContext:** Global authentication state (user, login, logout)
- **ThemeContext:** Theme switching (light/dark mode)
- **Local State:** Page-level state managed with useState/useReducer
- **Data Fetching:** React Query for server state (if applicable)

## Data Sources
- **API Services:** RESTful API integration via `/services` folder
- **Authentication:** Token-based auth stored in localStorage
- **Mock Data:** Development mocks available in `/mockData` folder
- **Service Layer:** Centralized API calls with error handling

## Theming & Styling
- **Design System:** Fluent UI 9 components
- **Theme Detection:** Browser preference-based (prefers-color-scheme)
- **Custom Theme:** Brand colors defined in `/theme/customTheme.ts`
- **Responsive Design:** Mobile-first approach with Fluent UI breakpoints
- **Styling Approach:** Fluent UI's `makeStyles` for component styles

## Project Structure Quick Reference
```
src/
├── README.md          # This file
├── components/        # Reusable UI components
├── pages/            # Page components (with individual READMEs)
├── routes/           # Router configuration
├── contexts/         # React Context providers
├── hooks/            # Custom React hooks
├── services/         # API integration layer
├── theme/            # Theme configuration
├── models/           # TypeScript types
└── utils/            # Utility functions
```

## Getting Started
1. Install dependencies: `npm install`
2. Start dev server: `npm run dev`
3. Build for production: `npm run build`

## Code Conventions
- TypeScript strict mode enabled
- Component structure: TSX, styles, types, barrel export
- Imports: Use path aliases (@/components, @/hooks, etc.)
- Naming: PascalCase for components, camelCase for functions/variables
```

This template should be adapted based on the specific application's needs while maintaining these core sections.

---

## 2. UX/UI Guidelines

### 2.1 Fluent UI 9

**Use Fluent UI 9 as the primary design system for all projects.**

#### Technical Prerequisites

- **React Version**: Downgrade to React 18.x (Fluent UI 9 requires React 18)
  ```bash
  npm install react@18 react-dom@18
  ```

#### Design Principles

- Follow [Fluent UI design guidelines](https://react.fluentui.dev/)
- Use Fluent UI components as the foundation for all UI elements
- Maintain consistency with Microsoft's design language

#### Theming Implementation

- **Default Theme**: Use browser theme detection (light vs. dark mode)
- **Custom Primary Color**: Implement a custom primary color in your theme
- Use Fluent UI's theming system with `FluentProvider` and `createLightTheme`/`createDarkTheme`

**Example Theme Setup:**

```typescript
// theme/customTheme.ts
import {
  BrandVariants,
  createLightTheme,
  createDarkTheme,
  Theme,
} from '@fluentui/react-components';

const brandColors: BrandVariants = {
  10: '#020305',
  20: '#0f1419',
  30: '#1a2129',
  40: '#242d38',
  50: '#2e3a47',
  60: '#394756',
  70: '#445566',
  80: '#506375',
  90: '#5c7184',
  100: '#698093',
  110: '#768fa2',
  120: '#839eb1',
  130: '#91aec0',
  140: '#9fbecf',
  150: '#adcede',
  160: '#bcdfed',
};

export const lightTheme: Theme = createLightTheme(brandColors);
export const darkTheme: Theme = createDarkTheme(brandColors);
```

**Browser Theme Detection:**

```typescript
const prefersDark = window.matchMedia('(prefers-color-scheme: dark)').matches;
const theme = prefersDark ? darkTheme : lightTheme;
```

---

### 2.2 Accessibility

Ensure all applications are accessible and comply with WCAG 2.1 Level AA standards.

#### 2.2.1 ARIA Best Practices

- **Labels**: Always include `aria-label` or `aria-labelledby` for interactive elements
- **Descriptions**: Use `aria-describedby` for additional context when needed
- **Focus Management**: Implement proper focus management with `tabIndex`
- **Semantic HTML**: Use semantic HTML elements (`<button>`, `<nav>`, `<main>`, etc.) when possible

**Example:**

```tsx
<Button
  aria-label="Close dialog"
  aria-describedby="close-description"
  onClick={handleClose}
>
  <Dismiss24Regular />
</Button>
<span id="close-description" hidden>
  Closes the dialog without saving changes
</span>
```

#### 2.2.2 Keyboard Navigation

- **Keyboard Accessibility**: Ensure all interactive elements are keyboard accessible
- **Tab Order**: Implement proper tab order with `tabIndex` (0 for natural order, -1 to remove from tab order)
- **Arrow Keys**: Use arrow keys for grid/list navigation
- **Skip Links**: Provide skip links for long content sections

**Example:**

```tsx
const handleKeyDown = (event: React.KeyboardEvent) => {
  if (event.key === 'Enter' || event.key === ' ') {
    handleAction();
    event.preventDefault();
  }
};
```

#### 2.2.3 Color and Contrast

- **Design Tokens**: Use Fluent UI design tokens for consistent colors
- **Contrast Ratios**: Ensure minimum contrast ratios:
  - 4.5:1 for normal text
  - 3:1 for large text (18pt+)
- **Color Independence**: Don't rely solely on color to convey information (use icons, text, or patterns)

**Example:**

```typescript
// Use Fluent UI tokens instead of hardcoded colors
import { tokens } from '@fluentui/react-components';

const styles = {
  error: {
    color: tokens.colorPaletteRedForeground1,
    backgroundColor: tokens.colorPaletteRedBackground1,
  },
};
```

---

## 3. Performance Optimization

### 3.1 React Best Practices

- **Memoization**: Use `useMemo` and `useCallback` for expensive computations and callback stability
  ```tsx
  const expensiveValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
  const handleClick = useCallback(() => doSomething(id), [id]);
  ```

- **Effect Dependencies**: Implement proper dependency arrays in `useEffect`
  ```tsx
  useEffect(() => {
    fetchData(userId);
  }, [userId]); // Include all dependencies
  ```

- **Component Memoization**: Use `React.memo` for components that don't need frequent re-renders
  ```tsx
  export const MyComponent = React.memo(({ data }) => {
    return <div>{data}</div>;
  });
  ```

- **List Keys**: Always use proper, stable `key` props for lists (avoid using array indices when possible)
  ```tsx
  {items.map(item => (
    <ListItem key={item.id} data={item} />
  ))}
  ```

### 3.2 Bundle Optimization

- **Code Splitting**: Use dynamic imports for route-based code splitting
  ```tsx
  const Dashboard = lazy(() => import('./pages/Dashboard'));
  ```

- **Tree Shaking**: Ensure proper tree shaking by using ES6 imports
- **Asset Optimization**: Optimize images and assets (use WebP, lazy loading)
- **Vite Optimization**: Leverage Vite's built-in optimization features
- **Barrel Imports**: Use barrel imports following best practices:
  - ✅ Named exports only: `export { ComponentName } from './ComponentName'`
  - ❌ Avoid `export *` (prevents tree shaking)
  - ❌ Avoid circular references
  - ❌ Avoid deeply nested barrel imports (max 2-3 levels)

**Example Barrel Import:**

```typescript
// components/index.ts
export { Button } from './Button';
export { Input } from './Input';
export { Card } from './Card';
// Do NOT use: export * from './Button';
```

---

## 4. Responsive Design

### 4.1 Mobile-First Approach

- **Design Philosophy**: Design for mobile screens first, then enhance for larger screens
- **Breakpoints**: Use Fluent UI's responsive breakpoints or define custom ones
- **Touch Targets**: Implement touch-friendly interactions with minimum 44px tap targets
- **Mobile Gestures**: Support common mobile gestures (swipe, pinch-to-zoom where appropriate)

**Fluent UI Breakpoints:**

```typescript
const breakpoints = {
  mobile: '(max-width: 640px)',
  tablet: '(min-width: 641px) and (max-width: 1024px)',
  desktop: '(min-width: 1025px)',
};
```

### 4.2 Layout Patterns

- **Modern CSS**: Use CSS Grid and Flexbox for responsive layouts
- **Viewport Meta**: Implement proper viewport meta tags
  ```html
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  ```

- **Relative Units**: Use relative units (`rem`, `em`, `%`, `vw`, `vh`) instead of fixed pixels
- **Testing**: Test on various screen sizes and orientations (portrait/landscape)

**Example Responsive Layout:**

```typescript
// ComponentName.style.ts
import { makeStyles, shorthands } from '@fluentui/react-components';

export const useStyles = makeStyles({
  container: {
    display: 'grid',
    gridTemplateColumns: '1fr',
    ...shorthands.gap('1rem'),
    ...shorthands.padding('1rem'),
    
    '@media (min-width: 768px)': {
      gridTemplateColumns: 'repeat(2, 1fr)',
      ...shorthands.padding('1.5rem'),
    },
    
    '@media (min-width: 1024px)': {
      gridTemplateColumns: 'repeat(3, 1fr)',
      ...shorthands.padding('2rem'),
    },
  },
});
```

---

## 5. Routing with React Router

### 5.1 Client-Side Rendering with React Router

Use React Router for managing multiple pages in single-page applications (SPA) with client-side routing.

#### Installation

```bash
npm install react-router-dom
```

#### Routing Structure

Organize routes in a dedicated `routes` folder with a centralized configuration:

```typescript
// routes/index.tsx
import { createBrowserRouter, RouterProvider } from 'react-router-dom';
import { lazy, Suspense } from 'react';
import { Spinner } from '@fluentui/react-components';

// Lazy load pages for code splitting
const Home = lazy(() => import('../pages/Home'));
const Dashboard = lazy(() => import('../pages/Dashboard'));
const Profile = lazy(() => import('../pages/Profile'));
const NotFound = lazy(() => import('../pages/NotFound'));

// Loading component
const PageLoader = () => (
  <div style={{ display: 'flex', justifyContent: 'center', alignItems: 'center', height: '100vh' }}>
    <Spinner label="Loading..." />
  </div>
);

// Router configuration
export const router = createBrowserRouter([
  {
    path: '/',
    element: <Home />,
    errorElement: <NotFound />,
  },
  {
    path: '/dashboard',
    element: <Dashboard />,
  },
  {
    path: '/profile/:userId',
    element: <Profile />,
  },
  {
    path: '*',
    element: <NotFound />,
  },
]);

// App Router Component
export const AppRouter = () => (
  <Suspense fallback={<PageLoader />}>
    <RouterProvider router={router} />
  </Suspense>
);
```

#### Using the Router in App

```typescript
// App.tsx
import { FluentProvider } from '@fluentui/react-components';
import { AppRouter } from './routes';
import { lightTheme } from './theme/customTheme';

export const App = () => {
  return (
    <FluentProvider theme={lightTheme}>
      <AppRouter />
    </FluentProvider>
  );
};
```

#### Navigation Best Practices

- **Use Link Component**: Use `<Link>` instead of `<a>` tags for internal navigation
- **Programmatic Navigation**: Use `useNavigate()` hook for navigation in event handlers
- **Route Parameters**: Use `useParams()` to access URL parameters
- **Query Strings**: Use `useSearchParams()` for query string management

**Example Navigation:**

```tsx
import { Link, useNavigate, useParams } from 'react-router-dom';

const MyComponent = () => {
  const navigate = useNavigate();
  const { userId } = useParams();

  const handleNavigation = () => {
    navigate('/dashboard', { state: { from: 'home' } });
  };

  return (
    <>
      <Link to="/profile/123">View Profile</Link>
      <Button onClick={handleNavigation}>Go to Dashboard</Button>
    </>
  );
};
```

#### Layout and Nested Routes

Use layout components for shared UI elements (header, sidebar, footer):

```typescript
// routes/index.tsx with layouts
import { Outlet } from 'react-router-dom';
import { MainLayout } from '../components/MainLayout';

export const router = createBrowserRouter([
  {
    path: '/',
    element: <MainLayout />,
    children: [
      {
        index: true,
        element: <Home />,
      },
      {
        path: 'dashboard',
        element: <Dashboard />,
      },
      {
        path: 'profile/:userId',
        element: <Profile />,
      },
    ],
  },
]);

// components/MainLayout/MainLayout.tsx
export const MainLayout = () => (
  <div>
    <Header />
    <main>
      <Outlet /> {/* Child routes render here */}
    </main>
    <Footer />
  </div>
);
```

#### Protected Routes

Implement authentication guards for protected pages:

```typescript
// components/ProtectedRoute/ProtectedRoute.tsx
import { Navigate } from 'react-router-dom';
import { useAuth } from '../hooks/useAuth';

export const ProtectedRoute = ({ children }: { children: React.ReactNode }) => {
  const { isAuthenticated } = useAuth();

  if (!isAuthenticated) {
    return <Navigate to="/login" replace />;
  }

  return <>{children}</>;
};

// Usage in routes
{
  path: '/dashboard',
  element: (
    <ProtectedRoute>
      <Dashboard />
    </ProtectedRoute>
  ),
}
```

#### Route Configuration Best Practices

- ✅ Use lazy loading for all pages to optimize bundle size
- ✅ Implement error boundaries at the route level
- ✅ Use suspense with meaningful loading states
- ✅ Organize routes hierarchically for nested layouts
- ✅ Define route constants to avoid magic strings
- ✅ Use TypeScript for type-safe route parameters

**Route Constants Example:**

```typescript
// routes/constants.ts
export const ROUTES = {
  HOME: '/',
  DASHBOARD: '/dashboard',
  PROFILE: (userId: string) => `/profile/${userId}`,
  NOT_FOUND: '*',
} as const;

// Usage
navigate(ROUTES.PROFILE('123'));
```

---

## Summary

Following these guidelines will ensure:
- ✅ Consistent project structure across all React applications
- ✅ Accessible, user-friendly interfaces
- ✅ Optimal performance and bundle sizes
- ✅ Responsive designs that work on all devices
- ✅ Maintainable, scalable codebases
- ✅ Efficient client-side routing with code splitting

Always refer to this document when starting a new feature or component, and update it as best practices evolve.
