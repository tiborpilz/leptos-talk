---
layout: intro-image
theme: apple-basic
mdc: true
---

# Rust & Leptos for TypeScript Developers
## Tibor Pilz

June, 2025

---

# About Me

Tibor Pilz

- Senior Fullstack Engineer @ Team Web Core
- Overengineering enthusiast
- A bit of a nerd about programming languages

::socials
  [tiborpilz](https://github.com/tiborpilz) on Github <Github />  
  [@tibor@bumscode](https://bumscode.com/@tibor) on Mastodon <Mastodon />
::

---

# Agenda

1. Rust in a Nutshell (for TS Devs)
2. What is Leptos?
3. Reactivity: Signals vs Vue
4. SSR & Hydration
5. Server Functions vs APIs
6. JS Interop & WASM
7. Takeaways for TS Devs

---

# Rust in a Nutshell

::v-clicks
- Modern, fast, statically typed (like TS, but compiled)
- Memory safety via ownership (no GC)
- No nulls, no uncaught exceptions by default
- Compiles to native code & WebAssembly
::

<!--
Rust aims for C/C++ performance with high-level ergonomics. Ownership model enforces memory/thread safety at compile time. No garbage collector, so performance is predictable. Errors are handled with Result<T, E> instead of exceptions.
-->

---

# Rust Ownership Model

::v-clicks
- Each value has a single owner
- Borrowing checked at compile time
- Prevents data races & null-pointer bugs
- Contrast: JS/TS uses GC, allows null/undefined
::

---

# Error Handling: Rust vs TypeScript

::v-clicks
- Rust: No nulls, no exceptions by default
- Use `Option<T>` for optional values
- Use `Result<T, E>` for errors
- Forces explicit error handling
::

---

# Rust & WebAssembly

::v-clicks
- Rust compiles to efficient machine code
- Can target WebAssembly (WASM)
- Enables running Rust in the browser
- Leptos uses WASM for client-side code
::

---

# What is Leptos?

::v-clicks
- Full-stack Rust web framework
- Write frontend & backend in one codebase
- Supports SSR, client-side, and hybrid apps
- Isomorphic: server functions callable from client
- Fine-grained reactivity (no virtual DOM)
::

---

# Leptos: Mental Model

::v-clicks
- Like combining Vue (UI) + NestJS (server) in Rust
- Components, state, routing feel familiar
- One language for UI & backend logic
::

---

# Leptos Reactivity: Signals

::v-clicks
- Signals: reactive values, notify subscribers on change
- Similar to Vue's ref/reactivity
- Built-in state management
::

---

# Vue Ref Example

```typescript
import { ref } from 'vue';

const count = ref(0);

function increment() {
  count.value++;
}
```

---

# Leptos Signal Example

```rust
let (count, set_count) = leptos::signal(0);

let increment = move |_| set_count.update(|c| *c += 1);
```

<!--
You might've seen something similar with the `useState` hooks in React
-->


---

# Leptos Counter Example

```rust {*|1,2|3|4,5|6-12}
#[component]
fn Counter() -> impl IntoView {
    let (count, set_count) = leptos::signal(0);
    let increment = move |_| set_count.update(|c| *c += 1);
    let decrement = move |_| set_count.update(|c| *c -= 1);
    view! {
       <div class="counter">
         <button on:click=decrement>"-1"</button>
         <span>"Count: " {count} </span>
         <button on:click=increment>"+1"</button>
       </div>
    }
}
```

---

# Vue Counter Example

```vue
<template>
  <div class="counter">
    <button @click="decrement">-1</button>
    <span>Count: {{ count }}</span>
    <button @click="increment">+1</button>
  </div>
</template>
<script setup lang="ts">
import { ref } from 'vue';
const count = ref(0);
const decrement = () => { count.value -= 1; };
const increment = () => { count.value += 1; };
</script>
```

---

# SSR & Hydration in Leptos

::v-clicks
- SSR: Render HTML on server for fast first load & SEO
- Hydration: WASM takes over, attaches event handlers
- Like Nuxt.js/Vue SSR, but in Rust
::

---

# Leptos SSR Workflow

1. Server executable renders HTML
2. Client WASM bundle hydrates in browser
3. After hydration, SPA behavior

---

# SSR Benefits

::v-clicks
- Faster initial render
- Better SEO
- Streaming SSR (send HTML in chunks)
::

---

# Full-Stack Magic: Server Functions

::v-clicks
- Rust functions annotated with #[server]
- Run on server, callable from client as async fn
- Leptos generates HTTP endpoint & client stub
- No manual API layer needed
::

---

# Server Function Example

```rust
#[server]
pub async fn add_todo(title: String) -> Result<(), ServerFnError> {
    let mut conn = db().await?;
    sqlx::query("INSERT INTO todos (title) VALUES ($1)")
        .bind(title)
        .execute(&mut conn).await
        .map_err(|e| ServerFnError::ServerError(e.to_string()))?;
    Ok(())
}
```

---

# Calling Server Function in Leptos

```rust
view! {
  <button on:click=move |_| {
      spawn_local(async {
          if let Err(e) = add_todo("Buy milk".to_string()).await {
              log::error!("Failed to add todo: {e}");
          }
      });
  }>
    "Add Todo"
  </button>
}
```

---

# Server Functions: TS Comparison

::v-clicks
- NestJS: define controller, expose endpoint
- Vue: call endpoint via fetch/axios
- Leptos: one function, both API & client call
::

---

# Security Note

::v-clicks
- Server functions are public HTTP APIs
- Secure them as you would any endpoint
::

---

# JS Interop & WASM

::v-clicks
- Leptos uses wasm-bindgen & web_sys for JS interop
- Call JS/Web APIs from Rust
- Integrate JS libraries (with care)
::

---

# Example: localStorage in Rust

```rust
use web_sys::window;
if let Some(storage) = window().and_then(|w| w.local_storage().ok().flatten()) {
    storage.set_item("key", "value").expect("failed to set item");
}
```

---

# Integrating JS Libraries

::v-clicks
- Use wasm-bindgen for extern bindings
- Best for utility libs or Web APIs
- DOM-manipulating libs may conflict with Leptos
::

---

# Takeaways for TS Devs

::v-clicks
- Explicit error handling (Result mindset)
- Make invalid states unrepresentable (discriminated unions)
- Prefer immutability, minimize side effects
- Embrace strict tooling & documentation
- Rust/WASM for perf-critical TS tasks
::

---

# Rust-Inspired Error Handling in TS

```ts
import { Result, Ok, Err } from 'ts-results';
function parseJsonSafe(str: string): Result<any, Error> {
  try {
    return Ok(JSON.parse(str));
  } catch(e) {
    return Err(e instanceof Error ? e : new Error(String(e)));
  }
}
```

---

# Make Invalid States Unrepresentable

```ts
type ArticleState = 
  | { status: 'Loading' }
  | { status: 'Error'; error: string }
  | { status: 'Loaded'; content: Article };
```

---

# Conclusion

::v-clicks
- Rust & Leptos: strong guarantees, full-stack in one language
- Familiar concepts for TS devs (components, SSR, reactivity)
- Try Leptos for a small project or adopt Rust-inspired patterns in TS
::

---

# Q&A

*Thank you!*

---

# Sources & References

- [Rust Ownership Model](https://nrempel.com/understanding-rusts-ownership-model/)
- [Leptos Framework](https://github.com/leptos-rs/leptos)
- [Leptos Reactivity](https://blog.logrocket.com/migrating-javascript-frontend-leptos-rust-framework)
- [Leptos SSR](https://leptos-use.rs/server_side_rendering.html)
- [Leptos Server Functions](https://book.leptos.dev/server/25_server_functions.html)
- [Rust/TS Error Handling](https://dev.to/alexanderop/robust-error-handling-in-typescript-a-journey-from-naive-to-rust-inspired-solutions-1mdh)
