# Isometric Tower Defense

A real-time strategy tower defense game with isometric graphics, built from scratch in Python using Pygame. Features dynamic AI pathfinding, spatial grid optimization, and round-based survival gameplay.

## ✨ Features

### Core Gameplay
- **Isometric 3D Rendering**: Custom projection system for pseudo-3D perspective
- **Round-Based Survival**: Progressive difficulty with increasing enemy waves
- **Strategic Tower Placement**: Build defensive structures during gameplay
- **Soldier Deployment**: Spawn mobile units to defend your castle
- **Main Castle Defense**: Protect the central fortress with shield mechanics

### AI Systems
- **Zombie AI**: Autonomous pathfinding, target prioritization, collision avoidance
- **Soldier AI**: Automatic enemy tracking, ranged combat, formation behavior
- **Tower AI**: Automatic targeting within attack range
- **Target Prioritization**: Enemies choose between soldiers and towers dynamically

### Technical Features
- **Spatial Grid Optimization**: Efficient collision detection using spatial partitioning
- **Separation Steering**: Flocking behavior prevents entity clustering
- **Dynamic Audio System**: Background music with seamless victory/defeat track integration
- **Zoom & Pan Controls**: Free camera movement with scaling
- **Health & Shield Systems**: Multiple damage layers with visual feedback

## 📋 Requirements

- Python 3.7+
- Pygame
- NumPy (optional, for enhanced math operations)

## 🚀 Installation

```bash
# Clone or download the project
cd tower-defense-game

# Install dependencies
pip install pygame

# Run the game
python main.py
```

## 🎮 Controls

| Input | Action |
|-------|--------|
| **W / A / S / D** | Pan camera up / left / down / right |
| **Mouse Wheel Up/Down** | Zoom in / out |
| **B** | Toggle Build Mode (place towers) |
| **F** | Toggle Spawn Mode (deploy soldiers) |
| **1-6** | Select tower type (in Build Mode) |
| **Mouse Wheel** | Rotate tower (in Build Mode) |
| **Left Click** | Place tower/soldier or select tile |
| **Space** | Restart game (Game Over) / Next round (Victory) |

## 🏗️ Project Structure

```
.
├── main.py              # Entry point and game loop
├── config.py            # Game constants and settings
├── Renderer.py          # Isometric rendering engine
├── AIManager.py         # AI behavior for all entities
├── Audio.py             # Music and sound management
├── Enemy.py             # Zombie entity class
├── Soldier.py           # Soldier entity class
├── Tower.py             # Tower/Castle entity class
├── Spatial.py           # Spatial grid for collision optimization
└── res/                 # Resources (not included)
    ├── Zombie/         # Enemy sprite sheets
    ├── Soldier/        # Soldier animations
    ├── castle/         # Tower textures
    ├── music/          # Background tracks
    ├── sound/          # Sound effects
    └── ...
```

## 🎯 Game Mechanics

### Round System

**Setup Phase:**
- Each round grants you placement tokens
- Build towers before enemies spawn
- Deploy soldiers strategically

**Combat Phase:**
- Zombies spawn at map edges and attack
- Towers auto-target enemies in range
- Soldiers actively hunt zombies
- Survive all waves to advance

**Victory Conditions:**
- Eliminate all zombies → Next round
- Lose main castle → Game Over

### Entity Types

**Main Castle (Spawn: Center)**
- HP: 2000
- Shield: 1000 (regenerates each round)
- Damage: 120
- Special: Loses shield before HP damage

**Towers (Buildable)**
- HP: 500
- Damage: 100
- Range: Configurable radius
- Cost: 1 build token per tower

**Soldiers (Spawnable)**
- HP: 100
- Damage: 100
- Range: Long-range (40 units)
- Behavior: Auto-hunt nearest zombie

**Zombies (Enemy)**
- HP: 100
- Damage: 5
- Speed: 0.5 units/frame
- Behavior: Attack nearest tower or soldier

### AI Behavior

