# Bomberman Frontend Architecture Document

## Template and Framework Selection

Based on the PRD and Brief, the project will be a web-based game. The PRD suggests using a game library like Phaser.js or building a custom engine with Canvas/WebGL. Given the nature of the game (real-time rendering, grid-based movement, animations), a dedicated game development library is highly recommended over raw Canvas APIs for productivity and maintainability.

**Decision: Use Phaser.js**

*   **Rationale:** Phaser.js is a mature, well-documented, and popular framework for 2D HTML5 game development. It handles rendering, asset loading, input management, physics (if needed), and the game loop efficiently. It aligns well with the project's need for a performant, web-based 2D grid game.
*   **Starter Template:** We will use Vite with a TypeScript template (`npm create vite@latest my-bomberman --template vanilla-ts`) and then integrate Phaser.js. Vite offers fast development server and build times, and good TypeScript support, fitting the PRD's technical assumptions.
*   **Constraints:** Using Phaser.js means our architecture will be shaped by its paradigms (Scenes, GameObjects, etc.).

### Change Log
| Date | Version | Description | Author |
| :--- | :--- | :--- | :--- |
| 2025-08-22 | 1.0.0 | Initial Draft based on PRD/Brief | Winston (Architect) |

## Frontend Tech Stack

| Category | Technology | Version | Purpose | Rationale |
| :--- | :--- | :--- | :--- | :--- |
| Framework | Phaser.js | ^3.87.0 | 2D Game Framework | Specialized for HTML5 game development, handles rendering, game loop, input. |
| UI Library | N/A | N/A | Game UI | Phaser.js provides primitives for game UI. Standard HTML/CSS for menus/settings if needed. |
| State Management | N/A | N/A | Game State | Phaser.js Scenes and GameObjects manage game state. For complex UI state, consider Context API or a lightweight solution if needed. |
| Routing | N/A | N/A | Scene Navigation | Phaser.js Scenes inherently manage navigation between different game states (Menu, Game, Pause). |
| Build Tool | Vite | ^5.4.0 | Bundling, Dev Server | Fast, modern build tool with good TypeScript support, aligns with PRD assumptions. |
| Styling | CSS3 | N/A | Menus/Non-game UI | Standard CSS for any non-game UI elements (menus, settings). Phaser handles in-game visuals. |
| Testing | Vitest/Jest + Testing Library | ^2.0.0 | Unit/Integration Tests | Vite's native testing solution or Jest for logic. Testing Library for DOM/UI tests if needed. |
| Component Library | N/A | N/A | Reusable UI Components | Not applicable for core game. Standard HTML/CSS/JS for menus. |
| Form Handling | N/A | N/A | Forms | Not a primary concern for core gameplay. Standard HTML forms or simple JS for settings if needed. |
| Animation | Phaser.js/Tween.js | Built-in | Game Animations | Phaser provides robust animation/tweening capabilities. |
| Dev Tools | Vite Dev Server, Browser Dev Tools | N/A | Development | Vite provides HMR. Browser tools for debugging. |

## Project Structure

```
bomberman/
├── .gitignore
├── index.html                 # Main HTML entry point
├── package.json               # NPM dependencies and scripts
├── vite.config.ts             # Vite configuration
├── tsconfig.json              # TypeScript configuration
├── src/                       # Source code
│   ├── main.ts                # Main application entry point (Phaser game initialization)
│   ├── types/                 # Shared TypeScript types and interfaces
│   │   └── game.ts            # Core game types (Player, Bomb, Grid, etc.)
│   ├── scenes/                # Phaser Scenes
│   │   ├── BootScene.ts       # Initial loading scene
│   │   ├── MenuScene.ts       # Main menu scene
│   │   ├── GameScene.ts       # Main game scene
│   │   ├── PauseScene.ts      # Pause overlay scene
│   │   └── GameOverScene.ts   # Game over/results scene
│   ├── objects/               # Phaser GameObjects (custom or specific instances)
│   │   ├── Player.ts          # Player character GameObject
│   │   ├── Bomb.ts            # Bomb GameObject
│   │   ├── Explosion.ts       # Explosion effect GameObject
│   │   └── Block.ts           # Destructible/Indestructible block GameObjects
│   ├── utils/                 # Utility functions
│   │   ├── grid.ts            # Grid logic helpers
│   │   ├── collision.ts       # Collision detection helpers
│   │   └── constants.ts       # Game constants (grid size, bomb timers, etc.)
│   ├── assets/                # Game assets (images, sounds, data files)
│   │   ├── images/            # Sprite sheets, tilemaps, UI images
│   │   ├── sounds/            # Sound effects, background music
│   │   └── data/              # Level data, configuration files
│   └── styles/                # Global CSS styles (for menus, if any)
│       └── main.css           # Main stylesheet
├── tests/                     # Test files
│   ├── unit/                  # Unit tests for game logic, utils
│   └── integration/           # Integration tests for scenes, objects
└── dist/                      # Build output (ignored by Git)
```

