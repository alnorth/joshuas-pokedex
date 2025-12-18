# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

This is a single-file HTML Pokédex application built with React (loaded via CDN) that integrates with the PokeAPI. The entire application lives in `index.html` with inline JavaScript using Babel for JSX transformation.

## Architecture

### Single-File Application Structure

The application is entirely self-contained in `index.html`:
- **React/React-DOM**: Loaded via unpkg CDN (v18 production builds)
- **Babel Standalone**: Transforms JSX in-browser
- **Tailwind CSS**: Loaded via CDN for styling
- **No build process**: Runs directly in browser without compilation

### Main Components

1. **PokemonExplorer** (index.html:131) - Root component managing:
   - Pokemon data fetching from PokeAPI
   - Search and type filtering
   - View mode (grid/list) toggling
   - Type effectiveness chart display
   - Progressive loading with batch fetching (50 Pokemon at a time)

2. **PokemonDetail** (index.html:672) - Detail view showing:
   - Pokemon stats (HP, Attack, Defense, etc.)
   - Evolution chain with detailed trigger information
   - Types and visual artwork

3. **TypeEffectivenessChart** (index.html:1082) - Type effectiveness display showing:
   - Card-based layout (responsive grid: 1 column mobile, 2 medium, 3 large screens)
   - Each type card displays weaknesses, resistances, and immunities
   - Color-coded type badges for visual clarity

### Data Flow

- **Pokemon List**: Fetched from `https://pokeapi.co/api/v2/pokemon?limit=1025&offset=0`
- **Pokemon Details**: Individual fetches to `https://pokeapi.co/api/v2/pokemon/{id}`
- **Evolution Chains**: Two-step process:
  1. Fetch species data: `https://pokeapi.co/api/v2/pokemon-species/{id}`
  2. Fetch evolution chain from URL in species data
- **Caching**: Evolution chains are cached in component state to avoid redundant API calls

### Evolution System

The evolution chain parser (index.html:262-487) handles complex evolution requirements:

- **Trigger Types**: level-up, trade, use-item, shed, other
- **Detailed Conditions**: Minimum level, happiness, affection, beauty, time of day, location, known moves, party composition, physical stats comparison, gender, weather, device orientation, trade partners
- **Special "Other" Triggers**: Hardcoded lookup table (index.html:292-304) for game-specific mechanics like:
  - Walk-based evolutions (Pawmot, Brambleghast, Rabsca)
  - Combat requirements (Annihilape, Kingambit)
  - Collection mechanics (Gholdengo)
  - Special items with conditions (Ursaluna)
- **Fallback Handling**: Unhandled evolution details are displayed as hints to aid debugging

## Development

### Running the Application

Simply open `index.html` in a web browser. No build step required.

### Testing

`test-evolution.html` is a standalone test file for debugging evolution data from PokeAPI, specifically testing the Primeape → Annihilape evolution chain.

**Browser Testing with Playwright MCP**:
- The Playwright MCP server can be used to test the application in a real browser
- Screenshots are saved to `.playwright-mcp/` directory during testing
- **IMPORTANT**: After completing browser testing, clear the screenshot directory: `rm -rf .playwright-mcp`
- The `.playwright-mcp/` directory is already in `.gitignore`

### Making Changes

Since this is a single-file app:
- All JavaScript is in `<script type="text/babel">` tag in index.html
- All styles are in `<style>` tag or inline Tailwind classes
- React component logic uses hooks (useState, useEffect)
- Icons are defined as inline SVG components (Zap, ChevronRight, Search, Grid3x3, List)

### Key Data Structures

**Pokemon Object**:
```javascript
{
  id: number,
  name: string,
  types: string[], // e.g., ['fire', 'flying']
  evolves: null
}
```

**Evolution Chain Item**:
```javascript
{
  id: number,
  name: string,
  method: string // Human-readable evolution requirement
}
```

**Type Colors**: Defined in typeColors object (index.html:144-150)
**Type Effectiveness**: Defined in typeEffectiveness object (index.html:152-171)

## Common Modifications

### Adding New Evolution Triggers

If PokeAPI introduces new evolution requirements:
1. Add to the `otherEvolutionDetails` lookup table if trigger is "other" (index.html:292-304)
2. Or add new condition parsing in the evolution detail handler (index.html:309-472)
3. Ensure unhandled fields are displayed via the fallback mechanism (index.html:441-469)

### Adjusting Pokemon Data Range

Modify the limit parameter in the PokeAPI fetch (index.html:180):
```javascript
const listResponse = await fetch('https://pokeapi.co/api/v2/pokemon?limit=1025&offset=0');
```

### Styling Changes

- Global styles: Edit `<style>` tag (index.html:11-91)
- Component styles: Modify Tailwind classes in JSX
- Animations: Defined in CSS keyframes (pulse, slideIn)
