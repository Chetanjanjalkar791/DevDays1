---
description: 'Comment philosophy, JSDoc/TSDoc, and documentation standards'
applyTo: '**/*.{ts,astro,tsx,jsx}'
---

# Documentation and Commenting Standards

## Comment Philosophy

**Comment intent, not mechanics.** Code that is clear and well-structured should speak for itself. Comments exist to explain *why* a piece of code exists, *what problem it solves*, and the reasoning behind non-obvious decisions. Comments should **not** restate what the code already says.

### What to Comment

- **Intent and rationale**: Why this approach was chosen over alternatives, especially for non-obvious solutions
- **Business logic**: Constraints, edge cases, or domain-specific reasoning
- **Workarounds**: Explanations for temporary fixes, platform-specific code, or compatibility concerns
- **Future improvements**: TODOs that signal planned work with clear context
- **Configuration and constants**: Why a value is set to a particular number

### What NOT to Comment

- **Restating code**: Don't write comments that merely paraphrase the line(s) below
  ```ts
  // BAD: Comment just repeats the code
  // Increment the counter
  counter++;

  // GOOD: Code is clear, no comment needed (or explain why if there's a reason)
  counter++;
  ```

- **Obvious mechanics**: Don't explain what a standard library function or language construct does
  ```ts
  // BAD: This just explains Array.map
  // Map over games and transform each one
  const titles = games.map((game) => game.title);

  // GOOD: If needed, explain *why* we're extracting titles
  // Extract game titles for the search index
  const titles = games.map((game) => game.title);
  ```

- **Redundant inline comments**: Let well-named variables and functions do the talking

### Keeping Comments Current

Treat outdated or incorrect comments as bugs. When you change code, update or remove related comments in the same change. Stale comments are worse than no comments — they mislead and erode trust.

## JSDoc / TSDoc for Exported Functions

Every **exported** function in `db/` and `src/lib/` must include a JSDoc/TSDoc comment block describing:

- **Summary**: One-line description of what the function does
- **Parameters**: Each parameter with its type and purpose (even though TypeScript has the type)
- **Return value**: What the function returns and any important caveats
- **Example** (optional): For complex helpers, include a usage example

### JSDoc Pattern

```ts
/**
 * Fetches all published games, ordered by title.
 * @param db - The Drizzle database client (injectable for testability)
 * @returns Array of games with their associated publisher and category
 */
export async function getAllGames(db: Database): Promise<Game[]> {
  // implementation
}
```

### Complex Helper with Example

```ts
/**
 * Fetches a paginated list of games.
 * @param db - The Drizzle database client
 * @param page - The page number (1-indexed)
 * @param limit - Number of games per page (default: 10)
 * @returns Object containing games for the page and total count
 * @example
 * const result = await getPaginatedGames(db, 1, 20);
 * console.log(result.games, result.total);
 */
export async function getPaginatedGames(
  db: Database,
  page: number,
  limit: number = 10
): Promise<{ games: Game[]; total: number }> {
  // implementation
}
```

### Injectable `db` Parameter

When documenting the `db` parameter, always call out that it's injectable for testing:

```ts
/**
 * Looks up a game by ID.
 * @param db - The Drizzle database client (injectable for testability)
 * @param id - The numeric game ID
 * @returns The game with its relations, or null if not found
 */
export async function getGameById(db: Database, id: number): Promise<Game | null> {
  // implementation
}
```

### Transforms and Pure Functions

For pure functions in `db/transforms.ts`, document the transformation logic:

```ts
/**
 * Generates a deterministic star rating from a game title.
 * Always returns a value between 3.0 and 5.0 to ensure realistic ratings.
 * @param title - The game title
 * @returns A consistent rating for the given title
 */
export function ratingFromTitle(title: string): number {
  // implementation — must be deterministic!
}
```

## Component Props Documentation

Every **reusable `.astro` component** must document its `Props` interface with a JSDoc comment:

```astro
---
/**
 * Displays a single game card with title, description, rating, and publisher.
 * @param game - The game object with title, description, starRating, and publisher
 * @param showDescription - Whether to show the full description (default: true)
 */
interface Props {
  game: Game;
  showDescription?: boolean;
}

const { game, showDescription = true } = Astro.props;
---

<!-- Component markup -->
```

### Layout Props

Layouts that accept props should document them similarly:

```astro
---
/**
 * Main site layout with header, footer, and content area.
 * @param title - Page title for the <title> tag and heading
 * @param description - Optional meta description
 */
interface Props {
  title: string;
  description?: string;
}

const { title, description } = Astro.props;
---

<!DOCTYPE html>
<html>
  <!-- Layout content -->
</html>
```

## TypeScript Conventions

### Type Annotations

- **Explicit types on exported functions**: Always include parameter and return types for exported functions
  ```ts
  // GOOD
  export function getGameById(db: Database, id: number): Promise<Game | null> {
    // implementation
  }
  ```

- **Explicit types on public properties**: Class properties and module exports should have explicit types
  ```ts
  // GOOD
  export const DEFAULT_PAGE_SIZE: number = 10;

  export interface Game {
    id: number;
    title: string;
    starRating: number | null;
  }
  ```

- **Infer types for local variables** (when obvious): Local variables can use inference when the type is clear from context
  ```ts
  // GOOD — type is obvious from the literal
  const pageSize = 10;

  // GOOD — type is obvious from the function return
  const games = await getAllGames(db);
  ```

### Naming Conventions

- **Descriptive names**: Use clear, descriptive names that express intent
  ```ts
  // BAD
  const g = getGames();
  const sr = 4.5;

  // GOOD
  const games = getAllGames();
  const starRating = 4.5;
  ```

- **Consistent terminology**: Use the same term across the codebase (e.g., always "game" not "game" and "product")
- **Avoid single-letter variables** except in obvious loops: `for (let i = 0; i < array.length; i++)`

### Imports and Exports

- **Named exports for functions and types**: Makes it clear what is part of the module's API
  ```ts
  // GOOD
  export function getAllGames(db: Database): Promise<Game[]> { }
  export type Game = typeof games.$inferSelect;
  ```

- **Default exports for layouts and pages** (Astro convention)
  ```astro
  // In Layout.astro
  export default Layout; // or just the component itself
  ```

## ESLint and Type Checking

- Run `npm run lint` to check TypeScript + Astro code quality
- Run `npm run typecheck:all` to verify types across pure TypeScript and `.astro` files
- Both commands must pass before committing
- Address linting issues in code (don't disable rules without clear justification in a comment)

## Summary

| Aspect | Rule |
|--------|------|
| **Comments** | Explain *why*, not *what* — remove comments that restate code |
| **Exported functions** | JSDoc with summary, parameters, return, and optional example |
| **Injectable `db`** | Always document that the parameter is injectable for testing |
| **Component Props** | JSDoc comment on the `Props` interface describing the component |
| **Type annotations** | Explicit on exported functions and properties; infer for local variables |
| **Names** | Descriptive, consistent across the codebase |
| **Linting** | `npm run lint` and `npm run typecheck:all` must pass |
