# Tecmo Super Bowl (NES, 1991): what's inside the cartridge

Findings from two GoodEye videos:
- **Episode 7**, Bo Jackson and speed: https://www.youtube.com/watch?v=pY1aZiRFf2g
- **Episode 8**, passing and the computer's boost: coming November 19 to https://www.youtube.com/@GoodEye-Studios

Every player in Tecmo Super Bowl has a block of ratings in the ROM. We read the game's code to find out how the speed
ratings really work, then raced the players in an emulator, frame by frame, to check.

**Confirmed** = tested in the running game. **From code** = read in the code, not tested yet.

## The short version

- **Every kicker is faster than Bo Jackson.** All 28 kickers have max speed 81; Bo has 75. Over 100 yards, running
  straight, Bo comes in last against five of them: Matt Bahr 12.98 s, Bo 13.46 s. *Confirmed.*
- **Max speed gives you two top speeds:** one on offense, one on defense. On offense, with the ball or not (a
  receiver on his route too), Bo tops out at about 7.5 yards a second; on defense the same rating gives over 11. So
  every defender running free is faster than every ball carrier in the game. *Confirmed.* (Corrected: this said
  "with the ball / without the ball". The slow top speed is for the whole offense, not just the ball carrier.)
- **Five players have max speed 75:** Bo and four corners (Deion Sanders, Darrell Green, Rod Woodson, Wayne Haddix).
  Bo still beats Deion, Green and Woodson by 0.07-0.08 s over 100 yards, because his **rushing power** is 69 and
  theirs is 56. *Confirmed.*
- **Running speed and rushing power aren't top speed.** Running speed is how fast you are right after a hard cut;
  rushing power is how fast you get back up to full speed. *Confirmed.*
- **Cuts:** turn 45 degrees and you keep your speed. Turn 90 degrees or more and your speed resets. Bo over 100 yards:
  13.46 s straight, 28.60 s cutting 90 degrees every 16 frames. *Confirmed.*
- **When everybody zig-zags, Bo is no longer fast.** 40 yards, everyone cutting 90 degrees every 16 frames:
  Christian Okoye (running speed 63) 9.58 s, Bo (38) 11.46 s. *Confirmed.*
- **How to break away:** the cut that pays is the one where *you* turn 45 degrees and the *defender* has to turn
  90 or more. Against Deion Sanders and Lawrence Taylor: running straight, tackled; one 90-degree cut, tackled; a
  45-degree cut, then a second cut when he's right on you: touchdown. *Confirmed.*

## The short version: passing (Episode 8)

- **A covered receiver needs his QB and his hands to beat the defender by 5 steps.** Add the quarterback's pass control
  step and the receiver's receptions step, take away the defender's interceptions step: under 5, he can't catch it
  at all; at exactly 5 it's a 2% catch. *Confirmed.* (The video first says 6 and corrects it on screen.)
- **Aikman to Irvin against Bears safety Mark Carrier:** 6 + 7 = 13 vs 12, one step better. The math: 0% caught,
  40% picked off. 20 throws: 0 caught, 14 incomplete, 6 intercepted. *Confirmed.*
- **Montana to Rice against Carrier:** 12 + 12 = 24 vs 12. The math: 31% caught, 8% picked off. 20 throws: 6 caught,
  12 incomplete, 2 intercepted. Wide open, the catch is certain: 4 of 4. *Confirmed.*
- **Deion Sanders can't pick off Montana to Rice:** 24 vs 8. The math: 45% caught, 0% picked off. 20 throws: 8 caught,
  12 incomplete, 0 intercepted. **Only 15 defenders can ever pick that pass off** (interceptions 63 or more), when
  everyone is in AVERAGE condition. *Confirmed.*
- **The computer gets better the more you win.** In a season against the computer, your wins minus your losses picks
  a row of boosts. At 16-0 its defenders get +6 interceptions steps (Deion 56 plays like 94) and its quarterback and
  receivers +9. Same Montana to Rice, 20 throws: 6 caught, 12 incomplete, 2 intercepted. *Confirmed.*

