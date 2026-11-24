# NBA Live 95 (Sega Genesis, 1994): what's inside the cartridge

Findings from the GoodEye video:
- **Episode 11**, NBA Live 95: The Octagon: coming soon to https://www.youtube.com/@GoodEye-Studios

Every player in NBA Live 95 has a block of ratings in the cartridge, and next to them, his real stats from the
1993-94 season. We read the game's code to find out how a shot is really decided, then tested it in the running game,
shot by shot, with the 1994 Knicks: Patrick Ewing, Charles Oakley and John Starks.

**Tested** = seen in the running game. **From code** = read in the code, not tested separately. **Worked out** =
calculated from tested and code numbers. Everything is the Genesis version unless it says Super Nintendo.

## The short version

- **The game can't do circles.** To know how far a shooter is from the rim, it takes the longer side (out or over)
  and adds a quarter of the shorter one. That's an octagon, not a circle: on the diagonals you count as closer than
  you are. *Tested and from code.*
- **Patrick Ewing has a 69% three-pointer.** In one spot on the diagonal, the octagon measures him inside his range,
  so his three uses his Field Goals rating (92) instead of his 3 Points rating (72): 69.1%, for three points. One
  step back: 57.4%. *Tested.*
- **The 3-point line isn't where it's painted.** The line the game scores by sits behind the painted one all the way
  around; near the corner, by more than 3 feet. *Tested (the scoreboard) and worked out.*
- **The game has a cheat switch:** Sega Assistance. On, the team down by 3 or more in the last 2 minutes of a quarter
  gets a big boost: the same Starks three goes from 57% to 73%. Off (the default), there's no hidden catch-up.
  *Tested (the boost); from code (no catch-up).*
- **Tired players dunk better.** With Fatigue on, a fresh Ewing dunks 83% of the time, an exhausted one 94%. *Tested.*
- **The ratings were built from the real 1993-94 stats**, which are stored in the cartridge: free throws almost
  exactly, field goals hardly at all. *From code and worked out.*

## Files

| File | What it is |
|---|---|
| [`csv/player_ratings.csv`](csv/player_ratings.csv) | All 359 players (27 teams, the two All-Star teams and 11 spare players with no team), every rating, shooting range, and the real 1993-94 stats stored with each player |
| [`csv/ratings_vs_real.csv`](csv/ratings_vs_real.csv) | Each rating against its real 1993-94 stat, the "fair" rating a straight line through all players gives, and how far over or under each player is |
| [`csv/best_shooters.csv`](csv/best_shooters.csv) | Every player's chance and points per shot from four spots (open, squared up, good release, Starter level), worked out with the game's formula |
| [`csv/three_point_line_painted_vs_game.csv`](csv/three_point_line_painted_vs_game.csv) | The painted 3-point line against the line the game scores by, every foot across the floor |
| [`charts/01_chance_by_distance.png`](charts/01_chance_by_distance.png) | Starks', Miller's and Ewing's chance by distance: the steps, and the jump at the end of each one's range |
| [`charts/02_octagon_vs_circle.png`](charts/02_octagon_vs_circle.png) | A real 20-foot circle and what the game calls 20 feet |
| [`charts/03_three_point_contest_by_rack.png`](charts/03_three_point_contest_by_rack.png) | Starks vs Miller: the game's chance per ball, rack by rack |
| [`charts/04_ratings_vs_real_stats.png`](charts/04_ratings_vs_real_stats.png) | Free Throws and Field Goals ratings against the real 1993-94 percentages |
| [`charts/05_dunk_fatigue_bug.png`](charts/05_dunk_fatigue_bug.png) | Ewing's dunk chance, fresh to exhausted |
| [`tests/ewing_corner_three.csv`](tests/ewing_corner_three.csv) | Ewing, 5 shots from his secret spot and 5 from one step back |
| [`tests/starks_step_back.csv`](tests/starks_step_back.csv) | Starks, 5 shots inside his range and 5 from one step back |
| [`tests/painted_line_two_or_three.csv`](tests/painted_line_two_or_three.csv) | Starks behind the painted line (2 points) and behind the game's line (3 points) |
| [`tests/release_timing_maxwell.csv`](tests/release_timing_maxwell.csv) | The same Vernon Maxwell jumper with the shoot button held 6 to 44 frames |
| [`tests/three_point_contest_starks_vs_miller.csv`](tests/three_point_contest_starks_vs_miller.csv) | The 3-point contest, all 50 balls |
| [`tests/starks_four_spots.csv`](tests/starks_four_spots.csv) and [`tests/starks_four_spots_with_miller_ratings.csv`](tests/starks_four_spots_with_miller_ratings.csv) | The same four Starks shots, with his own ratings and with Reggie Miller's |
| [`tests/sega_assistance_off_vs_on.csv`](tests/sega_assistance_off_vs_on.csv) | Three pairs of the same Starks shot, Sega Assistance off and on |
| [`tests/sega_assistance_follows_the_score.csv`](tests/sega_assistance_follows_the_score.csv) | Who gets the boost as the score changes in the last 2 minutes |
| [`tests/sega_assistance_on_houston_down_10.csv`](tests/sega_assistance_on_houston_down_10.csv) | 2 minutes of computer vs computer with Sega Assistance on and Houston down 10: every shot and dunk |
| [`tests/dunk_fatigue_bug_ewing.csv`](tests/dunk_fatigue_bug_ewing.csv) | The same Ewing dunk at five energy levels |
| [`tests/every_dunk_starks.csv`](tests/every_dunk_starks.csv) | Starks does all six dunks |
| [`tests/finals_rematch_first_3_minutes.csv`](tests/finals_rematch_first_3_minutes.csv) | The first 3 minutes of Knicks at Rockets, computer vs computer: every shot, its chance and the roll |
| [`tests/finals_rematch_20_games.csv`](tests/finals_rematch_20_games.csv) | Knicks at Rockets, 20 full computer-vs-computer games: scores and points per player |

