# Ironman Mode Implementation Checklist

## Goal

Implement OSRS-style account modes in the 2004 progressive server while preserving the current 2004 content model and RuneScript-driven gameplay.

Supported modes:

- `NORMAL`
- `IRONMAN`
- `HARDCORE_IRONMAN`
- `ULTIMATE_IRONMAN`
- `GROUP_IRONMAN`
- `HARDCORE_GROUP_IRONMAN`

This is an implementation tracker, not a request to modernise the game. Ironman rules must be applied to the content that actually exists in this repository. Do **not** add a Grand Exchange, modern death office, Looting Bag, raids, modern minigames, or other later systems just because OSRS Ironman has rules for them.

## Asset guardrails

The only modern assets intended to be backported for this feature are:

- Ironman / Hardcore / Ultimate / Group / Hardcore Group helmet/status icons used beside player names and in chat/player identity rendering.
- The corresponding Ironman armour sets and their required object/model/icon dependencies.

Everything else should reuse native 2004 cache content and existing interfaces where possible.

- The Tutorial Island Ironman NPC should be a new server spawn using a suitable NPC already present in the native cache.
- Group creation/management should reuse dialogue, text input, player options, and existing container/interface components rather than backporting modern Group Ironman UI.
- Group storage should reuse an existing compatible bank/container interface rather than importing the OSRS Group Storage interface.
- Do not replace the 2004 cache wholesale.

## Confirmed 2004 integration points

The implementation must integrate with the existing systems instead of creating parallel gameplay paths:

- `engine/src/engine/entity/Player.ts` for player state and appearance-facing data.
- `engine/src/engine/entity/PlayerLoading.ts` for versioned player save loading/migration.
- `content/scripts/login_logout/login.rs2` for login/Tutorial Island lifecycle and player options.
- `content/scripts/interface_trade/scripts/trade.rs2` for player trading.
- `content/scripts/player/scripts/death.rs2` for PvM/PvP death and dropped items.
- `content/scripts/shop/scripts/shop.rs2` for NPC shops and stock behaviour.
- Existing RuneScript inventory/object opcodes for ground items and containers.
- Existing duel-arena content for staking/duels.
- Existing bot players: bots must count as other players for every Ironman anti-transfer rule.

---

# Phase 0 — Freeze the rules matrix

- [ ] Write one authoritative game-mode interaction matrix before adding scattered checks.
- [ ] Define `NORMAL` as the backward-compatible default for all existing saves.
- [ ] Define solo Ironman family: `IRONMAN`, `HARDCORE_IRONMAN`, `ULTIMATE_IRONMAN`.
- [ ] Define group family: `GROUP_IRONMAN`, `HARDCORE_GROUP_IRONMAN`.
- [ ] Define which transitions are legal and which are irreversible.
- [ ] `HARDCORE_IRONMAN -> IRONMAN` on a qualifying death.
- [ ] `HARDCORE_GROUP_IRONMAN -> GROUP_IRONMAN` when the group's hardcore lives are exhausted.
- [ ] Never allow an established `NORMAL` account to promote itself into a ranked Ironman mode.
- [ ] Never let a downgrade/reset path restore lost Hardcore status.
- [ ] Do not silently introduce an unrequested `UNRANKED_GROUP_IRONMAN` mode; document any leave/kick downgrade policy explicitly.
- [ ] Freeze whether same-group `GROUP_IRONMAN` members may trade/drop/share items directly: expected **yes**.
- [ ] Freeze whether same-group `HARDCORE_GROUP_IRONMAN` members use the same economic permissions: expected **yes**.
- [ ] Freeze safe-vs-dangerous death rules against the minigames/death contexts that currently exist in this repo.
- [ ] Freeze the Group Ironman maximum size: target OSRS-style groups of 2–5 players.
- [ ] Freeze HCGIM life initialization: derive lives from the finalized starting roster and never regenerate lives through later membership changes.
- [ ] Freeze group prestige/new-member rules if ranked group prestige is implemented.
- [ ] Freeze quest-specific exceptions for content that genuinely requires another player.

# Phase 1 — Core game-mode model and persistence

