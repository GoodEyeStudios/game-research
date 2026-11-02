# Tecmo NBA Basketball (NES, 1992): what's inside the cartridge

Findings from **GoodEye Episode 5**, the Tecmo NBA Basketball deep dive: the video is coming soon to https://www.youtube.com/@GoodEye-Studios

Every player in Tecmo NBA Basketball has a block of numbers in the ROM. Some are on the team screens (height, weight,
jersey); the rest are hidden. We read the game's code to find out what each number does, then tested it in an emulator,
shot by shot, to check.

**Confirmed** = tested in the running game. **From code** = read in the code, not tested yet.

## The short version

- **Your shooting percentage barely matters. Your range does.** Every player has a hidden range from 1 to 10. Shoot from
  past it and your chance is cut in half, then in half again. Greg Dreiling is stored as a 97% three-point shooter, but
  his range is 5: in 20 open threes from the top of the key he made **5**. Rik Smits, stored at **0%**, made 7.
  John Starks (range 10) made 16. *Confirmed.*
- **Release timing is the biggest thing you control.** Let go at the top of your jump (B held about 27-28 frames).
  Reggie Miller's open top-of-key three: about 32% if you let go early, about 75% at the top. *Confirmed.*
- **A defender in your face barely changes the make chance** (about 5 points). Defense works through blocks. *Confirmed.*
- **Height doesn't matter for blocks. A hidden jump number does.** Same shot, same jump moment, 20 shots each:
  Manute Bol (7'7") 18 blocks, David Robinson (7'1") 18, Michael Jordan (6'6") 19, Rod Strickland (6'3") 6,
  Muggsy Bogues (5'3") 3. *Confirmed.*
- **The roster bug:** when a computer-controlled team loads, every player gets the same block and steal ratings,
  copied from the team's first roster record (usually the point guard). Computer-run David Robinson blocks 12 of 20
  instead of 17. *Confirmed.*
- **Coach mode** (COA) gives your team about +10% on every shot. **Losing** adds a little to every shot, more the further
  behind you are. *Confirmed.*

## Files

| File | What it is |
|---|---|
| [`player_ratings.csv`](player_ratings.csv) | All 324 players and every rating, with readable column names |
| [`charts/`](charts) | Where to shoot (court maps), release timing curve, defender distance curve |
| [`charts/expected_points_by_spot.csv`](charts/expected_points_by_spot.csv) | Every value behind the court maps |
| [`tests/shooting_20_open_shots.csv`](tests/shooting_20_open_shots.csv) | Starks, Dreiling and Smits: 20 open shots from three spots, every shot |
| [`tests/free_throws.csv`](tests/free_throws.csv) | The same three players, 20 real free throws each |
| [`tests/blocks_robinson_by_rating.csv`](tests/blocks_robinson_by_rating.csv) | David Robinson with five different block ratings, 20 shots each |
| [`tests/blocks_by_defender.csv`](tests/blocks_by_defender.csv) | Bol, Robinson, Jordan, Strickland, Bogues at the same block rating |
| [`tests/roster_bug_check.csv`](tests/roster_bug_check.csv) | Four computer teams: each player's own ratings vs what the game actually used |

## The ratings (`player_ratings.csv`)

| Column | What it does | Status |
|---|---|---|
| `fg_pct`, `ft_pct`, `three_pt_pct` | The player's real 1991-92 percentages. The starting point for every shot | Confirmed |
| `shot_range_1to10` | How far out he can shoot before his chance gets halved. Shooters 10, centers 5-6 | Confirmed |
| `speed` | Running speed | Confirmed |
| `pass_intercept` | Chance to pick off a pass to the man he guards, when the ball flies right past him. Not used for steals off the dribble | Confirmed |
| `block_rating` | Added to the block roll (x4 in the game) | Confirmed |
| `jump_power` | How fast a jump starts, so how high he gets: his block reach | Confirmed |
| `stamina_guess` | Copied into the game's "condition" table at tip-off | Guess |
| `sim_shot_volume` | How many shots he takes when the game simulates a game you don't play | From code |
| `position_guess` | 0-4, lines up with PG/SG/SF/PF/C | Guess |
| `skin_tone_flag`, `graphics_flag` | Sprite colours / another sprite flag | From code / unknown |
| `unused_nibble` | Not read by any code found | Unknown |
| `jersey`, `height`, `weight_lb` | As shown on the team screens. **Height isn't used by any code found** | Confirmed |

Names are spelled the way the game spells them (yes, "Lasalle Thopmson" and "Alezander Volkov").

## How a shot works

Every shot ends with a dice roll from 0 to 255: at or under the final chance, it's good. (About 1 shot in 10 that the
dice call a miss still rattles in.) The final chance is built roughly like this:

1. Start with the stored percentage (FT%, FG% or 3P%).
2. Add a bit for being open (small).
3. Divide by a shot-style number.
4. **Release timing:** up to about +40% at the top of the jump; early or late releases lose it. The computer's timing is
   worse on easier settings.
5. **Range:** past the player's range, the chance is **halved**, more halvings the further out. This applies to free
   throws too: the free-throw line is past range 6-7, so big men shoot free throws badly whatever their FT%.
6. **Comeback:** +1/256 per point the shooter's team is behind, up to 80 points.
7. Long-distance divider: from mid-range out it climbs 2, 3 ... 9, then 16, 25, 36, 49, 64, 81, 100 for very long shots.
   *From code.*
8. Layups and dunks at the rim get a bonus; a **coached** team gets +25/256.

**How percentages are stored:** real 1991-92 percentage x 250, rounded down, as a value out of 256. For the
1991-92 Pacers that matches 28 of 30 numbers exactly (Miller's and Schrempf's 3P% are 1-2 units lower). So a stored
"97.3%" (249/256) is the top of the scale: a perfect 1.000. Greg Dreiling went **1 for 1** from three that season.
Jon Sundvold and Pete Chilcutt are stored at the same 97.3%.

## How a block works

Each frame a jumping defender touches the rising ball (up to 16 times a shot), the game rolls. The chance is a bonus
for being at the top of his jump plus his block value. A winning roll only blocks if the ball is within his reach, and
reach comes from `jump_power`, not height.

**Bug 1, the first touch:** on the first frame of contact, while the defender is still rising fast, the timing table
gives a small negative number that gets treated as a big positive one, so the first roll almost always wins. That's why
Robinson blocks 12 of 20 even with a block rating of 0. *Confirmed.*

**Bug 2, the roster slot:** when a game loads, the code reads the block and pass-intercept ratings from a fixed spot
instead of each player's own record, so all five players get the first player's values (`tests/roster_bug_check.csv`).
This hits computer-controlled teams, and human teams on the hardest setting. On the normal setting the game gives every
human-controlled player the maximum (block 64, intercept 128) instead. *Confirmed.*

| David Robinson's block value | 0 | 8 (the bug) | 44 (his real one) | 64 (2-player games) | 128 |
|---|---|---|---|---|---|
| Blocks out of 20 | 12 | 12 | 17 | 16 | 20 |

## Other things in the cartridge

- The save file is checked for the name **BOBHENDERSON**.
- The ROM contains the title **TECMO SUPER PRO BASKETBALL**, printed by a typewriter-style text routine.
- Magic Johnson starts for the Lakers, though he retired early in the 1991-92 season without playing a game.
- Steals off the dribble use no rating: once the defender is close, it's a flat roll of about 15%.
- John Starks's numbers slid one box over: his real 1991-92 free-throw percentage (.778) is in his three-point box
  (stored 75.8%), and his real field-goal percentage (.449) is in his free-throw box (stored 43.8%).
  His field-goal box (stored 42.6%) holds his 1990-91 field-goal percentage (.439).

## How we tested

In the Mesen emulator, with a small script that sets up each test (who's on the floor, where the defender stands, when
the shooter lets go) and logs what the game rolled. Each test player shot with fresh dice every time, the score and
clock running. Makes and blocks were counted from what the game did on screen, not from the dice alone.

Testing used a copy of the ROM with a logging routine added and some ratings changed for specific tests. The game itself
is not included here and never will be: **no ROMs, no game code, no game graphics.** Only our own notes, measurements
and charts.

Found something wrong, or something new? Open an issue.

---

Not affiliated with Tecmo, Koei Tecmo or the NBA. Game and team names belong to their owners.
