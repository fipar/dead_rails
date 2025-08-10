# Dead Rails - Defold Game Development Log

## Project Overview
A simple zombie chase game built in Defold 1.10.0 for learning purposes. Player moves around avoiding a zombie that chases them. Game over when caught, restart with R key.

## Current Working State

### Player Movement System
**BREAKTHROUGH**: We successfully implemented smooth diagonal movement using vector math with speed normalization!

#### Working Smooth Diagonal Movement (Current):
```lua
-- In player.script - SMOOTH DIAGONAL VERSION
function update(self, dt)
    if self.moving and (self.direction_x ~= 0 or self.direction_y ~= 0) then
        local pos = go.get_position()
        
        -- Calculate distance for normalization (like zombie does)
        local distance = math.sqrt(self.direction_x * self.direction_x + self.direction_y * self.direction_y)
        
        -- Move using normalized vector (like zombie approach)
        if distance > 0 then
            local move_x = (self.direction_x / distance) * self.speed * dt
            local move_y = (self.direction_y / distance) * self.speed * dt
            
            pos.x = pos.x + move_x
            pos.y = pos.y + move_y
            
            go.set_position(pos)
        end
    end
end
```

#### Diagonal Movement Solution:
**Key insight**: The zombie's vector math approach works perfectly for player movement too! The solution uses:
- Multi-key tracking with `keys_pressed` table ✅ 
- Vector math with speed normalization ✅
- Direction vectors (-1, 0, 1) for each axis ✅
- Smart animation selection based on primary movement direction ✅

### Game Components

#### Player (`Player.go` + `player.script`)
- **Speed**: 120 pixels/second (2x faster than zombie)
- **Movement**: Smooth vector-based movement with diagonal support
- **Animation**: Uses player.atlas with left/left-2, right/right-2, up, down images
- **Scale**: 30% of original size
- **Input**: Arrow keys + WASD (supports key combinations)
- **Smooth**: ✅ Works perfectly in all directions including diagonals

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

#### Movement Smoothness Solution ✅
**SOLVED**: Diagonal movement achieved using vector math with speed normalization!

**Working approach**: Vector math with proper implementation:
1. **Multi-key tracking** - Track all pressed keys in `keys_pressed` table
2. **Direction vectors** - Convert keys to -1/0/1 direction values  
3. **Speed normalization** - Use `math.sqrt()` and division like zombie
4. **Smart animations** - Choose animation based on primary movement direction

**Previous failed attempts** were likely due to improper implementation details, not the fundamental approach.

#### Speed Tuning
- **Original**: Player 850 speed - smooth but too fast
- **Previous problem**: Lowering speed with old diagonal code introduced jerkiness
- **Current**: Player 120, Zombie 60 - perfect balance with new vector math movement

#### Animation System
- **Player**: Uses atlas with separate left/right animations (left.png + left-2.png, etc.)
- **Zombie**: Uses atlas with walking cycle (zombie.png, zombie-1.png, zombie-2.png)
- **Both**: 30% scale looks good on screen
- **Frame rates**: Player animations at 8fps, zombie at 6fps

#### Splash Screen System
- **Implementation**: Collection factory approach
- **Image**: Custom splash.png (960x640) with "Dead Rails" title and red "Jugar" button
- **Button detection**: Click coordinates (190-365 x, 380-460 y) 
- **Transition**: Hides splash, creates main game via factory, transfers input focus
- **Standalone**: No manual file changes or engine restart needed

### Game Settings
- **Resolution**: 960x640
- **Frame rate**: 60fps with vsync
- **Update frequency**: 60hz

### Known Issues

#### Minor Issues
- Game over text only shows in console, not on screen (overlay shows but no text)
- No boundaries - player can move off screen

### Next Session Goals
1. **Add game over text display on screen** - Currently only shows in console
2. **Add screen boundaries** - Prevent player from moving off screen  
3. **Add sound effects** - Movement sounds, game over sounds
4. **Add scoring system** - Track survival time
5. **Add multiple zombie difficulty** - More zombies as time progresses

### Code Snippets for Reference

#### Smooth Diagonal Movement Input System (WORKING)
```lua
-- Input handling with multi-key support
function on_input(self, action_id, action)    
    if action_id == hash("up") then
        if action.pressed then
            self.keys_pressed["up"] = true
        elseif action.released then
            self.keys_pressed["up"] = false
        end
    -- ... similar for down, left, right
    end
    
    update_movement(self)
end

function update_movement(self)
    -- Calculate direction vector based on pressed keys
    self.direction_x = 0
    self.direction_y = 0
    
    if self.keys_pressed["up"] then self.direction_y = 1 end
    if self.keys_pressed["down"] then self.direction_y = -1 end
    if self.keys_pressed["left"] then self.direction_x = -1 end
    if self.keys_pressed["right"] then self.direction_x = 1 end
    
    -- Set movement and animation based on direction
    if self.direction_x ~= 0 or self.direction_y ~= 0 then
        self.moving = true
        -- Smart animation selection based on primary direction
        if math.abs(self.direction_x) > math.abs(self.direction_y) then
            play_animation(self, self.direction_x > 0 and "right" or "left")
        else
            play_animation(self, self.direction_y > 0 and "up" or "down")
        end
    else
        stop_movement(self)
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

## Status: COMPLETE SPLASH SCREEN + SMOOTH DIAGONAL MOVEMENT ACHIEVED ✅
## Next: ADD GAMEPLAY FEATURES 🎮