- [ ] Add a typed `GameMode` enum with exactly the six requested values.
- [ ] Store the mode on `Player` as authoritative server state.
- [ ] Add centralized helpers such as `isIronman`, `isSoloIronman`, `isHardcore`, `isUltimate`, `isGroupIronman`, and `sameIronmanGroup`.
- [ ] Add a centralized `GameModeRules`/`IronmanRules` layer instead of repeating mode comparisons across scripts.
- [ ] Add rule helpers for trade eligibility, ground-item eligibility, NPC-drop eligibility, shop-stock eligibility, bank/storage eligibility, duel-stake eligibility, and assist eligibility.
- [ ] Expose the minimum script-facing game-mode/group queries required by RuneScript.
- [ ] Do not trust client-supplied mode/icon information for gameplay decisions.
- [ ] Version the player save format in `PlayerLoading` to persist game mode cleanly.
- [ ] Existing save version 7 players load as `NORMAL` without corruption.
- [ ] Persist Hardcore-loss metadata needed for auditing/hiscores if used.
- [ ] Persist group ID/reference separately from the visible mode.
- [ ] Validate unknown/corrupt persisted mode values safely.
- [ ] Add round-trip save/load tests for all six modes.
- [ ] Add migration coverage proving an old save remains playable and unchanged apart from the new default mode field.

# Phase 2 — Group Ironman persistence model

- [ ] Add persistent group records independent of an individual player's save.
- [ ] Give each group a stable ID that cannot be recycled into another live group accidentally.
- [ ] Persist group type (`GROUP_IRONMAN` or `HARDCORE_GROUP_IRONMAN`).
- [ ] Persist leader/owner.
- [ ] Persist member identities and membership state.
- [ ] Persist group creation time.
- [ ] Persist shared Hardcore lives for HCGIM.
- [ ] Persist prestige/ranked state if included in the frozen rules matrix.
- [ ] Persist membership-change history needed to enforce join/leave rules.
- [ ] Make membership and life changes atomic enough that disconnect/restart cannot duplicate membership or restore a consumed Hardcore life.
- [ ] Add audit logging for create, invite, join, leave, kick, leadership change, life loss, and group downgrade.
- [ ] Add startup validation for orphaned player->group references.
- [ ] Add tests for offline members receiving a group-wide HCGIM downgrade correctly.

# Phase 3 — Tutorial Island selection and group formation

- [ ] Spawn a new Ironman tutor/guide on Tutorial Island using an NPC already available in the native 2004 cache.
- [ ] Do not backport a modern Ironman tutor NPC model.
- [ ] Integrate with the existing `%tutorial` / `^tutorial_complete` flow.
- [ ] Let a new account choose `NORMAL`, `IRONMAN`, `HARDCORE_IRONMAN`, `ULTIMATE_IRONMAN`, `GROUP_IRONMAN`, or `HARDCORE_GROUP_IRONMAN` before completing Tutorial Island.
- [ ] Explain the major restrictions in dialogue before final confirmation.
- [ ] Require a second confirmation for permanent/restrictive modes.
- [ ] Keep the selection editable only while the account is still in the allowed pre-completion state.
- [ ] Lock the selected solo mode when Tutorial Island is completed.
- [ ] Provide group creation for players selecting a group mode.
- [ ] Provide group invitations/acceptance without importing modern GIM interfaces.
- [ ] Only allow eligible Tutorial Island/new-account players to join a ranked group.
- [ ] Prevent GIM/HCGIM group-type mixing.
- [ ] Enforce group size 2–5.
- [ ] Do not allow later invitations to regenerate Hardcore lives.
- [ ] Prevent a player from belonging to two groups.
- [ ] Handle leader disconnect during formation cleanly.
- [ ] Handle an incomplete one-player group before Tutorial completion according to the frozen rules matrix.
- [ ] Add a clear final confirmation of the starting roster for HCGIM before lives are initialized.
- [ ] Give/reclaim the correct armour set through this tutor or an appropriate existing-world tutor path.
- [ ] A normal account completing Tutorial Island must behave exactly as it does today.

# Phase 4 — Player trading and direct wealth transfer

- [ ] Gate `opplayer4` / trade initiation through the centralized rules layer.
- [ ] Re-check eligibility when the second player accepts the trade request.
- [ ] Re-check eligibility before the confirmation screen.
- [ ] Re-check eligibility immediately before final item settlement to avoid membership/mode TOCTOU exploits.
- [ ] `NORMAL <-> NORMAL` trading remains unchanged.
- [ ] Solo Ironman/HCIM/UIM cannot trade another player.
- [ ] GIM/HCGIM may trade only a member of the exact same group.
- [ ] Cross-group GIM trading is blocked.
- [ ] Normal <-> any Ironman trading is blocked.
- [ ] Bots count as players: no Ironman may use a bot as a trade mule.
- [ ] Same-group bot interaction is not a loophole unless bot group membership is explicitly supported and persisted.
- [ ] Cancel an open trade if either player's mode/group membership changes before settlement.
- [ ] Preserve existing normal-player trade behaviour and messages.
- [ ] Consider removing/hiding the generic `Trade with` option for solo Ironmen as UX only; server validation remains authoritative.

