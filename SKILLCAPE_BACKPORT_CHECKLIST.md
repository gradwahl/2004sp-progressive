# Skillcape backport checklist

Target: backport the original 18 October 2006-era Capes of Accomplishment assets and emotes into the 2004 progressive base without replacing the 2004 cache wholesale.

Primary source target: RuneScape revision 435 (late October 2006), using a clean matching cache/client as the reference set.

> Historical note: the original 2006 cape emote was triggered through the cape emote in the emotes interface while the cape was worn. This backport intentionally adds a repo-specific right-click **Emote** action to the skillcape so selecting it plays that cape's emote.

## Definition of done

- [ ] Every skillcape included in the chosen r435 source set has a working regular cape item.
- [ ] Every corresponding trimmed cape present in the source has a working item and uses the same emote as its untrimmed version.
- [ ] Every corresponding hood present in the source is backported if required for a complete item set.
- [ ] Cape inventory models/icons render correctly.
- [ ] Male worn cape models render correctly.
- [ ] Female worn cape models render correctly.
- [ ] Cape colours/recolours match the r435 source.
- [ ] Equip/unequip behaviour works through the normal equipment system.
- [ ] Right-clicking a supported skillcape exposes an **Emote** action in the intended context.
- [ ] Selecting **Emote** while the cape is worn starts the correct player animation.
- [ ] Any graphics/spotanims used by that cape emote play at the correct time and height.
- [ ] Any temporary models/props used by an emote render correctly.
- [ ] The emote completes cleanly and restores the normal player state.
- [ ] The implementation works in both the Java client and webclient where applicable.
- [ ] No unrelated 2004-era cache/config content is replaced or renumbered.
- [ ] All backported IDs and source mappings are documented.

## 1. Freeze the source data

- [ ] Choose one exact r435 cache snapshot and record its archive/source identifier and date.
- [ ] Use the matching r435 client/config decoding rules when extracting data.
- [ ] Do not mix object definitions from one revision with sequences/models/spotanims from another unless the difference is documented and verified.
- [ ] Build a source manifest for every cape, trimmed cape, hood, model, sequence, animation frame set, skeleton/base, spotanim and sound required.
- [ ] Record original r435 numeric IDs beside every source asset before assigning local IDs.
- [ ] Record hashes/sizes for imported binary assets so accidental source changes can be detected later.
- [ ] Keep Hunter out of the initial r435 scope unless a later revision is deliberately introduced.
- [ ] Decide whether the Quest point cape is in scope; document it separately from skill capes.

## 2. Enumerate the cape set

Create a manifest rather than assuming IDs or names.

- [ ] Attack
- [ ] Defence
- [ ] Strength
- [ ] Hitpoints
- [ ] Ranged
- [ ] Prayer
- [ ] Magic
- [ ] Cooking
- [ ] Woodcutting
- [ ] Fletching
- [ ] Fishing
- [ ] Firemaking
- [ ] Crafting
- [ ] Smithing
- [ ] Mining
- [ ] Herblore
- [ ] Agility
- [ ] Thieving
- [ ] Slayer
- [ ] Farming
- [ ] Runecrafting
- [ ] Construction
- [ ] For each skill above, identify regular cape, trimmed cape and hood objects actually present in r435.
- [ ] For each skill above, identify the exact emote sequence chain and every dependent graphic/model/frame.

## 3. Plan local ID allocation

- [ ] Inspect free/extension ranges in `content/pack/obj.pack`.
- [ ] Inspect free/extension ranges in `content/pack/model.pack`.
- [ ] Inspect free/extension ranges in `content/pack/seq.pack`.
- [ ] Inspect free/extension ranges in `content/pack/spotanim.pack`.
- [ ] Allocate stable local IDs without colliding with existing 2004/custom content.
- [ ] Add an r435-to-local-ID mapping table to this document or a dedicated manifest.
- [ ] Keep regular and trimmed variants adjacent where practical, but do not renumber existing content merely for neatness.

## 4. Backport models

