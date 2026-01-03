# heyy

A lightweight, reactive state management system with event emitter capabilities. Built with TypeScript, `heyy` combines proxy-based reactivity with an event-driven architecture to enable automatic UI updates and cross-component communication.

## ✨ Features

- 🔄 **Reactive State Updates** - Automatic event emission when state properties change
- 📡 **Event-Driven Architecture** - Loose coupling through event listeners
- 🎯 **One-Time Event Listeners** - Built-in support for initialization events
- 🔗 **Nested Object Reactivity** - Deep reactivity through proxy chaining
- 🧹 **Automatic Cleanup** - Easy listener management with cleanup functions
- 🚀 **Lightweight & Performant** - Zero dependencies, minimal overhead
- 📘 **TypeScript Support** - Full type safety with module augmentation

## 📦 Installation

```bash
npm install heyy
# or
pnpm add heyy
# or
yarn add heyy
```

## 🚀 Quick Start

```typescript
import hey from 'heyy'

// Set up event listeners for state changes
hey.on('user', (userData) => {
  console.log('User data changed:', userData)
  updateUserInterface(userData)
})

// Update state (triggers events automatically)
hey.user = { name: 'John', age: 30 }

// Listen for one-time events
hey.once('initialized', () => {
  console.log('App initialized!')
})

// Remove event listeners
const handler = (data) => console.log(data)
hey.on('update', handler)
hey.off('update', handler)
```

## 📖 Basic Usage

### Setting Up Listeners

```typescript
import hey from 'heyy'

// Listen for state changes
hey.on('user', (userData) => {
  console.log('User updated:', userData)
})

hey.on('cart', (cartData) => {
  updateCartDisplay(cartData)
})

// One-time listener (auto-removes after first call)
hey.once('app:ready', () => {
  initializeApp()
})
```

### Updating State

```typescript
// Simple property assignment triggers events automatically
hey.user = { name: 'John', email: 'john@example.com' }
hey.cart = { items: [], total: 0 }

// Nested objects are also reactive
hey.user.profile = { bio: 'Hello World' }
```

### Cleanup

```typescript
// The 'on' method returns a cleanup function
const unsubscribe = hey.on('user', (userData) => {
  console.log('User:', userData)
})

// Later, remove the listener
unsubscribe()

// Or use the 'off' method with the same handler reference
const handler = (data) => console.log(data)
hey.on('update', handler)
hey.off('update', handler)
```

## 🎯 TypeScript Support

Extend the `HeyState` interface via module augmentation for full type safety:

```typescript
// hey.d.ts or in your project's type definitions
declare module 'heyy' {
  interface HeyState {
    user?: { name: string; email: string }
    cart?: { items: any[]; total: number }
    settings?: { theme: 'light' | 'dark'; language: string }
  }
}

// Now you get full type safety:
import hey from 'heyy'
hey.user = { name: 'John', email: 'john@example.com' } // ✅ Typed
hey.settings = { theme: 'dark', language: 'en' } // ✅ Typed
```

## 📚 API Reference

See [API Documentation](./docs/api.md) for complete API reference.

### Core Methods

- `hey.on(event, handler)` - Register an event listener
- `hey.once(event, handler)` - Register a one-time event listener
- `hey.off(event, handler)` - Remove an event listener

### State Access

- `hey.property = value` - Set state property (triggers events)
- `hey.property` - Get state property

## 🔧 Advanced Usage

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

  return <div>{user?.name}</div>
}
```

### Vue Integration

```typescript
import { ref, onMounted, onUnmounted } from 'vue'
import hey from 'heyy'

export default {
  setup() {
    const user = ref(hey.user)

    const unsubscribe = hey.on('user', (userData) => {
      user.value = userData
    })

    onUnmounted(() => {
      unsubscribe()
    })

    return { user }
  }
}
```

### Vanilla JavaScript

```javascript
import hey from 'heyy'

// Set up listeners
hey.on('user', (userData) => {
  document.getElementById('username').textContent = userData.name
})

hey.on('cart', (cartData) => {
  document.getElementById('cart-count').textContent = cartData.items.length
})

// Update state from anywhere in your app
hey.user = { name: 'John', email: 'john@example.com' }
hey.cart = { items: [{ id: 1, name: 'Product' }], total: 29.99 }
```

## 🏗️ Architecture

`heyy` consists of three main components:

1. **SimpleEmitter** - Handles event registration and emission
2. **State** - Manages the reactive state and proxy creation
3. **Proxy Handler** - Intercepts property access and mutations

The system uses JavaScript Proxies to intercept property assignments and automatically emit events when state changes occur.

## ⚡ Performance

- Event emission is synchronous and lightweight
- Proxy overhead is minimal for typical use cases
- Memory usage scales linearly with event listeners
- No external dependencies

## 🔒 Security

- No external dependencies
- No `eval()` or dynamic code execution
- Type-safe event handling with TypeScript

## 📝 Best Practices

1. **Use descriptive event names** - e.g., `'user:updated'`, `'cart:item:added'`
2. **Always remove event listeners** - Prevent memory leaks by cleaning up
3. **Prefer `once` for initialization events** - Auto-cleanup for one-time operations
4. **Use TypeScript interfaces** - Define state structure via module augmentation
5. **Keep handlers focused** - Single responsibility for better maintainability
6. **Store handler references** - Required for proper cleanup with `off()`

## 📄 License

MIT

## 🤝 Contributing

Contributions are welcome! Please feel free to submit a Pull Request.

---

**Made with ❤️ for reactive state management**
