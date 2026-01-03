# API Reference

Complete API documentation for `heyy` - a reactive state management system with event emitter capabilities.

## Table of Contents

- [Import](#import)
- [State Management](#state-management)
- [Event Methods](#event-methods)
- [TypeScript Support](#typescript-support)
- [Examples](#examples)

## Import

```typescript
import hey from 'heyy'
```

The default export is a reactive state instance that combines state properties with event management methods.

## State Management

### Setting State

Assign values to properties on the `hey` instance to update state. This automatically triggers events for registered listeners.

```typescript
// Simple property assignment
hey.user = { name: 'John', email: 'john@example.com' }

// Nested objects are automatically made reactive
hey.user.profile = { bio: 'Hello World' }

// Primitive values
hey.count = 42
hey.isLoading = true
```

**Note:** When you assign an object, it's automatically wrapped in a proxy to enable nested reactivity.

### Getting State

Access state properties directly:

```typescript
const user = hey.user
const count = hey.count
const isLoading = hey.isLoading
```

## Event Methods

### `hey.on(event, handler)`

Register an event listener for state changes.

**Parameters:**
- `event` (string) - The name of the state property to listen for changes
- `handler` (function) - Callback function that receives the new state value

**Returns:** A cleanup function that removes the listener when called

**Example:**
```typescript
// Basic usage
hey.on('user', (userData) => {
  console.log('User updated:', userData)
})

// With cleanup function
const unsubscribe = hey.on('cart', (cartData) => {
  updateCartUI(cartData)
})

// Later, remove the listener
unsubscribe()
```

**Best Practices:**
- Use descriptive property names for better debugging
- Store the cleanup function for later removal
- Remove listeners in cleanup functions to prevent memory leaks

### `hey.once(event, handler)`

Register an event listener that will be called only once and then automatically removed.

**Parameters:**
- `event` (string) - The name of the state property to listen for changes
- `handler` (function) - Callback function that receives the new state value

**Returns:** A cleanup function that removes the listener when called (though it will auto-remove after first call)

**Example:**
```typescript
// One-time initialization
hey.once('initialized', () => {
  console.log('Application initialized')
  startApp()
})

// One-time user data fetch
hey.once('user', (userData) => {
  showWelcomeMessage(userData)
})
```

**Best Practices:**
- Use for initialization events that should only happen once
- Avoid using for events that may occur multiple times
- Consider using for cleanup operations

### `hey.off(event, handler)`

Remove an event listener.

**Parameters:**
- `event` (string) - The name of the state property
- `handler` (function) - The exact same function reference that was used with `on()`

**Returns:** `void`

**Example:**
```typescript
const handler = (userData) => {
  console.log('User:', userData)
}

// Add listener
hey.on('user', handler)

// Later, remove listener
hey.off('user', handler)
```

**Important:** You must use the exact same function reference when removing a listener. Anonymous functions cannot be removed.

**Best Practices:**
- Always store handler references when adding listeners
- Use the exact same handler reference when removing
- Consider using the cleanup function from `on()` instead

## TypeScript Support

### Extending the HeyState Interface

To get full type safety, extend the `HeyState` interface via module augmentation:

```typescript
// hey.d.ts or in your project's type definitions
declare module 'heyy' {
  interface HeyState {
    user?: {
      name: string
      email: string
      profile?: {
        bio: string
        avatar?: string
      }
    }
    cart?: {
      items: Array<{
        id: string
        name: string
        price: number
      }>
      total: number
    }
    settings?: {
      theme: 'light' | 'dark'
      language: string
    }
  }
}
```

After extending the interface, you'll get full type safety:

```typescript
import hey from 'heyy'

// ✅ Typed - TypeScript knows the structure
hey.user = {
  name: 'John',
  email: 'john@example.com',
  profile: {
    bio: 'Hello World'
  }
}

// ✅ Typed - TypeScript enforces the theme type
hey.settings = {
  theme: 'dark', // ✅ Valid
  language: 'en'
}

// ❌ TypeScript error - 'blue' is not a valid theme
hey.settings = {
  theme: 'blue', // ❌ Type error
  language: 'en'
}
```

## Examples

### Basic State Management

```typescript
import hey from 'heyy'

// Set up listeners
hey.on('user', (userData) => {
  console.log('User updated:', userData)
  updateUserProfile(userData)
})

hey.on('cart', (cartData) => {
  updateCartCount(cartData.items.length)
  updateCartTotal(cartData.total)
})

// Update state
hey.user = { name: 'John', email: 'john@example.com' }
hey.cart = {
  items: [{ id: 1, name: 'Product', price: 29.99 }],
  total: 29.99
}
```

### React Integration

```typescript
import { useEffect, useState } from 'react'
import hey from 'heyy'

function UserProfile() {
  const [user, setUser] = useState(hey.user)

  useEffect(() => {
    const unsubscribe = hey.on('user', (userData) => {
      setUser(userData)
    })

    return unsubscribe // Cleanup on unmount
  }, [])

  return (
    <div>
      <h1>{user?.name}</h1>
      <p>{user?.email}</p>
    </div>
  )
}
```

### Vue Integration

```typescript
import { ref, onMounted, onUnmounted } from 'vue'
import hey from 'heyy'

export default {
  setup() {
    const user = ref(hey.user)
    const cart = ref(hey.cart)

    const unsubscribeUser = hey.on('user', (userData) => {
      user.value = userData
    })

    const unsubscribeCart = hey.on('cart', (cartData) => {
      cart.value = cartData
    })

    onUnmounted(() => {
      unsubscribeUser()
      unsubscribeCart()
    })

    return { user, cart }
  }
}
```

### Multiple Listeners

```typescript
import hey from 'heyy'

// Multiple listeners for the same event
hey.on('user', (userData) => {
  updateHeader(userData)
})

hey.on('user', (userData) => {
  updateSidebar(userData)
})

hey.on('user', (userData) => {
  logUserActivity(userData)
})

// All listeners are called when state changes
hey.user = { name: 'John', email: 'john@example.com' }
```

### Cleanup Pattern

```typescript
import hey from 'heyy'

class UserManager {
  private cleanup: (() => void)[] = []

  constructor() {
    // Store cleanup functions
    this.cleanup.push(
      hey.on('user', this.handleUserChange.bind(this))
    )
    this.cleanup.push(
      hey.on('cart', this.handleCartChange.bind(this))
    )
  }

  private handleUserChange(userData: any) {
    console.log('User changed:', userData)
  }

  private handleCartChange(cartData: any) {
    console.log('Cart changed:', cartData)
  }

  destroy() {
    // Clean up all listeners
    this.cleanup.forEach(fn => fn())
    this.cleanup = []
  }
}
```

### Initialization Pattern

```typescript
import hey from 'heyy'

// Set up one-time initialization
hey.once('app:ready', () => {
  console.log('App is ready!')
  initializeComponents()
})

// Later, trigger initialization
hey['app:ready'] = true // Triggers the once listener
```

### Nested Object Reactivity

```typescript
import hey from 'heyy'

// Listen for top-level changes
hey.on('user', (userData) => {
  console.log('User object changed:', userData)
})

// Set initial user
hey.user = {
  name: 'John',
  profile: {
    bio: 'Initial bio'
  }
}

// Update nested property (triggers 'user' event)
hey.user.profile.bio = 'Updated bio'
```

## Event Naming Conventions

While `heyy` doesn't enforce any naming conventions, here are some recommended patterns:

- **Simple names**: `'user'`, `'cart'`, `'settings'`
- **Namespaced**: `'user:updated'`, `'cart:item:added'`, `'app:initialized'`
- **Consistent**: Use the same naming pattern throughout your application

## Performance Considerations

- Event emission is synchronous and lightweight
- Proxy overhead is minimal for typical use cases
- Memory usage scales linearly with the number of event listeners
- Consider removing unused listeners to prevent memory leaks
- Avoid deeply nested objects for better performance

## Common Patterns

### State Initialization

```typescript
import hey from 'heyy'

// Initialize state with default values
hey.user = null
hey.cart = { items: [], total: 0 }
hey.settings = { theme: 'light', language: 'en' }
```

### Conditional Updates

```typescript
import hey from 'heyy'

hey.on('user', (userData) => {
  if (userData) {
    showUserProfile(userData)
  } else {
    showLoginForm()
  }
})
```

### State Synchronization

```typescript
import hey from 'heyy'

// Sync state with external source
async function fetchUser() {
  const user = await api.getUser()
  hey.user = user
}

// Listeners automatically react to the update
hey.on('user', (userData) => {
  console.log('User synced:', userData)
})
```
