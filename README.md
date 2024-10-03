# Doom Game

Here's a simplified explanation for your README.md that incorporates the code you shared. I'll format it step by step so it's easy to follow along with the code:

---

# **Raycasting Explained**

### **Overview**
Raycasting is a technique used to create a 3D-like effect by simulating how a player "sees" the world. It works by casting rays from the player's position to detect walls in the world map and rendering them on the screen based on their distance. 

This project uses raycasting to simulate a 3D environment, with rays shot from the player's perspective to detect the surroundings, including NPCs, and block vision when walls are in the way.

---

### **How Raycasting Works**

1. **Player Shoots Rays:**
   The player "shoots" rays from their position in a range based on their field of view. Each ray corresponds to one small vertical slice of the screen.

2. **Ray Moves and Checks for Walls:**
   Each ray travels step by step, using trigonometric functions (sine and cosine) to calculate how much it should move horizontally and vertically. It keeps moving until it either hits a wall or reaches the maximum depth.

3. **Determine Wall Hit:**
   - A ray can hit a **horizontal edge** of a wall (top or bottom) or a **vertical edge** (side).
   - The ray keeps track of which edge it hits first.
   - The code compares which is closer and picks that as the point where the wall is hit.

4. **Calculate Depth and Render:**
   The distance between the player and the wall is calculated. Closer walls are drawn taller on the screen, and further walls are drawn shorter. The ray casting corrects the "fishbowl effect" by adjusting the distance based on the angle of the ray.

---

### **Code Breakdown**

#### **Ray Casting for Rendering Walls**

```python
def ray_cast(self):
    self.ray_casting_result = []
    ox, oy = self.game.player.pos  # Player's position
    x_map, y_map = self.game.player.map_pos  # Player's map position

    texture_vert, texture_hor = 1, 1
    ray_angle = self.game.player.angle - HALF_FOV + 0.0001  # Start angle for the first ray

    for ray in range(NUM_RAYS):
        sin_a = math.sin(ray_angle)
        cos_a = math.cos(ray_angle)

        # Check Horizontal Lines (top/bottom of walls)
        y_hor, dy = (y_map + 1, 1) if sin_a > 0 else (y_map - 1e-6, -1)
        depth_hor = (y_hor - oy) / sin_a  # Distance to horizontal hit
        x_hor = ox + depth_hor * cos_a  # Calculate x position
        delta_depth = dy / sin_a  # Step depth horizontally
        dx = delta_depth * cos_a  # Step x horizontally

        for i in range(MAX_DEPTH):
            tile_hor = int(x_hor), int(y_hor)
            if tile_hor in self.game.map.world_map:  # Wall hit in horizontal direction
                texture_hor = self.game.map.world_map[tile_hor]
                break
            x_hor += dx
            y_hor += dy
            depth_hor += delta_depth

        # Check Vertical Lines (sides of walls)
        x_vert, dx = (x_map + 1, 1) if cos_a > 0 else (x_map - 1e-6, -1)
        depth_vert = (x_vert - ox) / cos_a  # Distance to vertical hit
        y_vert = oy + depth_vert * sin_a  # Calculate y position
        delta_depth = dx / cos_a  # Step depth vertically
        dy = delta_depth * sin_a  # Step y vertically

        for i in range(MAX_DEPTH):
            tile_vert = int(x_vert), int(y_vert)
            if tile_vert in self.game.map.world_map:  # Wall hit in vertical direction
                texture_vert = self.game.map.world_map[tile_vert]
                break
            x_vert += dx
            y_vert += dy
            depth_vert += delta_depth

        # Determine Closest Hit (horizontal or vertical)
        if depth_vert < depth_hor:
            depth, texture = depth_vert, texture_vert
            y_vert %= 1
            offset = y_vert if cos_a > 0 else (1 - y_vert)
        else:
            depth, texture = depth_hor, texture_hor
            x_hor %= 1
            offset = (1 - x_hor) if sin_a > 0 else x_hor

        # Remove Fishbowl Effect
        depth *= math.cos(self.game.player.angle - ray_angle)

        # Projection Height
        proj_height = SCREEN_DIST / (depth + 0.0001)

        # Ray Casting Result for Rendering
        self.ray_casting_result.append((depth, proj_height, texture, offset))

        ray_angle += DELTA_ANGLE  # Move to next ray angle
```

---

#### **Ray Casting for Aiming Between Player and NPC**

This version of the raycasting is used to check if an NPC is within the player's line of sight (without any walls blocking the view). The raycasting technique is similar but adapted to compare distances between the player, NPC, and walls.

```python
def ray_cast_player_npc(self):
    if self.game.player.map_pos == self.map_pos:
        return True  # Player is at the same position as NPC
    
    wall_dist_v, wall_dist_h = 0, 0
    player_dist_v, player_dist_h = 0, 0
    ox, oy = self.game.player.pos  # Player's position
    x_map, y_map = self.game.player.map_pos  # Player's map position

    ray_angle = self.theta  # Angle to the NPC
    sin_a = math.sin(ray_angle)
    cos_a = math.cos(ray_angle)

    # Horizontals
    y_hor, dy = (y_map + 1, 1) if sin_a > 0 else (y_map - 1e-6, -1)
    depth_hor = (y_hor - oy) / sin_a
    x_hor = ox + depth_hor * cos_a
    delta_depth = dy / sin_a
    dx = delta_depth * cos_a

    for i in range(MAX_DEPTH):
        tile_hor = int(x_hor), int(y_hor)
        if tile_hor == self.map_pos:  # Hit NPC
            player_dist_h = depth_hor
            break
        if tile_hor in self.game.map.world_map:  # Hit Wall
            wall_dist_h = depth_hor
            break
        x_hor += dx
        y_hor += dy
        depth_hor += delta_depth

    # Verticals
    x_vert, dx = (x_map + 1, 1) if cos_a > 0 else (x_map - 1e-6, -1)
    depth_vert = (x_vert - ox) / cos_a
    y_vert = oy + depth_vert * sin_a
    delta_depth = dx / cos_a
    dy = delta_depth * sin_a

    for i in range(MAX_DEPTH):
        tile_vert = int(x_vert), int(y_vert)
        if tile_vert == self.map_pos:  # Hit NPC
            player_dist_v = depth_vert
            break
        if tile_vert in self.game.map.world_map:  # Hit Wall
            wall_dist_v = depth_vert
            break
        x_vert += dx
        y_vert += dy
        depth_vert += delta_depth

    # Compare Player and Wall Distances
    player_dist = max(player_dist_v, player_dist_h)
    wall_dist = max(wall_dist_v, wall_dist_h)

    # If NPC is closer than any wall or no walls are in the way, return True
    if 0 < player_dist < wall_dist or not wall_dist:
        return True
    return False
```

---

### **Key Calculations**
- **Sine and Cosine:** Used to determine how much the ray moves horizontally and vertically based on its angle.
- **Depth Calculation:** The distance from the player to the wall or NPC is calculated using basic trigonometry.
- **Fishbowl Effect Fix:** The distance is adjusted to prevent walls from looking distorted when viewed at an angle.

---

By following these steps, the game determines what the player can see in the environment, checks for obstacles like walls, and calculates whether the NPC is within the player's sight.

This should give you a clear understanding of the raycasting mechanics used in the game, both for rendering walls and for aiming at NPCs.

--- 

Let me know if you'd like any additional explanations or adjustments!





