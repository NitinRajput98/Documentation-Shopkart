# AI Code Assistants Usage

As part of developing the Shop Kart prototype, we leveraged AI-powered coding assistants to accelerate development, ensure consistency, and explore alternative implementations. Below is an overview of which assistants we evaluated, how we used them, our prompting strategy, and why we ultimately chose the ones we did.

---

## 1. Assistants Evaluated

| Assistant            | Provider      | Mode                        | Notes                                    |
|----------------------|---------------|-----------------------------|------------------------------------------|
| **ChatGPT (GPT-4)**  | OpenAI        | Chat & Code Generation      | Conversational, great for complex logic  |
| **GitHub Copilot**   | GitHub/AI21   | Inline VS Code Suggestions  | Fast completions, IDE-integrated         |
| **TabNine**          | TabNine Inc.  | Inline VS Code Suggestions  | Lightweight, local model options         |
| **Codeium**          | Codeium Inc.  | Inline & Snippet Generation | Free tier, good for boilerplate          |

---

## 2. Selection Criteria

- **Accuracy of generated code** (passes our linters/tests)
- **Context-awareness** (understands our project’s file structure)
- **Prompt flexibility** (chat vs. inline)
- **Integration experience** (IDE plugins vs. web interface)
- **Privacy & cost** (local model vs. cloud, free tier limits)

After a week of parallel trials, **ChatGPT (GPT-4)** and **GitHub Copilot** emerged as the most valuable:

- **ChatGPT (GPT-4)** for architecture diagrams, step-by-step guides, and multi-file refactors.
- **GitHub Copilot** for day-to-day inline completions and boilerplate.

---

## 3. Prompting Strategy

1. **Scoped requests**  
   - *Example:* “Generate a React component that fetches a paginated product list from `/api/products`, shows a skeleton loader during fetch, and infinite-scrolls as the user reaches the bottom.”

2. **Iterative refinement**  
   - Start with a minimal prompt, inspect the output, then follow up with “Refine to handle error states” or “Convert styling to Tailwind classes.”

3. **Context injection**  
   - Pasted snippets of existing code or directory structures to keep suggestions aligned.

4. **Multi-step breakdown**  
   - Complex features (e.g. cart dropdown animation + skeleton loader) were broken into:
     1. UI structure
     2. State & hooks
     3. Animation classes
     4. Loading skeleton

---

## 4. How We Used Each Assistant

### ChatGPT (GPT-4)
- **Architecture & Docs**  
  Asked for system-level diagrams, README outlines, and step-by-step Azure deployment guides.
- **Refactoring & Patterns**  
  “Rewrite this Flask route to use SQLAlchemy batch updates” or “Add optimistic React Query cache updates.”
- **Edge-case brainstorming**  
  “How should we handle CORS preflight failures in Flask?”  

### GitHub Copilot
- **Inline completions**  
  Quickly scaffolded new React page components, Tailwind classes.
- **Type annotations & PropTypes**  
  Suggested prop validation on React components.
- **Quick fixes**  
  Catching typos and forgetting dependencies in `useEffect`.

---

## 5. Why We Liked Them

- **ChatGPT (GPT-4)**  
  - **Pros:** Deep conversational context, multi-file awareness in chat, excellent at documentation.  
  - **Cons:** Requires copy/pasting code snippets into a separate window, limited local IDE integration.

- **GitHub Copilot**  
  - **Pros:** Instant inline suggestions, learns from your codebase, keeps your hands on the keyboard.  
  - **Cons:** Sometimes over-eager, may suggest unnecessary dependencies.

---

## 6. Best Practices & Takeaways

- **Always review generated code** for security, performance, and style consistency.  
- **Use AI assistants as collaborators**, not as a source of truth.  
- **Document your prompts** alongside generated code when it unlocks non-obvious logic.  
- **Maintain human oversight** on complex business rules, data governance, and compliance.

---

> _By thoughtfully integrating AI assistants, we sped up routine tasks, elevated our documentation quality, and remained focused on core business logic—while ensuring every line of code was reviewed and aligned with Shop Kart’s standards._  
