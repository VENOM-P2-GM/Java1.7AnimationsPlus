### Unreleased

- Fixed the cape physics segments not moving (the cape rendered again, but `cape_seg_1/2/3` stayed frozen)
    - `animation.player.cape_physics` is now played from the root animation controller's third-person
      state, right next to `animation.player.cape`, instead of only from `scripts.animate`. The dedicated
      cape render pass re-runs the animation controller state machine for the cape geometry, and an
      animation wired up only through `scripts.animate` is never applied there, so the physics animation
      never ran and the segments never moved. The controller entry keeps the same `v.is_valid_player`
      gate, so the physics still never applies in menus/paperdolls
- Strengthened the cape cloth response (the trailing/flutter was so weak it was barely visible)
    - Snappier wind sampling (speed smoothing 0.35 -> 0.5), stronger trailing (h-speed factor 1.6 -> 2.0,
      clamp 18 -> 24, per-segment gains 0.6/0.85/1.1 -> 0.95/1.4/1.85, response 0.45/0.38/0.3 ->
      0.55/0.48/0.4), restored swim/glide streaming terms that the pre_animation rewrite dropped
      (swim +8, glide +18), stronger yaw sway (`relative_cape_rotation * 10` -> `* 14`) and a more
      pronounced speed-scaled travelling flutter (`min(h_speed, 4) * 0.25` -> `min(h_speed, 6) * 0.35`
      with ~50% higher wave multipliers in the animation)
- Fixed the cape not rendering at all (it disappeared entirely after the cape physics update)
    - The cape render controller's geometry expression now references the segmented cape exactly the
      way the game's own vanilla cape render controller does: plain `Geometry.cape`, with the
      `cape` slot of `player.entity.json` bound to `geometry.cape` again. The previous update pointed
      the dedicated engine cape render pass at non-vanilla geometry identifiers through an
      `(q.is_emoting || v.is_in_menu)` ternary, after which the pass drew nothing
    - Menu/emote handling is unchanged: `animation.player.cape` already applies the in-menu pose
      (`v.is_in_menu`), like it always has
    - `rebuild_animation_matrices` is kept (it is required for the cape animation to run), and the
      segmented cape model + cape physics simulation are untouched, so the cape keeps its cloth
      physics while rendering like vanilla
- Fixed the cape not animating at all in game
    - `controller.render.player.cape` was missing `rebuild_animation_matrices` (it is part of the vanilla
      cape render controller). The cape is drawn with its own geometry, so without it the cape pass re-uses
      the player model's cached bone matrices - where the `cape` bone does not exist - and the cape renders
      frozen in its bind pose instead of following `animation.player.cape`
- Cape physics (segmented cape with simulated cloth response)
    - `geometry.cape` / `geometry.cape_root_parent` are now a chain of four bones
      (`cape` -> `cape_seg_1` -> `cape_seg_2` -> `cape_seg_3`) instead of a single rigid plate
    - `geometry.cape_physics` / `geometry.cape_physics_root_parent` (bound to the `cape` /
      `cape_root_parent` slots in player.entity.json) are kept as identical copies, so the segmented
      cape resolves no matter whether the cape pass reads the render controller's geometry expression
      or the client entity slot
    - New `animation.player.cape_physics` drives the three lower segments with velocity based trailing,
      smoothed sway, a speed scaled travelling flutter and a vertical-speed billow
    - The simulation runs in `player.entity.json`'s `pre_animation` (same pattern as
      `v.relative_cape_rotation`) so every value is integrated exactly once per frame, and all cape
      variables are initialised
    - Segment X rotations are positive: the `cape` bone is rotated 180 degrees around Y, which mirrors the
      X/Z axes of its children, so the segments have to use the opposite sign of the Java 1.7
      `animation.player.cape` to keep trailing in the same direction
    - `part_visibility` rules were added for `cape_seg_1/2/3` so the segments hide with the cape
      (elytra, spectator mode, map face icon, first person)
    - The Java 1.7 `animation.player.cape` keeps full control of the `cape` bone unchanged;
      at rest the cape hangs exactly like vanilla

### v2.5.1

- Added `No Armor Overlay + Console Edition Eat Animation` subpack
- The trident charge animation now properly plays when the player is crawling
- The crawl animation no longer transforms the player root based on head pitch
- Minor bug fixes

### v2.5.0

- Fixed most major bugs introduced since Minecraft Bedrock Edition v1.19.0, *within the capacity that resource packs allow*
    - **Minecraft Bedrock v1.21.0 to v1.21.51 are officially supported**, but future versions can't be guaranteed
- **Proper cape/persona support!** (thanks to [Rainvay_ZCYF](https://mcpedl.com/user/zouchenyunfei/) for sharing a workaround!)
    - The cape overlay pack is now deprecated for this reason
- Revamped subpack configurations
- Proper leather armor color support while armor hurt overlay is active (for applicable subpacks)
    - This means that leather armor textures are no longer overwritten
- Tweaked armor hurt overlay color values
    - `Redder Armor Overlay` subpacks now use the same RGBA values as vanilla Minecraft Bedrock Edition (`r=1.0, g=0.0, b=0.0, a=0.6`)
- Improved shield animations
    - Now uses the shield blocking flag instead of the player sneaking flag to determine whether a player should block with their shield
- Improved elytra animations and model scale
- Tweaked spyglass animations/1st person positioning
- Tweaked trident wielding animations
- Tweaked swimming animations
- Tweaked crawling animations
- Projectile particles start to emit after 1 tick (instead of 2) after spawning
- Damage particle color now more closely aligns with Minecraft Java Edition
- Damage particles now emit at the center of a player's hitbox
- Armor and elytra models now use `textures/misc/enchanted_actor_glint` instead of `textures/misc/enchanted_item_glint` for the enchantment glint texture
- The player's arms no longer bob while being rendered in menus, and instead are fixed to match the angle of that in Minecraft Bedrock Edition v1.12.1 and lower
- Improved the eat animation to match Minecraft Console Edition (for applicable subpacks)
- Partially fixed armor trims not showing
    - Currently it isn't possible to show armor trims on the hurt armor overlay due to resource pack limitations
- Banners now properly render on shields
- The vanilla fire overlay now correctly shows on mobs