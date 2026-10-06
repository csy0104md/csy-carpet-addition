# CSY Carpet Addition

**English** | [中文](README.zh-CN.md)

A Carpet extension (Carpet Addition): lets magma blocks connect to nearby rails like a rail does, and lets phantoms spawn with a white horse riding them.

- Mod ID: `csy-carpet-addition`
- Source repository: <https://github.com/csy0104md/csy-carpet-addition>
- License: [MIT](LICENSE)

## Rules

| Rule | Default | Description |
| --- | --- | --- |
| `magmaBlockConnectsRails` | `false` | A magma block connects other rails like a rail does: it only tries to connect once, when it is placed; vanilla magma block behaviour is unchanged, it only gains the ability to connect rails |
| `phantomWhiteHorse` | `false` | When a phantom spawns there is a chance that a white horse created by the rule rides it: `true` = the default 20%, `false` = fully disabled, or a whole number `0`-`100` (an optional `%` is allowed) for a custom chance |

## Magma block connects rails (`magmaBlockConnectsRails`)

With the rule enabled, rails next to a magma block connect to it as if the face toward it were a rail, including:

- the four horizontal directions (same height);
- one block diagonally above and one block diagonally below (connected as an ascending / descending slope).

So `rail — magma block — rail` becomes a continuous route, and when rails surround a magma block they each connect to it, forming a junction.

The connections use the vanilla rail's own connection logic (the same checks as rail-to-rail connections), so the shapes match vanilla rails exactly.

**The connection is only attempted once, at the moment the magma block is placed**: when it is placed, the surrounding rails recompute their shapes with the vanilla logic and connect to the magma block; after that the magma block no longer takes part in rail connection checks. That means:

- placing a rail **next to an existing magma block** does not connect (the rail treats it as an ordinary block and its shape does not point at the magma block);
- if a connected rail recomputes its shape for some other reason (a redstone update, etc.), **the connection may be lost**;
- to connect again, break and re-place the magma block (as long as there are rails around it when it is placed, it will connect once more).

**A rail connected to a magma block only connects to that magma block**: a connected rail no longer connects (bends) toward other rails next to it, and it does not pull distant rails in to reconnect either. For example, when you place a magma block next to a north-south rail line, that one rail block only faces the magma block (it becomes east-west), while the rails one block north and south keep their original shape and are not bent. Rails connected to the magma block along the same axis (`rail — magma block — rail`, and the following blocks of a straight line) keep their original shape, so straight travel is unaffected; diagonal up / down slope connections are unchanged as well.

**The update order is the same as for vanilla rails**: when the magma block is placed, the order in which it refreshes the surrounding rails is the vanilla `RailPlacementHelper` connection-check order — north → south → west → east, and for each direction same height first, then one block above, then one block below (the vanilla `getNeighboringRail` order).

**Client compatibility**: this mod does not add or modify any block state (no extra magma block property, no block state id shift), so a client can play on the server without installing the mod (the connected rail shapes are computed by the server and synced).

**Breaking the magma block does not refresh the surrounding rails**: the rails keep the shape they got when they connected, until something else (a redstone signal, an update of the rail itself, etc.) makes them recompute. Placing the magma block refreshes them once so the surrounding rails connect.

Vanilla magma block behaviour is completely unchanged (walking damage and the bubble column above it are unaffected). With the rule off, the behaviour is exactly vanilla.

## Phantom white horse (`phantomWhiteHorse`)

When a phantom spawns, the rule rolls the configured chance and - if it hits - spawns a white horse riding on the phantom (the same kind of attachment as a spider jockey, so the horse moves together with the phantom).

### Rule values

| Value | Meaning |
| --- | --- |
| `false` | the rule is completely off (default) |
| `true` | the default 20% chance |
| `0` … `100` | a custom chance in percent; a `%` suffix is allowed, so `35` and `35%` are the same |

`0` disables the rule exactly like `false`, but it is **not merged into `false`**: the value stays `0`, so you can always tell whether the rule was switched off by `false` or by a custom chance of `0`. Anything that is neither `true`, `false` nor a whole number from 0 to 100 is rejected in game, with a message saying so.

### The white horse

- it is a white horse without markings and behaves like a normal horse: it **can be tamed** the vanilla way (getting on it repeatedly);
- its **speed and jump strength are the lowest a horse can have** (movement speed `0.1125`, jump strength `0.4`);
- when it dies it **always drops a milk named "白粥"** (a milk bucket with that custom name; 100%, not affected by Looting);
- it carries the entity tag `csy_phantom_white_horse` (visible with `/data get entity <horse> Tags`), which is what marks it as created by the rule.

