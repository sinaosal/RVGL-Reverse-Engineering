# RVGL Master Function Index
**Last Updated:** March 2026 — Batch 11 / Printf Engine + BigNum + VFS + Progression  
**Total Functions Documented:** 310+

---

## 📋 Quick Navigation
- [RVGL.exe Functions](#rvglexe-functions) (42 functions)
- [libGLESv2.dll Functions](#libglesv2dll-functions) (25 functions)
- [Memory Addresses](#memory-addresses) (60+ addresses)
- [SDL2.dll Imports](#sdl2dll-imports)

---

## RVGL.exe Functions

### 🎮 Core Game Loop (Priority: CRITICAL)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x004014c0 | entry() | func_entry.txt | ✅ Analyzed | Program entry point |
| 0x00402ec0 | MainGameLoop() | maingameloop.txt | ✅ Analyzed | Main entry, command-line parsing, game loop |
| 0x00402980 | ProcessSDLEvents() | func_00401500.txt | ✅ Analyzed | SDL event processor (input handling) |
| 0x00401610 | FrameUpdateDispatcher() | RGameModeManagement/func_00401610.txt | ✅ Analyzed | Per-frame timing dispatch: calls delta-time, updates accumulator `DAT_0ae48dc8`, iterates linked list at `DAT_0acee9e8` (stride `lVar3+0x10`) adding delta to `+0xf90`. Handles game-mode branch via `DAT_006e34c0 & 0xFFFFFFFD == 4` |

### 🏁 Game State Functions (Priority: CRITICAL)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00499b30 | SplashScreenState() | gameplay/lab_00499b30.txt | ✅ Analyzed | Splash/logo screen, transitions to racing |
| 0x00499690 | **RacingGameLoop()** | **[NOT EXPORTED YET]** | ⚠️ **ULTRA PRIORITY** | **Main racing loop - physics, AI, rendering!** |
| 0x00543890 | MultiplayerLobbyState() | gameplay/lab_00543890.txt | ⏳ Partial | Multiplayer/lobby mode |
| 0x00406b40 | UnknownGameState() | gameplay/lab_00406b40.txt | ⏳ Partial | Another game mode (menu?) |
| 0x0044c130 | GameGaugeReplayState() | RGameModeManagement/func_0044c130.txt | ✅ Analyzed | Loads `game_gauge_replay.rpl`, sets `DAT_0ae8e906=1`. Copies `DAT_00f3e140` block (0x135 qwords) to `DAT_00f3eb00`. Sets game mode `DAT_0ae8e920=5`, `DAT_006e34c0=5`. Reads track index from `FUN_00452780`. Sets game-state fn ptr `DAT_006e30a0 = &LAB_00406b40` |

### 🚗 Car System (Priority: HIGH)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x0043f140 | LoadAllCars() | game_initialization/func_0043f140.txt | ✅ Analyzed | Main car loading loop (49 cars) |
| 0x0043b6c0 | ParseParametersTxt() | game_initialization/func_0043b6c0.txt | ✅ Analyzed | Parse car parameters.txt (16 keywords) |
| 0x005bd590 | InitializeCarStructure() | game_initialization/func_005bd590.txt | ✅ Analyzed | Set default car values |
| 0x0043ba50 | FinalizeCarLoad() | [NOT EXPORTED] | ⏳ Not Analyzed | Post-processing after car load |

### 🏎️ Track System (Priority: HIGH)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00452190 | LoadAllTracks() | game_initialization/func_00452190.txt | ✅ Analyzed | Main track scanning loop |
| 0x00451da0 | ParseTrackInf() | game_initialization/func_00451da0.txt | ✅ Analyzed | Parse track .inf file (6 keywords) |
| 0x005370e0 | CheckDirectoryExists() | [NOT EXPORTED] | ⏳ Not Analyzed | Verify levels/[name]/ exists |
| 0x005370a0 | CheckFileExists() | [NOT EXPORTED] | ⏳ Not Analyzed | Verify .inf file exists |

### ⚙️ Initialization Functions (Priority: MEDIUM)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00536bb0 | InitSubsystem1() | game_initialization/func_00536bb0.txt | ✅ Analyzed | Unknown initialization |
| 0x00529310 | InitSubsystem2() | game_initialization/func_00529310.txt | ✅ Analyzed | Unknown initialization |
| 0x004ee170 | InitSubsystem3() | game_initialization/func_004ee170.txt | ✅ Analyzed | Unknown initialization |
| 0x0044bae0 | InitSubsystem4() | game_initialization/func_0044bae0.txt | ✅ Analyzed | Major subsystem init |
| 0x00452280 | PostCarLoadInit() | [NOT EXPORTED] | ⏳ Not Analyzed | Called after cars loaded |
| 0x0043fac0 | LoadUserCars() | RuntimeCarArray1/func_0043fac0.txt | ✅ Analyzed | Scans `cars/` dir via `_wfindfirst64`, `realloc`s `DAT_006fab50` to grow car pool beyond 49. Sorts entries by class rating. `DAT_006e3431` = skip-user-cars flag |
| 0x0044bb80 | UnknownInit2() | [NOT EXPORTED] | ⏳ Not Analyzed | Called during setup |

### 🎬 Race Initialization (Priority: HIGH)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x004995f0 | RaceSetup() | race_initialization/func_004995f0.txt | ✅ Analyzed | Called before racing loop — loads GFX for current lap |
| 0x004889c0 | RaceInit1() | race_initialization/func_004889c0.txt | ✅ Analyzed | Race initialization function |
| 0x0048a560 | RaceInit2() | race_initialization/func_0048a560.txt | ✅ Analyzed | Race initialization function |

### �️ AI & Physics System (Priority: CRITICAL — NEW Batch 3)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00408a60 | CarPhysicsUpdate_AIBrain() | [in rvgl.exe.c line 4912] | ✅ Analyzed | **THE AI brain (1517 lines)**. Per-car physics + 13-state AI state machine. Reads/writes 80+ CarEntity fields. Computes steering/throttle output at `+0x34/+0x36`. Updates race position. Calls avoidance, rubber-band, stuck detection. |
| 0x004071c0 | AI_Avoidance() | [in rvgl.exe.c line 4021] | ✅ Analyzed | Core steering computation from `+0x6924` (abs_steer) and `+0x6934` (lateral deviation). Manages reverse state byte `+0x69d0`, sets timer 4.0f. `DAT_006e343f` = global reverse test flag. |
| 0x00407400 | InitAI_State() | [in rvgl.exe.c line ~4080] | ✅ Analyzed | Initialize AI state on car spawn. Resets AI fields, calls `FUN_0040ff20` + `FUN_0040e150`. `DAT_006e3ae4` = initial waypoint index. |
| 0x004074a0 | ResetAIBehavior() | [in rvgl.exe.c line 4124] | ✅ Analyzed | Reset all AI nav vars to defaults. lookahead=600f, max_steer=50f, angle=100f, height=150f, track_room=300f, engine_power=0.5f. |
| 0x004075f0 | UpdateRacePositions() | [in rvgl.exe.c line 4169] | ✅ Analyzed | Insertion-sort all cars by race score. Score = `(lap * DAT_006e3ae0) - track_t`. Updates `DAT_006e39c0[]` sorted array, sets `car+0x69c0` = rank. |
| 0x00407930 | AI_StuckDetection() | [in rvgl.exe.c line 4322] | ✅ Analyzed | Detect stuck/stranded cars. Uses rubber-band config table at `PTR_DAT_0065c100`. Sets `car+0x38 \|= 0x40` if stuck. Forces AI state 3 if `stuck_slow_timer` exceeds threshold. |
| 0x00407cb0 | CompareCarPassing() | [in rvgl.exe.c line 4604] | ✅ Analyzed | Bool comparison for car overtaking. Reads path node types from `car+0x28/+0x30`. Returns whether target car is ahead of self. |
| 0x004c84a0 | PerFramePhysicsWorldStep() | [in rvgl.exe.c line 86109] | ✅ Analyzed | Main per-frame physics dispatcher. Calls UpdateRacePositions, decrements `car+0x104c` cooldown, calls `FUN_004f95b0`+`FUN_00488ad0`. Also updates game object linked list at `DAT_0ace4800`. |
| 0x004ce3b0 | CarSwapAttach() | [in rvgl.exe.c line ~86190] | ✅ Analyzed | Car entity swap/attach to physics body. |
| 0x004ce590 | UpdateCarGhostTransparency() | [in rvgl.exe.c line 86201] | ✅ Analyzed | Sets `DAT_0ace49a0 = 0.5f` ghost alpha. Per camera slot: sets `slot+0xc0 = 0.5f` fade alpha. |
| 0x004d0c70 | AnimationUpdate() | [in rvgl.exe.c line 86257] | ✅ Analyzed | **Per-object keyframe animation update** (NPC/pedestrian system). Reads animation state struct at `obj+0x338`. 16 bone channels (loop 0..0xf, stride 0x6c). Easing modes: linear/ease-in/ease-out/smoothstep/bounce. Loop modes: loop/play-once/ping-pong. |
| 0x004d1f20 | LoadAnimationData() | [in rvgl.exe.c line ~86850] | ✅ Analyzed | Zeros keyframe data tables, inits audio slot table to -1. Loads animation definitions from files into DAT_01815xxx arrays. Max 16 animations, 16 channels each. |
| 0x004b35b0 | MeshVertexOBBCull() | [in rvgl.exe.c line 75371] | ✅ Analyzed | Mark mesh vertices visible/occluded via OBB test. Vertex stride 0x58. Mesh: `+0x2c`=vert_count, `+0x58`=vert_buffer. Vis flag at vertex+0x54 (0=hidden, FLT_MIN=visible). |
| 0x004b3830 | MeshVertexOBBCull_Variant() | [in rvgl.exe.c line 75450] | ✅ Analyzed | Alternate version of MeshVertexOBBCull. |
| 0x00499c40 | GaussianElimination() | [in rvgl.exe.c line 63600] | ✅ Analyzed | Solves AxB=C linear system for physics constraints/suspension. Matrix stored as `[row*0x20+col]` float array. |
| 0x0049a3e0 | ConjugateGradientSolver() | [in rvgl.exe.c line ~64200] | ⏳ Partial | Iterative SIMD-unrolled conjugate gradient solver. Used for physics/suspension. Large function. |

### 🗺️ Level Editor System (Priority: MEDIUM — NEW Batch 3)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00418900 | EditorModeDispatch() | [in rvgl.exe.c line 12838] | ✅ Analyzed | Dispatches to 10 editor sub-modes based on `DAT_006e3500`. |
| 0x00418a20 | EditModeUpdate() | [in rvgl.exe.c line 13002] | ✅ Analyzed | Main editor per-frame update. Calls sub-mode handlers, always calls `FUN_004931f0` (HUD text centering). |
| 0x00418d60 | ShowStatusMessage() | [in rvgl.exe.c line ~13100] | ✅ Analyzed | Display status message in editor mode. |
| 0x00418d90 | ScreenToWorldRay() | [in rvgl.exe.c line ~13120] | ✅ Analyzed | Convert screen-space mouse coords to world-space ray using view matrix. |
| 0x00418eb0 | WorldPointVisible() | [in rvgl.exe.c line ~13160] | ✅ Analyzed | Test if a world-space point is within the view frustum. |
| 0x00414c60 | AINodeBoxRender() | [in rvgl.exe.c line 10551] | ✅ Analyzed | Renders the AI node bounding box visualization in editor. |
| 0x004236b0 | WireframeBoundingBoxRender() | [in rvgl.exe.c line ~15800] | ✅ Analyzed | Draws wireframe bounding box for selected track section. |
| 0x0042e8c0 | TrackSectionDelete() | [in rvgl.exe.c line 19372] | ✅ Analyzed | Delete a track section in editor mode. Updates `DAT_006f9e6c` (track section count). |
| 0x004931f0 | RenderCenteredHUDText() | [in rvgl.exe.c — UNDECOMPILEABLE] | ❌ Timeout | Ghidra decompile timeout. Renders centered HUD text at X offset. Called by EditModeUpdate. |

### 🚗 Car Entity System (Priority: CRITICAL — NEW Batch 4)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x004f03a0 | CreateCarEntity() | [in rvgl.exe.c line 93015] | ✅ Analyzed | **Creates car entity from free pool**. Args: (type, subtype, model, spawn_quat, pos_data, rot_matrix). Inserts into `DAT_0acee9e8` linked list. Sets `car+0x40`=physics_body, calls `FUN_004ab810`+`FUN_004a76f0`+`FUN_004efd50`+`FUN_00407400`+`FUN_004074a0`. Increments `_DAT_0acee7e4`. |
| 0x004f0630 | DestroyCarEntity() | [in rvgl.exe.c line 93123] | ✅ Analyzed | Frees car entity back to pool. Calls FUN_004a0ac0+FUN_004aa0b0+FUN_004ec470. Removes from linked list, returns to `DAT_0adb69b0`. Decrements `_DAT_0acee7e4`. |
| 0x004f06d0 | DestroyAllCarEntities() | [in rvgl.exe.c line 93145] | ✅ Analyzed | Destroys all cars in active linked list. Iterates `DAT_0acee9e8` → +0x10 chain. |
| 0x004f0850 | FindCarByNetworkID() | [in rvgl.exe.c line 93223] | ✅ Analyzed | Searches active car list for `car+0x6a84 == param1`. Returns car ptr or 0. O(n) walk of linked list. |
| 0x004f0890 | RegisterFinishTime() | [in rvgl.exe.c line 93239] | ✅ Analyzed | Record race finish: insertion-sort into `DAT_0acee800` leaderboard (max 30). Sets `car+0x6a48`=finish_time_ms, `car+0x6a4c`=position. Prints lap time (MM:SS:mmm format). |
| 0x004f0af0 | SetAllCarsBoostState() | [in rvgl.exe.c line 93348] | ✅ Analyzed | Sets `physics+0x340=LAB_005234f0` for all non-(type 0,4,9) cars. Used for boost/speed-start?. |
| 0x004f0b50 | RestoreAllCarsStartState() | [in rvgl.exe.c line 93370] | ✅ Analyzed | Restores `physics+0x340..+0x34c` from `physics+0x458..+0x464` (saved state rollback). |
| 0x004f0ba0 | SetCarInvincible() | [in rvgl.exe.c line 93408] | ✅ Analyzed | Sets `car+0x6a44=1`, clears `car+0x6a38..+0x6a40`. Makes car invincible. |
| 0x004f0bc0 | ClearCarInvincible() | [in rvgl.exe.c line 93415] | ✅ Analyzed | Sets `car+0x6a44=0`. Removes invincibility. |
| 0x004f1560 | DispatchTriggerHandlers() | [in rvgl.exe.c line 93517] | ✅ Analyzed | Iterates `DAT_0ace4800` list, calls fn ptr at `entity+0x358` for each entity. |
| 0x004f1910 | LoadTriggersFromFile() | [in rvgl.exe.c line 93543] | ✅ Analyzed | Reads trigger file: `DAT_0adb6cc8`=count, `DAT_0adb6cc0`=malloc(count×0x68). Each entry 0x68 bytes (active+file_data+dispatch_fn). |
| 0x0044cb60 | InitPlayerCar() | [in rvgl.exe.c line 33712] | ✅ Analyzed | Creates/reinitializes player's car. Uses `PTR_DAT_0065fca8` (params), calls `FUN_004f03a0(4,0,model,...)`. Sets `car+0x67d4=0`, `car+0x690d=1`, `car+0x6910=DAT_006e3ae4`, `car+0x6914=0`. Calls `FUN_0040ffb0`+`FUN_0040ff20`+`FUN_0040e150`. |
| 0x0044ce10 | ResetPlayerCarBuffer() | [in rvgl.exe.c line 33801] | ✅ Analyzed | Frees player car via `FUN_004f0630`. Clears params buffer. Sets `DAT_006fd140=0`, `DAT_006fd148=0`. |
| 0x0044cec0 | RecordGhostFrame() | [in rvgl.exe.c line 33855] | ✅ Analyzed | Records current car position/orientation into ghost replay buffer `PTR_DAT_0065fcb0`. Stores quaternion XYZ (`physics+0x14..+0x1c`) + color RGBA bytes + speed. Stride 0x18/frame. Max 0x7FFF frames. |
| 0x0044d040 | CommitGhostLapFrame() | [in rvgl.exe.c line 33934] | ✅ Analyzed | Swaps ghost double buffers: `PTR_DAT_0065fca8` ↔ `PTR_DAT_0065fcb0`. Updates `DAT_006fd150` (active ghost path). Gates on `DAT_00f42560` and lap count. |
| 0x0044d0c0 | RecordGhostPosition() | [in rvgl.exe.c line 33974] | ✅ Analyzed | Per-frame ghost frame recorder. Writes timestamp/speed/quaternion. SLERP test: skips frame if orientation < 0.6 and distance < 1000 from prev frame. |
| 0x004102d0 | CreateGhostPath() | [in rvgl.exe.c line 9203] | ✅ Analyzed | Builds ghost replay path from 0x1c20 car samples. Stores 3 floats (XYZ) per sample at `DAT_006e4320`. Sets `DAT_006e430c`=sample_count, `DAT_006e4310=1`. |
| 0x004442b0 | RenderScrollingText() | [in rvgl.exe.c line 28869] | ✅ Analyzed | Credits/title screen text scroll renderer. Uses `DAT_006fb6ec` (section), `ScreenResPointer`, calls `FUN_00492150` (DrawTextLine). Sections from `PTR_s_RE_VOLT_0065cf00` table. |

### 🏁 Race Session Setup (Priority: CRITICAL — NEW Batch 4)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00450700 | RaceSetup() | [in rvgl.exe.c line 35952] | ✅ Analyzed | **CRITICAL: Main race session initializer**. Giant switch on `DAT_006e34c0` (GameMode). Case1=single, Case2=multi, Case3=practice, Case4/6=lobby. Copies race config, calls `FUN_0044f4f0`/`FUN_0044fb30`/`FUN_0044f7b0`. Sets up all global race state. |
| 0x0044f7b0 | SetupAllRaceCars() | [in rvgl.exe.c line 35314] | ✅ Analyzed | Creates ALL race cars from `DAT_00f3eb50` entry table. Loop: `FUN_004a71d0` (spawn orientation) → `FUN_004f03a0` (create car). Sets `DAT_0acee7e8` = local player's car when `is_local != 0`. Falls through to `FUN_0044f640` if player_slot = -1. |
| 0x0044f640 | CreateSinglePlayerCar() | [in rvgl.exe.c line 35225] | ✅ Analyzed | Creates single player car: type=9, subtype=7 (player behavior). Model from `DAT_006e34cc`. Network ID `DAT_0ae6f6ac` → `car+0x6a84`. Copies name to `car+0x6a70`. |
| 0x0044f4f0 | AddRaceEntry() | [in rvgl.exe.c line 35196] | ✅ Partial | Fills `RaceCarEntry` in `DAT_00f3eb50` table. Increments `_DAT_00f3eb04`. Args: (type, slot, model, variant, is_player, slot_num, name_ptr). |
| 0x0044f1f0 | SetupPlayerRaceEntry() | [in rvgl.exe.c ~35066] | ✅ Partial | Initializes player's race entry: model, start position, name. Called for case1 (single player). |
| 0x0044fb30 | RandomizeCarPicks() | [in rvgl.exe.c line 35447] | ✅ Analyzed | Builds per-class car pick tables (6 classes from `CarInfo+0xec`). Fisher-Yates shuffle via `FUN_00533360` (RandInt). Assigns cars to AI slots avoiding duplicates with player's 3 previous unique cars. |
| 0x004504e0 | SelectRandomCar() | [in rvgl.exe.c line 35845] | ✅ Analyzed | Picks random valid car from `CarModelDatabase` (skipping `+0xe5=0` unavailable/missing cars). Returns car model index. |
| 0x004505c0 | GetCarColor() | [in rvgl.exe.c line 35887] | ✅ Analyzed | Random colour variant for `param_1` car model. Uses `(short*)(model*0x110+DB+0xe0)` as upper range for `FUN_00533360`. Gates on `DAT_00f434a2`=1. |
| 0x00450600 | GetRandomTrack() | [in rvgl.exe.c line 35903] | ✅ Analyzed | Picks random available track. Iterates `DAT_006e34d0` tracks via `FUN_004548d0` (available check) + `FUN_004549b0` (select). Returns track index. |
| 0x004a71d0 | ComputeSpawnOrientation() | [called in FUN_0044f7b0] | ✅ Partial | Converts spawn type to rotation matrix. Args: (spawn_type, local[12], out_matrix[48]). |
| 0x0044f980 | AssignStartPositions() | [in rvgl.exe.c line 35314] | ✅ Analyzed | Assigns grid start positions to race entries using Fisher-Yates partial shuffle. Writes start pos indices to `DAT_00f3eb54` (offset +0x04 in each RaceCarEntry, stride 0x50). |

### 🛤️ Waypoint / Route Navigation (Priority: HIGH — NEW Batch 4)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x0040fc20 | FreeWaypointTables() | [in rvgl.exe.c line 8949] | ✅ Analyzed | `free(DAT_006e3b08); free(DAT_006e3b00)`. Frees both waypoint and zone tables. |
| 0x0040fd90 | FindCarInZoneSectionOBB() | [in rvgl.exe.c line 8959] | ✅ Analyzed | 3D OBB inside-test against all waypoints in zone section `param_2`. Writes `car+0x67dc`=section idx, `car+0x67d8`=waypoint data. Returns 0/1. |
| 0x0040ff20 | FindCarCurrentZone() | [in rvgl.exe.c line 9040] | ✅ Analyzed | Calls `FindCarInZoneSectionOBB` for current zone `car+0x67d4`, then tries ±1 zone (with modular wrap). Updates `car+0x67d4` on match. |
| 0x0040ffb0 | FindCarCurrentZoneBrute() | [in rvgl.exe.c line 9080] | ✅ Analyzed | Brute-force all `DAT_006e3b10` zones for OBB match. Sets `car+0x67d8=-1` if no table. Used on spawn. |
| 0x0040e150 | UpdateCarRoutePosition() | [in rvgl.exe.c line 8049] | ✅ Analyzed | **CORE: 14 XREFs**. Per-frame route tracker. OBB-tests `DAT_006e3b00` zones, if car inside zone: updates `car+0x6910` (route_section), `car+0x6914` (track_t, wraps by `DAT_006e3ae0`), `car+0x6918` (norm), `car+0x690c` (facing_dir). Updates `car+0x690c = dot(car_fwd, route_dir) > 0.6`. |
| 0x00410070 | DeleteWaypointNode() | [in rvgl.exe.c line 9162] | ✅ Analyzed | Removes node from `DAT_006f94a0` array (stride 0x108). Shifts remaining nodes. Clears references in `DAT_006e42f0/f8/4300`. |
| 0x00410260 | ShowErrorDialog() | [in rvgl.exe.c line 9200] | ✅ Analyzed | `sprintf → printf → MessageBox("Error", msg, 0)`. Uses FUN_00533540+00529490+00404720. |
| 0x00410500 | WaypointNodeLinkPatch() | [in rvgl.exe.c line 9336] | ✅ Analyzed | Fixes link fields at node+0x18,+0x20,+0x28,+0x30 during node deletion/insertion. |
| 0x00410680 | GetWaypointNodeClosestToRay() | [in rvgl.exe.c line 9416] | ✅ Analyzed | Editor ray-to-node distance test. Tests 3 points per node (L/R/center). Returns node ptr. Sets `DAT_006e3c6c` = sub-node indicator (0/1/3). |
| 0x00410860 | WaypointDijkstraRelax() | [in rvgl.exe.c line 9504] | ✅ Analyzed | Recursive Dijkstra shortest-path relaxation. Updates `node+0x0c` = distance cost. Uses `DAT_006e3c5c` (min_dist), `DAT_006e3c68` (start_node). |
| 0x004103e0 | FindNearestWaypointNode() | [in rvgl.exe.c line 9284] | ✅ Analyzed | Finds nearest waypoint node to float[3] position. Iterates all `DAT_006f94aa` nodes, stride 0x108, centroid `(+0x38..+0x50)`. Returns node index. |
| 0x00410020 | WaypointLinkPromote() | [in rvgl.exe.c line 9120] | ✅ Analyzed | Promotes link from `node+0x20` → `node+0x18` if `+0x18` is null. Same for `+0x30` → `+0x28`. |

### �🎹 Input Processing (Priority: HIGH)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x0052b250 | TouchInputHandler() | [NOT EXPORTED] | ⏳ Not Analyzed | Multi-touch handler (4 fingers) |
| 0x00527da0 | WindowFocusGain() | [NOT EXPORTED] | ⏳ Not Analyzed | Mouse button 0x0c handler |
| 0x00527cf0 | WindowFocusLoss() | [NOT EXPORTED] | ⏳ Not Analyzed | Mouse button 0x0d handler |
| 0x005397c0 | DebugToggle() | [NOT EXPORTED] | ⏳ Not Analyzed | 'D' key handler |
| 0x00539750 | FrequentUpdate() | [NOT EXPORTED] | ⏳ Not Analyzed | Called often, unknown purpose |
| 0x00539b60 | DebugRelated() | [NOT EXPORTED] | ⏳ Not Analyzed | Called with debug toggle |

### 📄 File I/O Helpers (Priority: LOW)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x00533f60 | ReadNextToken() | [NOT EXPORTED] | ✅ Analyzed | Lexer/token reader — reads next whitespace-delimited token from config text stream. *(was ReadLineFromFile)* |
| 0x00533ef0 | CompareKeyword() | [NOT EXPORTED] | ⏳ Not Analyzed | Case-insensitive string compare |
| 0x00534370 | ReadStringParameter() | [NOT EXPORTED] | ⏳ Not Analyzed | Parse string value from file |
| 0x005343c0 | ReadByteParameter() | [NOT EXPORTED] | ⏳ Not Analyzed | Parse 0/1 flag from file |
| 0x005345e0 | ReadFloatParameter() | [NOT EXPORTED] | ⏳ Not Analyzed | Parse float from file |
| 0x005346e0 | ReadIntParameter() | [NOT EXPORTED] | ⏳ Not Analyzed | Parse integer from file |
| 0x005334f0 | SafeStringCopy() | [NOT EXPORTED] | ⏳ Not Analyzed | Copy string with bounds checking |
| 0x005335a0 | FormatPath() | [NOT EXPORTED] | ⏳ Not Analyzed | sprintf for file paths |

---

## libGLESv2.dll Functions

### 🔧 Context Management (Priority: CRITICAL)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x67c77ae0 | **GetCurrentContext()** | func_67c77ae0.txt | ✅ Analyzed | **Main context getter (called by ALL 601 GL functions)** |
| 0x67f20810 | TlsGetValueWrapper() | func_67f20810.txt | ✅ Analyzed | Wraps Windows TlsGetValue |
| 0x67c779b0 | CreateContext() | func_67c779b0.txt | ✅ Analyzed | Allocates 32-byte handle, calls init |
| 0x67cec510 | ValidateContext() | func_67cec510.txt | ✅ Analyzed | Checks context_lost_flag, handles errors |
| 0x67cec310 | InitializeContextHandle() | func_ContextHandleInitializer.txt | ✅ Analyzed | Initializes 32-byte handle structure |
| 0x67c8a750 | IsContextLost() | func_ContextLostChecker.txt | ✅ Analyzed | Returns byte at context+0x39f1 |

### 🎨 Core GL Functions (Priority: HIGH)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| 0x67c95040 | glDrawArrays_Impl() | func_glDrawArrays.txt | ✅ Analyzed | Draw call implementation |
| 0x67d5da10 | glDrawArrays_Validate() | func_glDrawArrays.txt | ✅ Analyzed | Validation before draw |
| [Similar] | glDrawElements() | func_glDrawElements.txt | ✅ Analyzed | Indexed draw call |
| 0x67c8bfd0 | glClear_Impl() | func_glClear.txt | ✅ Analyzed | Clear framebuffer |
| 0x67d5d600 | glClear_Validate() | func_glClear.txt | ✅ Analyzed | Clear validation |
| 0x67c8de90 | glUseProgram_Impl() | func_glUseProgram.txt | ✅ Analyzed | Bind shader program |
| 0x67d5d970 | glUseProgram_Validate() | func_glUseProgram.txt | ✅ Analyzed | Program validation |

### 🖼️ Texture Functions (Priority: MEDIUM)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| [Address] | glBindTexture() | func_glBindTexture.txt | ✅ Analyzed | Bind texture to target |
| [Address] | glTexImage2D() | func_glTexImage2D.txt | ✅ Analyzed | Upload texture data |
| [Address] | glTexParameteri() | func_glTexParameteri.txt | ✅ Analyzed | Set texture parameters |
| [Address] | glGenTextures() | func_glGenTextures.txt | ✅ Analyzed | Generate texture IDs |

### 📦 Buffer Functions (Priority: MEDIUM)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| [Address] | glBindBuffer() | func_glBindBuffer.txt | ✅ Analyzed | Bind buffer to target |
| [Address] | glBufferData() | func_glBufferData.txt | ✅ Analyzed | Upload buffer data |
| [Address] | glGenBuffers() | func_glGenBuffers.txt | ✅ Analyzed | Generate buffer IDs |

### 🎨 Shader Functions (Priority: MEDIUM)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| [Address] | glCreateShader() | func_glCreateShader.txt | ✅ Analyzed | Create shader object |
| [Address] | glCompileShader() | func_glCompileShader.txt | ✅ Analyzed | Compile shader source |
| [Address] | glCreateProgram() | func_glCreateProgram.txt | ✅ Analyzed | Create shader program |
| [Address] | glGetUniformLocation() | func_glGetUniformLocation.txt | ✅ Analyzed | Get uniform location |

### 🔄 Other GL Functions (Priority: LOW)
| Address | Function Name | File Location | Status | Description |
|---------|--------------|---------------|--------|-------------|
| [Address] | glFlush() | func_glFlush.txt | ✅ Analyzed | Flush GL commands |

---

## Memory Addresses

### 🎮 Game State & Control
| Address | Type | Name | Description | File Reference |
|---------|------|------|-------------|----------------|
| 0x006e30a0 | function* | GameStateFunction | Current game mode function pointer | ultra_priortity1/dat_006e30a0.txt |
| 0x006e30a8 | byte | GameQuitFlag | 1=quit, 0=running | - |
| 0x006e30a9 | byte | AKeyFlag | Set when 'A' pressed | - |

### 🖥️ Window & Display
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x0065c020 | int | DebugModeFlag | Toggled by 'D' key |
| 0x0065c021 | char | WindowMinimizedFlag | 0=minimized, 1=active |
| 0x0ae85df0 | void* | SDLWindowPointer | SDL window handle |
| 0x006e3414 | int | WindowWidth | From -window argument |
| 0x006e3410 | int | WindowHeight | From -window argument |
| 0x006e340c | int | MultisampleLevel | MSAA samples |

### ⚙️ Game Settings (Command-Line)
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

### 🚗 Car System
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x006fab50 | CarInfo[49] | CarArray | Array of 49 car structures (272 bytes each) |
| 0x00f3f4c0 | int | StandardTrackCount | Number of standard tracks |
| 0x00f3f4c4 | int | OtherTrackCount | Other track type count |

### 🧵 Threading
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

### ⏱️ Timing
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x0ae48de0 | uint64 | PerformanceFrequency | QueryPerformanceFrequency |

### 👆 Touch Input (4 simultaneous touches!)
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

### 🌐 Network
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x0ae6f6a0 | int | LobbyModeFlag | Lobby mode enabled |
| 0x0ae6f6a1 | byte | LobbyModeFlag2 | Secondary lobby flag |
| 0x0ae6f5a0 | char[256] | LobbyServerAddress | Server address string |
| 0x0065c030 | int | P2PEnabled | -nop2p flag (inverted) |
| 0x006625a2 | byte | MulticastEnabled | -nomulticast (inverted) |
| 0x006625a1 | byte | LateJoinEnabled | -nolatejoin (inverted) |
| 0x0065c02c | int | NetworkPort | -port |

### 👤 Profile
| Address | Type | Name | Description |
|---------|------|------|-------------|
| 0x006635e0 | int | ProfileLoadedFlag | Profile loaded |
| 0x006e3640 | char[16] | ProfileName | From -profile argument |

### 🎨 libGLESv2.dll Memory
| Address | Type | Name | Description | File Reference |
|---------|------|------|-------------|----------------|
| 0x6814c020 | DWORD | TLS_INDEX | Windows TLS slot index (-1 = uninitialized) | dat_6814c020.txt |

---

## SDL2.dll Imports

### Functions Used by RVGL
| Import Name | Used For |
|-------------|----------|
| SDL_Init | Initialize SDL subsystems |
| SDL_CreateWindow | Create game window |
| SDL_GL_CreateContext | Create OpenGL context |
| SDL_GL_SetSwapInterval | VSync control |
| SDL_GL_SwapWindow | Present frame (swap buffers) |
| SDL_GL_GetProcAddress | Load GL function pointers |
| SDL_PollEvent | Get next event from queue |
| SDL_Delay | Sleep for milliseconds |
| SDL_GetTicks | Get time in milliseconds |
| SDL_ThreadID | Get current thread ID |
| SDL_CreateMutex | Create mutex |
| SDL_LockMutex | Lock mutex |
| SDL_UnlockMutex | Unlock mutex |
| SDL_CreateSemaphore | Create semaphore |
| SDL_SemWait | Wait on semaphore |
| SDL_SemPost | Signal semaphore |
| SDL_CreateThread | Create background thread |

---

## File Organization

### RVGL.exe Structure
```
rvgl.exe/
├── func_entry.txt                    (Entry point)
├── maingameloop.txt                  (Main game loop)
├── func_00401500.txt                 (Event processor)
│
├── gameplay/                         (Game state functions)
│   ├── lab_00499b30.txt             (Splash screen)
│   ├── lab_00543890.txt             (Multiplayer)
│   └── lab_00406b40.txt             (Unknown state)
│
├── game_initialization/              (Loading & init)
│   ├── func_0043f140.txt            (Load all cars)
│   ├── func_0043b6c0.txt            (Parse parameters.txt)
│   ├── func_00452190.txt            (Load all tracks)
│   ├── func_00451da0.txt            (Parse track .inf)
│   ├── func_005bd590.txt            (Init car structure)
│   ├── func_00536bb0.txt            (Init subsystem 1)
│   ├── func_00529310.txt            (Init subsystem 2)
│   ├── func_004ee170.txt            (Init subsystem 3)
│   └── func_0044bae0.txt            (Init subsystem 4)
│
├── race_initialization/              (Race setup)
│   ├── func_004995f0.txt            (Race setup)
│   ├── func_004889c0.txt            (Race init 1)
│   └── func_0048a560.txt            (Race init 2)
│
├── ultra_priortity1/                 (Critical data)
│   └── dat_006e30a0.txt             (Game state function pointer)
│
├── sdl2.dll (pointers to external functions)/
│   └── imports.txt                   (SDL2 imports)
│
└── imports.txt                       (All imports summary)
```

### libGLESv2.dll Structure
```
libGLESv2.dll/
├── dat_6814c020.txt                  (TLS index global)
│
├── Context Management/
│   ├── func_67c77ae0.txt            (GetCurrentContext - HOT PATH!)
│   ├── func_67c779b0.txt            (CreateContext)
│   ├── func_67cec510.txt            (ValidateContext)
│   ├── func_67cec310.txt            (InitializeContextHandle)
│   ├── func_67f20810.txt            (TlsGetValueWrapper)
│   └── func_ContextLostChecker.txt  (IsContextLost)
│
├── Draw Functions/
│   ├── func_glDrawArrays.txt        (Draw call)
│   └── func_glDrawElements.txt      (Indexed draw)
│
├── State Functions/
│   ├── func_glClear.txt             (Clear buffers)
│   └── func_glUseProgram.txt        (Bind shader)
│
├── Texture Functions/
│   ├── func_glBindTexture.txt
│   ├── func_glTexImage2D.txt
│   ├── func_glTexParameteri.txt
│   └── func_glGenTextures.txt
│
├── Buffer Functions/
│   ├── func_glBindBuffer.txt
│   ├── func_glBufferData.txt
│   └── func_glGenBuffers.txt
│
├── Shader Functions/
│   ├── func_glCreateShader.txt
│   ├── func_glCompileShader.txt
│   ├── func_glCreateProgram.txt
│   └── func_glGetUniformLocation.txt
│
└── Other/
    └── func_glFlush.txt
```

---

## Analysis Status Summary

### Completion Overview
```
RVGL.exe:        80% complete  (up from 72% in Batch 10)
  ✅ Main loop and entry
  ✅ Event processing (SDL all event types)
  ✅ Car loading system (static pool, 49 cars)
  ✅ Track loading system
  ✅ File format parsers (parameters.txt, track .inf)
  ✅ Delta time / frame timer (CalcDeltaTime)
  ✅ GL state setup / rendering helpers
  ✅ Input: keyboard, mouse, controller (all 3)
  ✅ Screen fade / race-end state machine
  ✅ Game mode management (11+ modes)
  ✅ Item / pickup system
  ✅ FPS profiler
  ✅ AI Brain (FUN_00408a60 — 1517 lines COMPLETE)
  ✅ AI state machine (13 states, all transitions)
  ✅ AI avoidance / stuck detection / rubber band
  ✅ Race position sorting (UpdateRacePositions)
  ✅ AI node graph structure (0x108 bytes/node)
  ✅ AI node editor (8 sub-modes identified)
  ✅ Physics world step (PerFramePhysicsWorldStep)
  ✅ Ghost car transparency system
  ✅ Keyframe animation system (16 bone channels)
  ✅ Gaussian / conjugate gradient physics solvers
  ✅ Mesh vertex OBB culling
  ✅ Editor mode dispatch + sub-modes
  ✅ Custom Printf engine (InternalFormatEngine + all primitives)
  ✅ BigNum arbitrary-precision float-to-string engine (15 functions)
  ✅ Virtual File System (VFS) — path resolution + mount system
  ✅ Race result CSV logging (5 functions)
  ✅ Track unlock / progression system (6 functions)
  ✅ strlen / strncasecmp / ReadNextToken (custom CRT strings)
  ⏳ Racing game LOOP BODY (func_00499690 — entry only)
  ❌ Runtime car array builder (FUN_0044****c — see priority queue)
  ❌ Full audio system (AL function tables partially mapped)
  ❌ Full multiplayer packet handler (1013 lines, only partial)

libGLESv2.dll:   95% complete
  ✅ TLS system
  ✅ Context management
  ✅ All GL function patterns
  ✅ Validation systems
  ✅ Error handling
```

### Priority Queue
*See updated Priority Queue at the bottom of this file.*

---

## 🆕 RuntimeCarArray1 — Analyzed Functions
> Static car pool lives at `*(DAT_006fab50)`. Stride = 0x110 per `CarInfo`. Count at `DAT_006fab58` (starts 0×31 = 49). Runtime car struct = 0xAAC bytes, template at `DAT_006fab80`.
> **All files now in `rvgl.exe/01_CarSystem/`**

| Address | Function Name | File | Status | Description |
|---------|--------------|------|--------|-------------|
| 0x0043c580 | InitRuntimeCarStruct() | 01_CarSystem/func_0043c580.txt | ✅ Analyzed | **Copies 0xAAC-byte template from `DAT_006fab80` into `param_1` (runtime car slot), then applies per-car-index overrides**. Special cases for indices 0x1b, 0x16, 0x21, 0x22. Physics params from table at `0x670fa4` (stride 0x28). Called from `FUN_004ab810` and `FUN_005589b0` |
| 0x004ab810 | InitCarSlotFull() | 01_CarSystem/func_004ab810.txt | ✅ Analyzed | **Full car slot initializer**. Takes `(int *slot, carIdx, carType)`. Stores car IDs at `slot+0x48`. Calls `FUN_0043c580` to init local template. Checks `DAT_0acee7e8==slot` to detect car-0 (player). Sets `slot[1]` (byte+4) = car type (`9`=spectator/inactive, `4`=special, `1`=player). Writes physics params to `slot[0x416..0x41c]`, model ptr at `slot+0x24`, track data ptr at `slot+0x32`. 572 lines. Called from `0x00445***` (race participant builder) and `FUN_005589b0` |
| 0x005589b0 | InitReplayCarSlot() | 01_CarSystem/func_005589b0.txt | ✅ Analyzed | Wrapper: if `DAT_0aea0c48` (replay scene handle) is valid and car index matches `(&DAT_0ae8e988)[DAT_0aea0c84]`, calls `InitCarSlotFull`. Otherwise just calls `FUN_0043c580` on a local. Uses `DAT_0aea0c84` as index, `DAT_0ae8e988/998` as car arrays |
| 0x0043c280 | UnknownCarFn() | 01_CarSystem/func_0043c280.txt | ⚠️ Empty export | File body empty — re-export needed |
| 0x0043c160 | FindCarIndexByName() | 01_CarSystem/func_0043c160.txt | ✅ Analyzed | Searches `DAT_006fab50 + i*0x110 + 0x14` (display name field). Returns car INDEX or 0x22 if not found. Called from `FUN_00479e50` |
| 0x0043bdf0 | UpdateCarSelectability() | 01_CarSystem/func_0043bdf0.txt | ✅ Analyzed | Iterates static pool (stride 0x110), sets `+0xE5` (selectableByPlayer flag). `DAT_0065c0f8` mode: 0=all selectable, 1=class filter, 2=first-8 only. Unlock check at `+0xF0` via `FUN_0044a250/…` |
| 0x004070e0 | CheckFileIntegrity() | 01_CarSystem/func_004070e0.txt | ✅ Analyzed | Binary file integrity checker. `.prm` files: reads last 8 bytes, checks magic `0x7830`. Other files: reads last 4 bytes. Returns bool |
| 0x0049c080 | PackPhysicsVectors() | 01_CarSystem/func_0049c080.txt | ✅ Analyzed | Packs 10 float params into `float[4][3]` array. Simple helper called 49× from LoadAllCars |
| 0x0043fac0 | LoadUserCars() | 01_CarSystem/func_0043fac0.txt | ✅ Analyzed | Scans `cars/` dir via `_wfindfirst64`, `realloc`s `DAT_006fab50`. Sorts by class rating. `DAT_006e3431` = skip-user-cars flag |
| **`DAT_016f8e60`** | **WeaponArraySlot0Ptr** | **01_CarSystem/dat_016f8e60.txt** | ✅ Analyzed | **⭐ 259 XREFs. CLEARED to 0 by `FUN_004a5500` on reset. During active play: holds pointer to slot 0 of the weapons/projectile array (base at `0x016f8ee0`). `FUN_004a1290` compares `param_1 == DAT_016f8e60` to detect the player's entity. NOT a heap-allocated car array base — static inline array. Stride 0x140 (320 bytes), 8 slots, end at `0x016f98e0`. Also read by `FUN_004a1ae0/1d40/20f0/2340/4910` and `FUN_004a5ce0`** |
| 0x004a5500 | ResetProjectileArray() | to be look at/func_004a5500.txt | ✅ Analyzed | **RESET function** (NOT allocator). Does-while loop from `0x016f8ee0` to `0x016f98e0`, stride `+0x50` (int4) = **320 bytes per slot**, 8 iterations. Per slot: writes `0xFFFFFFFFFFFFFFFF` at `puVar1[-8]`, zeros all fields. After loop: clears 10 management globals (`0x016f8e40`–`0x016f8e80`) and **sets `DAT_016f8e60 = 0`**. Called from 3 places: `FUN_00455420(0x0045548e)`, `FUN_0053bbb0(0x0053d857)`, `006b6400(*)` |
| 0x004a1290 | InitWeaponSlot() | to be look at/func_004a1290.txt | ✅ Analyzed | **Weapon/projectile slot initializer**. `void FUN_004a1290(int *param_1, longlong param_2, int param_3)`. Sets `param_1[0]=0` (state), `param_1[1]=param_3` (weapon type). Type 6→ `LAB_004a3610` fn ptr; type 7→ `LAB_004a3100 + LAB_0049fd60`; else→ `LAB_004a3100 + FUN_0049faf0`. Reads `piVar8 = DAT_016f8e60`. At end: **if `piVar8 == param_1`** (this is the player's slot 0), calls `FUN_004a1170(param_1)`. 34 XREFs — called from projectile/weapon-spawn sites |
| 0x004a4910 | CameraEntityViewportUpdate() | to be look at/func_004a4910.txt | ✅ Analyzed | **Per-frame viewport rect writer**. No params. Reads 9 camera entity pointers from `DAT_016f8e40/48/50/58/60/68/70/78/80`. For each non-null ptr: checks `ptr+0x12c` (state; skip if 0 or 3), `ptr+0x128` (viewport slot index 0–3 or -1=player). Copies viewport dims from `DAT_016f9a60` or `DAT_016f9aa0 + idx*0x28` into `ptr+0x100..0x124` (viewport rect). Sets global `ScreenResPointer = ptr+0x100`. Handles split-screen minimap sub-viewports at `016f8e68/70/78`. 5 XREFs |
| 0x004a5ce0 | CameraEntityInit() | to be look at/func_004a5ce0.txt | ✅ Analyzed | **Camera/pickup entity initializer**. `void FUN_004a5ce0(int *param_1, longlong param_2, int param_3)`. `bVar3 = (DAT_016f8e60 == param_1)` = is this the player primary camera slot? Sets `param_1[0]=2` (type), `param_1[1]=param_3` (subtype), `param_1+0x32=param_2` (attached car ptr). If player slot: calls `FUN_005334a0(0x40a00000)`, sets `DAT_00660800` (FOV/zoom). Calls `FUN_004a5b50(-1, param_2+0x24)` (attach lookup); if returns 0 → falls back to `InitWeaponSlot(param_1,param_2,0)`. Else: assigns fn-ptr label `LAB_004a6700` (type 0) or `LAB_004a6600` (type 1). 21 XREFs — spawned per race participant camera |
| 0x00450700 | RaceSessionSetup() | to be look at/func_00450700.txt | ✅ Analyzed | **Race session participant builder** (NOT the per-frame race loop). `void FUN_00450700(char param_1)`. Switch on `DAT_006e34c0` (game mode 1–0x10). Builds participant table at `DAT_00f3eb70 + i*0x50` (stride 0x50=80 bytes, max 0x1e=30). **`_DAT_00f3eb04`** = live participant count. **`DAT_00f3eb00`** = player race index. `DAT_00f3eb08` high-dword = mode tag. Entries built via `FUN_0044f1f0` (player slot) and `FUN_0044f4f0` (AI fill). Car pool: `DAT_006fab50 + idx*0x110`. `DAT_0ae8e934`=event count, `DAT_0ae8e930`=max target. Modes: 1=normal, 2=lobby/custom, 3=time-trial, 4/6=MP sync-wait, 5=replay, 7=exit, 8/9=practice, 0xb=timed, 0xf=tournament, 0x10=free-race. 765 lines, 3 callers |
| 0x0044f1f0 | AddParticipantAndCount() | to be look at/func_0044f1f0.txt | ✅ Analyzed | **Writes one entry to race participant table then increments count.** 7 args: `(type, flags2, carPoolIdx, skinIdx, slotParam5, slotParam6, namePtr)`. Appends to `DAT_00f3eb70 + i*0x50` (copies 0x14 dwords of car info from `DAT_006fab50+carIdx*0x110`), skin string `+0x14`, player name at `+0x20`. Parallel header arrays written at `DAT_00f3eb14+i*0x14` and `DAT_00f3eb30`. Increments `_DAT_00f3eb04`. If `type==1` also increments player-count in `DAT_00f3eb08._0_4_`. Returns 1. **Does NOT allocate 0xAAC runtime car struct.** 8 XREFs, all from RaceSessionSetup |
| 0x0044f4f0 | AddParticipantNoCount() | to be look at/func_0044f4f0.txt | ✅ Analyzed | **Identical to 0x0044f1f0 except: does NOT increment `_DAT_00f3eb04`.** Used for AI fill slots where count is managed externally. 10 XREFs — called from RaceSessionSetup cases 2,3,f,0x10 and from `FUN_00451ab0`, `FUN_00451c70` |
| 0x004a5b50 | FindNearestPickupNode() | to be look at/func_004a5b50.txt | ✅ Analyzed | **Nearest node lookup.** `int* FUN_004a5b50(int type, float* pos)`. Iterates node array at `DAT_016f8ea0` (stride 0x38), count in `DAT_016f8e98`. If `type==-1`: finds closest node ignoring type. Else: finds closest where `node[0]==type`. Filters by `DAT_016e4428` bitmask (skip nodes with matching bits). Node layout: `[0]`=type, `[3..5]`=XYZ floats, `[0xc]`=bitmask. Returns pointer to nearest, or NULL. Called by CameraEntityInit and 3 other camera-spawn sites |

---

## 🆕 RaceSysCore1 — Analyzed Functions
> **All files now in `rvgl.exe/04_RaceCore/` (race logic) and `rvgl.exe/06_Rendering/` (viewport) and `rvgl.exe/07_Input/` (input)**

| Address | Function Name | File | Status | Description |
|---------|--------------|------|--------|-------------|
| 0x0048a5c0 | GetRaceState() | RaceSysCore1/func_0048a5c0.txt | ✅ Analyzed | Returns `DAT_016942dc` (race state 0–6). Single-instruction stub |
| 0x0048a5d0 | UpdateFadeTransition() | RaceSysCore1/func_0048a5d0.txt | ✅ Analyzed | Per-frame fade animator. Writes `DAT_016942d8` (fade float). State 1=fade-in, 3=fade-out, 5=fully opaque. Also dispatches OpenAL events and calls ViewportAndCameraSetup. Multiplayer cmd buffer at `DAT_01694228/22c/30` |
| 0x0048ae80 | RaceBackgroundRenderer() | RaceSysCore1/func_0048ae80.txt | ✅ Analyzed | Race background color (sin wave from `DAT_0ace49a0`) + audio state manager. Issues GL calls via `DAT_0aea77a8/26c0/6540/3178`. Calls ViewportAndCameraSetup |
| 0x004674d0 | ItemSystemUpdater() | RaceSysCore1/func_004674d0.txt | ✅ Analyzed | Item/pickup per-frame update (NOT car physics). Active item at `DAT_0ace47c0`. Item state array: `DAT_016f8ec0 + DAT_016f9a44 * 0x28`. Item data: `DAT_01800298 + index*0x138`. Timer at `DAT_00f3fbc8` |
| 0x00445d20 | CheckPlayerReady() | RaceSysCore1/func_00445d20.txt | ✅ Analyzed | Scans keyboard (F1–F10, special keys) via `DAT_0ae6ef00/ed00`. Checks `SDL_GetModState` for Shift. Touch active at `DAT_0ae6dce9`. Returns 1 when player presses ready key |
| 0x00482e40 | NetworkRenderStub() | RaceSysCore1/func_00482e40.txt | ✅ Analyzed | If `DAT_0ae7fc4c != 0` (network mode), calls `FUN_004827e0()`. Otherwise no-op |
| 0x00495a90 | FreeTextureSlot() | RaceSysCore1/func_00495a90.txt | ✅ Analyzed | Frees sprite/texture slot `param_1`. Clears `DAT_016cf6a0 + idx*0x25`. Calls `SDL_FreeSurface`, then `(*DAT_0aea2c68)(1, &ptr)` to delete GL texture. Thread-safe via `SDL_ThreadID` |
| 0x00451ab0 | SetupRaceTrackRandom() | RaceSysCore1/func_00451ab0.txt | ✅ Analyzed | Sets `DAT_0ae8e920=0xd`, `DAT_006e34c0=0xd`. Picks random track via `FUN_00533360(0xe)` → `DAT_006e34c8`. Sets up race event list via `FUN_0044f4f0`. `DAT_006fb6a0` = forced-track mode |
| 0x005250c0 | CalcDeltaTime() | RaceSysCore1/func_005250c0.txt | ✅ Analyzed | `SDL_GetPerformanceCounter` → `DAT_0065c038` (raw ratio) + `DAT_0065c03c` (seconds per frame). Slow-motion via `DAT_0ae48da0`. Freq in `DAT_0ae48de0` |
| 0x00539bc0 | ScreenshotAndFrameEnd() | RaceSysCore1/func_00539bc0.txt | ✅ Analyzed | If `DAT_006e30a9` (A key): `glReadPixels` → `SDL_CreateRGBSurface` → `IMG_SavePNG_RW`. Also calls `(*DAT_0aea20a8)` and `(*DAT_0aea22e0)` for window/buffer ops |
| 0x00529f60 | MouseInputUpdater() | RaceSysCore1/func_00529f60.txt | ✅ Analyzed | `SDL_GetRelativeMouseState` → `DAT_0ae6dcb4/b8` (delta XY). Accumulated in `DAT_0ae6dcc8/cc`. `SDL_GetMouseState` → `DAT_0ae6dcc0`. Controller struct `DAT_00f43400` (+0x28c/290 = axis bindings, +0x28f/293 = axis type flags). Final output: `DAT_0ae6dca0` (packed steering XY) |
| 0x00529e50 | KeyboardStateSnapshot() | RaceSysCore1/func_00529e50.txt | ✅ Analyzed | Copies previous keyboard state `DAT_0ae6ef00` → `DAT_0ae6ed00`, then calls `SDL_GetKeyboardState()` and memcpy-s 0x200 bytes into `DAT_0ae6ef00`. `DAT_006fcec8` = "no keyboard" mode clears both buffers |
| 0x0052a910 | ControllerStateUpdater() | RaceSysCore1/func_0052a910.txt | ✅ Analyzed | Iterates up to 2 controllers (`DAT_00f43400` stride 0x44, `DAT_0ae6e520` stride 500). Copies previous state → `DAT_0ae6dd40`. `SDL_GameControllerGetAxis` / `SDL_JoystickGetAxis` per-axis → `DAT_0ae6e520`. `SDL_GameControllerGetButton` for buttons → offset +400. Dead-zone and sensitivity at `DAT_00f436c4/c8` |
| 0x00498ab2 | ViewportSetup_Trampoline1() | RaceSysCore1/func_00498ab2.txt | ✅ Analyzed | Single call: `ViewportAndCameraSetup(&DAT_016f9a60, DAT_006e3470)` |
| 0x00498d8d | ViewportSetup_Trampoline2() | RaceSysCore1/func_00498d8d.txt | ✅ Analyzed | Same as above |
| 0x004997df | ViewportSetup_Trampoline3() | RaceSysCore1/func_004997df.txt | ✅ Analyzed | Same as above |
| 0x00450700 | RacingLoopBody() | RaceSysCore1/func_00450700.txt | ⚠️ Partial | 764-line function — likely main racing game loop body. Only header read. **Export body urgently** ⭐⭐⭐ |

---

## 🆕 RGameModeManagement — Analyzed Functions
> **All files now in `rvgl.exe/00_CoreLoop/` (frame dispatch), `rvgl.exe/04_RaceCore/` (mode logic), `rvgl.exe/08_Network/` (multiplayer)**

| Address | Function Name | File | Status | Description |
|---------|--------------|------|--------|-------------|
| 0x00401610 | FrameUpdateDispatcher() | RGameModeManagement/func_00401610.txt | ✅ Analyzed | Per-frame: calls `CalcDeltaTime`, `FUN_00525350`. Game mode 4/6 branch runs render pipeline (`FUN_0052e760`, physics, AI). Otherwise updates `DAT_0ae48dc8` accumulator and iterates linked list at `DAT_0acee9e8` (+0x10 next ptr, updates `+0xf90` timer) |
| 0x0044c130 | GameGaugeReplayState() | RGameModeManagement/func_0044c130.txt | ✅ Analyzed | Loads `game_gauge_replay.rpl`, sets mode `DAT_0ae8e920=5 / DAT_006e34c0=5`. Copies replay data block. Sets game-state fn `DAT_006e30a0 = &LAB_00406b40` |
| 0x0044c820 | GameGaugeFPSRecorder() | RGameModeManagement/func_0044c820.txt | ✅ Analyzed | Per-second FPS: increments `DAT_006fceb0` (frames this second), `DAT_006fceac` (total frames). After 1000ms: records min (`DAT_006fcea8`), max (`DAT_006fcea4`), history array `DAT_006fcee0[300]`. On trigger `DAT_00f432a4`, calls `FUN_0048a560(3)`, writes `profiles/fps.txt` |
| 0x004efd50 | SetCarBehaviourState() | RGameModeManagement/func_004efd50.txt | ✅ Analyzed | Sets car behaviour slot `param_2` (0–5) on car object `param_1`. Writes function pointers into `param_1->sub+0x340/348/350/358/370/378`. Key state0 = idle, 1 = racing, 2 = spectating, 3 = battle, 4 = game-gauge, 5 = ghost. Calls `FUN_0048a560(1)` when winner detected at race state 4 |
| 0x0052f770 | MultiplayerPacketHandler() | RGameModeManagement/func_0052f770.txt | ⚠️ Partial | 1013-line monster. Processes network packets (`DAT_0ae6fc38` packet buf, type at `*buf`). Cases 1–N = different packet types. Reads player IDs from `DAT_0ae6f6a4/ac`. `DAT_006e34c4` = race mode. Only first 120 lines read |
| 0x00541f80 | RaceEndManager() | RGameModeManagement/func_00541f80.txt | ✅ Analyzed | State machine on `DAT_0ae8e860` (0–3+). State 0: init via `FUN_005415c0`, advances to 1 after 3s. State 1: `FUN_00541220`, advance to 2. State 2: runs lap/eliminate logic (`FUN_00541bb0/00541490/00540ee0`). State 3: after delay calls `FUN_0048a560(3)` (fade out). At race-state 6 → cleans up objects (`FUN_005437d0/0053f530/004f06d0`), restarts race or goes to menu (`DAT_006e34c0=0xd`). Key: `DAT_0ae8e8c8` = winner car pointer |

---

## 🆕 RenderingPipeline — Analyzed Functions
> **All files now in `rvgl.exe/06_Rendering/`**

| Address | Function Name | File | Status | Description |
|---------|--------------|------|--------|-------------|
| 0x004628b0 | DrawSprite2D() | RenderingPipeline/func_004628b0.txt | ✅ Analyzed | Draws a 2D screen-space sprite. Params: x,y,w,h (world units), UV coords, color RGBA. Uses `ScreenCenterX/Y` and scale `DAT_006e3468/46c`. Converts to NDC, fills `DAT_00f92ba0`–`DAT_00f92c40` quad vertex buffer. 62 call sites. Multiplayer: buffers into `DAT_00f40dca` array |
| 0x00491f60 | SetupGLRenderState() | RenderingPipeline/func_00491f60.txt | ✅ Analyzed | Sets up GL blend/depth state for a new render pass. Calls `(*DAT_0aea2d28)(0x8457)` (disable something), `(*DAT_0aea2d20)(0xb60/b71)` (enable), `(*DAT_0aea2f40)(0xbe2/bc0)`. Sets blend eqn `(*DAT_0aea2288)(0x302, 0x303)`. Binds texture via `(*DAT_0aea2138)(0xde1, DAT_016d6468)`. 70 call sites |
| 0x00492150 | DrawTexturedQuad() | RenderingPipeline/func_00492150.txt | ⚠️ Partial | 661-line function — heavily used (510 call sites). Full texture-mapped quad draw with clipping, string rendering, and complex UV logic. Re-export for full analysis |
| 0x004962d0 | LoadTextureByName() | RenderingPipeline/func_004962d0.txt | ✅ Analyzed | Loads a texture by filename string `param_1` into slot `param_2` (max 0x95). Calls `FUN_005370a0` to check file exists. Handles MIP extension variants (loops `.bmp0`–`.bmp9`). 47 call sites |
| 0x00539fa0 | SetDepthWriteMode() | RenderingPipeline/func_00539fa0.txt | ✅ Analyzed | One-liner: calls `(*DAT_0aea23a0)((param_1=='\0') ? 0x4100 : 0x14100)`. Enables/disables depth mask (GL_DEPTH_WRITEMASK). 15 call sites |

---

## 🆕 RFunctionPointerTableTargets — GL Function Pointer Map
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
| 0x0aea7910 | GL viewport fn variant | 1 | `ViewportAndCameraSetup` (0x004a4301) |

---

## Quick Reference

### Most Important Addresses
```
0x006e30a0 = Game state function pointer ⭐
0x00450700 = RacingLoopBody() — 764 lines, MAIN RACE LOOP ⭐⭐⭐
0x00499690 = RacingGameLoop() entry ⭐⭐⭐
0x004ab810 = RuntimeCarArrayBuilder1 — calls InitRuntimeCarStruct ⭐⭐
0x005589b0 = RuntimeCarArrayBuilder2 — calls InitRuntimeCarStruct ⭐⭐
0x00402ec0 = Main game loop entry
0x00402980 = Event processor
0x016942dc = Race state (0-6) ⭐
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

### Top 50 Export Priority Queue
> Status key: ✅=analyzed  ⏳=partial  ❌=not yet exported
```
ZONE 1 — Runtime car slot builders (who calls InitCarSlotFull?)
─────────────────────────────────────────────────────────────────
 1. ❌ 0x004452?? — FUN containing 0x0044530c   calls InitCarSlotFull ⭐⭐⭐
                    (navigate to 0x0044530c in Ghidra → find function start)
                    ALSO contains 0x00445401 + 0x00445511 (3 InitCarSlotFull calls = main race car BUILDER)
 2. ❌ 0x0044cb60 — FUN_0044cb60                 calls InitCarSlotFull at 0x0044cc31 ⭐⭐⭐
 3. ❌ 0x004f03a0 — FUN_004f03a0                 calls InitCarSlotFull at 0x004f05b8 ⭐⭐⭐
 4. ❌ 0x004e7fb0  — FUN containing 0x004e7fb2   calls InitCarSlotFull ⭐⭐⭐
 5. ❌ 0x00455420 — FUN_00455420                 calls FUN_004a5500 (cam reset) at 0x0045548e ⭐⭐⭐

ZONE 2 — Race game loop body
─────────────────────────────────────────────────────────────────
 6. ❌ 0x00499690 — RacingGameLoop()              main racing loop entry (per-frame) ⭐⭐⭐
 7. ❌ 0x00450700 — RaceSessionSetup()           ✅ DONE — no need to re-export
 8. ❌ 0x004a1ae0 — FUN_004a1ae0                 reads DAT_016f8e60, 2 XREFs inside ⭐⭐
 9. ❌ 0x004a1d40 — FUN_004a1d40                 reads DAT_016f8e60, 2 XREFs inside ⭐⭐
10. ❌ 0x004a20f0 — FUN_004a20f0                 reads DAT_016f8e60 ⭐⭐
11. ❌ 0x004a2340 — FUN_004a2340                 reads DAT_016f8e60 (2 XREFs) ⭐⭐

ZONE 3 — Camera / entity system
─────────────────────────────────────────────────────────────────
12. ❌ 0x004a5e90 — FUN_004a5e90                 called from camera system (XREFs: 004a6059) ⭐
13. ❌ 0x004a60b0 — FUN_004a60b0                 called from camera system ⭐
14. ❌ 0x004a4e50 — FUN_004a4e50                 calls CameraEntityViewportUpdate ⭐
15. ❌ 0x004a1170 — FUN_004a1170                 player-slot special init (player slot-0 detected) ⭐

ZONE 4 — Race session / mode helpers
─────────────────────────────────────────────────────────────────
16. ❌ 0x0044bab0 — FUN_0044bab0                 called at start of every RaceSessionSetup ⭐
17. ❌ 0x0044f980 — FUN_0044f980                 called at end of cases 1,2,3,0xf ⭐
18. ❌ 0x0044fb30 — FUN_0044fb30                 called in cases 1,2 ⭐
19. ❌ 0x004504e0 — FUN_004504e0                 car selector for race (extended mode path)
20. ❌ 0x00450600 — FUN_00450600                 host-restart car selector (case 4/6)
21. ❌ 0x004493d0 — FUN_004493d0                 called in cases 5,7 (race end/exit)
22. ❌ 0x00449dc0 — FUN_00449dc0                 called in case 7 (exit path)
23. ❌ 0x0044ba10 — FUN_0044ba10                 called in case 7 (exit path)
24. ❌ 0x00532ba0 — FUN_00532ba0                 sync wait loop (case 4)
25. ❌ 0x00451c70 — FUN_00451c70                 calls AddParticipantNoCount (alongside SetupRaceTrackRandom)

ZONE 5 — Race teardown / cleanup
─────────────────────────────────────────────────────────────────
26. ❌ 0x0053bbb0 — FUN_0053bbb0                 calls cam-reset FUN_004a5500 at 0x0053d857 (race teardown) ⭐⭐
27. ❌ 0x00455500 — FUN containing 0x0045548e   (parent of the camera-reset call site) — likely FUN_00455420

ZONE 6 — Entity physics plumbing
─────────────────────────────────────────────────────────────────
28. ❌ 0x0049faf0 — FUN_0049faf0                 entity state fn-ptr (in weapon/camera init)
29. ❌ 0x0049c0f0 — FUN_0049c0f0                 called in camera entity reset loop
30. ❌ 0x0049c110 — FUN_0049c110                 physics param in InitWeaponSlot state==0
31. ❌ 0x0049b650 — FUN_0049b650                 vector math (InitWeaponSlot)
32. ❌ 0x0049d340 — FUN_0049d340                 motion state update (InitWeaponSlot state==1)
33. ❌ 0x0049e560 — FUN_0049e560                 physics update (InitWeaponSlot state==1)
34. ❌ 0x005a7a20 — FUN_005a7a20                 sqrt-NaN handler

ZONE 7 — Car pool helpers
─────────────────────────────────────────────────────────────────
35. ❌ 0x0043c0e0 — FUN_0043c0e0                 car index resolver (used in RaceSessionSetup)
36. ❌ 0x0043c1e0 — FUN_0043c1e0                 skin index resolver
37. ✅ 0x0044a250 — CheckIfDifficultyTierCompleted()  unlock/progression checker (Batch 11)

ZONE 8 — Networking / multiplayer
─────────────────────────────────────────────────────────────────
38. ❌ 0x0052f770 — FUN_0052f770                 1013-line multiplayer packet handler ⭐⭐
39. ❌ 0x0052d470 — FUN_0052d470                 ENet event type 2 handler
40. ❌ 0x005312a0 — FUN_005312a0                 ENet event type 3 handler
41. ❌ 0x0052c660 — FUN_0052c660                 ENet event type 1 handler

ZONE 9 — Rendering
─────────────────────────────────────────────────────────────────
42. ❌ 0x00492150 — FUN_00492150                 DrawTexturedQuad — 510 call sites
43. ❌ 0x00491f60 — FUN_00491f60                 called in sync-screen case 4
44. ❌ 0x004884d0 — FUN_004884d0                 called in sync-screen case 4
45. ❌ 0x00539fa0 — FUN_00539fa0                 called in sync-screen case 4

ZONE 10 — Utility / string / math
─────────────────────────────────────────────────────────────────
46. ❌ 0x005334f0 — FUN_005334f0                 memcpy/strcpy helper (used everywhere)
47. ❌ 0x00533360 — FUN_00533360                 AI car randomizer
48. ✅ 0x00533aa0 — strlen()                      custom string length (Batch 11)
49. ✅ 0x00452780 — GetTrackIdByName()            track ID resolver (Batch 11)
50. ✅ 0x00452130 — GetTrackInfo()                track data loader (Batch 11)
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
  _DAT_00f3eb04 = ⭐ live participant count (max 0x1e=30)
  DAT_00f3eb00  = ⭐ player race index in participant table
  DAT_00f3eb14  = per-entry header array (stride 0x14: type | flags2)
  DAT_00f3eb30  = per-entry car+skin index (stride 0x28: skinIdx | carPoolIdx)

CONFIRMED — Camera entity system (NOT car structs):
  DAT_016f8ee0  = camera entity inline array (stride 0x140, 9 slots)
  DAT_016f8e40–016f8e80 = 9 camera entity pointers
  DAT_016f8e60  = player primary camera entity ptr
  DAT_016f8e98  = node count for pickup/camera node array
  DAT_016f8ea0  = camera node array base (stride 0x38, FindNearestPickupNode)

CONFIRMED — Runtime car slot system:
  DAT_0acee7e8  = ⭐⭐ pointer to the PLAYER runtime car slot (0xAAC bytes)
  DAT_0acee9e8  = ⭐⭐ linked list head of ALL runtime race car structs (node+0x10 = next)
  slot[1]       = ⭐⭐ car type: 1=player, 9=spectator/disabled, 4=special
  slot[0x423]   = spectate flag byte

UNKNOWN — runtime car array flat base:
  WHO allocates the 0xAAC slots? → FUN containing 0x0044530c (PRIORITY #1 export)
  FUN_004ab810 (InitCarSlotFull) INITIALIZES a slot but is CALLED FROM outside
```

---

**Next Action:** In Ghidra, navigate to address `0x0044530c` → find the parent function (use "F" key or Function Start). Export that function — it calls `InitCarSlotFull` 3× (at 0x0044530c, 0x00445401, 0x00445511) and is the **race car slot builder**. Also export `FUN_0044cb60` (0x0044cb60) and `FUN_004f03a0` (0x004f03a0) which are other callers of `InitCarSlotFull`.

---

## 🆕 Batch 5 — New Functions (Main Loop, Race State, Audio, Timer)

### ⏰ Timer / Delta Time System
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x005250c0 | **CalcDeltaTime()** | ✅ Analyzed | ⭐ Core frame-timing function. Called every frame. Updates `DAT_0065c03c` (DeltaTime float). Reads `SDL_GetPerformanceCounter`, computes dt = FrameTicks / PerfFrequency, caps at 100ms/frame. Optional slowmo-adjust via `DAT_0ae48da0` (GameSpeedPercent). |
| 0x00525350 | AccumulateDeltaTime() | ✅ Analyzed | Accumulates `DAT_0ae48dd8` into `_DAT_0ae48e10`. Gates `DAT_0065c03c = 0.0` if under threshold (rapid frames). |
| 0x00525470 | GetPerfCounter() | ✅ Analyzed | Returns `SDL_GetPerformanceCounter()` (raw perf ticks). Used as timestamps throughout engine. |

### 🔴 Race State Machine
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x0048a560 | **SetRaceState(int state)** | ✅ Analyzed | Sets `DAT_016942dc`. Transitions 0→1 initializes StateBlend to -0.2; 1→3/4 initializes to +0.6. Guards against same-state transitions. |
| 0x0048a5c0 | **GetRaceState()** | ✅ Analyzed | Returns `DAT_016942dc` (int, 0-6). Called ~15 times per frame. |
| 0x0048a5d0 | **UpdateRaceState()** | ✅ Analyzed | Advances state blend by `DeltaTime × 4.0`. State 1 → 5 (paused) when blend > 1.2. State 3 → 6 (end) when blend < -0.2. |
| 0x004995f0 | LoadLapImages() | ✅ Analyzed | Loads lap-specific image assets for `DAT_016f7908` (current lap). Loads "gfx/devlogoNa/b/c.bmp" variants. |
| 0x00499c40 | SortCarsForRace() | ✅ Analyzed | Complex car ordering/priority matrix algorithm. Sorts up to N cars by score matrix for position assignment. |

### 🚀 Level Load System
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00404c30 | **LevelLoad()** | ✅ Analyzed | ⭐ Full level loading pipeline. Sets up viewport, loads track textures, spawns loading thread, renders loading screen with progress bar (% from `DAT_006e3968`), waits for thread. Calls `FUN_0045f690` + `FUN_004a1170` at end. |
| 0x00401610 | FrameUpdateDispatcher() | ✅ Analyzed | Idle/unfocused handler. Calls `FUN_005250c0()` (CalcDeltaTime), updates time accumulator, iterates car linked list. Called when `DAT_0065c021 = 0` (window minimized) with `SDL_Delay(10)`. |

### 🎵 Audio System — Init / Lifecycle
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00527230 | **AudioInit()** | ✅ Analyzed | ⭐ Initializes OpenAL via `FUN_00561a20`. Prints vendor/version. Calls `alDistanceModel(0)`, `alGenSources(1, &MusicALSourceID)`. Sets FLUID_SOUNDFONT env var. Sets `DAT_006e3434=1` on failure. |
| 0x00527400 | **AudioShutdown()** | ✅ Analyzed | Stops music stream, `alDeleteSources(1, &MusicALSourceID)`, calls `FUN_00561ad0()` (OpenAL context shutdown). |
| 0x005277d0 | **LoadAllSfx()** | ✅ Analyzed | ⭐ Loads all 38 built-in WAV SFXs from `PTR_s_wavs_moto_wav_00661c40` table. Allocates up to 128 AL sources via `alGenSources`. Stores buffer IDs in `DAT_0ae4d240[]`, source handles in `DAT_0ae4c640[]`. |
| 0x00527a20 | **FreeAllSfx()** | ✅ Analyzed | Stops+deletes all AL sources and buffers. Clears `DAT_0ae4d240[]`, `DAT_0ae4da40[]`, `DAT_0ae48e50[]`. |
| 0x00527ba0 | **StopAllSfx()** | ✅ Analyzed | Calls `alSourceStop` on all active sources. Also stops music if playing. Clears loop-sfx array. |

### 🎵 Audio System — SFX Playback
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00526d70 | **GetFreeSfxSource(priority)** | ✅ Analyzed | ⭐ Source preemption logic. Finds AL_STOPPED/AL_INITIAL source. If none free, steals lowest-priority playing source. Returns ptr into `DAT_0ae4c640` pool. |
| 0x00527520 | LoadSfxSlot(int slot) | ✅ Analyzed | Loads WAV file for slot index from `PTR_s_wavs_moto_wav_00661c40[slot]` into `DAT_0ae4d240[slot]`. |
| 0x00527630 | GetSfxSlotByName(char* name) | ✅ Analyzed | Searches built-in SFX names (0-105) then custom name table. Returns slot index. Returns 0xFFFFFFFF if not found. |
| 0x005279b0 | FreeSfxSlot(int slot) | ✅ Analyzed | Deletes AL buffer for dynamic/custom SFX slot (slot >= 0x6A). Clears custom name entry. |
| 0x00527e50 | FindSfxInstanceByID(int sfxID) | ✅ Analyzed | Returns oldest playing instance of a given SFX ID from `DAT_0ae4c640` pool. Used for looping checks. |
| 0x00527f20 | **PlaySfxSimple(sfxID, vol, pan, pitch, loop)** | ✅ Analyzed | ⭐ One-shot SFX playback. Sets `AL_BUFFER`, `AL_LOOPING`, `AL_GAIN=(vol*MasterVol)/10000`, `AL_POSITION=(pan/50-1, 0, 0)`, `AL_PITCH=pitch/22050`. Calls `alSourcePlay`. |
| 0x005280a0 | **PlaySfx3D(sfxID, vol, pos3D, ...)** | ✅ Analyzed | 3D positional SFX. Uses `FUN_005263c0` for 3D attenuation calc before `alSource3f`. |
| 0x005282d0 | **StartLoopingSfx(sfxID, priority, loopFlag, pos, handle)** | ✅ Analyzed | ⭐ Allocates `LoopSfxHandle` entry (stride 0x38), binds to free AL source, calls `alSourcePlay`. Returns int* handle for later updates. |
| 0x005284b0 | ChangeSfxAndStop(int* handle, int newSfxID) | ✅ Analyzed | Stops existing AL source linked to handle, sets new SFX ID in handle. Used to swap loop sounds (e.g. engine pitch change). |
| 0x00528510 | StopLoopingSfx(longlong loopState) | ✅ Analyzed | Stops AL source at `loopState+0x18` ptr, clears the ptr. |
| 0x00528550 | ClearLoopingSfx(longlong loopState) | ✅ Analyzed | Clears the `al_src_entry` pointer in the `LoopSfxHandle` array without stopping. |
| 0x00527b80 | StopSfxInstance(longlong instance) | ✅ Analyzed | Calls `alSourceStop(instance+0x10)` (= stop one AL source). |

### 🎵 Audio System — Focus / Window Events
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00527cf0 | **PauseAllSfx()** | ✅ Analyzed | Called on `SDL_WINDOWEVENT_FOCUS_LOST`. Pauses all AL_PLAYING sources via `alSourcePause`. Pauses music via `FUN_00565880`. Sets `DAT_0ae48e34 = 1`. |
| 0x00527da0 | **ResumeAllSfx()** | ✅ Analyzed | Called on `SDL_WINDOWEVENT_FOCUS_GAINED`. Resumes all AL_PAUSED sources via `alSourcePlay`. Resumes music. Sets `DAT_0ae48e34 = 0`. |

### 🎵 Audio System — Music / BGM
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x005285b0 | **PlayMusic(char* file)** | ✅ Analyzed | ⭐ Starts BGM. Loads OGG/WAV stream via `FUN_00563f60(file, 250000)`. Binds to `MusicALSourceID`. Plays with `FUN_005668b0(source, stream, 3, looping, callback, 0)`. Sets volume via `alSourcef(AL_GAIN)`. |
| 0x00526f10 | **UpdateMusicPlayback()** | ✅ Analyzed | Advances to next track (`MusicTrackIndex`). Loads `"redbook/track%02d.%s"` format. Calls `FUN_005668b0` to start playback. Updates `MusicVolume` → `alSourcef(AL_GAIN)`. |
| 0x00526b60 | ScanMusicDirectory() | ✅ Analyzed | Scans current music directory, collects file names matching supported extensions into `DAT_0ae5db40[]` sorted array. Max 0xFF entries = `MusicFileCount`. |
| 0x00527400 | AudioShutdown() | ✅ Analyzed | Already listed above — also handles music stream cleanup. |

### 🔧 Audio Low-Level (OpenAL wrappers)
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00561a20 | OpenAL_Init() | ✅ Analyzed | `alcOpenDevice` + `alcCreateContext` + `alcMakeContextCurrent`. Returns true/false. |
| 0x005618a0 | OpenAL_GetError() | ✅ Analyzed | Returns error string from OpenAL context. |
| 0x00561ad0 | OpenAL_Shutdown() | ✅ Analyzed | `alcDestroyContext` + `alcCloseDevice`. |
| 0x00562e90 | LoadWAVToBuffer() | ✅ Analyzed | Loads WAV file data, calls `alBufferData`. Returns AL buffer ID or 0 on fail. |
| 0x00563f60 | OpenMusicStream() | ✅ Analyzed | Opens OGG/WAV file as streaming object (250000 byte buffer). Returns stream ptr or 0. |
| 0x00563ad0 | CloseMusicStream() | ✅ Analyzed | Frees music stream object. |
| 0x00565880 | PauseMusicStream() | ✅ Analyzed | Pauses streaming AL source. |
| 0x00565990 | ResumeMusicStream() | ✅ Analyzed | `alSourcePlay` on streaming source. |
| 0x00565aa0 | SetMusicVolume(float vol) | ✅ Analyzed | `alSourcef(MusicALSourceID, AL_GAIN, vol)`. |
| 0x00565e90 | StopMusicStream() | ✅ Analyzed | `alSourceStop` + cleanup. |
| 0x005668b0 | StartMusicPlayback() | ✅ Analyzed | Binds stream to source, sets loop mode, callback. Starts streaming playback. |
| 0x005263c0 | Calc3DAtten() | ✅ Analyzed | Computes 3D sound attenuation/panning from position. Used by PlaySfx3D. |

### 🖼️ Rendering Pipeline — GL State Management
> GL function pointers all at `DAT_0aea****` (loaded from libGLESv2.dll). State cache at `DAT_0ae7fc**`.

| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x004628b0 | **DrawTexturedQuad(x,y,w,h,u1,v1,u2,v2,color,blend,texid)** | ✅ Analyzed | ⭐ Core 2D sprite/HUD primitive. Converts screen coords (via ScreenCenterX/Y + DAT_006e3468/46c scale) to NDC. Builds 4 vertices (stride 0x2c). If `DAT_0ae7fc4c`: pushes to DAT_00f40dc0 batch buffer (stride 0xb8). Else: glVertexAttribPointer x4 + glDrawArrays(GL_TRIANGLE_STRIP, 0, 4). param_11: 0=alpha-blend, 1=additive, -1=no-state-change. |
| 0x004884d0 | **GL_SetupNormalRenderState()** | ✅ Analyzed | Standard state setup: glDepthMask(1), glEnable(GL_DEPTH_TEST), glDepthFunc(GL_LESS=0x203), glBlendFunc(SRC_ALPHA,ONE_MINUS_SRC_ALPHA). Caches in DAT_0ae7fc8c/8d/80. |
| 0x004827e0 | **FlushBatchedQuads()** | ✅ Analyzed | Flushes 2D VAO sprite batch. Disables GL_CULL_FACE(0xb44), glDepthMask(0), disables depth test. Calls FUN_00461d30+FUN_004621b0+UBO/shader setup. Binds VAO(DAT_01694260), sets 5 attrib ptrs (stride 0x2c), loops DAT_01694230 batch (DAT_01694228 entries), glDrawArrays per entry. |
| 0x004886b0 | **RenderSceneFull()** | ✅ Analyzed | ⭐ THE main full-pass renderer. Guards: DAT_0ae7fc4c!=0. Resets DAT_00f43a00=0. Sequence: GL_SetupNormalRenderState → CommitFrameRenderWork → SortDynamicObjects → SortTrackMeshObjects → DrawGameObjects → (optionally) UpdateGhostRender/RenderCarMeshes/RenderSkidmarksShadows → FlushBatchedQuads → ClearDynamicMeshBatch + ClearTrackMeshBatch + ClearGameObjectsCount. |
| 0x00488750 | **RenderSceneLightBatch()** | ✅ Analyzed | Lighter render pass. Skips dynamic/gameobj sorts. Calls: GL_SetupNormalRenderState → (depth-off override) → SortTrackMeshObjects → optional ghost/car/skid → FlushBatchedQuads. |
| 0x00488800 | **SetFogDistances(float near, float far)** | ✅ Analyzed | DAT_006e347c=near, DAT_006e3480=(far*DrawDistLevel)/5. Computes DAT_006e3484=(near+far)/range, DAT_006e3488=(2*near*far)/range. |
| 0x004888a0 | **SetFogParams(float a, float b, float c)** | ✅ Analyzed | Sets DAT_006e3494 (near clip), DAT_006e3490 (fog near), DAT_006e348c (fog mid), DAT_006e34a0 (linear slope = 1/(c-b)). |

### 🖼️ Rendering Pipeline — Mesh Sort & Submit
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x004b6b60 | **SortTrackMeshObjects()** | ✅ Analyzed | Iterates DAT_018146b0 (stride 0x2d8, count DAT_018146a8). Shadow pass if DAT_0ae7fc63. Bucket-sorts visible entries (obj+0xc0==-1) into DAT_01813a80[] linked list (193 texture-slot buckets, keyed by obj+0x5c). |
| 0x004b7700 | **ClearTrackMeshBatch()** | ✅ Analyzed | DAT_018146a8=0, zeros DAT_018140a0[0xc1] + DAT_01813a80[0xc1]. |
| 0x00500f60 | **SortDynamicObjects()** | ✅ Analyzed | Parallel to SortTrackMeshObjects for dynamic objects. Uses DAT_0adce3c0/3ac, buckets DAT_0adcdda0[]/DAT_0adcd780[]. Optional shadow via FUN_004fdef0. |
| 0x00501c10 | **ClearDynamicMeshBatch()** | ✅ Analyzed | Zeros DAT_0adcdda0/d780/d160/ccb40[0xc1] each. Resets DAT_0adce3ac=0, DAT_0adce3a8=0. |
| 0x00501c80 | **FreeDynamicMeshBuffers()** | ✅ Analyzed | Frees all dynamic car mesh device buffers. Iterates DAT_0adccb08 (stride 0x1e0), frees +0x48/0x98/0xa8/0xb0. Frees DAT_0adccb18/10/ccae0. |
| 0x004d4260 | **DrawGameObjects()** | ✅ Analyzed | Renders DAT_0ace47b0 (stride 0x2d8, count DAT_0ace47a8). Per object: binds VAO(obj[0]), glUseProgram(0x8892,0x8893), 5x glVertexAttribPointer (stride 0x2c), FUN_00480fb0(obj,0) draw call, toggles GL_CULL_FACE on obj[0x2d], texture bind from obj_meshinfo+4. |
| 0x004d46e0 | **ClearGameObjectsCount()** | ✅ Analyzed | DAT_0ace47a8=0. |
| 0x004d46f0 | **FrustumCullSphere(float* center, float radius, float* zdist)** | ✅ Analyzed | Tests sphere vs 4 frustum planes (DAT_016f98c0..016f98fc). Returns 0=rejected,1=partial,2=inside. Clamps to [DAT_006e347c,DAT_006e3480] depth range. |
| 0x0053de80 | **CommitFrameRenderWork()** | ✅ Analyzed | FUN_0053b020 (VAO/VBO commit) + FUN_0053b530 (UBO update). Stamps DAT_0ae86180 draw list entries with frame tag. Sets _DAT_0ae7e9bc=1. |
| 0x0053df00 | **ClearMeshBatchState()** | ✅ Analyzed | Zeros 9 counters at DAT_0ae85e20..0ae85e54. |
| 0x0053df60 | **SetShaderUniformBlock(slot,a,b,c)** | ✅ Analyzed | Updates 16-slot uniform cache (DAT_0ae85e80, stride 0xc). Calls glUniform4iv(0x8a11) only on change. |
| 0x004b77d0 | **TransformMeshVertices(mesh, rotmat, pos)** | ✅ Analyzed | CPU vertex transform: multiplies each vertex by 3x3 rotation + translate. Stores at vertex+0x24..+0x2c. Used for shadow/reflection prep. |

### 🖼️ Rendering Pipeline — Per-Object Draw Helpers
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00480cd0 | **AddToSemiTransparentList(obj)** | ✅ Analyzed | Appends obj ptr to `DAT_01694220` (dynamic realloc, starts 0x1000 entries). Increments `DAT_01694218`. Clears `obj+0x2d0`. Used for delayed transparent/occluded object rendering. |
| 0x00480d60 | **InitSemiTransparentList()** | ✅ Analyzed | malloc(0x8000) for `DAT_01694220`, capacity `DAT_0169421c = 0x1000`. |
| 0x00480dd0 | **FreeSemiTransparentList()** | ✅ Analyzed | Frees `DAT_01694220`, `DAT_0169421c = 0`. |
| 0x00480e00 | **SetupMeshRenderSlot(obj)** | ✅ Analyzed | Allocates GL resource slot for mesh. If `obj+0x5c == -1`: calls `FUN_0053a070(obj+0x58, flags)` to get batch slot. Stores opaque slot at `obj+0x5c/0x60`, transparent/double-sided at `obj+0x64/0x68`. If shadow pass: updates `obj+0x6c` shadow flags. |
| 0x00480fb0 | **DrawMeshObject(obj, flags)** | ✅ Analyzed | ⭐ THE per-object draw call. Calls SetupMeshRenderSlot. Selects opaque vs transparent VBO (flag bit2). Calls `FUN_00539fd0(mesh, slot)` to bind VBO+VAO. Per flag bits: 0=shadow map, 1=env map, 2=transparent, 3=light map, 4=specular, 5=normal map, 8=decal. Shadow mode: calls SetShaderUniformBlock for material color + world matrix (`obj+8..0x34`). |
| 0x00481320 | **RenderCarMeshes()** | ✅ Analyzed | Iterates `DAT_01694240` (count `DAT_01694238`). For visible entries (`obj+0xc0 == -1`): binds shared VAO `DAT_016942b8/bc`, sets 5 vertex attribs (stride 0x2c), calls DrawMeshObject(obj,0), binds texture from `obj+0x10+4`. Non-visible: deferred to AddToSemiTransparentList. |
| 0x00481760 | **RenderTransparentAndSortedObjects()** | ✅ Analyzed | Two-pass semi-transparent renderer. **Pass 1**: Shell-sorts `DAT_01694220[DAT_01694218]` by `obj+0xc4` (depth). Merges consecutive same-mesh entries (obj+0xac count). Renders with blend by `obj+0x30`: 0=SRC_ALPHA/DST_ALPHA, 1=additive (GL_ONE,GL_ONE), 2=premult (GL_ONE,1-SRC_ALPHA). Incr `DAT_00f43a18` by tri count. **Pass 2**: skipped objects (obj+0xb9). Shell-sort gaps from `DAT_00676384..006763a0`. |
| 0x0047d220 | **UpdateGhostRenderEntries()** | ✅ Analyzed | Processes ghost car replay for rendering. Reads `DAT_016942a0` (stride 0x180, count `DAT_016942ac`). Filters active ghosts (obj+0x174 != 0.0). Shell-sorts by depth `obj+0x174`. Builds render packets at `DAT_016942c0` (stride 0x38, count `DAT_016942c8`). Each packet: `+0x1c`=playerIdx, `+0x20`=animState, `+0x24`=depthZ. |

### 🖼️ Rendering Pipeline — Shader/GPU Buffer Layer (Batch 10)
> These are the lowest-level GL abstraction functions — VBO allocation, data upload, draw submission, and 2D sprite batch flush.
| Address | Name | Status | Description |
|---------|------|--------|-------------|
| 0x00461d30 | **FlushBatchedPanelPolys()** | ✅ Analyzed | Flushes 2D panel/HUD quad batch. Guards on `DAT_00f40dca > 0`. Builds unique (texId, blendMode) groups into `DAT_00f40dd8` (stride 0x38, count `DAT_00f40de0`). Calls `FUN_0053a460` to alloc VBO+EBO (`DAT_00f40dcc/dd0`, usage=GL_DYNAMIC_DRAW=0x88e0). For each group: copies vertex data into `DAT_0ae7e998` (vertex staging) + builds quad index list (0,1,2,2,0,3 per quad) into `DAT_0ae7e990` (index staging). Calls `FUN_0053a800(vtx_ptr, vtx_count, idx_ptr, idx_count)` = glBufferSubData upload. Stores (VBO_handle, draw_count) at group+0x30. Calls `FUN_0053ab10()` to submit. Resets `DAT_00f40dca=0`. |
| 0x004621b0 | **DrawBatchedPanelPolys()** | ✅ Analyzed | Issues actual GL draw calls for panel batch. Guards on `DAT_00f40de0 > 0`. If shader changed vs `DAT_0ae7fca8`: binds vertex shader (0x8892) + fragment shader (0x8893) via `DAT_0aea2028`, sets 5 `glVertexAttribPointer` attribs (stride 0x2c), enables attribs 0-4. Per group: sets blend state by mode (0=SRC_ALPHA/ONE_MINUS_SRC_ALPHA, 1=GL_ONE/GL_ONE additive). Binds texture: `glBindTexture(0xde1, DAT_016cf7b8[texId*0x4a])`. Calls `FUN_0053ab80(4, vbo_handle, draw_count, 1)` = indexed glDrawElements. Increments `DAT_0ae7e9ac` (GL state change counter) on every state change. |
| 0x00491900 | **FlushBatchedTextPolys()** | ✅ Analyzed | Same as FlushBatchedPanelPolys but for text rendering system. Source: `DAT_016cf560` (capacity `DAT_016cf568` = 0x1000 quads = 0xb0000 byte pool). Builds VBO/EBO pair `DAT_016cf56c/570`. Stores result at `DAT_016cf5a8` = CONCAT44(VBO_handle, index_count). Calls `FUN_0053ab10()`. Resets `DAT_016cf56a=0`. |
| 0x00491be0 | **DrawBatchedTextPolys()** | ✅ Analyzed | Issues draw calls for text batch. Guards on `DAT_016cf5ac > 0`. Same shader/attrib bind pattern as DrawBatchedPanelPolys. Disables GL_CULL_FACE (0xb60) instead of 0xb44. Enables GL_BLEND (0xbe2), sets SRC_ALPHA/ONE_MINUS_SRC_ALPHA blend. Calls `FUN_0053ab80(4, vbo, count, 1)`. `DAT_0ae7fc92` = SSO (separate shader objects) flag — if set, calls `(*DAT_0aea2d70)(mode)` instead of glUseProgram pair. |
| 0x0053a070 | **FindBestGLResourceSlot(stream, flags)** | ✅ Analyzed | Searches slot table `DAT_0ae7e9e8` (stride 0x20/slot, 0x188 entries per stream = 49 slots). Finds best available slot matching `flags` bitmask. Two search modes: bit2=0 → find partial match (most flags); bit2=1 → find exact match. Returns best slot index or -1 if none. Used by SetupMeshRenderSlot to allocate VAO/VBO resource slots. |
| 0x0053a460 | **AllocOrResizeVBO(vtxBytes, idxBytes, vbo, ebo, usage)** | ✅ Analyzed | Thread-safe VBO+EBO allocation/resize. If VBO handle==0: `(*DAT_0aea3398)(1, vbo)` = glGenBuffers. Binds vertex shader (0x8892) + fragment shader (0x8893), sets 5 attribs. Calls `(*DAT_0aea2308)(GL_ARRAY_BUFFER, size, NULL, usage)` = glBufferData. Same for EBO. usage=0x88e0=GL_DYNAMIC_DRAW. Thread dispatch via `FUN_00404b20` if off main thread. |
| 0x0053a800 | **UploadVBOData(vtx_ptr, vtx_count, idx_ptr, idx_count)** | ✅ Analyzed | Uploads vertex + index data to GPU. Calls `(*DAT_0aea2308)(GL_ARRAY_BUFFER, size, NULL, usage)` if buffer too small (grows to next power-of-2). Then `(*DAT_0aea2348)(GL_ARRAY_BUFFER, 0, size, data)` = glBufferSubData for vertex data. Same pattern for index EBO. Returns VBO handle. `DAT_0ae7fc68` flag selects 16-bit vs 32-bit indices. Thread-safe (SDL_LockMutex + cross-thread dispatch). |
| 0x0053ab80 | **GLDrawCall(primitive, count, idx_count, indexed)** | ✅ Analyzed | THE final GPU draw command. `indexed != 0`: calls `(*DAT_0aea2e28)(primitive, idx_count, GL_UNSIGNED_SHORT/INT, offset)` = **glDrawElements**. `indexed == 0`: calls `(*DAT_0aea2da0)(primitive, count)` = **glDrawArrays**. `DAT_0ae7fc68` flag picks index type: 0→GL_UNSIGNED_SHORT(0x1403), 1→GL_UNSIGNED_INT(0x1405). |
| 0x0053b020 | **SetupFrameUBOs()** | ✅ Analyzed | Per-frame uniform/UBO commit. Extracts fog color from `DAT_0ae7e9a0` (packed RGBA → R,G,B floats /255). Reads fog density from `DAT_006e3490`, fog matrix from `DAT_006e3480`. Iterates program list `DAT_0ae86180`: for each program calls `(*DAT_0aea6d38)(prog)` = glUseProgram/glBindProgramPipeline. Then `(*DAT_0aea6a00)(loc, fogDensity, fogMatrix)` or `(*DAT_0aea5750)(prog, loc, density, matrix)` = glUniform1f/glProgramUniform1f. Also `(*DAT_0aea6ab0)(loc, fogR, fogG, fogB)` = glUniform3f for fog color. Cached in `DAT_0ae7fcac`. |
| 0x00462090 | **FreePanelBatchBuffers()** | ✅ Analyzed | Frees panel quad batch GPU+CPU resources. glDeleteBuffers for VBO `DAT_00f40dcc` + EBO `DAT_00f40dd0`. Frees CPU staging `DAT_00f40dc0` + `DAT_00f40dd8`. Zeros `DAT_00f40dc8=0`. |
| 0x00462140 | **InitPanelBatchBuffers()** | ✅ Analyzed | Allocates panel quad batch buffers. `DAT_00f40dc8=0x80` (128 capacity). malloc(0x5c00) → `DAT_00f40dc0` (cpu quad buffer). malloc(0x1c00) → `DAT_00f40dd8` (group/bucket buffer). |
| 0x00491b00 | **FreeTextBatchBuffers()** | ✅ Analyzed | Frees text quad batch GPU resources. glDeleteBuffers for `DAT_016cf56c` + `DAT_016cf570`. Frees cpu `DAT_016cf560`. Zeros `DAT_016cf568=0`. |
| 0x00491b90 | **InitTextBatchBuffers()** | ✅ Analyzed | Allocates text quad batch. `DAT_016cf568=0x1000` (4096 cap). malloc(0xb0000) → `DAT_016cf560`. Then immediately calls FlushBatchedTextPolys() to init VBO. |

---

## 🆕 Batch 11 — Printf Engine, BigNum, VFS, Progression, CSV Logging

### 🖨️ Custom Printf / vsnprintf Engine
> RVGL ignores the OS libc printf entirely and ships its own complete formatting stack to guarantee byte-identical cross-platform float output. Call tree: `SafeVsnprintf` → `vsnprintf` → `InternalFormatEngine` → output primitives + float engine + BigNum arena.

| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x005bd560 | **SafeVsnprintf()** | ✅ Analyzed | Top-level format wrapper. Clamps buffer to 255 bytes, writes, then UTF-8 heals the tail byte. Used by `WriteLogMessage` and `File_Vfprintf`. |
| 0x005acdf0 | **vsnprintf()** | ✅ Analyzed | Standard vsnprintf shim — delegates directly to `InternalFormatEngine`. |
| 0x005b27f0 | **InternalFormatEngine()** | ✅ Analyzed | Master printf formatting engine. Handles all format specifiers: `%d/%i/%u/%x/%o/%f/%e/%g/%a/%s/%c/%wc/%ws`. Dispatches to output primitives, the float sub-engine, and the BigNum path for exact decimal representation. |
| 0x005a9230 | **vfprintf_l() / PrintToStdout()** | ✅ Analyzed | Safe vfprintf wrapper with stream locking. Acquires file-stream critical section before calling the format engine. Used for all log output to stdout. |
| 0x00534640 | **LogWarning()** | ✅ Analyzed | Variadic warning emitter — formats a string with printf-style args and forwards to `WriteLogMessage`. |
| 0x00529490 | **WriteLogMessage()** | ✅ Analyzed | Master log dispatcher. Formats, sanitizes, and fans the message out to both the in-game console window and the log file. All game log output passes through here. |

### 🖨️ Printf Output Primitives
| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x00560fc0 | OutputNarrowString() | ✅ Analyzed | Writes a standard narrow C-string (`char*`) to the active format buffer. |
| 0x005b0d30 | OutputWideString() | ✅ Analyzed | Writes a wide-character string (`wchar_t*`) to the format buffer. |
| 0x005b0e90 | OutputNarrowChar() | ✅ Analyzed | Writes a single narrow character to the format buffer. |
| 0x005b0cd0 | OutputSingleChar() | ✅ Analyzed | Outputs one hardcoded character — used for padding/fill bytes. |
| 0x005b10e0 | WriteStreamData() | ✅ Analyzed | Flushes accumulated characters to the string output buffer or file stream. |
| 0x005b15e0 | FormatBase10Int() | ✅ Analyzed | Converts and writes standard base-10 decimal integers (`%d/%i`). |
| 0x005b10c0 | FormatHexOctalInt() | ✅ Analyzed | Converts and writes hex (`%x/%X`), octal (`%o`), or unsigned (`%u`) integers. |
| 0x005b61b0 | OutputDecimalPoint() | ✅ Analyzed | Prints a decimal point character using locale rules (`.` vs `,`). |
| 0x005b7020 | FormatThousandsSeparator() | ✅ Analyzed | Multibyte-to-Wide conversion for locale thousand-separator strings. |
| 0x005b7380 | _wctomb_internal() | ✅ Analyzed | Internal worker for `wctomb` — handles locale-aware wide→multibyte conversion. |
| 0x005b7410 | wctomb() | ✅ Analyzed | Public `wctomb` entry — converts one wide character to its multibyte sequence. |
| 0x00566c60 | wcsnlen() | ✅ Analyzed | Wide-string length with a maximum-count safety cap. |
| 0x00533aa0 | strlen() | ✅ Analyzed | Custom string-length measurer (replaces libc `strlen` to avoid OS libc dependency). |
| 0x00533880 | String_CompareIgnoreCaseUTF8() | ✅ Analyzed | Case-insensitive string comparison with proper UTF-8 codepoint handling (`strncasecmp` equivalent). |
| 0x00533f60 | ReadNextToken() | ✅ Analyzed | Lexer / token reader — reads the next whitespace-delimited token from a config text stream. *(Previously listed as `ReadLineFromFile`.)* |
| 0x005b7560 | get_iob() | ✅ Analyzed | Returns a pointer to the internal C `FILE` array (`_iob`). Used by the printf engine for stdout/stderr access. |
| 0x005b76d0 | _unlock_file() | ✅ Analyzed | Unlocks a file stream after critical-section usage in the printf/log system. |

### 🖨️ Float Formatting Sub-Engine
| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x005b0bb0 | ConvertFloatToString() | ✅ Analyzed | Core float→string math engine (`_fltout` equivalent). Entry point for all `%f/%e/%g` conversions. Delegates to BigNum for exact precision. |
| 0x005b1af0 | OutputFloatData() | ✅ Analyzed | Formatting wrapper that prints the actual float digit string, applying sign, exponent, padding, and width rules. |
| 0x005b2250 | FormatHexFloat() | ✅ Analyzed | Engine for `%a` and `%A` (hexadecimal float) format specifiers. |
| 0x005b3690 | _fptostr() | ✅ Analyzed | Core ASCII digit generator for mantissas (`_ftoa_engine` equivalent). Generates raw digit strings from IEEE 754 float bits. |
| 0x005b1010 | OutputNaNInf() | ✅ Analyzed | Safely outputs `"NaN"` or `"Inf"` strings when the float value is a special IEEE 754 case. |

### 🔢 BigNum Arbitrary-Precision Engine
> To guarantee byte-identical cross-platform floating-point decimal output, RVGL ships a full arbitrary-precision math engine. It is used exclusively by the float→string path (`ConvertFloatToString` + `_fptostr`) to compute exact decimal representations without rounding drift. An arena allocator manages the BigNum pool to avoid heap fragmentation.

| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x005b5dc0 | **Bignum_Allocate()** | ✅ Analyzed | Allocates a Bignum structure from the pre-allocated pool. |
| 0x005b5ec0 | Bignum_Free() | ✅ Analyzed | Returns a Bignum structure to the linked-list free pool. |
| 0x005b5fe0 | Bignum_FromInt() | ✅ Analyzed | Creates a Bignum from a 32-bit integer value. |
| 0x005b6510 | Bignum_Compare() | ✅ Analyzed | Checks whether one Bignum is greater than another. |
| 0x005b5f30 | Bignum_Multiply() | ✅ Analyzed | Multiplies a Bignum by a 32-bit integer block. |
| 0x005b3510 | Bignum_ExtractDigit() | ✅ Analyzed | Performs long division on two Bignums and returns the top integer digit (core of decimal digit extraction). |
| 0x005b3470 | Bignum_StringAlloc() | ✅ Analyzed | Treats a Bignum block as a `char` array to store static digit strings during formatting. |
| 0x005b3430 | Bignum_BufferAlloc() | ✅ Analyzed | Allocates a dynamic scratchpad buffer during digit generation for intermediate results. |
| 0x005b56a0 | Bignum_ShiftRight() | ✅ Analyzed | Bitwise right-shift — divides a Bignum by a power of 2. |
| 0x005b6400 | Bignum_ShiftLeft() | ✅ Analyzed | Bitwise left-shift — multiplies a Bignum by a power of 2. |
| 0x005b57a0 | Bignum_CountTrailingZeros() | ✅ Analyzed | Bit-scan utility — counts trailing zero bits to determine lowest set bit. |
| 0x005b6200 | Bignum_MultiplyByPowerOf10() | ✅ Analyzed | Square-and-multiply algorithm combined with precalculated tables to scale by 10^n. |
| 0x005b60a0 | Bignum_MultiplyFull() | ✅ Analyzed | Multiplies two full Bignums together using schoolbook O(n²) multiplication. |
| 0x005b6560 | Bignum_Subtract() | ✅ Analyzed | General-purpose Bignum subtraction with handling for negative intermediate results. |
| 0x005b6730 | Bignum_ToDoubleNormalized() | ✅ Analyzed | Packs a Bignum back into a standard IEEE 754 64-bit double. High-performance BigNum→double packing utility. |

### 📁 Virtual File System (VFS) & Low-Level I/O
> The VFS resolves all asset paths between the read-only install directory and the writable user data directory. Mount priorities determine which physical path wins. Strings are ultimately converted to UTF-16 before Windows API calls.

| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x00534c6d | **VFS_ResolvePath()** | ✅ Analyzed | Resolves a virtual game path to an absolute OS filesystem path by walking active mount points in priority order. |
| 0x00535f20 | CheckVirtualMount() | ✅ Analyzed | Checks whether a given path segment exists as an archive or mount entry. |
| 0x00536070 | VFS_FindFileInMounts() | ✅ Analyzed | Searches all active mount points for a file. Returns the mount index where the file was found, or -1. |
| 0x00534f80 | Engine_OpenFile() | ✅ Analyzed | Opens a file after VFS path resolution. Converts the resolved UTF-8 path to UTF-16 via `AllocUTF8ToWide` then calls Windows `CreateFileW`. |
| 0x005a8ef0 | OS_GetFileStat() | ✅ Analyzed | Thread-safe, Unicode-aware file status checker (`VFS_Stat`). Returns file size and timestamps. |
| 0x005a7c90 | AllocUTF8ToWide() | ✅ Analyzed | Allocates and converts a UTF-8 string to a `wchar_t*` wide string for Windows API consumption. |
| 0x00406d00 | Path_ReplaceExtension() | ✅ Analyzed | Safely replaces a file extension in a path string in-place. |

### 📊 Race Result CSV Logging
| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x00473c50 | SessionLog_Init() | ✅ Analyzed | Generates the session log filename and initializes the `.csv` output file at race start. |
| 0x00473f10 | ExportRaceResultsToCSV() | ✅ Analyzed | Result Export Dispatcher — writes the final race standings to a CSV file at race end. |
| 0x00460360 | LogMatchSettings() | ✅ Analyzed | Logs physics mode, lap count, weapon settings, and lobby details to the session CSV. |
| 0x004604c0 | LogRaceStandings() | ✅ Analyzed | Master CSV dumper — writes the full final standings table including positions, times, names. |
| 0x0044a2f0 | CalculateDNFStandings() | ✅ Analyzed | Extrapolates final positions and race times for cars that Did Not Finish (DNF). |

### 🏆 Progression & Track Unlocks
| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x004541f0 | UpdateTrackUnlocks() | ✅ Analyzed | Track Availability Initializer — populates the `trackModesAvailable` bitmask for each track based on save data, difficulty completion, and active cheat flags. |
| 0x0044a250 | CheckIfDifficultyTierCompleted() | ✅ Analyzed | Progression checker — returns whether the player has completed all required races in a given difficulty tier. |
| 0x00449f50 | CheckIfTierTimeTrialsBeaten() | ✅ Analyzed | Checks whether all Time Trial star times have been beaten for a given tier. |
| 0x00452780 | GetTrackIdByName() | ✅ Analyzed | Converts a track folder name string to an integer track ID. Searches Vanilla tracks (IDs 0–20, static array) first, then Custom tracks (dynamic heap array, index offset by `×0x9D8`). |
| 0x00452130 | GetTrackInfo() | ✅ Analyzed | Takes a track index and returns a pointer to the `TrackInfo` struct for that track (0x78 / 120 bytes). |
| 0x0065a9b0 | TimeTrial_InitChallengeTimes() | ✅ Analyzed | Hardcodes the official Time Trial challenge target times for all Vanilla tracks at startup. |

### 🖥️ UI Popup System
| Address | Function Name | Status | Description |
|---------|--------------|--------|-------------|
| 0x00468200 | UI_PopupDispatcher() | ✅ Analyzed | Saves popup text into global memory and calculates a proportional popup panel width. Used for in-race notifications, error messages, and countdown popups. |

---

## 📌 Key Technical Notes (Batch 11)

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

Mount priority determines which root wins for a given path. All resolved paths are converted to UTF-16 (`AllocUTF8ToWide`) before Windows API calls, ensuring Unicode support.

### Graceful Modding & Job Objects
When using external injectors like `RVGLAutoInjector.exe`, wrap child processes in a Windows **Job Object** (`ChildProcessTracker` pattern) or add a `WaitForSingleObject` heartbeat. This automatically terminates the injector/tool when the main game process exits, preventing ghost processes.