# Phase 5 — Ground items, deliberate drops, and item provenance

- [ ] Add enough provenance to ground objects to distinguish world/NPC-generated items from player-sourced items.
- [ ] Record the originating player/group where needed until the object despawns; becoming public must not erase Ironman eligibility information.
- [ ] Allow an Ironman to pick up its own deliberately dropped items where normal game rules permit.
- [ ] Allow GIM/HCGIM to pick up eligible items dropped by a member of the same group.
- [ ] Block solo Ironman/HCIM/UIM from items deliberately dropped by another player.
- [ ] Block GIM/HCGIM from items deliberately dropped by players outside their group.
- [ ] Bots count as player sources.
- [ ] Normal world spawns remain eligible.
- [ ] Scripted quest/resource spawns remain eligible unless specifically player-transferred.
- [ ] NPC drops use kill/drop eligibility rather than simply appearing as generic public ground items.
- [ ] A ground item becoming visible to everyone after its private timer must not make it valid for an ineligible Ironman.
- [ ] Wealth may leave an Ironman account by dropping/dying where the game normally allows it; restrictions primarily prevent wealth entering from outsiders.
- [ ] Prevent drop-trading through disconnect/reconnect, region reload, or ownership-timer expiry.
- [ ] Test stacked-item merging so provenance cannot be laundered by merging an ineligible stack into an eligible stack.

# Phase 6 — NPC combat contribution and drop eligibility

- [ ] Track enough damage/contribution provenance on NPCs to decide whether an Ironman earned a drop legitimately.
- [ ] A solo Ironman/HCIM/UIM loses drop eligibility when another player materially assists the kill under the chosen OSRS-style rules.
- [ ] A GIM/HCGIM may receive eligible drops when contributors are exclusively members of the same group.
- [ ] Any outside-group contributor invalidates the GIM/HCGIM drop when the rules say assistance is disallowed.
- [ ] Bots count as outside players unless they are explicitly valid members of the same group.
- [ ] Handle poison/damage-over-time attribution.
- [ ] Handle multi-combat contribution.
- [ ] Handle another player finishing an NPC after the Ironman disengages.
- [ ] Handle the Ironman finishing an NPC previously damaged by another player.
- [ ] Handle NPC regeneration/reset so stale contribution does not poison future legitimate kills forever.
- [ ] Apply the rule through shared drop infrastructure so Giant Mole, KQ, KBD, Chaos Elemental, Scorpia/custom bosses, and normal NPC drops do not each grow inconsistent one-off checks.
- [ ] Preserve existing loot tables/drop rates; this phase changes eligibility, not item tables.
- [ ] Ensure kill-credit/XP behaviour remains consistent with the existing combat system.

# Phase 7 — PvP, death drops, and anti-transfer rules

- [ ] Integrate with `content/scripts/player/scripts/death.rs2` rather than creating a separate Ironman death engine.
- [ ] Prevent an Ironman from receiving tradeable wealth from a player it kills.
- [ ] Preserve normal-player PvP death drops exactly as they work now.
- [ ] Ensure public PvP loot cannot later become eligible simply because the private timer expired.
- [ ] Decide/document narrow generated-on-death exceptions such as bones if desired; never use them to permit victim inventory/equipment transfer.
- [ ] A normal player may still receive items lost/dropped by an Ironman where existing death/drop rules allow it.
- [ ] Bots count as PvP/player deaths for Ironman anti-transfer rules.
- [ ] Prevent bot coin-drop behaviour from becoming an Ironman money source.
- [ ] Ensure Protect Item/skull/deathkeep logic still works for the dying Ironman itself.
- [ ] Preserve destroy-on-death item semantics.
- [ ] Test wilderness deaths, NPC deaths, poison deaths, and disconnect-adjacent deaths.

# Phase 8 — Hardcore Ironman death state