- [ ] Extract every inventory model used by the cape/hood object definitions.
- [ ] Extract every male worn model.
- [ ] Extract every female worn model.
- [ ] Extract any alternate/head models needed by hoods.
- [ ] Extract temporary prop models used by cape emotes.
- [ ] Extract models referenced by cape spotanims/graphics.
- [ ] Verify the 2004 model decoder can read the r435 model format directly.
- [ ] If r435 contains unsupported model features, convert/down-port the model into the format expected by this client's model decoder rather than changing unrelated cache formats.
- [ ] Preserve vertex skins/bone assignments needed by animation.
- [ ] Preserve face colours, textures/texture references where supported, alpha and priorities needed for the original appearance.
- [ ] Update `content/pack/model.pack` with stable names for every imported model.
- [ ] Verify model IDs resolve identically in Java and web clients.

## 5. Backport animation frames and skeletons

- [ ] Identify every sequence used by each original skillcape emote.
- [ ] Trace every sequence to its frame IDs.
- [ ] Trace every frame group to its skeleton/base data.
- [ ] Extract all required frame groups and skeleton/base data from r435.
- [ ] Check whether frame/skeleton encoding changed between the 2004 client and r435.
- [ ] Add decoding compatibility only where required, or convert the imported data to the existing 2004 format.
- [ ] Confirm transformed model groups/bones still line up after conversion.
- [ ] Verify frame duration/timing values against r435.
- [ ] Verify looping/replay fields are correct.
- [ ] Verify sequence priority and movement interruption fields are correct.
- [ ] Verify main-hand/off-hand overrides used during an emote are preserved if the source sequence uses them.
- [ ] Verify the player returns to the correct idle/walk state after the sequence ends.

## 6. Backport sequence definitions

- [ ] Add named local sequence entries to `content/pack/seq.pack`.
- [ ] Preserve r435 frame ordering.
- [ ] Preserve frame delays.
- [ ] Preserve replay/loop information.
- [ ] Preserve sequence priority.
- [ ] Preserve walking/turning interruption behaviour.
- [ ] Preserve weapon/shield overrides where applicable.
- [ ] Preserve any additional sequence flags required by the clients.
- [ ] Document original r435 sequence ID -> local sequence name/ID for each cape.

## 7. Backport spotanims / graphics

- [ ] Identify every r435 spotanim used by every cape emote.
- [ ] Extract each spotanim's model dependencies.
- [ ] Extract each spotanim's animation sequence dependencies.
- [ ] Add stable entries to `content/pack/spotanim.pack`.
- [ ] Preserve resize/scale values.
- [ ] Preserve rotation/orientation values.
- [ ] Preserve recolours.
- [ ] Preserve lighting/ambient/contrast values.
- [ ] Preserve height offsets.
- [ ] Verify player-attached graphics render on the correct tile and plane.
- [ ] Verify map/world graphics are used instead when the original emote places an effect away from the player.
- [ ] Verify graphics start on the correct game tick relative to the player animation.

## 8. Backport item/object definitions

- [ ] Add all selected capes/trimmed capes/hoods to `content/pack/obj.pack`.
- [ ] Copy the original names and examine text where appropriate.
- [ ] Copy inventory model references.
- [ ] Copy male/female worn model references.
- [ ] Copy zoom, xan/yan/zan rotation and 2D offsets used by the inventory icon.
- [ ] Copy recolour mappings.
- [ ] Copy members-only state if represented by the local config format.
- [ ] Copy equipment slot/wear behaviour expected by the existing equipment system.
- [ ] Preserve tradeability/value behaviour intentionally rather than inheriting a random local object.
- [ ] Add the repo-specific **Emote** interaction without breaking Wear/Remove/Drop/Examine behaviour.
- [ ] Ensure trimmed and untrimmed variants map to the same skill emote handler.
- [ ] Ensure hoods do not accidentally expose the cape emote action.

## 9. Decide and implement the right-click UX

The requested UX is custom relative to the original 2006 activation path.

