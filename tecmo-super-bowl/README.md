# Tecmo Super Bowl (NES, 1991): what's inside the cartridge

Findings from **GoodEye Episode 7**, the Tecmo Super Bowl Bo Jackson video: VIDEO_LINK_TBD

Every player in Tecmo Super Bowl has a block of ratings in the ROM. We read the game's code to find out how the speed
ratings really work, then raced the players in an emulator, frame by frame, to check.

**Confirmed** = tested in the running game. **From code** = read in the code, not tested yet.

## The short version

- **Every kicker is faster than Bo Jackson.** All 28 kickers have max speed 81; Bo has 75. Over 100 yards, running
  straight, Bo comes in last against five of them: Matt Bahr 12.98 s, Bo 13.46 s. *Confirmed.*
- **Max speed gives you two top speeds:** one with the ball, one without. With the ball, Bo tops out at about
  7.5 yards a second; chasing without the ball, the same rating gives over 11. So every defender running free is
  faster than every ball carrier in the game. *Confirmed.*
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

## The ratings (`player_ratings.csv`)

Every rating is one of 16 steps: **6, 13, 19, 25, 31, 38, 44, 50, 56, 63, 69, 75, 81, 88, 94, 100**. The game stores
the step (0 to 15), not the number; there's no real 100-point scale inside the game.

| Column | What it does | Status |
|---|---|---|
| `max_speed` | Top speed. Gives two top speeds: one with the ball and one without (below) | Confirmed |
| `running_speed` | The speed you start at: from a standstill and after every hard cut | Confirmed |
| `rushing_power` | How fast you build back up to top speed | Confirmed |
| `hitting_power` | Used when two players grapple (not traced fully yet) | From code |
| `passing_speed`, `pass_control`, `receptions`, `pass_interceptions` | Passing and catching. More with the next video | |
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

1. **Top speed with the ball** (a human-controlled ball carrier): 21 + max speed step. Bo (75, step 11): 32, about
   7.5 yards a second, 15 mph. A kicker (81, step 12): 33.
2. **Top speed without the ball** (a defender chasing, a player on kickoff coverage): 37 + max speed step. Deion
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

## The one test change in the video

The end-zone trucking (Bo running through defenders) is **not** the normal game: it's a copy of the game with Bo's
hitting power raised from 31 to 100 and the goal line and dives switched off, said on screen in the video. Every race
and chase above is the unmodified game.

## How we tested

In the Mesen emulator, with a script that sets up each run (who's on the field, where they stand, which buttons are
held on which frame) and logs every player's position and speed each frame. The game has only one ball carrier at a
time, so each runner in a race is his own run of the game, every run started on the same frame, in his own team's
uniform with his own ratings. Times are counted in frames from the first frame to the frame he crosses the goal line.

The game itself is not included here and never will be: **no ROMs, no game code, no game graphics.** Only our own
notes, measurements and charts.

Found something wrong, or something new? Open an issue.

More Tecmo Super Bowl (passing, the computer's hidden boost, injuries) comes with the next video. The other games are
on the [main page](../README.md).

---

Not affiliated with Tecmo, Koei Tecmo, the NFL or the NFLPA. Game, team and player names belong to their owners.
