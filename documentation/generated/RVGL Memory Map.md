# RVGL Memory Map & Data Structures
**Last Updated:** February 2026 — Batch 10 / Full 222k-line rvgl.exe.c pass  
**Architecture:** x64 Windows

---

## 📋 Table of Contents
1. [Memory Regions](#memory-regions)
2. [Data Structures](#data-structures)
3. [Global Variables (Sorted by Address)](#global-variables-sorted-by-address)
4. [String Tables](#string-tables)
5. [Function Pointer Tables](#function-pointer-tables)

---

## Memory Regions

### RVGL.exe Memory Layout
```
Base Address: 0x00400000 (typical)

.text (Code):        0x00401000 - 0x006xxxxx
.data (Initialized): 0x00661000 - 0x00xxxxxx
.rdata (Read-only):  0x00xxxxxx - 0x00xxxxxx
.bss (Uninitialized): Large data arrays
```

### libGLESv2.dll Memory Layout
```
Base Address: 0x68000000 (typical for DLLs)

.text (Code):        0x68001000 - 0x681xxxxx
.data (Initialized): 0x6814c000 - 0x681xxxxx
.rdata (Read-only):  0x68200000 - 0x682xxxxx
```

---

## Data Structures

### CarInfo Structure (272 bytes / 0x110)
```c
struct CarInfo {
    // String Fields
    char internalName[20];           // +0x00 (0x14 bytes)
    char displayName[19];            // +0x14 (0x13 bytes)
    char padding1[29];               // +0x27 to +0x54
    char tpageFilename[64];          // +0x54 (TPAGE texture file)
    char tpageNull;                  // +0x93 (null terminator)
    char tcarboxFilename[64];        // +0x94 (TCARBOX icon file)
    char tcarboxNull;                // +0xD3 (null terminator)
    
    // Unknown Fields
    byte unknown1[16];               // +0xD4 to +0xE4
    
    // Boolean Flags (1 byte each)
    byte bestTimeEnabled;            // +0xE4 (BESTTIME: 0/1)
    byte selectableByPlayer;         // +0xE5 (SELECTABLE: 0/1)
    byte selectableByCPU;            // +0xE6 (CPUSELECTABLE: 0/1)
    byte statisticsEnabled;          // +0xE7 (STATISTICS: 0/1)
    
    // Integer Stats (4 bytes each)
    int32 carClass;                  // +0xE8 (CLASS: 0-4)
    int32 starRating;                // +0xEC (RATING: stars)
    int32 obtainCondition;           // +0xF0 (OBTAIN: unlock method)
    int32 topSpeedStat;              // +0xF4 (TOPEND: top speed)
    int32 accelerationStat;          // +0xF8 (ACCELERATION: accel)
    int32 weightValue;               // +0xFC (WEIGHT: weight in kg?)
    int32 transmissionType;          // +0x100 (TRANS: 0=auto?)
    
    // Unknown/Padding
    byte unknown2[4];                // +0x104 to +0x108
    
    // Status Flags
    uint16 statusFlags;              // +0x108 (various flags)
    byte isInvalid;                  // +0x109 (1 = invalid/missing car)
    
    // End padding
    byte padding2[6];                // +0x10A to +0x110
};

// Array Information:
// Base Address: 0x006fab50
// Element Count: 49 (0x31)
// Total Size: 49 * 272 = 13,328 bytes
// Element Access: CarInfo* car = &DAT_006fab50[index * 0x110];
```

### TrackInfo Structure (120 bytes / 0x78)
```c
struct TrackInfo {
    // Unknown header
    byte unknown1[16];               // +0x00 to +0x10
    
    // Track name
    char displayName[64];            // +0x10 (NAME: track display name)
    
    // Unknown data
    byte unknown2[20];               // +0x50 to +0x64
    
    // Track properties (4 bytes each)
    int32 difficultyRating;          // +0x64 (DIFFICULTY: 1-5?)
    int32 gameType;                  // +0x68 (GAMETYPE: 2=standard, 3=?)
    int32 challengeTime;             // +0x6C (CHALLENGE: time in milliseconds)
    int32 challengeReverseTime;      // +0x70 (CHALLENGEREV: reverse time)
    
    // Unknown footer
    byte unknown3[4];                // +0x74 to +0x78
};

// Array Information:
// Base Address: s_nhood1_0065fe20
// Element Size: 120 (0x78)
// Element Access: TrackInfo* track = &s_nhood1_0065fe20[index * 0x78];
```

### GLContextHandle (32 bytes / 0x20)
```c
struct GLContextHandle {
    void* vtable;                    // +0x00 (points to 0x68214410)
    void* reserved;                  // +0x08 (always NULL)
    uint64 versionFlags;             // +0x10 (0x30a000003000)
    RealGLContext* realContext;      // +0x18 (pointer to actual context)
};

// Stored in Thread Local Storage (TLS)
// TLS Index stored at: 0x6814c020
// Retrieved via: TlsGetValue(TLS_INDEX) → returns GLContextHandle*
```

### RealGLContext (14,833+ bytes!)
```c
struct RealGLContext {
    // Header (first ~136 bytes)
    byte unknown_header[136];        // +0x0000 to +0x0088
    
    // Debug & Metadata
    char debugFlag;                  // +0x0088 (0=normal, 1=debug mode)
    byte padding1[7];                // +0x0089 to +0x0090
    void* currentFunctionName;       // +0x0090 (points to function name string)
    void* drawCallParameters;        // +0x0098 (draw call params)
    
    // D3D11 Backend (estimated locations)
    // ID3D11Device* d3dDevice;      // Unknown offset
    // ID3D11DeviceContext* d3dContext; // Unknown offset
    // IDXGISwapChain* swapChain;    // Unknown offset
    
    // State cache (textures, buffers, shaders)
    // ... thousands of bytes of state ...
    
    // Context lost flag (near end)
    byte contextLostFlag;            // +0x39f1 (0=OK, 1=GPU lost)
    
    // ... more unknown data to reach 14,833+ bytes total
};

// Allocation: malloc() or operator new
// Initialization: Complex multi-stage process
// Lifetime: Per-thread, destroyed when thread exits
```

### TrackWaypointNode Struct (AI racing line graph)
> Linked graph of track waypoints used by AI navigation. Each node represents one segment.
> Stored in a flat array; linked via pointer fields.
```c
struct TrackWaypointNode {
    byte    type;               // +0x00  node type (same codes as AINode)
    byte    type2;              // +0x01
    /* +0x02..+0x03 pad */
    float   left_lateral;       // +0x04  left lane boundary bias
    float   right_lateral;      // +0x08  right lane boundary bias
    /* +0x0c..+0x0f unknown */
    void*   back_link1;         // +0x10  previous node (link 1)
    void*   back_link2;         // +0x18  previous node (link 2)
    void*   fwd_link1;          // +0x20  next node (link 1)  — confirmed: CompareCarPassing uses +0x28[iVar4*8]
    void*   fwd_link2;          // +0x28  next node (link 2)
    /* +0x30..+0x37 unknown */
    float   left_edge[3];       // +0x38..+0x40  left track edge XYZ
    float   right_edge[3];      // +0x48..+0x50  right track edge XYZ
    int     zone_index;         // +0x58  spatial zone index (0x7fffffff = invalid/unset)
    /* +0x5c..+0x5f unknown */
    float   center_x;           // +0x60  center of segment X
    float   center_y;           // +0x64  center of segment Y
    float   center_z;           // +0x68  center of segment Z
    /* +0x6c..+0xa7 unknown */
    float   segment_length;     // +0xa8  arc length of this segment
    byte    obstacle_flags;     // +0xb6  bitfield for obstacle directions
    /* ... rest to +0x107 unknown but stride = 0x108 */
};
```

### TrackZoneLookupTable (spatial grid for track zone lookup)
> `DAT_006e3b00` — stride 0x10 per zone entry.
> Used by FUN_0040d840 (GetCurrentTrackPolygon) to spatial-query car position.
```c
struct ZoneEntry {       // 0x10 bytes
    int   waypoint_count;   // +0x00  number of waypoints in this zone
    int   unused;           // +0x04
    void* waypoint_list;    // +0x08  pointer to linked waypoint list (stride 0x78)
};
// Zone waypoint entries (stride 0x78, linked via +0x88):
//   +0x28  = linked child node ptr
//   +0x38..+0x50 = 4 corner XZ coords (quad polygon for AABB)
//   +0x60..+0x64 = X/Z min/max for pre-filter AABB
//   +0x68 = entry count per sub-cell
//   +0x70 = sub-cell list ptr
//   +0x88 = next entry in zone (linked list)
//   +0xb8 = AABB car_x min
//   +0xbc = AABB car_z min
//   +0xc0 = AABB car_x max  
//   +0xc4 = AABB car_z max
```

### Additional CarEntity Fields Discovered (Path Navigation)
> These extend the CarEntity struct defined above.
```c
// Navigation path state (extends CarEntity+0x67xx):
    int    current_zone_section;    // +0x67d4  current track zone/section index
    void*  primary_wp_path;         // +0x67e0  primary waypoint path ptr (from current pos)
    void*  secondary_wp_path;       // +0x67e8  secondary/alternate route ptr
    float  left_clearance_bias;     // +0x689c  left side clearance metric
    float  right_clearance_bias;    // +0x68a0  right side clearance metric
    short  current_border_count;    // +0x68b6  path direction result (from CompareCarPassing)  
    short  track_border_count;      // +0x68b8  border segment count (max 0xf = 15)
    void*  border_ptrs[15];         // +0x6810..+0x6880  up to 15 track border waypoint ptrs
```

### CarEntity Runtime Struct (size: ~0x6B00+ bytes)
> The primary runtime car object. The linked list head is `DAT_0acee9e8`.
> Each car's `+0x10` field is the next-car pointer.
```c
struct CarEntity {
    // --- Identity & Control ---
    int    car_array_index;             // +0x00  index into DAT_006fab50 CarInfo array
    int    car_type;                    // +0x04  1=player, 9=spectator/disabled, 4=special
    /* 0x08..0x10 unknown */
    void*  next_car;                    // +0x10  linked list next pointer
    /* 0x14..0x30 unknown */
    int8   ai_steering;                 // +0x34  -127=full left, +127=full right
    int8   ai_throttle;                 // +0x36  -127=full brake, +127=full throttle
    uint16 control_flags;               // +0x38  bit4=draft, 0x4000=drift_right, 0x1000=drift_left
    uint16 flags2;                      // +0x3e
    int    car_slot_index;              // +0x48  index into DAT_006fab50
    float  handling_param;              // +0xa4
    float  track_length_param;          // +0xb8
    void*  physics_body_ptr;            // +0xC8  see PhysicsBody struct
    WheelData wheel[4];                 // +0x4b8  stride 0x64 per wheel

    // --- Race Progress ---
    int    lap_count_alt;               // +0x1008
    int    physics_contacts_count;      // +0x100c  if < 3 → stuck detection
    int    lap_count;                   // +0x1010
    byte   prevent_reverse_flag;        // +0x107c
    byte   car_type_byte;               // +0x107d
    byte   stuck_flag;                  // +0x107f
    int    lap_count_int;               // +0xf40
    float  race_time_accumulator;       // +0xf90  incremented by delta_time each frame
    byte   some_state_flag;             // +0xfb0
    int    cooldown_timer;              // +0x104c  decremented per-frame (FUN_004c84a0)
    int    countdown_timer;             // +0x1050

    // --- AI Rubber Band State ---
    float  rubber_band_window;          // +0x6900  checkout window countdown
    longlong track_border_ptrs[2];      // +0x6808..+0x6818
    float  clearance_left_right[2];     // +0x68a4..+0x68a8
    float  path_width_bounds[2];        // +0x68ac..+0x68b0
    uint16 gap_to_car_ahead;            // +0x68b4
    short  track_border_count;          // +0x68b8
    int    rubber_band_acc[2];          // +0x68bc..+0x68c0
    byte   car_just_ahead_flag;         // +0x68c4

    // --- AI State Machine ---
    int    ai_state_current;            // +0x68c8  (see AI State codes below)
    int    ai_state_previous;           // +0x68cc
    float  ai_state_timer;              // +0x68d0
    int    position_score;              // +0x68d8
    int    position_score2;             // +0x68e4
    float  stuck_still_timer;           // +0x68e8
    float  stuck_no_target_timer;       // +0x68ec
    float  stuck_slow_timer;            // +0x68f0
    float  last_pos_checkpoint[3];      // +0x68f4..+0x68fc
    
    // --- AI Navigation ---
    float  steering_angle;              // +0x6920  1.0=full right, -1.0=full left
    float  abs_steering_magnitude;      // +0x6924
    int    lane_position;               // +0x6928  -1=left, 0=center, 1=right
    float  turning_rate_yaw;            // +0x692c
    float  lateral_path_deviation;      // +0x6934
    float  path_normal[3];              // +0x6938..+0x6940
    float  heading_direction[3];        // +0x6944..+0x694c
    float  target_direction[3];         // +0x6950..+0x6958
    float  target_position[3];          // +0x695c..+0x6964
    float  lookahead_distance;          // +0x6968  default 600.0f
    float  max_steer_limit;             // +0x696c  default 50.0f
    float  angle_threshold;             // +0x6970  default 100.0f
    float  height_target;               // +0x6974  default 150.0f
    float  ai_param5;                   // +0x6978  default 50.0f
    float  ai_param6;                   // +0x697c  default 300.0f (track side room)
    int    race_zone;                   // +0x6980  0xffffffff = unset
    float  wait_time_acc;               // +0x6984
    float  track_length;                // +0x6988  = car_params+0xb8
    float  speed_cm_s;                  // +0x698c  current speed in cm/s
    float  track_side_room;             // +0x6990
    float  dist_to_route;               // +0x6994
    void*  optimal_target_entity;       // +0x69a8
    void*  backup_target_entity;        // +0x69b0
    float  dist_to_optimal_target;      // +0x69b8
    float  dist_to_backup_target;       // +0x69bc
    int    race_position;               // +0x69c0  0-based
    int    race_position_score;         // +0x69c4
    int    rubber_band_direction;       // +0x69cc  1=ahead of player, -1=behind
    byte   reverse_state;               // +0x69d0
    byte   prev_reverse_state;          // +0x69d1
    float  reverse_timer;               // +0x69d4
    float  engine_power_current;        // +0x69d8
    float  engine_power_target;         // +0x69dc  default 0.5f
    byte   obstacle_front;              // +0x69e0
    byte   obstacle_rear;               // +0x69e1
    byte   obstacle_left;               // +0x69e2
    byte   obstacle_right;              // +0x69e3
    float  correction_factor;           // +0x69e4
    float  slip_accumulator[2];         // +0x69e8..+0x69ec
    byte   braking_flag;                // +0x69f0
    int    ai_route_zone;               // +0x6a20
    int    some_counter;                // +0x6a30
    void*  overflow_buffer_ptr;         // +0x6a38
    int    ai_freeze_flag;              // +0x6a48
    float  race_time_in_tag_mode;       // +0x6a54
};

// AI State Machine constants (car+0x68c8):
//   0  = Normal racing (optimal path following)
//   1  = Startup / reset (speed 0, resets controls)
//   2  = Normal racing → transitions to 3 if car nearby
//   3  = Avoidance (FUN_004071c0)
//   4  = Rubber-band behavior (chase/slow)
//   6  = Braking
//   7  = Partial avoidance (25% steering)
//   8/9  = Left track alternative
//   10/11 = Left turn recovery
//   12/13 = Right turn recovery

// Speed thresholds (cm/s → km/h):
//   268.3363  ≈  9.65 km/h  (low/stuck threshold)
//   447.2272  ≈ 16.1  km/h
//   894.4543  ≈ 32.2  km/h
//   1341.682  ≈ 48.3  km/h
//   1788.909  ≈ 64.4  km/h
//   2236.136  ≈ 80.5  km/h
//   2683.363  ≈ 96.5  km/h
```

### PhysicsBody Struct (at CarEntity+0xC8)
```c
struct PhysicsBody {
    /* +0x00..+0x10 unknown */
    float  x_pos;                   // +0x14
    float  y_pos;                   // +0x18
    float  z_pos;                   // +0x1c
    float  linear_acc_or_vel[3];    // +0x20..+0x28
    /* +0x2c..+0x6b unknown */
    float  velocity_x;              // +0x6c
    /* +0x70..+0x73 unknown */
    float  velocity_z;              // +0x74
    /* +0x78..+0x13b unknown */
    float  rot_matrix[5];           // +0x13c..+0x150  (partial rotation matrix)
    float  rot_m00;                 // +0x140  CONFIRMED
    float  rot_m10;                 // +0x144
    /* ... rotation data continues */
    int    on_ground_flag;          // +0x210
    int    wheel_contact_count;     // +0x214
    float  some_threshold;          // +0x21c   (< 0.1f check)
    float  collision_normals[7];    // +0x248..+0x264
    float  front_track_angle;       // +0x260
    float  side_track_angle;        // +0x264
};
```

### AINode Graph Struct (stride 0x108 bytes, max 0x1000 nodes)
> Buffer at `DAT_006f94a0` (0x108000 bytes), count at `DAT_006f94aa` (short).
```c
struct AINode {              // 0x108 bytes
    byte    type;           // +0x00  node type code
    byte    type2;          // +0x01
    /* +0x02..+0x03 pad */
    float   track_t;        // +0x04  position along track (0.0..1.0)
    uint32  extra[3];       // +0x08..+0x14
    void*   back_link1;     // +0x18  backward graph link
    void*   back_link2;     // +0x20
    void*   fwd_link1;      // +0x28  forward graph link 1
    void*   fwd_link2;      // +0x30  forward graph link 2
    float   left_edge[3];   // +0x38..+0x40  left track edge XYZ
    uint32  flags1;         // +0x44
    float   right_edge[3];  // +0x48..+0x50  right track edge XYZ
    uint32  flags2;         // +0x54
    /* +0x58..+0xb5 unknown */
    byte    obstacle_flags; // +0xb6
    /* +0xb7..+0x107 padding/unknown */
};

// Node type codes:
//   0x01 = normal segment
//   0x02 = shortcut
//   0x03 = hazard
//   0x05 = hazard alt
//   0x09 = turbo boost
//   0x0a = lead car target
//   0x0b = follow car target
//   0x0c = dead-end
//   0x0f = shortcut variant
```

### GameObjectEntity Struct (in-world objects: pickups, projectiles, etc.)
> Linked list head at `DAT_0ace4800`. Next pointer at `+0x08`. Updated in `FUN_004c84a0`.
```c
struct GameObjectEntity {
    /* +0x00..+0x07 unknown */
    void*  next_object;     // +0x08  linked list next
    /* +0x10..+0x23 unknown */
    float  x_pos;           // +0x24
    float  y_pos;           // +0x28
    float  z_pos;           // +0x2c
    uint32 flags;           // +0x38  bit6 = respawn trigger, bit1 = skip render
    /* +0x40..+0x277 unknown */
    int    type;            // +0x278  -1 = projectile/bullet
    /* +0x27c..+0x29f unknown */
    void*  owning_car;      // +0x2a0  pointer to owner CarEntity
    /* +0x2a8..+0x34f unknown */
    void (*update_fn)(void*); // +0x350  per-frame update function pointer
};
```

### NetworkCarEntry Struct (stride 0x1e0 bytes)
> Base array at `DAT_0adccb08`. Count at `DAT_0adccb00`.
```c
struct NetworkCarEntry {    // 0x1e0 bytes
    /* +0x00..+0x37 unknown */
    byte   flags;           // +0x38  bit1=skip_render
    /* +0x3c..+0x57 unknown */
    int    tri_count_1;     // +0x58
    int    tri_count_2;     // +0x5c
    int    tri_count_3;     // +0x60
    int    tri_count_4;     // +0x64
    int    face_count;      // +0x68  total mesh faces
    int    vert_count1;     // +0x70
    int    vert_count2;     // +0x74
    int    vert_count3;     // +0x78
    int    vert_count4;     // +0x7c
    int    extra_obj_count; // +0x80
    int    mesh_count;      // +0xb8
    void*  face_data_ptr;   // +0x98
    void*  vert_buf_ptr;    // +0xc0   or at +0xc8
    void*  index_buf_ptr;   // +0xc8
    void** extra_obj_arr;   // +0xa8
    /* ... more fields to 0x1e0 */
};
```

### CarEntity Linked List & Pool (Batch 4)
> Active list: head=`DAT_0acee9e8`, tail=`DAT_0acee9e0`. Free pool head: `DAT_0adb69b0`.
> Total active count: `_DAT_0acee7e4`.
```c
// CarEntity linked-list header fields (offsets within CarEntity):
// +0x08  = prev car ptr  (prev in active list; 0 = tail)
// +0x10  = next car ptr  (next in active list; 0 = head)
// +0x04  = car_type (int): 1=player, 2=AI, 3=ghost, 4=reset_car, 9=spectator
// +0x18  = subtype/behavior (int, from param_2 of CreateCarEntity)
// +0x40  = physics_body ptr (PhysicsBody*)

// Car entity initialization (from FUN_004f03a0):
// car+0x20..+0x2f = zeroed (16 bytes)
// car+0x690c      = 0 (control state byte)
// car+0x6904      = 0 (byte)
// car+0x6908      = 0 (int)
// car+0xf34       = 1 (int flag)
// car+0x6a80      = 0 (ushort)
// car+0x6a82      = 0 (byte)
// car+0x6a84      = 0 (int = network_player_id, init 0)
// car+0x6a44      = 0 (byte = invincible flag)
// car+0x6a38      = 0 (uint64)
// car+0x6a40      = 0 (int)
// car+0x6a48      = 0x00000000 (int = finish_time_ms, 0=not finished)
// car+0x6a4c      = 0xffffffff (int = final_race_position, 0xffffffff=not finished)
// car+0x6a88      = 0x7fffffff00000000
// car+0x6a20      = 0xffffffff (int)
// car+0x6a28..6a31 = 0
// car+0x6a64      = 0
// car+0x6a5c      = 0
// car+0x6a90      = 0x7fffffff
// car+0x6a98      = 0xffffffffffffffff (-1)
// car+0x6aa0      = 0 (byte)
// car+0x6a54      = (float)DAT_006e34dc
```

### RouteSection Struct (stride 0x50 = 80 bytes)
> Array at `DAT_006e3af0`. Default section: `DAT_006e3ae4`. Secondary: `DAT_006e3aec`.
> Count gates on `DAT_006e3ae8 != 0` and `DAT_006e3b14 != 0`.
```c
struct RouteSection {        // 0x50 bytes per entry
    float  x;                // +0x00  center position X
    float  y;                // +0x04  center position Y
    float  z;                // +0x08  center position Z
    float  route_val;        // +0x0c  cumulative route distance at this node
    float* fwd_neighbor1;    // +0x10  pointer to fwd neighbor RouteSection (or NULL)
    float* fwd_neighbor2;    // +0x18
    float* fwd_neighbor3;    // +0x20
    float* fwd_neighbor4;    // +0x28
    float* bwd_neighbor1;    // +0x30  backward neighbor pointer (or NULL)
    float* bwd_neighbor2;    // +0x38
    float* bwd_neighbor3;    // +0x40
    float* bwd_neighbor4;    // +0x48
};
// route_section_index = (float* - DAT_006e3af0) >> 4 (each 4-float group = 1 entry)
// car+0x6910 = route_section_index (int): which RouteSection the car is currently on
// car+0x6914 = track_t (float): car's distance progress along route [0, DAT_006e3ae0)
// car+0x6918 = norm_track_t (float): track_t / DAT_006e3ae0 (0.0..1.0)
// car+0x690c = facing_route (bool): 0.6 < dot(car_forward, route_dir)
// car+0x690f = spectator_mode (byte): non-zero → skip route close-to-XYZ search
```

### CarEntity Route / AI Fields (Batch 4 additions)
```c
// Confirmed fields from FUN_0040e150 (UpdateCarRoutePosition) and FUN_0044cb60:
// car+0x690c  = facing_route_direction (bool/byte, 1 = facing forward on route)
// car+0x690d  = is_facing_forward (byte, set to 1 on spawn by FUN_0044cb60)
// car+0x690e  = ushort flags (set to 0x0101 on spawn)
// car+0x690f  = byte, 0=normal search, 1=spectator override (skips position test)
// car+0x6910  = route_section_index (int, current RouteSection; init = DAT_006e3ae4)
// car+0x6914  = track_t (float, distance progress; 0 on spawn)
// car+0x6918  = norm_track_t_last (float, from last OBB hit = fVar10 value)
// car+0x6968  = lookahead_distance (float, written by GetTargetPosition)
// car+0xf10..+0xf34 = transform matrix data (10 floats, used by ghost recorder)
// car+0xf44   = ghost_frame_count? (int, set 0 on player car init)
// car+0xf90   = ghost_timer? (set = DAT_0ae48df0 on player car init)
// car+0x6a70..+0x6a9f = 16-float config array (copied from PTR_DAT_0065fca8+0x2c)
// car+0x53c   = current_speed (float)
// car+0x80    = ptr to physics sub-struct (different from car+0xc8)
//              physics_sub+0x14 = position X or quat_x
//              physics_sub+0x18 = position Y or quat_y
//              physics_sub+0x1c = position Z or quat_z
//              physics_sub+0x44..+0x50 = quaternion WXYZ (current orientation)
// car+0xc8    = physics_body+0x10 (physics data ptr)
//              +0x14 = pos X (float), +0x18 = pos Y, +0x1c = pos Z
//              +0x6c = car_forward_X, +0x70 = car_forward_Y, +0x74 = car_forward_Z
```

### PhysicsBody Struct (partial, accessed via car+0x40)
```c
// car+0x40 = physics_body ptr (lVar1 created by FUN_004ec110)
// physics_body+0x10   = data sub-block (ptr stored at car+0xc8)
// physics_body+0x24   = position X (= physics_body+0x10 + 0x14)
// physics_body+0x28   = position Y
// physics_body+0x2c   = position Z
// physics_body+0x7c   = car_forward_X (= physics_body+0x10 + 0x6c)
// physics_body+0x80   = car_forward_Y
// physics_body+0x84   = car_forward_Z
// physics_body+0x278  = int (set to -1 = 0xffffffff on create)
// physics_body+0x284  = ushort (set to 0x100 on create)
// physics_body+0x310  = uint64 (0 on create)
// physics_body+0x318  = int (1 on create)
// physics_body+0x340  = behavior fn ptr (used by FUN_004f0af0 boost state)
// physics_body+0x360..+0x37f = two 16-byte zeroed areas
// physics_body+0x458..+0x464 = saved behavior data (restored by FUN_004f0b50)
// physics_body+0x2a0  = back-reference to car entity
```

### GhostReplayBuffer (variable size)
> Two buffers: `PTR_DAT_0065fca8` = current, `PTR_DAT_0065fcb0` = previous.
> Swapped by `FUN_0044d040` (CommitGhostFrame).
```c
struct GhostReplayBuffer {   // variable size header + frames
    /* header fields at offsets relative to buffer base: */
    // +0x00..+0x27  = car creation params (quaternion, model etc.)
    // +0x28         = int, car_model_type
    // +0x2c..+0x6b  = 16-float array (config, copied to car+0x6a70)
    // +0x68         = int, frame_count (current number of recorded frames)
    // +0x6c...      = GhostFrame[frame_count]
};

struct GhostFrame {          // stride 0x18 = 24 bytes
    uint   timestamp_ms;     // +0x00  absolute time (from DAT_0ae48da8)
    byte   speed_enc;        // +0x04  speed * 5 (as byte)
    byte   pad[3];           // +0x05..+0x07
    float  quat_x;           // +0x08  (ghost frame orientation)
    float  quat_y;           // +0x0c
    float  quat_z;           // +0x10
    float  quat_w;           // +0x14
};
// car+0x80 →  data with  quaternion (physics_sub+0x14..+0x1c = xyz, +0x44 = w?)
```

### RaceResult Leaderboard (stride 0x10, max 30 entries)
> Base: `DAT_0acee800` (array). Overflow guard at entry 30.  
> Sorted ascending by time_ms; managed by `FUN_004f0890` (RegisterFinishTime).
```c
struct RaceResult {          // 0x10 bytes per entry
    longlong car_ptr;        // +0x00  pointer to CarEntity
    uint     time_ms;        // +0x08  finish time in milliseconds
    byte     dnf;            // +0x0c  1 = did not finish
    byte     pad[3];         // +0x0d
};
// car+0x6a48 = finish_time_ms (uint, 0 = not finished)
// car+0x6a4c = final_race_position (int, 0xffffffff = not set)
// car+0x6a84 = network_player_id (int, used by FUN_004f0850 lookup)
// car+0x1a93 (as int* = car+0x6a4c) = final_race_position slot index (1-based)
```

### TriggerEntity Struct (stride 0x68 = 104 bytes)
> Array at `DAT_0adb6cc0`. Count at `DAT_0adb6cc8`. Loaded by `FUN_004f1910`.
> Dispatched from linked list at `DAT_0ace4800` via dispatch `FUN_004f1560`.
```c
struct TriggerEntity {       // 0x68 bytes
    byte   active;           // +0x00  1 = active
    byte   pad[7];           // +0x01
    byte   file_data[0x44];  // +0x08..+0x4b  raw data from .trg file
    void*  dispatch_fn;      // +0x58  function ptr (from LAB_00683100 table)
    byte   dispatch_sub;     // +0x60  sub-function specifier
    /* +0x61..+0x67 unknown */
};
// TriggerDispatchTable: LAB_00683100 (stride 0x10 per type, indexed by local_b8/type)
```

### SDL_Event Union (56 bytes estimate)
```c
union SDL_Event {
    uint32 type;                     // +0x00 (event type)
    
    // Different event structures based on type
    struct {
        uint32 type;                 // 0x100 = SDL_QUIT
        uint32 timestamp;
    } quit;
    
    struct {
        uint32 type;                 // 0x300 = SDL_KEYDOWN
        uint32 timestamp;
        uint32 windowID;
        uint8 state;
        uint8 repeat;
        uint16 padding;
        SDL_Keysym keysym;
    } key;
    
    struct {
        uint32 type;                 // 0x200 = SDL_MOUSEBUTTONDOWN
        uint32 timestamp;
        uint32 windowID;
        uint32 which;
        uint8 button;
        uint8 state;
        uint8 clicks;
        uint8 padding;
        int32 x;
        int32 y;
    } button;
    
    struct {
        uint32 type;                 // 0x700 = SDL_FINGERDOWN
        uint32 timestamp;
        int64 touchId;
        int64 fingerId;
        float x;
        float y;
        float dx;
        float dy;
        float pressure;
    } tfinger;
    
    // ... many more event types
    
    byte padding[56];                // Ensure union is large enough
};
```

### Touch Input Structure (24 bytes per touch / 0x18)
```c
struct TouchState {
    int64 fingerID;                  // +0x00 (SDL finger ID, -1 if not active)
    byte isActive;                   // +0x08 (currently touching)
    byte wasPressed;                 // +0x09 (pressed this frame)
    byte padding[2];                 // +0x0A to +0x0C
    uint64 positionAndPressure;      // +0x0C (encoded position + pressure)
};

// Array of 4 touches:
// Touch[0]: 0x0ae6dce0
// Touch[1]: 0x0ae6dcf8 (+0x18)
// Touch[2]: 0x0ae6dd10 (+0x18)
// Touch[3]: 0x0ae6dd28 (+0x18)
```

---

## Global Variables (Sorted by Address)

### RVGL.exe Globals (0x00400000 - 0x0FFFFFFF)

#### Low Memory (0x00400000 - 0x00499FFF)
*Code section - no global variables*

#### Mid Memory (0x00661000 - 0x006E3FFF)
```
Address    | Size | Type      | Name                  | Description
-----------|------|-----------|---------------------- |---------------------------
0x00661640 | 4    | int       | CubeVisibility        | -cubevisi flag
0x00662a00 | 4    | float     | AlphaReference        | -alpharef value (0-255)
0x006625a1 | 1    | byte      | LateJoinEnabled       | -nolatejoin (inverted)
0x006625a2 | 1    | byte      | MulticastEnabled      | -nomulticast (inverted)
0x006635e0 | 4    | int       | ProfileLoadedFlag     | Profile loaded
0x0065c020 | 4    | int       | DebugModeFlag         | 'D' key toggle
0x0065c021 | 1    | char      | WindowMinimizedFlag   | 0=minimized, 1=active
0x0065c024 | 4    | float     | EditScale             | -editscale value
0x0065c02c | 4    | int       | NetworkPort           | -port value
0x0065c030 | 4    | int       | P2PEnabled            | -nop2p (inverted)
0x0065fe20 | 120  | TrackInfo | s_nhood1              | First track structure
[more tracks continue here at +0x78 intervals]
0x006607f8 | ?    | ?         | [end of track array]  | Last track
0x006e3088 | 4    | int       | MainThreadID          | SDL_ThreadID
0x006e308c | 4    | int       | LockFlag              | Thread lock state
0x006e3090 | 8    | void*     | MutexHandle           | SDL_Mutex*
0x006e31e8 | 4    | float     | EditOffsetX           | -editoffset X
0x006e31ec | 4    | float     | EditOffsetY           | -editoffset Y
0x006e31f0 | 4    | float     | EditOffsetZ           | -editoffset Z
0x006e30a0 | 8    | function* | GameStateFunction     | ⭐ Current game mode
0x006e30a8 | 1    | byte      | GameQuitFlag          | 1=quit, 0=running
0x006e30a9 | 1    | byte      | AKeyFlag              | 'A' key state
0x006e340c | 4    | int       | MultisampleLevel      | -multisample value
0x006e3410 | 4    | int       | WindowHeight          | -window height
0x006e3414 | 4    | int       | WindowWidth           | -window width
0x006e3429 | 1    | byte      | NoGUIKeysFlag         | -noguikey
0x006e342d | 1    | byte      | NoGammaFlag           | -nogamma
0x006e3432 | 1    | byte      | NoUserProfilesFlag    | -nouser
0x006e3433 | 1    | byte      | NoJoystickFlag        | -nojoy
0x006e3434 | 1    | byte      | NoSoundFlag           | -nosound
0x006e3438 | 1    | byte      | LargeReplaysFlag      | -largereplays
0x006e343a | 1    | byte      | TextureInfoFlag       | -texinfo
0x006e34f9 | 1    | byte      | UnknownStateFlag1     | Unknown
0x006e3640 | 16   | char[16]  | ProfileName           | -profile name
0x006e3974 | 1    | byte      | ThreadStopFlag        | Stop load thread
0x006e3976 | 1    | byte      | ThreadErrorFlag       | Thread error
0x006e3978 | 8    | void*     | Semaphore2            | SDL_sem*
0x006e3980 | 8    | void*     | Semaphore1            | SDL_sem*
0x006e3988 | 8    | void*     | LoadThreadHandle      | SDL_Thread*
```

#### High Memory (0x006F0000 - 0x0FFFFFFF)
```
Address    | Size  | Type         | Name                  | Description
-----------|-------|--------------|---------------------- |---------------------------
0x006fab50 | 8     | CarInfo*     | CarArrayPtr           | ⭐ POINTER to heap-alloc'd CarInfo array (NOT the array itself!)
0x006fab58 | 4     | int          | CarCount              | Number of cars in pool (starts 0x31 = 49, grows with user cars)
0x006fab80 | 2732  | RuntimeCar   | RuntimeCarTemplate    | ⭐ 0xAAC-byte template car — copied by InitRuntimeCarStruct (0x0043c580)
0x00f3f4c0 | 4     | int          | StandardTrackCount    | Type 2 tracks
0x00f3f4c4 | 4     | int          | OtherTrackCount       | Type 3 tracks
0x0ae48de0 | 8     | uint64       | PerformanceFrequency  | QueryPerformanceFrequency
0x0ae6dce0 | 24    | TouchState   | Touch0                | First touch input
0x0ae6dcf8 | 24    | TouchState   | Touch1                | Second touch input
0x0ae6dd10 | 24    | TouchState   | Touch2                | Third touch input
0x0ae6dd28 | 24    | TouchState   | Touch3                | Fourth touch input
0x0ae6f5a0 | 256   | char[256]    | LobbyServerAddress    | -lobby server
0x0ae6f6a0 | 4     | int          | LobbyModeFlag         | Lobby enabled
0x0ae6f6a1 | 1     | byte         | LobbyModeFlag2        | Secondary flag
0x0ae85df0 | 8     | void*        | SDLWindowPointer      | SDL_Window*
```

### libGLESv2.dll Globals (0x68000000 - 0x68FFFFFF)

```
Address    | Size | Type      | Name                  | Description
-----------|------|-----------|---------------------- |---------------------------
0x6814c020 | 4    | DWORD     | TLS_INDEX             | ⭐ Windows TLS slot (-1 = uninitialized)
0x68214410 | ?    | void*[]   | GLContextVTable       | Virtual function table for contexts
```

---

## String Tables

### Command-Line Arguments
```
Argument String     | Address Found | Sets Memory Location
--------------------|---------------|---------------------
"-nosound"          | .rdata        | 0x006e3434
"-nojoy"            | .rdata        | 0x006e3433
"-window"           | .rdata        | 0x006e3414, 0x006e3410
"-multisample"      | .rdata        | 0x006e340c
"-nogamma"          | .rdata        | 0x006e342d
"-cubevisi"         | .rdata        | 0x00661640
"-editscale"        | .rdata        | 0x0065c024
"-editoffset"       | .rdata        | 0x006e31e8/ec/f0
"-alpharef"         | .rdata        | 0x00662a00
"-nouser"           | .rdata        | 0x006e3432
"-carinfo"          | .rdata        | (sets temporary flag)
"-dev"              | .rdata        | (sets temporary flag)
"-texinfo"          | .rdata        | 0x006e343a
"-noguikey"         | .rdata        | 0x006e3429
"-profile"          | .rdata        | 0x006e3640
"-lobby"            | .rdata        | 0x0ae6f6a0, 0x0ae6f5a0
"-nop2p"            | .rdata        | 0x0065c030
"-nomulticast"      | .rdata        | 0x006625a2
"-nolatejoin"       | .rdata        | 0x006625a1
"-port"             | .rdata        | 0x0065c02c
"-serverport"       | .rdata        | (unknown location)
"-largereplays"     | .rdata        | 0x006e3438
```

### Car parameters.txt Keywords
```
Keyword String      | Field Offset | Type   | Max Size
--------------------|--------------|--------|----------
"NAME"              | +0x14        | string | 63 chars
"TPAGE"             | +0x54        | string | 63 chars
"TCARBOX"           | +0x94        | string | 63 chars
"BESTTIME"          | +0xE4        | byte   | 0/1
"SELECTABLE"        | +0xE5        | byte   | 0/1
"CPUSELECTABLE"     | +0xE6        | byte   | 0/1
"STATISTICS"        | +0xE7        | byte   | 0/1
"CLASS"             | +0xE8        | int32  | 4 bytes
"RATING"            | +0xEC        | int32  | 4 bytes
"OBTAIN"            | +0xF0        | int32  | 4 bytes
"TOPEND"            | +0xF4        | int32  | 4 bytes
"ACCELERATION"      | +0xF8        | int32  | 4 bytes
"WEIGHT"            | +0xFC        | int32  | 4 bytes
"TRANS"             | +0x100       | int32  | 4 bytes
```

### Track .inf Keywords
```
Keyword String      | Field Offset | Type   | Format
--------------------|--------------|--------|------------------
"NAME"              | +0x10        | string | 63 chars
"GAMETYPE"          | +0x68        | int32  | Integer
"DIFFICULTY"        | +0x64        | int32  | Integer
"OBTAIN"            | ?            | int32  | Integer (not stored?)
"CHALLENGE"         | +0x6C        | int32  | MM SS MS → milliseconds
"CHALLENGEREV"      | +0x70        | int32  | MM SS MS → milliseconds
```

### Developer Messages
```
String                                  | Address   | Found In
----------------------------------------|-----------|-------------------
"Developed by the RV team"              | .rdata    | Splash screen
"(c) 2010-2023"                         | .rdata    | Splash screen
"For testing purposes only"             | .rdata    | Splash screen
"NOT FOR SALE"                          | .rdata    | Splash screen
"Context has been lost."                | .rdata    | libGLESv2 error
"GL_INVALID_OPERATION"                  | .rdata    | libGLESv2 error
"GL_INVALID_ENUM"                       | .rdata    | libGLESv2 error
"GL_INVALID_VALUE"                      | .rdata    | libGLESv2 error
```

---

## Function Pointer Tables

### Game State Function Pointer
```
Location: 0x006e30a0 (8 bytes)
Type: void (*GameStateFunction)(void)

Known Values:
0x00499b30  →  SplashScreenState()      Initial state (logos)
0x00499690  →  RacingGameLoop()         ⭐ THE RACING LOOP
0x00543890  →  MultiplayerLobbyState()  Multiplayer mode
0x00406b40  →  UnknownGameState()       Another mode
0x0044c130  →  GameGaugeReplayState()   Game Gauge / Replay mode

Usage Pattern:
    while (!quit) {
        ProcessEvents();
        (*GameStateFunction)();  // Call current state
    }
```

### SDL2 Function Imports (Partial List)
```
Import Name             | Address (IAT)   | Type
------------------------|-----------------|-------------------------
SDL_Init                | varies          | int (*)(uint32)
SDL_CreateWindow        | varies          | SDL_Window* (*)(...)
SDL_GL_CreateContext    | varies          | SDL_GLContext (*)(SDL_Window*)
SDL_GL_SwapWindow       | varies          | void (*)(SDL_Window*)
SDL_PollEvent           | varies          | int (*)(SDL_Event*)
SDL_Delay               | varies          | void (*)(uint32)
... (many more)

Note: Import Address Table (IAT) dynamically resolved at load time
```

### OpenGL ES Function Pointers (Dynamic)
```
All GL functions loaded via SDL_GL_GetProcAddress()
Function count: 601 functions
Pattern: Each wrapped by libGLESv2.dll

Example:
    glDrawArrays = SDL_GL_GetProcAddress("glDrawArrays");
    // Returns: func pointer to libGLESv2.dll wrapper at 0x67??????
```

---

## Memory Allocation Patterns

### Stack Allocations (Common Patterns)
```
Function Entry:
    sub rsp, 0x28       ; Allocate 40 bytes (0x28) for local variables
    ...
    add rsp, 0x28       ; Deallocate before return
    ret

Common sizes:
    0x20 (32)   - Small functions, few locals
    0x28 (40)   - Windows x64 shadow space + small locals
    0x38 (56)   - Medium functions
    0x100+ - Large local arrays/structures
```

### Heap Allocations
```
Car Loading:
    malloc(272)         ; One CarInfo structure
    malloc(13328)       ; All 49 cars at once?

GL Context:
    malloc(32)          ; GLContextHandle
    malloc(14833+)      ; RealGLContext

String Handling:
    malloc(256)         ; Path buffers
    malloc(64)          ; Filename buffers
```

---

## Address Range Summary

### RVGL.exe Address Ranges
```
0x00400000 - 0x004fffff : Code (.text section)
0x00661000 - 0x006e3fff : Initialized data (.data)
0x006fab50 - 0x00f3ffff : Large arrays (cars, tracks)
0x0ae00000 - 0x0aeffff : Runtime allocations (SDL, touch, etc.)
```

### libGLESv2.dll Address Ranges
```
0x68000000 - 0x681fffff : Code (.text section)
0x6814c000 - 0x6814ffff : Initialized data (.data)
0x68200000 - 0x682fffff : Read-only data (.rdata)
```

### Key Insight: Memory Organization
```
Low addresses (0x00400000):  Game code
Mid addresses (0x00660000):  Game settings/flags
High addresses (0x00f00000): Game data arrays
Very high (0x0a000000):       Runtime SDL/dynamic data
```

---

## Alignment & Padding Notes

### Structure Alignment
- All structures are x64 aligned (8-byte boundaries)
- Pointers are 8 bytes
- int32/float are 4 bytes (but may have padding)
- Strings are null-terminated, may have extra padding

### Array Alignment
- CarInfo array: Each element 272 bytes (0x110) - nicely aligned
- TrackInfo array: Each element 120 bytes (0x78) - divisible by 8
- Touch array: Each element 24 bytes (0x18) - 8-byte aligned

### Track / Waypoint System Globals
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x006e3ac8 | 8     | void*   | AIZoneBuffer             | Heap-allocated AI zone buffer (freed by FUN_0040cc60)
0x006e3b00 | 8     | void*   | ZoneLookupTable          | ⭐ Spatial zone-to-waypoints table (stride 0x10 per zone)
0x006e3b08 | 8     | void*   | WaypointDataTable        | Waypoint section lookup array (stride 0x78)
0x006e3b10 | 4     | int     | TotalWaypointNodeCount   | ⭐ Total count of waypoint nodes (used as modular wrap)
0x006e3b14 | 4     | int     | WaypointZoneCount        | Count of spatial zones in lookup table
0x006fd148 | 8     | void*   | GlobalDefaultCarPtr      | Default/global car entity for AI path ops
```

### Race Session Config Globals (Batch 4)
> The race is initialized by `FUN_00450700` (RaceSetup) big switch on `GameMode`.
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x006e34c0 | 4     | int     | GameMode                 | ⭐ Race mode: 1=single race, 2=multi, 3=practice/TT, 4/6=online lobby
0x006e34c4 | 4     | int     | GameModeVariant          | Sub-mode variant (1 = restarting / spectate)
0x006e34c8 | 16    | char[]  | TrackPathString          | Current track path/name string (16 chars)
0x006e34cc | 4     | int     | PlayerCarModelIndex      | ⭐ Player's selected car model index (into CarModelDatabase)
0x006e34d0 | 4     | int     | TrackCount               | Number of track slots available
0x006e34d4 | 4     | uint    | LapsConfig               | Number of laps (uint)
0x006e34d8 | 4     | int     | CurrentTrackID           | Active track index/ID
0x006e34dc | 4     | float   | LapTimerMax              | Max lap time for rubber band? (= laps * 60)
0x006e34e0 | 4     | int     | TimeLimitSeconds         | Race time limit in seconds
0x006e34ec | 4     | int     | GameModeFlags2           | Additional game mode flags
0x006e34ed | 1     | byte    | GhostEnabledFlag         | 0=no ghost, cleared on race start
0x006e34e8 | 4     | int     | SomeRaceTimer            | Race timer/countdown, cleared on start
0x006e34ee | 1     | byte    | RandomCarFlag            | 1=randomly select cars each race
0x006e34f0 | 1     | byte    | HumanPlayerFlag          | 1=human player present
0x006e34f2 | 1     | byte    | NumLaps                  | Number of laps (byte copy)
0x006e34f3 | 1     | byte    | RandomizeTrackFlag       | 1=randomly pick a track
0x00660b04 | 4     | float   | ViewScale                 | Visual scale factor * 0.5
0x0ae8e920 | 4     | int     | CurrentGameMode          | Copy of GameMode used by race logic
0x0ae8e928 | ?     | char[]  | CurrentTrackName         | Active track name/path (source for TrackPathString)
0x0ae8e930 | 4     | int     | TotalAICarsTarget        | Target AI car count
0x0ae8e934 | 4     | int     | HumanPlayerCount         | Number of human players in session
0x0ae8e948 | 16    | char[]  | LocalPlayerName          | ⭐ Local player name string (16 chars)
0x0ae8e988 | 4     | int     | DefaultCarModel          | Default car model index
0x0ae8e998 | 4     | int     | DefaultCarColor          | Default car color variant
0x0ae8e9bc | 2     | short   | CarSelectState           | Car selection state
0x0ae8e9be | 1     | byte    | NumLaps2                 | Number of laps (source byte)
0x0ae8e9c0 | 4     | int     | LapsConfig2              | Laps config (source for DAT_006e34d4)
0x0ae8e9cc | ?     | byte    | TimeLimitRaw             | Time limit (minutes, * 0x3c = secs)
0x0ae8e9d0 | 4     | uint32  | RaceOptionsFlags         | Packed race options
0x0ae8e9d5 | 1     | byte    | SomeRaceFlag             | Race option flag  
0x0ae8e9d8 | 4     | int     | DisplayScaleFlags        | Display scale option  
0x0ae8e9dc | 4     | int     | ViewScaleRaw             | View scale raw int (→ float * 0.5 = DAT_00660b04)
0x0ae6f6ac | 4     | int     | LocalPlayerNetworkID     | ⭐ Local player's network session ID
```

### Race Entry Table Globals (Batch 4)
> `FUN_0044f1f0` = AddRaceEntry. `FUN_0044f7b0` = SetupAllRaceCars.  
> Each session has up to 30 entries (max `_DAT_00f3eb04 < 0x1e`).
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
_0x00f3eb00| 4     | int     | PlayerSlotIndex          | ⭐ Player's slot index in race entries (0-based; -1=not in race)
_0x00f3eb04| 4     | int     | TotalRaceEntries         | ⭐ Total car entries built for this race
0x00f3eb08 | 8     | u64     | RaceTypePacked           | High dword=race_mode, low dword=player_car_slot
0x00f3eb10 | 4     | int     | RaceTrackID              | Track ID for current session
0x00f3eb14 | 1     | byte    | RaceFlag1                | Race config flag 1
0x00f3eb18 | 1     | byte    | RaceFlag2                | Race config flag 2
0x00f3eb20 | 4     | uint    | RaceLaps                 | Laps for this session
0x00f3eb28 | 4     | int     | RaceLapsConfig2          | Laps config 2
0x00f3eb2c | 1     | byte    | RaceRandomCarFlag        | Random cars for session
0x00f3eb34 | 8     | u64     | RaceKartPacked           | Car choice slot packed data
0x00f3eb40 | 16    | char[]  | SessionTrackPath         | Track path for this session
0x00f3d780 | ?     | struct  | SavedSessionConfig       | Saved session config (0x136*8 = 2480 bytes) for restore
0x00f3eb50 | 0x50  |struct[] | RaceEntryTable           | ⭐ Array of RaceCarEntry (stride 0x50 = 80 bytes, max 30)
```

### RaceCarEntry Struct (stride 0x50 = 80 bytes)
> Stored at `DAT_00f3eb50[0..n]`. Built by `FUN_0044f4f0`. Used by `FUN_0044f7b0`.
```c
struct RaceCarEntry {     // 0x50 bytes per entry
    int    car_type;      // +0x00  (1=player, 3=AI, 9=spectator?)
    int    spawn_type;    // +0x04  (used in FUN_004a71d0 = ComputeSpawnOrientation)
    int    car_model;     // +0x08  index into CarModelDatabase
    int    spawn_quat;    // +0x0c  spawn quaternion
    int    pad0[2];       // +0x10
    int    network_id;    // +0x18  player's network session ID
    int    is_local;      // +0x1c  1 = this is the local human player
    int    pad1[8];       // +0x20
    char   name[16];      // +0x40  player/bot name string (16 chars)
};
// Note: puVar2 in FUN_0044f7b0 points to the name field (+0x40),
//       so accesses are puVar2[-0x40/4..] relative to name
```

### Local Player Car Globals (Batch 4)
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x0acee7e8 | 8     | void*   | LocalPlayerCarPtr        | ⭐ The LOCAL PLAYER's car entity (set by FUN_0044f7b0 when is_local != 0)
0x00f43400 | ?     | ?       | InputBindingTable        | Input binding table (stride 0x44 per binding, indexed by slot)
0x00f436c0 | 4     | int     | JoystickDeviceID         | Joystick device ID (-1=none)
0x00f434a0 | 1     | byte    | TimeTrialModeActive      | 1=time trial mode active (uses DAT_016f8e40 ghost tables)
0x00f434a2 | 1     | byte    | RandomColorEnabled       | 1=use random colour variant for car
0x00f434b0 | 4     | int     | GhostType                | Ghost display type (FUN_0047a310 "GhostType")
0x00f4349b | 1     | byte    | ShowGhost                | Show ghost flag (FUN_0047a310 "ShowGhost")
```

### Route / Track Progress Globals (Batch 4)
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x006e3ae0 | 4     | float   | TotalRouteLength         | ⭐ Total track route length (lap length in route units)
0x006e3ae4 | 4     | int     | DefaultRouteSectionIndex | Default RouteSection index (car+0x6910 init)
0x006e3ae8 | 4     | int     | WaypointSystemReady      | Gate flag: !=0 → UpdateCarRoutePosition runs
0x006e3aec | 4     | int     | SecondaryRouteSectionIdx | Secondary route section for multi-path tracks
0x006e3af0 | 8     | void*   | RouteSectionTable        | ⭐ Array of RouteSection nodes (stride 0x50 = 80 bytes)
0x006fab50 | 8     | void*   | CarModelDatabase         | Per-car model DB (stride 0x110 = 272 bytes, max 6 cars)
0x006fd140 | 1     | byte    | PlayerCarActiveFlag      | 1 = player car slot is active/valid
0x006fd141 | 1     | byte    | PlayerCarInitializedFlag | 1 = player car needs init/reinit
0x006fd150 | 8     | void*   | ActiveGhostPathPtr       | Current ghost path data ptr (into GhostReplayBuffer)
```

### Car Entity Pool / Linked List Globals (Batch 4)
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
_0acee7e4  | 4     | int     | TotalActiveCarCount      | ⭐ Count of cars currently alive (++/-- by Create/Destroy)
0x0acee9e0 | 8     | void*   | CarListTail              | Tail of active car linked list (oldest car added)
0x0acee9e8 | 8     | void*   | CarListHead              | ⭐ Head of active car linked list (most recently added)
0x0adb69b0 | 8     | void*   | FreeCarPoolHead          | Head of free/pooled car entity list
```

### Ghost / Lap Replay Globals (Batch 4)
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x0065fca8 | 8     | PTR     | PTR_GhostBufCurrent      | ⭐ Ptr to current ghost replay buffer (creation params)
0x0065fcb0 | 8     | PTR     | PTR_GhostBufPrevious     | Ptr to previous ghost replay buffer (double-buffer swap)
0x0093d2d8 | 4     | int     | GhostDirtyFlag           | Set 0 on init/clear (FUN_0044cb60/ce10)
0x00abd3e0 | ?     | struct  | GhostReplayStorage       | Per-car ghost replay arrays (stride 0x3001b each)
0x00f42560 | 1     | byte    | LapSwapGate              | Controls ghost buffer swap (FUN_0044d040 gate)
0x00f434a0 | 1     | byte    | TimeTrialModeFlag        | 0=race, 1=time trial (selects ghost/leaderboard paths)
```

### Race Finish Leaderboard Globals (Batch 4)
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x0acee800 | 240   | struct  | RaceResultTable          | ⭐ Sorted race results array (stride 0x10, max 30 entries)
0x0acee808 | 4+    | uint    | RaceResultTimes          | Finish times within RaceResultTable (+0x08 per entry)
0x0acee80c | 1+    | byte    | RaceResultDNF            | DNF flags within RaceResultTable (+0x0c per entry)
0x0acee9c0 | 8     | void*   | RaceResultEnd            | Past-end sentinel for RaceResultTable shift during insert
```

### Trigger System Globals (Batch 4)
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x0adb6cc0 | 8     | void*   | TriggerArray             | malloc'd TriggerEntity array (stride 0x68)
0x0adb6cc8 | 4     | int     | TriggerCount             | Number of triggers loaded from file
0x0ace4800 | 8     | void*   | TriggerDispatchList      | Linked list of active trigger entities (+0x08 links)
0x00683100 | ?     | void*[] | TriggerDispatchTable     | Function pointer table indexed by trigger type
```

### Race / AI Race Globals (Additional)
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x006e3443 | 1     | byte    | EditorModeActive2        | Secondary editor activation flag  
0x006e34f9 | 1     | byte    | UnknownStateFlag1        | Checked in FUN_004d0c70 dispatch
0x0ae6fc67 | 1     | byte    | RaceActiveFlag2          | Gate check: FUN_004d0c70 returns if 0
```

### Batch 3: Additional Network / Rendering Globals
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x0053a460 | funk  | void*   | GLBufferResize()         | Resize GL vertex/index buffers (allocate if too small)
0x0053a800 | funk  | void*   | GLBufferFlush()          | Upload buffered geometry to GL and draw
0x0053ab10 | funk  | void*   | GLPostDraw()             | GL post-draw cleanup
0x004fe260 | funk  | void*   | RenderNetworkCar()       | Render one networked car entity
0x004fdfc0 | funk  | void*   | NetworkCarRenderAll()    | Render all networked cars (mode-dependent)
```

---

## 🆕 New Globals Discovered (Batch 3 — Full File Pass)

### Command-Line / Window Flags
```
Address    | Size | Type    | Name                     | Description
-----------|------|---------|--------------------------|-------------------------------------------
0x006e3433 | 1    | byte    | NoJoystickFlag           | -nojoy → set to 1
0x006e3434 | 1    | byte    | NoSoundFlag              | -nosound → set to 1
0x006e340c | 4    | int     | MultisampleLevel         | -multisample N
0x006e3410 | 4    | int     | WindowHeight             | -window H
0x006e3414 | 4    | int     | WindowWidth              | -window W
0x006e342d | 1    | byte    | NoGammaFlag              | -nogamma
0x006e343f | 1    | byte    | GlobalReverseMode        | 'D' key toggle for reverse test mode
0x006e3440 | 1    | byte    | EditModeActive           | Editor is active
0x006e3442 | 1    | byte    | PlayerInputEnabled       | Player input allowed
0x006e3443 | ?    | ?       | [edit flags]             | Several editor-related flags follow
0x0065c020 | 4    | int     | WindowedModeFlag         | 0=fullscreen, 1=windowed
0x0065c021 | 1    | byte    | WindowIsFocused          | SDL window has focus
0x0065c024 | 4    | float   | EditScaleFactor          | -editscale value
0x00661640 | 4    | int     | CubeVisibility           | 0=hidden (-cubevisi), default=1
```

### SDL / Event System
```
Address    | Size | Type    | Name                     | Description
-----------|------|---------|--------------------------|-------------------------------------------
0x0ae85df0 | 8    | void*   | SDLMainWindow            | SDL_Window* main game window
0x006e30a8 | 1    | byte    | QuitRequested            | 1 = SDL_QUIT received
0x006e30a9 | 1    | byte    | AltA_KeyFlag             | Alt+A key was pressed
0x006e30c0 | 256  | char[]  | TextInputBuffer          | SDL_TEXTINPUT text (max 256 chars)
0x006e3088 | 4    | int     | MainThreadID             | SDL_GetThreadID(main)
0x006e308c | 4    | int     | MutexLockedFlag          | Thread lock state
0x006e3090 | 8    | void*   | MutexHandle              | SDL_Mutex*
0x006e3975 | 1    | byte    | ErrorInLoadThread        | Load thread error flag
0x006e3976 | 1    | byte    | ThreadErrorFlag2         | Secondary thread error
0x006e3978 | 8    | void*   | ErrorSemaphore           | SDL_sem* for load thread sync
```

### Timing / Frame Pacing
```
Address    | Size | Type    | Name                     | Description
-----------|------|---------|--------------------------|-------------------------------------------
0x0ae48da4 | 4    | int     | GamePaused               | Non-zero = game is paused
0x0ae48dc0 | 4    | float   | PausedTimeAcc            | Time accumulator during pause
0x0ae48dc8 | 8    | float   | TotalTimeAccumulator     | Total accumulated game time
0x0ae48dd8 | 8    | float   | DeltaTimeRaw             | Raw delta time this frame
0x0ae48de0 | 8    | uint64  | TimestampDenominator     | QueryPerformanceFrequency denominator
0x0ae48df0 | 8    | uint64  | TimestampNumerator        | Current QPC value
0x0065c03c | 4    | float   | DeltaTime                | ⭐ Scaled delta time (seconds/frame) — used by ALL physics
```

### Race / Track Globals
```
Address    | Size | Type    | Name                     | Description
-----------|------|---------|--------------------------|-------------------------------------------
0x006e34c0 | 4    | int     | GameMode                 | Current game type (4=race, 5=replay, 0xd=tag, 0x10=time trial)
0x006e34c4 | 4    | int     | RaceSubstate             | Sub-state within race modes
0x006e34dc | 4    | int     | TotalLaps                | Lap count for this race
0x006e34e4 | 4    | int     | RaceState                | 2=racing, other=transition
0x006e34f2 | 1    | byte    | SpecialModeFlag          | Special race mode flag
0x006e39c0 | n*8  | void*[] | SortedRacePositions      | ⭐ Car pointer array sorted by race position
0x006e3ab0 | 4    | int     | ActiveCarCount           | Number of cars in race
0x006e3ae0 | 4    | float   | TotalWaypointCount       | Track waypoint constant (used for scoring)
0x006e3ae4 | 4    | int     | InitialWaypointIndex     | Starting waypoint index
0x0ae6efe0 | 1    | byte    | Player1InputEnabled      | Player 1 input active
0x0ae6efe1 | 1    | byte    | AIInputEnabled           | AI cars receive input
0x0ae8e920 | 4    | int     | GameMode2                | Mirror of GameMode (0xd=tag, 0x10=time trial)
0x0ae6fc67 | 1    | byte    | SomeRaceActiveFlag       | Gate check in FUN_004d0c70
0x00f43406 | 1    | byte    | MultiplayerStartedFlag   | MP race started
0x00f4349d | 1    | byte    | RubberBandEnabled        | Rubber band AI enabled
```

### AI / Rubber Band Config Table (`PTR_DAT_0065c100`)
```
Table Offset | Type  | Name                      | Description
-------------|-------|---------------------------|-------------------------------------------
+0x20        | float | CheckpointWindowTimeout   | Time window for checkpoint recording
+0x24        | float | CheckpointWindowTimeout2  | Secondary timeout
+0x28        | float | MinMovementDistanceSq     | Min displacement² for stuck detection
+0x2c        | float | StuckTimerMax             | Max stuck timer before rescue
+0x30        | float | RubberBandSpeedThreshold  | Speed at which rubber band activates
```

### Camera / Viewport System
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x016f9928 | 48    | float[] | ViewMatrix3x4            | 3×4 view matrix (row-major)
0x016f99e0 | ?     | float[] | ProjectionInfo           | Projection parameters
0x006e3470 | 4     | float   | FocalLength              | near * FOV product
0x006e347c | 4     | float   | NearClip                 | Near clip plane distance
0x006e3480 | 4     | float   | FarClip                  | Far clip plane distance
0x00661848 | 4     | float   | HalfScreenWidth          | 320.0 (for 640-wide)
0x00661844 | 4     | float   | HalfScreenHeight         | 240.0 (for 480-tall)
0x016f9a44 | 4     | int     | ActiveCameraSlot         | Current camera slot (0-7)
0x016f9a48 | 4     | float   | ViewportBottom           | Viewport bottom Y
0x016f9a4c | 4     | float   | ViewportYTop             | Viewport top Y
0x016f9a50 | 4     | float   | ViewportXOffset          | Viewport X offset
0x016f9a68 | 4     | float   | ViewportX2               | Viewport right X
0x016f9a6c | 4     | float   | ViewportY2               | Viewport bottom Y (2nd)
```

Camera slot layout (8 slots × 0x140 bytes each, base `DAT_016f8ee0`):
```
Slot N base = DAT_016f8ee0 + N * 0x140
  +0x10c = some_enabled_flag  (set to 1 when slot active)
  +0xc0  = fade_alpha          (set to 0.5f by ghost transparency update)
```

Known slot flag addresses:
```
Address    | Slot | Field
-----------|------|-------
0x016f8fec | [0]  | slot0_enabled_flag
0x016f9128 | [1]  | slot1_enabled_flag
0x016f9268 | [2]  | slot2_enabled_flag
0x016f93a8 | [3]  | slot3_enabled_flag
0x016f94e8 | [4]  | slot4_enabled_flag
0x016f9628 | [5]  | slot5_enabled_flag
0x016f9768 | [6]  | slot6_enabled_flag
0x016f98a8 | [7]  | slot7_enabled_flag
```

ScreenResPointer (pointer stored at `DAT_016f9a60`):
```
+0x00  float viewport_x
+0x04  float viewport_y
+0x08  float viewport_w
+0x0c  float viewport_h
+0x10  float screen_width   (640.0)
+0x14  float screen_height  (480.0)
```

### Physics World
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x0ace4800 | 8     | void*   | GameObjectListHead       | Head of track-object linked list
0x0ace47d0 | 4     | float   | WorldMinX                | World bounding box min X
0x0ace47d4 | 4     | float   | WorldMaxX                | World bounding box max X
0x0ace47d8 | 4     | float   | WorldMinY                | World bounding box min Y
0x0ace47dc | 4     | float   | WorldMaxY                | World bounding box max Y
0x0ace47e0 | 4     | float   | WorldMinZ                | World bounding box min Z
0x0ace47e4 | 4     | float   | WorldMaxZ                | World bounding box max Z
0x0ace49a0 | 4     | float   | GhostCarAlpha            | Ghost car transparency (0.5f = 50%)
```

### AI Node Editor
```
Address    | Size    | Type    | Name                     | Description
-----------|---------|---------|--------------------------|-------------------------------------------
0x006f94a0 | 0x108000| AINode[]| AINodeEditBuffer         | Edit-mode AI node array (max 0x1000 nodes × 0x108 bytes)
0x006f94aa | 2       | short   | AINodeCount              | Number of AI nodes in edit buffer
0x006e3500 | 4       | int     | EditorModeType           | Current editor sub-mode (0=off, 1-11=types below)
0x006e3b20 | 8       | void*   | AINodeMesh1              | Runtime AI node render mesh 1
0x006e3bb8 | 8       | void*   | AINodeMesh2              | Runtime AI node render mesh 2
0x006e3c50 | 4       | int     | AINodeEditCount          | Number of nodes in current edit session
0x006f9b00 | 8       | void*   | SelectedAINodePtr1       | First selected AI node
0x006f9b20 | 8       | void*   | SelectedAINodePtr2       | Second selected AI node
0x006f9b28 | 1       | byte    | AINodeEditMode           | AI node edit sub-mode flag
0x006f9b29 | 1       | byte    | AINodeEditMode2          | Secondary AI node edit mode
```

Editor modes (DAT_006e3500):
```
1  = AI node edit (primary)
2  = AI node edit (alternate)
3  = zone edit
4  = surface/material edit
5  = AI node file I/O
6  = region edit
7  = portal edit
8  = instance object edit
9  = (unknown)
10 = vertex color edit
11 = (unknown)
```

### Multiplayer Rendering Buffers
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x0adccb00 | 4     | int     | MultiplayerCarCount      | Number of networked cars to render
0x0adccb08 | 8     | void*   | MultiplayerCarArray      | ⭐ Base of NetworkCarEntry array (stride 0x1e0)
0x0adccb20 | 4     | int     | MultiplayerVtxBufTarget  | Vertex buffer target for MP render
0x0adccb24 | 4     | int     | MultiplayerIdxBufTarget  | Index buffer target for MP render
0x0adccb28 | 4     | int     | MultiplayerVtxBuf2       | Second vertex buffer target
0x0adccb2c | 4     | int     | MultiplayerIdxBuf2       | Second index buffer target
0x0ae7e990 | 8     | void*   | MPVertexBufferPtr        | Vertex data buffer for MP
0x0ae7e998 | 8     | void*   | MPIndexBufferPtr         | Index/element data buffer for MP
0x0ae7fc5b | 1     | byte    | RGBASwapMode             | 0=swap R/B channels, 1=no swap
```

### UI System
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x01694228 | 4     | int     | UIElementCursor          | Current position in UI element pool
0x0169422c | 4     | int     | UIElementCapacity        | Total UI element pool capacity
0x01694230 | 8     | void*   | UIElementPoolBase        | Base pointer of UI element pool
0x016942d8 | 4     | float   | FadeAmount               | Screen fade: 0.0=transparent, 1.0=opaque
0x016942dc | 4     | int     | RaceState2               | Race state machine (0-6)
0x0ae7fc4c | 1     | byte    | NetworkModeActive        | Non-zero = multiplayer mode
0x0ae7fc8d | 1     | byte    | TransitionFlag1          | UI transition state 1
0x0ae7fc92 | 1     | byte    | TransitionFlag2          | UI transition state 2
0x0ae7e9ac | 4     | int     | UITransitionCounter      | Transition frame counter
0x0ae8f478 | 8     | void*   | MenuTextStringPtr        | Current menu text string
0x0ae8fe88 | 8     | void*   | MenuTextPtr2             | Second menu text pointer
0x0ae8ffe8 | 8     | void*   | MenuTextPtr3             | Third menu text pointer
```

### Rendering Scratch / Global Temps
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x017dfba0 | 4     | float   | DefaultVertexLight       | Default per-vertex lighting value
0x017fffe8 | 12    | float[3]| TempDeltaVector          | Global scratch 3D vector for calculations
0x006e3468 | 4     | float   | ScreenScaleX             | Horizontal world-to-screen scale
0x006e346c | 4     | float   | ScreenScaleY             | Vertical world-to-screen scale
0x00660b04 | 4     | float   | ParticleSpawnRate        | Particle emission rate constant (used in FUN_004d0c70)
```

### Animation System Globals (FUN_004d0c70)
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x01815060 | ?     | blob    | AnimKeyframeDataBase     | Keyframe data table (zeroed at init)
0x01814760 | ?     | int64[] | AnimAudioSlotTable       | Audio slot indices (-1=none, 16 entries)
0x01815d28 | ?     | float[] | AnimFrameDurations       | Per-frame duration data (stride 0x252 per anim)
0x01815860 | ?     | byte[]  | AnimChannelFlags         | Per-channel active flags (stride 0x4c)
0x018aa528 | ?     | int[]   | AnimFrameCounts          | Frame count per animation
0x01815864 | ?     | byte[]  | AnimLoopFlags            | Loop mode flags per animation
```

Animation loop modes (0x01815860):
```
0 = loop (wraps frame counter)
1 = play-once and stop (sets done flag at end)
2 = ping-pong (bounces forward/backward)
3 = (default/unknown)
```

Animation easing types (in keyframe data at +0x2c):
```
0 = linear
1 = ease-in (quadratic)
2 = ease-out (quadratic)
3 = smooth-step (cubic Hermite)
4 = bounce
```

---

## 🆕 New Globals Discovered (Batch 2)

### Race State
```
Address    | Size | Type    | Name                   | Description
-----------|------|---------|------------------------|-------------------------------------------
0x016942dc | 4    | int     | RaceState              | ⭐ Race state 0-6. Read by GetRaceState (0x0048a5c0), written by RaceInit2 (0x0048a560)
0x016942d8 | 4    | float   | FadeAmount             | Screen fade: 0.0=transparent, 1.0=full. Updated every frame by UpdateFadeTransition
0x01694228 | ?    | byte[]  | MultiplayerCmdBuf      | Multiplayer command buffer (also 0x22c/0x30 offsets)
```

### Rendering & Animation
```
Address    | Size | Type    | Name                   | Description
-----------|------|---------|------------------------|-------------------------------------------
0x0ace49a0 | 4    | float   | BgColorOscillator      | Drives background color sine wave in RaceBackgroundRenderer
0x016d6468 | ?    | void*   | ActiveTextureHandle    | Bound texture, used in SetupGLRenderState
0x016cf6a0 | ?    | Sprite[]| SpriteSlotArray        | Sprite/texture slot table, stride 0x25 per slot. Base for FreeTextureSlot
0x00f92ba0 | 64+  | float[] | QuadVertexBuffer       | 2D quad vertices filled by DrawSprite2D (NDC coords)
0x006e3468 | 4    | float   | ScreenScaleX           | Horizontal world-to-screen scale (used by DrawSprite2D)
0x006e346c | 4    | float   | ScreenScaleY           | Vertical world-to-screen scale
```

### Input - Keyboard / Mouse
```
Address    | Size  | Type     | Name                   | Description
-----------|-------|----------|------------------------|-------------------------------------------
0x0ae6ef00 | 512   | byte[512]| KeyStateCurrent        | Current keyboard state (SDL scancodes, 0x200 bytes)
0x0ae6ed00 | 512   | byte[512]| KeyStatePrevious       | Previous frame keyboard state (for pressed/released detection)
0x0ae6dcc0 | 4     | int32    | MousePosition          | Absolute mouse XY from SDL_GetMouseState
0x0ae6dcb4 | 4     | float    | MouseDeltaX            | X delta this frame (SDL_GetRelativeMouseState)
0x0ae6dcb8 | 4     | float    | MouseDeltaY            | Y delta this frame
0x0ae6dcc8 | 4     | int32    | MouseAccumX            | Accumulated horizontal mouse movement
0x0ae6dccc | 4     | int32    | MouseAccumY            | Accumulated vertical mouse movement
0x0ae6dca0 | 8     | int32[2] | SteeringOutput         | Packed XY steering output (written by MouseInputUpdater)
0x0ae6dca8 | 4     | int32    | ThrottleOutput         | Throttle/brake output
0x0ae6dcbc | 4     | int32    | MouseScrollWheel       | Scroll wheel value
0x006fcec8 | 1     | byte     | NoKeyboardMode         | When set: clears keyboard/controller state each frame
```

### Input - Controllers
```
Address     | Size  | Type     | Name                   | Description
------------|-------|----------|------------------------|-------------------------------------------
0x00f43400  | ?     | struct[] | ControllerInfoArray    | Array of controller descriptors. Stride 0x44 (68 bytes). Slots: DAT_00f43400, +0x44, +0x88...
0x00f43400+0x2c0 | 4  | int    | Controller0Handle      | SDL_GameController* (or -1 if absent)
0x00f43400+0x28c | 2  | short  | Axis0Binding           | Axis index for steering/X
0x00f43400+0x290 | 2  | short  | Axis1Binding           | Axis index for Y / throttle
0x00f43400+0x28f | 1  | byte   | Axis0Type              | 2 = centered stick
0x00f43400+0x293 | 1  | byte   | Axis1Type              | 2 = centered stick
0x00f43400+0x2c4 | 4  | int    | DeadZone               | Dead-zone threshold
0x00f43400+0x2c8 | 4  | int    | Sensitivity            | Sensitivity max (default 100)
0x00f436c4  | 4     | int      | Axis0DeadZone          | Controller 0 dead-zone (stride 0x11 for per-controller)
0x00f436c8  | 4     | int      | Axis0Sensitivity       | Controller 0 sensitivity
0x0ae6e520  | 500   | struct   | ControllerStateOutput  | Current controller axis/button values (stride 500 per controller)
0x0ae6dd40  | 500   | struct   | ControllerStatePrev    | Previous frame controller state
0x0ae6f100  | ?     | struct   | ControllerAxisConfig   | Axis config array (stride 0x13 qwords = 0x98 bytes per controller)
```

### Car System - Static Pool
```
Address    | Size  | Type     | Name                   | Description
-----------|-------|----------|------------------------|-------------------------------------------
0x006fab50 | 8     | CarInfo* | CarArrayPtr            | ⭐ Pointer to heap-alloc'd CarInfo array
0x006fab58 | 4     | int      | CarCount               | Car count (49 + user cars). Growth via realloc in LoadUserCars
0x006fab80 | 2732  | Runtime  | RuntimeCarTemplate     | ⭐ 0xAAC-byte template copied by InitRuntimeCarStruct
0x0065c0f8 | 4     | int      | CarSelectionMode       | 0=all selectable, 1=class filter, 2=first-8 only
0x006e3431 | 1     | byte     | SkipUserCarsFlag       | When set, LoadUserCars skips the cars/ directory scan
0x006fb640 | ?     | ?        | CarClassFilter1        | Used by UpdateCarSelectability class filtering
0x006fb64a | ?     | ?        | CarClassFilter2        | Same
0x006fb642 | ?     | ?        | CarClassFilter3        | Same
0x670fa4   | ?     | float[]  | CarPhysicsParamTable   | Physics constants table, stride 0x28 per car class
```

### Race / Track Setup
```
Address    | Size  | Type     | Name                   | Description
-----------|-------|----------|------------------------|-------------------------------------------
0x006e34c0 | 4     | int      | GameMode               | Current game type (0xD=race, 4/6=?, 5=replay). Written by multiple state funcs
0x0ae8e920 | 4     | int      | GameMode2              | Mirrors GameMode — both set together in state transitions
0x006e34c8 | 4     | int      | CurrentTrackIndex      | Index of current track
0x006fb6a0 | 4     | int      | ForcedTrackMode        | When set, skips random-track pick (used by SetupRaceTrackRandom)
0x006e34f0 | 1     | byte     | ReverseRaceFlag        | Bit 1: race in reverse direction (from replay data)
0x006e34d8 | 4     | int      | LapCount               | Number of laps (from replay/session header)
0x006e34d4 | 4     | int      | PlayerStartPos         | Starting position (default 1)
0x006e34ec | 4     | int      | RaceParticipantCount   | Number of participants
```

### Race-End / Timing
```
Address    | Size  | Type     | Name                   | Description
-----------|-------|----------|------------------------|-------------------------------------------
0x0ae8e860 | 4     | int      | RaceEndState           | State machine: 0=init, 1=transition, 2=active, 3=fading
0x0ae8e864 | 4     | float    | RaceEndTimer           | Accumulates delta time during race-end sequence
0x0ae8e870 | 4     | float    | RaceEndDuration        | How long the end sequence lasts
0x0ae8e86c | 4     | int      | WinnerLapCount         | Laps completed by winner
0x0ae8e868 | 4     | int      | LapThreshold           | Lap count needed to consider win
0x0ae8e88c | 4     | int      | RaceEndDisplayCode     | Display mode code (0x17=normal, 0x13=early)
0x0ae8e8c8 | 8     | void*    | WinnerCarPointer       | ⭐ Pointer to the winner's runtime car struct
0x0ae8e905 | 1     | byte     | ChampionshipContinue   | Flag: continue championship or end
0x0ae8e907 | 1     | byte     | ChampionshipWin2       | Secondary championship win flag
0x00662b20 | 4     | float    | UITimerFloat           | Countdown/display float (reset to 0x3f59999a = 0.85)
```

### Timing / Delta Time
```
Address    | Size  | Type     | Name                   | Description
-----------|-------|----------|------------------------|-------------------------------------------
0x0065c03c | 4     | float    | DeltaTime              | ⭐ Seconds per frame. Written by CalcDeltaTime every frame
0x0065c038 | 4     | float    | DeltaTimeRaw           | Raw (unscaled) delta ratio
0x0ae48da0 | 4     | float    | SlowMotionMultiplier   | When != 0, scales delta time (for debug slow-mo)
0x0ae48dc8 | 8     | int64    | AccumulatedTime        | Accumulated game time (incremented by delta each frame)
0x0ae48dd8 | 8     | int64    | DeltaTimeSigned        | Signed frame delta for accumulator
0x0ae48dac | 4     | uint     | FrameCounter           | Frame count (used by multiplayer, timing)
```

### Multiplayer / Network
```
Address    | Size  | Type     | Name                   | Description
-----------|-------|----------|------------------------|-------------------------------------------
0x0ae7fc4c | 1     | byte     | NetworkModeActive      | ⭐ Non-zero = network/MP mode active (skips rendering stubs)
0x0ae7fc52 | 1     | byte     | MulticastSendFlag      | Triggers glBlendFuncSeparate call in SetupGLRenderState
0x0ae7fc5a | 1     | byte     | StencilBufferEnabled   | Enables glDisable(0x8457) = GL_STENCIL_TEST
0x0ae7fc5b | 1     | byte     | RGBASwapMode           | When 0 in DrawSprite2D: swaps R/B channels of color
0x0ae7e9ac | 4     | int      | GLStateChangeCounter   | Counts how many GL state changes were made this frame
0x0ae7e9a8 | 4     | int      | GLDrawCallCounter      | Counts draw calls (glBindTexture etc.) this frame
0x0ae7fc84 | 4     | int      | BlendSrcFactor         | Cached glBlendEquation src factor (0x302)
0x0ae7fc88 | 4     | int      | BlendDstFactor         | Cached glBlendEquation dst factor (0x303)
0x0ae7fc9c | 4     | int      | TextureBindState       | Cached texture bind slot (0x5e = bound)
0x0ae6fc38 | 8     | byte*    | PacketBufferPtr        | Points at current inbound network packet
0x0ae6fc34 | 4     | int      | PacketSize             | Size of current packet
0x0ae6fc20 | 4     | uint     | PlayerCount            | Number of network players
0x0ae6f6a4 | 4     | uint     | LocalPlayerID          | Local player's network ID
0x0ae6f6ac | 4     | uint     | LocalPlayerAltID       | Alternate local player ID
0x006625a3 | 1     | byte     | NetworkReadyFlag       | Must be 0x01 for certain packet types to process
```

### FPS / Profiling
```
Address    | Size  | Type     | Name                   | Description
-----------|-------|----------|------------------------|-------------------------------------------
0x006fceb0 | 4     | int      | FramesThisSecond       | Incremented each frame, reset each second
0x006fceac | 4     | int      | TotalFrameCount        | Cumulative frame count since race start
0x006fcea8 | 4     | int      | MinFPS                 | Minimum FPS recorded
0x006fcea4 | 4     | int      | MaxFPS                 | Maximum FPS recorded
0x006fcee0 | 600   | short[300]| FPSHistoryArray       | Per-second FPS history (max 300 seconds)
0x006fd138 | 4     | int      | FPSHistoryCount        | Number of entries in FPSHistoryArray
0x006fd13c | 4     | int      | FPSFlushPending        | Flag to suppress FPS recording temporarily
0x00f432a4 | 1     | byte     | FPSDumpTrigger         | When set: writes profiles/fps.txt on next second boundary
0x00f3fb94 | 4     | int      | FPSDumpState           | State for write (set to 7 when triggered)
```

### Item / Pickup System
```
Address    | Size  | Type     | Name                   | Description
-----------|-------|----------|------------------------|-------------------------------------------
0x0ace47c0 | 4     | int      | ActiveItemIndex        | Currently active item/pickup index
0x016f8ec0 | ?     | struct[] | ItemStateArray         | Item state: base + (DAT_016f9a44 * 0x28) offset
0x016f9a44 | 4     | int      | ItemStateIndex         | Index into ItemStateArray
0x01800298 | ?     | struct[] | ItemDataArray          | Item data: base + (index * 0x138) offset
0x01800228 | ?     | struct[] | ItemTrackArray         | Item track data: base + (index * 0x27) offset
0x00f3fbc8 | 4     | float    | ItemCountdownTimer     | Timer for item visuals / pickup countdown
0x00660820 | 4     | float    | ItemAnimFloat1         | Item animation helpers
0x00660824 | 4     | float    | ItemAnimFloat2         | Item animation helpers
```

### Gameplay - Linked Car List (Runtime)
```
Address    | Size  | Type     | Name                   | Description
-----------|-------|----------|------------------------|-------------------------------------------
0x0acee9e8 | 8     | void*    | RuntimeCarListHead     | ⭐ Head of linked list of runtime car structs
                                                          | Each node: [+4]=type, [+0x10]=next ptr, [+0xf90]=timer
0x0acee7e8 | 8     | int*     | PlayerCarSlotPtr       | ⭐ Pointer to the PLAYER car slot (car 0).
                                                          | InitCarSlotFull checks if param_1==this to detect car-0.
                                                          | When set: copies template to DAT_006fa0a0
0x0acee7e8 | 8     | int*     | [same as above]        | (DAT_0acee7e8 = linked list containing player car)
0x006fa0a0 | 2732  | RuntimeCar | PlayerCarDataCopy    | Copy of player's current car data (0xAAC bytes) stored apart
0x006fab44 | ?     | ?        | PlayerCarDataTail      | Tail/suffix of player car copy
0x016f8e40 | 8     | void*    | CamEntityPtr_0         | ⭐ Pointer to camera entity slot 0 (split-screen cam). Read by FUN_004a4910.
                                                          | Points into inline array at 0x016f8ee0. Cleared by FUN_004a5500.
0x016f8e48 | 8     | void*    | CamEntityPtr_1         | ⭐ Pointer to camera entity slot 1
0x016f8e50 | 4     | void*    | CamEntityPtr_2         | Pointer to camera entity slot 2
0x016f8e54 | 4     | (pad)    | CamEntityPtr_2_hi      | High dword of above (8-byte ptr)
0x016f8e58 | 8     | void*    | CamEntityPtr_3         | Pointer to camera entity slot 3
0x016f8e60 | 8     | void*    | CamEntityPlayerPrimary | ⭐⭐ Pointer to PLAYER primary camera entity.
                                                          | FUN_004a5ce0 checks (param_1 == DAT_016f8e60) to detect player cam.
                                                          | FUN_004a1290 same check. 259 XREFs. Cleared to 0 by FUN_004a5500.
0x016f8e68 | 8     | void*    | CamEntityPtr_MiniA     | Pointer to minimap/sub-viewport entity A
0x016f8e70 | 8     | void*    | CamEntityPtr_MiniB     | Pointer to minimap/sub-viewport entity B
0x016f8e78 | 8     | void*    | CamEntityPtr_MiniC     | Pointer to minimap/sub-viewport entity C
0x016f8e80 | 8     | void*    | CamEntityPtr_Extra     | Pointer to 9th camera entity (extra)
0x016f8ec0 | ?     | ?        | ItemStateArray         | Item/pickup state array (ItemSystemUpdater: DAT_016f8ec0 + idx*0x28)
0x016f8ee0 | 9×320 | CamEnt[] | CameraEntityArray      | ⭐ Inline static camera entity array.
                                                          | 9 entries × 0x140 (320 bytes) each. Ends at 0x016f98e0.
                                                          | Each slot: ptr[-8]=0xFFFFFFFFFFFFFFFF (ID/handle), state at +0x12c,
                                                          | viewport index at +0x128, viewport rect at +0x100..+0x124.
                                                          | Type field at [0]=2 for camera, [1]=subtype.
                                                          | Reset by FUN_004a5500; initialized by FUN_004a5ce0.
0x016f9a44 | 4     | int      | ItemActiveViewport     | Item system viewport index (ItemSystemUpdater)
0x016f9a48 | 16    | float[4] | ViewportFOVScaled      | Scaled FOV values (written by CameraEntityViewportUpdate)
0x016f9a54 | 4     | float    | ViewportDepthA         | Viewport depth param A (written by FUN_004a4910)
0x016f8e95 | ?     | (pad)    | [CamNodeArrayPad]      | (between camera entity ptrs and node array)
0x016f8e98 | 4     | int      | PickupNodeCount        | ⭐ Count of entries in node array at 0x016f8ea0. Used by FindNearestPickupNode.
0x016f8ea0 | n×56  | Node[]   | PickupNodeArray        | ⭐ Camera attachment / pickup node array. Stride 0x38 (56 bytes) per node.
                                                          | Node layout: [0]=type(int), [3..5]=XYZ floats, [0xc]=bitmask(uint64).
                                                          | FindNearestPickupNode (0x004a5b50) iterates this with dist^2 nearest-check.
0x016e4428 | 8     | uint64   | NodeBitmaskFilter      | Bitmask applied in FindNearestPickupNode: skip nodes where (node[0xc] & this) != 0
0x016f9a80 | 8     | ?        | ViewportExtra          | Extra viewport data (written to cam+0x120 by FUN_004a4910)
0x016f9aa0 | ?     | VpRect[] | ViewportRectTable      | ⭐ Split-screen viewport table. `0x016f9aa0 + idx*0x28` per viewport.
                                                          | idx=0..3 for up to 4 split-screen players.
0x016f9a60 | 8     | float*   | ScreenResPointer [alias]| Written by CameraEntityViewportUpdate: = player_cam_entity+0x100
0x0aea0c48 | 8     | void*    | ReplaySceneHandle      | Replay scene object pointer. Used by InitReplayCarSlot
0x0aea0c84 | 4     | int      | ReplayCarIndex         | Current replay car index into DAT_0ae8e988 array
0x0ae8e988 | ?     | int[]    | ReplayCarIdArray       | Array of car IDs for replay playback / race setup car IDs
0x0ae8e998 | ?     | int[]    | ReplayCarTypeArray     | Array of car types for replay / race setup
0x0ae8e928 | 4     | int      | SessionTrackIndex      | Track index for current race session
0x0ae8e92c | 4     | int      | SessionLapCount        | Lap count for session
0x0ae8e930 | 4     | int      | SessionMaxCarTarget    | Max AI fill target (FUN_00450700 fills up to this)
0x0ae8e934 | 4     | int      | SessionEventCarCount   | Number of cars in the race event participant list
0x0ae6f6ac | 4     | int      | PlayerCarUID           | Player's car unique identifier (compared against participant entries)
0x00f3d780 | ?     | blob     | RaceSaveStateTemplate  | 0x136 qwords (2472 bytes) — race config template, copied by cases 4/5
0x00f3eb00 | 4     | int      | RacePlayerIndex        | ⭐ Player's own race index (0-based) in participant table
0x00f3eb04 | 4     | int      | RaceParticipantCount   | ⭐ Live race participant count (max 0x1e = 30)
0x00f3eb08 | 8     | int64    | RaceModeAndCount       | High-dword = race mode tag; low-dword = count/flags
0x00f3eb10 | 4     | int      | RaceLapCount           | Lap count for this race instance
0x00f3eb14 | 4     | ?        | RaceFlag_14            | Race config flag
0x00f3eb18 | 4     | ?        | RaceFlag_18            | Race config flag
0x00f3eb20 | 4     | int      | RaceShowLaps           | Whether to show lap count (bool int)
0x00f3eb24 | 4     | int      | RaceMaxCars            | Max cars for this race (from DAT_0ae8e930)
0x00f3eb28 | 4     | int      | RaceTimeParam          | Race time limit param
0x00f3eb2c | 4     | ?        | RaceEEFlag             | Race extended flag
0x00f3eb34 | 8     | ?        | RaceTrackFlags         | Track/session flags (CONCAT from DAT_006fb645, DAT_006e34dc)
0x00f3eb40 | ?     | TrackID  | RaceTrackID            | Track identifier for current race
0x00f3eb70 | 30×80 | Part[]   | RaceParticipantTable   | ⭐⭐ Race participant entries. Stride 0x50 (80 bytes) per entry.
                                                          | Max 30 entries. Written by FUN_0044f1f0 and FUN_0044f4f0.
                                                          | Part[i]+0x30 = car name string; Part[i]-0x20 = type (1=player,2=AI)
                                                          | Part[i]-0x18 = car UI info offset; Part[i]-0x14 = car index
                                                          | THIS IS THE RACE PARTICIPANT LIST — target for remove_car_from_race
0x00660800 | 4     | float    | PlayerCameraFOV        | Player camera FOV or zoom (set by CameraEntityInit player-slot path)
```

### Race Participant Entry Layout (stride 0x50 = 80 bytes, base `DAT_00f3eb70`)
> Confirmed from FUN_0044f1f0 / FUN_0044f4f0 analysis. This is SETUP metadata only — NOT runtime physics.
> The participant table is READ at race-start to build actual runtime car slots.
```
Header arrays (parallel, NOT the 0x50 table):
  DAT_00f3eb14 + i*0x14   = CONCAT44(param_2=flags, param_1=type)     8 bytes
  DAT_00f3eb30 + i*0x28   = CONCAT44(param_4=skinIdx, param_3=carIdx) 8 bytes
  DAT_00f3eb60 + i*0x50   = 0 (reserved)                              4 bytes
  DAT_00f3eb64 + i*0x50   = param_5 (slot param)                      4 bytes
  DAT_00f3eb68 + i*0x50   = param_6 (slot param)                      4 bytes
  DAT_00f3eb6c + i*0x50   = 0 (reserved)                              4 bytes

Participant data block: DAT_00f3eb70 + i*0x50 (80 bytes):
  +0x00..+0x4F  = 0x14 dwords (80 bytes) copied from DAT_006fab50 + carIdx*0x110
                  (i.e. the CarInfo struct: name, class, stats etc.)
  +0x14..+0x23  = skin/design name   (12 bytes, from car skin table at carInfo+0xd8 + skinIdx*0xc)
  +0x20..+0x2F  = player/AI name     (15 chars + null, from param_7)
  +0x2F         = null terminator

type codes (param_1):
  1 = player (also increments player-count in DAT_00f3eb08._0_4_)
  2 = AI / CPU
  3 = spectator/fill
```

### Runtime Car Slot Layout (int*, 0xAAC = 2732 bytes)
> ✅ CONFIRMED: 0xAAC-byte structs are initialized by FUN_004ab810 (InitCarSlotFull) and
> template-copied by FUN_0043c580 (InitRuntimeCarStruct). The camera entity array at
> 0x016f8ee0 (stride 0x140) is separate — that is the CAMERA ENTITY SYSTEM, not cars.
> The flat car array location is still unknown; find it via FUN_0044f1f0 / FUN_0044f4f0.
```
int* slot:
  slot[0]    byte+0x00  = car array index
  slot[1]    byte+0x04  = ⭐ CAR TYPE: 1=player, 9=spectator/disabled, 4=special
  slot+0x12  byte+0x48  = packed car definition IDs (param2 | param3<<32)
  slot+0x24  byte+0x90  = pointer to physics/model sub-struct
  slot+0x32  byte+0xC8  = pointer to track/wheel sub-struct
  slot[0x28] byte+0xA0  = physics param group 1
  slot[0x2a] byte+0xA8  = physics param group 2
  slot+0x2d  16 bytes   = physics XMM data (quaternion?)
  slot[0x3c6..0x3c9]    = position offsets
  slot[0x3e0..0x3e3]    = bounds (-4.0, -4.0 = off-screen defaults)
  slot[0x3e9]           = 0x3E4CCCCD = float 0.2 (likely grip default)
  slot[0x400..0x404]    = zero-init zone
  slot[0x416]           = top speed (int, param-derived)
  slot[0x417]           = acceleration (int, param-derived)
  slot[0x418]           = weight (int, param-derived)
  slot[0x419]           = stat 1
  slot[0x41a]           = stat 2
  slot[0x41b]           = stat 3
  slot[0x41c]           = stat 4
  slot[0x41d]           = int param
  slot[0x41e]           = uint16 param
  slot[0x41f]           = uint16 param
  slot[0x422]           = AI control state byte
  slot[0x423]           = spectate/disable flag (byte)
  slot[0x6A80/0x107e/0x1056] = game-mode flags
  slot+0xF34 (slot[0x3CD]) = behaviour state (4 bytes)
  slot+0x40  = ptr to sub-object (SetCarBehaviourState: lVar2 = *(param_1+0x40))
  sub+0x340  = behaviour fn pointer (set by SetCarBehaviourState)
  sub+0x308  = behaviour mode int


---

**Next Step:** In Ghidra navigate to `0x0044530c` → find parent function → export it. That function calls `InitCarSlotFull` 3 times and is the **race car slot builder** — it will reveal the runtime car array base address needed for `remove_car_from_race`. Also export `FUN_0044cb60` (0x0044cb60) and `FUN_004f03a0` (0x004f03a0), other callers of `InitCarSlotFull`.

---

## 🆕 Batch 5 — Main Loop, Audio, Race State Machine

### ALSourceEntry (stride 0x18 = 24 bytes)
> Pool of up to 0x80 (128) AL sources stored at `DAT_0ae4c640`. Each entry tracks
> the playing sound's priority, SFX index, play timestamp, and AL source handle.
```c
struct ALSourceEntry {               // stride 0x18 (6×int = 24 bytes)
    int   priority;                  // +0x00  priority score (lower = preemptible)
    int   sfx_id;                    // +0x04  index into SFX buffer array
    int64 play_timestamp;            // +0x08  DAT_0ae48df0 value when started (perf ticks)
    uint  al_source_id;              // +0x10  OpenAL source handle (from alGenSources)
    uint  pad;                       // +0x14
};
// Base array: DAT_0ae4c640 (ALSourceEntry[0x80])
// Entry[i].al_source_id = DAT_0ae4c650[i*6] (at +0x10 from base of each entry)
```

### LoopSfxHandle (stride 0x38 = 56 bytes)
> Persistent looping SFX tracking array. Up to 0x100 (256) concurrent loops possible.
> Managed by `FUN_005282d0` (StartLoopingSfx) and `FUN_005284b0` (ChangeSfxAndStop).
```c
struct LoopSfxHandle {   // stride 0x38 (56 bytes)
    byte          active;            // +0x00  1=slot in use
    byte          sfx_id_lo;         // +0x01  sfx ID low byte
    int           sfx_id;            // +0x00  sfx ID (int, overlaps with active byte via CONCAT)
    int           priority;          // +0x04  (= param_2 priority)
    byte          loop_type;         // +0x08  looping type flag (param_3)
    void*         al_src_entry;      // +0x10  ptr into ALSourceEntry pool when playing
    float3        position;          // +0x18  3D position (x,y,z)
    int           loop_count;        // +0x24  times looped (for one-shot loops)
    float         volume;            // +0x28  (0..1.0, param_5 / 10000.0; but actually 0x54=1.0 float init)
    int           extra1;            // +0x2c
    int           extra2;            // +0x30
    int           extra3;            // +0x34
};
// Array at DAT_0ae48e50, max 0x100 entries
```

### Main Game Loop Globals (Batch 5)
> The entire game runs on a `while (!QuitRequested)` loop in `FUN_00402ec0` (= `WinMain`/`SDL_main`).
> One function pointer dispatch selects the per-frame game state handler.
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x006e30a0 | 8     | code*   | GameStateFuncPtr         | ⭐ Current per-frame handler; called every frame
0x006e30a8 | 1     | byte    | QuitRequestedFlag        | ⭐ 1 = SDL_QUIT received → loop exits
0x006e30a9 | 1     | byte    | F8_KeyFlag               | F8 was pressed (scancode 0x41)
0x006e30c0 | 256   | char[]  | TextInputBuffer          | SDL_TEXTINPUT text (max 256 chars)
```

Known `GameStateFuncPtr` values:
```
LAB_00499b30   Main racing game loop (per-frame race logic + rendering)
LAB_00543890   Multiplayer lobby / matchmaking
LAB_00406b40   Special editor/tool state (when -editor flag)
FUN_0044c130   another game state handler (when DAT_006fcec8 != 0)
LAB_00498c80   Race results screen (set after race_finished + state==6)
```

### Timer / Delta Time System (CalcDeltaTime = FUN_005250c0)
> `FUN_005250c0` is called every frame from the race loop to update delta time.
> `DAT_0065c03c` = the delta time float used by ALL physics code.
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x0ae48de0 | 8     | uint64  | PerfFrequency            | ⭐ SDL_GetPerformanceFrequency() result (ticks/second)
0x0ae48de8 | 8     | uint64  | LastPerfCounter          | Performance counter at previous frame
0x0ae48df0 | 8     | uint64  | CurrentPerfCounter       | Current SDL_GetPerformanceCounter() value
0x0ae48dd0 | 8     | uint64  | FrameTicks               | Raw FrameTicks = CurrentPerfCounter - LastPerfCounter (capped at 100ms)
0x0ae48dd8 | 8     | uint64  | FrameTicksRaw            | Uncapped raw frame ticks (for stats)
0x0065c038 | 4     | float   | RawDeltaTime             | Unscaled dt = FrameTicks / PerfFrequency (float)
0x0065c03c | 4     | float   | DeltaTime                | ⭐ Final dt (possibly slowmo-adjusted) — used by ALL update code
0x0ae48da0 | 4     | int     | GameSpeedPercent         | Speed multiplier % (default 100); slowmo = 20
0x0ae48e08 | 8     | uint64  | SlowmoTicksSaved         | Saved ticks from slowmo calculation
0x006e34b2 | 1     | byte    | SlowMoActiveFlag         | If set slows game by 20% additional
```

### Race State Machine (Batch 5)
> `FUN_0048a560(state)` = SetRaceState — updates `DAT_016942dc` with transition timer logic.
> `FUN_0048a5c0()` = GetRaceState — returns `DAT_016942dc`.
> `FUN_0048a5d0()` = UpdateRaceState — advances state blending per delta time.
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x016942dc | 4     | int     | RaceStateMachine         | ⭐ Current race state (0-6)
0x016942d8 | 4     | float   | RaceStateBlend           | State transition blend (0..1, driven by DeltaTime × 4.0)
```

State values for `RaceStateMachine`:
```
0  = Idle / inactive
1  = Starting (countdown or race in progress start)
2  = Fading in 
3  = Racing / going (post-start state, also RecordFinished)
4  = Finishing (fade still running)
5  = Paused (window unfocused)
6  = End-of-lap / race complete
```

Racing loop globals (`LAB_00499b30` / `FUN_00499681`):
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x016f7901 | 1     | byte    | RaceFinishedFlag         | ⭐ 1 = player has finished all laps
0x016f7900 | 1     | byte    | RaceModeMPFlag           | 1 = multiplayer race (affects next state)
0x016f7904 | 4     | int     | TotalLaps                | ⭐ Total lap count for this race
0x016f7908 | 4     | int     | CurrentLap               | ⭐ Current lap number (0-indexed)
0x016f790c | 4     | float   | LapTimer                 | ⭐ Lap timer (seconds, += DeltaTime; reset on new lap)
0x016f9a60 | 8     | void*   | RaceViewportPtr          | ⭐ Pointer to viewport/camera struct (passed to FUN_004a4120)
0x006e3470 | 4     | float   | FocalLength              | Second param to FUN_004a4120 (near * FOV product)
```

### SDL Event Types (FUN_00402980 dispatch)
> `FUN_00402980()` = ProcessSDLEvents — called once per main loop iteration.
```
SDL Event type → Action:
  0x100 = SDL_QUIT             → DAT_006e30a8 = 1 (quit)
  0x200 = SDL_WINDOWEVENT      → sub-dispatch on event.event:
    event.event = 6  (SDL_WINDOWEVENT_SIZE_CHANGED)   → FUN_00539750 (resize)
    event.event = 12 (SDL_WINDOWEVENT_FOCUS_GAINED)   → FUN_00527da0 (ResumeAllSfx)
    event.event = 13 (SDL_WINDOWEVENT_FOCUS_LOST)     → FUN_00527cf0 (PauseAllSfx) + FUN_0052b380
  0x300 = SDL_KEYDOWN          → keyboard dispatch:
    scancode 0x44 (SDL_SCANCODE_F11) + no mod + not repeat → toggle fullscreen (DAT_0065c020 ^= 1)
    scancode 0x41 (SDL_SCANCODE_F8)  + no mod + not repeat → DAT_006e30a9 = 1
  0x303 = SDL_TEXTINPUT        → copy text to DAT_006e30c0
  0x403 = SDL_JOYBUTTONDOWN    → joystick button handler
  0x700 = SDL_FINGERDOWN       → touch finger slot register (up to 4 slots at DAT_0ae6dce0..dd30)
  0x701 = SDL_FINGERUP         → update touch slot (clear velocity)
  0x702 = SDL_FINGERMOTION     → update touch slot position+velocity
```

Joystick slot system (4 slots, stride 0x18 bytes each):
```
Slot i base: DAT_0ae6dce0 + i * 0x18:
  +0x00  int64   = finger ID (touch device ID)
  +0x08  float6  = velocity/position data (6 bytes per slot from CONCAT71)
  +0x10  uint16  = state flags (0x100 = just pressed, 0=released)
```
Joystick active flags: `DAT_0ae6dce8, DAT_0ae6dd00, DAT_0ae6dd18, DAT_0ae6dd30` (1 byte each)

### Audio System Globals (Batch 5)
> Audio uses OpenAL for SFX + a custom streaming system (OGG/WAV via FUN_00563640 etc.)
> for music. Up to 128 simultaneous AL sources with priority-based preemption.
```
Address    | Size   | Type    | Name                     | Description
-----------|--------|---------|--------------------------|-------------------------------------------
0x006e3434 | 1      | byte    | NoSoundFlag              | ⭐ 1 = audio disabled (-nosound cmdline)
0x0ae48e24 | 4      | int     | SfxMasterVolume          | SFX master volume (0-10000, default 10000)
0x0ae48e28 | 4      | int     | TotalALSources           | ⭐ Count of AL sources created (max 0x80 = 128)
0x0ae48e34 | 1      | byte    | AudioPausedFlag          | 1 = audio paused (on focus loss)
0x0ae48e40 | ?      | int[2]  | LoopSfxBaseInts          | Start of LoopSfxHandle[0] sfx_id + priority
0x0ae48e50 | 0x3800 | struct[]| LoopSfxArray             | ⭐ Array of LoopSfxHandle (stride 0x38, max 0x100 = 256)
0x0ae4c640 | 0xC00  | struct[]| ALSourcePool             | ⭐ Array of ALSourceEntry (stride 0x18, max 0x80 = 128)
0x0ae4c650 | 4      | uint    | ALSource0_ID             | First AL source ID (= ALSourcePool[0].al_source_id)
0x0ae4d240 | 0x400  | uint[]  | SfxBufferArray           | ⭐ OpenAL buffer IDs per SFX slot (0x100 int = 256)
0x0ae4da40 | 0x10000| char[]  | CustomSfxNameTable       | Custom SFX file names (stride 0x100 per slot, 0x100 slots)
0x0ae4db3f | 1      | byte    | CustomSfxNullTerms       | Null terminator bytes within custom SFX table
0x0ae5da40 | 256    | char[]  | CurrentMusicDir          | Current music directory path string
0x0ae5db40 | ?      | char[]  | MusicFileList            | Sorted music file names (stride 0x100, up to 0xFF entries)
0x0ae6db40 | 1      | byte    | MusicCycleFlag           | 1 = music was interrupted (DAT_0ae6db40)
0x0ae6db41 | 1      | byte    | MusicCustomFlag          | 1 = music from custom track (not redbook)
0x0ae6db44 | 4      | int     | MusicFileCount           | Number of music tracks found in current dir
0x0ae6db48 | 4      | uint    | MusicALSourceID          | ⭐ AL source handle for BGM/music stream
0x0ae6db50 | 8      | void*   | MusicStreamPtr           | ⭐ Active music stream object (OGG/WAV stream)
0x0ae6db58 | 4      | float   | MusicVolume              | Music volume (float, 1.0 = full)
0x0ae6db60 | 1      | byte    | MusicReadyState          | 1 = music source ready
0x0ae6db61 | 1      | byte    | MusicLoopFlag            | 1 = music looping
0x0ae6db62 | 1      | byte    | MusicIsPlaying           | 1 = currently playing
0x0ae6db64 | 4      | int     | MusicTrackIndex          | Current track index (cycles DAT_0ae6db68..0ae6db6c)
0x0ae6db68 | 4      | int     | MusicTrackStartIdx       | First track index (usually 0)
0x0ae6db6c | 4      | int     | MusicTrackEndIdx         | Last track index (= MusicFileCount)
0x00f4347c | 8      | u64     | AudioDeviceParams        | Packed device init params (freq + channel config)
```

Built-in SFX table pointer: `PTR_s_wavs_moto_wav_00661c40` — array of 0x6A (106) WAV path strings.
Music file format: `"redbook/track%02d.%s"` (e.g. `redbook/track02.ogg`).

### Rendering Pipeline Globals (Batch 9)

#### GL State Cache (prevents redundant driver calls)
```
Address     | Type | Name                     | Notes
------------|------|--------------------------|---------------------------------------------
0x0ae7fc4c  | byte | BatchingModeFlag          | 0=immediate draws, 1=batch mode
0x0ae7fc63  | byte | ShadowPassEnabled         | 1=shadow/UBO pass active
0x0ae7fc80  | u32  | CachedDepthFunc           | cached glDepthFunc arg (0x203=GL_LESS)
0x0ae7fc84  | u32  | CachedBlendSrc            | cached src blend factor
0x0ae7fc88  | u32  | CachedBlendDst            | cached dst blend factor
0x0ae7fc8c  | byte | DepthMaskState            | 0=off, 1=on (cached glDepthMask)
0x0ae7fc8d  | byte | DepthTestState            | 0=off, 1=on (cached GL_DEPTH_TEST)
0x0ae7fc8e  | byte | BlendEnabledState         | cached GL_BLEND enable state
0x0ae7fc90  | byte | BlendAlphaModeState       | 1=additive blend active
0x0ae7fc91  | byte | PremultAlphaState         | premultiplied alpha mode
0x0ae7fc92  | byte | DepthWriteActiveState     | depth write active
0x0ae7fc93  | byte | CullFaceState             | 1=GL_CULL_FACE enabled
0x0ae7fc9c  | i32  | CachedBoundTextureSlot    | -1=none, else slot index
0x0ae7fca8  | i32  | CachedBoundVAO            | last-bound VAO id (avoids rebind)
0x0ae7e9a8  | u32  | GLTextureBindCount        | frame texture bind call counter
0x0ae7e9ac  | u32  | GLStateChangeCount        | frame GL state change counter
0x0ae7e9bc  | byte | FrameRenderTag            | incremented each frame fence
```

#### 2D Sprite Batch Buffer
```
Address     | Type | Name                     | Notes
------------|------|--------------------------|---------------------------------------------
0x00f40dc0  | ptr  | SpriteBatchBufferPtr      | quad entry array, stride 0xb8 per quad
0x00f40dc8  | i32  | SpriteBatchCapacity       | max quad entries
0x00f40dca  | i32  | SpriteBatchCount          | current entry count
0x01694228  | i32  | RenderedQuadCount         | quads ready to flush
0x0169422c  | i32  | (padding)                 | -
0x01694230  | ptr  | QuadBatchArrayPtr         | per-frame quad list for FlushBatchedQuads
0x01694260  | u32  | ActiveVAOHandle           | current VAO binding handle
0x01694264  | u32  | ActiveVBOHandle           | current VBO binding handle
```

#### Track / Static Mesh Batch
```
Address     | Type | Name                     | Notes
------------|------|--------------------------|---------------------------------------------
0x018146a8  | i32  | TrackMeshObjectCount      | entries in track mesh array
0x018146b0  | ptr  | TrackMeshArrayPtr         | stride 0x2d8 per entry
0x018140a0  | ptr[] | MeshBucketTailPtrs[0xc1] | 193-slot texture bucket tail pointers
0x01813a80  | ptr[] | MeshBucketHeadPtrs[0xc1] | 193-slot texture bucket head pointers
0x01813a74  | u32  | ShadowMatrixHandle        | shadow map transform id
0x01813a78  | u32  | ShadowMatrixSlot          | shadow bind slot
```

#### Dynamic / Car Mesh Batch
```
Address     | Type | Name                     | Notes
------------|------|--------------------------|---------------------------------------------
0x0adce3ac  | i32  | DynMeshObjectCount        | entries in dynamic mesh array
0x0adce3a8  | i32  | DynMeshObjectCount2       | secondary count
0x0adce3c0  | ptr  | DynMeshArrayPtr           | stride 0x2d8 per entry
0x0adcdda0  | ptr[] | DynBucketTailPtrs[0xc1]  | 193-slot texture bucket tails
0x0adcd780  | ptr[] | DynBucketHeadPtrs[0xc1]  | 193-slot texture bucket heads
0x0adcd160  | ptr[] | DynBucketAltPtrs[0xc1]   | alt bucket array
0x0adccb40  | ptr[] | DynBucketExtra[0xc1]      | extra bucket array
0x0adccb00  | i32  | NetworkCarMeshCount       | count of network car mesh entries
0x0adccb08  | ptr  | NetworkCarMeshArrayPtr    | stride 0x1e0 per entry
0x0adccb18  | ptr  | NetworkCarNameArrayPtr    | per-car name buffer array
0x0adccb10  | ptr  | NetworkCarBufferPtr       | network car VBO buffer pool
0x0adccae0  | ptr  | NetworkCar2ArrayPtr       | secondary network car array
0x0adccae8  | i32  | NetworkCar2Count          | count of secondary network car entries
```

#### Game Object Render List
```
Address     | Type | Name                     | Notes
------------|------|--------------------------|---------------------------------------------
0x0ace47a8  | i32  | GameObjectRenderCount     | entries ready to draw (cleared by ClearGameObjectsCount)
0x0ace47b0  | ptr  | GameObjectRenderArrayPtr  | stride 0x2d8 per entry
0x0ace47a0  | u32  | GameObjShadowMatHandle    | shadow pass matrix
0x0ace47a4  | u32  | GameObjShadowSlot         | shadow bind slot
0x0ace47d0  | f32  | WorldBoundsXMin           | world AABB X min (OOB check in WorldStep)
0x0ace47d4  | f32  | WorldBoundsXMax           | world AABB X max
0x0ace47d8  | f32  | WorldBoundsYMin           | world AABB Y min
0x0ace47dc  | f32  | WorldBoundsYMax           | world AABB Y max
0x0ace47e0  | f32  | WorldBoundsZMin           | world AABB Z min
0x0ace47e4  | f32  | WorldBoundsZMax           | world AABB Z max
```

#### Screen / Viewport Globals
```
Address     | Type | Name                     | Notes
------------|------|--------------------------|---------------------------------------------
0x006e3468  | f32  | ViewportScaleX            | screen X scale (pixels / in-game unit)
0x006e346c  | f32  | ViewportScaleY            | screen Y scale
0x006e347c  | f32  | FogNear                   | near fog distance (SetFogDistances param_1)
0x006e3480  | f32  | FogFar                    | far fog distance (adjusted by DrawDist)
0x006e3484  | f32  | FogA                      | (near+far)/range
0x006e3488  | f32  | FogB                      | (2*near*far)/range
0x006e348c  | f32  | FogMid                    | fog midpoint
0x006e3490  | f32  | FogNearClip               | near fog clip
0x006e3494  | f32  | FogRange                  | far-near range
0x006e34a0  | f32  | FogLinearSlope            | 1/(far2-near2)
0x006e34a4  | i32  | DrawDistLevel             | 0-4 from config "DrawDist"
```

#### Shader Uniform Cache
```
Address     | Type | Name                     | Notes
------------|------|--------------------------|---------------------------------------------
0x0ae85e80  | f32[] | UniformBlockCache[16]    | stride 0xc per slot: (a, b, c) int3 per entry
0x0ae85e20  | i32  | MeshBatchStat0            | zeroed by ClearMeshBatchState
0x0ae85e28  | i32  | MeshBatchStat1            | zeroed by ClearMeshBatchState
0x0ae85e30  | i32  | MeshBatchStat2            | zeroed
0x0ae85e38  | i32  | MeshBatchStat3            | zeroed
0x0ae85e40  | i32  | MeshBatchStat4            | zeroed
0x0ae85e44  | i32  | MeshBatchStat5            | zeroed
0x0ae85e4c  | i32  | MeshBatchStat6            | zeroed
0x0ae85e50  | i32  | MeshBatchStat7            | zeroed
0x0ae85e54  | i32  | MeshBatchStat8            | zeroed
```

#### Frustum / View Matrix Globals (used by FrustumCullSphere)
```
Address     | Type | Name                     | Notes
------------|------|--------------------------|---------------------------------------------
0x016f98c0  | f32  | FrustumPlane0_X           | Left frustum plane normal X
0x016f98c4  | f32  | FrustumPlane0_Y           | Left frustum plane normal Y
0x016f98c8  | f32  | FrustumPlane0_Z           | Left frustum plane normal Z
0x016f98cc  | f32  | FrustumPlane0_D           | Left plane D (dot product constant)
0x016f98d0  | f32  | FrustumPlane1_X           | Right frustum plane
0x016f98d4..0x016f98dc | f32x4 | FrustumPlane1     | Right plane XYZD
0x016f98e0..0x016f98ec | f32x4 | FrustumPlane2     | Top plane XYZD
0x016f98f0..0x016f98fc | f32x4 | FrustumPlane3     | Bottom plane XYZD
0x016f99e8  | f32  | ViewMatrix_Row2_X         | View matrix Z-col X (depth projection)
0x016f99f4  | f32  | ViewMatrix_Row2_Y         | View matrix Z-col Y
0x016f9a00  | f32  | ViewMatrix_Row2_Z         | View matrix Z-col Z
0x016f9930  | f32  | ViewMatrix_Row2_W         | View matrix Z-col W (translation)
```

### RenderableObject Struct (stride 0x2d8 = 712 bytes) — Batch 9
> All render arrays (TrackMesh, DynamicMesh, GameObjects, CarMesh) use this layout.
```c
struct RenderableObject {  // stride 0x2d8
    // --- Shader / Batch identity ---
    u32    vao_handle;          // +0x00  VAO handle (matches CachedBoundVAO)
    u32    vbo_handle;          // +0x04  VBO/shader program handle
    u32    mesh_type;           // +0x08  mesh type index (0=static, 1=with lights, 2=special)
    u32    color_r;             // +0x08  material color R (reused for uniform block)
    u32    color_g;             // +0x0c  material color G
    u32    color_b;             // +0x10  material color B
    u64    mesh_info_ptr;       // +0x10  ptr to mesh resource (tex slot at +4)
    u32    world_mat_col0;      // +0x20  world matrix col 0 base
    u32    world_mat_col1;      // +0x24  world matrix col 1
    u32    world_mat_col2;      // +0x28  world matrix col 2
    u32    world_mat_col3;      // +0x2c  draw call index / triangle count
    i32    tri_face_count;      // +0x2b  triangle count (added to DAT_00f43a18)
    i32    draw_call_index;     // +0x2c  FUN_0053ab80 draw index
    i32    merge_primitive_count; // +0xac  consecutive same-mesh elements to merge
    // --- Flags ---
    byte   cull_disabled;       // +0x2d  0=cull enabled, 1=two-sided
    byte   in_range;            // +0x2e  skidmark depth-in-range flag
    byte   blend_mode;          // +0x30  0=src_alpha, 1=additive, 2=premultiplied
    byte   vertex_color_flag;   // +0x9f  1=use vertex color override
    byte   shadow_mesh_flag;    // +0xb5  1=env map flag
    byte   two_sided_flag;      // +0xb7  1=two-sided/no-cull for transparent
    byte   skidmark_flag;       // +0xb8  1=decal/skidmark (depth tested)
    byte   skip_pass2_flag;     // +0xb9  1=skip second transparent pass
    byte   vertex_color_a;      // +0xa0  vertex alpha override byte
    u8     vertex_color_r;      // +0x27d vertex color R override
    u8     vertex_color_g;      // +0x27e vertex color G override
    u8     vertex_color_b;      // +0x27f vertex color B override
    // --- Texture slots ---
    i32    tex_slot;            // +0x5c  primary texture slot ID (-1=unassigned)
    u32    tex_flags;           // +0x60  texture flags (bit0=shadow, bit1=env)
    i32    tex_slot_transp;     // +0x64  transparent variant slot
    u32    tex_flags_transp;    // +0x68  transparent slot flags
    u32    tex_flags_shadow;    // +0x6c  shadow pass slot flags
    i32    secondary_tex_slot;  // +0xd4  env/lightmap texture slot
    // --- Spatial / visibility ---
    i32    visibility_flag;     // +0xc0  -1=visible, else=culled/occluded
    float  depth_z;             // +0xc4  Z depth (for sort in transparent pass)
    // --- Links ---
    void*  next_batch_ptr;      // +0xb4  next same-texture-bucket ptr (linked list)
    void*  next_obj_ptr;        // +0x2d0 sorting temp link ptr
};
```

### Semi-Transparent Render List (Batch 9)
```
Address     | Type | Name                      | Notes
------------|------|---------------------------|---------------------------------------------
0x01694218  | i32  | SemiTransparentCount      | current entries in list
0x0169421c  | i32  | SemiTransparentCapacity   | max entries (starts 0x1000)
0x01694220  | ptr* | SemiTransparentListPtr    | ptr-to-ptr array (realloc on overflow)
```

### Car Mesh Render Globals (Batch 9)
```
Address     | Type | Name                      | Notes
------------|------|---------------------------|---------------------------------------------
0x01694238  | i32  | CarMeshObjectCount        | # of car mesh entries to render
0x01694240  | ptr  | CarMeshArrayPtr           | stride 0x2d8 per RenderableObject
0x016942b8  | u32  | SharedCarVAO              | shared VAO for all car meshes
0x016942bc  | u32  | SharedCarVBO              | shared VBO/program for all car meshes
```

### Ghost Car Render Globals (Batch 9)
```
Address     | Type | Name                      | Notes
------------|------|---------------------------|---------------------------------------------
0x016942a0  | ptr  | GhostEntriesArrayPtr      | stride 0x180 per ghost entry
0x016942a8  | i32  | GhostEntriesCapacity      | max ghost entries (doubles on overflow)
0x016942ac  | i32  | GhostEntriesCount         | active ghost count
0x016942b0  | i32  | GhostReservations         | (reservation count)
0x016942b4  | i32  | GhostReservations2        | (secondary)
0x016942b8  | u32  | [shared with CarVAO]      | see above
0x016942c0  | ptr  | GhostRenderPacketArrayPtr | stride 0x38 per packet
0x016942c8  | i32  | GhostRenderPacketCount    | packets built by UpdateGhostRenderEntries
0x016942d0  | ptr* | GhostSortBufferPtr        | temp sort buffer (ptrs)
```

```c
struct GhostCarEntry {  // stride 0x180, at DAT_016942a0
    // ... unknown ...
    int    ghost_player_idx;    // +0x164
    int    ghost_anim_state;    // +0x168
    int    ghost_color_idx;     // +0x16c
    int    ghost_priority;      // +0x170  0=normal, 1=secondary
    float  ghost_depth_z;       // +0x174  0.0=inactive OR at origin
    void*  render_packet_link;  // +0x178  ptr to GhostRenderPacket
};

struct GhostRenderPacket {  // stride 0x38, at DAT_016942c0
    // ... unknown +0x00..+0x18 ...
    int    player_idx;          // +0x1c
    int    anim_state;          // +0x20
    float  depth_z;             // +0x24
    // ... +0x28..+0x37 ...
};
```

### Performance / Frame Stats (Batch 9)
```
Address     | Type | Name                      | Notes
------------|------|---------------------------|---------------------------------------------
0x00f43a00  | i32  | FrameRenderResetFlag      | set 0 at start of RenderSceneFull
0x00f43a18  | i32  | TriangleCountAcculator    | incremented by tri_count/3 per transparent draw
0x0065c0f8  | i32  | TriangleCountDisableFlag  | 0=count, 1=skip
0x00676384  | i32[] | ShellSortGapTable        | Ciura/classic gap sequence for object sort
0x006763a0  | (end)| ShellSortGapTableEnd      | sentinel address, loop terminator
```

### Command-Line Flags Summary (Batch 5 consolidated)
> All parsed in `FUN_00402ec0` (= main entry point) argument loop.
```
Flag            | Address    | Effect
----------------|------------|-----------------------------------------------------
-nosound        | 006e3434   | Disable all audio (NoSoundFlag = 1)
-nojoy          | 006e3433   | Disable joystick/controller input
-carinfo        | (takes arg) | Load specific car info file
-window [W H]   | 0065c020=0 | Windowed mode; DAT_006e3414=W, DAT_006e3410=H
-multisample N  | 006e340c   | Set MSAA sample count
-nogamma        | 006e342d   | Disable gamma correction
-cubevisi       | 00661640=0 | Disable cube visibility (hidden rooms)
-editscale F    | 0065c024   | Editor scale factor (float)
-editoffset V   | (takes arg) | Editor position offset
-prefpath PATH  | 006e343c   | Set preferences/data path; DAT_006e3200=path
-packlist FILE  | 006e30e0   | Asset pack list file
-largereplays   | 006e3438   | Enable larger replay buffer
```

### Panel 2D Batch System — CPU Buffers (Batch 10)
> Used by DrawTexturedQuad when batch mode (`DAT_0ae7fc4c != 0`) is active.
```
Address     | Type | Name                      | Notes
------------|------|---------------------------|-------------------------------------------
0x00f40dc0  | ptr  | PanelBatchCPUBuf          | malloc(0x5c00) = 128 quads * 0xb8 bytes each
0x00f40dc8  | i32  | PanelBatchCapacity        | init 0x80 (128 quads)
0x00f40dca  | i32  | PanelBatchCount           | current # of queued quads (reset by flush)
0x00f40dcc  | u32  | PanelVBO                  | GL VBO handle (vertex buffer)
0x00f40dd0  | u32  | PanelEBO                  | GL EBO handle (index buffer)
0x00f40dd8  | ptr  | PanelGroupBuf             | malloc(0x1c00) — unique (texId,blend) groups, stride 0x38
0x00f40de0  | i16  | PanelGroupCount           | # of unique texture/blend groups in current batch
```
Panel group struct (stride 0x38):
```
+0x04 = texId (i32)      — GL texture index into DAT_016cf7b8 array
+0x20 = blendMode (i32)  — 0=alpha-blend, 1=additive, -1=no-state-change
+0x30 = VBO_handle (u32) — set by UploadVBOData after GPU upload
+0x34 = draw_count (i32) — # of indices for this group's glDrawElements call
```

### Text 2D Batch System — CPU Buffers (Batch 10)
> Used by text rendering path (FUN_00491be0 / FUN_00491900).
```
Address     | Type | Name                      | Notes
------------|------|---------------------------|-------------------------------------------
0x016cf560  | ptr  | TextBatchCPUBuf           | malloc(0xb0000) = 4096 quads * 4 verts * 0x2c stride
0x016cf568  | i32  | TextBatchCapacity         | 0x1000 = 4096 quads max
0x016cf56a  | i16  | TextBatchCount            | current # of queued text quads
0x016cf56c  | u32  | TextVBO                   | GL VBO handle
0x016cf570  | u32  | TextEBO                   | GL EBO handle
0x016cf5a8  | u64  | TextBatch_VBO_DrawCount   | CONCAT44(VBO_handle, index_count) packed result from UploadVBOData
0x016cf5ac  | i32  | TextBatchDrawCount        | # of draw calls; checked at start of DrawBatchedTextPolys
```

### Shared Vertex/Index Staging Buffers (Batch 10)
> Shared between ALL batch systems (panel quads, text quads, mesh).
```
Address     | Type | Name                      | Notes
------------|------|---------------------------|-------------------------------------------
0x0ae7e988  | ptr  | StagingVtxBuf             | heap vertex staging buffer (cpu-side upload buffer)
0x0ae7e990  | ptr  | StagingIdxBuf             | heap index staging buffer (shared)
0x0ae7e998  | ptr  | StagingVtxBuf2            | secondary vertex staging — used by 2D/panel paths
0x0ae7e9a0  | u32  | FogColorPacked            | fog color RGBA packed — decoded to float R,G,B in SetupFrameUBOs
0x0ae7e9a8  | i32  | ShaderStateChangeCount    | incremented on every glBlendFunci/glBlendEquation call
0x0ae7e9ac  | i32  | GLStateChangeCount        | incremented on every GL state change (blend/depth/cull/tex)
0x0ae7e9bc  | i32  | FrameCommitFlag           | set to 1 by CommitFrameRenderWork, guards re-upload
```

### GL Program / UBO / State Cache (Batch 10)
```
Address     | Type | Name                      | Notes
------------|------|---------------------------|-------------------------------------------
0x0ae7fc4c  | i8   | BatchModeEnabled          | 0=immediate draw, nonzero=batch all calls
0x0ae7fc52  | i8   | PremulAlphaFlag           | premultiplied-alpha content mode
0x0ae7fc5a  | i8   | SeparateBlendFuncFlag     | 0=glBlendFunc, 1=glBlendFunci per draw buffer
0x0ae7fc5b  | i8   | BigEndianColorFlag        | 0=swap R/B bytes in DrawTexturedQuad, 1=RGBA as-is
0x0ae7fc63  | i8   | ShadowPassActiveFlag      | nonzero = shadow map render pass active
0x0ae7fc64  | i8   | SSOEnabled                | Separate Shader Objects (glProgramUniform* path)
0x0ae7fc68  | i8   | Use32BitIndices           | 0=GL_UNSIGNED_SHORT, 1=GL_UNSIGNED_INT for glDrawElements
0x0ae7fc80  | u32  | CachedBlendSrc            | cached glBlendFunc src factor
0x0ae7fc84  | u32  | CachedBlendDst            | cached glBlendFunc dst factor (stored as pair at fc84)
0x0ae7fc8e  | i8   | CachedAlphaTestEnabled    | GL 0xbc0 enable/disable state cache
0x0ae7fc90  | i8   | CachedBlendEnabled        | GL 0xbe2 (GL_BLEND) enable/disable state cache
0x0ae7fc91  | i8   | CachedAlphaBlendMode      | 0=SRC_ALPHA/ONE_MINUS, 1=additive blend
0x0ae7fc92  | i8   | SSOActiveFlag             | separate shader objects currently bound
0x0ae7fc9c  | i32  | CachedTexture0            | last bound texture on unit 0xde1 (GL_TEXTURE_2D)
0x0ae7fca8  | u32  | CachedVertexShader        | last bound vertex shader ID — avoids redundant glUseProgram
0x0ae7fcac  | u32  | CachedProgram             | last bound full program (used by SetupFrameUBOs)
```

### GL UBO / Mesh Uniform Buffer Management (Batch 10)
```
Address     | Type | Name                      | Notes
------------|------|---------------------------|-------------------------------------------
0x0ae85e08  | u32  | UBO_Handle_0              | primary UBO GL handle
0x0ae85e0c  | u32  | UBO_Handle_1              | secondary UBO GL handle
0x0ae85e20  | i32[8]| MeshUBOOffsets           | per-stream UBO byte offsets (array of 8, stride 4)
0x0ae85e28  | i32[4]| MeshUBOOffsets_2         | continuation of offset array
0x0ae85e40  | i8   | UBO_NeedsResize           | set when glBufferData needed before upload
0x0ae85e44  | i32  | UBO_UsageHint             | GL usage hint (0x88e0=GL_DYNAMIC_DRAW)
0x0ae85e48  | i32  | UBO_MaxSize               | allocated GPU buffer size in bytes
0x0ae85e4c  | i32  | UBO_CurrentSize           | grows to next power-of-2 as needed
0x0ae85e50  | i32  | UBO_HighWaterMark         | max byte offset reached this frame
0x0ae85e54  | i32  | UBO_WriteOffset           | current write cursor for sub-data uploads
0x0ae85e58  | ptr  | UBO_StagingPtr            | cpu staging buffer for UBO uploads
0x0ae85e80  | f32[] | PerFrameUBOData          | 0x18 f32 slots — projection/view/fog matrices
0x0ae85f40  | u32  | LastUploadedVBO           | VBO handle returned by last UploadVBOData
0x0ae85f44  | i32  | LastUploadedVBOSize       | byte size of last VBO upload
0x0ae85f48  | u32  | FBO_Handle                | main framebuffer object handle
0x0ae86180  | u32  | ProgramListHead           | first GL program in linked list (stride 0x80 per entry)
0x0ae7ede8  | i32[][] | StreamAttribTable      | per-stream vertex attrib config (stride 0x620 per stream, 8 streams, 49 slots each)
0x0ae7e9e8  | u32[] | GLResourceSlotTable      | slot flags table (stride 0x20/slot, searched by FindBestGLResourceSlot)
```

### GL Device Resources (Batch 10)
```
Address     | Type | Name                      | Notes
------------|------|---------------------------|-------------------------------------------
0x0ae85de0  | u32  | SDL_GL_Context0           | primary SDL GL context handle
0x0ae85de8  | u32  | SDL_GL_Context1           | secondary/worker GL context handle
0x0ae85df0  | ptr  | SDL_WindowHandle          | SDL2 window pointer
0x0ae8e184  | u32  | RBO_Color                 | GL renderbuffer (color attachment)
0x0ae8e188  | u32  | RBO_Depth                 | GL renderbuffer (depth attachment)
0x0ae8e18c  | u32  | FBO_Handle2               | second FBO (for MSAA resolve or postfx)
```


