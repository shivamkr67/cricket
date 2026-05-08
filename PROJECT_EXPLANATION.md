# Cricket Game Project Explanation

This project is divided into simple parts so it is easy to open in VS Code and explain.

## 00_Game

- `CricketGameManager.cs`: keeps score, overs, wickets, messages, and scoreboard.
- `ShotType.cs`: stores the shot names like Drive, Defence, Loft, Pull, and Cut.

## 01_Batting

- `BatSwingController.cs`: reads batting keys, swings the bat, moves the arms, and hits the ball.
- `BatContactReporter.cs`: tells the bat script when the ball touches the bat.

## 02_Bowling

- `BowlerController.cs`: launches the ball when Space is pressed.
- `CricketBallController.cs`: handles ball physics, bounce, hit state, and stopping.

## 03_Fielding

- `FielderController.cs`: makes fielders chase and catch/stop the ball.
- `BoundaryZone.cs`: gives four or six when the ball reaches boundary.
- `Wicket.cs`: detects bowled wicket.
- `GroundSurface.cs`: marks ground/pitch so the game knows when the ball bounced.

## 04_Camera

- `CricketCameraController.cs`: follows the ball during play and returns to batting view.

## Editor Scene Builder

- `CricketPrototypeSceneBuilder.cs`: creates the ground, pitch, players, fielders, umpire, camera, and materials.

## Demo Controls

- `Space`: bowl
- `J`: drive shot
- `K`: defence shot
- `L`: loft shot
- `I`: pull shot
- `O`: cut shot
- `A/D`: aim shot left/right
- `R`: reset match
