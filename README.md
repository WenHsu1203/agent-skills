# Agent Skills

A collection of AI agent skills for coding assistants like Cursor, Claude Code, Codex, and OpenCode.

## Installation

```bash
# Install all skills
npx add-skill wenhsu/agent-skills

# Install specific skill
npx add-skill wenhsu/agent-skills --skill react-composition

# Install globally for specific agents
npx add-skill wenhsu/agent-skills -g -a cursor -a claude-code
```

## Available Skills

### react-composition

Guide for building scalable React components using composition instead of boolean props.

**Use when:**
- Building complex UI components with multiple variations
- Refactoring components with many boolean props (`isEditing`, `isThread`, `shouldRender*`)
- Avoiding "boolean prop hell"

**Key patterns:**
1. Compound Components (like Radix)
2. Just use JSX instead of config arrays
3. Context Provider for state interface
4. Lift state up when needed

Based on Fernando Rojo's talk "Composition Is All You Need" at React Universe Conf 2025.
https://www.youtube.com/watch?v=4KvbVq3Eg5w

## License

MIT