In the CSVs, **game distance** is the distance the game thinks a shot is from (the octagon distance, below), in feet.
A shooting range is in the same game feet. Names are spelled the way the game stores them (it adds an initial to
tell players apart: "D. Harper" is Derek Harper, "R. Miller" is Reggie Miller). In the ratings file, `stamina_shown`
is the Stamina on the player screen and `hidden_real_stamina` is the one the game really uses (see "Other findings").

## Round 1: the octagon

When you shoot, the game has to know how far you are from the rim. In real life every spot 20 feet away sits on a
circle. NBA Live 95 doesn't work it out that way: **it takes the longer side, out from the basket or over to the side,
and adds a quarter of the shorter one.** *From code, and tested: the game's own distance matched it at 12 spots.*

- Straight out from the basket, or along the baseline, that's exact.
- On the diagonal it comes out short. Stand 16 feet out and 16 feet over: you're really 22.6 feet away; the game says
  20. At 45 degrees every distance reads 12% short. *Worked out.*
- The rings where a shot gets harder (one every 4 feet) are octagons too, so **a diagonal shot gets a better chance
  than a straight-on shot from the same real distance.** *From code and tested.*
- The Super Nintendo version uses the same shortcut. *From code.*

Why the shortcut? Probably speed: a square root is slow on a 1994 console, and this is close enough. (That's our
guess, not something in the code.) See `charts/02_octagon_vs_circle.png`.

The court is to scale: a real NBA court at 8 game units a foot, so the feet here are real feet. *Worked out from the
painted court lines.*

## Range: Field Goals or 3 Points

Every player has a **shooting range** (from 9.5 feet for Dennis Rodman to 27 feet for the best shooters). Inside it,
his jump shot uses his **Field Goals** rating; past it, his **3 Points** rating. *Tested (a player shooting from past
his range used his 3 Points rating).*

That's a separate question from whether a basket counts 2 or 3: that comes only from where he shot from and the
3-point line. So a long two can use a player's 3 Points rating, and a three can use his Field Goals rating.

**Starks gets better by stepping back.** His Field Goals is 78, his 3 Points 86, his range 26.1 game feet. From
25.4 feet, inside his range: 56.5%. One step back, 26.5 feet, past his range: 60.8%. Both are twos, both in the same
distance band; only the rating changed. Five shots from each spot: 2 went in, then 4. *Tested*
(`tests/starks_step_back.csv`).

## Round 2: Ewing's corner three

Ewing's range is 21.4 game feet: it ends just short of the 3-point line, everywhere except on one diagonal. There the
octagon measures him closer than he really is.

- In the whole half court there are **exactly five floor spots** that are behind the 3-point line and still inside
  Ewing's range. *Worked out from the formula; one of them tested.*
- The spot we tested: 17 feet out and 17 feet over. Really 24.1 feet from the rim; the game says 21.4, right on his
  range. His three uses **Field Goals 92: 69.1%**. That's 2.07 points a shot, his best spot on the floor. *Tested.*
