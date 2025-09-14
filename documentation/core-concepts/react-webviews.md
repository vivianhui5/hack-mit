# React Webviews

> Build React-based webviews for MentraOS using @mentra/react. Learn setup, authentication, making authenticated API calls, CORS configuration, and troubleshooting with a full example.

The [`@mentra/react`](https://www.npmjs.com/package/@mentra/react) library simplifies building React-based webviews that integrate seamlessly with MentraOS authentication. When users open your webview through the MentraOS Mobile App, they are automatically authenticated without requiring a separate login process.

## What Are React Webviews?

React webviews are web applications built with React that run inside the MentraOS Mobile App. They provide rich user interfaces for:

* **Settings and Configuration**: Let users customize your app's behavior
* **Data Visualization**: Display charts, graphs, and analytics
* **Content Management**: Create, edit, and organize user content
* **Dashboard Interfaces**: Show personalized information and controls

The `@mentra/react` library handles all the authentication complexity, automatically extracting and verifying user tokens from the MentraOS system.

## Complete Example

There's a complete example of a React webview in the [`MentraOS-React-Example-App`](https://github.com/Mentra-Community/MentraOS-React-Example-App) repository. Simply follow along with the README to get off the ground quickly.

## Installation

Install the React authentication library in your webview project:

```bash
npm install @mentra/react
# or
yarn add @mentra/react
# or
bun add @mentra/react
```

## Prerequisites

* **React 16.8+**: The library uses React Hooks
* **MentraOS App Server**: Your backend must be deployed, either on the same domain as your frontend or on a different domain that allows CORS requests from your frontend.
* **Developer Console Setup**: Set the webview url in the developer console to your frontend server

## Basic Setup

### 1. Wrap Your App with the Authentication Provider

The `MentraAuthProvider` component manages authentication state for your entire React application:

```tsx
// src/main.tsx
import React from "react"
import ReactDOM from "react-dom/client"
import App from "./App"
import {MentraAuthProvider} from "@mentra/react"

/**
 * Application entry point that provides MentraOS authentication context
 * to the entire React component tree
 */
ReactDOM.createRoot(document.getElementById("root")!).render(
  <React.StrictMode>
    <MentraAuthProvider>
      <App />
    </MentraAuthProvider>
  </React.StrictMode>,
)
```

### 2. Access Authentication State

Use the `UseMentraAuth` hook to access user information and authentication status:

```tsx
// src/components/UserProfile.tsx
import React from "react"
import {UseMentraAuth} from "@mentra/react"

/**
 * Component that displays user authentication status and profile information.
 * Demonstrates basic usage of the MentraOS authentication hook.
 *
 * @returns {React.JSX.Element} User profile component with authentication state
 */
const UserProfile: React.FC = (): React.JSX.Element => {
  const {userId, isLoading, error, isAuthenticated, logout} = UseMentraAuth()

  // Handle loading state during authentication
  if (isLoading) {
    return <div>Loading authentication...</div>
  }

  // Handle authentication errors
  if (error) {
    return (
      <div style={{color: "red"}}>
        <p>Authentication Error: {error}</p>
        <p>Please ensure you are opening this page from the MentraOS app.</p>
      </div>
    )
  }

  // Handle unauthenticated state
  if (!isAuthenticated || !userId) {
    return <div>Not authenticated. Please open from the MentraOS Mobile App.</div>
  }

  // Display authenticated user information
  return (
    <div>
      <h2>Welcome, MentraOS User!</h2>
      <p>
        User ID: <strong>{userId}</strong>
      </p>
      <button onClick={logout}>Logout</button>
    </div>
  )
}

export default UserProfile
```

## Complete Example Application

Here's a comprehensive example that demonstrates authentication, API calls, and error handling:

```tsx
/**
 * Main application component that demonstrates MentraOS authentication integration.
 * This file serves as the root component for testing the mentraos-react package functionality.
 */

import React, {useState} from "react"
import {MentraAuthProvider, UseMentraAuth} from "@mentra/react"

/**
 * Type definition for the API response from the notes endpoint
 */
interface NotesApiResponse {
  [key: string]: any // Allow flexible response structure
}

/**
 * Content component that displays authentication status and user information.
 * This component consumes the MentraAuth context to show loading states,
 * errors, and authenticated user data. It also provides functionality to make
 * authenticated API calls to the App backend.
 *
 * @returns {React.JSX.Element} The rendered content based on authentication state
 */
function Content(): React.JSX.Element {
  const {userId, isLoading, error, isAuthenticated, frontendToken} = UseMentraAuth()

  // State for managing API call results and loading state
  const [apiResult, setApiResult] = useState<NotesApiResponse | null>(null)
  const [apiError, setApiError] = useState<string | null>(null)
  const [isLoadingApi, setIsLoadingApi] = useState<boolean>(false)

  /**
   * Makes an authenticated API call to the app backend notes endpoint
   * Uses the frontendToken for authorization
   */
  const fetchNotesFromBackend = async (): Promise<void> => {
    if (!frontendToken) {
      setApiError("No frontend token available for backend call.")
      return
    }

    setIsLoadingApi(true)
    setApiError(null)
    setApiResult(null)

    try {
      const response = await fetch(`/api/notes`, {
        method: "GET",
        headers: {
          "Content-Type": "application/json",
          "Authorization": `Bearer ${frontendToken}`,
        },
      })

      if (!response.ok) {
        throw new Error(`Backend request failed: ${response.status} ${response.statusText}`)
      }

      const data: NotesApiResponse = await response.json()
      setApiResult(data)
    } catch (error) {
      const errorMessage = error instanceof Error ? error.message : "Unknown error occurred"
      setApiError(errorMessage)
    } finally {
      setIsLoadingApi(false)
    }
  }

  // Handle loading state
  if (isLoading) {
    return <p>Loading authentication...</p>
  }

  // Handle error state
  if (error) {
    return <p style={{color: "red"}}>Error: {error}</p>
  }

  // Handle unauthenticated state
  if (!isAuthenticated) {
    return <p>Not authenticated. Please log in.</p>
  }

  // Handle authenticated state
  return (
    <div>
      <p style={{color: "green"}}>✓ Successfully authenticated!</p>
      <p>
        User ID: <strong>{userId}</strong>
      </p>

      {/* API Testing Section */}
      <div style={{marginTop: "30px", padding: "20px", border: "1px solid #ccc", borderRadius: "8px"}}>
        <h2>Backend API Test</h2>
        <p>Test authenticated calls to your app backend:</p>

        <button
          onClick={fetchNotesFromBackend}
          disabled={isLoadingApi}
          style={{
            padding: "10px 20px",
            backgroundColor: isLoadingApi ? "#ccc" : "#007bff",
            color: "white",
            border: "none",
            borderRadius: "4px",
            cursor: isLoadingApi ? "not-allowed" : "pointer",
            fontSize: "16px",
          }}>
          {isLoadingApi ? "Loading..." : "Fetch Notes from Backend"}
        </button>

        {/* Display API loading state */}
        {isLoadingApi && <p style={{color: "#666", marginTop: "10px"}}>Making authenticated request...</p>}

        {/* Display API errors */}
        {apiError && (
          <div
            style={{
              marginTop: "15px",
              padding: "10px",
              backgroundColor: "#ffe6e6",
              border: "1px solid #ff9999",
              borderRadius: "4px",
            }}>
            <strong style={{color: "#cc0000"}}>API Error:</strong>
            <p style={{color: "#cc0000", margin: "5px 0 0 0"}}>{apiError}</p>
          </div>
        )}

        {/* Display API results */}
        {apiResult && (
          <div
            style={{
              marginTop: "15px",
              padding: "10px",
              backgroundColor: "#e6ffe6",
              border: "1px solid #99ff99",
              borderRadius: "4px",
            }}>
            <strong style={{color: "#006600"}}>API Response:</strong>
            <pre
              style={{
                backgroundColor: "#f5f5f5",
                padding: "10px",
                borderRadius: "4px",
                overflow: "auto",
                maxHeight: "300px",
                fontSize: "14px",
                fontFamily: 'Monaco, Consolas, "Courier New", monospace',
              }}>
              {JSON.stringify(apiResult, null, 2)}
            </pre>
          </div>
        )}
      </div>
    </div>
  )
}

/**
 * Root App component that provides authentication context to the entire application.
 * This component wraps the Content component with the MentraAuthProvider
 * to enable authentication functionality throughout the app.
 *
 * @returns {React.JSX.Element} The main application component
 */
function App(): React.JSX.Element {
  return (
    <MentraAuthProvider>
      <div style={{padding: "20px", fontFamily: "Arial, sans-serif"}}>
        <h1>MentraOS React Test App</h1>
        <Content />
      </div>
    </MentraAuthProvider>
  )
}

export default App
```

## Making Authenticated API Calls

The `frontendToken` from `UseMentraAuth` is a JWT token that you should include in the `Authorization` header when making requests to your App backend:

```tsx
/**
 * Hook for making authenticated API calls to the app backend
 *
 * @returns {Object} Functions for making authenticated requests
 */
const useAuthenticatedApi = () => {
  const {frontendToken} = UseMentraAuth()

  /**
   * Makes an authenticated GET request to the specified endpoint
   *
   * @param {string} endpoint - The API endpoint to call
   * @returns {Promise<any>} The response data
   * @throws {Error} If the request fails or token is missing
   */
  const authenticatedGet = async (endpoint: string): Promise<any> => {
    if (!frontendToken) {
      throw new Error("No authentication token available")
    }

    const response = await fetch(endpoint, {
      method: "GET",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${frontendToken}`,
      },
    })

    if (!response.ok) {
      throw new Error(`API request failed: ${response.status} ${response.statusText}`)
    }

    return response.json()
  }

  /**
   * Makes an authenticated POST request with JSON data
   *
   * @param {string} endpoint - The API endpoint to call
   * @param {any} data - The data to send in the request body
   * @returns {Promise<any>} The response data
   * @throws {Error} If the request fails or token is missing
   */
  const authenticatedPost = async (endpoint: string, data: any): Promise<any> => {
    if (!frontendToken) {
      throw new Error("No authentication token available")
    }

    const response = await fetch(endpoint, {
      method: "POST",
      headers: {
        "Content-Type": "application/json",
        "Authorization": `Bearer ${frontendToken}`,
      },
      body: JSON.stringify(data),
    })

    if (!response.ok) {
      throw new Error(`API request failed: ${response.status} ${response.statusText}`)
    }

    return response.json()
  }

  return {authenticatedGet, authenticatedPost}
}
```

## Authentication Hook API

The `UseMentraAuth` hook returns an object with the following properties:

```typescript
interface MentraAuthContextType {
  /** Unique identifier for the authenticated user */
  userId: string | null

