# Dead Rails - Defold Game Development Log

## Project Overview
A simple zombie chase game built in Defold 1.10.0 for learning purposes. Player moves around avoiding a zombie that chases them. Game over when caught, restart with R key.

## Current Working State

### Player Movement System
**IMPORTANT DISCOVERY**: We found that **single-direction movement is perfectly smooth**, but **diagonal movement introduces jerkiness**.

#### Working Smooth Movement (Current):
```lua
-- In player.script - SMOOTH VERSION
function update(self, dt)
    if self.moving then
        local pos = go.get_position()
        if self.current_animation == "up" then
            pos.y = pos.y + self.speed * dt
        elseif self.current_animation == "down" then
            pos.y = pos.y - self.speed * dt
        elseif self.current_animation == "left" then
            pos.x = pos.x - self.speed * dt
        elseif self.current_animation == "right" then
            pos.x = pos.x + self.speed * dt
        end
        go.set_position(pos)
    end
end
```

#### Problematic Diagonal Movement:
Any approach that allows multiple keys to affect movement simultaneously causes jerkiness:
- Multi-key tracking with `keys_pressed` table
- Velocity-based systems
- Vector math approaches
- Acceleration/deceleration systems

### Game Components

#### Player (`Player.go` + `player.script`)
- **Speed**: 120 pixels/second (2x faster than zombie)
- **Movement**: Simple directional movement (one direction at a time)
- **Animation**: Uses player.atlas with left/left-2, right/right-2, up, down images
- **Scale**: 30% of original size
- **Input**: Arrow keys + WASD
- **Smooth**: ✅ Works perfectly in single directions

#### Zombie (`Zombie.go` + `zombie.script`)
- **Speed**: 60 pixels/second
- **Movement**: Smooth vector-based chasing using distance/direction math
- **Animation**: Uses zombie.atlas with zombie.png, zombie-1.png, zombie-2.png
- **Scale**: 30% of original size
- **AI**: Continuously chases player, stops when game over
- **Smooth**: ✅ Perfectly smooth movement

#### Game Over System
- **Trigger**: When zombie gets within 30 pixels of player
- **Visual**: Dark overlay sprite on screen + console messages
- **Behavior**: Stops both player and zombie movement
- **Restart**: R key restarts game, respawns zombie randomly

#### Input System
- **Movement**: Arrow keys or WASD
- **Restart**: R key (only works during game over)
- **Bindings**: Defined in `input/game.input_binding`

### File Structure
```
/Users/fernandoipar/Dead Rails/
├── Player.go                    # Player game object
├── player.script               # Player movement logic (SMOOTH)
├── Player.sprite               # Player sprite component
├── player.atlas                # Player animations (left, right, up, down, idle)
├── Zombie.go                   # Zombie game object  
├── zombie.script               # Zombie AI chase logic
├── Zombie.sprite               # Zombie sprite component
├── zombie.atlas                # Zombie animations (walk, idle)
├── game_over_overlay.go        # Game over visual overlay
├── game_over_overlay.script    # Game over display logic
├── overlay.atlas               # Overlay graphics
├── overlay.sprite              # Overlay sprite
├── main/
│   ├── main.collection         # Main scene with player, zombie, game over
│   └── main.script             # Game state management
├── input/
│   └── game.input_binding      # Key bindings
├── game.project                # Defold project settings (60fps, vsync)
└── *.png                       # Image assets
```

### Technical Discoveries

#### Movement Smoothness Investigation
We tried multiple approaches to achieve smooth diagonal movement:

1. **Multi-key velocity system** - Jerky
2. **Acceleration/deceleration** - Jerky  
3. **Vector math (zombie approach)** - Jerky
4. **Target-based movement** - Jerky
5. **Simple continuous velocity** - Jerky

**Root Cause**: Unknown, but isolated to diagonal movement code. Single-direction movement using the original simple approach is perfectly smooth.

#### Speed Tuning
- **Original**: Player 850 speed - smooth but too fast
- **Problem**: Lowering speed to reasonable levels introduced jerkiness
- **Solution**: Found that jerkiness was from diagonal code, not speed
- **Current**: Player 120, Zombie 60 - good balance when using single-direction movement

#### Animation System
- **Player**: Uses atlas with separate left/right animations (left.png + left-2.png, etc.)
- **Zombie**: Uses atlas with walking cycle (zombie.png, zombie-1.png, zombie-2.png)
- **Both**: 30% scale looks good on screen
- **Frame rates**: Player animations at 8fps, zombie at 6fps

### Game Settings
- **Resolution**: 960x640
- **Frame rate**: 60fps with vsync
- **Update frequency**: 60hz

### Known Issues

#### Critical Issue: Diagonal Movement Jerkiness
- **Status**: Unresolved
- **Impact**: Player can only move in single directions smoothly
- **Workaround**: Currently using single-direction movement only
- **Next Steps**: Need to investigate why diagonal code causes jerkiness when single-direction is smooth

#### Minor Issues
- Game over text only shows in console, not on screen (overlay shows but no text)
- No boundaries - player can move off screen

### Next Session Goals
1. **Investigate diagonal movement jerkiness** - try to understand why single-direction is smooth but multi-key isn't
2. **Possible approaches to try**:
   - Physics-based movement system
   - Different input polling method
   - Frame-rate independent movement calculations
   - Defold-specific movement patterns

### Code Snippets for Reference

#### Smooth Single-Direction Movement (WORKING)
```lua
-- This works perfectly smooth
function update(self, dt)
    if self.moving then
        local pos = go.get_position()
        if self.current_animation == "up" then
            pos.y = pos.y + self.speed * dt
        elseif self.current_animation == "down" then
            pos.y = pos.y - self.speed * dt
        elseif self.current_animation == "left" then
            pos.x = pos.x - self.speed * dt
        elseif self.current_animation == "right" then
            pos.x = pos.x + self.speed * dt
        end
        go.set_position(pos)
    end
end
```

#### Zombie Smooth Movement (WORKING)
```lua
-- Zombie moves smoothly using vector math
local dx = player_pos.x - zombie_pos.x
local dy = player_pos.y - zombie_pos.y
local distance = math.sqrt(dx * dx + dy * dy)

local move_x = (dx / distance) * self.speed * dt
local move_y = (dy / distance) * self.speed * dt

zombie_pos.x = zombie_pos.x + move_x
zombie_pos.y = zombie_pos.y + move_y
go.set_position(zombie_pos)
```

## Status: SMOOTH SINGLE-DIRECTION MOVEMENT ACHIEVED ✅
## Next: INVESTIGATE DIAGONAL MOVEMENT JERKINESS ⚠️