**Vanilla horses are not affected in any way.** Only the horse this rule creates for a phantom has these properties; naturally spawned horses (and horses spawned by commands, spawn eggs, breeding, ...) keep their vanilla colours, stats, taming and drops - the vanilla horse loot table is untouched, and only horses carrying the tag above drop the milk. The rule only ever rolls when a phantom spawns, so with the rule off nothing changes at all.

No client mod is needed for this rule either: no new block, item or entity is added - the milk is a plain milk bucket with a custom name.

## Enabling the rules

In game:

```
/carpet magmaBlockConnectsRails true
/carpet phantomWhiteHorse true
/carpet phantomWhiteHorse 35
```

Or write them into the server's `config/carpet.conf`:

```
magmaBlockConnectsRails true
phantomWhiteHorse true
```

## Building

```
gradlew build
```

The artifact is `build/libs/csy-carpet-addition-<version>.jar`; drop it into the server's (or the client's) `mods` folder. Carpet must be installed as well.

## Supported versions

- Minecraft 1.21
- Fabric Loader 0.15.11 or later
- Carpet 1.4.147 (`1.21-1.4.147+v240613`)

## Implementation notes

The vanilla rail connection logic lives in `RailPlacementHelper`:

- `getNeighboringRail` / `isVerticallyNearRail` only recognise real rail blocks;
- `canConnect` decides whether a direction can connect;
- `RailBlock#updateBlockState` only recomputes the shape when "a redstone-emitting block update arrives and the number of neighbouring rails is 3".

So this mod does the following:

1. when `RailPlacementHelper#getNeighboringRail` cannot find a rail, it also returns the magma block as a "neighbouring rail" (building the helper with a powered rail state), because the magma block has no shape property and cannot be used as a rail block directly;
2. `RailPlacementHelper#isNeighbor(BlockPos)`: the magma block's helper treats any horizontally adjacent position as its neighbour, so it connects in all four directions;
3. `RailPlacementHelper#computeRailShape`: cancelled for the magma block's helper so the shape is never written back to the world (otherwise the magma block would be replaced by a rail block);
4. `AbstractRailBlock#isRail(World, BlockPos)` also treats the magma block as a rail, for the slope checks and `getNeighborCount`;
5. when a magma block is placed, the rails at the 12 positions that could connect to it are actively made to recompute their shape (vanilla rails ignore updates from blocks that do not emit redstone); the order is the vanilla rail check order: north → south → west → east, and for each direction same height → above → below;
6. items 1, 2 and 4 **only apply while step 5 runs** (a "connecting" flag in `MagmaRailUtils`, `isConnecting()`): the magma block takes part in rail connection checks only at the moment it is placed and is an ordinary block afterwards — placing a rail next to an existing magma block does not connect, and a rail recomputing its own shape no longer takes the magma block into account;
7. `RailPlacementHelper#canConnect(BlockPos)`: for a rail that is connecting to a magma block, only the direction toward the magma block counts as a connection; every other direction (even one with another rail) returns false — so it only connects to the magma block and does not bend toward rails beside it; at the same time `computeRailShape` (the vanilla cascade that pushes a connection onto adjacent rails) is skipped entirely during the "connecting" window, so the connection does not spread to the rails one block further away;
8. `onBlockAdded` only runs step 5 when the **block is really placed**: vanilla `WorldChunk.setBlockState` also calls `onBlockAdded` when a block changes its own state, so only a real placement triggers the connection (the same test as the vanilla `AbstractRailBlock#onBlockAdded`);
9. breaking the magma block does **not** run step 5, so the surrounding rails keep their shape;
10. the magma block itself is never changed as a block or in behaviour.

The phantom white horse uses three hooks:

11. `PhantomEntity#initialize` (TAIL): rolls the chance and creates the horse right there, attaching it to the phantom before the phantom is added to the world (the way a spider jockey's skeleton is spawned), so the rider is there from the first tick;
12. an `@Invoker` on the private `HorseEntity#setHorseVariant(HorseColor, HorseMarking)` forces the white colour and the "no marking" that `initialize` rolled randomly;
13. `AbstractHorseEntity#dropInventory` (TAIL): only a horse carrying the `csy_phantom_white_horse` tag gets the milk named "白粥" - the vanilla horse loot table is untouched, and a vanilla horse never carries the tag.

## License

This project is open-sourced under the [MIT](LICENSE) license. Source repository: <https://github.com/csy0104md/csy-carpet-addition>.
