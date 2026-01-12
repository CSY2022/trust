# AI Coding Agent Instructions - The Evolution of Trust

## Project Overview
Interactive game theory educational experience explaining the Prisoner's Dilemma through visual simulations. Built with vanilla JavaScript, PIXI.js (graphics), Howler.js (audio), and Tween.js (animations). No build process required - runs directly in browser.

## Core Architecture

### Slideshow System (`js/core/Slideshow.js`)
- Central orchestrator managing slide transitions
- Each slide object has lifecycle hooks: `onstart(self)`, `onend(self)`, `onjump(self)`
- Slides are pushed to global `SLIDES` array in `js/slides/*.js` files
- Navigation via `publish("slideshow/next")` or `publish("slideshow/goto", "slide_id")`
- Objects added via `self.add({id:"name", type:"ComponentClass", x:0, y:0})`

### Communication Pattern (MinPubSub)
- **Publish**: `publish("topic", [arg1, arg2])`
- **Subscribe**: `subscribe("topic", callback)` or `listen(object, "topic", callback)` (auto-unbinds on object removal)
- Common topics: `slideshow/*`, `iterated/*`, `tournament/*`, `pd/*`, `rules/*`

### Asset Loading (`js/core/Loader.js`)
- Two manifests: `Loader.manifestPreload` (splash screen), `Loader.manifest` (main content)
- Add assets: `Loader.addToManifest(Loader.manifest, {key: "path/to/asset"})`
- PIXI.js loads images/sprites (`.json` = sprite sheet, `.png` = single image)
- Howler.js loads sounds (`.mp3`)
- Access loaded sprites: `PIXI.loader.resources[key].texture`

### Translation System (`js/core/Words.js`)
- All text in `words.html` as `<p id="word_id">content</p>`
- Access via `Words.get("word_id")`
- Never hardcode text - always use `Words.get()` for internationalization

## Game Theory Core (`js/sims/PD.js`)

### Agent Strategies
Eight strategies, each with `play()` and `remember(ownMove, otherMove)` methods:
- `tft` (Tit-for-Tat): Copy opponent's last move
- `all_d` (Always Defect): Always cheat
- `all_c` (Always Cooperate): Always cooperate
- `grudge`: Cooperate until cheated, then always defect
- `prober`: Test opponent (C,D,C,C), then TFT or all_d
- `tf2t` (Tit-for-Two-Tats): Defect only after two consecutive cheats
- `pavlov`: Win-stay, lose-shift strategy
- `random`: 50/50 random choice

### Payoff Matrix
```javascript
PD.PAYOFFS = {
  P: 0,  // Punishment (both cheat)
  S: -1, // Sucker (you cooperate, they cheat)
  R: 2,  // Reward (both cooperate)
  T: 3   // Temptation (you cheat, they cooperate)
};
```
Modify via `publish("pd/editPayoffs/P", [value])` or `publish("pd/defaultPayoffs")`

### Game Functions
- `PD.playOneGame(playerA, playerB)`: Single round
- `PD.playRepeatedGame(playerA, playerB, turns)`: Multiple rounds
- `PD.playOneTournament(agents, turns)`: Round-robin tournament
- Noise factor: `PD.NOISE` (0-1) for random mistakes

## Simulation Components

### Tournament (`js/sims/Tournament.js`)
- Agents arranged in circle, connected by lines
- Evolution: eliminate worst X, reproduce best X
- Global config: `Tournament.INITIAL_AGENTS`, `Tournament.SELECTION`, `Tournament.NUM_TURNS`
- Messages: `tournament/play`, `tournament/step`, `tournament/reset`, `tournament/autoplay/start|stop`

### Iterated (`js/sims/Iterated.js`)
- Two-player repeated game visualization
- Messages: `iterated/cooperate`, `iterated/cheat`, `iterated/reset`

### UI Components (`js/core/`)
- **Button**: `{x, y, text_id, onclick, message, size, tooltip}`
- **TextBox**: `{x, y, text_id, width, height, align, color, size}`
- **Slider**: `{x, y, min, max, value, onchange}`
- **IncDecNumber**: Increment/decrement number control
- **ImageBox**: Display images

## Helper Functions (`js/lib/helpers.js`)
- `$(selector)`: Poor man's jQuery (querySelector)
- `_makeLabel(word_id, config)`: Create text label
- `_configText(config, dom)`: Apply text styling
- `_s(seconds)`: Convert seconds to animation ticks
- `_fadeIn(object, delay)`, `_hide(object)`, `_show(object)`: Animation helpers
- `Tween_get(target)`: Create Tween.js tween

## Agent Creation Pattern
```javascript
// Strategy functions are in PD.js
// Agents created dynamically by strategy name:
var agent = new TournamentAgent({
  angle: angle,
  strategy: "tft",  // matches Logic_tft function name
  tournament: self
});
```

## Development Workflow
- **Local testing**: Use `http-server` or any static file server
- **No build process**: Direct file editing
- **No tests**: Manual testing only
- **Asset paths**: All relative to project root

## Key Files to Understand
- `js/main.js`: Entry point, initialization
- `js/sims/PD.js`: Game theory logic & strategies
- `js/sims/Tournament.js`: Tournament & evolution simulation
- `js/sims/Iterated.js`: Two-player game visualization
- `js/core/Slideshow.js`: Slide management system
- `js/lib/helpers.js`: Utility functions
- `words.html`: All translatable text

## Common Patterns

### Adding a New Slide
```javascript
SLIDES.push({
  id: "my_slide",
  onstart: function(self){
    // Add objects
    self.add({id:"btn", type:"Button", x:100, y:100, text_id:"label_start"});
  },
  onend: function(self){
    // Cleanup
    self.remove("btn");
  }
});
```

### Creating Custom Component
```javascript
function MyComponent(config){
  var self = this;
  self.id = config.id;
  self.dom = document.createElement("div");
  self.dom.className = "object";
  // ... setup ...
  self.add = function(){ _add(self); };
  self.remove = function(){ _remove(self); };
}
```

### Listening to Events with Auto-Cleanup
```javascript
listen(self, "some/event", function(data){
  // Handle event
  // Automatically unsubscribed when self.remove() is called
});
```

## Important Conventions
- Use `Math.TAU` instead of `2*Math.PI`
- Store slide-specific state in global `_` object (cleared between slides)
- Always call `unlisten(self)` in component `remove()` methods
- Use `Loader.addToManifest()` before `Loader.loadAssets()`
- Coordinate system: (0,0) is top-left, slideshow is 960x540
- Colors: Use hex codes or strategy colors from `PEEP_METADATA`
