# Grand Exchange r478 backport checklist

Target: backport the complete Grand Exchange interface and exchange functionality from a clean RuneScape revision 478 cache/reference into the 2004 progressive base, while keeping the native r254 world and NPC assets.

Base branch: `Testing`

Source target: one exact revision 478 cache/client snapshot, frozen and documented before asset import begins.

## Hard scope rules

### No r478 Grand Exchange location/world backport

The r478 Grand Exchange area itself is deliberately **out of scope**.

- [ ] Do not import r478 Grand Exchange map squares, terrain or landscape data.
- [ ] Do not import GE buildings, booths, counters, signs, walls, stairs or decorative world scenery solely to recreate the r478 location.
- [ ] Do not import collision/clipping or minimap/mapscene data for the r478 GE area.
- [ ] Do not replace any existing r254 map region to make room for the GE.
- [ ] Access the Grand Exchange only through a native r254 NPC placed in the existing world.

### No r478 NPC backport

The r478 Grand Exchange clerk NPC is also deliberately **out of scope**.

- [ ] Do not import r478 GE clerk NPC definitions.
- [ ] Do not import clerk NPC models, head models, recolours, sequences, sounds or spawn definitions.
- [ ] Do not renumber or replace existing `content/pack/npc.pack` entries.
- [ ] Select a suitable NPC already present in the native r254 cache.
- [ ] Preserve that NPC's native model, head, animation and visual data.
- [ ] Add only the minimum spawn/interaction/dialogue hook required to open the GE interface.

## Definition of done

- [ ] A native r254 NPC placed in the existing world opens the Grand Exchange.
- [ ] The complete r478 GE interface family is available and visually functional.
- [ ] All required interface widgets/components are backported or compatibly recreated.
- [ ] All required GE sprites and 2D resources are backported.
- [ ] Required item model/icon rendering works inside the GE interface.
- [ ] Buy-offer item search works.
- [ ] Buy and sell offer setup works, including quantity and price controls.
- [ ] Offer slots display empty, active, partial, complete and cancelled states correctly.
- [ ] Players can submit buy and sell offers.
- [ ] Offers match correctly with partial and full fills.
- [ ] Players can cancel active offers safely.
- [ ] Players can collect purchased items, sale proceeds, refunded coins and unsold items.
- [ ] Offers persist across logout and server restart.
- [ ] Coins and items cannot be duplicated or lost through submit/match/cancel/collect races.
- [ ] Java client supports the complete GE interface flow.
- [ ] Webclient supports the same logical GE flow where applicable.
- [ ] No r478 GE map/location assets are imported.
- [ ] No r478 GE NPC assets are imported.
- [ ] Existing r254 world, maps, NPCs, inventory, bank and trade systems remain intact.

---

## Phase 0 — Freeze the r478 source and build a manifest

- [ ] Choose one exact r478 cache/client snapshot as the source of truth.
- [ ] Record its archive/source identifier and date.
- [ ] Record extraction/conversion tooling and hashes where practical.
- [ ] Enumerate the complete Grand Exchange interface family.
- [ ] Enumerate all interface/component IDs used by the GE.
- [ ] Enumerate all GE sprites, fonts, containers, varps, varbits and client-state dependencies.
- [ ] Enumerate item-model/config dependencies required specifically for interface rendering/search.
- [ ] Build a source-ID -> local-ID mapping for imported assets.
- [ ] Explicitly mark GE map/location assets as excluded.
- [ ] Explicitly mark r478 NPC assets as excluded.

## Phase 1 — Audit existing interface/client architecture

- [ ] Audit `content/pack/interface.pack` and `interface.order`.
- [ ] Audit `content/pack/inv.pack` and existing item-container handling.
- [ ] Audit existing sprite/font loading in the Java client.
- [ ] Audit equivalent sprite/interface handling in the webclient.
- [ ] Identify supported r254-era widget types and properties.
- [ ] Identify r478 GE widget features missing from the current clients.
- [ ] Identify existing component-operation packets and server trigger paths.
- [ ] Identify existing numeric/text input flows that can be reused for GE quantity, price and search.
- [ ] Prefer adapting the GE to the existing interface engine instead of replacing that engine globally.