- [ ] Add one authoritative dangerous-death classification hook used by Hardcore modes.
- [ ] `HARDCORE_IRONMAN` immediately becomes `IRONMAN` on a qualifying dangerous death.
- [ ] Persist the loss before/with respawn handling so a disconnect or crash cannot roll Hardcore status back.
- [ ] Record killer/context/tick/time where practical for audit/hiscore purposes.
- [ ] Refresh the player's visible game-mode icon immediately after status loss.
- [ ] Update any mode-specific armour access/reclaim rules after status loss without deleting legitimately owned items unexpectedly.
- [ ] Safe deaths remain safe only where current content explicitly marks them safe.
- [ ] Audit Duel Arena, Gnome Ball, Fishing Trawler and every existing minigame/death context instead of importing a modern safe-death list blindly.
- [ ] Add tests for PvM death, PvP death, poison death, safe death, and restart immediately after death.

# Phase 9 — Hardcore Group Ironman lives

- [ ] Initialize HCGIM shared lives from the finalized starting group roster.
- [ ] Never increase lives merely because a new member joins/rejoins.
- [ ] Consume exactly one shared life for each qualifying HCGIM death.
- [ ] Persist the life loss atomically before completing respawn/status updates.
- [ ] Broadcast/update the remaining-life state to group members using existing chat/message systems.
- [ ] When lives reach zero, convert the whole group to `GROUP_IRONMAN`.
- [ ] Update online members immediately.
- [ ] Ensure offline members load as regular `GROUP_IRONMAN` next login.
- [ ] Ensure group storage and same-group trade continue working after the downgrade.
- [ ] Ensure no member can retain an HCGIM icon after the group has lost Hardcore status.
- [ ] Test simultaneous/near-simultaneous deaths so lives cannot go negative or be consumed twice incorrectly.

# Phase 10 — Ultimate Ironman storage restrictions

- [ ] Block all bank access for `ULTIMATE_IRONMAN` at the authoritative server/script boundary.
- [ ] Audit banker NPCs.
- [ ] Audit bank booths/chests.
- [ ] Audit direct interface-open helpers/opcodes that can expose bank inventory.
- [ ] Audit deposit-only or alternate bank entry points if any exist in current content.
- [ ] Audit scripts that manipulate the permanent bank inventory without opening the normal bank UI.
- [ ] Do not rely solely on hiding an interface/client option.
- [ ] Ensure a legacy/corrupt UIM save with existing bank contents cannot withdraw them through a forgotten path.
- [ ] Do not add modern UIM storage content that does not exist in the 2004 progressive world.
- [ ] Where the current game already has a storage mechanic, explicitly classify whether UIM may use it.
- [ ] UIM must not gain access to Group Ironman shared storage.
- [ ] Keep quest completion possible without requiring bank access; patch genuine quest assumptions narrowly if discovered.
- [ ] Make UIM death/recovery playable through the existing ground-item/death system without backporting the modern death-office UI.
- [ ] If implementing OSRS-style personal UIM death piles, reuse server-owned ground-object mechanics and existing UI only.

# Phase 11 — NPC shops and stock isolation

- [ ] Integrate Ironman checks with `content/scripts/shop/scripts/shop.rs2` and the existing shop inventory/restock model.
- [ ] Preserve ordinary NPC base stock/restocking for Ironmen.
- [ ] Prevent an outsider from selling an item to a shop and an Ironman immediately buying that player-supplied stock.
- [ ] Preserve provenance or otherwise distinguish natural/restocked stock from player-supplied excess stock.
- [ ] Ironman sales to shops must not become a hidden transfer path into another unrelated Ironman.
- [ ] GIM/HCGIM must not use shared shop stock as a cross-group transfer path.
- [ ] Same-group transfers should use the explicit group trade/storage systems rather than laundering items through shops.
- [ ] Preserve normal-player shop behaviour.
- [ ] Audit specialty shops with unusual currencies or stock logic.
- [ ] Test restock ticks, overstock, understock, sell-then-buy, multiple players, bots, and world restart.

# Phase 12 — Duel Arena, staking, minigames, and communal rewards