  /** JWT token for making authenticated requests to your app backend */
  frontendToken: string | null

  /** True while authentication is being processed */
  isLoading: boolean

  /** Error message if authentication fails */
  error: string | null

  /** True if the user is successfully authenticated */
  isAuthenticated: boolean

  /** Function to clear authentication state and log out */
  logout: () => void
}
```

## How Authentication Works

1. **Token Extraction**: When the MentraOS Mobile App opens your webview, it appends an `aos_signed_user_token` parameter to the URL
2. **Token Verification**: The library verifies the token's signature against the MentraOS Cloud public key
3. **User Identification**: If valid, it extracts the `userId` and `frontendToken` from the token payload
4. **Persistence**: The authentication data is stored in `localStorage` for the session
5. **Context Sharing**: All authentication state is made available through React Context

## CORS Configuration

If your webview frontend is hosted separately from your app backend, configure CORS properly:

```typescript
// In your app server setup
import cors from "cors"

/**
 * Configure CORS to allow requests from your webview domain
 */
app.use(
  cors({
    origin: [
      "https://your-webview-domain.com",
      "http://localhost:3000", // For development
    ],
    credentials: true, // If you use cookies
    methods: ["GET", "POST", "PUT", "DELETE"],
    allowedHeaders: ["Content-Type", "Authorization"],
  }),
)
```

## Troubleshooting

### Common Issues

**"MentraOS signed user token not found"**

* Ensure your webview is opened through the MentraOS Mobile App
* Check that the manager app is correctly appending the token to your URL
* Verify your webview URL is configured correctly in the Developer Console

**"Token validation failed"**

* Verify the device clock is synchronized (token expiration is time-sensitive)
* Check that you're using the latest version of `@mentra/react`
* Ensure the MentraOS Mobile App is up to date

**Backend authentication fails**

* Check that you're sending the `Authorization: Bearer ${frontendToken}` header
* Verify your app server is configured to accept the frontend token
* Ensure CORS is configured properly if frontend and backend are on different domains

**Changes not reflecting**

* Clear browser cache and localStorage
* Restart the MentraOS Mobile App
* Check browser developer tools for JavaScript errors

### Debugging Tips

Enable debug logging to troubleshoot authentication issues:

```tsx
// Add this to see authentication state changes
const Content = () => {
  const auth = UseMentraAuth()

  // Log authentication state for debugging
  React.useEffect(() => {
    console.log("Auth state:", {
      userId: auth.userId,
      isAuthenticated: auth.isAuthenticated,
      isLoading: auth.isLoading,
      error: auth.error,
      hasToken: !!auth.frontendToken,
    })
  }, [auth])

  // ... rest of component
}
```

## Next Steps

* **[Complete Example Application](https://github.com/Mentra-Community/MentraOS-React-Example-App)**: See a complete example of a React webview
* **[Deploying a React App](https://github.com/Mentra-Community/MentraOS-React-Example-App/blob/main/DEPLOYMENT-single-server.md)**: This guide covers deploying the MentraOS React Example App to production
* **[Webview Authentication Overview](/webview-auth-overview)**: Learn about the broader authentication system
* **[Core Concepts](/core-concepts)**: Understand the full MentraOS ecosystem