## Component Standards

### Component Template

For Phaser, "components" are typically GameObjects or mixins. A base template for a GameObject might look like this:

```typescript
// src/objects/MyGameObject.ts
import Phaser from 'phaser';

export interface MyGameObjectConfig {
  // Define specific configuration options for this object
  initialX: number;
  initialY: number;
  // ... other config
}

export class MyGameObject extends Phaser.GameObjects.Sprite { // Or Image, Container, etc.
  constructor(scene: Phaser.Scene, config: MyGameObjectConfig) {
    super(scene, config.initialX, config.initialY, 'sprite-key'); // Assuming a sprite

    // Initialize properties based on config
    // this.customProperty = config.someValue;

    // Add to scene
    scene.add.existing(this);

    // Perform initial setup
    this.setup();
  }

  private setup(): void {
    // Perform initial setup logic, e.g., animations, physics, input
    // this.setInteractive(); // Example
  }

  // Public methods for interaction/control
  public doSomething(): void {
    // Implementation
  }

  // Phaser update method (if needed for continuous logic)
  // Note: Prefer using Tweens/physics or Scene update for performance
  // override update(time: number, delta: number): void {
  //   super.update(time, delta);
  //   // Update logic here (use sparingly)
  // }
}
```

### Naming Conventions

*   **Files:** `camelCase.ts` for general files, `PascalCase.ts` for classes/components (e.g., `player.ts`, `Player.ts`).
*   **Classes/Components:** `PascalCase` (e.g., `Player`, `Bomb`, `GameScene`).
*   **Functions/Variables:** `camelCase` (e.g., `createPlayer`, `playerSpeed`).
*   **Constants:** `UPPER_SNAKE_CASE` (e.g., `GRID_WIDTH`, `BOMB_TIMER`).
*   **Types/Interfaces:** `PascalCase`, often prefixed with `I` for interfaces if clarity is needed (e.g., `PlayerState`, `IGameConfig`).
*   **Phaser Scenes:** `PascalCase` ending in `Scene` (e.g., `MenuScene`, `GameScene`).
*   **Phaser GameObjects:** `PascalCase` (e.g., `Player`, `Bomb`).

## State Management

### Store Structure

For a game like Bomberman, state is primarily managed by Phaser itself within Scenes and GameObjects. However, for global settings or simple UI state, a lightweight approach is sufficient.

```
src/
├── state/
│   └── gameSettings.ts   # Simple module for global settings (volume, controls, etc.)
```

### State Management Template

If a more formal state management is needed (e.g., for a complex menu system), a simple global state object or Context API could be used.

```typescript
// src/state/gameSettings.ts
interface GameSettings {
  volume: number;
  // ... other settings
}

let settings: GameSettings = {
  volume: 1.0,
  // ... default values
};

export const getSettings = (): GameSettings => settings;

export const updateSettings = (newSettings: Partial<GameSettings>): void => {
  settings = { ...settings, ...newSettings };
};

// Usage in a scene or component:
// import { getSettings, updateSettings } from '../state/gameSettings';
// const currentVolume = getSettings().volume;
// updateSettings({ volume: 0.5 });
```

## API Integration

### Service Template

As the initial MVP is client-side only, there might not be an API. However, if multiplayer or a backend for scores/user data is added later, a service pattern would be useful.