- **One step back** (18 out, 18 over, really 25.5 feet; the game says 22.5): past his range, **3 Points 72: 57.4%**.
  *Tested.*
- All 10 shots went in. Two of them (one from each spot) were dice misses that dropped anyway: a bad roll doesn't
  force a miss, it knocks the ball off line, and sometimes it still finds the net. *Tested*
  (`tests/ewing_corner_three.csv`).
- The spot is too thin to stop on by hand: in the test his feet were held on it.

## The 3-point line vs the painted one

- Anything more than about 22 feet to the side of the rim (the corners) is a three. Elsewhere the game keeps its own
  line in a table. *From code, tested with the scoreboard.*
- **The line the game scores by is behind the painted one all the way around:** about half a foot to a foot on the
  far side of the floor and at the top, about 1.5 feet on the near wing, and more than 3 feet near the near corner.
  *Worked out (the painted line against the game's table,* `csv/three_point_line_painted_vs_game.csv`*).* The painted
  arc itself is lopsided, and the game's line follows it.
- **Starks on the near wing, 2 feet behind the painted line: 2 points** (twice). About 2 feet further back, behind the
  game's line: 3 points. *Tested with the scoreboard* (`tests/painted_line_two_or_three.csv`).
- Because of the octagon, the easiest threes are the corners and the wing on the short side of the arc. The same
  shooter has 8-16% more chance there than at the top of the key or on the long wing. *Worked out.*

## Round 3: how a shot is decided

The whole jump shot comes down to: **chance = how close you are x your rating.** The game predicted every shot we
logged with this. *Tested.*

**1. How close.** Every 4 feet out (game feet: the octagon distance) the shot takes a bigger bite:

| Game distance | Chance for a shooter value of 100 | Starks, open and squared up |
|---|---|---|
| 0-8 ft | 85% | 89% |
| 8-12 ft | 78% | 81% |
| 12-16 ft | 70% | 73% |
| 16-20 ft | 62% | 65% |
| 20-24 ft | 59% | 61% |
| 24-28 ft | 54% | 56.5% inside his range, 60.8% past it |
| 28-32 ft | 51% | 57% |
| 32-36 ft | 39% | 44% |
| 36-40 ft | 31% | 35% |

Beyond 40 feet it keeps dropping, to about 7% for a shooter value of 100 from 48 feet and out. *Tested.*

**2. The shooter value** = the rating (Field Goals or 3 Points, by range) plus or minus:
- +16 (with Fatigue off; with Fatigue on it shrinks as the player tires). *Tested.*
- +10 squared up to the basket; less, or minus, when he isn't (not facing it, moving). *Values tested; which body
  position gets which, from code.*
- -10 if one defender jumps at the shot, -15 if two or more. *Values tested.*
- Level: on Rookie, your players +40 and the computer's -10; on All-Star, the computer's +40 and yours -10; Starter,
  nothing. *Tested.*
- **Hot hand:** a player who scored his team's last 3 baskets (before the other team scores) gets x1.25: the same
  Starks shot went from 60.8% to 76.1%. *Tested.*

So an open, squared-up Starks shoots with 78 + 16 + 10 = 104 inside his range and 86 + 26 = 112 past it. See
`charts/01_chance_by_distance.png`.

**3. Release timing.** You shoot by holding the button and letting go. Let go near the top of the jump and you keep
the full value. Too quick or too slow, and **the value is cut in half.** The same Maxwell jumper (`tests/release_timing_maxwell.csv`):
held 18, 22 or 26 frames (about a third of a second to just under half a second): 69.1%. Held 6, 12, 30 or 36
frames: 34.2%. Held 44: he lands with the ball and never shoots. *Tested.* The window moves a little from shot to
shot (a random amount, and a higher Field Goals rating makes it wider). *From code.*

**4. The dice.** The game rolls, and if the roll lands inside the chance, the ball flies straight in. If it doesn't,
the ball gets a random error added to its flight: usually a miss, but it can still roll in. *Tested.* The roll is
made when the ball leaves the shooter's hands, so before it's anywhere near the rim the game already knows. *From
code.* Shots right at the basket (layups, and dunks, which have their own roll) don't use these dice.

On All-Star level, and only there, a three gets a slightly worse distance number than a two from the same spot. On
Rookie and Starter the chance never depends on whether the shot is worth 2 or 3. *Tested.*

## Round 4: Starks vs Miller, a 3-point contest

The game has no 3-point contest mode (*from code*), so we staged one: a Knicks at Pacers game, Starks at one end,
Miller at the other, **real teams and real ratings**. Five racks of five balls from the same spots for both, a good
release on every ball, 1 point a ball and 2 for the last ball of each rack (the money ball).

- **Miller 20, Starks 16** (18 makes to 14). Two of Miller's makes were dice misses that rolled in. *Tested.*
- The game rates Miller's shot better from every rack: rack 1 69.7% vs 60.9%, rack 2 64.6% vs 56.5%, rack 3 64.1% vs
  60.8%, racks 4 and 5 59.8% vs 56.7%. *Tested* (`charts/03_three_point_contest_by_rack.png`).
- **The swap:** the same four Starks shots from the same moment, with the same dice, then again with Reggie Miller's
  ratings put in Starks' place. Every one got a better chance: 55.1% to 63.9%, 55.1% to 63.9%, 51.7% to 54.7%, 41.5%
  to 44.6%. *Tested* (`tests/starks_four_spots*.csv`; a defender jumped at every shot, so these are lower than open
  shots).
- In the swap, one shot rolled out of the rim for Starks and dropped for "Miller", with the same dice and the same
  ball flight up to the rim. The rim roll itself is random: Miller's ratings changed how the computer players moved,
  and that changed the random numbers. *Tested.*

Miller: Field Goals 93, 3 Points 92, range 27.2 game feet. Starks: 78, 86, 26.1.

## Round 5: where the ratings came from

The cartridge stores every player's **real 1993-94 stats** next to his ratings: games, minutes, field goals, threes,
free throws, rebounds, assists, steals, turnovers, blocks and points. *From code (read from the cartridge).* The game
doesn't use them in play; they're for the stats screens and simulated season games. Comparing them with the ratings
(*worked out*, `csv/ratings_vs_real.csv`, `charts/04_ratings_vs_real_stats.png`):

- **Free Throws follow the real free-throw percentage almost exactly** (correlation 0.98 over 230 players). Ewing 77,
  real 76.5%. Oakley 78, real 77.6%. Starks 75, real 75.4%. The one big miss: Chris Mullin, 91 with 75.3% that season.
- **Field Goals barely follow the real field-goal percentage** (correlation 0.20). It's star power: Derek Harper is
  84 and really shot 41%. Mark Price, Karl Malone, David Robinson and Hakeem Olajuwon are 95 and up. Because the jump
  shot uses this rating, the stars really do shoot better in the game than they did.
- **Charles Oakley: Blocks 85.** His real blocks that season: 18 in 82 games (0.22 per 36 minutes), the most
  overrated shot-blocker in the game. **Anthony Mason: Blocks 79**, with 9 in 73 games.
- **Doc Rivers** made 36.5% of his threes and got 3 Points 70: the most underrated shooter on the Knicks.
- Steals, blocks, rebounds and passing follow the real stats fairly well (correlation 0.65 to 0.79).

## Round 6: Sega Assistance, the cheat switch

**Sega Assistance** is a real option (Game Setup, Set Options), **off by default**. *Tested.*

- **On:** in the last 2 minutes of a quarter, the team **down by 3 or more** gets +32 on its shooter value. The same
  Starks three, from the same moment with the same dice: 56.7% with it off (miss), 73.0% with it on (in). A long two:
  56.5% to 73.9%. *Tested* (`tests/sega_assistance_off_vs_on.csv`).
- In a computer-vs-computer game with Houston down 10 and under 2 minutes left: Olajuwon shot at 96%, and a Maxwell
  dunk couldn't miss. *Tested* (`tests/sega_assistance_on_houston_down_10.csv`).
- **It follows the score live.** Inside the last 2 minutes the boost goes off as soon as the gap is under 3, comes
  back at 3, and moves to the other team if they fall behind. Before the last 2 minutes it's off. *Tested*
  (`tests/sega_assistance_follows_the_score.csv`). It applies in every quarter. *From code.*
- Also with it on: the trailing team's dunks get +35 (*tested*); the leading team's dunk chance is halved and only one
  of its defenders can contest a shot (*from code*).

**With it off, there's no hidden catch-up.** We checked every place the game looks at the score during play. *From
code.* There are only two others:
- Late in the 4th quarter (or overtime) with 2 minutes or less left, a computer team down 4 or more goes for the ball
  whatever its team fouls. Down 10 with 1:30 left, the trailing team lunged at the ball; tied, or in the 2nd quarter,
  or with 5:00 left, it didn't. Shot, steal and block chances don't change. That's good basketball, not a cheat.
  *Tested.*
- A "blowout" check in the computer's substitutions that works out whether it's down big, then does nothing either
  way. Switched off. *From code.*

## Round 7: the Super Nintendo version

The Super Nintendo version has its own lead programmer, and its shot code is a rewrite. From each game's own
credits list: the Genesis lead programmer is Amory 'Iceman' Wong (programming Mike Kiernan and Brian Krause; additional
programming Allan Johanson, Daniel Ng and Len Paul). The Super Nintendo lead programmer is Allan Johanson (programming
Rod Reddekopp, Daniel Ng and Dan Scott; additional programming Amory Wong). So the two teams overlap, with the roles
swapped. *Read from both games' credits.* **Same octagon, same 3-point line, different shot formula.** *From code.*

- On the Super Nintendo the rating only puts the shooter in **one of four groups** (85 and up, 75 and up, 66 and up,
  below), and each distance band takes a fixed amount off. **A shooter with 85 shoots exactly like one with 99.**
  *From code; the computer's shots checked in the game (7 shots), a human player's too (4 shots).*
- An open three by a human player on Starter, from a 3 Points 85+ shooter like Starks: **about 78-82%** just past the
  line on the Super Nintendo, up to 89% at a few diagonal spots. On the Genesis: 61% at best. *Worked out from the
  tested parts.*
- **No Ewing secret three there:** on the Super Nintendo a three always uses the 3 Points rating. *From code.*

## Final round: at the rim

- **Layups and dunks:** shoot on the move within 12 game feet of the basket and you go up for a layup or a dunk
  instead of a jump shot. *Tested.* That has its own roll, set by the Dunking rating: Dunking 50 is 78%,
  Dunking 99 is 97%, Ewing (94) 95%; every Dunking point is worth about 0.4%. *Tested.*
- **The fatigue bug: tired players dunk better.** With Fatigue on, jump shots and passes get worse as a player tires,
  but the dunk does the opposite: the game takes off a quarter of the player's energy, so a fresh player loses the
  most. Same Ewing dunk, same moment, only his energy changed: fresh 83.2%, then 85.9%, 89.1%, exhausted 93.8%. With
  Fatigue off: 95.3%. *Tested* (`tests/dunk_fatigue_bug_ewing.csv`, `charts/05_dunk_fatigue_bug.png`).
- **Six dunks, picked at random:** generic slam, one-hand slam, double pump, double pump 2, backward slam and windmill.
  Nobody has a signature dunk; the generic slam comes up twice as often as the others. *From code.* All six done by
  Starks: `tests/every_dunk_starks.csv`. *Tested (each one forced in turn).*
- **The Dunking rating decides dunk or layup:** under 76 always a layup, 76-84 a dunk half the time, 85-91 three
  times in four, 92 and up always a dunk. *From code.*
- The game has no dunk contest mode either. *From code.* A dunk contest in the video (Dominique Wilkins vs Isaiah
  Rider) is staged the same way: both players forced through the same six dunks.

## How the random numbers work

The dice come from **two 16-bit shift-register generators**: two small number scramblers that each turn their last
number into the next one. *From code.*

- The game sets them to the **same fixed starting values** every time. *From code.*
- **They only move on when the game asks for a roll**, not on a clock. Waiting a different number of frames at the
  intro screen before tip-off gave two games that were identical to the frame. *Tested.*
- So the "random" numbers repeat: load the same saved moment, do the same thing, and you get the same dice. *Tested.*
  Anything that changes which rolls the game asks for changes everything after it: turning the Fatigue option on
  gave the same dunk a different roll, and Miller's ratings moved the computer players differently and changed a rim
  roll. *Tested.*

That's what made the tests possible: the same shot, from the same moment, with the same dice, and only one thing
changed.

## Other findings

- **Blocks:** each frame a defender touches a shot in the air, he can block it, with a chance that goes up with his
  Blocks rating (Ewing's 92: about 18% a frame). **On Rookie and Starter, a computer-controlled team blocks at half its rating**;
  human teams, and everyone on All-Star, use the full rating. *Tested.* Poor shot-blockers jump at the shot early;
  blockers rated 85 or more never do. *From code.*
- **Steals:** each frame a defender is close (and not behind the ball handler) he can steal it, about 4% a frame for
  a Stealing rating of 87. No level or human/computer difference. *Tested.* If he wins it, his Passing rating decides
  whether he catches it or just knocks it loose. *From code.*
- **Free throws:** the computer's free throws follow the rating: 50 = 50%, 70 = 69.5%, 80 = 77%, 90 = 89%, 99 = 97%,
  with no level, fatigue or score effect. Yours have no dice: two button presses on a swinging cursor, and the Free
  Throws rating only sets how fast the cursor moves (faster on harder levels). *From code.*
- **Passes:** on Starter and All-Star a long pass can go off line, less often with a better Passing rating. On Rookie,
  passes never go off line. *From code.*
- **Ratings that do nothing:** Def. Rebounding, Speed, Jumping, Dribbling, Strength and the Stamina on the player
  screen are never read in a game. Every rebound, at both ends, uses Off. Rebounding. The position on the player card
  isn't used either; height only picks the size of the player's picture, and weight is what counts when two players
  bump. *Tested (we watched which ratings the game reads through a long stretch of play).*
- **The real stamina is hidden.** Each player has a rating that's never shown, and that's the one that sets how fast
  he tires. Sam Cassell (Stamina 83 on screen) tires the fastest of the five Rockets we measured; Otis Thorpe (79) is one of
  the two slowest. *Tested.* It's in `csv/player_ratings.csv` as `hidden_real_stamina`.
- **Season games you don't play** are simulated from each team's real 1993-94 points scored and allowed, with a random
  amount, and a rubber band: after 8 games, a team winning more than it did in 1993-94 gets lower scores, and one
  doing worse gets higher ones. Trades count: a new starting five changes the team's scoring and defense numbers.
  Player box scores are made up from each player's real 1993-94 averages. *From code.*
- **The 1994 Finals rematch, 20 times:** Knicks at Rockets, computer vs computer (Starter, Arcade style, 5-minute
  quarters, a different dice seed each game): **10-10**. The Knicks won games 1-4, so a best-of-7 in game order is a
  Knicks sweep. Average score 62.4-61.1; Ewing 13.8 points a game, Olajuwon 14.2; top scorer Vernon Maxwell, 16.9.
  *Tested,* with caveats: Arcade style has no free throws, all games were in Houston, 20 games is a small sample, and
  player points are given to the last player of the scoring team who held the ball (not read from a box score)
  (`tests/finals_rematch_20_games.csv`).

## How we tested

In the BizHawk emulator, with a script that sets up each test (which buttons are pressed on which frame), saves and
reloads exact moments of a game, and logs every shot: the shooter, his distance, the rating used, every bonus, the
chance, the roll and the result. The chance the game worked out matched the formula above on every logged shot.

- **Players placed on exact spots:** the shooter is walked to a spot (or, for Ewing's thin spot, his feet held on
  it), turned to face the basket, and the shot taken with a good release.
- **The same dice reused, so shots compare:** a shot is taken from a saved moment, then the moment is reloaded and
  the same shot taken again with one thing changed (Sega Assistance, the energy, the ratings). Because the random
  numbers repeat (above), the roll is the same, and only the chance moves.
- **The Miller ratings on Starks:** Reggie Miller's ratings were written over John Starks' in the emulator's memory,
  everything else the same, and the four Starks shots replayed.

**Test changes** (the tests are lab tests, and the video says so on screen):
- In the step-back, Ewing, painted-line, Sega Assistance and 3-point contest tests, the other players are moved off
  to the far end of the court, the shot clock is held and the camera is held still. The score box after a basket was
  hidden in some clips (looks only).
- Sega Assistance tests: the score and clock were set in memory (Knicks down 0-10 at 1:30, or Houston down 0-10 at
  1:51), and the option was switched in memory between the off and on shots.
- Dunk fatigue test: Ewing's energy was written at the moment the dunk is checked.
- Every-dunk test and the dunk contest: each dunk was forced in turn (the game normally picks at random); in the
  every-dunk test the layup choice was skipped too.
- Painted-line test: the shots were forced in, to read the scoreboard (2 or 3).
- The 20-game Finals rematch and the 3-minute Knicks at Rockets clip are the unmodified game, computer vs computer.

The game itself is not included here and never will be: **no ROMs, no game code, no game graphics.** Only our own
notes, measurements and charts.

Found something wrong, or something new? Open an issue.

The other games are on the [main page](../README.md).

---

Not affiliated with Electronic Arts, EA Sports, Sega, Nintendo, the NBA or the NBPA. Game, team and player names
belong to their owners.
