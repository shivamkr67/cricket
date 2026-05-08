# Next Version Explanation

These are the improvements added after the first evaluation.

## 1. Stadium and boundary improvement

The boundary now has extra visual details:

- Buildings near the boundary
- Sponsor boards around the ground
- Corner towers
- Pavilion and scoreboard
- Flood lights
- Seating stands
- Trees outside the ground
- Oval stadium wall and oval seating rings
- Sponsor boards arranged around the oval boundary

These objects are made from simple primitives like cubes, cylinders, and spheres, so they are easy to explain.

## 2. Miss ball means bowled

The bowling target is now aligned toward the batting wicket.

If the player does not press a shot key at the correct time:

1. The bat collider stays disabled.
2. The ball passes the batsman.
3. The ball hits the wicket.
4. `Wicket.cs` informs `CricketGameManager.cs`.
5. Scoreboard shows the batsman is bowled.

## 3. Next batsman walking in

After a wicket:

1. `CricketGameManager.cs` increments wickets.
2. It calls `BatsmanEntryController.cs`.
3. The new batsman appears near the entrance.
4. The new batsman walks toward the crease.
5. Then the next ball can be played.

## Files to show

- `Assets/Scripts/00_Game/CricketGameManager.cs`
- `Assets/Scripts/01_Batting/BatsmanEntryController.cs`
- `Assets/Scripts/02_Bowling/BowlerController.cs`
- `Assets/Scripts/03_Fielding/Wicket.cs`
- `Assets/Editor/SceneBuilder/CricketSceneFactory.cs`

## Simple explanation line

"I improved the project by adding stadium buildings and a real cricket dismissal flow. If the batsman misses, the ball hits the stumps, the wicket is counted, and a new batsman walks in."

## 4. Powerplay and 2 overs

The match is now a two-over game.

- First two balls are powerplay.
- During powerplay only two fielders are deep.
- After two balls, five fielders move near the boundary.

This is controlled by `CricketGameManager.IsPowerplay` and the fielder home positions in `FielderController.cs`.

## 5. Extra shots

The batting script supports more shots:

- `J` Drive
- `K` Block
- `L` Loft
- `I` Pull
- `O` Cut
- `U` Sweep
- `Y` Upper Cut
- `H` Helicopter
