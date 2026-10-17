# NOVA STRIKE

A pixel-art space shooter in the arcade tradition. You pilot a lone fighter at the bottom of the screen and fight off waves of alien ships pouring in from above. Every stage ends in a boss fight, and every stage is harder than the one before. How far can you get?

Everything (graphics, sound effects and the chiptune soundtrack) is generated live in your browser. No downloads, no installs, no plugins.

## How to start the game

You need a modern web browser (Chrome, Firefox, Edge or Safari) and a small local web server. Opening `index.html` directly by double-clicking it **won't work**, because browsers block this kind of game when it's opened from a file.

1. Open a terminal in the game folder.
2. Run:
   ```sh
   python3 -m http.server
   ```
3. Open **http://localhost:8000** in your browser.
4. Press **Enter** or tap the screen.

No Python? Any static web server works, for example `npx serve` if you have Node.js installed.

## Controls

| Action | Keyboard | Gamepad | Touch screen |
|---|---|---|---|
| Move | Arrow keys or WASD | Left stick or D-pad | Drag anywhere on the screen |
| Fire | Hold Z or Space | A or right trigger | Fires automatically while touching |
| Bomb | X | B or X | Touch with a second finger, or tap the bomb icons |
| Precise movement | Hold Shift | LB or RB | |
| Pause | P or Esc | Start | |
| Sound on/off | M | | |
| Music on/off | N | | |

**Precise movement** slows your ship down and shows its hitbox, the small dot in the middle of the ship. **Only that dot can be hit**, so bullets that brush your wings are harmless. Use it to thread through dense bullet patterns.

The game pauses itself if you switch to another window.

## Your ship

- You start with **3 spare ships**, shown at the bottom left.
- You start with **3 bombs**, shown at the bottom right. A bomb wipes every enemy bullet off the screen, damages everything, and makes you briefly invincible. Save them for emergencies.
- Losing a ship costs you one power level and one missile level. The lost power capsule falls nearby, so you can grab it back. You always come back with at least 2 bombs.
- You earn an extra ship at **50,000** points, and another every **70,000** after that.

## Power-ups

Shoot enemies to make capsules drop. Fly near one and it gets pulled toward you.

| Capsule | Effect |
|---|---|
| **P** (red) | Power up your main gun. There are 5 levels, from a single shot to a wide 7-way spread. |
| **M** (green) | Adds homing-missile pods beside your ship. The second M makes them fire faster. |
| **S** (blue) | A shield that absorbs one hit. |
| **B** (yellow) | One extra bomb (up to 6). |
| **Heart** | An extra ship. Rare: only the UFO and bosses drop these. |

Picking up a power-up you've already maxed out gives you bonus points instead.

The **UFO** that sometimes glides across the top of the screen always drops two capsules. Don't let it escape!

## Enemies

| Enemy | Points | What it does |
|---|---|---|
| Red drone | 100 | Flies down in rows and snaking lines |
| Blue swooper | 120 | Swings across the screen in a wide arc |
| Purple diver | 150 | Stops, takes aim, then rams straight at you |
| Green gunner | 300 | Parks itself and fires bursts of aimed shots |
| Orange spinner | 600 | Spins in place, spraying bullets in every direction |
| Blue gunship | 1,500 | Heavily armoured; fires spreads and twin cannons |
| Asteroid | 40–80 | Big rocks break into smaller ones |
| UFO | 2,500 | Harmless, and carries two power-ups |

## Bosses

A warning siren sounds at the end of each stage, then a boss arrives. Each boss has a health bar and **three phases**. Its attacks change and intensify as it takes damage, and the last third ("FINAL PHASE") is the most dangerous.

1. **Crimson Crab**: aimed bullet fans, rapid side cannons, bullet rings and spirals.
2. **Hive Queen**: spirals, bullet rain, swaying bullet curtains, and swarms of divers.
3. **Void Dreadnought**: huge laser beams that track you after a short warning flash, plus everything above.

After the third boss the cycle repeats with tougher versions (MK2, MK3, ...): more health, more bullets, faster patterns. The game never ends.

## Scoring

- **Chains**: kill enemies in quick succession to build a chain. Every 12 kills raises your score multiplier, up to **x8**. The chain breaks if you go about 2 seconds without a kill, or if you lose a ship.
- **Grazing**: every enemy bullet that passes very close to you without hitting is worth **20 points**.
- **Bullet cancelling**: bombs, and a boss changing phase, turn every bullet on screen into **10 points** each.
- **Boss bonus**: 10,000 for the first boss, 20,000 for the second and 30,000 for the third, doubled on the second loop and so on.
- **No-miss bonus**: clear a stage without losing a ship to earn 5,000 × the stage number.

Your high score is saved in your browser.

## Tips

- Stay low and keep moving sideways. Most shots are aimed at where you are *now*.
- When a bullet pattern looks impossible, hold Shift and look for the gaps. Your hitbox is much smaller than your ship.
- Prioritise gunners and spinners. They fill the screen with bullets the longer they live.
- When a laser's thin red warning line appears, get out of its column.
- A bomb used a moment too early beats a ship lost a moment too late.

## Trouble?

- **Black screen with a message about `file://`**: you opened the file directly. Follow "How to start the game" above.
- **No sound**: browsers only allow sound after you interact with the page, so press a key or tap once. Also check that M (sound) and N (music) aren't switched off; the title screen shows "SOUND OFF" or "MUSIC OFF" when they are.
- **The game looks blurry**: make the window bigger. The game picks the sharpest scale that fits.
