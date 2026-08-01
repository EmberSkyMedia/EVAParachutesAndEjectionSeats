# EVA Parachutes & Ejection Seats Fork - Inventory-Based Emergency Ejection System

## Project Goal

Fork the existing EVA Parachutes & Ejection Seats mod and refactor the parachute handling system.

The current system uses a pseudo emergency parachute that is not a real inventory item. Replace that behavior with a fully inventory-driven system using actual KSP inventory parts.

The goal is:

- Ejection systems carry actual emergency parachute inventory items (# determined by part, currently 1 seat will be 1 parachute and 3 seat part will hold 3 parachutes).
- Kerbals use existing equipped parachutes whenever possible.
- Missing parachutes are supplied from available inventory sources.
- Ejection capacity is limited by available parachutes.
- The system prioritizes which Kerbals survive when parachutes are limited.
- Inventory volume/mass restrictions are ignored when emergency swapping occurs.

Do NOT create a separate mod. Modify the existing mod architecture.

---

# New Inventory Items

The mod will support three parachute item categories.

## ejectChute

Provided by this fork as a new inventory item.
This fork has multiple Parachute options (just the chute themselves currently but I'll update with packs as well)
So the ejection part/module will need to specify which parachute it will hold by default (default is random in VAB, players can then swap out that parachute inventory as desired).
During Ejection the part will ignore all none parachute parts as things are swapped out.

Purpose:
- Emergency parachute supplied by ejection systems.

Properties:
- Normal KSP InventoryPart.
- Physical inventory item.
- Lightweight emergency chute.
- Approximately half the mass and volume of a standard chute.
- Pre-installed into ejection system inventory slots.

The mod should recognize this item by part name.

Example:

```

ejectChute

```

---

## Stock Parachute

The existing stock EVA parachute.

Recognize as:

```

evaChute

```

(or appropriate stock part name)

This is treated as a valid parachute.

---

## personalChute

Provided by other mods.

Definition:

Any inventory item containing a parachute module.

Detection should not rely on a fixed part name.

Instead:

- Check inventory item part.
- Detect if it contains a parachute-capable ModulePart/module.
- Treat it as a valid parachute source.

Examples:
- RealChute-based personal parachutes.
- Other EVA parachute inventory items.

---

# Core Ejection Logic

Replace the existing pseudo parachute assignment.

The new system must:

1. Analyze all crew.
2. Determine which Kerbals already have parachutes.
3. Determine available parachutes from inventories.
4. Generate the maximum survival ejection plan.
5. Execute ejection only for Kerbals with assigned parachutes.

---

# Parachute Priority

Crew assignment must maximize the number of Kerbals successfully ejected.
Speed is also a factor, so if a Tourist or Crew needs to go get a Parachute and there is time to eject another Kerbal who has a chute equipped, they go out of sequence, they don't wait.

Order:

## First Priority

Tourists with existing equipped parachutes.

## Second Priority

Tourists using available ejectChute inventory.

## Third Priority

Science Kerbals with equipped parachutes.

## Fourth Priority

Engineers with equipped parachutes.

## Fifth Priority

Pilots with equipped parachutes.

## Sixth Priority

Scientists receiving ejectChute.

## Seventh Priority

Engineers receiving ejectChute.

## Eighth Priority

Pilots receiving ejectChute.

## Remaining Kerbals

Remain aboard.

---

# Ethical Tourism Toggle

Add configuration option:

```

Ethical Tourism

```

Default:

OFF

Behavior:

## OFF

Tourists may use:
- Their own equipped parachute.
- Available emergency parachutes.

They cannot take parachutes away from crew.

---

## ON

If no parachutes remain:

A tourist may take a parachute from another Kerbal.

Example:

```

Tourist: no chute
Pilot: equipped chute

Result:
Tourist receives chute
Pilot remains aboard

```

This maximizes tourist survival.

---

# Inventory Sources

Search parachutes in this order:

## 1. Kerbal Inventory

Immediate.

No delay.

---

## 2. Current Capsule Inventory

Requires suiting up.

Delay:

```

0.5 seconds

```

---

## 3. Adjacent Crewable Parts

Requires retrieval.

Delay:

```

2 seconds

```

---

## 4. Other Inventories

Kerbal retrives a pack from somewhere else on the ship, this takes time.

If Connected Living Space mod is installed, Kerbal is limited to parts Connected Living Space considers the path valid and immediately adjacent parts.

Delay:

```

5 seconds

```

---

# Inventory Transfer Rules

When assigning parachutes:

Ignore:

- Inventory volume limits.
- Inventory mass limits.

Emergency equipment always overrides normal inventory mass/volume restrictions.

Don't Ignore:
- Slot Assignement
Kerbals can't equip more than they have for a slot, so if there is no room, inventory items need to be swapped out/replaced.

---

# Inventory Replacement Rules

If a Kerbal has no free inventory slot:

Replace another item.

Priority:

1. Empty slot
2. Any non-EVAJetpack item
3. EVAJetpack

The EVA Jetpack is only protected until no other option exists.

A Kerbal without a jetpack after landing is preferable to a Kerbal trapped inside an exploding spacecraft.
Swapped out part (parts if it was a stack) takes the previous position of chute, so mass/inventory is maintained.

---

# Multi-Kerbal Planning

Do NOT eject sequentially without planning.

Example:

Crew:

```

Jeb - no chute
Bill - no chute
Bob - no chute

```

Ejection System:

```

1 ejectChute

```

Result:

```

One Kerbal ejects safely.
Two remain aboard.

```

---

Example:

Crew:

```

Jeb - stock chute
Bill - stock chute
Bob - no chute

```

Ejection System:

```

1 ejectChute

```

Result:

```

Jeb ejects.
Bill ejects.
Bob receives ejectChute and ejects.

```

---

# Ejection Status Messages

After each successful ejection:

Display:

```

<Jeb> ejected safely with <Parachute Type>

```

Examples:

```

Jebediah Kerman ejected safely with Paraglider.

Bob Kerman ejected safely with Emergency Chute.

```

After all possible ejections complete:

Wait 1 second.

If anyone remains:

Display:

```

2 Kerbals left behind.

```

If everyone survives:

Display:

```

All crew successfully evacuated.

```

---

# Suggested Code Structure

Refactor into separate responsibilities.

## EjectionPlanner

Responsibilities:

- Analyze crew.
- Rank crew.
- Determine available parachutes.
- Generate evacuation plan.

Output:

```

Kerbal -> Assigned Parachute
Kerbal -> Cannot Eject

```

---

## ParachuteManager

Responsibilities:

- Detect parachute items.
- Identify stock/ejectChute/personalChute.
- Transfer inventory items.
- Replace items.

---

## InventorySearcher

Responsibilities:

Find available parachutes from:

- Kerbal inventory.
- Capsule.
- Ejection system.
- All Inventory in Parts on Ship - Restrict to CLS-connected part inventories if CLS is installed.

---

## EjectionExecutor

Responsibilities:

- Execute the generated plan.
- Handle delays.
- Fire seats.
- Display messages.

---

# Future Compatibility

Do not hardcode only stock parachutes.

Support:

- Stock EVA chute.
- ejectChute.
- Any inventory item containing parachute modules.

The ejection system should treat all valid parachute items equally once assigned.
```