## Phase 2 — Complete r478 GE interface manifest

- [ ] Identify the overview interface.
- [ ] Identify buy-offer setup interface/state.
- [ ] Identify sell-offer setup interface/state.
- [ ] Identify item-search/result interface/state.
- [ ] Identify active-offer detail state.
- [ ] Identify partially completed offer state.
- [ ] Identify completed offer state.
- [ ] Identify cancelled/aborted offer state.
- [ ] Identify collection interface/components.
- [ ] Identify confirm/back/abort/collect controls.
- [ ] Identify offer slot backgrounds, progress indicators and status icons.
- [ ] Identify all text fields and dynamic values.
- [ ] Identify item model/icon components.
- [ ] Identify all scroll areas and clipping requirements.
- [ ] Identify hover/pressed/disabled sprite states.

## Phase 3 — GE sprites and 2D resources

- [ ] Extract every GE-specific sprite required by the interface family.
- [ ] Import buy/sell icons.
- [ ] Import offer-slot graphics.
- [ ] Import progress/status graphics.
- [ ] Import search-result visuals.
- [ ] Import collection-state visuals.
- [ ] Preserve dimensions, offsets, transparency and alpha behaviour.
- [ ] Reuse native fonts where compatible.
- [ ] Import/convert only genuinely required font resources.
- [ ] Verify sprite rendering in the Java client.
- [ ] Verify equivalent rendering in the webclient.

## Phase 4 — Interface/widget compatibility

- [ ] Map every required r478 widget type to an existing r254-compatible representation where possible.
- [ ] Add the smallest client compatibility extensions needed for unsupported GE widget features.
- [ ] Allocate stable local interface/component IDs without colliding with existing content.
- [ ] Preserve component dimensions, positions, clipping and draw order.
- [ ] Preserve text alignment and dynamic text updates.
- [ ] Preserve sprite states and hover behaviour.
- [ ] Preserve item-model rendering inside offer/search components.
- [ ] Preserve scrolling behaviour for item search results.
- [ ] Ensure close/back operations leave the player in a valid interface state.
- [ ] Ensure opening the GE does not corrupt sidebars, inventory, chatbox or modal interfaces.

## Phase 5 — GE containers and interface item rendering

- [ ] Identify all GE-specific item containers required by offer setup and collection.
- [ ] Add required definitions to `content/pack/inv.pack` or the equivalent registry.
- [ ] Define slot counts and stack behaviour correctly.
- [ ] Ensure offer setup can represent selected item, quantity and price.
- [ ] Ensure collection slots can represent item and coin outputs safely.
- [ ] Import or map only item/model/config dependencies required for correct GE item icons/models.
- [ ] Do not import world-location models solely because they exist in the r478 GE cache region.
- [ ] Keep server container state authoritative.

## Phase 6 — Native r254 NPC access

- [ ] Pick and document the exact native r254 NPC used to access the GE.
- [ ] Place/spawn that NPC at the desired existing-world location.
- [ ] Add a clear GE interaction option or dialogue route.
- [ ] Open the GE overview through the normal server interface-open path.
- [ ] Preserve the NPC's native model, chathead, animations and unrelated interactions.
- [ ] Verify no r478 NPC cache dependency is required.
- [ ] Verify no r478 GE map or world object is required.

## Phase 7 — Interface operation routing

- [ ] Map every clickable GE component to an existing operation packet where possible.
- [ ] Add minimal packet support only where the current clients cannot express a required operation.
- [ ] Route operations through the existing server trigger/script architecture.
- [ ] Validate interface ID, component ID, slot and item ID on every economic operation.
- [ ] Reject clicks for interfaces/components the player does not currently have open.
- [ ] Keep all economic state changes server-authoritative.
- [ ] Make repeated/double-clicked economic operations idempotent where necessary.

