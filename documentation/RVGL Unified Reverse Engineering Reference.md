# RVGL Unified Reverse Engineering Reference

Last Updated: 2026-06-07 (merged from RVGL Master Function Index and RVGL Memory Map)

## Contents
- Master Function Index material
- Memory map and data structure material
- Batch notes and queues

---

## Section 1: Master Function Index

# RVGL Master Function Index
**Last Updated:** March 2026 — Batch 11 / Printf Engine + BigNum + VFS + Progression  
**Total Functions Documented:** 310+

---

## Quick Navigation
- [RVGL.exe Functions](#rvglexe-functions) (42 functions)
- [libGLESv2.dll Functions](#libglesv2dll-functions) (25 functions)
- [Memory Addresses](#memory-addresses) (60+ addresses)
- [SDL2.dll Imports](#sdl2dll-imports)

---

## RVGL.exe Functions

### Core Game Loop
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x004014c0 | entry() | func_entry.txt | Analyzed | Program entry point |
| 0x00402ec0 | MainGameLoop() | maingameloop.txt | Analyzed | Main entry, command-line parsing, game loop |
| 0x00402980 | ProcessSDLEvents() | func_00401500.txt | Analyzed | SDL event processor (input handling) |
| 0x00401610 | FrameUpdateDispatcher() | RGameModeManagement/func_00401610.txt | Analyzed | Per-frame timing dispatch: calls delta-time, updates accumulator `DAT_0ae48dc8`, iterates linked list at `DAT_0acee9e8` (stride `lVar3+0x10`) adding delta to `+0xf90`. Handles game-mode branch via `DAT_006e34c0 & 0xFFFFFFFD == 4` |

### Game State Functions
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00499b30 | SplashScreenState() | gameplay/lab_00499b30.txt | Analyzed | Splash/logo screen, transitions to racing |
| 0x00499690 | **RacingGameLoop()** | **[NOT EXPORTED YET]** | **** | **Main racing loop - physics, AI, rendering!** |
| 0x00543890 | MultiplayerLobbyState() | gameplay/lab_00543890.txt | Partial | Multiplayer/lobby mode |
| 0x00406b40 | UnknownGameState() | gameplay/lab_00406b40.txt | Partial | Another game mode (menu?) |
| 0x0044c130 | GameGaugeReplayState() | RGameModeManagement/func_0044c130.txt | Analyzed | Loads `game_gauge_replay.rpl`, sets `DAT_0ae8e906=1`. Copies `DAT_00f3e140` block (0x135 qwords) to `DAT_00f3eb00`. Sets game mode `DAT_0ae8e920=5`, `DAT_006e34c0=5`. Reads track index from `FUN_00452780`. Sets game-state fn ptr `DAT_006e30a0 = &LAB_00406b40` |

### Car System
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x0043f140 | LoadAllCars() | game_initialization/func_0043f140.txt | Analyzed | Main car loading loop (49 cars) |
| 0x0043b6c0 | ParseParametersTxt() | game_initialization/func_0043b6c0.txt | Analyzed | Parse car parameters.txt (16 keywords) |
| 0x005bd590 | InitializeCarStructure() | game_initialization/func_005bd590.txt | Analyzed | Set default car values |
| 0x0043ba50 | FinalizeCarLoad() | [NOT EXPORTED] | Not Analyzed | Post-processing after car load |

### Track System
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00452190 | LoadAllTracks() | game_initialization/func_00452190.txt | Analyzed | Main track scanning loop |
| 0x00451da0 | ParseTrackInf() | game_initialization/func_00451da0.txt | Analyzed | Parse track .inf file (6 keywords) |
| 0x005370e0 | CheckDirectoryExists() | [NOT EXPORTED] | Not Analyzed | Verify levels/[name]/ exists |
| 0x005370a0 | CheckFileExists() | [NOT EXPORTED] | Not Analyzed | Verify .inf file exists |

### Initialization Functions
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00536bb0 | InitSubsystem1() | game_initialization/func_00536bb0.txt | Analyzed | Unknown initialization |
| 0x00529310 | InitSubsystem2() | game_initialization/func_00529310.txt | Analyzed | Unknown initialization |
| 0x004ee170 | InitSubsystem3() | game_initialization/func_004ee170.txt | Analyzed | Unknown initialization |
| 0x0044bae0 | InitSubsystem4() | game_initialization/func_0044bae0.txt | Analyzed | Major subsystem init |
| 0x00452280 | PostCarLoadInit() | [NOT EXPORTED] | Not Analyzed | Called after cars loaded |
| 0x0043fac0 | LoadUserCars() | RuntimeCarArray1/func_0043fac0.txt | Analyzed | Scans `cars/` dir via `_wfindfirst64`, `realloc`s `DAT_006fab50` to grow car pool beyond 49. Sorts entries by class rating. `DAT_006e3431` = skip-user-cars flag |
| 0x0044bb80 | UnknownInit2() | [NOT EXPORTED] | Not Analyzed | Called during setup |

### Race Initialization
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x004995f0 | RaceSetup() | race_initialization/func_004995f0.txt | Analyzed | Called before racing loop — loads GFX for current lap |
| 0x004889c0 | RaceInit1() | race_initialization/func_004889c0.txt | Analyzed | Race initialization function |
| 0x0048a560 | RaceInit2() | race_initialization/func_0048a560.txt | Analyzed | Race initialization function |

### AI & Physics System
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00408a60 | CarPhysicsUpdate_AIBrain() | [in rvgl.exe.c line 4912] | Analyzed | **THE AI brain (1517 lines)**. Per-car physics + 13-state AI state machine. Reads/writes 80+ CarEntity fields. Computes steering/throttle output at `+0x34/+0x36`. Updates race position. Calls avoidance, rubber-band, stuck detection. |
| 0x004071c0 | AI_Avoidance() | [in rvgl.exe.c line 4021] | Analyzed | Core steering computation from `+0x6924` (abs_steer) and `+0x6934` (lateral deviation). Manages reverse state byte `+0x69d0`, sets timer 4.0f. `DAT_006e343f` = global reverse test flag. |
| 0x00407400 | InitAI_State() | [in rvgl.exe.c line ~4080] | Analyzed | Initialize AI state on car spawn. Resets AI fields, calls `FUN_0040ff20` + `FUN_0040e150`. `DAT_006e3ae4` = initial waypoint index. |
| 0x004074a0 | ResetAIBehavior() | [in rvgl.exe.c line 4124] | Analyzed | Reset all AI nav vars to defaults. lookahead=600f, max_steer=50f, angle=100f, height=150f, track_room=300f, engine_power=0.5f. |
| 0x004075f0 | UpdateRacePositions() | [in rvgl.exe.c line 4169] | Analyzed | Insertion-sort all cars by race score. Score = `(lap * DAT_006e3ae0) - track_t`. Updates `DAT_006e39c0[]` sorted array, sets `car+0x69c0` = rank. |
| 0x00407930 | AI_StuckDetection() | [in rvgl.exe.c line 4322] | Analyzed | Detect stuck/stranded cars. Uses rubber-band config table at `PTR_DAT_0065c100`. Sets `car+0x38 \|= 0x40` if stuck. Forces AI state 3 if `stuck_slow_timer` exceeds threshold. |
| 0x00407cb0 | CompareCarPassing() | [in rvgl.exe.c line 4604] | Analyzed | Bool comparison for car overtaking. Reads path node types from `car+0x28/+0x30`. Returns whether target car is ahead of self. |
| 0x004c84a0 | PerFramePhysicsWorldStep() | [in rvgl.exe.c line 86109] | Analyzed | Main per-frame physics dispatcher. Calls UpdateRacePositions, decrements `car+0x104c` cooldown, calls `FUN_004f95b0`+`FUN_00488ad0`. Also updates game object linked list at `DAT_0ace4800`. |
| 0x004ce3b0 | CarSwapAttach() | [in rvgl.exe.c line ~86190] | Analyzed | Car entity swap/attach to physics body. |
| 0x004ce590 | UpdateCarGhostTransparency() | [in rvgl.exe.c line 86201] | Analyzed | Sets `DAT_0ace49a0 = 0.5f` ghost alpha. Per camera slot: sets `slot+0xc0 = 0.5f` fade alpha. |
| 0x004d0c70 | AnimationUpdate() | [in rvgl.exe.c line 86257] | Analyzed | **Per-object keyframe animation update** (NPC/pedestrian system). Reads animation state struct at `obj+0x338`. 16 bone channels (loop 0..0xf, stride 0x6c). Easing modes: linear/ease-in/ease-out/smoothstep/bounce. Loop modes: loop/play-once/ping-pong. |
| 0x004d1f20 | LoadAnimationData() | [in rvgl.exe.c line ~86850] | Analyzed | Zeros keyframe data tables, inits audio slot table to -1. Loads animation definitions from files into DAT_01815xxx arrays. Max 16 animations, 16 channels each. |
| 0x004b35b0 | MeshVertexOBBCull() | [in rvgl.exe.c line 75371] | Analyzed | Mark mesh vertices visible/occluded via OBB test. Vertex stride 0x58. Mesh: `+0x2c`=vert_count, `+0x58`=vert_buffer. Vis flag at vertex+0x54 (0=hidden, FLT_MIN=visible). |
| 0x004b3830 | MeshVertexOBBCull_Variant() | [in rvgl.exe.c line 75450] | Analyzed | Alternate version of MeshVertexOBBCull. |
| 0x00499c40 | GaussianElimination() | [in rvgl.exe.c line 63600] | Analyzed | Solves AxB=C linear system for physics constraints/suspension. Matrix stored as `[row*0x20+col]` float array. |
| 0x0049a3e0 | ConjugateGradientSolver() | [in rvgl.exe.c line ~64200] | Partial | Iterative SIMD-unrolled conjugate gradient solver. Used for physics/suspension. Large function. |

### Level Editor System
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00418900 | EditorModeDispatch() | [in rvgl.exe.c line 12838] | Analyzed | Dispatches to 10 editor sub-modes based on `DAT_006e3500`. |
| 0x00418a20 | EditModeUpdate() | [in rvgl.exe.c line 13002] | Analyzed | Main editor per-frame update. Calls sub-mode handlers, always calls `FUN_004931f0` (HUD text centering). |
| 0x00418d60 | ShowStatusMessage() | [in rvgl.exe.c line ~13100] | Analyzed | Display status message in editor mode. |
| 0x00418d90 | ScreenToWorldRay() | [in rvgl.exe.c line ~13120] | Analyzed | Convert screen-space mouse coords to world-space ray using view matrix. |
| 0x00418eb0 | WorldPointVisible() | [in rvgl.exe.c line ~13160] | Analyzed | Test if a world-space point is within the view frustum. |
| 0x00414c60 | AINodeBoxRender() | [in rvgl.exe.c line 10551] | Analyzed | Renders the AI node bounding box visualization in editor. |
| 0x004236b0 | WireframeBoundingBoxRender() | [in rvgl.exe.c line ~15800] | Analyzed | Draws wireframe bounding box for selected track section. |
| 0x0042e8c0 | TrackSectionDelete() | [in rvgl.exe.c line 19372] | Analyzed | Delete a track section in editor mode. Updates `DAT_006f9e6c` (track section count). |
| 0x004931f0 | RenderCenteredHUDText() | [in rvgl.exe.c — UNDECOMPILEABLE] | Timeout | Ghidra decompile timeout. Renders centered HUD text at X offset. Called by EditModeUpdate. |

### Car Entity System
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x004f03a0 | CreateCarEntity() | [in rvgl.exe.c line 93015] | Analyzed | **Creates car entity from free pool**. Args: (type, subtype, model, spawn_quat, pos_data, rot_matrix). Inserts into `DAT_0acee9e8` linked list. Sets `car+0x40`=physics_body, calls `FUN_004ab810`+`FUN_004a76f0`+`FUN_004efd50`+`FUN_00407400`+`FUN_004074a0`. Increments `_DAT_0acee7e4`. |
| 0x004f0630 | DestroyCarEntity() | [in rvgl.exe.c line 93123] | Analyzed | Frees car entity back to pool. Calls FUN_004a0ac0+FUN_004aa0b0+FUN_004ec470. Removes from linked list, returns to `DAT_0adb69b0`. Decrements `_DAT_0acee7e4`. |
| 0x004f06d0 | DestroyAllCarEntities() | [in rvgl.exe.c line 93145] | Analyzed | Destroys all cars in active linked list. Iterates `DAT_0acee9e8` → +0x10 chain. |
| 0x004f0850 | FindCarByNetworkID() | [in rvgl.exe.c line 93223] | Analyzed | Searches active car list for `car+0x6a84 == param1`. Returns car ptr or 0. O(n) walk of linked list. |
| 0x004f0890 | RegisterFinishTime() | [in rvgl.exe.c line 93239] | Analyzed | Record race finish: insertion-sort into `DAT_0acee800` leaderboard (max 30). Sets `car+0x6a48`=finish_time_ms, `car+0x6a4c`=position. Prints lap time (MM:SS:mmm format). |
| 0x004f0af0 | SetAllCarsBoostState() | [in rvgl.exe.c line 93348] | Analyzed | Sets `physics+0x340=LAB_005234f0` for all non-(type 0,4,9) cars. Used for boost/speed-start?. |
| 0x004f0b50 | RestoreAllCarsStartState() | [in rvgl.exe.c line 93370] | Analyzed | Restores `physics+0x340..+0x34c` from `physics+0x458..+0x464` (saved state rollback). |
| 0x004f0ba0 | SetCarInvincible() | [in rvgl.exe.c line 93408] | Analyzed | Sets `car+0x6a44=1`, clears `car+0x6a38..+0x6a40`. Makes car invincible. |
| 0x004f0bc0 | ClearCarInvincible() | [in rvgl.exe.c line 93415] | Analyzed | Sets `car+0x6a44=0`. Removes invincibility. |
| 0x004f1560 | DispatchTriggerHandlers() | [in rvgl.exe.c line 93517] | Analyzed | Iterates `DAT_0ace4800` list, calls fn ptr at `entity+0x358` for each entity. |
| 0x004f1910 | LoadTriggersFromFile() | [in rvgl.exe.c line 93543] | Analyzed | Reads trigger file: `DAT_0adb6cc8`=count, `DAT_0adb6cc0`=malloc(count×0x68). Each entry 0x68 bytes (active+file_data+dispatch_fn). |
| 0x0044cb60 | InitPlayerCar() | [in rvgl.exe.c line 33712] | Analyzed | Creates/reinitializes player's car. Uses `PTR_DAT_0065fca8` (params), calls `FUN_004f03a0(4,0,model,...)`. Sets `car+0x67d4=0`, `car+0x690d=1`, `car+0x6910=DAT_006e3ae4`, `car+0x6914=0`. Calls `FUN_0040ffb0`+`FUN_0040ff20`+`FUN_0040e150`. |
| 0x0044ce10 | ResetPlayerCarBuffer() | [in rvgl.exe.c line 33801] | Analyzed | Frees player car via `FUN_004f0630`. Clears params buffer. Sets `DAT_006fd140=0`, `DAT_006fd148=0`. |
| 0x0044cec0 | RecordGhostFrame() | [in rvgl.exe.c line 33855] | Analyzed | Records current car position/orientation into ghost replay buffer `PTR_DAT_0065fcb0`. Stores quaternion XYZ (`physics+0x14..+0x1c`) + color RGBA bytes + speed. Stride 0x18/frame. Max 0x7FFF frames. |
| 0x0044d040 | CommitGhostLapFrame() | [in rvgl.exe.c line 33934] | Analyzed | Swaps ghost double buffers: `PTR_DAT_0065fca8` ↔ `PTR_DAT_0065fcb0`. Updates `DAT_006fd150` (active ghost path). Gates on `DAT_00f42560` and lap count. |
| 0x0044d0c0 | RecordGhostPosition() | [in rvgl.exe.c line 33974] | Analyzed | Per-frame ghost frame recorder. Writes timestamp/speed/quaternion. SLERP test: skips frame if orientation < 0.6 and distance < 1000 from prev frame. |
| 0x004102d0 | CreateGhostPath() | [in rvgl.exe.c line 9203] | Analyzed | Builds ghost replay path from 0x1c20 car samples. Stores 3 floats (XYZ) per sample at `DAT_006e4320`. Sets `DAT_006e430c`=sample_count, `DAT_006e4310=1`. |
| 0x004442b0 | RenderScrollingText() | [in rvgl.exe.c line 28869] | Analyzed | Credits/title screen text scroll renderer. Uses `DAT_006fb6ec` (section), `ScreenResPointer`, calls `FUN_00492150` (DrawTextLine). Sections from `PTR_s_RE_VOLT_0065cf00` table. |

### Race Session Setup
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00450700 | RaceSetup() | [in rvgl.exe.c line 35952] | Analyzed | **CRITICAL: Main race session initializer**. Giant switch on `DAT_006e34c0` (GameMode). Case1=single, Case2=multi, Case3=practice, Case4/6=lobby. Copies race config, calls `FUN_0044f4f0`/`FUN_0044fb30`/`FUN_0044f7b0`. Sets up all global race state. |
| 0x0044f7b0 | SetupAllRaceCars() | [in rvgl.exe.c line 35314] | Analyzed | Creates ALL race cars from `DAT_00f3eb50` entry table. Loop: `FUN_004a71d0` (spawn orientation) → `FUN_004f03a0` (create car). Sets `DAT_0acee7e8` = local player's car when `is_local != 0`. Falls through to `FUN_0044f640` if player_slot = -1. |
| 0x0044f640 | CreateSinglePlayerCar() | [in rvgl.exe.c line 35225] | Analyzed | Creates single player car: type=9, subtype=7 (player behavior). Model from `DAT_006e34cc`. Network ID `DAT_0ae6f6ac` → `car+0x6a84`. Copies name to `car+0x6a70`. |
| 0x0044f4f0 | AddRaceEntry() | [in rvgl.exe.c line 35196] | Partial | Fills `RaceCarEntry` in `DAT_00f3eb50` table. Increments `_DAT_00f3eb04`. Args: (type, slot, model, variant, is_player, slot_num, name_ptr). |
| 0x0044f1f0 | SetupPlayerRaceEntry() | [in rvgl.exe.c ~35066] | Partial | Initializes player's race entry: model, start position, name. Called for case1 (single player). |
| 0x0044fb30 | RandomizeCarPicks() | [in rvgl.exe.c line 35447] | Analyzed | Builds per-class car pick tables (6 classes from `CarInfo+0xec`). Fisher-Yates shuffle via `FUN_00533360` (RandInt). Assigns cars to AI slots avoiding duplicates with player's 3 previous unique cars. |
| 0x004504e0 | SelectRandomCar() | [in rvgl.exe.c line 35845] | Analyzed | Picks random valid car from `CarModelDatabase` (skipping `+0xe5=0` unavailable/missing cars). Returns car model index. |
| 0x004505c0 | GetCarColor() | [in rvgl.exe.c line 35887] | Analyzed | Random colour variant for `param_1` car model. Uses `(short*)(model*0x110+DB+0xe0)` as upper range for `FUN_00533360`. Gates on `DAT_00f434a2`=1. |
| 0x00450600 | GetRandomTrack() | [in rvgl.exe.c line 35903] | Analyzed | Picks random available track. Iterates `DAT_006e34d0` tracks via `FUN_004548d0` (available check) + `FUN_004549b0` (select). Returns track index. |
| 0x004a71d0 | ComputeSpawnOrientation() | [called in FUN_0044f7b0] | Partial | Converts spawn type to rotation matrix. Args: (spawn_type, local[12], out_matrix[48]). |
| 0x0044f980 | AssignStartPositions() | [in rvgl.exe.c line 35314] | Analyzed | Assigns grid start positions to race entries using Fisher-Yates partial shuffle. Writes start pos indices to `DAT_00f3eb54` (offset +0x04 in each RaceCarEntry, stride 0x50). |

### Waypoint / Route Navigation
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x0040fc20 | FreeWaypointTables() | [in rvgl.exe.c line 8949] | Analyzed | `free(DAT_006e3b08); free(DAT_006e3b00)`. Frees both waypoint and zone tables. |
| 0x0040fd90 | FindCarInZoneSectionOBB() | [in rvgl.exe.c line 8959] | Analyzed | 3D OBB inside-test against all waypoints in zone section `param_2`. Writes `car+0x67dc`=section idx, `car+0x67d8`=waypoint data. Returns 0/1. |
| 0x0040ff20 | FindCarCurrentZone() | [in rvgl.exe.c line 9040] | Analyzed | Calls `FindCarInZoneSectionOBB` for current zone `car+0x67d4`, then tries ±1 zone (with modular wrap). Updates `car+0x67d4` on match. |
| 0x0040ffb0 | FindCarCurrentZoneBrute() | [in rvgl.exe.c line 9080] | Analyzed | Brute-force all `DAT_006e3b10` zones for OBB match. Sets `car+0x67d8=-1` if no table. Used on spawn. |
| 0x0040e150 | UpdateCarRoutePosition() | [in rvgl.exe.c line 8049] | Analyzed | **CORE: 14 XREFs**. Per-frame route tracker. OBB-tests `DAT_006e3b00` zones, if car inside zone: updates `car+0x6910` (route_section), `car+0x6914` (track_t, wraps by `DAT_006e3ae0`), `car+0x6918` (norm), `car+0x690c` (facing_dir). Updates `car+0x690c = dot(car_fwd, route_dir) > 0.6`. |
| 0x00410070 | DeleteWaypointNode() | [in rvgl.exe.c line 9162] | Analyzed | Removes node from `DAT_006f94a0` array (stride 0x108). Shifts remaining nodes. Clears references in `DAT_006e42f0/f8/4300`. |
| 0x00410260 | ShowErrorDialog() | [in rvgl.exe.c line 9200] | Analyzed | `sprintf → printf → MessageBox("Error", msg, 0)`. Uses FUN_00533540+00529490+00404720. |
| 0x00410500 | WaypointNodeLinkPatch() | [in rvgl.exe.c line 9336] | Analyzed | Fixes link fields at node+0x18,+0x20,+0x28,+0x30 during node deletion/insertion. |
| 0x00410680 | GetWaypointNodeClosestToRay() | [in rvgl.exe.c line 9416] | Analyzed | Editor ray-to-node distance test. Tests 3 points per node (L/R/center). Returns node ptr. Sets `DAT_006e3c6c` = sub-node indicator (0/1/3). |
| 0x00410860 | WaypointDijkstraRelax() | [in rvgl.exe.c line 9504] | Analyzed | Recursive Dijkstra shortest-path relaxation. Updates `node+0x0c` = distance cost. Uses `DAT_006e3c5c` (min_dist), `DAT_006e3c68` (start_node). |
| 0x004103e0 | FindNearestWaypointNode() | [in rvgl.exe.c line 9284] | Analyzed | Finds nearest waypoint node to float[3] position. Iterates all `DAT_006f94aa` nodes, stride 0x108, centroid `(+0x38..+0x50)`. Returns node index. |
| 0x00410020 | WaypointLinkPromote() | [in rvgl.exe.c line 9120] | Analyzed | Promotes link from `node+0x20` → `node+0x18` if `+0x18` is null. Same for `+0x30` → `+0x28`. |

### Input Processing
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x0052b250 | TouchInputHandler() | [NOT EXPORTED] | Not Analyzed | Multi-touch handler (4 fingers) |
| 0x00527da0 | WindowFocusGain() | [NOT EXPORTED] | Not Analyzed | Mouse button 0x0c handler |
| 0x00527cf0 | WindowFocusLoss() | [NOT EXPORTED] | Not Analyzed | Mouse button 0x0d handler |
| 0x005397c0 | DebugToggle() | [NOT EXPORTED] | Not Analyzed | 'D' key handler |
| 0x00539750 | FrequentUpdate() | [NOT EXPORTED] | Not Analyzed | Called often, unknown purpose |
| 0x00539b60 | DebugRelated() | [NOT EXPORTED] | Not Analyzed | Called with debug toggle |

### File I/O Helpers
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00533f60 | ReadNextToken() | [NOT EXPORTED] | Analyzed | Lexer/token reader — reads next whitespace-delimited token from config text stream. *(was ReadLineFromFile)* |
| 0x00533ef0 | CompareKeyword() | [NOT EXPORTED] | Not Analyzed | Case-insensitive string compare |
| 0x00534370 | ReadStringParameter() | [NOT EXPORTED] | Not Analyzed | Parse string value from file |
| 0x005343c0 | ReadByteParameter() | [NOT EXPORTED] | Not Analyzed | Parse 0/1 flag from file |
| 0x005345e0 | ReadFloatParameter() | [NOT EXPORTED] | Not Analyzed | Parse float from file |
| 0x005346e0 | ReadIntParameter() | [NOT EXPORTED] | Not Analyzed | Parse integer from file |
| 0x005334f0 | SafeStringCopy() | [NOT EXPORTED] | Not Analyzed | Copy string with bounds checking |
| 0x005335a0 | FormatPath() | [NOT EXPORTED] | Not Analyzed | sprintf for file paths | ---

## libGLESv2.dll Functions

### Context Management
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x67c77ae0 | **GetCurrentContext()** | func_67c77ae0.txt | Analyzed | **Main context getter (called by ALL 601 GL functions)** |
| 0x67f20810 | TlsGetValueWrapper() | func_67f20810.txt | Analyzed | Wraps Windows TlsGetValue |
| 0x67c779b0 | CreateContext() | func_67c779b0.txt | Analyzed | Allocates 32-byte handle, calls init |
| 0x67cec510 | ValidateContext() | func_67cec510.txt | Analyzed | Checks context_lost_flag, handles errors |
| 0x67cec310 | InitializeContextHandle() | func_ContextHandleInitializer.txt | Analyzed | Initializes 32-byte handle structure |
| 0x67c8a750 | IsContextLost() | func_ContextLostChecker.txt | Analyzed | Returns byte at context+0x39f1 |

### Core GL Functions
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x67c95040 | glDrawArrays_Impl() | func_glDrawArrays.txt | Analyzed | Draw call implementation |
| 0x67d5da10 | glDrawArrays_Validate() | func_glDrawArrays.txt | Analyzed | Validation before draw |
| [Similar] | glDrawElements() | func_glDrawElements.txt | Analyzed | Indexed draw call |
| 0x67c8bfd0 | glClear_Impl() | func_glClear.txt | Analyzed | Clear framebuffer |
| 0x67d5d600 | glClear_Validate() | func_glClear.txt | Analyzed | Clear validation |
| 0x67c8de90 | glUseProgram_Impl() | func_glUseProgram.txt | Analyzed | Bind shader program |
| 0x67d5d970 | glUseProgram_Validate() | func_glUseProgram.txt | Analyzed | Program validation |

### Texture Functions
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| [Address] | glBindTexture() | func_glBindTexture.txt | Analyzed | Bind texture to target |
| [Address] | glTexImage2D() | func_glTexImage2D.txt | Analyzed | Upload texture data |
| [Address] | glTexParameteri() | func_glTexParameteri.txt | Analyzed | Set texture parameters |
| [Address] | glGenTextures() | func_glGenTextures.txt | Analyzed | Generate texture IDs |

### Buffer Functions
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| [Address] | glBindBuffer() | func_glBindBuffer.txt | Analyzed | Bind buffer to target |
| [Address] | glBufferData() | func_glBufferData.txt | Analyzed | Upload buffer data |
| [Address] | glGenBuffers() | func_glGenBuffers.txt | Analyzed | Generate buffer IDs |

### Shader Functions
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| [Address] | glCreateShader() | func_glCreateShader.txt | Analyzed | Create shader object |
| [Address] | glCompileShader() | func_glCompileShader.txt | Analyzed | Compile shader source |
| [Address] | glCreateProgram() | func_glCreateProgram.txt | Analyzed | Create shader program |
| [Address] | glGetUniformLocation() | func_glGetUniformLocation.txt | Analyzed | Get uniform location |

### Other GL Functions
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| [Address] | glFlush() | func_glFlush.txt | Analyzed | Flush GL commands | ---

## Memory Addresses

### Game State & Control
| Address | Type | Name | Description | File Reference |
|---------|------|------|-------------|----------------|
| 0x006e30a0 | function* | GameStateFunction | Current game mode function pointer | ultra_priortity1/dat_006e30a0.txt |
| 0x006e30a8 | byte | GameQuitFlag | 1=quit, 0=running | - |
| 0x006e30a9 | byte | AKeyFlag | Set when 'A' pressed | - |

### Window & Display
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x0065c020 | int | DebugModeFlag | Toggled by 'D' key |
| 0x0065c021 | char | WindowMinimizedFlag | 0=minimized, 1=active |
| 0x0ae85df0 | void* | SDLWindowPointer | SDL window handle |
| 0x006e3414 | int | WindowWidth | From -window argument |
| 0x006e3410 | int | WindowHeight | From -window argument |
| 0x006e340c | int | MultisampleLevel | MSAA samples |

### Game Settings (Command-Line)
| Address | Type | Name | CLI Argument |
|---------|------|------|--------------|
| 0x006e3434 | byte | NoSoundFlag | -nosound |
| 0x006e3433 | byte | NoJoystickFlag | -nojoy |
| 0x006e342d | byte | NoGammaFlag | -nogamma |
| 0x006e3432 | byte | NoUserProfilesFlag | -nouser |
| 0x006e343a | byte | TextureInfoFlag | -texinfo |
| 0x006e3429 | byte | NoGUIKeysFlag | -noguikey |
| 0x006e3438 | byte | LargeReplaysFlag | -largereplays |
| 0x0065c024 | float | EditScale | -editscale |
| 0x006e31e8 | float | EditOffsetX | -editoffset |
| 0x006e31ec | float | EditOffsetY | -editoffset |
| 0x006e31f0 | float | EditOffsetZ | -editoffset |
| 0x00662a00 | float | AlphaReference | -alpharef |
| 0x00661640 | int | CubeVisibility | -cubevisi |

### Car System
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x006fab50 | CarInfo[49] | CarArray | Array of 49 car structures (272 bytes each) |
| 0x00f3f4c0 | int | StandardTrackCount | Number of standard tracks |
| 0x00f3f4c4 | int | OtherTrackCount | Other track type count |

### Threading
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x006e3088 | int | MainThreadID | SDL_ThreadID |
| 0x006e3090 | void* | MutexHandle | SDL_Mutex |
| 0x006e308c | int | LockFlag | Thread lock |
| 0x006e3988 | void* | LoadThreadHandle | Background loader |
| 0x006e3980 | void* | Semaphore1 | Sync primitive |
| 0x006e3978 | void* | Semaphore2 | Sync primitive |
| 0x006e3974 | byte | ThreadStopFlag | Stop background thread |
| 0x006e3976 | byte | ThreadErrorFlag | Thread error state |

### Timing
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x0ae48de0 | uint64 | PerformanceFrequency | QueryPerformanceFrequency |

### Touch Input (4 simultaneous touches!)
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x0ae6dce0 | int64 | Touch0FingerID | Touch 0 finger identifier |
| 0x0ae6dce8 | byte | Touch0Active | Touch 0 currently down |
| 0x0ae6dce9 | byte | Touch0Pressed | Touch 0 pressed this frame |
| 0x0ae6dcec | uint64 | Touch0PositionPressure | Position and pressure data |
| 0x0ae6dcf8 | int64 | Touch1FingerID | Touch 1 (+0x18 from Touch 0) |
| 0x0ae6dd00 | byte | Touch1Active | ... |
| 0x0ae6dd10 | int64 | Touch2FingerID | Touch 2 (+0x18 from Touch 1) |
| 0x0ae6dd28 | int64 | Touch3FingerID | Touch 3 (+0x18 from Touch 2) |

### Network
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x0ae6f6a0 | int | LobbyModeFlag | Lobby mode enabled |
| 0x0ae6f6a1 | byte | LobbyModeFlag2 | Secondary lobby flag |
| 0x0ae6f5a0 | char[256] | LobbyServerAddress | Server address string |
| 0x0065c030 | int | P2PEnabled | -nop2p flag (inverted) |
| 0x006625a2 | byte | MulticastEnabled | -nomulticast (inverted) |
| 0x006625a1 | byte | LateJoinEnabled | -nolatejoin (inverted) |
| 0x0065c02c | int | NetworkPort | -port |

### Profile
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x006635e0 | int | ProfileLoadedFlag | Profile loaded |
| 0x006e3640 | char[16] | ProfileName | From -profile argument |

### libGLESv2.dll Memory
| Address | Type | Name | Description | File Reference |
|---------|------|------|-------------|----------------|
| 0x6814c020 | DWORD | TLS_INDEX | Windows TLS slot index (-1 = uninitialized) | dat_6814c020.txt | ---

## SDL2.dll Imports

### Functions Used by RVGL
| Import Name | Used For |
|-------------|----------| | SDL_Init | Initialize SDL subsystems | | SDL_CreateWindow | Create game window | | SDL_GL_CreateContext | Create OpenGL context | | SDL_GL_SetSwapInterval | VSync control | | SDL_GL_SwapWindow | Present frame (swap buffers) | | SDL_GL_GetProcAddress | Load GL function pointers | | SDL_PollEvent | Get next event from queue | | SDL_Delay | Sleep for milliseconds | | SDL_GetTicks | Get time in milliseconds | | SDL_ThreadID | Get current thread ID | | SDL_CreateMutex | Create mutex | | SDL_LockMutex | Lock mutex | | SDL_UnlockMutex | Unlock mutex | | SDL_CreateSemaphore | Create semaphore | | SDL_SemWait | Wait on semaphore | | SDL_SemPost | Signal semaphore | | SDL_CreateThread | Create background thread | ---

## File Organization

### RVGL.exe Structure
```
rvgl.exe/
 func_entry.txt                    (Entry point)
 maingameloop.txt                  (Main game loop)
 func_00401500.txt                 (Event processor)

 gameplay/                         (Game state functions)
    lab_00499b30.txt             (Splash screen)
    lab_00543890.txt             (Multiplayer)
    lab_00406b40.txt             (Unknown state)

 game_initialization/              (Loading & init)
    func_0043f140.txt            (Load all cars)
    func_0043b6c0.txt            (Parse parameters.txt)
    func_00452190.txt            (Load all tracks)
    func_00451da0.txt            (Parse track .inf)
    func_005bd590.txt            (Init car structure)
    func_00536bb0.txt            (Init subsystem 1)
    func_00529310.txt            (Init subsystem 2)
    func_004ee170.txt            (Init subsystem 3)
    func_0044bae0.txt            (Init subsystem 4)

 race_initialization/              (Race setup)
    func_004995f0.txt            (Race setup)
    func_004889c0.txt            (Race init 1)
    func_0048a560.txt            (Race init 2)

 ultra_priortity1/                 (Critical data)
    dat_006e30a0.txt             (Game state function pointer)

 sdl2.dll (pointers to external functions)/
    imports.txt                   (SDL2 imports)

 imports.txt                       (All imports summary)
```

### libGLESv2.dll Structure
```
libGLESv2.dll/
 dat_6814c020.txt                  (TLS index global)

 Context Management/
    func_67c77ae0.txt            (GetCurrentContext - HOT PATH!)
    func_67c779b0.txt            (CreateContext)
    func_67cec510.txt            (ValidateContext)
    func_67cec310.txt            (InitializeContextHandle)
    func_67f20810.txt            (TlsGetValueWrapper)
    func_ContextLostChecker.txt  (IsContextLost)

 Draw Functions/
    func_glDrawArrays.txt        (Draw call)
    func_glDrawElements.txt      (Indexed draw)

 State Functions/
    func_glClear.txt             (Clear buffers)
    func_glUseProgram.txt        (Bind shader)

 Texture Functions/
    func_glBindTexture.txt
    func_glTexImage2D.txt
    func_glTexParameteri.txt
    func_glGenTextures.txt

 Buffer Functions/
    func_glBindBuffer.txt
    func_glBufferData.txt
    func_glGenBuffers.txt

 Shader Functions/
    func_glCreateShader.txt
    func_glCompileShader.txt
    func_glCreateProgram.txt
    func_glGetUniformLocation.txt

 Other/
     func_glFlush.txt
```

---

## Analysis Status Summary

### Completion Overview
```
RVGL.exe:        80% complete  (up from 72% in Batch 10)
   Main loop and entry
   Event processing (SDL all event types)
   Car loading system (static pool, 49 cars)
   Track loading system
   File format parsers (parameters.txt, track .inf)
   Delta time / frame timer (CalcDeltaTime)
   GL state setup / rendering helpers
   Input: keyboard, mouse, controller (all 3)
   Screen fade / race-end state machine
   Game mode management (11+ modes)
   Item / pickup system
   FPS profiler
   AI Brain (FUN_00408a60 — 1517 lines COMPLETE)
   AI state machine (13 states, all transitions)
   AI avoidance / stuck detection / rubber band
   Race position sorting (UpdateRacePositions)
   AI node graph structure (0x108 bytes/node)
   AI node editor (8 sub-modes identified)
   Physics world step (PerFramePhysicsWorldStep)
   Ghost car transparency system
   Keyframe animation system (16 bone channels)
   Gaussian / conjugate gradient physics solvers
   Mesh vertex OBB culling
   Editor mode dispatch + sub-modes
   Custom Printf engine (InternalFormatEngine + all primitives)
   BigNum arbitrary-precision float-to-string engine (15 functions)
   Virtual File System (VFS) — path resolution + mount system
   Race result CSV logging (5 functions)
   Track unlock / progression system (6 functions)
   strlen / strncasecmp / ReadNextToken (custom CRT strings)
   Racing game LOOP BODY (func_00499690 — entry only)
   Runtime car array builder (FUN_0044****c — see Queue)
   Full audio system (AL function tables partially mapped)
   Full multiplayer packet handler (1013 lines, only partial)

libGLESv2.dll:   95% complete
   TLS system
   Context management
   All GL function patterns
   Validation systems
   Error handling
```

### Queue
*See updated Queue at the bottom of this file.*

---

## RuntimeCarArray1 — Analyzed Functions
> Static car pool lives at `*(DAT_006fab50)`. Stride = 0x110 per `CarInfo`. Count at `DAT_006fab58` (starts 0×31 = 49). Runtime car struct = 0xAAC bytes, template at `DAT_006fab80`.
> **All files now in `rvgl.exe/01_CarSystem/`**

| Address | Function Name | File | Status | Description |
|---------|--------------|------|--------|-------------|
| 0x0043c580 | InitRuntimeCarStruct() | 01_CarSystem/func_0043c580.txt | Analyzed | **Copies 0xAAC-byte template from `DAT_006fab80` into `param_1` (runtime car slot), then applies per-car-index overrides**. Special cases for indices 0x1b, 0x16, 0x21, 0x22. Physics params from table at `0x670fa4` (stride 0x28). Called from `FUN_004ab810` and `FUN_005589b0` |
| 0x004ab810 | InitCarSlotFull() | 01_CarSystem/func_004ab810.txt | Analyzed | **Full car slot initializer**. Takes `(int *slot, carIdx, carType)`. Stores car IDs at `slot+0x48`. Calls `FUN_0043c580` to init local template. Checks `DAT_0acee7e8==slot` to detect car-0 (player). Sets `slot[1]` (byte+4) = car type (`9`=spectator/inactive, `4`=special, `1`=player). Writes physics params to `slot[0x416..0x41c]`, model ptr at `slot+0x24`, track data ptr at `slot+0x32`. 572 lines. Called from `0x00445***` (race participant builder) and `FUN_005589b0` |
| 0x005589b0 | InitReplayCarSlot() | 01_CarSystem/func_005589b0.txt | Analyzed | Wrapper: if `DAT_0aea0c48` (replay scene handle) is valid and car index matches `(&DAT_0ae8e988)[DAT_0aea0c84]`, calls `InitCarSlotFull`. Otherwise just calls `FUN_0043c580` on a local. Uses `DAT_0aea0c84` as index, `DAT_0ae8e988/998` as car arrays |
| 0x0043c280 | UnknownCarFn() | 01_CarSystem/func_0043c280.txt | Empty export | File body empty — re-export needed |
| 0x0043c160 | FindCarIndexByName() | 01_CarSystem/func_0043c160.txt | Analyzed | Searches `DAT_006fab50 + i*0x110 + 0x14` (display name field). Returns car INDEX or 0x22 if not found. Called from `FUN_00479e50` |
| 0x0043bdf0 | UpdateCarSelectability() | 01_CarSystem/func_0043bdf0.txt | Analyzed | Iterates static pool (stride 0x110), sets `+0xE5` (selectableByPlayer flag). `DAT_0065c0f8` mode: 0=all selectable, 1=class filter, 2=first-8 only. Unlock check at `+0xF0` via `FUN_0044a250/…` |
| 0x004070e0 | CheckFileIntegrity() | 01_CarSystem/func_004070e0.txt | Analyzed | Binary file integrity checker. `.prm` files: reads last 8 bytes, checks magic `0x7830`. Other files: reads last 4 bytes. Returns bool |
| 0x0049c080 | PackPhysicsVectors() | 01_CarSystem/func_0049c080.txt | Analyzed | Packs 10 float params into `float[4][3]` array. Simple helper called 49× from LoadAllCars |
| 0x0043fac0 | LoadUserCars() | 01_CarSystem/func_0043fac0.txt | Analyzed | Scans `cars/` dir via `_wfindfirst64`, `realloc`s `DAT_006fab50`. Sorts by class rating. `DAT_006e3431` = skip-user-cars flag | | **`DAT_016f8e60`** | **WeaponArraySlot0Ptr** | **01_CarSystem/dat_016f8e60.txt** | Analyzed | ** 259 XREFs. CLEARED to 0 by `FUN_004a5500` on reset. During active play: holds pointer to slot 0 of the weapons/projectile array (base at `0x016f8ee0`). `FUN_004a1290` compares `param_1 == DAT_016f8e60` to detect the player's entity. NOT a heap-allocated car array base — static inline array. Stride 0x140 (320 bytes), 8 slots, end at `0x016f98e0`. Also read by `FUN_004a1ae0/1d40/20f0/2340/4910` and `FUN_004a5ce0`** |
| 0x004a5500 | ResetProjectileArray() | to be look at/func_004a5500.txt | Analyzed | **RESET function** (NOT allocator). Does-while loop from `0x016f8ee0` to `0x016f98e0`, stride `+0x50` (int4) = **320 bytes per slot**, 8 iterations. Per slot: writes `0xFFFFFFFFFFFFFFFF` at `puVar1[-8]`, zeros all fields. After loop: clears 10 management globals (`0x016f8e40`–`0x016f8e80`) and **sets `DAT_016f8e60 = 0`**. Called from 3 places: `FUN_00455420(0x0045548e)`, `FUN_0053bbb0(0x0053d857)`, `006b6400(*)` |
| 0x004a1290 | InitWeaponSlot() | to be look at/func_004a1290.txt | Analyzed | **Weapon/projectile slot initializer**. `void FUN_004a1290(int *param_1, longlong param_2, int param_3)`. Sets `param_1[0]=0` (state), `param_1[1]=param_3` (weapon type). Type 6→ `LAB_004a3610` fn ptr; type 7→ `LAB_004a3100 + LAB_0049fd60`; else→ `LAB_004a3100 + FUN_0049faf0`. Reads `piVar8 = DAT_016f8e60`. At end: **if `piVar8 == param_1`** (this is the player's slot 0), calls `FUN_004a1170(param_1)`. 34 XREFs — called from projectile/weapon-spawn sites |
| 0x004a4910 | CameraEntityViewportUpdate() | to be look at/func_004a4910.txt | Analyzed | **Per-frame viewport rect writer**. No params. Reads 9 camera entity pointers from `DAT_016f8e40/48/50/58/60/68/70/78/80`. For each non-null ptr: checks `ptr+0x12c` (state; skip if 0 or 3), `ptr+0x128` (viewport slot index 0–3 or -1=player). Copies viewport dims from `DAT_016f9a60` or `DAT_016f9aa0 + idx*0x28` into `ptr+0x100..0x124` (viewport rect). Sets global `ScreenResPointer = ptr+0x100`. Handles split-screen minimap sub-viewports at `016f8e68/70/78`. 5 XREFs |
| 0x004a5ce0 | CameraEntityInit() | to be look at/func_004a5ce0.txt | Analyzed | **Camera/pickup entity initializer**. `void FUN_004a5ce0(int *param_1, longlong param_2, int param_3)`. `bVar3 = (DAT_016f8e60 == param_1)` = is this the player primary camera slot? Sets `param_1[0]=2` (type), `param_1[1]=param_3` (subtype), `param_1+0x32=param_2` (attached car ptr). If player slot: calls `FUN_005334a0(0x40a00000)`, sets `DAT_00660800` (FOV/zoom). Calls `FUN_004a5b50(-1, param_2+0x24)` (attach lookup); if returns 0 → falls back to `InitWeaponSlot(param_1,param_2,0)`. Else: assigns fn-ptr label `LAB_004a6700` (type 0) or `LAB_004a6600` (type 1). 21 XREFs — spawned per race participant camera |
| 0x00450700 | RaceSessionSetup() | to be look at/func_00450700.txt | Analyzed | **Race session participant builder** (NOT the per-frame race loop). `void FUN_00450700(char param_1)`. Switch on `DAT_006e34c0` (game mode 1–0x10). Builds participant table at `DAT_00f3eb70 + i*0x50` (stride 0x50=80 bytes, max 0x1e=30). **`_DAT_00f3eb04`** = live participant count. **`DAT_00f3eb00`** = player race index. `DAT_00f3eb08` high-dword = mode tag. Entries built via `FUN_0044f1f0` (player slot) and `FUN_0044f4f0` (AI fill). Car pool: `DAT_006fab50 + idx*0x110`. `DAT_0ae8e934`=event count, `DAT_0ae8e930`=max target. Modes: 1=normal, 2=lobby/custom, 3=time-trial, 4/6=MP sync-wait, 5=replay, 7=exit, 8/9=practice, 0xb=timed, 0xf=tournament, 0x10=free-race. 765 lines, 3 callers |
| 0x0044f1f0 | AddParticipantAndCount() | to be look at/func_0044f1f0.txt | Analyzed | **Writes one entry to race participant table then increments count.** 7 args: `(type, flags2, carPoolIdx, skinIdx, slotParam5, slotParam6, namePtr)`. Appends to `DAT_00f3eb70 + i*0x50` (copies 0x14 dwords of car info from `DAT_006fab50+carIdx*0x110`), skin string `+0x14`, player name at `+0x20`. Parallel header arrays written at `DAT_00f3eb14+i*0x14` and `DAT_00f3eb30`. Increments `_DAT_00f3eb04`. If `type==1` also increments player-count in `DAT_00f3eb08._0_4_`. Returns 1. **Does NOT allocate 0xAAC runtime car struct.** 8 XREFs, all from RaceSessionSetup |
| 0x0044f4f0 | AddParticipantNoCount() | to be look at/func_0044f4f0.txt | Analyzed | **Identical to 0x0044f1f0 except: does NOT increment `_DAT_00f3eb04`.** Used for AI fill slots where count is managed externally. 10 XREFs — called from RaceSessionSetup cases 2,3,f,0x10 and from `FUN_00451ab0`, `FUN_00451c70` |
| 0x004a5b50 | FindNearestPickupNode() | to be look at/func_004a5b50.txt | Analyzed | **Nearest node lookup.** `int* FUN_004a5b50(int type, float* pos)`. Iterates node array at `DAT_016f8ea0` (stride 0x38), count in `DAT_016f8e98`. If `type==-1`: finds closest node ignoring type. Else: finds closest where `node[0]==type`. Filters by `DAT_016e4428` bitmask (skip nodes with matching bits). Node layout: `[0]`=type, `[3..5]`=XYZ floats, `[0xc]`=bitmask. Returns pointer to nearest, or NULL. Called by CameraEntityInit and 3 other camera-spawn sites | ---

## RaceSysCore1 — Analyzed Functions
> **All files now in `rvgl.exe/04_RaceCore/` (race logic) and `rvgl.exe/06_Rendering/` (viewport) and `rvgl.exe/07_Input/` (input)**

| Address | Function Name | File | Status | Description |
|---------|--------------|------|--------|-------------|
| 0x0048a5c0 | GetRaceState() | RaceSysCore1/func_0048a5c0.txt | Analyzed | Returns `DAT_016942dc` (race state 0–6). Single-instruction stub |
| 0x0048a5d0 | UpdateFadeTransition() | RaceSysCore1/func_0048a5d0.txt | Analyzed | Per-frame fade animator. Writes `DAT_016942d8` (fade float). State 1=fade-in, 3=fade-out, 5=fully opaque. Also dispatches OpenAL events and calls ViewportAndCameraSetup. Multiplayer cmd buffer at `DAT_01694228/22c/30` |
| 0x0048ae80 | RaceBackgroundRenderer() | RaceSysCore1/func_0048ae80.txt | Analyzed | Race background color (sin wave from `DAT_0ace49a0`) + audio state manager. Issues GL calls via `DAT_0aea77a8/26c0/6540/3178`. Calls ViewportAndCameraSetup |
| 0x004674d0 | ItemSystemUpdater() | RaceSysCore1/func_004674d0.txt | Analyzed | Item/pickup per-frame update (NOT car physics). Active item at `DAT_0ace47c0`. Item state array: `DAT_016f8ec0 + DAT_016f9a44 * 0x28`. Item data: `DAT_01800298 + index*0x138`. Timer at `DAT_00f3fbc8` |
| 0x00445d20 | CheckPlayerReady() | RaceSysCore1/func_00445d20.txt | Analyzed | Scans keyboard (F1–F10, special keys) via `DAT_0ae6ef00/ed00`. Checks `SDL_GetModState` for Shift. Touch active at `DAT_0ae6dce9`. Returns 1 when player presses ready key |
| 0x00482e40 | NetworkRenderStub() | RaceSysCore1/func_00482e40.txt | Analyzed | If `DAT_0ae7fc4c != 0` (network mode), calls `FUN_004827e0()`. Otherwise no-op |
| 0x00495a90 | FreeTextureSlot() | RaceSysCore1/func_00495a90.txt | Analyzed | Frees sprite/texture slot `param_1`. Clears `DAT_016cf6a0 + idx*0x25`. Calls `SDL_FreeSurface`, then `(*DAT_0aea2c68)(1, &ptr)` to delete GL texture. Thread-safe via `SDL_ThreadID` |
| 0x00451ab0 | SetupRaceTrackRandom() | RaceSysCore1/func_00451ab0.txt | Analyzed | Sets `DAT_0ae8e920=0xd`, `DAT_006e34c0=0xd`. Picks random track via `FUN_00533360(0xe)` → `DAT_006e34c8`. Sets up race event list via `FUN_0044f4f0`. `DAT_006fb6a0` = forced-track mode |
| 0x005250c0 | CalcDeltaTime() | RaceSysCore1/func_005250c0.txt | Analyzed | `SDL_GetPerformanceCounter` → `DAT_0065c038` (raw ratio) + `DAT_0065c03c` (seconds per frame). Slow-motion via `DAT_0ae48da0`. Freq in `DAT_0ae48de0` |
| 0x00539bc0 | ScreenshotAndFrameEnd() | RaceSysCore1/func_00539bc0.txt | Analyzed | If `DAT_006e30a9` (A key): `glReadPixels` → `SDL_CreateRGBSurface` → `IMG_SavePNG_RW`. Also calls `(*DAT_0aea20a8)` and `(*DAT_0aea22e0)` for window/buffer ops |
| 0x00529f60 | MouseInputUpdater() | RaceSysCore1/func_00529f60.txt | Analyzed | `SDL_GetRelativeMouseState` → `DAT_0ae6dcb4/b8` (delta XY). Accumulated in `DAT_0ae6dcc8/cc`. `SDL_GetMouseState` → `DAT_0ae6dcc0`. Controller struct `DAT_00f43400` (+0x28c/290 = axis bindings, +0x28f/293 = axis type flags). Final output: `DAT_0ae6dca0` (packed steering XY) |
| 0x00529e50 | KeyboardStateSnapshot() | RaceSysCore1/func_00529e50.txt | Analyzed | Copies previous keyboard state `DAT_0ae6ef00` → `DAT_0ae6ed00`, then calls `SDL_GetKeyboardState()` and memcpy-s 0x200 bytes into `DAT_0ae6ef00`. `DAT_006fcec8` = "no keyboard" mode clears both buffers |
| 0x0052a910 | ControllerStateUpdater() | RaceSysCore1/func_0052a910.txt | Analyzed | Iterates up to 2 controllers (`DAT_00f43400` stride 0x44, `DAT_0ae6e520` stride 500). Copies previous state → `DAT_0ae6dd40`. `SDL_GameControllerGetAxis` / `SDL_JoystickGetAxis` per-axis → `DAT_0ae6e520`. `SDL_GameControllerGetButton` for buttons → offset +400. Dead-zone and sensitivity at `DAT_00f436c4/c8` |
| 0x00498ab2 | ViewportSetup_Trampoline1() | RaceSysCore1/func_00498ab2.txt | Analyzed | Single call: `ViewportAndCameraSetup(&DAT_016f9a60, DAT_006e3470)` |
| 0x00498d8d | ViewportSetup_Trampoline2() | RaceSysCore1/func_00498d8d.txt | Analyzed | Same as above |
| 0x004997df | ViewportSetup_Trampoline3() | RaceSysCore1/func_004997df.txt | Analyzed | Same as above |
| 0x00450700 | RacingLoopBody() | RaceSysCore1/func_00450700.txt | Partial | 764-line function — likely main racing game loop body. Only header read. **Export body urgently**  | ---

## RGameModeManagement — Analyzed Functions
> **All files now in `rvgl.exe/00_CoreLoop/` (frame dispatch), `rvgl.exe/04_RaceCore/` (mode logic), `rvgl.exe/08_Network/` (multiplayer)**

| Address | Function Name | File | Status | Description |
|---------|--------------|------|--------|-------------|
| 0x00401610 | FrameUpdateDispatcher() | RGameModeManagement/func_00401610.txt | Analyzed | Per-frame: calls `CalcDeltaTime`, `FUN_00525350`. Game mode 4/6 branch runs render pipeline (`FUN_0052e760`, physics, AI). Otherwise updates `DAT_0ae48dc8` accumulator and iterates linked list at `DAT_0acee9e8` (+0x10 next ptr, updates `+0xf90` timer) |
| 0x0044c130 | GameGaugeReplayState() | RGameModeManagement/func_0044c130.txt | Analyzed | Loads `game_gauge_replay.rpl`, sets mode `DAT_0ae8e920=5 / DAT_006e34c0=5`. Copies replay data block. Sets game-state fn `DAT_006e30a0 = &LAB_00406b40` |
| 0x0044c820 | GameGaugeFPSRecorder() | RGameModeManagement/func_0044c820.txt | Analyzed | Per-second FPS: increments `DAT_006fceb0` (frames this second), `DAT_006fceac` (total frames). After 1000ms: records min (`DAT_006fcea8`), max (`DAT_006fcea4`), history array `DAT_006fcee0[300]`. On trigger `DAT_00f432a4`, calls `FUN_0048a560(3)`, writes `profiles/fps.txt` |
| 0x004efd50 | SetCarBehaviourState() | RGameModeManagement/func_004efd50.txt | Analyzed | Sets car behaviour slot `param_2` (0–5) on car object `param_1`. Writes function pointers into `param_1->sub+0x340/348/350/358/370/378`. Key state0 = idle, 1 = racing, 2 = spectating, 3 = battle, 4 = game-gauge, 5 = ghost. Calls `FUN_0048a560(1)` when winner detected at race state 4 |
| 0x0052f770 | MultiplayerPacketHandler() | RGameModeManagement/func_0052f770.txt | Partial | 1013-line monster. Processes network packets (`DAT_0ae6fc38` packet buf, type at `*buf`). Cases 1–N = different packet types. Reads player IDs from `DAT_0ae6f6a4/ac`. `DAT_006e34c4` = race mode. Only first 120 lines read |
| 0x00541f80 | RaceEndManager() | RGameModeManagement/func_00541f80.txt | Analyzed | State machine on `DAT_0ae8e860` (0–3+). State 0: init via `FUN_005415c0`, advances to 1 after 3s. State 1: `FUN_00541220`, advance to 2. State 2: runs lap/eliminate logic (`FUN_00541bb0/00541490/00540ee0`). State 3: after delay calls `FUN_0048a560(3)` (fade out). At race-state 6 → cleans up objects (`FUN_005437d0/0053f530/004f06d0`), restarts race or goes to menu (`DAT_006e34c0=0xd`). Key: `DAT_0ae8e8c8` = winner car pointer | ---

## RenderingPipeline — Analyzed Functions
> **All files now in `rvgl.exe/06_Rendering/`**

| Address | Function Name | File | Status | Description |
|---------|--------------|------|--------|-------------|
| 0x004628b0 | DrawSprite2D() | RenderingPipeline/func_004628b0.txt | Analyzed | Draws a 2D screen-space sprite. Params: x,y,w,h (world units), UV coords, color RGBA. Uses `ScreenCenterX/Y` and scale `DAT_006e3468/46c`. Converts to NDC, fills `DAT_00f92ba0`–`DAT_00f92c40` quad vertex buffer. 62 call sites. Multiplayer: buffers into `DAT_00f40dca` array |
| 0x00491f60 | SetupGLRenderState() | RenderingPipeline/func_00491f60.txt | Analyzed | Sets up GL blend/depth state for a new render pass. Calls `(*DAT_0aea2d28)(0x8457)` (disable something), `(*DAT_0aea2d20)(0xb60/b71)` (enable), `(*DAT_0aea2f40)(0xbe2/bc0)`. Sets blend eqn `(*DAT_0aea2288)(0x302, 0x303)`. Binds texture via `(*DAT_0aea2138)(0xde1, DAT_016d6468)`. 70 call sites |
| 0x00492150 | DrawTexturedQuad() | RenderingPipeline/func_00492150.txt | Partial | 661-line function — heavily used (510 call sites). Full texture-mapped quad draw with clipping, string rendering, and complex UV logic. Re-export for full analysis |
| 0x004962d0 | LoadTextureByName() | RenderingPipeline/func_004962d0.txt | Analyzed | Loads a texture by filename string `param_1` into slot `param_2` (max 0x95). Calls `FUN_005370a0` to check file exists. Handles MIP extension variants (loops `.bmp0`–`.bmp9`). 47 call sites |
| 0x00539fa0 | SetDepthWriteMode() | RenderingPipeline/func_00539fa0.txt | Analyzed | One-liner: calls `(*DAT_0aea23a0)((param_1=='\0') ? 0x4100 : 0x14100)`. Enables/disables depth mask (GL_DEPTH_WRITEMASK). 15 call sites | ---

## RFunctionPointerTableTargets — GL Function Pointer Map
> **All files now in `rvgl.exe/09_GLPointers/`**

> These are **runtime GL function pointers** loaded at startup from libGLESv2.dll via `SDL_GL_GetProcAddress` / similar. All are `undefined8` (= 8-byte function pointers).

| Address | GL Equivalent | XREF Count | Key Callers |
|---------|--------------|-----------|-------------|
| 0x0aea23a0 | `glDepthMask` | 2 | `SetDepthWriteMode` (0x00539fa0), `FUN_0053bbb0` |
| 0x0aea23e8 | Unknown GL fn | 1 | `FUN_004889c0` |
| 0x0aea2cb0 | Unknown GL fn | 1 | `FUN_004884d0` |
| 0x0aea2cb8 | `glBindVertexArray` / VAO fn | 17 | `FUN_0042ce80`, `FUN_00481760`, `FUN_004827e0`, `FUN_004908a0`, `FUN_004d4e90`, `FUN_004de430/50` |
| 0x0aea2d20 | `glDisable` | 213 | `SetupGLRenderState` and most render functions |
| 0x0aea2d28 | `glEnable` | 58 | `SetupGLRenderState`, `FUN_00443ec0`, many |
| 0x0aea2d70 | `glBlendFunc` | 43 | `FUN_00443ec0`, `FUN_004621b0`, `FUN_0048ae80`, many |
| 0x0aea2f40 | `glEnable` variant / `glBlendEquation` | 279 | Most rendering functions |
| 0x0aea31f0 | Unknown GL fn | 1 | `FUN_004889c0` |
| 0x0aea5eb0 | GL viewport fn | 1 | `ViewportAndCameraSetup` (0x004a4307) |
| 0x0aea6570 | `glBlendFuncSeparate` | 85 | `SetupGLRenderState`, `FUN_004621b0`, `FUN_004884d0` |
| 0x0aea7910 | GL viewport fn variant | 1 | `ViewportAndCameraSetup` (0x004a4301) | ---

## Quick Reference

### Most Important Addresses
```
0x006e30a0 = Game state function pointer 
0x00450700 = RacingLoopBody() — 764 lines, MAIN RACE LOOP 
0x00499690 = RacingGameLoop() entry 
0x004ab810 = RuntimeCarArrayBuilder1 — calls InitRuntimeCarStruct 
0x005589b0 = RuntimeCarArrayBuilder2 — calls InitRuntimeCarStruct 
0x00402ec0 = Main game loop entry
0x00402980 = Event processor
0x016942dc = Race state (0-6) 
0x006fab50 = CarInfo heap pointer (static pool)
0x006fab58 = Car count (starts 0x31 = 49)
0x006fab80 = Runtime car template (0xAAC bytes)
0x67c77ae0 = GetCurrentContext (called by ALL GL functions)
0x6814c020 = TLS index for GL contexts
```

### Most Important Files
```
RVGL.exe/RaceSysCore1/func_00450700.txt    - RACE LOOP BODY (export more!)
RVGL.exe/maingameloop.txt                  - Start here to understand flow
RVGL.exe/ultra_priortity1/dat_006e30a0.txt - Game state function pointer
libGLESv2.dll/func_67c77ae0.txt            - Context getter (hot path)
libGLESv2.dll/dat_6814c020.txt             - TLS system root
```

### Top 50 Export Queue
> Status key: =analyzed  =partial  =not yet exported
```
ZONE 1 — Runtime car slot builders (who calls InitCarSlotFull?)

 1.  0x004452?? — FUN containing 0x0044530c   calls InitCarSlotFull 
                    (navigate to 0x0044530c in Ghidra → find function start)
                    ALSO contains 0x00445401 + 0x00445511 (3 InitCarSlotFull calls = main race car BUILDER)
 2.  0x0044cb60 — FUN_0044cb60                 calls InitCarSlotFull at 0x0044cc31 
 3.  0x004f03a0 — FUN_004f03a0                 calls InitCarSlotFull at 0x004f05b8 
 4.  0x004e7fb0  — FUN containing 0x004e7fb2   calls InitCarSlotFull 
 5.  0x00455420 — FUN_00455420                 calls FUN_004a5500 (cam reset) at 0x0045548e 

ZONE 2 — Race game loop body

 6.  0x00499690 — RacingGameLoop()              main racing loop entry (per-frame) 
 7.  0x00450700 — RaceSessionSetup()            DONE — no need to re-export
 8.  0x004a1ae0 — FUN_004a1ae0                 reads DAT_016f8e60, 2 XREFs inside 
 9.  0x004a1d40 — FUN_004a1d40                 reads DAT_016f8e60, 2 XREFs inside 
10.  0x004a20f0 — FUN_004a20f0                 reads DAT_016f8e60 
11.  0x004a2340 — FUN_004a2340                 reads DAT_016f8e60 (2 XREFs) 

ZONE 3 — Camera / entity system

12.  0x004a5e90 — FUN_004a5e90                 called from camera system (XREFs: 004a6059) 
13.  0x004a60b0 — FUN_004a60b0                 called from camera system 
14.  0x004a4e50 — FUN_004a4e50                 calls CameraEntityViewportUpdate 
15.  0x004a1170 — FUN_004a1170                 player-slot special init (player slot-0 detected) 

ZONE 4 — Race session / mode helpers

16.  0x0044bab0 — FUN_0044bab0                 called at start of every RaceSessionSetup 
17.  0x0044f980 — FUN_0044f980                 called at end of cases 1,2,3,0xf 
18.  0x0044fb30 — FUN_0044fb30                 called in cases 1,2 
19.  0x004504e0 — FUN_004504e0                 car selector for race (extended mode path)
20.  0x00450600 — FUN_00450600                 host-restart car selector (case 4/6)
21.  0x004493d0 — FUN_004493d0                 called in cases 5,7 (race end/exit)
22.  0x00449dc0 — FUN_00449dc0                 called in case 7 (exit path)
23.  0x0044ba10 — FUN_0044ba10                 called in case 7 (exit path)
24.  0x00532ba0 — FUN_00532ba0                 sync wait loop (case 4)
25.  0x00451c70 — FUN_00451c70                 calls AddParticipantNoCount (alongside SetupRaceTrackRandom)

ZONE 5 — Race teardown / cleanup

26.  0x0053bbb0 — FUN_0053bbb0                 calls cam-reset FUN_004a5500 at 0x0053d857 (race teardown) 
27.  0x00455500 — FUN containing 0x0045548e   (parent of the camera-reset call site) — likely FUN_00455420

ZONE 6 — Entity physics plumbing

28.  0x0049faf0 — FUN_0049faf0                 entity state fn-ptr (in weapon/camera init)
29.  0x0049c0f0 — FUN_0049c0f0                 called in camera entity reset loop
30.  0x0049c110 — FUN_0049c110                 physics param in InitWeaponSlot state==0
31.  0x0049b650 — FUN_0049b650                 vector math (InitWeaponSlot)
32.  0x0049d340 — FUN_0049d340                 motion state update (InitWeaponSlot state==1)
33.  0x0049e560 — FUN_0049e560                 physics update (InitWeaponSlot state==1)
34.  0x005a7a20 — FUN_005a7a20                 sqrt-NaN handler

ZONE 7 — Car pool helpers

35.  0x0043c0e0 — FUN_0043c0e0                 car index resolver (used in RaceSessionSetup)
36.  0x0043c1e0 — FUN_0043c1e0                 skin index resolver
37.  0x0044a250 — CheckIfDifficultyTierCompleted()  unlock/progression checker (Batch 11)

ZONE 8 — Networking / multiplayer

38.  0x0052f770 — FUN_0052f770                 1013-line multiplayer packet handler 
39.  0x0052d470 — FUN_0052d470                 ENet event type 2 handler
40.  0x005312a0 — FUN_005312a0                 ENet event type 3 handler
41.  0x0052c660 — FUN_0052c660                 ENet event type 1 handler

ZONE 9 — Rendering

42.  0x00492150 — FUN_00492150                 DrawTexturedQuad — 510 call sites
43.  0x00491f60 — FUN_00491f60                 called in sync-screen case 4
44.  0x004884d0 — FUN_004884d0                 called in sync-screen case 4
45.  0x00539fa0 — FUN_00539fa0                 called in sync-screen case 4

ZONE 10 — Utility / string / math

46.  0x005334f0 — FUN_005334f0                 memcpy/strcpy helper (used everywhere)
47.  0x00533360 — FUN_00533360                 AI car randomizer
48.  0x00533aa0 — strlen()                      custom string length (Batch 11)
49.  0x00452780 — GetTrackIdByName()            track ID resolver (Batch 11)
50.  0x00452130 — GetTrackInfo()                track data loader (Batch 11)
```

### Runtime Car Struct Layout (int*, 0xAAC bytes)
```
slot[0]   (byte +0x00) = car array index / ID
slot[1]   (byte +0x04) = car TYPE: 1=player, 9=spectator/off, 4=?
slot+0x12 (byte +0x48) = packed car IDs (param_2 | param_3 << 32)
slot+0x24 (byte +0x90) = ptr to physics/model data  
slot+0x32 (byte +0xC8) = ptr to track/wheel data
slot+0x416..0x41c     = physics speed/accel params
slot+0x3e9             = 0x3E4CCCCD (float 0.2 = default grip?)
slot[0x422]            = AI/control state
slot[0x423]            = spectate flag (byte)
```

### Key Global Variables for remove_car_from_race
```
CONFIRMED — Participant table (setup metadata, NOT runtime physics):
  DAT_00f3eb70  = race participant table (stride 0x50, max 30 entries)
  _DAT_00f3eb04 =  live participant count (max 0x1e=30)
  DAT_00f3eb00  =  player race index in participant table
  DAT_00f3eb14  = per-entry header array (stride 0x14: type | flags2)
  DAT_00f3eb30  = per-entry car+skin index (stride 0x28: skinIdx | carPoolIdx)

CONFIRMED — Camera entity system (NOT car structs):
  DAT_016f8ee0  = camera entity inline array (stride 0x140, 9 slots)
  DAT_016f8e40–016f8e80 = 9 camera entity pointers
  DAT_016f8e60  = player primary camera entity ptr
  DAT_016f8e98  = node count for pickup/camera node array
  DAT_016f8ea0  = camera node array base (stride 0x38, FindNearestPickupNode)

CONFIRMED — Runtime car slot system:
  DAT_0acee7e8  =  pointer to the PLAYER runtime car slot (0xAAC bytes)
  DAT_0acee9e8  =  linked list head of ALL runtime race car structs (node+0x10 = next)
  slot[1]       =  car type: 1=player, 9=spectator/disabled, 4=special
  slot[0x423]   = spectate flag byte

UNKNOWN — runtime car array flat base:
  WHO allocates the 0xAAC slots? → FUN containing 0x0044530c (#1 export)
  FUN_004ab810 (InitCarSlotFull) INITIALIZES a slot but is CALLED FROM outside
```

---

**Next Action:** In Ghidra, navigate to address `0x0044530c` → find the parent function (use "F" key or Function Start). Export that function — it calls `InitCarSlotFull` 3× (at 0x0044530c, 0x00445401, 0x00445511) and is the **race car slot builder**. Also export `FUN_0044cb60` (0x0044cb60) and `FUN_004f03a0` (0x004f03a0) which are other callers of `InitCarSlotFull`.

---

## Batch 5 — New Functions (Main Loop, Race State, Audio, Timer)

### Timer / Delta Time System
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x005250c0 | **CalcDeltaTime()** | Analyzed | Core frame-timing function. Called every frame. Updates `DAT_0065c03c` (DeltaTime float). Reads `SDL_GetPerformanceCounter`, computes dt = FrameTicks / PerfFrequency, caps at 100ms/frame. Optional slowmo-adjust via `DAT_0ae48da0` (GameSpeedPercent). |
| 0x00525350 | AccumulateDeltaTime() | Analyzed | Accumulates `DAT_0ae48dd8` into `_DAT_0ae48e10`. Gates `DAT_0065c03c = 0.0` if under threshold (rapid frames). |
| 0x00525470 | GetPerfCounter() | Analyzed | Returns `SDL_GetPerformanceCounter()` (raw perf ticks). Used as timestamps throughout engine. |

### Race State Machine
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x0048a560 | **SetRaceState(int state)** | Analyzed | Sets `DAT_016942dc`. Transitions 0→1 initializes StateBlend to -0.2; 1→3/4 initializes to +0.6. Guards against same-state transitions. |
| 0x0048a5c0 | **GetRaceState()** | Analyzed | Returns `DAT_016942dc` (int, 0-6). Called ~15 times per frame. |
| 0x0048a5d0 | **UpdateRaceState()** | Analyzed | Advances state blend by `DeltaTime × 4.0`. State 1 → 5 (paused) when blend > 1.2. State 3 → 6 (end) when blend < -0.2. |
| 0x004995f0 | LoadLapImages() | Analyzed | Loads lap-specific image assets for `DAT_016f7908` (current lap). Loads "gfx/devlogoNa/b/c.bmp" variants. |
| 0x00499c40 | SortCarsForRace() | Analyzed | Complex car ordering matrix algorithm. Sorts up to N cars by score matrix for position assignment. |

### Level Load System
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00404c30 | **LevelLoad()** | Analyzed | Full level loading pipeline. Sets up viewport, loads track textures, spawns loading thread, renders loading screen with progress bar (% from `DAT_006e3968`), waits for thread. Calls `FUN_0045f690` + `FUN_004a1170` at end. |
| 0x00401610 | FrameUpdateDispatcher() | Analyzed | Idle/unfocused handler. Calls `FUN_005250c0()` (CalcDeltaTime), updates time accumulator, iterates car linked list. Called when `DAT_0065c021 = 0` (window minimized) with `SDL_Delay(10)`. |

### Audio System — Init / Lifecycle
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00527230 | **AudioInit()** | Analyzed | Initializes OpenAL via `FUN_00561a20`. Prints vendor/version. Calls `alDistanceModel(0)`, `alGenSources(1, &MusicALSourceID)`. Sets FLUID_SOUNDFONT env var. Sets `DAT_006e3434=1` on failure. |
| 0x00527400 | **AudioShutdown()** | Analyzed | Stops music stream, `alDeleteSources(1, &MusicALSourceID)`, calls `FUN_00561ad0()` (OpenAL context shutdown). |
| 0x005277d0 | **LoadAllSfx()** | Analyzed | Loads all 38 built-in WAV SFXs from `PTR_s_wavs_moto_wav_00661c40` table. Allocates up to 128 AL sources via `alGenSources`. Stores buffer IDs in `DAT_0ae4d240[]`, source handles in `DAT_0ae4c640[]`. |
| 0x00527a20 | **FreeAllSfx()** | Analyzed | Stops+deletes all AL sources and buffers. Clears `DAT_0ae4d240[]`, `DAT_0ae4da40[]`, `DAT_0ae48e50[]`. |
| 0x00527ba0 | **StopAllSfx()** | Analyzed | Calls `alSourceStop` on all active sources. Also stops music if playing. Clears loop-sfx array. |

### Audio System — SFX Playback
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00526d70 | **GetFreeSfxSource(rank)** | Analyzed | Source preemption logic. Finds AL_STOPPED/AL_INITIAL source. If none free, steals lowest-rank playing source. Returns ptr into `DAT_0ae4c640` pool. |
| 0x00527520 | LoadSfxSlot(int slot) | Analyzed | Loads WAV file for slot index from `PTR_s_wavs_moto_wav_00661c40[slot]` into `DAT_0ae4d240[slot]`. |
| 0x00527630 | GetSfxSlotByName(char* name) | Analyzed | Searches built-in SFX names (0-105) then custom name table. Returns slot index. Returns 0xFFFFFFFF if not found. |
| 0x005279b0 | FreeSfxSlot(int slot) | Analyzed | Deletes AL buffer for dynamic/custom SFX slot (slot >= 0x6A). Clears custom name entry. |
| 0x00527e50 | FindSfxInstanceByID(int sfxID) | Analyzed | Returns oldest playing instance of a given SFX ID from `DAT_0ae4c640` pool. Used for looping checks. |
| 0x00527f20 | **PlaySfxSimple(sfxID, vol, pan, pitch, loop)** | Analyzed | One-shot SFX playback. Sets `AL_BUFFER`, `AL_LOOPING`, `AL_GAIN=(vol*MasterVol)/10000`, `AL_POSITION=(pan/50-1, 0, 0)`, `AL_PITCH=pitch/22050`. Calls `alSourcePlay`. |
| 0x005280a0 | **PlaySfx3D(sfxID, vol, pos3D, ...)** | Analyzed | 3D positional SFX. Uses `FUN_005263c0` for 3D attenuation calc before `alSource3f`. |
| 0x005282d0 | **StartLoopingSfx(sfxID, rank, loopFlag, pos, handle)** | Analyzed | Allocates `LoopSfxHandle` entry (stride 0x38), binds to free AL source, calls `alSourcePlay`. Returns int* handle for later updates. |
| 0x005284b0 | ChangeSfxAndStop(int* handle, int newSfxID) | Analyzed | Stops existing AL source linked to handle, sets new SFX ID in handle. Used to swap loop sounds (e.g. engine pitch change). |
| 0x00528510 | StopLoopingSfx(longlong loopState) | Analyzed | Stops AL source at `loopState+0x18` ptr, clears the ptr. |
| 0x00528550 | ClearLoopingSfx(longlong loopState) | Analyzed | Clears the `al_src_entry` pointer in the `LoopSfxHandle` array without stopping. |
| 0x00527b80 | StopSfxInstance(longlong instance) | Analyzed | Calls `alSourceStop(instance+0x10)` (= stop one AL source). |

### Audio System — Focus / Window Events
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00527cf0 | **PauseAllSfx()** | Analyzed | Called on `SDL_WINDOWEVENT_FOCUS_LOST`. Pauses all AL_PLAYING sources via `alSourcePause`. Pauses music via `FUN_00565880`. Sets `DAT_0ae48e34 = 1`. |
| 0x00527da0 | **ResumeAllSfx()** | Analyzed | Called on `SDL_WINDOWEVENT_FOCUS_GAINED`. Resumes all AL_PAUSED sources via `alSourcePlay`. Resumes music. Sets `DAT_0ae48e34 = 0`. |

### Audio System — Music / BGM
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x005285b0 | **PlayMusic(char* file)** | Analyzed | Starts BGM. Loads OGG/WAV stream via `FUN_00563f60(file, 250000)`. Binds to `MusicALSourceID`. Plays with `FUN_005668b0(source, stream, 3, looping, callback, 0)`. Sets volume via `alSourcef(AL_GAIN)`. |
| 0x00526f10 | **UpdateMusicPlayback()** | Analyzed | Advances to next track (`MusicTrackIndex`). Loads `"redbook/track%02d.%s"` format. Calls `FUN_005668b0` to start playback. Updates `MusicVolume` → `alSourcef(AL_GAIN)`. |
| 0x00526b60 | ScanMusicDirectory() | Analyzed | Scans current music directory, collects file names matching supported extensions into `DAT_0ae5db40[]` sorted array. Max 0xFF entries = `MusicFileCount`. |
| 0x00527400 | AudioShutdown() | Analyzed | Already listed above — also handles music stream cleanup. |

### Audio Low-Level (OpenAL wrappers)
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00561a20 | OpenAL_Init() | Analyzed | `alcOpenDevice` + `alcCreateContext` + `alcMakeContextCurrent`. Returns true/false. |
| 0x005618a0 | OpenAL_GetError() | Analyzed | Returns error string from OpenAL context. |
| 0x00561ad0 | OpenAL_Shutdown() | Analyzed | `alcDestroyContext` + `alcCloseDevice`. |
| 0x00562e90 | LoadWAVToBuffer() | Analyzed | Loads WAV file data, calls `alBufferData`. Returns AL buffer ID or 0 on fail. |
| 0x00563f60 | OpenMusicStream() | Analyzed | Opens OGG/WAV file as streaming object (250000 byte buffer). Returns stream ptr or 0. |
| 0x00563ad0 | CloseMusicStream() | Analyzed | Frees music stream object. |
| 0x00565880 | PauseMusicStream() | Analyzed | Pauses streaming AL source. |
| 0x00565990 | ResumeMusicStream() | Analyzed | `alSourcePlay` on streaming source. |
| 0x00565aa0 | SetMusicVolume(float vol) | Analyzed | `alSourcef(MusicALSourceID, AL_GAIN, vol)`. |
| 0x00565e90 | StopMusicStream() | Analyzed | `alSourceStop` + cleanup. |
| 0x005668b0 | StartMusicPlayback() | Analyzed | Binds stream to source, sets loop mode, callback. Starts streaming playback. |
| 0x005263c0 | Calc3DAtten() | Analyzed | Computes 3D sound attenuation/panning from position. Used by PlaySfx3D. |

### Rendering Pipeline — GL State Management
> GL function pointers all at `DAT_0aea****` (loaded from libGLESv2.dll). State cache at `DAT_0ae7fc**`.

| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x004628b0 | **DrawTexturedQuad(x,y,w,h,u1,v1,u2,v2,color,blend,texid)** | Analyzed | Core 2D sprite/HUD primitive. Converts screen coords (via ScreenCenterX/Y + DAT_006e3468/46c scale) to NDC. Builds 4 vertices (stride 0x2c). If `DAT_0ae7fc4c`: pushes to DAT_00f40dc0 batch buffer (stride 0xb8). Else: glVertexAttribPointer x4 + glDrawArrays(GL_TRIANGLE_STRIP, 0, 4). param_11: 0=alpha-blend, 1=additive, -1=no-state-change. |
| 0x004884d0 | **GL_SetupNormalRenderState()** | Analyzed | Standard state setup: glDepthMask(1), glEnable(GL_DEPTH_TEST), glDepthFunc(GL_LESS=0x203), glBlendFunc(SRC_ALPHA,ONE_MINUS_SRC_ALPHA). Caches in DAT_0ae7fc8c/8d/80. |
| 0x004827e0 | **FlushBatchedQuads()** | Analyzed | Flushes 2D VAO sprite batch. Disables GL_CULL_FACE(0xb44), glDepthMask(0), disables depth test. Calls FUN_00461d30+FUN_004621b0+UBO/shader setup. Binds VAO(DAT_01694260), sets 5 attrib ptrs (stride 0x2c), loops DAT_01694230 batch (DAT_01694228 entries), glDrawArrays per entry. |
| 0x004886b0 | **RenderSceneFull()** | Analyzed | THE main full-pass renderer. Guards: DAT_0ae7fc4c!=0. Resets DAT_00f43a00=0. Sequence: GL_SetupNormalRenderState → CommitFrameRenderWork → SortDynamicObjects → SortTrackMeshObjects → DrawGameObjects → (optionally) UpdateGhostRender/RenderCarMeshes/RenderSkidmarksShadows → FlushBatchedQuads → ClearDynamicMeshBatch + ClearTrackMeshBatch + ClearGameObjectsCount. |
| 0x00488750 | **RenderSceneLightBatch()** | Analyzed | Lighter render pass. Skips dynamic/gameobj sorts. Calls: GL_SetupNormalRenderState → (depth-off override) → SortTrackMeshObjects → optional ghost/car/skid → FlushBatchedQuads. |
| 0x00488800 | **SetFogDistances(float near, float far)** | Analyzed | DAT_006e347c=near, DAT_006e3480=(far*DrawDistLevel)/5. Computes DAT_006e3484=(near+far)/range, DAT_006e3488=(2*near*far)/range. |
| 0x004888a0 | **SetFogParams(float a, float b, float c)** | Analyzed | Sets DAT_006e3494 (near clip), DAT_006e3490 (fog near), DAT_006e348c (fog mid), DAT_006e34a0 (linear slope = 1/(c-b)). |

### Rendering Pipeline — Mesh Sort & Submit
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x004b6b60 | **SortTrackMeshObjects()** | Analyzed | Iterates DAT_018146b0 (stride 0x2d8, count DAT_018146a8). Shadow pass if DAT_0ae7fc63. Bucket-sorts visible entries (obj+0xc0==-1) into DAT_01813a80[] linked list (193 texture-slot buckets, keyed by obj+0x5c). |
| 0x004b7700 | **ClearTrackMeshBatch()** | Analyzed | DAT_018146a8=0, zeros DAT_018140a0[0xc1] + DAT_01813a80[0xc1]. |
| 0x00500f60 | **SortDynamicObjects()** | Analyzed | Parallel to SortTrackMeshObjects for dynamic objects. Uses DAT_0adce3c0/3ac, buckets DAT_0adcdda0[]/DAT_0adcd780[]. Optional shadow via FUN_004fdef0. |
| 0x00501c10 | **ClearDynamicMeshBatch()** | Analyzed | Zeros DAT_0adcdda0/d780/d160/ccb40[0xc1] each. Resets DAT_0adce3ac=0, DAT_0adce3a8=0. |
| 0x00501c80 | **FreeDynamicMeshBuffers()** | Analyzed | Frees all dynamic car mesh device buffers. Iterates DAT_0adccb08 (stride 0x1e0), frees +0x48/0x98/0xa8/0xb0. Frees DAT_0adccb18/10/ccae0. |
| 0x004d4260 | **DrawGameObjects()** | Analyzed | Renders DAT_0ace47b0 (stride 0x2d8, count DAT_0ace47a8). Per object: binds VAO(obj[0]), glUseProgram(0x8892,0x8893), 5x glVertexAttribPointer (stride 0x2c), FUN_00480fb0(obj,0) draw call, toggles GL_CULL_FACE on obj[0x2d], texture bind from obj_meshinfo+4. |
| 0x004d46e0 | **ClearGameObjectsCount()** | Analyzed | DAT_0ace47a8=0. |
| 0x004d46f0 | **FrustumCullSphere(float* center, float radius, float* zdist)** | Analyzed | Tests sphere vs 4 frustum planes (DAT_016f98c0..016f98fc). Returns 0=rejected,1=partial,2=inside. Clamps to [DAT_006e347c,DAT_006e3480] depth range. |
| 0x0053de80 | **CommitFrameRenderWork()** | Analyzed | FUN_0053b020 (VAO/VBO commit) + FUN_0053b530 (UBO update). Stamps DAT_0ae86180 draw list entries with frame tag. Sets _DAT_0ae7e9bc=1. |
| 0x0053df00 | **ClearMeshBatchState()** | Analyzed | Zeros 9 counters at DAT_0ae85e20..0ae85e54. |
| 0x0053df60 | **SetShaderUniformBlock(slot,a,b,c)** | Analyzed | Updates 16-slot uniform cache (DAT_0ae85e80, stride 0xc). Calls glUniform4iv(0x8a11) only on change. |
| 0x004b77d0 | **TransformMeshVertices(mesh, rotmat, pos)** | Analyzed | CPU vertex transform: multiplies each vertex by 3x3 rotation + translate. Stores at vertex+0x24..+0x2c. Used for shadow/reflection prep. |

### Rendering Pipeline — Per-Object Draw Helpers
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00480cd0 | **AddToSemiTransparentList(obj)** | Analyzed | Appends obj ptr to `DAT_01694220` (dynamic realloc, starts 0x1000 entries). Increments `DAT_01694218`. Clears `obj+0x2d0`. Used for delayed transparent/occluded object rendering. |
| 0x00480d60 | **InitSemiTransparentList()** | Analyzed | malloc(0x8000) for `DAT_01694220`, capacity `DAT_0169421c = 0x1000`. |
| 0x00480dd0 | **FreeSemiTransparentList()** | Analyzed | Frees `DAT_01694220`, `DAT_0169421c = 0`. |
| 0x00480e00 | **SetupMeshRenderSlot(obj)** | Analyzed | Allocates GL resource slot for mesh. If `obj+0x5c == -1`: calls `FUN_0053a070(obj+0x58, flags)` to get batch slot. Stores opaque slot at `obj+0x5c/0x60`, transparent/double-sided at `obj+0x64/0x68`. If shadow pass: updates `obj+0x6c` shadow flags. |
| 0x00480fb0 | **DrawMeshObject(obj, flags)** | Analyzed | THE per-object draw call. Calls SetupMeshRenderSlot. Selects opaque vs transparent VBO (flag bit2). Calls `FUN_00539fd0(mesh, slot)` to bind VBO+VAO. Per flag bits: 0=shadow map, 1=env map, 2=transparent, 3=light map, 4=specular, 5=normal map, 8=decal. Shadow mode: calls SetShaderUniformBlock for material color + world matrix (`obj+8..0x34`). |
| 0x00481320 | **RenderCarMeshes()** | Analyzed | Iterates `DAT_01694240` (count `DAT_01694238`). For visible entries (`obj+0xc0 == -1`): binds shared VAO `DAT_016942b8/bc`, sets 5 vertex attribs (stride 0x2c), calls DrawMeshObject(obj,0), binds texture from `obj+0x10+4`. Non-visible: deferred to AddToSemiTransparentList. |
| 0x00481760 | **RenderTransparentAndSortedObjects()** | Analyzed | Two-pass semi-transparent renderer. **Pass 1**: Shell-sorts `DAT_01694220[DAT_01694218]` by `obj+0xc4` (depth). Merges consecutive same-mesh entries (obj+0xac count). Renders with blend by `obj+0x30`: 0=SRC_ALPHA/DST_ALPHA, 1=additive (GL_ONE,GL_ONE), 2=premult (GL_ONE,1-SRC_ALPHA). Incr `DAT_00f43a18` by tri count. **Pass 2**: skipped objects (obj+0xb9). Shell-sort gaps from `DAT_00676384..006763a0`. |
| 0x0047d220 | **UpdateGhostRenderEntries()** | Analyzed | Processes ghost car replay for rendering. Reads `DAT_016942a0` (stride 0x180, count `DAT_016942ac`). Filters active ghosts (obj+0x174 != 0.0). Shell-sorts by depth `obj+0x174`. Builds render packets at `DAT_016942c0` (stride 0x38, count `DAT_016942c8`). Each packet: `+0x1c`=playerIdx, `+0x20`=animState, `+0x24`=depthZ. |

### Rendering Pipeline — Shader/GPU Buffer Layer (Batch 10)
> These are the lowest-level GL abstraction functions — VBO allocation, data upload, draw submission, and 2D sprite batch flush.
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00461d30 | **FlushBatchedPanelPolys()** | Analyzed | Flushes 2D panel/HUD quad batch. Guards on `DAT_00f40dca > 0`. Builds unique (texId, blendMode) groups into `DAT_00f40dd8` (stride 0x38, count `DAT_00f40de0`). Calls `FUN_0053a460` to alloc VBO+EBO (`DAT_00f40dcc/dd0`, usage=GL_DYNAMIC_DRAW=0x88e0). For each group: copies vertex data into `DAT_0ae7e998` (vertex staging) + builds quad index list (0,1,2,2,0,3 per quad) into `DAT_0ae7e990` (index staging). Calls `FUN_0053a800(vtx_ptr, vtx_count, idx_ptr, idx_count)` = glBufferSubData upload. Stores (VBO_handle, draw_count) at group+0x30. Calls `FUN_0053ab10()` to submit. Resets `DAT_00f40dca=0`. |
| 0x004621b0 | **DrawBatchedPanelPolys()** | Analyzed | Issues actual GL draw calls for panel batch. Guards on `DAT_00f40de0 > 0`. If shader changed vs `DAT_0ae7fca8`: binds vertex shader (0x8892) + fragment shader (0x8893) via `DAT_0aea2028`, sets 5 `glVertexAttribPointer` attribs (stride 0x2c), enables attribs 0-4. Per group: sets blend state by mode (0=SRC_ALPHA/ONE_MINUS_SRC_ALPHA, 1=GL_ONE/GL_ONE additive). Binds texture: `glBindTexture(0xde1, DAT_016cf7b8[texId*0x4a])`. Calls `FUN_0053ab80(4, vbo_handle, draw_count, 1)` = indexed glDrawElements. Increments `DAT_0ae7e9ac` (GL state change counter) on every state change. |
| 0x00491900 | **FlushBatchedTextPolys()** | Analyzed | Same as FlushBatchedPanelPolys but for text rendering system. Source: `DAT_016cf560` (capacity `DAT_016cf568` = 0x1000 quads = 0xb0000 byte pool). Builds VBO/EBO pair `DAT_016cf56c/570`. Stores result at `DAT_016cf5a8` = CONCAT44(VBO_handle, index_count). Calls `FUN_0053ab10()`. Resets `DAT_016cf56a=0`. |
| 0x00491be0 | **DrawBatchedTextPolys()** | Analyzed | Issues draw calls for text batch. Guards on `DAT_016cf5ac > 0`. Same shader/attrib bind pattern as DrawBatchedPanelPolys. Disables GL_CULL_FACE (0xb60) instead of 0xb44. Enables GL_BLEND (0xbe2), sets SRC_ALPHA/ONE_MINUS_SRC_ALPHA blend. Calls `FUN_0053ab80(4, vbo, count, 1)`. `DAT_0ae7fc92` = SSO (separate shader objects) flag — if set, calls `(*DAT_0aea2d70)(mode)` instead of glUseProgram pair. |
| 0x0053a070 | **FindBestGLResourceSlot(stream, flags)** | Analyzed | Searches slot table `DAT_0ae7e9e8` (stride 0x20/slot, 0x188 entries per stream = 49 slots). Finds best available slot matching `flags` bitmask. Two search modes: bit2=0 → find partial match (most flags); bit2=1 → find exact match. Returns best slot index or -1 if none. Used by SetupMeshRenderSlot to allocate VAO/VBO resource slots. |
| 0x0053a460 | **AllocOrResizeVBO(vtxBytes, idxBytes, vbo, ebo, usage)** | Analyzed | Thread-safe VBO+EBO allocation/resize. If VBO handle==0: `(*DAT_0aea3398)(1, vbo)` = glGenBuffers. Binds vertex shader (0x8892) + fragment shader (0x8893), sets 5 attribs. Calls `(*DAT_0aea2308)(GL_ARRAY_BUFFER, size, NULL, usage)` = glBufferData. Same for EBO. usage=0x88e0=GL_DYNAMIC_DRAW. Thread dispatch via `FUN_00404b20` if off main thread. |
| 0x0053a800 | **UploadVBOData(vtx_ptr, vtx_count, idx_ptr, idx_count)** | Analyzed | Uploads vertex + index data to GPU. Calls `(*DAT_0aea2308)(GL_ARRAY_BUFFER, size, NULL, usage)` if buffer too small (grows to next power-of-2). Then `(*DAT_0aea2348)(GL_ARRAY_BUFFER, 0, size, data)` = glBufferSubData for vertex data. Same pattern for index EBO. Returns VBO handle. `DAT_0ae7fc68` flag selects 16-bit vs 32-bit indices. Thread-safe (SDL_LockMutex + cross-thread dispatch). |
| 0x0053ab80 | **GLDrawCall(primitive, count, idx_count, indexed)** | Analyzed | THE final GPU draw command. `indexed != 0`: calls `(*DAT_0aea2e28)(primitive, idx_count, GL_UNSIGNED_SHORT/INT, offset)` = **glDrawElements**. `indexed == 0`: calls `(*DAT_0aea2da0)(primitive, count)` = **glDrawArrays**. `DAT_0ae7fc68` flag picks index type: 0→GL_UNSIGNED_SHORT(0x1403), 1→GL_UNSIGNED_INT(0x1405). |
| 0x0053b020 | **SetupFrameUBOs()** | Analyzed | Per-frame uniform/UBO commit. Extracts fog color from `DAT_0ae7e9a0` (packed RGBA → R,G,B floats /255). Reads fog density from `DAT_006e3490`, fog matrix from `DAT_006e3480`. Iterates program list `DAT_0ae86180`: for each program calls `(*DAT_0aea6d38)(prog)` = glUseProgram/glBindProgramPipeline. Then `(*DAT_0aea6a00)(loc, fogDensity, fogMatrix)` or `(*DAT_0aea5750)(prog, loc, density, matrix)` = glUniform1f/glProgramUniform1f. Also `(*DAT_0aea6ab0)(loc, fogR, fogG, fogB)` = glUniform3f for fog color. Cached in `DAT_0ae7fcac`. |
| 0x00462090 | **FreePanelBatchBuffers()** | Analyzed | Frees panel quad batch GPU+CPU resources. glDeleteBuffers for VBO `DAT_00f40dcc` + EBO `DAT_00f40dd0`. Frees CPU staging `DAT_00f40dc0` + `DAT_00f40dd8`. Zeros `DAT_00f40dc8=0`. |
| 0x00462140 | **InitPanelBatchBuffers()** | Analyzed | Allocates panel quad batch buffers. `DAT_00f40dc8=0x80` (128 capacity). malloc(0x5c00) → `DAT_00f40dc0` (cpu quad buffer). malloc(0x1c00) → `DAT_00f40dd8` (group/bucket buffer). |
| 0x00491b00 | **FreeTextBatchBuffers()** | Analyzed | Frees text quad batch GPU resources. glDeleteBuffers for `DAT_016cf56c` + `DAT_016cf570`. Frees cpu `DAT_016cf560`. Zeros `DAT_016cf568=0`. |
| 0x00491b90 | **InitTextBatchBuffers()** | Analyzed | Allocates text quad batch. `DAT_016cf568=0x1000` (4096 cap). malloc(0xb0000) → `DAT_016cf560`. Then immediately calls FlushBatchedTextPolys() to init VBO. | ---

## Batch 11 — Printf Engine, BigNum, VFS, Progression, CSV Logging

### Custom Printf / vsnprintf Engine
> RVGL ignores the OS libc printf entirely and ships its own complete formatting stack to guarantee byte-identical cross-platform float output. Call tree: `SafeVsnprintf` → `vsnprintf` → `InternalFormatEngine` → output primitives + float engine + BigNum arena.

| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x005bd560 | **SafeVsnprintf()** | Analyzed | Top-level format wrapper. Clamps buffer to 255 bytes, writes, then UTF-8 heals the tail byte. Used by `WriteLogMessage` and `File_Vfprintf`. |
| 0x005acdf0 | **vsnprintf()** | Analyzed | Standard vsnprintf shim — delegates directly to `InternalFormatEngine`. |
| 0x005b27f0 | **InternalFormatEngine()** | Analyzed | Master printf formatting engine. Handles all format specifiers: `%d/%i/%u/%x/%o/%f/%e/%g/%a/%s/%c/%wc/%ws`. Dispatches to output primitives, the float sub-engine, and the BigNum path for exact decimal representation. |
| 0x005a9230 | **vfprintf_l() / PrintToStdout()** | Analyzed | Safe vfprintf wrapper with stream locking. Acquires file-stream critical section before calling the format engine. Used for all log output to stdout. |
| 0x00534640 | **LogWarning()** | Analyzed | Variadic warning emitter — formats a string with printf-style args and forwards to `WriteLogMessage`. |
| 0x00529490 | **WriteLogMessage()** | Analyzed | Master log dispatcher. Formats, sanitizes, and fans the message out to both the in-game console window and the log file. All game log output passes through here. |

### Printf Output Primitives
| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x00560fc0 | OutputNarrowString() | Analyzed | Writes a standard narrow C-string (`char*`) to the active format buffer. |
| 0x005b0d30 | OutputWideString() | Analyzed | Writes a wide-character string (`wchar_t*`) to the format buffer. |
| 0x005b0e90 | OutputNarrowChar() | Analyzed | Writes a single narrow character to the format buffer. |
| 0x005b0cd0 | OutputSingleChar() | Analyzed | Outputs one hardcoded character — used for padding/fill bytes. |
| 0x005b10e0 | WriteStreamData() | Analyzed | Flushes accumulated characters to the string output buffer or file stream. |
| 0x005b15e0 | FormatBase10Int() | Analyzed | Converts and writes standard base-10 decimal integers (`%d/%i`). |
| 0x005b10c0 | FormatHexOctalInt() | Analyzed | Converts and writes hex (`%x/%X`), octal (`%o`), or unsigned (`%u`) integers. |
| 0x005b61b0 | OutputDecimalPoint() | Analyzed | Prints a decimal point character using locale rules (`.` vs `,`). |
| 0x005b7020 | FormatThousandsSeparator() | Analyzed | Multibyte-to-Wide conversion for locale thousand-separator strings. |
| 0x005b7380 | _wctomb_internal() | Analyzed | Internal worker for `wctomb` — handles locale-aware wide→multibyte conversion. |
| 0x005b7410 | wctomb() | Analyzed | Public `wctomb` entry — converts one wide character to its multibyte sequence. |
| 0x00566c60 | wcsnlen() | Analyzed | Wide-string length with a maximum-count safety cap. |
| 0x00533aa0 | strlen() | Analyzed | Custom string-length measurer (replaces libc `strlen` to avoid OS libc dependency). |
| 0x00533880 | String_CompareIgnoreCaseUTF8() | Analyzed | Case-insensitive string comparison with proper UTF-8 codepoint handling (`strncasecmp` equivalent). |
| 0x00533f60 | ReadNextToken() | Analyzed | Lexer / token reader — reads the next whitespace-delimited token from a config text stream. *(Previously listed as `ReadLineFromFile`.)* |
| 0x005b7560 | get_iob() | Analyzed | Returns a pointer to the internal C `FILE` array (`_iob`). Used by the printf engine for stdout/stderr access. |
| 0x005b76d0 | _unlock_file() | Analyzed | Unlocks a file stream after critical-section usage in the printf/log system. |

### Float Formatting Sub-Engine
| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x005b0bb0 | ConvertFloatToString() | Analyzed | Core float→string math engine (`_fltout` equivalent). Entry point for all `%f/%e/%g` conversions. Delegates to BigNum for exact precision. |
| 0x005b1af0 | OutputFloatData() | Analyzed | Formatting wrapper that prints the actual float digit string, applying sign, exponent, padding, and width rules. |
| 0x005b2250 | FormatHexFloat() | Analyzed | Engine for `%a` and `%A` (hexadecimal float) format specifiers. |
| 0x005b3690 | _fptostr() | Analyzed | Core ASCII digit generator for mantissas (`_ftoa_engine` equivalent). Generates raw digit strings from IEEE 754 float bits. |
| 0x005b1010 | OutputNaNInf() | Analyzed | Safely outputs `"NaN"` or `"Inf"` strings when the float value is a special IEEE 754 case. |

### BigNum Arbitrary-Precision Engine
> To guarantee byte-identical cross-platform floating-point decimal output, RVGL ships a full arbitrary-precision math engine. It is used exclusively by the float→string path (`ConvertFloatToString` + `_fptostr`) to compute exact decimal representations without rounding drift. An arena allocator manages the BigNum pool to avoid heap fragmentation.

| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x005b5dc0 | **Bignum_Allocate()** | Analyzed | Allocates a Bignum structure from the pre-allocated pool. |
| 0x005b5ec0 | Bignum_Free() | Analyzed | Returns a Bignum structure to the linked-list free pool. |
| 0x005b5fe0 | Bignum_FromInt() | Analyzed | Creates a Bignum from a 32-bit integer value. |
| 0x005b6510 | Bignum_Compare() | Analyzed | Checks whether one Bignum is greater than another. |
| 0x005b5f30 | Bignum_Multiply() | Analyzed | Multiplies a Bignum by a 32-bit integer block. |
| 0x005b3510 | Bignum_ExtractDigit() | Analyzed | Performs long division on two Bignums and returns the top integer digit (core of decimal digit extraction). |
| 0x005b3470 | Bignum_StringAlloc() | Analyzed | Treats a Bignum block as a `char` array to store static digit strings during formatting. |
| 0x005b3430 | Bignum_BufferAlloc() | Analyzed | Allocates a dynamic scratchpad buffer during digit generation for intermediate results. |
| 0x005b56a0 | Bignum_ShiftRight() | Analyzed | Bitwise right-shift — divides a Bignum by a power of 2. |
| 0x005b6400 | Bignum_ShiftLeft() | Analyzed | Bitwise left-shift — multiplies a Bignum by a power of 2. |
| 0x005b57a0 | Bignum_CountTrailingZeros() | Analyzed | Bit-scan utility — counts trailing zero bits to determine lowest set bit. |
| 0x005b6200 | Bignum_MultiplyByPowerOf10() | Analyzed | Square-and-multiply algorithm combined with precalculated tables to scale by 10^n. |
| 0x005b60a0 | Bignum_MultiplyFull() | Analyzed | Multiplies two full Bignums together using schoolbook O(n²) multiplication. |
| 0x005b6560 | Bignum_Subtract() | Analyzed | General-purpose Bignum subtraction with handling for negative intermediate results. |
| 0x005b6730 | Bignum_ToDoubleNormalized() | Analyzed | Packs a Bignum back into a standard IEEE 754 64-bit double. High-performance BigNum→double packing utility. |

### Virtual File System (VFS) & Low-Level I/O
> The VFS resolves all asset paths between the read-only install directory and the writable user data directory. mount order determine which physical path wins. Strings are ultimately converted to UTF-16 before Windows API calls.

| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x00534c6d | **VFS_ResolvePath()** | Analyzed | Resolves a virtual game path to an absolute OS filesystem path by walking active mount points in order. |
| 0x00535f20 | CheckVirtualMount() | Analyzed | Checks whether a given path segment exists as an archive or mount entry. |
| 0x00536070 | VFS_FindFileInMounts() | Analyzed | Searches all active mount points for a file. Returns the mount index where the file was found, or -1. |
| 0x00534f80 | Engine_OpenFile() | Analyzed | Opens a file after VFS path resolution. Converts the resolved UTF-8 path to UTF-16 via `AllocUTF8ToWide` then calls Windows `CreateFileW`. |
| 0x005a8ef0 | OS_GetFileStat() | Analyzed | Thread-safe, Unicode-aware file status checker (`VFS_Stat`). Returns file size and timestamps. |
| 0x005a7c90 | AllocUTF8ToWide() | Analyzed | Allocates and converts a UTF-8 string to a `wchar_t*` wide string for Windows API consumption. |
| 0x00406d00 | Path_ReplaceExtension() | Analyzed | Safely replaces a file extension in a path string in-place. |

### Race Result CSV Logging
| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x00473c50 | SessionLog_Init() | Analyzed | Generates the session log filename and initializes the `.csv` output file at race start. |
| 0x00473f10 | ExportRaceResultsToCSV() | Analyzed | Result Export Dispatcher — writes the final race standings to a CSV file at race end. |
| 0x00460360 | LogMatchSettings() | Analyzed | Logs physics mode, lap count, weapon settings, and lobby details to the session CSV. |
| 0x004604c0 | LogRaceStandings() | Analyzed | Master CSV dumper — writes the full final standings table including positions, times, names. |
| 0x0044a2f0 | CalculateDNFStandings() | Analyzed | Extrapolates final positions and race times for cars that Did Not Finish (DNF). |

### Progression & Track Unlocks
| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x004541f0 | UpdateTrackUnlocks() | Analyzed | Track Availability Initializer — populates the `trackModesAvailable` bitmask for each track based on save data, difficulty completion, and active cheat flags. |
| 0x0044a250 | CheckIfDifficultyTierCompleted() | Analyzed | Progression checker — returns whether the player has completed all required races in a given difficulty tier. |
| 0x00449f50 | CheckIfTierTimeTrialsBeaten() | Analyzed | Checks whether all Time Trial star times have been beaten for a given tier. |
| 0x00452780 | GetTrackIdByName() | Analyzed | Converts a track folder name string to an integer track ID. Searches Vanilla tracks (IDs 0–20, static array) first, then Custom tracks (dynamic heap array, index offset by `×0x9D8`). |
| 0x00452130 | GetTrackInfo() | Analyzed | Takes a track index and returns a pointer to the `TrackInfo` struct for that track (0x78 / 120 bytes). |
| 0x0065a9b0 | TimeTrial_InitChallengeTimes() | Analyzed | Hardcodes the official Time Trial challenge target times for all Vanilla tracks at startup. |

### UI Popup System
| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x00468200 | UI_PopupDispatcher() | Analyzed | Saves popup text into global memory and calculates a proportional popup panel width. Used for in-race notifications, error messages, and countdown popups. | ---

## Key Technical Notes (Batch 11)

### The "21th" Ordinal Bug
During UI string formatting for race position ordinals, the developers hardcoded special cases for 1st/2nd/3rd but failed to wrap the modulo check for positions above 20. As a result:
- Position 21 → `"21th"` (should be `"21st"`)
- Position 22 → `"22th"` (should be `"22nd"`)
- Position 23 → `"23th"` (should be `"23rd"`)
The fix is: apply `pos % 10` (not raw `pos`) to select the suffix, but only when `pos % 100` is not 11/12/13.

### Vanilla vs. Custom Track Arrays
RVGL actively tracks two separate track tables:
- **Vanilla** (IDs 0–20): stored in a static memory array; accessed directly by index.
- **Custom** (user-added): dynamically allocated on the heap; indices are offset by `×0x9D8` to bridge the gap seamlessly.
`GetTrackIdByName()` searches Vanilla first, then Custom, using pointer arithmetic to unify the lookup.

### Custom C Runtime & BigNum Engine
To guarantee byte-identical cross-platform output, RVGL ignores the OS-provided `printf`. Instead it ships:
1. **InternalFormatEngine** — a complete custom `printf`/`vsnprintf` implementation
2. **BigNum Arena** — a full arbitrary-precision math library used only to compute exact decimal digit strings for floating-point numbers
3. **Arena Allocator** — `Bignum_Allocate/Free` use a linked free-list pool to avoid runtime heap fragmentation during output

### Virtual File System Design
The VFS resolves paths between two roots:
- **Read-only install dir** — core game assets (`.w`, `.prm`, `.bmp`, etc.)
- **Writable user dir** — `profiles/`, `replays/`, `cache/`, `times/`

Mount order determines which root wins for a given path. All resolved paths are converted to UTF-16 (`AllocUTF8ToWide`) before Windows API calls, ensuring Unicode support.

### Graceful Modding & Job Objects
When using external injectors like `RVGLAutoInjector.exe`, wrap child processes in a Windows **Job Object** (`ChildProcessTracker` pattern) or add a `WaitForSingleObject` heartbeat. This automatically terminates the injector/tool when the main game process exits, preventing ghost processes.




---

## Section 2: Memory Map and Data Structures

# RVGL Memory Map & Data Structures
**Last Updated:** February 2026 — Batch 10 / Full 222k-line rvgl.exe.c pass  
**Architecture:** x64 Windows

---

## Table of Contents
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
0x006e30a0 | 8    | function* | GameStateFunction     | Current game mode
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
0x006fab50 | 8     | CarInfo*     | CarArrayPtr           | POINTER to heap-alloc'd CarInfo array (NOT the array itself!)
0x006fab58 | 4     | int          | CarCount              | Number of cars in pool (starts 0x31 = 49, grows with user cars)
0x006fab80 | 2732  | RuntimeCar   | RuntimeCarTemplate    | 0xAAC-byte template car — copied by InitRuntimeCarStruct (0x0043c580)
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
0x6814c020 | 4    | DWORD     | TLS_INDEX             | Windows TLS slot (-1 = uninitialized)
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
0x00499690  →  RacingGameLoop()          THE RACING LOOP
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
0x006e3b00 | 8     | void*   | ZoneLookupTable          | Spatial zone-to-waypoints table (stride 0x10 per zone)
0x006e3b08 | 8     | void*   | WaypointDataTable        | Waypoint section lookup array (stride 0x78)
0x006e3b10 | 4     | int     | TotalWaypointNodeCount   | Total count of waypoint nodes (used as modular wrap)
0x006e3b14 | 4     | int     | WaypointZoneCount        | Count of spatial zones in lookup table
0x006fd148 | 8     | void*   | GlobalDefaultCarPtr      | Default/global car entity for AI path ops
```

### Race Session Config Globals (Batch 4)
> The race is initialized by `FUN_00450700` (RaceSetup) big switch on `GameMode`.
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x006e34c0 | 4     | int     | GameMode                 | Race mode: 1=single race, 2=multi, 3=practice/TT, 4/6=online lobby
0x006e34c4 | 4     | int     | GameModeVariant          | Sub-mode variant (1 = restarting / spectate)
0x006e34c8 | 16    | char[]  | TrackPathString          | Current track path/name string (16 chars)
0x006e34cc | 4     | int     | PlayerCarModelIndex      | Player's selected car model index (into CarModelDatabase)
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
0x0ae8e948 | 16    | char[]  | LocalPlayerName          | Local player name string (16 chars)
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
0x0ae6f6ac | 4     | int     | LocalPlayerNetworkID     | Local player's network session ID
```

### Race Entry Table Globals (Batch 4)
> `FUN_0044f1f0` = AddRaceEntry. `FUN_0044f7b0` = SetupAllRaceCars.  
> Each session has up to 30 entries (max `_DAT_00f3eb04 < 0x1e`).
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
_0x00f3eb00| 4     | int     | PlayerSlotIndex          | Player's slot index in race entries (0-based; -1=not in race)
_0x00f3eb04| 4     | int     | TotalRaceEntries         | Total car entries built for this race
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
0x00f3eb50 | 0x50  |struct[] | RaceEntryTable           | Array of RaceCarEntry (stride 0x50 = 80 bytes, max 30)
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
0x0acee7e8 | 8     | void*   | LocalPlayerCarPtr        | The LOCAL PLAYER's car entity (set by FUN_0044f7b0 when is_local != 0)
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
0x006e3ae0 | 4     | float   | TotalRouteLength         | Total track route length (lap length in route units)
0x006e3ae4 | 4     | int     | DefaultRouteSectionIndex | Default RouteSection index (car+0x6910 init)
0x006e3ae8 | 4     | int     | WaypointSystemReady      | Gate flag: !=0 → UpdateCarRoutePosition runs
0x006e3aec | 4     | int     | SecondaryRouteSectionIdx | Secondary route section for multi-path tracks
0x006e3af0 | 8     | void*   | RouteSectionTable        | Array of RouteSection nodes (stride 0x50 = 80 bytes)
0x006fab50 | 8     | void*   | CarModelDatabase         | Per-car model DB (stride 0x110 = 272 bytes, max 6 cars)
0x006fd140 | 1     | byte    | PlayerCarActiveFlag      | 1 = player car slot is active/valid
0x006fd141 | 1     | byte    | PlayerCarInitializedFlag | 1 = player car needs init/reinit
0x006fd150 | 8     | void*   | ActiveGhostPathPtr       | Current ghost path data ptr (into GhostReplayBuffer)
```

### Car Entity Pool / Linked List Globals (Batch 4)
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
_0acee7e4  | 4     | int     | TotalActiveCarCount      | Count of cars currently alive (++/-- by Create/Destroy)
0x0acee9e0 | 8     | void*   | CarListTail              | Tail of active car linked list (oldest car added)
0x0acee9e8 | 8     | void*   | CarListHead              | Head of active car linked list (most recently added)
0x0adb69b0 | 8     | void*   | FreeCarPoolHead          | Head of free/pooled car entity list
```

### Ghost / Lap Replay Globals (Batch 4)
```
Address    | Size  | Type    | Name                     | Description
-----------|-------|---------|--------------------------|-------------------------------------------
0x0065fca8 | 8     | PTR     | PTR_GhostBufCurrent      | Ptr to current ghost replay buffer (creation params)
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
0x0acee800 | 240   | struct  | RaceResultTable          | Sorted race results array (stride 0x10, max 30 entries)
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

## New Globals Discovered (Batch 3 — Full File Pass)

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
0x0065c03c | 4    | float   | DeltaTime                | Scaled delta time (seconds/frame) — used by ALL physics
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
0x006e39c0 | n*8  | void*[] | SortedRacePositions      | Car pointer array sorted by race position
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
0x0adccb08 | 8     | void*   | MultiplayerCarArray      | Base of NetworkCarEntry array (stride 0x1e0)
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

## New Globals Discovered (Batch 2)

### Race State
```
Address    | Size | Type    | Name                   | Description
-----------|------|---------|------------------------|-------------------------------------------
0x016942dc | 4    | int     | RaceState              | Race state 0-6. Read by GetRaceState (0x0048a5c0), written by RaceInit2 (0x0048a560)
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
0x006fab50 | 8     | CarInfo* | CarArrayPtr            | Pointer to heap-alloc'd CarInfo array
0x006fab58 | 4     | int      | CarCount               | Car count (49 + user cars). Growth via realloc in LoadUserCars
0x006fab80 | 2732  | Runtime  | RuntimeCarTemplate     | 0xAAC-byte template copied by InitRuntimeCarStruct
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
0x0ae8e8c8 | 8     | void*    | WinnerCarPointer       | Pointer to the winner's runtime car struct
0x0ae8e905 | 1     | byte     | ChampionshipContinue   | Flag: continue championship or end
0x0ae8e907 | 1     | byte     | ChampionshipWin2       | Secondary championship win flag
0x00662b20 | 4     | float    | UITimerFloat           | Countdown/display float (reset to 0x3f59999a = 0.85)
```

### Timing / Delta Time
```
Address    | Size  | Type     | Name                   | Description
-----------|-------|----------|------------------------|-------------------------------------------
0x0065c03c | 4     | float    | DeltaTime              | Seconds per frame. Written by CalcDeltaTime every frame
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
0x0ae7fc4c | 1     | byte     | NetworkModeActive      | Non-zero = network/MP mode active (skips rendering stubs)
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
0x0acee9e8 | 8     | void*    | RuntimeCarListHead     | Head of linked list of runtime car structs
                                                          | Each node: [+4]=type, [+0x10]=next ptr, [+0xf90]=timer
0x0acee7e8 | 8     | int*     | PlayerCarSlotPtr       | Pointer to the PLAYER car slot (car 0).
                                                          | InitCarSlotFull checks if param_1==this to detect car-0.
                                                          | When set: copies template to DAT_006fa0a0
0x0acee7e8 | 8     | int*     | [same as above]        | (DAT_0acee7e8 = linked list containing player car)
0x006fa0a0 | 2732  | RuntimeCar | PlayerCarDataCopy    | Copy of player's current car data (0xAAC bytes) stored apart
0x006fab44 | ?     | ?        | PlayerCarDataTail      | Tail/suffix of player car copy
0x016f8e40 | 8     | void*    | CamEntityPtr_0         | Pointer to camera entity slot 0 (split-screen cam). Read by FUN_004a4910.
                                                          | Points into inline array at 0x016f8ee0. Cleared by FUN_004a5500.
0x016f8e48 | 8     | void*    | CamEntityPtr_1         | Pointer to camera entity slot 1
0x016f8e50 | 4     | void*    | CamEntityPtr_2         | Pointer to camera entity slot 2
0x016f8e54 | 4     | (pad)    | CamEntityPtr_2_hi      | High dword of above (8-byte ptr)
0x016f8e58 | 8     | void*    | CamEntityPtr_3         | Pointer to camera entity slot 3
0x016f8e60 | 8     | void*    | CamEntityPlayerPrimary | Pointer to PLAYER primary camera entity.
                                                          | FUN_004a5ce0 checks (param_1 == DAT_016f8e60) to detect player cam.
                                                          | FUN_004a1290 same check. 259 XREFs. Cleared to 0 by FUN_004a5500.
0x016f8e68 | 8     | void*    | CamEntityPtr_MiniA     | Pointer to minimap/sub-viewport entity A
0x016f8e70 | 8     | void*    | CamEntityPtr_MiniB     | Pointer to minimap/sub-viewport entity B
0x016f8e78 | 8     | void*    | CamEntityPtr_MiniC     | Pointer to minimap/sub-viewport entity C
0x016f8e80 | 8     | void*    | CamEntityPtr_Extra     | Pointer to 9th camera entity (extra)
0x016f8ec0 | ?     | ?        | ItemStateArray         | Item/pickup state array (ItemSystemUpdater: DAT_016f8ec0 + idx*0x28)
0x016f8ee0 | 9×320 | CamEnt[] | CameraEntityArray      | Inline static camera entity array.
                                                          | 9 entries × 0x140 (320 bytes) each. Ends at 0x016f98e0.
                                                          | Each slot: ptr[-8]=0xFFFFFFFFFFFFFFFF (ID/handle), state at +0x12c,
                                                          | viewport index at +0x128, viewport rect at +0x100..+0x124.
                                                          | Type field at [0]=2 for camera, [1]=subtype.
                                                          | Reset by FUN_004a5500; initialized by FUN_004a5ce0.
0x016f9a44 | 4     | int      | ItemActiveViewport     | Item system viewport index (ItemSystemUpdater)
0x016f9a48 | 16    | float[4] | ViewportFOVScaled      | Scaled FOV values (written by CameraEntityViewportUpdate)
0x016f9a54 | 4     | float    | ViewportDepthA         | Viewport depth param A (written by FUN_004a4910)
0x016f8e95 | ?     | (pad)    | [CamNodeArrayPad]      | (between camera entity ptrs and node array)
0x016f8e98 | 4     | int      | PickupNodeCount        | Count of entries in node array at 0x016f8ea0. Used by FindNearestPickupNode.
0x016f8ea0 | n×56  | Node[]   | PickupNodeArray        | Camera attachment / pickup node array. Stride 0x38 (56 bytes) per node.
                                                          | Node layout: [0]=type(int), [3..5]=XYZ floats, [0xc]=bitmask(uint64).
                                                          | FindNearestPickupNode (0x004a5b50) iterates this with dist^2 nearest-check.
0x016e4428 | 8     | uint64   | NodeBitmaskFilter      | Bitmask applied in FindNearestPickupNode: skip nodes where (node[0xc] & this) != 0
0x016f9a80 | 8     | ?        | ViewportExtra          | Extra viewport data (written to cam+0x120 by FUN_004a4910)
0x016f9aa0 | ?     | VpRect[] | ViewportRectTable      | Split-screen viewport table. `0x016f9aa0 + idx*0x28` per viewport.
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
0x00f3eb00 | 4     | int      | RacePlayerIndex        | Player's own race index (0-based) in participant table
0x00f3eb04 | 4     | int      | RaceParticipantCount   | Live race participant count (max 0x1e = 30)
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
0x00f3eb70 | 30×80 | Part[]   | RaceParticipantTable   | Race participant entries. Stride 0x50 (80 bytes) per entry.
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
>  CONFIRMED: 0xAAC-byte structs are initialized by FUN_004ab810 (InitCarSlotFull) and
> template-copied by FUN_0043c580 (InitRuntimeCarStruct). The camera entity array at
> 0x016f8ee0 (stride 0x140) is separate — that is the CAMERA ENTITY SYSTEM, not cars.
> The flat car array location is still unknown; find it via FUN_0044f1f0 / FUN_0044f4f0.
```
int* slot:
  slot[0]    byte+0x00  = car array index
  slot[1]    byte+0x04  =  CAR TYPE: 1=player, 9=spectator/disabled, 4=special
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

## Batch 5 — Main Loop, Audio, Race State Machine

### ALSourceEntry (stride 0x18 = 24 bytes)
> Pool of up to 0x80 (128) AL sources stored at `DAT_0ae4c640`. Each entry tracks
> the playing sound's rank, SFX index, play timestamp, and AL source handle.
```c
struct ALSourceEntry {               // stride 0x18 (6×int = 24 bytes)
    int   rank;                  // +0x00  rank score (lower = preemptible)
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
    int   rank;          // +0x04  (= param_2 rank)
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
0x006e30a0 | 8     | code*   | GameStateFuncPtr         | Current per-frame handler; called every frame
0x006e30a8 | 1     | byte    | QuitRequestedFlag        | 1 = SDL_QUIT received → loop exits
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
0x0ae48de0 | 8     | uint64  | PerfFrequency            | SDL_GetPerformanceFrequency() result (ticks/second)
0x0ae48de8 | 8     | uint64  | LastPerfCounter          | Performance counter at previous frame
0x0ae48df0 | 8     | uint64  | CurrentPerfCounter       | Current SDL_GetPerformanceCounter() value
0x0ae48dd0 | 8     | uint64  | FrameTicks               | Raw FrameTicks = CurrentPerfCounter - LastPerfCounter (capped at 100ms)
0x0ae48dd8 | 8     | uint64  | FrameTicksRaw            | Uncapped raw frame ticks (for stats)
0x0065c038 | 4     | float   | RawDeltaTime             | Unscaled dt = FrameTicks / PerfFrequency (float)
0x0065c03c | 4     | float   | DeltaTime                | Final dt (possibly slowmo-adjusted) — used by ALL update code
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
0x016942dc | 4     | int     | RaceStateMachine         | Current race state (0-6)
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
0x016f7901 | 1     | byte    | RaceFinishedFlag         | 1 = player has finished all laps
0x016f7900 | 1     | byte    | RaceModeMPFlag           | 1 = multiplayer race (affects next state)
0x016f7904 | 4     | int     | TotalLaps                | Total lap count for this race
0x016f7908 | 4     | int     | CurrentLap               | Current lap number (0-indexed)
0x016f790c | 4     | float   | LapTimer                 | Lap timer (seconds, += DeltaTime; reset on new lap)
0x016f9a60 | 8     | void*   | RaceViewportPtr          | Pointer to viewport/camera struct (passed to FUN_004a4120)
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
> for music. Up to 128 simultaneous AL sources with rank-based preemption.
```
Address    | Size   | Type    | Name                     | Description
-----------|--------|---------|--------------------------|-------------------------------------------
0x006e3434 | 1      | byte    | NoSoundFlag              | 1 = audio disabled (-nosound cmdline)
0x0ae48e24 | 4      | int     | SfxMasterVolume          | SFX master volume (0-10000, default 10000)
0x0ae48e28 | 4      | int     | TotalALSources           | Count of AL sources created (max 0x80 = 128)
0x0ae48e34 | 1      | byte    | AudioPausedFlag          | 1 = audio paused (on focus loss)
0x0ae48e40 | ?      | int[2]  | LoopSfxBaseInts          | Start of LoopSfxHandle[0] sfx_id + rank
0x0ae48e50 | 0x3800 | struct[]| LoopSfxArray             | Array of LoopSfxHandle (stride 0x38, max 0x100 = 256)
0x0ae4c640 | 0xC00  | struct[]| ALSourcePool             | Array of ALSourceEntry (stride 0x18, max 0x80 = 128)
0x0ae4c650 | 4      | uint    | ALSource0_ID             | First AL source ID (= ALSourcePool[0].al_source_id)
0x0ae4d240 | 0x400  | uint[]  | SfxBufferArray           | OpenAL buffer IDs per SFX slot (0x100 int = 256)
0x0ae4da40 | 0x10000| char[]  | CustomSfxNameTable       | Custom SFX file names (stride 0x100 per slot, 0x100 slots)
0x0ae4db3f | 1      | byte    | CustomSfxNullTerms       | Null terminator bytes within custom SFX table
0x0ae5da40 | 256    | char[]  | CurrentMusicDir          | Current music directory path string
0x0ae5db40 | ?      | char[]  | MusicFileList            | Sorted music file names (stride 0x100, up to 0xFF entries)
0x0ae6db40 | 1      | byte    | MusicCycleFlag           | 1 = music was interrupted (DAT_0ae6db40)
0x0ae6db41 | 1      | byte    | MusicCustomFlag          | 1 = music from custom track (not redbook)
0x0ae6db44 | 4      | int     | MusicFileCount           | Number of music tracks found in current dir
0x0ae6db48 | 4      | uint    | MusicALSourceID          | AL source handle for BGM/music stream
0x0ae6db50 | 8      | void*   | MusicStreamPtr           | Active music stream object (OGG/WAV stream)
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
    int    ghost_rank;      // +0x170  0=normal, 1=secondary
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