**Zombie AI (Priority System):**
```
1. Check for nearby soldiers
2. Check for nearby towers
3. Attack if in range (≤6 units)
4. Move toward target if farther
5. Collision avoidance with other zombies
6. Smooth rotation toward target
```

**Soldier AI:**
```
1. Find nearest zombie
2. Rotate to face target
3. Attack if in range (≤40 units)
4. Move closer if out of range
5. Avoid tower collisions
6. Separation from other soldiers
```

**Tower AI:**
```
1. Scan for enemies in radius
2. Target nearest enemy
3. Shoot if cooldown ready
4. Deal damage + play sound
```

## 🏛️ Architecture Overview

### Isometric Projection

The game uses a custom isometric projection system:

```python
# World coordinates (x, y, z) → Screen coordinates (screen_x, screen_y)
screen_x = MAP_CENTER_X + (x - z) * PROJECTION_WIDTH / 2
screen_y = MAP_CENTER_Y + (x + z) * PROJECTION_HEIGHT / 2 - y * PROJECTION_HEIGHT
```

**Key Features:**
- Depth sorting: Entities sorted by `(y, x + z)` for correct render order
- Dynamic zoom: All projections scale with `zoom_scale`
- Camera offset: `MAP_OFFSET_X/Y` for panning

### Spatial Grid Optimization

Instead of checking all entities against all others (O(n²)), the game uses spatial partitioning:

```python
class SpatialGrid:
    def __init__(self, cell_size=100):
        self.cells = {}  # Maps (cell_x, cell_z) → [entities]
    
    def nearby(self, x, z, radius):
        # Only check entities in neighboring cells
        # O(n) → O(k) where k << n
```

**Benefits:**
- Collision checks: From 2500 comparisons → ~25 comparisons
- Separation steering: Fast neighbor queries
- Scales with entity count

### Separation Steering

Zombies and soldiers use flocking behavior to avoid clustering:

```python
# For each nearby entity:
if distance < SEPARATION_RADIUS:
    repel_force = (SEPARATION_RADIUS - distance) / SEPARATION_RADIUS
    push_direction = (self.pos - other.pos).normalize()
    self.pos += push_direction * repel_force
```

**Effect:** Natural-looking formations without overlapping sprites

### Audio System

Custom `AudioManager` with dual-playlist support:

```python
# Background music loops randomly
audio.set_playlist(tracks, volume=0.5)
audio.play_next()  # Auto-advances on track end

# Temporary tracks (victory/defeat) pause background
audio.play_temp("victory.ogg", volume=0.6)
# Automatically resumes background when temp finishes
```

## 🎨 Rendering Pipeline

```
┌─────────────────────┐
│ Sort by Depth       │ ← (y, x+z) key
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Draw Ground Tiles   │ ← Grass grid
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Draw Entities       │ ← Towers, zombies, soldiers
│ (Depth-sorted)      │
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Draw Health Bars    │ ← HP/Shield overlays
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Draw Shield Dome    │ ← Main castle effect (if active)
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Draw Attack Lines   │ ← Visual feedback (sub-layer)
└──────────┬──────────┘
           │
           ▼
┌─────────────────────┐
│ Draw UI Overlay     │ ← Stats, mode indicators
└─────────────────────┘
```

## ⚙️ Configuration

### Game Balance (config.py)

```python
# Map settings
GRID_SIZE = 100              # World grid dimensions
GRASS_SCALE = 3              # Tile scale multiplier
SEPARATION_RADIUS = 2.0      # Entity spacing

# Spawn settings
NUM_ENEMIES = 50             # Starting zombie count
SETUP_DURATION = 5000        # Build phase time (ms)

# Projection settings
PROJECTION_WIDTH = 16        # Isometric tile width ratio
PROJECTION_HEIGHT = 8        # Isometric tile height ratio
```

### Entity Stats (in respective classes)

```python
# Enemy.py
self.speed = 0.5
self.attack_range = 6.0
self.attack_cooldown = 1500  # ms
self.rotation_speed = 5.0

# Tower.py
self.hp = 500  # (2000 for main castle)
self.attack_range = 65.0
self.attack_cooldown = 1000
self.damage = 100  # (120 for main)

# Soldier.py
self.attack_range = 40  # Long-range
self.attack_cooldown = 1500
self.damage = 100
```