## Phase 8 — Item search

- [ ] Implement the r478-style buy-offer search flow.
- [ ] Support partial name matching.
- [ ] Use deterministic result ordering.
- [ ] Limit results to what the interface can safely display.
- [ ] Handle zero results cleanly.
- [ ] Exclude untradeable/admin/debug objects.
- [ ] Validate the selected item server-side.
- [ ] Ensure displayed item name, model/icon and selected object ID always agree.

## Phase 9 — Quantity and price controls

- [ ] Support quantity increment/decrement controls used by the r478 interface.
- [ ] Support price increment/decrement controls used by the r478 interface.
- [ ] Support any r478 preset buttons that are part of the chosen reference.
- [ ] Reuse existing numeric input/chatbox mechanisms where practical.
- [ ] Reject zero and negative quantities.
- [ ] Reject zero and negative prices.
- [ ] Prevent integer/multiplication overflow.
- [ ] Keep displayed totals synchronized with authoritative server state.

## Phase 10 — Persistent offer model

- [ ] Define one authoritative persistent GE offer model.
- [ ] Store owner/account identifier.
- [ ] Store offer slot.
- [ ] Store buy/sell direction.
- [ ] Store object ID.
- [ ] Store requested quantity and fulfilled quantity.
- [ ] Store requested unit price and fulfilled value.
- [ ] Store collectible/refundable item and coin amounts.
- [ ] Store offer status.
- [ ] Store ordering/timestamp data required by matching rules.
- [ ] Define explicit empty, active, partial, complete, cancelled and collected states.
- [ ] Version persisted data for migration safety.

## Phase 11 — Offer slot behaviour

- [ ] Reproduce the intended number of offer slots from the r478 reference.
- [ ] Prevent creation in an occupied slot.
- [ ] Prevent two concurrent setup flows from claiming the same slot.
- [ ] Preserve complete/cancelled offers until all outputs are collected.
- [ ] Reset a slot only after collection is complete.
- [ ] Drive all slot visuals from authoritative offer state.

## Phase 12 — Buy-offer submission

- [ ] Validate selected item is GE-eligible.
- [ ] Validate quantity and unit price.
- [ ] Calculate maximum coin reservation safely.
- [ ] Atomically reserve/remove required coins when submitted.
- [ ] Do not create an offer if reservation fails.
- [ ] Persist the offer before it becomes matchable.
- [ ] Refresh the interface from authoritative state after submission.

## Phase 13 — Sell-offer submission

- [ ] Validate selected inventory item and quantity.
- [ ] Enforce tradeability and note/certificate rules intentionally.
- [ ] Atomically reserve/remove sold items when submitted.
- [ ] Do not create an offer if reservation fails.
- [ ] Preserve stack quantities exactly.
- [ ] Persist the offer before it becomes matchable.
- [ ] Refresh the interface from authoritative state after submission.

## Phase 14 — Matching engine

- [ ] Implement a central matching engine outside interface scripts.
- [ ] Match compatible buy and sell offers for the same underlying item.
- [ ] Reproduce/document the intended r478 price-time priority behaviour.
- [ ] Reproduce/document the transaction-price rule.
- [ ] Support partial fills.
- [ ] Support complete fills.
- [ ] Update both sides atomically for every match.
- [ ] Move reserved buyer coins into seller collectible proceeds.
- [ ] Move reserved seller items into buyer collectible items.
- [ ] Prevent one offer from being matched twice concurrently.
- [ ] Define and document self-match behaviour.
- [ ] Do not add modern GE tax or unrelated OSRS economy rules unless present in the frozen r478 reference.

## Phase 15 — Offer progress and live interface updates

- [ ] Update fulfilled quantity/value after every match.
- [ ] Update active/partial/complete status correctly.
- [ ] Refresh an online player's open GE interface when their offers change.
- [ ] Ensure offline owners see current state on next login/open.
- [ ] Keep progress bars and status text cosmetic representations of server state.

## Phase 16 — Cancel/abort

