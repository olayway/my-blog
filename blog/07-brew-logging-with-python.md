---
title: Brew Logging with Python
description: Sip, script, repeat—because the best way to taste your progress is to track it.
authors: [julie]
date: 2025-05-19
image: /assets/hero.png
---

> [!note]  
> **Who’s this for?**  
> Anyone who juggles coffee mugs and `pip install` commands—home baristas, data-curious developers, or both.

---

## Why Track Your Brews?

Keeping a brew log turns fuzzy taste memories into searchable data. You’ll discover that the “wow” shot you pulled last Friday had **19 g in / 38 g out in 27 s at 94 °C**, not just “kinda faster than usual.” Patterns appear; skill improves.

> [!tip]  
> **First step:** Pick _one_ variable to optimize (dose, temperature, or time). Too many knobs at once will drown your palate and your dataset.

---

## Setting Up the Roaster’s Notebook

1. Create a new virtual environment.  
2. Install two lightweight libs:  

   ```bash
   pip install sqlite-utils matplotlib
   ```

	3.	Spin up an empty SQLite database called brewlog.db.

> [!warning] Avoid bloated stacks
A single-file SQLite DB beats a heavyweight cloud service when you’re half-awake at 6 a.m.

---

## Designing the Brew Schema

```
Table: brews
─────────────
id            INTEGER  (PK)
date_time     TEXT     (ISO 8601)
bean          TEXT
method        TEXT     (espresso | pourover | coldbrew …)
dose_g        REAL
yield_g       REAL
brew_time_s   INTEGER
temp_c        REAL
rating_10     INTEGER
notes         TEXT
```

> [!note]
Keep notes free-form. Flavor language is messy; your schema shouldn’t police it.

---

## A 30-Second Logging Script

```python
from datetime import datetime
import sqlite3

def log_brew(bean, method, dose, yield_, time_s, temp, rating, notes=""):
    with sqlite3.connect("brewlog.db") as db:
        db.execute(
            """
            INSERT INTO brews
            (date_time, bean, method, dose_g, yield_g, brew_time_s, temp_c, rating_10, notes)
            VALUES (?, ?, ?, ?, ?, ?, ?, ?, ?)
            """,
            (
                datetime.now().isoformat(timespec="seconds"),
                bean,
                method,
                dose,
                yield_,
                time_s,
                temp,
                rating,
                notes,
            ),
        )
    print("☕️ Logged:", bean, rating, "/10")

# Example
log_brew(
    bean="Ethiopia Guji",
    method="espresso",
    dose=18.5,
    yield_=37,
    time_s=29,
    temp=93.5,
    rating=9,
    notes="Blueberry jam, silky body",
)
```

> [!tip]
Alias the script in your shell so that logbrew is only one keystroke away from caffeine.

---

## Charting Your Caffeine Arc

```python
import sqlite3, matplotlib.pyplot as plt
from datetime import datetime

with sqlite3.connect("brewlog.db") as db:
    rows = db.execute(
        "SELECT date_time, dose_g FROM brews ORDER BY date_time"
    ).fetchall()

dates = [datetime.fromisoformat(r[0]) for r in rows]
doses = [r[1] for r in rows]

plt.plot(dates, doses, marker="o")
plt.title("Daily Dose (g) Over Time")
plt.xlabel("Date")
plt.ylabel("Dose per Brew (g)")
plt.show()
```

> [!question]
But where are the caffeine mg?
Multiply dose by bean-specific extraction ratios if you want precision, or keep grams as a simple proxy. The trend matters more than the absolute number.

## Marrying Obsidian & Python

1.	Store the script in your Obsidian vault inside a _scripts/ folder.
2.	After each brew, run logbrew from the vault’s root.
3.	Embed charts via the Obsidian Image Embed or the Dataview plugin.

```markdown
![[brew_chart.png]]
```

[!TIP]
Add a Daily Note template that queries your brews table and surfaces yesterday’s extraction alongside today’s todo list.

⸻

## Closing Sip

Logging brews is less about chasing perfection and more about tasting your learning curve. Each record is a breadcrumb back to a flavor memory, and every plotted line is a quiet affirmation that you’re dialing in, cup by cup, commit by commit.

> [!note] Next experiment
> Compare pour-over flow rates with a Bluetooth scale & integrate the data via a WebSocket. Coffee meets IoT—stay tuned!