# meteor-client (World Clock HUD fix fork)

A fork of [MeteorDevelopment/meteor-client](https://github.com/MeteorDevelopment/meteor-client) for **Minecraft 26.2**, maintained to carry the **World Clock HUD time fix** until it lands upstream.

Upstream base: `master` @ `45dc747` (2026-10-07) — fix branch: `fix/world-clock-time-hud` (commit `6c75f6b`).

## The bug

Since Minecraft **26.1**, day/night progression and `/time` operate on data-driven **World Clocks**, while `Level#getGameTime()` is pure **world age** — explicitly unaffected by `/time` changes.

Meteor's `Utils#getWorldTime()` formatted the world-age counter as time-of-day:

```java
int ticks = (int) (mc.level.getGameTime() % 24000);
```

So the HUD "Server time" (Starscript `{time}`) looked like a clock but ticked independently of the real day/night cycle: `/time set day|night|add` never moved it, while the sky, clock items, and daylight detectors all reacted correctly. Every Meteor user on a server saw the same wrong value.

## The fix

```java
long ticks = Math.floorMod(mc.level.getDefaultClockTime(), 24000L);
ticks = (ticks + 6000L) % 24000L;
```

- Reads the **dimension's default World Clock** — the same clock `/time` targets when no explicit clock is given.
- Dimension-aware, and reacts immediately to `/time set`, `/time add`, `/time pause`, `/time resume`, `/time rate`.
- `floorMod` normalization also fixes the old hand-rolled wrap's edge case at exactly 24000 ticks and negative clock values.
- One method, one file: `src/main/java/meteordevelopment/meteorclient/utils/Utils.java`.

## Downloads

Grab `meteor-client-26.2-local.jar` from the [releases page](../../releases). Install like any Fabric mod (`.minecraft/mods`, Fabric Loader + Fabric API for 26.2). Remove the stock Meteor jar to avoid duplicate mod IDs.

## License

Meteor Client is **GPL-3.0**; this fork inherits that license. All credit for the client itself goes to the [Meteor Development](https://github.com/MeteorDevelopment) team.
