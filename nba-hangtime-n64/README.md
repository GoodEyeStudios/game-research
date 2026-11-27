# NBA Hangtime (Nintendo 64, 1996): the hidden rules

Findings from the GoodEye video on NBA Hangtime's hidden rules: coming soon to https://www.youtube.com/@GoodEye-Studios

NBA Hangtime on the N64 is a port of Midway's 1996 arcade game. We read the N64 game's code to find out how a shot
is really decided, compared it with the arcade original, then tested it in the running game, shot by shot.

**Tested** = seen in the running game. **From code** = read in the code, not tested separately. **Worked out** =
calculated from tested and code numbers. Everything is the N64 version (USA) unless it says arcade.

## The short version

- **Tied, last 4 seconds of the 4th quarter (or overtime): a jump shot drops to 10%.** The same shot is 70% with a
  minute left, and 85% if it would tie the game. *Tested.*
- **The best-paying shot in the game is the corner three.** It's the shortest three, so it has the best chance of any
  three, for the same 3 points: 2.61 points a shot for a top shooter, against 2.00 for a close two. *Worked out from the
  game's formula.*
- **The score changes every shot, for both teams.** +1.4% per point you're behind, and -1.4% per point you're ahead.
  The same open jump shot: 56% ahead by 10, 70% tied, 84% behind by 10. *Tested.*
- **The game can't do circles either.** Distance = the longer side plus three-eighths of the shorter one: an octagon.
  There isn't a single square root in the game's code. *Tested and from code.*
- **The tips screen gets a lot wrong.** No clutch rating, no rebound rating, no fatigue, and on the N64 a push takes
  the ball away from anyone. *Tested and from code.*
- **Fire, exactly:** 3 baskets in a row, then 4 more baskets, or one basket by the other team. A goaltended basket
  doesn't put it out, and it does nothing for layups. *Tested.*

## Files

| File | What it is |
|---|---|
| [`csv/player_ratings.csv`](csv/player_ratings.csv) | All 181 player records: the six ratings on the select screen and the two hidden ones (pass, clutch), team and height. Includes the secret players |
| [`csv/jump_shot_chance_by_distance.csv`](csv/jump_shot_chance_by_distance.csv) | An open jump shot's chance by distance and Shoot rating (0-10), tied game, worked out with the game's formula |
| [`csv/points_per_shot.csv`](csv/points_per_shot.csv) | Points per open shot: corner three, top-of-the-key three, close two, long two |
| [`csv/team_cpu_skill.csv`](csv/team_cpu_skill.csv) | The computer's hidden skill floor for each team it plays |
| [`tests/score_changes_the_shot.csv`](tests/score_changes_the_shot.csv) | The same jump shot with only the score changed, ahead by 10 to behind by 20 |
| [`tests/last_seconds_shot.csv`](tests/last_seconds_shot.csv) | The same jump shot with 3 seconds, 1.5 seconds and 1 minute left, in the 1st and 4th quarters, at different scores |
| [`tests/layups_697.csv`](tests/layups_697.csv) | 697 tries of the same layup, at four scores |
| [`tests/brick_and_fire.csv`](tests/brick_and_fire.csv) | The same jump shot and layup, normal, after 3 straight misses off the rim, and on fire |
| [`tests/three_point_line.csv`](tests/three_point_line.csv) | Spots just inside and outside the 3-point line, and along the corner strips: 2 or 3 points |
| [`tests/cpu_vs_cpu_16_games.csv`](tests/cpu_vs_cpu_16_games.csv) | 16 full games, computer vs computer: final scores |
| [`tests/goaltending_n64_vs_arcade.csv`](tests/goaltending_n64_vs_arcade.csv) | The same 14 goaltending touches under the N64 rule and, 50 times each, under the arcade's rule |
| [`tests/height_and_blocks.csv`](tests/height_and_blocks.csv) | 42 blocks replayed with only the blocker's height changed |
| [`tests/pushes_48.csv`](tests/pushes_48.csv) | 48 pushes on the ball carrier: who kept the ball |
| [`charts/`](charts) | The charts from the video: the octagon, points per shot, the corner three, the score effect, the re-tune, the computer's skill by team, the 16 games, the push table and goaltending |

