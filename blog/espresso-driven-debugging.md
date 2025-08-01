---
title: Espresso-Driven Debugging
description: How a double-shot of dark roast—and a quick conversation with a rubber duck—helped me squash a stubborn null-pointer before sunrise.
authors: [julie]
date: 2025-04-18
image: /assets/hero.png
---

## 5:15 AM, the Null Pointer Strikes

The build had been green for weeks—until my late-night refactor introduced a cryptic cssNullPointerException that only surfaced in production logs. Re-creating it locally felt like chasing steam off a cappuccino. So I set my alarm for “pre-dawn café o’clock,” grabbed the laptop, and headed to my favorite corner espresso bar.

## Pulling the Double Shot (Setting the Stage)

A barista’s first rule applies to debugging: dial in your environment. While she weighed beans and watched the crema rise, I:
1. Checked out the exact tag from prod.
2. Rebuilt with the same JVM flags.
3. Cloned a fresh database snapshot.

> [!tip] Pro Tip
> Keep a “debugging launch.json” just like a café keeps its grinder setting card—tiny tweaks, big consistency.

## Step 1: Observe the Crema (Reproduce the Bug)

Just as good crema tells you extraction is on point, a reliable repro case tells you you’re on the right beans. I traced the bug to an edge-case API call with an empty JSON body:

```bash
curl -X POST https://api.local/devices \
    -H "Content-Type: application/json" \
    -d '{}'
```

Locally, this crashed the same way every time—perfect.

### Step 2: Smell the Beans (Gather Clues)

While that first sip hit, I skimmed:
- Stack trace depth – zeroed in on DeviceService#createDefaultConfig.
- Recent commits – a late-night “quick fix” (spoiler: it wasn’t).
- Logs – a missing deviceType field bubbled up earlier than expected.

## Step 3: Stir, Sip, Step Through (Interactive Debugging)

Espresso shot #2 arrived; I fired up the debugger:

```java
public Config createDefaultConfig(Device device) {
    Objects.requireNonNull(device);
    return defaultConfigs
             .stream()
             .filter(cfg -> cfg.supports(device.getType()))   // ← breaks here
             .findFirst()
             .orElseThrow();
}
```

Paused execution revealed device.getType() was— you guessed it—null.

## Step 4: Ristretto Refactor (Minimal Fix)

I traced device back to the controller layer—turns out my “quick fix” bypassed a mapper that enforced defaults. The smallest repair?

A test before-and-after confirmed success.

## Step 5: Second Sip, Regression Test

Nothing pairs with a bug-free build like that final swallow of crema. I locked in a unit test so future me doesn’t repeat past-me’s mistake:

```java
@Test
void emptyBodyGetsDefaultType() {
    DeviceDto dto = new DeviceDto();      // empty JSON
    Device device = mapper.mapDto(dto);
    assertNotNull(device.getType());
}
```

CI pipeline turned green. 🎉

---

## Espresso Tricks for Sharper Debugging

- Caffeine cadence: one double every 90 minutes max—spikes hurt focus.
- Rubber-duck encore: verbalizing the bug (yes, to an actual toy duck) surfaced the missing mapper step.
- Café ambiance hack: 60–65 dB background chatter boosts creativity without drowning thought.

## Café Sidebar ☕

The barista used a natural-process Ethiopian roasted 10 days ago—bright acidity keeps you alert without the bitterness that fuels coder cynicism.

---

## Closing Thoughts

Great coffee won’t fix bad code—but the ritual can sharpen your senses, slow you down, and make debugging as satisfying as that first sip. Next time a null pointer ambushes you, grab a double shot, talk to a duck, and let the aroma guide your breakpoints.

Happy coding—and happy sipping!
