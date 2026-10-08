# CSY Carpet Addition

**English** | [中文](README.zh-CN.md)

A Carpet extension (Carpet Addition) for Minecraft 1.21 that adds two survival-flavoured rules:

- `magmaBlockConnectsRails` — magma blocks connect to the nearby rails the way a rail does;
- `phantomWhiteHorse` — phantoms can spawn with a white horse riding them.

- Mod ID: `csy-carpet-addition`
- Source repository: <https://github.com/csy0104md/csy-carpet-addition>
- License: [MIT](LICENSE)

## Rules

| Rule | Default | Description |
| --- | --- | --- |
| `magmaBlockConnectsRails` | `false` | A magma block connects other rails like a rail does: it only tries to connect once, when it is placed; vanilla magma block behaviour is unchanged, it only gains the ability to connect rails |
| `phantomWhiteHorse` | `false` | When a phantom spawns there is a chance that a white horse created by the rule rides it: `true` = the default 20%, `false` = fully disabled, or a whole number `0`-`100` (an optional `%` is allowed) for a custom chance |

## Magma block connects rails

With `magmaBlockConnectsRails` enabled, a magma block takes part in rail connection checks exactly like a rail block. Rails next to it connect to it as if the face toward it were a rail, including:

- the four horizontal directions (same height);
- one block diagonally above and one block diagonally below (an ascending / descending slope).

`rail — magma block — rail` becomes a continuous route, and rails on several sides of the magma block each connect to it, forming a junction. The shapes are produced by the vanilla rail connection logic (the same checks as rail-to-rail connections), so they match vanilla rails exactly.

### Connection rules

**The connection is only attempted once, at the moment the magma block is placed.** The surrounding rails recompute their shape with the vanilla logic and connect to the magma block; afterwards the magma block no longer takes part in rail connection checks. In practice:

- placing a rail **next to an existing magma block** does not connect — the rail treats the magma block as an ordinary block, so its shape does not point at it;
- if a connected rail recomputes its shape for another reason (a redstone update, etc.), **the connection can be lost**;
- to connect again, break and re-place the magma block (as long as there are rails around it when it is placed, it connects once more).

**A rail connected to a magma block only connects to that magma block.** It does not bend toward other rails next to it either, and it does not pull the rails one block further away in to reconnect. For example, when a magma block is placed next to a north-south line, that single rail block only faces the magma block (it becomes east-west), while the rails one block north and south keep their original shape and are not bent. Rails connected along the same axis (`rail — magma block — rail`, and the following blocks of a straight line) keep their shape, so straight travel is unaffected; the diagonal up / down slope connections are unchanged as well.

**The update order is the vanilla rail order.** When the magma block is placed, the surrounding rails are refreshed in the order the vanilla rail logic checks its connections — north → south → west → east, and for each direction same height first, then one block above, then one block below.

**Breaking the magma block does not refresh the surrounding rails.** The rails keep the shape they got when they connected, until something else (a redstone signal, an update of the rail itself, etc.) makes them recompute.

**Vanilla behaviour is unchanged.** The magma block itself is never changed: walking damage and the bubble column above it work as always, and with the rule off the behaviour is exactly vanilla.

**Client compatibility.** No block state is added or modified (no extra magma block property, no block state id shift), so a client can play on the server without installing the mod — the connected shapes are computed by the server and synced.

## Phantom white horse

When a phantom spawns, `phantomWhiteHorse` rolls the configured chance and — if it hits — creates a white horse riding on the phantom, attached the same way as a spider jockey's skeleton, so the horse moves together with the phantom.

### Rule values

| Value | Meaning |
| --- | --- |
| `false` | the rule is completely off (default) |
| `true` | the default 20% chance |
| `0` … `100` | a custom chance in percent; a `%` suffix is allowed, so `35` and `35%` are the same |

`0` disables the rule exactly like `false`, but it is **not merged into `false`**: the value stays `0`, so you can always tell whether the rule was switched off by `false` or by a custom chance of `0`. Anything that is neither `true`, `false` nor a whole number from 0 to 100 is rejected in game, with a message saying so.

### The white horse

- it is a white horse without markings and behaves like a normal horse — it **can be tamed** the vanilla way (getting on it repeatedly);
- its **speed and jump strength are the lowest a horse can have** (movement speed `0.1125`, jump strength `0.4`);
- when it dies it **always drops a milk named “白粥”** — a milk bucket with that custom name, a 100% drop that is not affected by Looting;
- it carries the entity tag `csy_phantom_white_horse` (visible with `/data get entity <horse> Tags`), which is what marks it as created by the rule.

**Vanilla horses are not affected in any way.** Only the horse the rule creates for a phantom has these properties. Naturally spawned horses — and horses spawned by commands, spawn eggs, breeding, and so on — keep their vanilla colours, stats, taming and drops; the vanilla horse loot table is untouched, and only a horse carrying the tag above drops the milk. The rule only rolls when a phantom spawns, so with the rule off nothing changes at all.

**Client compatibility.** No new block, item or entity is added — the milk is a plain milk bucket with a custom name — so no client mod is needed for this rule either.

## Requirements

- Minecraft 1.21
- Fabric Loader 0.15.11 or later
- Carpet 1.4.147 (`1.21-1.4.147+v240613`)
- The jar goes into the server's `mods` folder together with Carpet; installing it on the client is optional, since no client mod is needed for either rule.

## License

This project is open-sourced under the [MIT](LICENSE) license. Source repository: <https://github.com/csy0104md/csy-carpet-addition>.