- [ ] Decide exactly where **Emote** appears: inventory cape, worn cape/equipment slot, or both.
- [ ] Prefer a worn-cape action because the emote should represent the cape currently equipped.
- [ ] Verify what context-menu operations the 2004 Java client already supports for worn equipment.
- [ ] Verify the equivalent context-menu path in the webclient.
- [ ] If worn equipment lacks an item operation packet, add the smallest compatible client/server operation needed rather than hard-coding animation playback in the client.
- [ ] Keep the server authoritative: the client should request the operation; the server decides whether the emote may run.
- [ ] If the operation maps naturally to an existing `opheldN` trigger, use the existing trigger pipeline.
- [ ] If a worn-item operation requires an interface/component trigger instead, route it through the existing script system and identify the cape by the equipped object.
- [ ] Use one consistent menu option index for all supported capes.
- [ ] Make sure **Emote** does not replace Examine, Wear or Remove.
- [ ] Confirm menu ordering in both clients.

## 10. Server-side cape emote mapping

- [ ] Create one data-driven mapping from cape object -> skillcape emote definition.
- [ ] Map both regular and trimmed cape IDs to the same emote definition.
- [ ] Store at minimum the main player sequence and any required spotanim/graphic timing data.
- [ ] Avoid 22 unrelated copy-pasted handlers when a shared dispatcher can select the correct definition.
- [ ] Add an RS2 script trigger for the chosen right-click operation.
- [ ] Reuse the existing `anim(sequence, delay)` script opcode for the player sequence.
- [ ] Reuse existing spotanim/graphic script opcodes for visual effects.
- [ ] Use queued/delayed script steps when graphics need to start on later ticks.
- [ ] Call the normal stop/action APIs before starting where appropriate.
- [ ] Keep animation and graphics timing server-driven so nearby players see the same emote.

## 11. Validation before the emote starts

- [ ] Verify the requested object is actually the supported skillcape expected by the operation.
- [ ] Verify the player is wearing the cape if worn-cape activation is required.
- [ ] Reject spoofed packets that request an emote for a cape the player does not possess/wear.
- [ ] Decide whether level 99 is revalidated at use time or whether ownership/equipping is sufficient.
- [ ] Decide behaviour while dead, teleporting, cutscened, stunned or otherwise hard-locked.
- [ ] Decide behaviour during combat.
- [ ] Decide behaviour while moving/pathing.
- [ ] Decide behaviour while another animation/skill action is active.
- [ ] Stop or suspend conflicting actions consistently with other long emotes.
- [ ] Add a short anti-spam/cooldown guard so repeated clicks cannot stack graphics or corrupt state.

## 12. Emote timing/state handling

- [ ] For every cape, record total duration in ticks.
- [ ] Record the tick at which each spotanim starts.
- [ ] Record any tick at which a temporary prop/model changes.
- [ ] Record any sound start tick if sounds are included.
- [ ] Lock or delay conflicting player actions only as long as required.
- [ ] Confirm turning/face direction behaviour matches the desired reference.
- [ ] Confirm walking does not leave the animation/graphic desynchronised.
- [ ] Confirm teleporting/logging out during an emote cannot leave stale state.
- [ ] Confirm a second emote cannot overwrite the first in an unsafe way.
- [ ] Clear temporary state after normal completion and interruption.

## 13. Sound effects (for full fidelity)

- [ ] Determine which original r435 cape emotes use sound effects.
- [ ] Extract/map the corresponding synth/sound IDs where they exist in the 2004-compatible audio system.
- [ ] Trigger sounds at the correct emote tick.
- [ ] Verify sound range/volume behaviour for nearby players.
- [ ] If a source sound format is unsupported, document the limitation rather than silently substituting a wrong sound.

## 14. Equipment and appearance integration

- [ ] Confirm capes occupy the normal cape/back equipment slot.
- [ ] Confirm appearance rebuilding includes the new worn model IDs.
- [ ] Confirm male and female appearances select the correct model.
- [ ] Test common body/leg/head appearance combinations for clipping/regressions.
- [ ] Test with weapons and shields that may be hidden/replaced by an emote sequence.
- [ ] Test trimmed cape recolours separately from untrimmed capes.
- [ ] Confirm removing the cape immediately restores the previous appearance.

## 15. Client parity

### Java client

- [ ] Confirm all new object/model/sequence/spotanim configs decode.
- [ ] Confirm the skillcape inventory icon renders.
- [ ] Confirm the worn model renders.
- [ ] Confirm right-click **Emote** appears in the intended context.
- [ ] Confirm selecting it sends the expected operation packet and does not play the animation locally by itself.
- [ ] Confirm all emote animation frames and graphics render.