- [ ] Audit the existing Duel Arena because it is present in the current content.
- [ ] Allow no-stake duels if desired by the frozen rules matrix.
- [ ] Block Ironman stake offers and stake winnings.
- [ ] Do not allow same-group staking as an alternate transfer mechanism; same-group trading already exists explicitly.
- [ ] Re-check mode before duel settlement.
- [ ] Audit Fishing Trawler and other current multiplayer/minigame rewards for direct player-to-player wealth transfer loopholes.
- [ ] Audit party/drop-party style content if present.
- [ ] Ground-item provenance must cover communal/public drops created from player donations.
- [ ] Audit custom bosses/content currently present in progressive builds rather than assuming only strict 2004 launch content.
- [ ] Do not add rules or code for Grand Exchange, LootShare/CoinShare, raids, LMS, modern bounty systems, or other systems that are not present.

# Phase 13 — Multiplayer-required quests and narrow exceptions

- [ ] Audit existing quests that require cooperation or exchange between players.
- [ ] Explicitly verify Shield of Arrav.
- [ ] Explicitly verify Heroes' Quest if enabled in the current content set.
- [ ] Search for quest scripts that depend on ordinary trading, player drops, or another player's quest item.
- [ ] Keep the generic trade restriction intact.
- [ ] Add narrow quest-specific handoff/interact exceptions only where required to make the quest completable.
- [ ] Quest exceptions must accept only the intended quest object/state and must not transfer arbitrary wealth.
- [ ] GIM same-group help must not accidentally bypass quest state validation.
- [ ] Add regression tests for each exception.

# Phase 14 — Group storage

- [ ] Implement one persistent shared inventory per group.
- [ ] Reuse an existing 2004 bank/container interface; do not import modern Group Storage UI assets.
- [ ] Only current members of that exact group may open the storage.
- [ ] Only `GROUP_IRONMAN` / `HARDCORE_GROUP_IRONMAN` may use it.
- [ ] Prevent simultaneous writes/withdrawals from duplicating items.
- [ ] Use locking/version checks or another deterministic server-owned concurrency mechanism.
- [ ] Handle logout/disconnect while the storage is open without duping or deleting items.
- [ ] Handle world restart with storage open.
- [ ] Re-check membership on deposit and withdrawal.
- [ ] Define which untradeable/quest/degradable items may enter shared storage and enforce the policy centrally.
- [ ] Log deposits and withdrawals for anti-dupe/audit support.
- [ ] A group downgrade from HCGIM to GIM keeps the same storage.
- [ ] A kicked/left member loses access immediately.

# Phase 15 — Group membership lifecycle and ranked integrity

- [ ] Implement group create/invite/accept flow.
- [ ] Implement leader transfer if required.
- [ ] Implement leave/kick handling according to the frozen no-UGIM policy.
- [ ] Prevent membership hopping from becoming an item-transfer route.
- [ ] If new members are allowed after formation, apply the chosen prestige/ranking penalty.
- [ ] If OSRS-style new-member wealth caps are included, enforce them on both direct trade and group-storage withdrawal.
- [ ] Track any required individual/group prestige state.
- [ ] Membership changes must invalidate open trade/storage sessions immediately.
- [ ] Prevent a kicked player from retaining a stale group ID after relog.
- [ ] Prevent two leaders from being created by concurrent actions.
- [ ] Add admin-safe recovery tooling for genuinely broken group records without exposing a player promotion exploit.

# Phase 16 — Ironman armour asset backport

- [ ] Freeze one exact OSRS cache/client snapshot as source of truth for Ironman armour assets.
- [ ] Build a source-ID -> local-ID manifest before importing assets.
- [ ] Identify all armour sets required for the requested modes.
- [ ] Ironman armour set.
- [ ] Hardcore Ironman armour set.
- [ ] Ultimate Ironman armour set.
- [ ] Group Ironman armour set/variants needed by the chosen source snapshot.
- [ ] Hardcore Group Ironman armour set/variant if distinct in the chosen source snapshot.
- [ ] Import only required inventory models/icons.
- [ ] Import male worn models.
- [ ] Import female worn models.
- [ ] Preserve recolours/material properties needed for mode variants.
- [ ] Copy intended item properties/bonuses from the source snapshot instead of inventing combat stats.
- [ ] Keep the armour cosmetic/account-mode appropriate.
- [ ] Add stable object/model mappings without renumbering existing 2004 objects.
- [ ] Verify equip/remove/drop/destroy/examine behaviour.
- [ ] Define whether non-matching modes may equip reclaimed legacy pieces; default should preserve mode identity.
- [ ] Provide reclaim through the native-cache Ironman tutor without importing a modern NPC model.
- [ ] Prevent reclaim from becoming a sell/alch/wealth-generation exploit.
- [ ] Verify Java client and webclient model compatibility.

