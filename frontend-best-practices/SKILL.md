---
name: frontend-best-practices
description: Frontend code quality guidelines focusing on readability, predictability, cohesion, and coupling. Use when writing or reviewing frontend code, refactoring components, or improving code quality.
user-invocable: true
---

# Frontend Best Practices

This skill provides comprehensive frontend design principles and rules to help you write high-quality, maintainable frontend code. The guidelines are organized into four core principles:

## Core Principles

### 1. **Readability**
Improving the clarity and ease of understanding code.

Key topics:
- Naming Magic Numbers
- Abstracting Implementation Details
- Separating Code Paths for Conditional Rendering
- Simplifying Complex Ternary Operators
- Reducing Eye Movement (Colocating Simple Logic)
- Naming Complex Conditions

📖 See [readability.md](readability.md) for detailed patterns and examples.

### 2. **Predictability**
Ensuring code behaves as expected based on its name, parameters, and context.

Key topics:
- Standardizing Return Types
- Revealing Hidden Logic (Single Responsibility)
- Using Unique and Descriptive Names (Avoiding Ambiguity)

📖 See [predictability.md](predictability.md) for detailed patterns and examples.

### 3. **Cohesion**
Keeping related code together and ensuring modules have a well-defined, single purpose.

Key topics:
- Considering Form Cohesion
- Organizing Code by Feature/Domain
- Relating Magic Numbers to Logic

📖 See [cohesion.md](cohesion.md) for detailed patterns and examples.

### 4. **Coupling**
Minimizing dependencies between different parts of the codebase.

Key topics:
- Balancing Abstraction and Coupling (Avoiding Premature Abstraction)
- Scoping State Management (Avoiding Overly Broad Hooks)
- Eliminating Props Drilling with Composition

📖 See [coupling.md](coupling.md) for detailed patterns and examples.

## How to Use This Skill

When working on frontend code:

1. **Writing new code**: Apply these principles from the start to ensure quality
2. **Code review**: Use these guidelines as a checklist for reviewing pull requests
3. **Refactoring**: Identify areas that violate these principles and improve them
4. **Team discussions**: Reference specific patterns when discussing code design

## Quick Reference

For specific scenarios, consult the appropriate section:

- Need to improve code clarity? → Check **Readability** guidelines
- Code behavior is confusing? → Review **Predictability** patterns
- Files seem disorganized? → Apply **Cohesion** principles
- Components too interdependent? → Use **Coupling** reduction techniques

## Additional Resources

Each principle document contains:
- **Rule**: Clear, actionable guideline
- **Reasoning**: Why this pattern matters
- **Recommended Pattern**: Code examples showing the right way
- **Guidance**: When and how to apply the pattern

Load the specific principle documents above when you need detailed examples and explanations for that area.