```typescript
// src/services/apiService.ts
class ApiService {
  private baseUrl: string;

  constructor(baseUrl: string) {
    this.baseUrl = baseUrl;
  }

  async fetchScores(): Promise<any[]> { // Replace 'any' with specific type
    try {
      const response = await fetch(`${this.baseUrl}/scores`);
      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
      return await response.json();
    } catch (error) {
      console.error("Failed to fetch scores:", error);
      throw error; // Re-throw or handle gracefully
    }
  }

  async submitScore(scoreData: any): Promise<void> { // Replace 'any' with specific type
    try {
      const response = await fetch(`${this.baseUrl}/scores`, {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json',
        },
        body: JSON.stringify(scoreData),
      });

      if (!response.ok) {
        throw new Error(`HTTP error! status: ${response.status}`);
      }
    } catch (error) {
      console.error("Failed to submit score:", error);
      throw error;
    }
  }
}

// Export a singleton instance
export const apiService = new ApiService(import.meta.env.VITE_API_BASE_URL || 'http://localhost:3000/api');
```

### API Client Configuration

Vite allows using environment variables via `import.meta.env`. Define the base URL in `.env` files.

```env
# .env
VITE_API_BASE_URL=http://localhost:3000/api
```

The `apiService` example above uses `import.meta.env.VITE_API_BASE_URL`.

## Routing

### Route Configuration

Phaser handles "routing" through its Scene system. You define scenes and switch between them.

```typescript
// src/main.ts or a dedicated scene manager
import Phaser from 'phaser';
import BootScene from './scenes/BootScene';
import MenuScene from './scenes/MenuScene';
import GameScene from './scenes/GameScene';
import PauseScene from './scenes/PauseScene';
import GameOverScene from './scenes/GameOverScene';

const config: Phaser.Types.Core.GameConfig = {
  type: Phaser.AUTO,
  width: 800,
  height: 600,
  scene: [BootScene, MenuScene, GameScene, PauseScene, GameOverScene], // List of scenes
  // ... other config
};

const game = new Phaser.Game(config);

// To switch scenes within a scene:
// this.scene.start('MenuScene');
// this.scene.start('GameScene');
// this.scene.pause(); // Pause current scene
// this.scene.resume(); // Resume paused scene
```

## Styling Guidelines

### Styling Approach

For a game primarily built with Phaser, in-game visuals are handled by the framework. Standard HTML/CSS is used for menus, settings screens, or any non-game UI.

*   **Approach:** Use plain CSS or a lightweight CSS framework if needed for menus.
*   **Structure:** Global styles in `src/styles/main.css`. Component-specific styles can be co-located if necessary.

### Global Theme Variables

```css
/* src/styles/main.css */
:root {
  /* Colors */
  --color-primary: #e51400; /* Example primary color, adjust for Bomberman theme */
  --color-secondary: #2c3e50; /* Example secondary */
  --color-background: #ecf0f1; /* Example background */
  --color-text: #333; /* Example text */
  --color-accent: #3498db; /* Example accent */

  /* Spacing */
  --spacing-xs: 0.25rem;
  --spacing-sm: 0.5rem;
  --spacing-md: 1rem;
  --spacing-lg: 1.5rem;
  --spacing-xl: 2rem;

  /* Typography */
  --font-family-primary: 'Press Start 2P', cursive; /* Retro/pixel font for Bomberman feel */
  --font-family-secondary: Arial, sans-serif; /* Fallback */
  --font-size-sm: 0.75rem;
  --font-size-md: 1rem;
  --font-size-lg: 1.25rem;
  --font-size-xl: 1.5rem;

  /* Shadows */
  --shadow-sm: 0 1px 3px rgba(0,0,0,0.12), 0 1px 2px rgba(0,0,0,0.24);
  --shadow-md: 0 3px 6px rgba(0,0,0,0.16), 0 3px 6px rgba(0,0,0,0.23);
  --shadow-lg: 0 10px 20px rgba(0,0,0,0.19), 0 6px 6px rgba(0,0,0,0.23);

  /* Borders */
  --border-radius-sm: 4px;
  --border-radius-md: 8px;
  --border-radius-lg: 12px;
}

/* Dark mode (if applicable) */
@media (prefers-color-scheme: dark) {
  :root {
    --color-background: #2c3e50;
    --color-text: #ecf0f1;
    --color-primary: #e74c3c;
    --color-secondary: #34495e;
    --color-accent: #2980b9;
  }
}

body {
  margin: 0;
  padding: 0;
  font-family: var(--font-family-secondary);
  background-color: var(--color-background);
  color: var(--color-text);
}

/* Apply pixel font to headings or specific menu elements for thematic consistency */
h1, h2, h3, .pixel-font {
  font-family: var(--font-family-primary);
}
```