# Phase 17 — Ironman helmet/status icon asset backport

- [ ] Freeze the exact icon sprites from the chosen OSRS source snapshot.
- [ ] Backport only the required status/helmet icon assets.
- [ ] Add distinct icons for Ironman, Hardcore Ironman, Ultimate Ironman, Group Ironman, and Hardcore Group Ironman where the source provides them.
- [ ] Keep `NORMAL` icon-free.
- [ ] Keep game-mode identity separate from staff/moderator rights.
- [ ] Define staff-crown + Ironman-icon precedence/composition so an Ironman staff account does not lose moderation identity or spoof rights.
- [ ] Extend server identity/chat data with mode information in a backward-safe way.
- [ ] Render the icon beside the player's name in public chat.
- [ ] Render the correct icon in other existing chat/name surfaces that use player identity.
- [ ] Render the icon in player appearance/name presentation where supported by the clients.
- [ ] Refresh icon state immediately after HCIM/HCGIM status loss.
- [ ] Add the equivalent rendering support to the Java client and webclient paths as required.
- [ ] Do not make client-rendered icons authoritative for gameplay.

# Phase 18 — Login, social, and UX integration

- [ ] On login, load mode/group state before RuneScript can perform economy actions.
- [ ] Preserve current Tutorial Island restart behaviour for incomplete accounts.
- [ ] Show concise mode information on selection/login where useful without replacing the 2004 login flow.
- [ ] Make blocked trade/bank/shop/stake actions produce clear messages.
- [ ] GIM trade failure should distinguish "not in your group" from generic busy errors where safe.
- [ ] Hardcore loss should produce an unmistakable message.
- [ ] HCGIM life loss/group downgrade should notify affected online members.
- [ ] Do not rely on UI hiding as the security boundary.

# Phase 19 — Hiscores/site/API integration where current infrastructure supports it

- [ ] Expose game mode in the server-side data source used by hiscores/site integration, if present.
- [ ] Support mode-filtered leaderboards without changing normal hiscores semantics.
- [ ] Support Ironman, HCIM, UIM, GIM and HCGIM categories if the site/hiscore stack can represent them.
- [ ] Preserve final Hardcore death metadata/snapshot if the chosen hiscore behaviour needs it.
- [ ] Group hiscores should identify the persistent group, not a transient party.
- [ ] Any required changes in `2004sp-site` should be handled as a companion change rather than coupling site assets into this server branch.

# Phase 20 — Commands, moderation, audit, and recovery

- [ ] Add an admin inspection command/tool to display a player's mode and group ID.
- [ ] Add group inspection showing members/lives/storage metadata for debugging.
- [ ] Do not expose a normal live player command that promotes an established account to ranked Ironman.
- [ ] Testing/admin mode changes must be permission-gated and logged.
- [ ] Log blocked high-risk economy interactions where useful during rollout.
- [ ] Log group-storage deposits/withdrawals.
- [ ] Log Hardcore/HCGIM deaths and downgrades.
- [ ] Log membership changes.
- [ ] Add safe repair tooling for orphaned/corrupt group state without regenerating items/lives.

# Phase 21 — Anti-bypass test matrix

## Trading

- [ ] Normal -> Normal allowed.
- [ ] Normal -> Ironman blocked.
- [ ] Ironman -> Normal blocked.
- [ ] Ironman -> Ironman blocked.
- [ ] HCIM/UIM variants blocked identically for direct outsider trade.
- [ ] Same GIM group trade allowed.
- [ ] Different GIM groups blocked.
- [ ] Same HCGIM group trade allowed.
- [ ] Cross GIM/HCGIM groups blocked.
- [ ] Bot mule trade blocked.
- [ ] Mode/group change during open trade cancels safely.

## Ground items

- [ ] Own dropped item eligible where intended.
- [ ] Outsider dropped item blocked.
- [ ] Same-group dropped item eligible for GIM/HCGIM.
- [ ] Other-group dropped item blocked.
- [ ] Bot-dropped item blocked.
- [ ] Native world spawn eligible.
- [ ] Legitimate NPC drop eligible.
- [ ] Ineligible NPC drop stays blocked after public timer.
- [ ] Stack merging cannot wash provenance.

## NPC drops