**Game distance** is the distance the game thinks a shot is from (the octagon distance, below), in the game's court
units. A shot's **value** is out of 1000, and its chance is (value + 50) / 1000: a value of 650 is a 70% shot.
Ratings go from 0 to 10.

## Rule 1: the best shot in the game

**How a jump shot is decided** (the N64 code is the arcade's, ported almost line for line). *From code; the whole
formula matched the game's own number on 272 of 272 shots in 6 computer-vs-computer games* (*tested*):

1. Start from the Shoot rating: Shoot 0 = 860 up to Shoot 10 = 1200.
2. A three by a shooter with Shoot 5 or lower: -200 (that's 20%). A three from far behind the line: -150.
3. Inside 112 units of the hoop ("close in"): +250. Half court and beyond: -990.
4. Each defender close by: -240; a defender in front of the shot takes off more the closer he is.
5. **-2 for every unit of distance.**
6. The score (Rule 3), the last seconds (Rule 3), fire (Rule 5).
7. Chance = (value + 50) / 1000.

When you let go of the button isn't part of it (see Rule 4). *From code.*

**The corner three.** The 3-point line is a hand-made table, the same one as the arcade. Along both sidelines, past
the end of the table, every jump shot is a three. *From code, and tested: 23 spots, `tests/three_point_line.csv`.*
The corner spots are 182-216 units from the hoop; the top of the key is 232. A three is worth 3 wherever you take it,
and every unit closer is a better chance, so **the corner three pays the most of any shot.** Open shot, tied game,
Shoot 10 (*worked out*, `csv/points_per_shot.csv`):

| Shot | Chance | Points per shot |
|---|---|---|
| Corner three (about 190 units) | 87% | **2.61** |
| Top-of-the-key three (232) | 79% | 2.36 |
| Close two (inside 112) | 100% | 2.00 |
| Long two (200) | 85% | 1.70 |

A shooter with Shoot 5 or lower takes the -20% on every three, so the corner three is a good shooter's shot. A dunk
isn't in this table: dunks and layups have their own values (below). In the video, Scott Skiles hits one from the
corner in the unmodified game.

## Rule 2: the octagon, again

- To know how far you are from the hoop, the game takes **the longer side (out or across) and adds three-eighths of
  the shorter side.** *From code; tested: it matched the game's own distance on 272 of 272 shots.* NBA Live 95 does the
  same with a quarter (see [the NBA Live 95 folder](../nba-live-95)).
- So Hangtime's circles are octagons too. Its worst error is about 7% (shots about 20 degrees off straight on count up
  to 7% long; the diagonal counts about 3% short). NBA Live 95's diagonals count up to 12% short. *Worked out.*
- **There isn't a single square root in the game's code** (no square-root instruction anywhere), even though the N64's
  chip can do one in a single instruction. It kept the arcade's shortcut. *From code.*
- Step one unit past the close-in line (112) and an open shot loses the +250: **25% gone in one step.** *From code;
  `csv/jump_shot_chance_by_distance.csv`.*

## Rule 3: the rubber band

**The score is in every jump shot, both ways:** +14 (1.4%) per point behind and -14 per point ahead. The same open
jump shot, from the same moment with the same dice, only the score changed (*tested*, `tests/score_changes_the_shot.csv`):

| Shooter's team | Chance |
|---|---|
| ahead by 10 | 56% |
| ahead by 5 | 63% |
| tied | 70% |
| behind by 5 | 77% |
| behind by 10 | 84% |
| behind by 20 | 98% |

- A team that isn't losing also has a 5.5% chance that a jump shot is knocked down to a value of 350. *Tested (about
  5% of 316 tries tied or ahead, 0 of 162 behind).*
- **In 1 player vs the computer, the computer's help can't be switched off.** The CPU Assistance option (and the
  matchup-screen code for it) only works with humans on both teams. *From code.*
- **The computer's hidden skill:** for each team the computer plays, a hidden skill value goes up when it falls behind
  and down when it leads, between a floor and a ceiling that rise during the game. It sets the computer's pushes,
  steals, blocks, alley-oops and reactions, not its shot chance. The Bulls have by far the highest floor; nine teams
  share the bottom. *From code* (`csv/team_cpu_skill.csv`, `charts/05_computer_skill_by_team.png`). The computer
  partner of a human player gets none of it.