## Testing Requirements

### Component Test Template

Testing for Phaser games can be challenging. Focus on testing game logic, utility functions, and potentially the state of scenes/objects if they encapsulate logic. UI/rendering tests are often less critical initially.

```typescript
// tests/unit/grid.test.ts
import { describe, it, expect } from 'vitest'; // Or jest
import { isValidMove } from '../../src/utils/grid';
import { GRID_WIDTH, GRID_HEIGHT } from '../../src/utils/constants';

describe('Grid Utilities', () => {
  it('should allow valid moves within grid bounds', () => {
    expect(isValidMove(0, 0)).toBe(true);
    expect(isValidMove(GRID_WIDTH - 1, GRID_HEIGHT - 1)).toBe(true);
  });

  it('should disallow moves outside grid bounds', () => {
    expect(isValidMove(-1, 0)).toBe(false);
    expect(isValidMove(0, -1)).toBe(false);
    expect(isValidMove(GRID_WIDTH, 0)).toBe(false);
    expect(isValidMove(0, GRID_HEIGHT)).toBe(false);
  });
});
```

### Testing Best Practices

1.  **Unit Tests:** Test pure functions and game logic (e.g., collision detection, scoring rules, grid validation).
2.  **Integration Tests:** Test interactions between game objects or simple scene setup logic.
3.  **E2E Tests:** Potentially use tools like Playwright or Cypress for high-level gameplay flows, but this is often complex for games.
4.  **Coverage Goals:** Aim for high coverage (e.g., 80%) on core logic functions.
5.  **Test Structure:** Follow Arrange-Act-Assert.
6.  **Mock External Dependencies:** For tests, mock random number generators, timers, or API calls if services are used.

## Environment Configuration

List required environment variables. Vite uses `VITE_` prefix for client-side environment variables.

```env
# .env
VITE_API_BASE_URL=http://localhost:3000/api # If backend services are added
VITE_DEBUG_MODE=false # Custom flag for enabling debug features in development
VITE_GAME_VERSION=1.0.0 # Example for displaying version
```

Access them in code:
```typescript
const apiUrl = import.meta.env.VITE_API_BASE_URL;
const isDebug = import.meta.env.VITE_DEBUG_MODE === 'true';
```

## Frontend Developer Standards

### Critical Coding Rules

*   **Use TypeScript Strictly:** Enable strict type checking (`strict: true` in `tsconfig.json`) to catch errors early.
*   **Modularity:** Keep code in small, focused modules/files.
*   **Naming:** Follow the established naming conventions consistently.
*   **Phaser Best Practices:**
    *   Avoid heavy logic in `update()` methods of GameObjects/Scenes. Use Tweens, physics, or Scene `update` for continuous actions.
    *   Manage object lifecycle (creation, destruction) carefully to prevent memory leaks.
    *   Use object pooling for frequently created/destroyed objects like bullets/explosions.
*   **Performance:** Profile regularly. Optimize rendering (sprite batching), minimize object creation/destruction in loops.
*   **State Management:** Rely on Phaser's built-in state management (Scenes, GameObject properties) for game state. Use simple global objects or Context for UI settings.
*   **Asset Loading:** Use Phaser's Loader to manage assets efficiently.

### Quick Reference

*   **Dev Server:** `npm run dev` (from Vite)
*   **Build:** `npm run build` (from Vite)
*   **Test:** `npm run test` (assuming Vitest/Jest setup)
*   **Key Phaser Imports:**
    ```typescript
    import Phaser from 'phaser';
    import { Scene, GameObjects, Input, Physics, Tweens, ... } from 'phaser';
    ```
*   **File Naming:** `camelCase.ts` for files, `PascalCase.ts` for classes/components.
*   **Project Patterns:**
    *   Scenes for major game states.
    *   GameObjects for interactive entities (Player, Bomb).
    *   Utility functions for shared logic.
    *   Constants file for magic numbers/config.