## 🎯 Advanced Features

### Dynamic Target Switching

Zombies recalculate priorities every frame:

```python
def ZombieAI(enemy, all_towers, all_soldier):
    # Find nearest soldier
    target_soldier = min(soldiers, key=lambda s: distance(enemy, s))
    
    # Find nearest tower
    target_tower = min(towers, key=lambda t: distance(enemy, t))
    
    # Choose closer target
    if distance(enemy, soldier) < distance(enemy, tower):
        target = soldier
    else:
        target = tower
```

### Rotation Smoothing

Entities rotate gradually toward targets using offset-based control:

```python
def UpdateRotation(self, DESIRED_OFFSET=290, ROTATION_EPSILON=5):
    # Calculate angle difference
    offset_error = DESIRED_OFFSET - current_offset
    
    # Rotate in correct direction
    turn_direction = math.copysign(1, -offset_error)
    self.precise_angle += self.rotation_speed * turn_direction
    
    # Stop when close enough
    if abs(offset_error) <= ROTATION_EPSILON:
        return
```

**Why this works:**
- Visual direction (sprite facing): Snapped to discrete angles (0°, 22.5°, 45°, ...)
- Internal angle (`precise_angle`): Continuous rotation
- Offset maintains proper facing while entity rotates

### Collision Detection Layers

Three collision systems work together:

1. **Rect-based (fast, approximate):**
   ```python
   enemy_rect.colliderect(tower_rect)
   ```

2. **Circle-based (accurate, for physics):**
   ```python
   distance = hypot(dx, dz)
   return distance < (radius1 + radius2)
   ```

3. **Spatial grid (optimization):**
   ```python
   nearby_entities = grid.nearby(x, z, search_radius)
   # Only check these, not all entities
   ```

## 🐛 Known Issues & Limitations

- **Animation Loading**: Requires specific folder structure in `res/`
- **No Pathfinding**: Zombies move directly toward targets (no A* yet)
- **Fixed Camera Angle**: Isometric projection is locked (no rotation)
- **No Save System**: Progress resets on game close
- **Performance**: ~50+ entities may cause slowdown on older hardware

## 🔧 Extending the Game

### Adding New Tower Types

1. **Define in `main.py`:**
   ```python
   castle_definitions = {
       (x, y, z): texture_id,  # Add your position
   }
   ```

2. **Configure stats in `Tower.py`:**
   ```python
   def __init__(self, ...):
       if texture_id == YOUR_TYPE:
           self.attack_range = 100
           self.damage = 50
   ```

### Adding New Enemy Types

1. **Add sprite sheets to `res/YourEnemy/`**
2. **Copy `Enemy.py` → `YourEnemy.py`**
3. **Update `Zframelist` and `Zdirlist`**
4. **Modify `AIManager.py` to handle new behavior**

### Custom AI Behaviors

Edit `AIManager.py`:

```python
def CustomZombieAI(enemy, targets):
    # Your logic here
    if special_condition:
        enemy.SetState("SpecialAttack")
        # Custom behavior
```

## 🎓 Learning Takeaways

This project demonstrates:

- **Isometric projection math** (2D → pseudo-3D)
- **Spatial partitioning** for performance
- **State machine AI** (Idle, Run, Attack, Death)
- **Separation steering** (flocking behavior)
- **Depth sorting** for rendering order
- **Event-driven audio** (dynamic playlist system)
- **Component-based architecture** (separate Entity classes)

## 📊 Performance Tips

1. **Reduce enemy count** if FPS drops below 30
2. **Lower `GRID_SIZE`** for smaller maps (faster rendering)
3. **Increase `cell_size`** in `SpatialGrid` for fewer checks
4. **Disable debug rendering** (`DrawDebugInfo()` is commented out)

## 🤝 Contributing

This is an educational project. Feel free to:
- Add new enemy types
- Implement A* pathfinding
- Create new tower types
- Design custom levels
- Improve AI behaviors