- **The N64 re-tunes it 4 times a game instead of 12.** The arcade re-tunes the computer every minute of the clock;
  the N64 port only at the end of each quarter (a port bug: it checks the wrong number). *From code; tested: in a full
  logged game the skill only changed at the 4 quarter ends.*
- Losing the 3rd quarter on purpose isn't worth it: the 4th-quarter ease-off is small. *From code.*
- **16 full games, computer vs computer** (8 Lakers vs Bulls, 8 Bulls vs Grizzlies): every one ended within 8 points;
  two went to overtime, and one was still tied when the test stopped. The matchup screen said "Lakers favored by 9";
  the Bulls won 6, the Lakers 1. *Tested* (`tests/cpu_vs_cpu_16_games.csv`).
- **"Favored by N" is a random number from 1 to 10.** On a new save the team on the left is always the favourite.
  *From code.*

**The last 4 seconds.** In the last 4 seconds of a quarter the game sets minimum values by score. In the 4th quarter
or overtime, **a tied game's jump shot is set to 50: a 10% chance**, and a shot that would tie it (down 2) is set to
800: 85%. In the 1st quarter, or with a minute left, the same shot is normal (70%). Midway's own note in the arcade
code: "Last second shot of a tie game should almost never go in." *Tested with a jump shot*
(`tests/last_seconds_shot.csv`, and a 55-55 game in the video). Layups and dunks get their own values later in the
code, so this cap looks like it's for jump shots; *from code, not tested.*

## Rule 4: the tips screen lies

The game's own tips, checked against the code:

| Tip | Verdict |
|---|---|
| "The greater a player's hidden clutch rating, the more chance that he will make a great play during the last 10 seconds of any period." | **False.** Every player has a hidden clutch rating, and nothing in the game ever reads it. *From code.* (Kobe Bryant's is 1, one of the three lowest.) |
| "The greater a player's hidden rebound rating... He will also jump higher" | **False.** There's no rebound rating. The first hand to reach a loose ball gets it: height and the Block rating set the reach. *From code.* |
| "Powerful guys tend to hold onto the ball when cleared out!" | **False on the N64.** The table that gives a pushed ball carrier a chance to keep the ball by his Power is all zeros in the N64 version. Unless the two Power ratings are within 1, he always loses it. *Tested: 48 pushes, `tests/pushes_48.csv`.* |
| "Big guys are poor 3 point shooters!" | **It's Shoot, not size.** The -20% on threes is for a Shoot rating of 5 or lower, whatever the height. *From code.* |
| "Increase the chances of a jump shot going in by releasing the Shoot button at the peak of your jump." | **False.** Release timing isn't in the shot formula. *From code.* |
| "Some players may become fatigued during the second half!" | **False.** There's no fatigue at all. *From code.* |
| "Remember, small players can't clear out big players!" | **False.** A push lands if you're close and in front; size never stops it. *From code.* |
| "Layups are generally less successful than dunks." | **True.** Every layup is 90%, whatever the ratings. If your team is tied or ahead, 1 time in 10 it drops to 70%. *Tested: 697 layups, `tests/layups_697.csv`.* |
| "Smarter Drone" (a Create Player reward, 60 points) | **Does nothing on the N64.** It should boost your computer-run teammate; the code gives it to you, and you're not computer-run. *From code.* |

The N64 manual says the Smarter Drone needs 70 points; the code checks 60. *From code.*

**Goaltending:** on every frame a defender touches a falling ball, the arcade rolls the dice and skips the touch 4 times
out of 5 ("Took out 4/5/96 - 2 days before shipping... Genius!", says the programmer's note, but the check is still
there). The N64 port dropped it: the first touch is goaltending.
The same 14 touches: **N64, goaltending 14 of 14; the arcade's rule, 440 of 700 (63%).** So the N64 calls
goaltending about 1.6 times as often. *Tested* (`tests/goaltending_n64_vs_arcade.csv`).

## Rule 5: fire, exactly

- **3 baskets in a row** by the same player and he's on fire. *Tested (4 times).*
- **It lasts 4 more baskets**, or until the other team scores. *Tested.*
- **A teammate's basket doesn't count and doesn't put it out.** *Tested.*
- **A goaltended basket by the other team doesn't put it out either**: a goaltend is scored by a different routine.
  *Tested (in the video).*
- **On fire, a jump shot gets +750** (75%): almost every jump shot goes in. Past 327 units the shot is set to 100 instead (15%). *Tested: a 650 shot became 1400;
  the deep-shot rule from code.*
- **Layups get nothing from fire.** *Tested: the same layup was 850 with and without fire* (`tests/brick_and_fire.csv`).
- **3 straight misses off the rim, and the next shot is set to 990 out of 1000: almost a sure thing** (any shot short
  of a half-court heave; the score still adds or takes off after that). *Tested: a 650 shot became 990.*
- Team fire (3 alley-oops or double dunks in a row) lasts 25 game "seconds" of your own possession, about 21 real
  seconds. *From code.*

## Rule 6: locked in the cartridge

- **JASON 0729 works on the N64** (lists of codes say it doesn't): it gives Skiles. *Tested on the unmodified game.*
- **MUNDAY 5432 works, but gives Jamie Rivett**, not Munday. *Tested on the unmodified game.*
- **Marius** (code 1005 in the arcade) is still complete on the N64: photo, in-game head and ratings. His PIN was
  changed to numbers you can't type, so no code reaches him. *From code; tested on a modified game (the PIN changed in
  memory, then typed on the real entry screen).*
- **M-Doc** has a full photo but a PIN with digits you can't enter. **North** and **Jiggetts** show a red X; their
  heads are still in the cartridge, unused. Two more unused heads match **Bud** and **Munday** (the one with the green
  hat). *From code; which head is whose is our best match by the arcade's order.*
- The cartridge also holds 117 sounds (about 2 minutes) that no code can play. *From code.* We haven't listened to
  them all yet.

## Other findings

- **Height is reach.** The same 42 blocks with only the blocker's height changed: at 5'7" only a third were still
  blocks and half became baskets. Weight only widens the hit box a little. *Tested* (`tests/height_and_blocks.csv`).
- **Create Player:** +2 attribute points every 4th win, up to 62; the trivia champion reward adds 10 once, so 72 is the
  most. *From code.*
- **The clock runs 2.5% faster with 4 human players.** *Tested.*
- **No home-court advantage:** the team on the left only picks the arena. *From code.*
- **Steals use the Steal rating only** (12% to 22.5% a touch); size and speed aren't in it. *From code.*
- **No 3-point dunks.** Dunks and layups are always worth 2. *From code.* An arcade loophole (a human teammate
  catching your three in an alley-oop) is still in the code, but in 33 tries we couldn't make it happen. *Tested.*
- No hot spots: those are a Maximum Hangtime feature, and the N64 is built from regular Hangtime. *From code.*

## How we tested

In the BizHawk emulator, with a script that sets up each test, saves and reloads exact moments of a game, and logs
what the game's code works out: the shooter, the distance, every part of the shot value, the roll and the result. The
game's dice can be set, so the same shot can be replayed from the same moment with one thing changed.

**Test changes** (the tests are lab tests):
- The score, the time left, the quarter or the "on fire" and "missed 3 in a row" counters were set in memory for the
  same-shot tests; the 697 layups set the dice and the points-behind number before each try.
- The 16 games, the tied 55-55 ending and the Kobe layups are computer vs computer: all four players run by the game,
  with the dice set at tip-off. Nothing else was changed.
- The 3-point line spots: the player was placed on each spot in memory, then shot normally. The corner three in the
  video is the unmodified game (the player walked there with the controller).
- The blocks: the blocker's height was changed the way the game would set it for that height.
- Marius, M-Doc, North, Jiggetts and Bud: their codes were changed in memory to be typeable. In the video those clips
  are labelled MODIFIED GAME.
- The goaltending comparison: the arcade's random check was copied onto the N64 game in the emulator.

The game itself is not included here and never will be: **no ROMs, no game code, no game graphics.** Only our own
notes, measurements and charts.

Found something wrong, or something new? Open an issue.

The other games are on the [main page](../README.md).

---

Not affiliated with Midway, Williams Entertainment, Nintendo, the NBA or the NBPA. Game, team and player names belong
to their owners.
