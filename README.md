# 3D Cricket Prototype

This is a beginner-friendly Unity cricket prototype for Unity 6 `6000.3.4f1`.

## Open the project

1. Open Unity Hub.
2. Click **Add project from disk**.
3. Select `C:\Users\shiva\OneDrive\Documents\New project`.
4. Open it with Unity `6000.3.4f1` or a newer `6000.3.x` patch.

## Build the scene

After Unity finishes compiling:

1. Click `Cricket Prototype > Build Starter Scene`
2. Open `Assets/Scenes/CricketPrototype.unity`
3. Press Play

## Custom stadium design

Before you edit the stadium, click:

1. `Cricket Prototype > Make Editable Design Copy`
2. Open `Assets/Scenes/MyCricketStadiumDesign.unity`
3. Design and save that scene

Read `DESIGN_SAFETY_GUIDE.md` before making big visual changes.

## Controls

- `Space` bowl
- `Q / E` move bowling line
- `Up / Down` change pace
- `J` drive
- `K` block
- `L` loft
- `I` pull
- `O` cut
- `U` sweep
- `Y` upper cut
- `H` helicopter shot
- `A / D` place the shot
- `R` reset match

If you do not press a shot key, the ball is aimed at the stumps and can bowl the batsman.

## What is included

- Small playable batting prototype
- Bowler, batsman, non-striker, umpire
- Fielders who chase and try to catch the ball
- Boundary detection for fours and sixes
- Bowled wicket if the batsman misses the delivery
- Next batsman walking-in animation after a wicket
- Boundary-side buildings, sponsor boards, corner towers, flood lights, and pavilion
- Oval/circular stadium seating and boundary rope
- First 2 balls powerplay fielding, then normal fielding
- Two-over match format
- Score, wickets, overs, and basic run logic
- Small top-left score card that hides during live play
- Scene builder with simple hierarchy names

## Script structure

- `Assets/Scripts/00_Game`
- `Assets/Scripts/01_Batting`
- `Assets/Scripts/02_Bowling`
- `Assets/Scripts/03_Fielding`
- `Assets/Scripts/04_Ball`
- `Assets/Scripts/05_Camera`
- `Assets/Scripts/06_Visuals`

## VS Code

For the cleanest code view, double-click:

- `OPEN_CRICKET_CODE_IN_VSCODE.bat`

Or open this file manually in VS Code:

- `CricketGame.code-workspace`

If VS Code still does not show Unity C# IntelliSense properly, follow `UNITY_VSCODE_SETUP.md`.

## Visual Studio

If you want a simple code-only view in Visual Studio, open:

- `CricketPrototype.CodeView.sln`

## Visual setup

Material colors are defined in `Assets/Scripts/06_Visuals/CricketVisualPalette.cs`.

The visible material files are stored in:

- `Assets/Materials`

The editor script `Assets/Editor/SceneBuilder/CricketMaterialFactory.cs` keeps those material files updated when the scene is built.

## Demo checklist

Before showing the project, read:

- `PROJECT_CHECKLIST_50_MARKS.md`
