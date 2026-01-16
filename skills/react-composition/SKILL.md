---
name: react-composition
description: Guide for building scalable React components using composition instead of boolean props. Use when building complex UI components that have multiple variations, when refactoring components with many boolean props (isEditing, isThread, shouldRender*), or when asked to avoid "boolean prop hell". This pattern makes code easier for both humans and AI to work with.
version: "1.0.0"
author: wenhsu
license: MIT
platforms:
  - cursor
  - claude-code
  - codex
  - opencode
tags:
  - react
  - composition
  - patterns
  - component-design
---

# React Composition Pattern

Avoid boolean prop hell. Use composition to build flexible, maintainable components.

## The Problem: Boolean Prop Hell

```tsx
// ❌ BAD: Monolithic component with boolean props
<UserForm
  isUpdateUser
  isEditNameOnly
  hideWelcomeMessage
  hideTermsAndConditions
  isSlugRequired={false}
  onSuccess={...}
/>
```

This leads to:
- Components with 30+ conditions scattered everywhere
- Hard to understand which props affect which parts
- Difficult for both humans and AI to modify

## The Solution: Composition

### Pattern 1: Compound Components (like Radix)

Instead of one monolith with optional props, split into composable parts:

```tsx
// ✅ GOOD: Compound components
<Composer.Provider>
  <Composer.DropZone />  {/* Only render if needed */}
  <Composer.Frame>
    <Composer.Header />
    <Composer.Input />
    <Composer.Footer>
      <Composer.Actions.PlusMenu />
      <Composer.Actions.Emoji />
      <Composer.Actions.TextFormat />
      <Composer.Submit />
    </Composer.Footer>
  </Composer.Frame>
</Composer.Provider>
```

### Pattern 2: Just Use JSX

Don't pass arrays of actions with boolean flags. Just render what you need:

```tsx
// ❌ BAD: Array with conditions
const actions = [
  { id: 'plus', hidden: isEditing },
  { id: 'emoji', isMenu: true, menuItems: [...] },
  { id: 'format' },
]

// ✅ GOOD: Just JSX
<Footer>
  {!isEditing && <PlusMenu />}
  <EmojiButton />
  <TextFormatButton />
</Footer>

// ✅ EVEN BETTER: Separate components, no conditions
// In EditMessageComposer.tsx - only render what this variant needs
<Footer>
  <EmojiButton />
  <TextFormatButton />
  <CancelButton />
  <SaveButton />
</Footer>
```

### Pattern 3: Context Provider for State

The Provider defines the interface. Implementation is decided by the consumer:

```tsx
// Provider defines interface (state + actions)
interface ComposerContext {
  state: { text: string; attachments: File[] }
  actions: { 
    update: (text: string) => void
    submit: () => void 
  }
  meta: { inputRef: RefObject<HTMLInputElement> }
}

// Ephemeral state (forward message modal)
function ForwardMessageProvider({ children }) {
  const [state, setState] = useState({ text: '', attachments: [] })
  return (
    <ComposerContext.Provider value={{
      state,
      actions: { update: setState, submit: handleSubmit },
      meta: { inputRef }
    }}>
      {children}
    </ComposerContext.Provider>
  )
}

// Global synced state (channel composer)
function ChannelProvider({ channelId, children }) {
  const { state, actions } = useGlobalChannel(channelId) // syncs across devices
  return (
    <ComposerContext.Provider value={{ state, actions, meta: { inputRef } }}>
      {children}
    </ComposerContext.Provider>
  )
}
```

### Pattern 4: Lift State Up

When UI outside the component needs access to its state, lift the provider:

```tsx
// ❌ BAD: Trying to pass state back up
<ForwardMessageModal>
  <Composer onStateChange={setParentState} />
  <ActionBar state={parentState} /> {/* Messy */}
</ForwardMessageModal>

// ✅ GOOD: Lift provider, everything inside can access context
<ForwardMessageProvider>
  <Composer />  {/* Uses context internally */}
  <MessagePreview />  {/* Can also use context */}
  <ActionBar>
    <ForwardButton />  {/* Can call context.actions.submit */}
  </ActionBar>
</ForwardMessageProvider>
```

The `ForwardButton` is NOT inside the Composer frame, but it's inside the Provider - so it can access all state and actions.

## When to Apply

Use composition when:
- A component has 3+ boolean props that control rendering
- The same condition is checked in multiple places within a component
- Different use cases need different subsets of functionality
- State needs to be accessed by sibling components

## Key Takeaways

1. **No boolean props that determine component trees** - If a parent passes `isEditing` to control what renders, split into separate components instead
2. **Prefer JSX over config arrays** - Easier to customize, escape to custom implementations
3. **Provider defines interface, not implementation** - Swap state management at the provider level
4. **Lift state when needed** - Provider can wrap more than just the "main" component

## Reference

Based on Fernando Rojo's talk "Composition Is All You Need" at React Universe Conf 2025.
See `references/talk-transcript.md` for full context.