- [ ] Allow cancellation only for the owner's active/partial offer.
- [ ] Make cancellation atomic against concurrent matching.
- [ ] Stop future matching before calculating refundable remainder.
- [ ] Return unspent buy-offer coins exactly once.
- [ ] Return unsold sell-offer items exactly once.
- [ ] Preserve already fulfilled items/proceeds.
- [ ] Reject duplicate cancellation packets safely.

## Phase 17 — Collection

- [ ] Collect fulfilled buy items.
- [ ] Collect seller proceeds.
- [ ] Collect refunded buy-offer coins.
- [ ] Collect unsold sell-offer items.
- [ ] Support partial collection if required by inventory-space rules.
- [ ] Prevent collection duplication on repeated packets.
- [ ] Update persistent collectible amounts atomically.
- [ ] Reset the offer slot only after everything has been collected.

## Phase 18 — Persistence and restart safety

- [ ] Persist all active offers.
- [ ] Persist partial progress.
- [ ] Persist completed/cancelled offers awaiting collection.
- [ ] Persist reserved/collectible accounting accurately.
- [ ] Reload offers safely after server restart.
- [ ] Ensure restart cannot repeat a completed trade.
- [ ] Ensure restart cannot erase reserved items/coins or collectible outputs.

## Phase 19 — Concurrency and anti-duplication

- [ ] Protect submit vs submit races.
- [ ] Protect match vs cancel races.
- [ ] Protect match vs match races.
- [ ] Protect cancel vs collect races.
- [ ] Protect collect vs collect races.
- [ ] Reject spoofed interface, slot and item IDs.
- [ ] Reject stale setup state.
- [ ] Reject replayed submit/cancel/collect packets.
- [ ] Verify total item/coin conservation across matching and cancellation.

## Phase 20 — Java client validation

- [ ] GE overview renders correctly.
- [ ] Buy setup renders correctly.
- [ ] Sell setup renders correctly.
- [ ] Search results render and scroll correctly.
- [ ] Item models/icons render correctly.
- [ ] Quantity/price controls work.
- [ ] Offer progress/status displays correctly.
- [ ] Cancel/collect controls work.
- [ ] Returning/back/closing works cleanly.

## Phase 21 — Webclient validation

- [ ] GE overview renders correctly.
- [ ] Buy setup renders correctly.
- [ ] Sell setup renders correctly.
- [ ] Search results render and scroll correctly.
- [ ] Item models/icons render correctly.
- [ ] Quantity/price controls work.
- [ ] Offer progress/status displays correctly.
- [ ] Cancel/collect controls work.
- [ ] Returning/back/closing works cleanly.

## Phase 22 — Regression and scope validation

- [ ] Existing world maps remain unchanged.
- [ ] Existing world location/object definitions remain unchanged except the deliberate native NPC spawn/integration.
- [ ] Existing NPC configs/models/head models remain unchanged.
- [ ] Existing trade works normally.
- [ ] Existing inventory and bank behaviour works normally.
- [ ] Existing interfaces still open and operate normally.
- [ ] Existing item IDs remain stable.
- [ ] No r478 GE map squares are present in the diff.
- [ ] No r478 GE scenery/location definitions are present in the diff.
- [ ] No r478 GE clerk NPC config/model/head/animation assets are present in the diff.
- [ ] No unrelated r478 content is imported.

## Acceptance criteria

This draft is ready for review only when:

- [ ] The native r254 NPC opens the GE interface from the existing world.
- [ ] The complete r478 GE interface flow is present.
- [ ] Buy search/setup/submission works.
- [ ] Sell setup/submission works.
- [ ] Matching, partial fills and full fills work.
- [ ] Cancellation works without loss or duplication.
- [ ] Collection works without loss or duplication.
- [ ] Offers survive logout and restart.
- [ ] Java client passes the full interface flow.
- [ ] Webclient passes the equivalent interface flow where applicable.
- [ ] No r478 GE world/location assets were imported.
- [ ] No r478 GE NPC assets were imported.