### Webclient

- [ ] Confirm all new object/model/sequence/spotanim configs decode.
- [ ] Confirm the skillcape inventory icon renders.
- [ ] Confirm the worn model renders.
- [ ] Confirm right-click **Emote** appears in the intended context.
- [ ] Confirm selecting it sends the same semantic operation as the Java client.
- [ ] Confirm all emote animation frames and graphics render.

## 16. Cache/build pipeline

- [ ] Identify where raw imported model/frame/skeleton data belongs in this repo's build pipeline.
- [ ] Ensure clean checkout + normal pack/build process reproduces the modified cache.
- [ ] Do not require a developer's pre-existing local cache for the capes to work.
- [ ] Ensure generated cache indices/version tables are updated correctly.
- [ ] Ensure CRC/version changes are handled by both clients if applicable.
- [ ] Verify a fresh client downloads/loads the modified cache without stale-asset errors.
- [ ] Add source/import notes for binary data that cannot be meaningfully reviewed as text.

## 17. Acquisition / testing access

- [ ] Provide a developer/admin method to spawn every cape during implementation.
- [ ] Decide whether normal gameplay acquisition is part of this PR or a later PR.
- [ ] If acquisition is included, identify the appropriate skill masters/NPCs and level requirements.
- [ ] Do not block asset/emote testing on full NPC shop/dialogue implementation.

## 18. Per-cape test matrix

For every skill listed above, test both regular and trimmed variants where present.

- [ ] Item can be spawned/obtained.
- [ ] Inventory icon is correct.
- [ ] Examine/menu text is correct.
- [ ] Cape equips.
- [ ] Cape renders on male appearance.
- [ ] Cape renders on female appearance.
- [ ] Trim colours are correct.
- [ ] Right-click **Emote** appears.
- [ ] Clicking **Emote** reaches the server handler.
- [ ] Server selects the correct cape definition.
- [ ] Correct player sequence plays.
- [ ] Correct graphics/spotanims play.
- [ ] Correct temporary props/models appear.
- [ ] Timing matches the r435 reference closely.
- [ ] Nearby players see the same animation/graphics.
- [ ] Emote completes and player returns to idle.
- [ ] Movement/combat/action interruption does not corrupt player state.
- [ ] Repeated right-clicks do not stack/break the emote.

## 19. Regression tests

- [ ] Existing 2004 objects still render with unchanged IDs.
- [ ] Existing animations still resolve with unchanged IDs.
- [ ] Existing spotanims still resolve with unchanged IDs.
- [ ] Existing equipment operations still work.
- [ ] Existing `opheld` scripts still dispatch correctly.
- [ ] Existing emotes/animations are unaffected.
- [ ] Java client can log in and play normally with the modified cache.
- [ ] Webclient can log in and play normally with the modified cache.
- [ ] No cache index overflow/format assumptions are introduced by the new IDs.

## 20. Documentation before marking the PR ready

- [ ] Add the exact source revision/cache snapshot used.
- [ ] Add the complete source ID -> local ID manifest.
- [ ] Document any converted asset formats.
- [ ] Document any deliberate visual/timing differences from r435.
- [ ] Document the custom right-click activation difference from original 2006 behaviour.
- [ ] Add screenshots/video for at least several representative capes in both clients.
- [ ] Record which capes were manually verified.
- [ ] Record any remaining cape-specific bugs as explicit follow-ups.

## Acceptance criteria

This draft should not be marked ready until:

- [ ] All in-scope r435 skillcapes and trimmed variants render correctly.
- [ ] Every in-scope cape has its correct original-era emote assets mapped.
- [ ] With a cape equipped, the intended right-click **Emote** action is available.
- [ ] Selecting **Emote** sends a normal client operation to the server.
- [ ] The server validates the cape and starts the correct animation/graphics.
- [ ] Other players can observe the emote correctly.
- [ ] Animation, spotanim and prop timing completes without leaving the player locked or visually corrupted.
- [ ] Both Java and web clients pass the cape test matrix.
- [ ] Existing 2004 content remains intact.