## Files

| File | What it is |
|---|---|
| [`player_ratings.csv`](player_ratings.csv) | All 840 players (28 teams x 30) and every rating, with readable column names |
| [`charts/01_100_yard_times.png`](charts/01_100_yard_times.png) | Every 100-yard race time, fastest first |
| [`charts/02_chase_speeds_bo_vs_deion.png`](charts/02_chase_speeds_bo_vs_deion.png) | Bo's and Deion's speed, frame by frame, in the escape |
| [`tests/races_100_yards_straight.csv`](tests/races_100_yards_straight.csv) | The three 100-yard races: Bo vs five kickers, the max speed 75 club, kicker vs backs vs a QB |
| [`tests/bo_moves_100_yards.csv`](tests/bo_moves_100_yards.csv) | Bo over 100 yards with five different ways of moving, straight to hard zig-zag |
| [`tests/zigzag_race_40_yards.csv`](tests/zigzag_race_40_yards.csv) | Five runners, 40 yards, everyone cutting 90 degrees every 16 frames |
| [`tests/chase_bo_vs_deion_*.csv`](tests) | Bo against Deion Sanders, four runs: straight, 90-degree cut, 45-degree cut, two cuts. Every frame: both players' positions and speeds |
| [`tests/chase_bo_vs_lt_*.csv`](tests) | The same against Lawrence Taylor, three runs: straight, one 45-degree cut, two cuts |
| [`tests/pass_drills.csv`](tests/pass_drills.csv) | Episode 8's pass drills, every throw and its result: Aikman to Irvin vs Carrier, Montana to Rice vs Carrier, wide open, vs Deion, vs Deion at 16-0 |
| [`tests/pass_rule_checks.csv`](tests/pass_rule_checks.csv) | The in-game checks of the 5-step rule and of the "only 15" cutoff (the game's own roll and result) |

## The ratings (`player_ratings.csv`)

Every rating is one of 16 steps: **6, 13, 19, 25, 31, 38, 44, 50, 56, 63, 69, 75, 81, 88, 94, 100**. The game stores
the step (0 to 15), not the number; there's no real 100-point scale inside the game.

| Column | What it does | Status |
|---|---|---|
| `max_speed` | Top speed. Gives two top speeds: one on offense and one on defense (below) | Confirmed |
| `running_speed` | The speed you start at: from a standstill and after every hard cut | Confirmed |
| `rushing_power` | How fast you build back up to top speed | Confirmed |
| `hitting_power` | Used when two players grapple (not traced fully yet) | From code |
| `pass_control`, `receptions`, `pass_interceptions` | Who wins a pass when a defender is close, and whether an open receiver can drop it (below) | Confirmed |
| `passing_speed` | How fast the ball travels (not traced yet) | |
| `ball_control`, `kicking_ability`, `avoid_kick_block`, `avoid_pass_block`, `accuracy_of_passing`, `quickness` | More with the next video | |
| `team`, `roster_slot`, `position_slot`, `jersey` | The roster slot sets the record's layout, not the position: Washington's slot 3 (an RB slot) is WR Ricky Sanders | Confirmed |

Names are spelled the way the game stores them: first name in lower case, LAST NAME in capitals. Three quarterbacks
are unlicensed placeholders: **QB BILLS** (Jim Kelly), **QB EAGLES** (Randall Cunningham) and **QB BROWNS**
(Bernie Kosar).

The ratings in the file are the ones on the team screens. In a game, each player's **condition** moves every rating
by one step: BAD -1, AVERAGE 0, GOOD +1, EXCELLENT +2.

## How speed works

The field is 8 pixels to a yard; the game runs at about 60 frames a second. A player's **current speed** climbs toward
his **top speed**; every 32 speed units are 1 pixel a frame, so a speed of 32 is about 7.5 yards a second.

1. **Top speed on offense** (every offensive player, with the ball or not): 21 + max speed step. Bo (75, step 11): 32, about
   7.5 yards a second, 15 mph. A kicker (81, step 12): 33.
2. **Top speed on defense** (every defender, and the kicking team on kickoffs): 37 + max speed step. Deion
   (75): 48, over 11 yards a second. Lawrence Taylor (69): 47.
3. **Start speed:** from a standstill and after every hard cut, the current speed is set to 14 + running speed step.
   Bo (38, step 5): 19. Okoye (63, step 9): 23.
4. **Build-up:** the current speed then goes up by 1 every few frames, by rushing power: every 5 frames at 56, every
   3 at 69 or 75, every 2 at 81.
5. **Cuts:** a 45-degree change of direction (say Right to Up-Right) keeps full speed. A 90-degree or bigger change
   (Up-Right to Down-Right, Up to Down) drops the speed back to the start speed. Running diagonally, only about 71% of
   the speed goes toward the end zone.

That's why Bo beats the corners with the same max speed: he starts a little slower (19 vs 20) but builds up every
3 frames instead of every 5.

## The races

**100 yards, straight** (`tests/races_100_yards_straight.csv`, chart `01`):

| Runner | Max speed / running speed / rushing power | 100 yards |
|---|---|---|
| Matt Bahr, Scott Norwood, Morten Andersen, Nick Lowery, Gary Anderson (K) | 81 / 56 / 81 | 12.98-13.05 s |
| Bo Jackson | 75 / 38 / 69 | 13.46 s |
| Deion Sanders, Darrell Green, Rod Woodson | 75 / 44 / 56 | 13.53-13.54 s |
| Barry Sanders | 69 / 38 / 69 | 13.86 s |
| Christian Okoye | 50 / 63 / 75 | 15.27 s |
| Dan Marino | 6 / 25 / 69 | 20.35 s |

The five kickers have identical ratings, but their times differ by up to 0.07 s; we haven't traced why yet.

**Bo, 100 yards, five ways** (`tests/bo_moves_100_yards.csv`): straight 13.46 s, quick 45-degree jukes 14.01,
weaving at 45 degrees 15.77, 90-degree cuts every 32 frames 25.82, every 16 frames 28.60.

**40 yards, everyone zig-zagging** (`tests/zigzag_race_40_yards.csv`): Nick Lowery 9.52 s, Christian Okoye 9.58,
Bo Jackson 11.46, Deion Sanders 11.48, Dan Marino 12.66. Straight, Bo runs the same 40 yards in 5.47 s.

## Breaking away: Bo vs Deion Sanders and Lawrence Taylor

Bo is the kick returner on his own goal line; the defender starts 30 yards up the field, coming at him head-on, run
by the game's own chasing code. Everyone else is moved out of the way. One real run each:

| Bo does | vs Deion Sanders | vs Lawrence Taylor |
|---|---|---|
| Runs straight | Tackled | Tackled |
| One 90-degree cut | Tackled: Bo kills his own speed too | |
| One 45-degree cut | Deion has to turn around (48 -> 20), dives, misses, gets up and catches Bo from behind | LT turns around (47 -> 20), recovers and catches him |
| 45-degree cut, then a second cut when the defender is right behind him | Deion overruns twice: touchdown | LT overruns twice, dives, misses: touchdown |

The rule works on defenders too: a defender who has to turn 90 degrees or more drops to his start speed, and for a
moment he's slower than Bo. See `charts/02_chase_speeds_bo_vs_deion.png` and the `tests/chase_*.csv` files.

## How a pass is decided (Episode 8)

When the ball gets to the receiver, the game adds two steps: the **quarterback's pass control** and the **receiver's
receptions**.

- **Nobody close:** the catch is certain once those two steps add up to 18 or more (Montana to Rice: 12 + 12). Below
  that, the receiver can drop it: from about 70% caught (both at the bottom) up to 97%.
- **A defender close:** the gap is (pass control + receptions) minus the **defender's interceptions** step. The game
  rolls its dice once, using only the top half of the range (128 to 255). From the code: X = 71 + 2 x gap, and
  v = roll x X / 256. Under 51 is intercepted, under 80 incomplete, otherwise caught. Two or three close defenders
  each get their own roll, in turn.

| Steps better | Caught | Incomplete | Picked off |
|---|---|---|---|
| 0 | 0% | 56% | 44% |
| 1 | 0% | 60% | 40% |
| 4 | 0% | 70% | 30% |
| 5 | 2% | 71% | 27% |
| 6 | 7% | 70% | 23% |
| 10 | 23% | 64% | 12% |
| 12 | 31% | 61% | 8% |
| 16 | 45% | 55% | 0% |
| 24 | 65% | 35% | 0% |

At 4 steps better, even the best roll gives v = 78: never a catch. At 5, the three best rolls (253 to 255) give 80: a
2% catch. In the game we made the gap exactly 5 and 4 and made the defender roll 252, 253 and 255. Every result came
out as above (`tests/pass_rule_checks.csv`).

**Who can pick off Montana to Rice?** 12 + 12 against the defender's interceptions step. At 56 (step 8), even the
lowest roll gives v = 51: never. At 63 (step 9) it gives 50: a pick, about 2% of the time. Checked in the game the same
way. That's 15 players: Mark Carrier (81); Richard Johnson and Wayne Haddix (75); David Fulcher, Gill Byrd, Martin
Mayhew, Lemuel Stinson, Joey Browner and Dave Waymer (69); Ronnie Lippett, Erik McMillan, Rod Woodson, Gary Reasons,
Everson Walls and Donnell Woolford (63). Condition moves every rating (above): a Deion in EXCELLENT condition plays
like 69 and can do it, and the computer's boost (below) lets any corner do it.

## The computer's boost (Episode 8)

Only in a season game against the computer (one human team): the computer gets one row of boosts, picked by the
human team's **wins minus losses** (0 if you have a losing record, and in preseason). Values are steps, capped at
step 15 (100).

| Your record | Its defenders' speed | Its ball carrier's speed | Its interceptions | Its passing and catching |
|---|---|---|---|---|
| 0-0 (or worse) | 0 | +1 | 0 | 0 |
| 4-0 | +2 | +2 | +2 | +3 |
| 8-0 | +2 | +2 | +3 | +5 |
| 16-0 | +3 | +4 | +6 | +9 |

The rows were measured in the running game. Which players get each boost is from the code: the speed boost goes to
the computer's defenders at the snap and to its ball carrier when he gets the ball (even at 0-0 he gets +1); its
receivers on their routes and its blockers get none. The Deion drill at 16-0 is in `tests/pass_drills.csv`.

## Test changes

- **Episode 7:** the end-zone trucking (Bo running through defenders) is **not** the normal game: it's a copy of the
  game with Bo's hitting power raised from 31 to 100 and the goal line and dives switched off, said on screen in the
  video. Every race and chase is the unmodified game.
- **Episode 8:** in the pass drills the field is cleared down to the players being tested, and Carrier is moved up the
  depth chart so he's the one covering (no ratings changed). For the 16-0 drill the boost row and the standings record
  were set directly. For the rule checks (`pass_rule_checks.csv`) Rice's receptions or the defender's interceptions
  were changed in the emulator's memory, and the defender's roll was set to one the game can roll anyway.

## How we tested

In the Mesen emulator, with a script that sets up each run (who's on the field, where they stand, which buttons are
held on which frame) and logs every player's position and speed each frame. The game has only one ball carrier at a
time, so each runner in a race is his own run of the game, every run started on the same frame, in his own team's
uniform with his own ratings. Times are counted in frames from the first frame to the frame he crosses the goal line.

The game itself is not included here and never will be: **no ROMs, no game code, no game graphics.** Only our own
notes, measurements and charts.

Found something wrong, or something new? Open an issue.

More Tecmo Super Bowl (ratings that do nothing, the computer's play-calling, injuries) comes with the next video. The other games are
on the [main page](../README.md).

---

Not affiliated with Tecmo, Koei Tecmo, the NFL or the NFLPA. Game, team and player names belong to their owners.