- [ ] Solo kill eligible.
- [ ] Outside player tags first -> ineligible according to rules.
- [ ] Outside player finishes -> ineligible according to rules.
- [ ] Same-group GIM contributors -> eligible.
- [ ] Outside-group contributor -> ineligible.
- [ ] Bot contribution counts as outside contribution.
- [ ] NPC reset clears stale contribution.

## Shops

- [ ] Natural stock can be bought.
- [ ] Normal-player sold excess cannot be laundered into Ironman inventory.
- [ ] Bot-sold stock cannot be laundered into Ironman inventory.
- [ ] Other Ironman/group sales cannot create cross-account transfer.
- [ ] Normal shop behaviour remains unchanged.

## Banking/UIM

- [ ] Banker blocked.
- [ ] Booth/chest blocked.
- [ ] Direct script/interface bank open blocked.
- [ ] Direct bank inventory manipulation path blocked where player-controlled.
- [ ] Legacy bank contents cannot be withdrawn.
- [ ] Normal/other Ironman modes still bank normally.

## Hardcore

- [ ] HCIM PvM dangerous death -> Ironman.
- [ ] HCIM PvP dangerous death -> Ironman.
- [ ] HCIM safe death -> remains HCIM.
- [ ] Status persists across immediate reconnect/restart.
- [ ] HCGIM dangerous death consumes one life.
- [ ] HCGIM zero lives converts entire group.
- [ ] Offline members convert on next login.
- [ ] Concurrent deaths cannot duplicate/restore lives.

## Group storage

- [ ] Same-group deposit/withdraw works.
- [ ] Outsider cannot open.
- [ ] Kicked/left player loses access immediately.
- [ ] Simultaneous withdrawal cannot dupe.
- [ ] Disconnect cannot dupe.
- [ ] Restart cannot dupe.
- [ ] Item eligibility restrictions enforced.

## Quests/minigames

- [ ] Shield of Arrav remains completable through a narrow safe exception.
- [ ] Heroes' Quest remains completable if present.
- [ ] No quest exception transfers arbitrary items.
- [ ] Duel Arena cannot transfer stakes to/from Ironmen.
- [ ] Existing safe/death minigame classifications are tested.

# Phase 22 — Regression checks

- [ ] Existing `NORMAL` saves still load.
- [ ] Existing `NORMAL` player trade remains unchanged.
- [ ] Existing normal-player shop buying/selling remains unchanged.
- [ ] Existing normal-player PvM drops remain unchanged.
- [ ] Existing normal-player PvP/deathkeep/skull logic remains unchanged.
- [ ] Existing Duel Arena gameplay remains unchanged for normal players.
- [ ] Existing Tutorial Island progression remains unchanged for normal players.
- [ ] Existing bots continue to function for normal gameplay.
- [ ] Bots cannot bypass Ironman restrictions.
- [ ] Existing quest item flows remain unchanged except for documented Ironman exceptions.
- [ ] Existing 2004 object/model IDs remain stable.
- [ ] Java client still logs in and plays normally.
- [ ] Webclient still logs in and plays normally.
- [ ] Staff/moderator crowns still render and retain their security meaning.
- [ ] No Grand Exchange or unrelated modern content has been introduced.

# Phase 23 — Ready-for-review criteria

- [ ] All six requested modes persist correctly.
- [ ] New accounts can select their mode through Tutorial Island using native-cache NPC content.
- [ ] Trade restrictions are server-authoritative and tested.
- [ ] Ground-item provenance prevents drop trading into restricted accounts.
- [ ] NPC contribution rules prevent assisted outsider drops.
- [ ] PvP cannot transfer outsider wealth into Ironmen.
- [ ] Shops cannot be used to launder player-sold stock into Ironmen.
- [ ] UIM cannot access banking/storage paths classified as forbidden.
- [ ] HCIM loses status correctly on dangerous death.
- [ ] HCGIM shared lives and whole-group downgrade work across online/offline members.
- [ ] GIM/HCGIM same-group trading works without permitting cross-group trade.
- [ ] Group storage is persistent and dupe-safe.
- [ ] Multiplayer-required quests remain completable through narrow exceptions.
- [ ] Ironman armour sets render/equip/reclaim correctly.
- [ ] Correct helmet/status icon appears beside player names/chat without colliding with staff rights.
- [ ] Java client and webclient pass the relevant identity/asset checks.
- [ ] Normal-player behaviour remains unchanged.
- [ ] No modern systems absent from the current progressive codebase were added merely for Ironman parity.
