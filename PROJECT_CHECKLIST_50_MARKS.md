# Cricket Project Checklist

Use this as a quick revision sheet before showing the project.

## Gameplay features

- Bowling starts with `Space`.
- Batting works only when a shot key is pressed near the ball.
- Shot keys are `J` drive, `K` block, `L` loft, `I` pull, and `O` cut.
- Fielders chase the ball and can stop or catch it.
- Boundaries give `4` or `6`.
- Wickets, score, overs, and reset are handled by code.

## Visual features

- Batsman, non-striker, bowler, fielders, and umpire are present.
- Bat is attached to the batsman body.
- Ground, pitch, boundary rope, ball, bat, players, and umpire use visible materials.
- Material assets are stored in `Assets/Materials`.
- Simple stadium has walls, seats, pavilion, scoreboard, entrance gate, floodlights, and trees.
- Boundary area has extra buildings, corner towers, and sponsor boards.
- If the batsman misses, the ball can hit the wicket and trigger an out.
- After a wicket, a new batsman walks in from the entrance side.
- Match is now `2` overs.
- First `2` balls are powerplay: only `2` fielders are deep.
- After powerplay, `5` fielders move near the boundary.
- Extra shots added: sweep, upper cut, and helicopter shot.
- Stadium uses an oval/circular wall, seating, and boundary rope.

## Code features

- C# scripts are separated by topic inside `Assets/Scripts`.
- Editor-only scene creation code is separated inside `Assets/Editor/SceneBuilder`.
- The project has a VS Code workspace file: `CricketGame.code-workspace`.
- The project has a code-only Visual Studio solution: `CricketPrototype.CodeView.sln`.

## Demo order

1. Open `CricketGame.code-workspace` in VS Code.
2. Show `Assets/Scripts/README.md`.
3. Show `CricketGameManager.cs`, `BatSwingController.cs`, and `FielderController.cs`.
4. Show `CricketVisualPalette.cs` and `Assets/Materials`.
5. Open Unity and play the scene.

## Safe design scene

- Use `Assets/Scenes/MyCricketStadiumDesign.unity` for custom stadium work.
- Use `Cricket Prototype > Backup Design Scene` before large edits.
