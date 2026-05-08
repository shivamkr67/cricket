# Full C# Code Copy Paste

Use this file only if VS Code is not showing the scripts properly. Each section shows the exact file path and the full code that should be inside it.

## Runtime Gameplay Scripts

### `Assets\Scripts\00_Game\CricketGameManager.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public sealed class CricketGameManager : MonoBehaviour
    {
        [Header("Match")]
        [SerializeField] private int maxOvers = 2;
        [SerializeField] private int maxWickets = 10;
        [SerializeField] private float nextBallDelay = 1.15f;

        [Header("References")]
        [SerializeField] private BowlerController bowler;
        [SerializeField] private MatchHudController hud;
        [SerializeField] private BatsmanEntryController batsmanEntry;

        private bool deliveryResolved;
        private float nextBallReadyAt;

        public int Score { get; private set; }
        public int Wickets { get; private set; }
        public int LegalBalls { get; private set; }
        public CricketBallController LiveBall { get; private set; }
        public bool IsDeliveryActive { get; private set; }
        public bool IsMatchOver => LegalBalls >= maxOvers * 6 || Wickets >= maxWickets;
        public bool IsPowerplay => LegalBalls < 2;
        public int OversCompleted => LegalBalls / 6;
        public int BallsInCurrentOver => LegalBalls % 6;
        public bool ShowScoreCard => !IsDeliveryActive || IsMatchOver;
        public string ScoreLine => $"Score {Score}/{Wickets}   Overs {OversCompleted}.{BallsInCurrentOver}/{maxOvers}   {(IsPowerplay ? "Powerplay" : "Open Field")}";
        public string LastEvent { get; private set; } = "Press Space to bowl the next ball.";

        public void SetBowler(BowlerController controller)
        {
            bowler = controller;
        }

        public void SetHud(MatchHudController controller)
        {
            hud = controller;
        }

        public void SetBatsmanEntry(BatsmanEntryController controller)
        {
            batsmanEntry = controller;
        }

        private void Update()
        {
            if (hud == null)
            {
                hud = FindFirstObjectByType<MatchHudController>();
            }

            if (Input.GetKeyDown(KeyCode.R))
            {
                ResetMatch();
            }

            if (!IsDeliveryActive && !IsMatchOver && Time.time >= nextBallReadyAt && LastEvent == string.Empty)
            {
                LastEvent = "Press Space to bowl the next ball.";
            }
        }

        public void RegisterDelivery(CricketBallController ball)
        {
            LiveBall = ball;
            IsDeliveryActive = true;
            deliveryResolved = false;
            LastEvent = IsPowerplay
                ? "Powerplay ball. Only two fielders are deep."
                : "Open field. Five fielders are near the boundary.";
        }

        public void RegisterHit(CricketBallController ball, ShotType shotType, float quality)
        {
            if (ball != LiveBall || deliveryResolved)
            {
                return;
            }

            int timing = Mathf.RoundToInt(quality * 100f);
            LastEvent = $"{shotType} played. Timing {timing}%.";
        }

        public void RegisterGroundContact(CricketBallController ball)
        {
            if (ball == LiveBall && !deliveryResolved && ball.HasBeenHit)
            {
                LastEvent = "Ball along the ground. Fielders are chasing.";
            }
        }

        public void RegisterBoundary(CricketBallController ball, bool isSix)
        {
            if (ball != LiveBall || deliveryResolved || !ball.HasBeenHit)
            {
                return;
            }

            Score += isSix ? 6 : 4;
            ResolveLegalDelivery(isSix ? "Six. Clean lofted hit." : "Four. Beats the fielders.");
        }

        public void RegisterWicket(CricketBallController ball)
        {
            if (ball != LiveBall || deliveryResolved)
            {
                return;
            }

            Wickets++;
            ResolveWicketDelivery("Bowled. The stumps are hit.");
        }

        public void RegisterCaught(CricketBallController ball, string fielderName)
        {
            if (ball != LiveBall || deliveryResolved || ball.HasTouchedGroundAfterHit)
            {
                return;
            }

            Wickets++;
            ResolveWicketDelivery($"Caught by {fielderName}.");
        }

        public void RegisterBallCollected(CricketBallController ball, string fielderName)
        {
            if (ball != LiveBall || deliveryResolved)
            {
                return;
            }

            int runs = EstimateRuns(ball);
            Score += runs;
            string result = runs == 0
                ? $"Dot ball. {fielderName} stops it."
                : $"{runs} run{(runs == 1 ? string.Empty : "s")}. {fielderName} cuts it off.";
            ResolveLegalDelivery(result);
        }

        public void RegisterBallStopped(CricketBallController ball)
        {
            if (ball != LiveBall || deliveryResolved)
            {
                return;
            }

            int runs = EstimateRuns(ball);
            Score += runs;
            string result = runs == 0 ? "Dot ball." : $"{runs} run{(runs == 1 ? string.Empty : "s")}.";
            ResolveLegalDelivery(result);
        }

        public void ResetMatch()
        {
            if (LiveBall != null)
            {
                Destroy(LiveBall.gameObject);
            }

            Score = 0;
            Wickets = 0;
            LegalBalls = 0;
            LiveBall = null;
            IsDeliveryActive = false;
            deliveryResolved = false;
            nextBallReadyAt = Time.time + nextBallDelay;
            batsmanEntry?.ResetEntry();
            LastEvent = "Match reset. Two over game. First two balls are powerplay.";
        }

        private int EstimateRuns(CricketBallController ball)
        {
            if (!ball.HasBeenHit)
            {
                return 0;
            }

            float distance = Vector3.Distance(ball.FirstHitPosition, ball.transform.position);
            if (distance >= 18f)
            {
                return 3;
            }

            if (distance >= 11f)
            {
                return 2;
            }

            if (distance >= 4.5f)
            {
                return 1;
            }

            return 0;
        }

        private void ResolveLegalDelivery(string message)
        {
            deliveryResolved = true;
            IsDeliveryActive = false;
            LegalBalls++;
            LastEvent = IsMatchOver ? $"{message} Match over. Press R to reset." : message;
            nextBallReadyAt = Time.time + nextBallDelay;

            if (LiveBall != null)
            {
                Destroy(LiveBall.gameObject, 1.1f);
            }
        }

        private void ResolveWicketDelivery(string message)
        {
            ResolveLegalDelivery(message);

            if (!IsMatchOver)
            {
                LastEvent = $"{message} Next batsman is walking in.";
                batsmanEntry?.PlayEntry(Wickets);
            }
        }
    }
}
```

### `Assets\Scripts\00_Game\MatchHudController.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public sealed class MatchHudController : MonoBehaviour
    {
        [SerializeField] private CricketGameManager gameManager;

        private GUIStyle titleStyle;
        private GUIStyle bodyStyle;

        public void SetGameManager(CricketGameManager manager)
        {
            gameManager = manager;
        }

        private void OnGUI()
        {
            if (gameManager == null)
            {
                gameManager = FindFirstObjectByType<CricketGameManager>();
            }

            if (gameManager == null || !gameManager.ShowScoreCard)
            {
                return;
            }

            EnsureStyles();

            Rect panel = new Rect(12f, 12f, 370f, 124f);
            GUI.Box(panel, GUIContent.none);
            GUI.Label(new Rect(22f, 20f, 340f, 20f), gameManager.ScoreLine, titleStyle);
            GUI.Label(new Rect(22f, 42f, 340f, 18f), gameManager.LastEvent, bodyStyle);
            GUI.Label(new Rect(22f, 64f, 340f, 18f), "Space Bowl | Q/E Line | Up/Down Pace | R Reset", bodyStyle);
            GUI.Label(new Rect(22f, 84f, 340f, 18f), "J Drive | K Block | L Loft | I Pull | O Cut", bodyStyle);
            GUI.Label(new Rect(22f, 104f, 340f, 18f), "U Sweep | Y Upper Cut | H Helicopter | A/D Place", bodyStyle);
        }

        private void EnsureStyles()
        {
            if (titleStyle != null && bodyStyle != null)
            {
                return;
            }

            titleStyle = new GUIStyle(GUI.skin.label)
            {
                fontSize = 13,
                fontStyle = FontStyle.Bold,
                richText = false
            };

            bodyStyle = new GUIStyle(GUI.skin.label)
            {
                fontSize = 11,
                richText = false
            };
        }
    }
}
```

### `Assets\Scripts\00_Game\ShotType.cs`

```csharp
namespace CricketGame
{
    public enum ShotType
    {
        Drive,
        Block,
        Loft,
        Pull,
        Cut,
        Sweep,
        UpperCut,
        Helicopter
    }
}
```

### `Assets\Scripts\01_Batting\BatContactReporter.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public sealed class BatContactReporter : MonoBehaviour
    {
        [SerializeField] private BatSwingController bat;

        public void SetBat(BatSwingController controller)
        {
            bat = controller;
        }

        private void OnCollisionEnter(Collision collision)
        {
            Report(collision);
        }

        private void OnCollisionStay(Collision collision)
        {
            Report(collision);
        }

        private void Report(Collision collision)
        {
            if (bat == null)
            {
                bat = GetComponentInParent<BatSwingController>();
            }

            CricketBallController ball = collision.collider.GetComponentInParent<CricketBallController>();
            if (ball != null)
            {
                bat?.HandleBallContact(ball, collision);
            }
        }
    }
}
```

### `Assets\Scripts\01_Batting\BatsmanEntryController.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public sealed class BatsmanEntryController : MonoBehaviour
    {
        [SerializeField] private Vector3 startPosition = new Vector3(0f, 0f, -35f);
        [SerializeField] private Vector3 creasePosition = new Vector3(1.5f, 0f, -8.7f);
        [SerializeField] private float walkDuration = 2.8f;
        [SerializeField] private float stayVisibleSeconds = 1.4f;

        private float entryStartedAt;
        private bool isWalking;
        private bool isWaitingAtCrease;

        private void Start()
        {
            ApplyVisibleWalkInRoute();
        }

        public void SetRoute(Vector3 start, Vector3 end, float duration)
        {
            startPosition = start;
            creasePosition = end;
            walkDuration = duration;
            transform.position = startPosition;
            gameObject.SetActive(false);
        }

        public void PlayEntry(int wicketNumber)
        {
            if (isWalking)
            {
                return;
            }

            ApplyVisibleWalkInRoute();
            gameObject.SetActive(true);
            gameObject.name = $"New Batsman {wicketNumber + 1}";
            transform.position = startPosition;
            FaceTowards(creasePosition);
            entryStartedAt = Time.time;
            isWalking = true;
            isWaitingAtCrease = false;
        }

        public void ResetEntry()
        {
            ApplyVisibleWalkInRoute();
            isWalking = false;
            isWaitingAtCrease = false;
            transform.position = startPosition;
            gameObject.SetActive(false);
        }

        private void Update()
        {
            if (isWalking)
            {
                float t = Mathf.Clamp01((Time.time - entryStartedAt) / walkDuration);
                Vector3 position = Vector3.Lerp(startPosition, creasePosition, SmoothStep(t));
                position.y += Mathf.Sin(Time.time * 9f) * 0.035f;
                transform.position = position;
                FaceTowards(creasePosition);

                if (t >= 1f)
                {
                    isWalking = false;
                    isWaitingAtCrease = true;
                    entryStartedAt = Time.time;
                    transform.position = creasePosition;
                }

                return;
            }

            if (isWaitingAtCrease && Time.time - entryStartedAt > stayVisibleSeconds)
            {
                ResetEntry();
            }
        }

        private void FaceTowards(Vector3 target)
        {
            Vector3 lookDirection = target - transform.position;
            lookDirection.y = 0f;

            if (lookDirection.sqrMagnitude > 0.001f)
            {
                transform.rotation = Quaternion.LookRotation(lookDirection.normalized, Vector3.up);
            }
        }

        private static float SmoothStep(float value)
        {
            return value * value * (3f - 2f * value);
        }

        private void ApplyVisibleWalkInRoute()
        {
            startPosition = new Vector3(-6.5f, 0f, -30.8f);
            creasePosition = new Vector3(1.55f, 0f, -8.9f);
            walkDuration = 3.2f;
        }
    }
}
```

### `Assets\Scripts\01_Batting\BatSwingController.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public sealed class BatSwingController : MonoBehaviour
    {
        [Header("References")]
        [SerializeField] private Transform batVisual;
        [SerializeField] private Collider bladeCollider;
        [SerializeField] private Transform frontArm;
        [SerializeField] private Transform backArm;

        [Header("Input")]
        [SerializeField] private KeyCode driveKey = KeyCode.J;
        [SerializeField] private KeyCode blockKey = KeyCode.K;
        [SerializeField] private KeyCode loftKey = KeyCode.L;
        [SerializeField] private KeyCode pullKey = KeyCode.I;
        [SerializeField] private KeyCode cutKey = KeyCode.O;
        [SerializeField] private KeyCode sweepKey = KeyCode.U;
        [SerializeField] private KeyCode upperCutKey = KeyCode.Y;
        [SerializeField] private KeyCode helicopterKey = KeyCode.H;

        [Header("Swing")]
        [SerializeField] private float swingDuration = 0.21f;
        [SerializeField] private float recoveryDuration = 0.27f;
        [SerializeField] private float driveForce = 19f;
        [SerializeField] private float blockForce = 7f;
        [SerializeField] private float loftForce = 23f;
        [SerializeField] private float pullForce = 18f;
        [SerializeField] private float cutForce = 17f;
        [SerializeField] private float sweepForce = 16f;
        [SerializeField] private float upperCutForce = 20f;
        [SerializeField] private float helicopterForce = 24f;

        private const float RestYaw = -52f;
        private const float FollowThroughYaw = 78f;

        private ShotType currentShot = ShotType.Block;
        private float swingStartedAt = -99f;
        private bool isSwinging;
        private bool isRecovering;
        private float lastContactAt = -99f;

        private void Reset()
        {
            batVisual = transform;
            bladeCollider = GetComponentInChildren<Collider>();
        }

        private void Start()
        {
            SetBladeActive(false);
            ApplyBatPose(0f);
        }

        private void Update()
        {
            if (Input.GetKeyDown(driveKey))
            {
                StartSwing(ShotType.Drive);
            }
            else if (Input.GetKeyDown(blockKey))
            {
                StartSwing(ShotType.Block);
            }
            else if (Input.GetKeyDown(loftKey))
            {
                StartSwing(ShotType.Loft);
            }
            else if (Input.GetKeyDown(pullKey))
            {
                StartSwing(ShotType.Pull);
            }
            else if (Input.GetKeyDown(cutKey))
            {
                StartSwing(ShotType.Cut);
            }
            else if (Input.GetKeyDown(sweepKey))
            {
                StartSwing(ShotType.Sweep);
            }
            else if (Input.GetKeyDown(upperCutKey))
            {
                StartSwing(ShotType.UpperCut);
            }
            else if (Input.GetKeyDown(helicopterKey))
            {
                StartSwing(ShotType.Helicopter);
            }

            UpdateSwingAnimation();
        }

        public void StartSwing(ShotType shotType)
        {
            currentShot = shotType;
            swingStartedAt = Time.time;
            isSwinging = true;
            isRecovering = false;
        }

        public void HandleBallContact(CricketBallController ball, Collision collision)
        {
            if (ball == null || ball.HasBeenHit || !IsBladeActive() || Time.time - lastContactAt < 0.1f)
            {
                return;
            }

            lastContactAt = Time.time;
            float quality = EvaluateContactQuality(ball.transform.position);
            ShotType shot = isSwinging ? currentShot : ShotType.Block;
            Vector3 direction = BuildShotDirection(shot);
            float force = GetForceForShot(shot) * Mathf.Lerp(0.55f, 1.18f, quality);
            Vector3 spin = new Vector3(Random.Range(-9f, 9f), Random.Range(-5f, 5f), Random.Range(-12f, 12f));

            ball.MarkHit(shot, direction * force, spin, quality);
        }

        private void UpdateSwingAnimation()
        {
            if (isSwinging)
            {
                float t = Mathf.Clamp01((Time.time - swingStartedAt) / swingDuration);
                ApplyBatPose(SmoothStep(t));
                SetBladeActive(t >= 0.18f && t <= 0.7f);

                if (t >= 1f)
                {
                    isSwinging = false;
                    isRecovering = true;
                    swingStartedAt = Time.time;
                }

                return;
            }

            if (isRecovering)
            {
                float t = Mathf.Clamp01((Time.time - swingStartedAt) / recoveryDuration);
                ApplyBatPose(1f - SmoothStep(t));
                SetBladeActive(false);

                if (t >= 1f)
                {
                    isRecovering = false;
                }

                return;
            }

            SetBladeActive(false);
        }

        private void ApplyBatPose(float swing01)
        {
            Transform target = batVisual == null ? transform : batVisual;
            float yaw = Mathf.Lerp(RestYaw, FollowThroughYaw, swing01);
            float loftLift = currentShot == ShotType.Loft || currentShot == ShotType.UpperCut || currentShot == ShotType.Helicopter ? -24f : -6f;
            if (currentShot == ShotType.Sweep)
            {
                loftLift = Mathf.Lerp(18f, 4f, swing01);
                yaw = Mathf.Lerp(-85f, 95f, swing01);
            }

            float roll = currentShot == ShotType.Helicopter
                ? Mathf.Lerp(-28f, 55f, swing01)
                : Mathf.Lerp(-8f, 14f, swing01);
            target.localRotation = Quaternion.Euler(loftLift, yaw, roll);

            if (frontArm != null)
            {
                frontArm.localRotation = Quaternion.Euler(Mathf.Lerp(12f, -52f, swing01), 0f, Mathf.Lerp(-8f, 22f, swing01));
            }

            if (backArm != null)
            {
                backArm.localRotation = Quaternion.Euler(Mathf.Lerp(-6f, -34f, swing01), 0f, Mathf.Lerp(14f, -28f, swing01));
            }
        }

        public void SetReferences(Transform visual, Collider contactCollider, Transform frontArmPivot, Transform backArmPivot)
        {
            batVisual = visual;
            bladeCollider = contactCollider;
            frontArm = frontArmPivot;
            backArm = backArmPivot;
        }

        private float EvaluateContactQuality(Vector3 ballPosition)
        {
            if (!isSwinging || bladeCollider == null)
            {
                return 0.15f;
            }

            float swing01 = Mathf.Clamp01((Time.time - swingStartedAt) / swingDuration);
            float timingDistance = Mathf.Abs(swing01 - 0.5f);
            float timingQuality = Mathf.Clamp01(1f - timingDistance / 0.5f);
            float sweetSpotDistance = Vector3.Distance(ballPosition, bladeCollider.bounds.ClosestPoint(ballPosition));
            float sweetSpotQuality = Mathf.Clamp01(1f - sweetSpotDistance / 0.42f);
            return Mathf.Clamp01(timingQuality * 0.65f + sweetSpotQuality * 0.35f);
        }

        private Vector3 BuildShotDirection(ShotType shot)
        {
            float placement = 0f;
            if (Input.GetKey(KeyCode.A) || Input.GetKey(KeyCode.LeftArrow))
            {
                placement -= 0.35f;
            }

            if (Input.GetKey(KeyCode.D) || Input.GetKey(KeyCode.RightArrow))
            {
                placement += 0.35f;
            }

            Vector3 direction;
            switch (shot)
            {
                case ShotType.Block:
                    direction = new Vector3(placement * 0.3f, 0.1f, 1f);
                    break;
                case ShotType.Loft:
                    direction = new Vector3(placement * 0.55f, 0.92f, 1f);
                    break;
                case ShotType.Pull:
                    direction = new Vector3(-0.85f + placement, 0.25f, 0.55f);
                    break;
                case ShotType.Cut:
                    direction = new Vector3(0.85f + placement, 0.2f, 0.55f);
                    break;
                case ShotType.Sweep:
                    direction = new Vector3(-1f + placement, 0.16f, 0.25f);
                    break;
                case ShotType.UpperCut:
                    direction = new Vector3(0.75f + placement, 0.78f, 0.45f);
                    break;
                case ShotType.Helicopter:
                    direction = new Vector3(placement * 0.4f, 0.82f, 1f);
                    break;
                default:
                    direction = new Vector3(placement, 0.3f, 1f);
                    break;
            }

            return direction.normalized;
        }

        private float GetForceForShot(ShotType shot)
        {
            switch (shot)
            {
                case ShotType.Block:
                    return blockForce;
                case ShotType.Loft:
                    return loftForce;
                case ShotType.Pull:
                    return pullForce;
                case ShotType.Cut:
                    return cutForce;
                case ShotType.Sweep:
                    return sweepForce;
                case ShotType.UpperCut:
                    return upperCutForce;
                case ShotType.Helicopter:
                    return helicopterForce;
                default:
                    return driveForce;
            }
        }

        private static float SmoothStep(float value)
        {
            return value * value * (3f - 2f * value);
        }

        private bool IsBladeActive()
        {
            return bladeCollider != null && bladeCollider.enabled;
        }

        private void SetBladeActive(bool active)
        {
            if (bladeCollider != null)
            {
                bladeCollider.enabled = active;
            }
        }
    }
}
```

### `Assets\Scripts\02_Bowling\BowlerController.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public sealed class BowlerController : MonoBehaviour
    {
        [Header("References")]
        [SerializeField] private CricketGameManager gameManager;
        [SerializeField] private CricketBallController ballPrefab;
        [SerializeField] private Transform releasePoint;
        [SerializeField] private Transform targetPoint;

        [Header("Delivery")]
        [SerializeField] private float flightTime = 0.95f;
        [SerializeField] private float lineOffset;
        [SerializeField] private float maxLineOffset = 0.55f;
        [SerializeField] private float lineAdjustSpeed = 1.4f;
        [SerializeField] private float paceAdjust = 0f;
        [SerializeField] private float maxPaceAdjust = 0.18f;

        public bool CanBowl => gameManager == null || (!gameManager.IsDeliveryActive && !gameManager.IsMatchOver);
        public float LineOffset => lineOffset;
        public float PaceAdjust => paceAdjust;

        private void Update()
        {
            float lineInput = 0f;
            if (Input.GetKey(KeyCode.Q))
            {
                lineInput -= 1f;
            }

            if (Input.GetKey(KeyCode.E))
            {
                lineInput += 1f;
            }

            lineOffset = Mathf.Clamp(lineOffset + lineInput * lineAdjustSpeed * Time.deltaTime, -maxLineOffset, maxLineOffset);

            if (Input.GetKey(KeyCode.UpArrow))
            {
                paceAdjust = Mathf.Clamp(paceAdjust + Time.deltaTime * 0.08f, -maxPaceAdjust, maxPaceAdjust);
            }
            else if (Input.GetKey(KeyCode.DownArrow))
            {
                paceAdjust = Mathf.Clamp(paceAdjust - Time.deltaTime * 0.08f, -maxPaceAdjust, maxPaceAdjust);
            }

            if (Input.GetKeyDown(KeyCode.Space))
            {
                Bowl();
            }
        }

        public bool Bowl()
        {
            if (!CanBowl || ballPrefab == null || releasePoint == null || targetPoint == null)
            {
                return false;
            }

            CricketBallController ball = Instantiate(ballPrefab, releasePoint.position, Quaternion.identity);
            ball.gameObject.SetActive(true);
            ball.Prepare(gameManager);
            gameManager?.RegisterDelivery(ball);

            Vector3 target = targetPoint.position + Vector3.right * lineOffset;
            float adjustedFlightTime = Mathf.Clamp(flightTime - paceAdjust, 0.68f, 1.25f);
            Vector3 velocity = CalculateBallisticVelocity(releasePoint.position, target, adjustedFlightTime);
            Vector3 spin = new Vector3(Random.Range(-6f, 6f), Random.Range(-10f, 10f), Random.Range(-4f, 4f));
            ball.Launch(velocity, spin);
            return true;
        }

        public void SetReferences(CricketGameManager manager, CricketBallController prefab, Transform release, Transform target)
        {
            gameManager = manager;
            ballPrefab = prefab;
            releasePoint = release;
            targetPoint = target;
        }

        private static Vector3 CalculateBallisticVelocity(Vector3 origin, Vector3 target, float time)
        {
            Vector3 displacement = target - origin;
            return displacement / time - Physics.gravity * (time * 0.5f);
        }
    }
}
```

### `Assets\Scripts\03_Fielding\BoundaryZone.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    [RequireComponent(typeof(Collider))]
    public sealed class BoundaryZone : MonoBehaviour
    {
        [SerializeField] private CricketGameManager gameManager;

        private void Reset()
        {
            GetComponent<Collider>().isTrigger = true;
        }

        public void SetGameManager(CricketGameManager manager)
        {
            gameManager = manager;
        }

        private void OnTriggerEnter(Collider other)
        {
            CricketBallController ball = other.GetComponentInParent<CricketBallController>();
            if (ball == null || !ball.HasBeenHit)
            {
                return;
            }

            if (gameManager == null)
            {
                gameManager = FindFirstObjectByType<CricketGameManager>();
            }

            bool isSix = !ball.HasTouchedGroundAfterHit;
            gameManager?.RegisterBoundary(ball, isSix);
        }
    }
}
```

### `Assets\Scripts\03_Fielding\FielderController.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public sealed class FielderController : MonoBehaviour
    {
        [SerializeField] private CricketGameManager gameManager;
        [SerializeField] private string fielderName = "Fielder";
        [SerializeField] private float moveSpeed = 4.2f;
        [SerializeField] private float returnSpeed = 2.8f;
        [SerializeField] private float reactionDelay = 0.28f;
        [SerializeField] private float catchRadius = 1.25f;
        [SerializeField] private float stopRadius = 1.6f;
        [SerializeField] private float catchHeight = 2.2f;

        private CricketBallController trackedBall;
        private float chaseStartsAt;
        private Vector3 homePosition;
        private Vector3 powerplayHomePosition;
        private Vector3 normalHomePosition;
        private bool hasFieldPlan;

        public void SetGameManager(CricketGameManager manager, string displayName)
        {
            gameManager = manager;
            fielderName = displayName;
        }

        public void SetFieldPlan(CricketGameManager manager, string displayName, Vector3 powerplayHome, Vector3 normalHome)
        {
            gameManager = manager;
            fielderName = displayName;
            powerplayHomePosition = powerplayHome;
            normalHomePosition = normalHome;
            homePosition = powerplayHome;
            hasFieldPlan = true;
            transform.position = powerplayHome;
        }

        private void Start()
        {
            if (!hasFieldPlan)
            {
                homePosition = transform.position;
                powerplayHomePosition = homePosition;
                normalHomePosition = homePosition;
            }
        }

        private void Update()
        {
            if (gameManager == null)
            {
                gameManager = FindFirstObjectByType<CricketGameManager>();
            }

            if (gameManager == null)
            {
                return;
            }

            UpdateHomePosition();

            CricketBallController ball = gameManager.LiveBall;
            if (ball == null || !gameManager.IsDeliveryActive || !ball.HasBeenHit)
            {
                ReturnHome();
                trackedBall = null;
                return;
            }

            if (trackedBall != ball)
            {
                trackedBall = ball;
                chaseStartsAt = Time.time + reactionDelay;
            }

            if (Time.time < chaseStartsAt)
            {
                return;
            }

            MoveTowardsBall(ball);
            TryResolveBall(ball);
        }

        private void MoveTowardsBall(CricketBallController ball)
        {
            Vector3 target = ball.transform.position;
            Vector3 chasePosition = new Vector3(target.x, transform.position.y, target.z);
            transform.position = Vector3.MoveTowards(transform.position, chasePosition, moveSpeed * Time.deltaTime);

            Vector3 lookDirection = chasePosition - transform.position;
            if (lookDirection.sqrMagnitude > 0.001f)
            {
                transform.rotation = Quaternion.Slerp(
                    transform.rotation,
                    Quaternion.LookRotation(lookDirection.normalized, Vector3.up),
                    Time.deltaTime * 8f);
            }
        }

        private void TryResolveBall(CricketBallController ball)
        {
            float distance = Vector3.Distance(transform.position, ball.transform.position);

            if (!ball.HasTouchedGroundAfterHit && ball.transform.position.y <= catchHeight && distance <= catchRadius)
            {
                ball.CollectByFielder(transform.position + Vector3.up * 1.35f);
                gameManager.RegisterCaught(ball, fielderName);
                return;
            }

            if ((ball.HasTouchedGroundAfterHit || ball.transform.position.y <= 0.9f) && distance <= stopRadius)
            {
                ball.CollectByFielder(transform.position + Vector3.up * 0.65f);
                gameManager.RegisterBallCollected(ball, fielderName);
            }
        }

        private void ReturnHome()
        {
            transform.position = Vector3.MoveTowards(transform.position, homePosition, returnSpeed * Time.deltaTime);
            Vector3 lookDirection = homePosition - transform.position;
            if (lookDirection.sqrMagnitude > 0.05f)
            {
                transform.rotation = Quaternion.Slerp(
                    transform.rotation,
                    Quaternion.LookRotation(lookDirection.normalized, Vector3.up),
                    Time.deltaTime * 5f);
            }
        }

        private void UpdateHomePosition()
        {
            if (!hasFieldPlan)
            {
                return;
            }

            homePosition = gameManager.IsPowerplay ? powerplayHomePosition : normalHomePosition;
        }
    }
}
```

### `Assets\Scripts\03_Fielding\GroundSurface.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public sealed class GroundSurface : MonoBehaviour
    {
    }
}
```

### `Assets\Scripts\03_Fielding\Wicket.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public sealed class Wicket : MonoBehaviour
    {
        [SerializeField] private CricketGameManager gameManager;

        private Transform[] bails;
        private Vector3[] bailLocalPositions;
        private Quaternion[] bailLocalRotations;
        private bool isFallen;

        private void Start()
        {
            CacheBails();
        }

        public void SetGameManager(CricketGameManager manager)
        {
            gameManager = manager;
        }

        public void HitBy(CricketBallController ball)
        {
            if (gameManager == null)
            {
                gameManager = FindFirstObjectByType<CricketGameManager>();
            }

            KnockBails(ball == null ? Vector3.forward : ball.transform.forward);
            gameManager?.RegisterWicket(ball);
        }

        private void CacheBails()
        {
            if (bails != null)
            {
                return;
            }

            System.Collections.Generic.List<Transform> foundBails = new System.Collections.Generic.List<Transform>();
            foreach (Transform child in transform)
            {
                if (child.name.StartsWith("Bail"))
                {
                    foundBails.Add(child);
                }
            }

            bails = foundBails.ToArray();
            bailLocalPositions = new Vector3[bails.Length];
            bailLocalRotations = new Quaternion[bails.Length];

            for (int index = 0; index < bails.Length; index++)
            {
                bailLocalPositions[index] = bails[index].localPosition;
                bailLocalRotations[index] = bails[index].localRotation;
            }
        }

        private void KnockBails(Vector3 ballDirection)
        {
            if (isFallen)
            {
                return;
            }

            CacheBails();
            isFallen = true;

            for (int index = 0; index < bails.Length; index++)
            {
                Transform bail = bails[index];
                Rigidbody body = bail.GetComponent<Rigidbody>();
                if (body == null)
                {
                    body = bail.gameObject.AddComponent<Rigidbody>();
                }

                body.mass = 0.04f;
                body.isKinematic = false;
                body.linearVelocity = Vector3.zero;
                body.angularVelocity = Vector3.zero;
                Vector3 launch = (ballDirection.normalized + Vector3.up * 1.4f + Vector3.right * (index == 0 ? -0.5f : 0.5f)).normalized;
                body.AddForce(launch * 2.2f, ForceMode.Impulse);
                body.AddTorque(Random.insideUnitSphere * 1.2f, ForceMode.Impulse);
            }

            Invoke(nameof(ResetBails), 1.6f);
        }

        private void ResetBails()
        {
            CacheBails();

            for (int index = 0; index < bails.Length; index++)
            {
                Transform bail = bails[index];
                Rigidbody body = bail.GetComponent<Rigidbody>();
                if (body != null)
                {
                    body.isKinematic = true;
                    body.linearVelocity = Vector3.zero;
                    body.angularVelocity = Vector3.zero;
                }

                bail.SetParent(transform);
                bail.localPosition = bailLocalPositions[index];
                bail.localRotation = bailLocalRotations[index];
            }

            isFallen = false;
        }
    }
}
```

### `Assets\Scripts\04_Ball\CricketBallController.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    [RequireComponent(typeof(Rigidbody))]
    [RequireComponent(typeof(SphereCollider))]
    public sealed class CricketBallController : MonoBehaviour
    {
        [Header("Dead ball")]
        [SerializeField] private float minimumLiveSpeed = 0.35f;
        [SerializeField] private float stoppedSecondsToEndBall = 0.8f;
        [SerializeField] private float maxLifetimeSeconds = 12f;

        [Header("Bowled check")]
        [SerializeField] private float battingWicketZ = -9.25f;
        [SerializeField] private float wicketHalfWidth = 0.38f;
        [SerializeField] private float wicketMaxHeight = 1.15f;

        private Rigidbody body;
        private CricketGameManager gameManager;
        private float stoppedTimer;
        private float spawnedAt;
        private bool hasResolvedStop;
        private bool isCollected;
        private bool hasCheckedBowledZone;

        public bool HasBeenHit { get; private set; }
        public bool HasTouchedGroundAfterHit { get; private set; }
        public Vector3 FirstHitPosition { get; private set; }
        public ShotType LastShotType { get; private set; }

        private void Awake()
        {
            body = GetComponent<Rigidbody>();
            body.collisionDetectionMode = CollisionDetectionMode.ContinuousDynamic;
            body.interpolation = RigidbodyInterpolation.Interpolate;
        }

        public void Prepare(CricketGameManager manager)
        {
            gameManager = manager;
            spawnedAt = Time.time;
            stoppedTimer = 0f;
            hasResolvedStop = false;
            isCollected = false;
            hasCheckedBowledZone = false;
            HasBeenHit = false;
            HasTouchedGroundAfterHit = false;
            FirstHitPosition = transform.position;
            body.isKinematic = false;
        }

        public void Launch(Vector3 velocity, Vector3 angularVelocity)
        {
            if (body == null)
            {
                body = GetComponent<Rigidbody>();
            }

            spawnedAt = Time.time;
            body.linearVelocity = velocity;
            body.angularVelocity = angularVelocity;
        }

        public void MarkHit(ShotType shotType, Vector3 launchVelocity, Vector3 angularVelocity, float quality)
        {
            if (HasBeenHit)
            {
                return;
            }

            HasBeenHit = true;
            HasTouchedGroundAfterHit = false;
            FirstHitPosition = transform.position;
            LastShotType = shotType;
            body.linearVelocity = launchVelocity;
            body.angularVelocity = angularVelocity;
            gameManager?.RegisterHit(this, shotType, quality);
        }

        public void CollectByFielder(Vector3 holdPosition)
        {
            if (body == null)
            {
                body = GetComponent<Rigidbody>();
            }

            isCollected = true;
            hasResolvedStop = true;
            body.isKinematic = true;
            body.linearVelocity = Vector3.zero;
            body.angularVelocity = Vector3.zero;
            transform.position = holdPosition;
        }

        private void FixedUpdate()
        {
            if (hasResolvedStop || isCollected)
            {
                return;
            }

            if (!HasBeenHit)
            {
                TryResolveBowledByZone();
                if (hasResolvedStop)
                {
                    return;
                }
            }

            if (Time.time - spawnedAt > maxLifetimeSeconds)
            {
                ResolveStopped();
                return;
            }

            bool canEndBySpeed = HasBeenHit || Time.time - spawnedAt > 1.25f;
            if (!canEndBySpeed)
            {
                return;
            }

            if (body.linearVelocity.magnitude <= minimumLiveSpeed)
            {
                stoppedTimer += Time.fixedDeltaTime;
            }
            else
            {
                stoppedTimer = 0f;
            }

            if (stoppedTimer >= stoppedSecondsToEndBall)
            {
                ResolveStopped();
            }
        }

        private void ResolveStopped()
        {
            hasResolvedStop = true;
            gameManager?.RegisterBallStopped(this);
        }

        private void TryResolveBowledByZone()
        {
            if (hasCheckedBowledZone || transform.position.z > battingWicketZ)
            {
                return;
            }

            hasCheckedBowledZone = true;
            bool isInWicketLine = Mathf.Abs(transform.position.x) <= wicketHalfWidth && transform.position.y <= wicketMaxHeight;
            if (!isInWicketLine)
            {
                return;
            }

            Wicket battingWicket = FindNearestWicketToBattingEnd();
            if (battingWicket != null)
            {
                battingWicket.HitBy(this);
            }
            else
            {
                gameManager?.RegisterWicket(this);
            }

            hasResolvedStop = true;
        }

        private Wicket FindNearestWicketToBattingEnd()
        {
            Wicket[] wickets = FindObjectsByType<Wicket>(FindObjectsSortMode.None);
            Wicket nearestWicket = null;
            float nearestDistance = float.MaxValue;
            Vector3 battingEnd = new Vector3(0f, 0f, battingWicketZ);

            foreach (Wicket wicket in wickets)
            {
                float distance = (wicket.transform.position - battingEnd).sqrMagnitude;
                if (distance < nearestDistance)
                {
                    nearestDistance = distance;
                    nearestWicket = wicket;
                }
            }

            return nearestWicket;
        }

        private void OnCollisionEnter(Collision collision)
        {
            Wicket wicket = collision.collider.GetComponentInParent<Wicket>();
            if (wicket != null)
            {
                wicket.HitBy(this);
                hasResolvedStop = true;
                return;
            }

            GroundSurface ground = collision.collider.GetComponentInParent<GroundSurface>();
            if (ground != null && HasBeenHit)
            {
                HasTouchedGroundAfterHit = true;
                gameManager?.RegisterGroundContact(this);
            }
        }
    }
}
```

### `Assets\Scripts\05_Camera\CricketCameraController.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public sealed class CricketCameraController : MonoBehaviour
    {
        [SerializeField] private CricketGameManager gameManager;
        [SerializeField] private Transform battingView;
        [SerializeField] private Vector3 liveBallOffset = new Vector3(0f, 2.4f, -5.5f);
        [SerializeField] private float positionSmooth = 4.5f;
        [SerializeField] private float rotationSmooth = 7f;
        [SerializeField] private float extraBallFollowTime = 1.3f;

        private CricketBallController lastBall;
        private float followUntil;

        private void Start()
        {
            ApplyWideDemoView();
        }

        private void LateUpdate()
        {
            if (gameManager == null)
            {
                gameManager = FindFirstObjectByType<CricketGameManager>();
            }

            CricketBallController liveBall = gameManager == null ? null : gameManager.LiveBall;
            if (liveBall != null)
            {
                if (liveBall != lastBall)
                {
                    lastBall = liveBall;
                    followUntil = Time.time + extraBallFollowTime;
                }

                if (gameManager.IsDeliveryActive || Time.time <= followUntil)
                {
                    Vector3 desiredPosition = liveBall.transform.position + liveBallOffset;
                    MoveAndLookAt(desiredPosition, liveBall.transform.position + Vector3.up * 0.25f);
                    return;
                }
            }

            if (battingView != null)
            {
                MoveAndLookAt(battingView.position, battingView.position + battingView.forward * 8f);
            }
        }

        public void SetReferences(CricketGameManager manager, Transform view)
        {
            gameManager = manager;
            battingView = view;
            ApplyWideDemoView();
        }

        private void MoveAndLookAt(Vector3 desiredPosition, Vector3 lookTarget)
        {
            transform.position = Vector3.Lerp(transform.position, desiredPosition, Time.deltaTime * positionSmooth);
            Quaternion desiredRotation = Quaternion.LookRotation(lookTarget - transform.position, Vector3.up);
            transform.rotation = Quaternion.Slerp(transform.rotation, desiredRotation, Time.deltaTime * rotationSmooth);
        }

        private void ApplyWideDemoView()
        {
            Vector3 cameraPosition = new Vector3(0f, 8.2f, -24f);
            Vector3 lookTarget = new Vector3(0f, 1.25f, -1f);
            Quaternion cameraRotation = Quaternion.LookRotation(lookTarget - cameraPosition, Vector3.up);

            if (battingView != null)
            {
                battingView.position = cameraPosition;
                battingView.rotation = cameraRotation;
            }

            Camera camera = GetComponent<Camera>();
            if (camera != null)
            {
                camera.fieldOfView = 68f;
                camera.nearClipPlane = 0.08f;
                camera.farClipPlane = 160f;
            }

            transform.position = cameraPosition;
            transform.rotation = cameraRotation;
        }
    }
}
```

### `Assets\Scripts\06_Visuals\CricketMaterialNotes.cs`

```csharp
namespace CricketGame
{
    public static class CricketMaterialNotes
    {
        public const string Summary =
            "This file documents the color/material plan used in the project.";

        public const string Ground = "Ground_Grass is used for the outfield.";
        public const string Pitch = "Pitch_Clay is used for the cricket pitch.";
        public const string Lines = "Line_White is used for crease lines and boundary rope.";
        public const string Bat = "Bat_Wood is used for the bat and wickets.";
        public const string Ball = "Ball_Red is used for the cricket ball.";
        public const string Players = "Player_Blue is used for batsman, bowler, and fielders.";
        public const string Umpire = "Umpire_Black is used for the umpire.";
        public const string Stadium = "Wall_Maroon, Seat_Blue, Seat_Yellow, Floodlight_Cyan, Scoreboard_Black, Tree_Green, Building_Grey, Window_Blue, and Board_Purple are used for the simple stadium design.";
    }
}
```

### `Assets\Scripts\06_Visuals\CricketVisualPalette.cs`

```csharp
using UnityEngine;

namespace CricketGame
{
    public static class CricketVisualPalette
    {
        public const string GrassMaterialName = "Ground_Grass";
        public const string PitchMaterialName = "Pitch_Clay";
        public const string LineMaterialName = "Line_White";
        public const string BatMaterialName = "Bat_Wood";
        public const string BallMaterialName = "Ball_Red";
        public const string PlayerMaterialName = "Player_Blue";
        public const string TrimMaterialName = "Trim_White";
        public const string UmpireMaterialName = "Umpire_Black";
        public const string WallMaterialName = "Wall_Maroon";
        public const string SeatBlueMaterialName = "Seat_Blue";
        public const string SeatYellowMaterialName = "Seat_Yellow";
        public const string FloodlightMaterialName = "Floodlight_Cyan";
        public const string ScoreboardMaterialName = "Scoreboard_Black";
        public const string TreeMaterialName = "Tree_Green";
        public const string BuildingMaterialName = "Building_Grey";
        public const string WindowMaterialName = "Window_Blue";
        public const string BoardMaterialName = "Board_Purple";

        public static readonly Color GrassColor = new Color(0.16f, 0.48f, 0.2f);
        public static readonly Color PitchColor = new Color(0.66f, 0.49f, 0.29f);
        public static readonly Color LineColor = new Color(0.94f, 0.93f, 0.88f);
        public static readonly Color BatColor = new Color(0.74f, 0.57f, 0.36f);
        public static readonly Color BallColor = new Color(0.55f, 0.04f, 0.05f);
        public static readonly Color PlayerColor = new Color(0.12f, 0.2f, 0.42f);
        public static readonly Color TrimColor = new Color(0.87f, 0.89f, 0.92f);
        public static readonly Color UmpireColor = new Color(0.1f, 0.1f, 0.12f);
        public static readonly Color WallColor = new Color(0.47f, 0.07f, 0.32f);
        public static readonly Color SeatBlueColor = new Color(0.08f, 0.19f, 0.45f);
        public static readonly Color SeatYellowColor = new Color(0.95f, 0.79f, 0.08f);
        public static readonly Color FloodlightColor = new Color(0.05f, 0.78f, 0.82f);
        public static readonly Color ScoreboardColor = new Color(0.02f, 0.02f, 0.025f);
        public static readonly Color TreeColor = new Color(0.08f, 0.43f, 0.12f);
        public static readonly Color BuildingColor = new Color(0.62f, 0.64f, 0.66f);
        public static readonly Color WindowColor = new Color(0.22f, 0.55f, 0.78f);
        public static readonly Color BoardColor = new Color(0.36f, 0.05f, 0.5f);
    }
}
```

## Editor Scene Builder Scripts

### `Assets\Editor\SceneBuilder\CricketMaterialFactory.cs`

```csharp
using CricketGame;
using UnityEditor;
using UnityEngine;

namespace CricketGame.EditorTools
{
    public sealed class CricketMaterialFactory
    {
        public Material Grass => CreateMaterial(CricketVisualPalette.GrassMaterialName, CricketVisualPalette.GrassColor);
        public Material Pitch => CreateMaterial(CricketVisualPalette.PitchMaterialName, CricketVisualPalette.PitchColor);
        public Material Line => CreateMaterial(CricketVisualPalette.LineMaterialName, CricketVisualPalette.LineColor);
        public Material Bat => CreateMaterial(CricketVisualPalette.BatMaterialName, CricketVisualPalette.BatColor);
        public Material Ball => CreateMaterial(CricketVisualPalette.BallMaterialName, CricketVisualPalette.BallColor);
        public Material Player => CreateMaterial(CricketVisualPalette.PlayerMaterialName, CricketVisualPalette.PlayerColor);
        public Material Trim => CreateMaterial(CricketVisualPalette.TrimMaterialName, CricketVisualPalette.TrimColor);
        public Material Umpire => CreateMaterial(CricketVisualPalette.UmpireMaterialName, CricketVisualPalette.UmpireColor);
        public Material Wall => CreateMaterial(CricketVisualPalette.WallMaterialName, CricketVisualPalette.WallColor);
        public Material SeatBlue => CreateMaterial(CricketVisualPalette.SeatBlueMaterialName, CricketVisualPalette.SeatBlueColor);
        public Material SeatYellow => CreateMaterial(CricketVisualPalette.SeatYellowMaterialName, CricketVisualPalette.SeatYellowColor);
        public Material Floodlight => CreateMaterial(CricketVisualPalette.FloodlightMaterialName, CricketVisualPalette.FloodlightColor);
        public Material Scoreboard => CreateMaterial(CricketVisualPalette.ScoreboardMaterialName, CricketVisualPalette.ScoreboardColor);
        public Material Tree => CreateMaterial(CricketVisualPalette.TreeMaterialName, CricketVisualPalette.TreeColor);
        public Material Building => CreateMaterial(CricketVisualPalette.BuildingMaterialName, CricketVisualPalette.BuildingColor);
        public Material Window => CreateMaterial(CricketVisualPalette.WindowMaterialName, CricketVisualPalette.WindowColor);
        public Material Board => CreateMaterial(CricketVisualPalette.BoardMaterialName, CricketVisualPalette.BoardColor);

        public PhysicsMaterial BallPhysics => CreatePhysicsMaterial("Ball_Physics", 0.72f, 0.35f);

        private static Material CreateMaterial(string name, Color color)
        {
            string path = $"Assets/Materials/{name}.mat";
            Material material = AssetDatabase.LoadAssetAtPath<Material>(path);
            if (material != null)
            {
                material.color = color;
                EditorUtility.SetDirty(material);
                return material;
            }

            Shader shader = Shader.Find("Universal Render Pipeline/Lit");
            if (shader == null)
            {
                shader = Shader.Find("Standard");
            }

            material = new Material(shader)
            {
                name = name,
                color = color
            };

            AssetDatabase.CreateAsset(material, path);
            return material;
        }

        private static PhysicsMaterial CreatePhysicsMaterial(string name, float bounciness, float friction)
        {
            string path = $"Assets/Materials/{name}.asset";
            PhysicsMaterial material = AssetDatabase.LoadAssetAtPath<PhysicsMaterial>(path);
            if (material != null)
            {
                material.bounciness = bounciness;
                material.dynamicFriction = friction;
                material.staticFriction = friction;
                EditorUtility.SetDirty(material);
                return material;
            }

            material = new PhysicsMaterial(name)
            {
                bounciness = bounciness,
                dynamicFriction = friction,
                staticFriction = friction,
                bounceCombine = PhysicsMaterialCombine.Maximum,
                frictionCombine = PhysicsMaterialCombine.Average
            };

            AssetDatabase.CreateAsset(material, path);
            return material;
        }
    }
}
```

### `Assets\Editor\SceneBuilder\CricketMaterialMenu.cs`

```csharp
using UnityEditor;
using UnityEngine;

namespace CricketGame.EditorTools
{
    public static class CricketMaterialMenu
    {
        [MenuItem("Cricket Prototype/Create Materials Only")]
        public static void CreateMaterialsOnly()
        {
            if (!AssetDatabase.IsValidFolder("Assets/Materials"))
            {
                AssetDatabase.CreateFolder("Assets", "Materials");
            }

            CricketMaterialFactory factory = new CricketMaterialFactory();
            _ = factory.Grass;
            _ = factory.Pitch;
            _ = factory.Line;
            _ = factory.Bat;
            _ = factory.Ball;
            _ = factory.Player;
            _ = factory.Trim;
            _ = factory.Umpire;
            _ = factory.Wall;
            _ = factory.SeatBlue;
            _ = factory.SeatYellow;
            _ = factory.Floodlight;
            _ = factory.Scoreboard;
            _ = factory.Tree;
            _ = factory.Building;
            _ = factory.Window;
            _ = factory.Board;
            _ = factory.BallPhysics;

            AssetDatabase.SaveAssets();
            AssetDatabase.Refresh();
            Debug.Log("Cricket prototype materials created in Assets/Materials.");
        }
    }
}
```

### `Assets\Editor\SceneBuilder\CricketPrototypeSceneBuilder.cs`

```csharp
using System.IO;
using CricketGame;
using UnityEditor;
using UnityEditor.SceneManagement;
using UnityEngine;
using UnityEngine.SceneManagement;

namespace CricketGame.EditorTools
{
    public static class CricketPrototypeSceneBuilder
    {
        private const string ScenePath = "Assets/Scenes/CricketPrototype.unity";
        private const string DesignScenePath = "Assets/Scenes/MyCricketStadiumDesign.unity";
        private const string FinalScenePath = "Assets/Scenes/Cricket_50_Marks_Final.unity";
        private const string FinalSyncMarkerPath = "ProjectSettings/CricketFinalSync_20260506_force_v1.txt";
        private static readonly string[] PlayableScenePaths =
        {
            ScenePath,
            DesignScenePath,
            FinalScenePath
        };

        [InitializeOnLoadMethod]
        private static void BuildSceneIfMissing()
        {
            if (File.Exists(ScenePath))
            {
                return;
            }

            EditorApplication.delayCall += () =>
            {
                if (EditorApplication.isCompiling || EditorApplication.isPlayingOrWillChangePlaymode || File.Exists(ScenePath))
                {
                    return;
                }

                BuildStarterScene();
            };
        }

        [InitializeOnLoadMethod]
        private static void SyncLatestFinalSceneOnce()
        {
            if (File.Exists(FinalSyncMarkerPath))
            {
                return;
            }

            EditorApplication.delayCall += TrySyncLatestFinalSceneOnce;
        }

        private static void TrySyncLatestFinalSceneOnce()
        {
            if (File.Exists(FinalSyncMarkerPath))
            {
                return;
            }

            if (EditorApplication.isCompiling || EditorApplication.isPlayingOrWillChangePlaymode)
            {
                EditorApplication.delayCall += TrySyncLatestFinalSceneOnce;
                return;
            }

            try
            {
                RebuildAllPlayableScenes();
                Directory.CreateDirectory("ProjectSettings");
                File.WriteAllText(FinalSyncMarkerPath, "Synced latest 50 marks cricket scene update on 2026-05-05.");
            }
            catch (System.Exception exception)
            {
                Debug.LogError($"Could not auto-sync cricket final scene: {exception.Message}");
            }
        }

        [MenuItem("Cricket Prototype/Build Starter Scene")]
        public static void BuildStarterScene()
        {
            if (File.Exists(ScenePath))
            {
                bool shouldOverwrite = Application.isBatchMode || EditorUtility.DisplayDialog(
                    "Overwrite starter scene?",
                    "Build Starter Scene recreates CricketPrototype.unity. If you changed the design, make a design copy first or your edits can be replaced.",
                    "Backup and Rebuild",
                    "Cancel");

                if (!shouldOverwrite)
                {
                    Debug.Log("Starter scene rebuild cancelled. Your current scene was not changed.");
                    return;
                }

                BackupAsset(ScenePath, "StarterSceneBackup");
            }

            BuildSceneAtPath(ScenePath);
            Debug.Log("Cricket prototype scene built. Open Assets/Scenes/CricketPrototype and press Play.");
        }

        [MenuItem("Cricket Prototype/Build 50 Marks Final Scene")]
        public static void Build50MarksFinalScene()
        {
            if (File.Exists(FinalScenePath))
            {
                BackupAsset(FinalScenePath, "FinalSceneBackup");
            }

            BuildSceneAtPath(FinalScenePath);
            Selection.activeObject = AssetDatabase.LoadAssetAtPath<SceneAsset>(FinalScenePath);
            Debug.Log("50 marks final scene built. Open Assets/Scenes/Cricket_50_Marks_Final and press Play.");
        }

        [MenuItem("Cricket Prototype/Rebuild All Playable Scenes")]
        public static void RebuildAllPlayableScenes()
        {
            foreach (string scenePath in PlayableScenePaths)
            {
                if (File.Exists(scenePath))
                {
                    BackupAsset(scenePath, "BeforeSync");
                }

                BuildSceneAtPath(scenePath);
            }

            EditorSceneManager.OpenScene(FinalScenePath, OpenSceneMode.Single);
            Selection.activeObject = AssetDatabase.LoadAssetAtPath<SceneAsset>(FinalScenePath);
            Debug.Log("All playable cricket scenes were rebuilt with the latest stadium, wicket, and batsman-entry updates. Press Play in Cricket_50_Marks_Final.");
        }

        [MenuItem("Cricket Prototype/FORCE APPLY MAY 06 UPDATES")]
        public static void ForceApplyMay06Updates()
        {
            RebuildAllPlayableScenes();
            Debug.Log("FORCE APPLY complete: oval field, closer lights, 2 overs, powerplay, and extra shots are now in the open final scene.");
        }

        [MenuItem("Cricket Prototype/Open 50 Marks Final Scene")]
        public static void Open50MarksFinalScene()
        {
            if (!File.Exists(FinalScenePath))
            {
                Build50MarksFinalScene();
            }

            EditorSceneManager.OpenScene(FinalScenePath, OpenSceneMode.Single);
            Selection.activeObject = AssetDatabase.LoadAssetAtPath<SceneAsset>(FinalScenePath);
            Debug.Log("Opened Assets/Scenes/Cricket_50_Marks_Final.unity. Press Play to test the upgraded version.");
        }

        [MenuItem("Cricket Prototype/Make Editable Design Copy")]
        public static void MakeEditableDesignCopy()
        {
            Directory.CreateDirectory("Assets/Scenes");

            if (!File.Exists(ScenePath))
            {
                BuildStarterScene();
            }

            if (!File.Exists(ScenePath))
            {
                Debug.LogWarning("Could not create design copy because the starter scene does not exist.");
                return;
            }

            if (File.Exists(DesignScenePath))
            {
                bool shouldReplace = EditorUtility.DisplayDialog(
                    "Replace design copy?",
                    "MyCricketStadiumDesign.unity already exists. Replace it with a fresh copy of the starter scene?",
                    "Replace Copy",
                    "Keep Existing");

                if (!shouldReplace)
                {
                    Selection.activeObject = AssetDatabase.LoadAssetAtPath<SceneAsset>(DesignScenePath);
                    Debug.Log("Existing design scene kept. Open Assets/Scenes/MyCricketStadiumDesign to continue editing.");
                    return;
                }

                BackupAsset(DesignScenePath, "DesignSceneBackup");
                AssetDatabase.DeleteAsset(DesignScenePath);
            }

            AssetDatabase.CopyAsset(ScenePath, DesignScenePath);
            AssetDatabase.SaveAssets();
            AssetDatabase.Refresh();

            Selection.activeObject = AssetDatabase.LoadAssetAtPath<SceneAsset>(DesignScenePath);
            Debug.Log("Editable design scene created at Assets/Scenes/MyCricketStadiumDesign.unity. Edit this scene for your custom stadium.");
        }

        [MenuItem("Cricket Prototype/Backup Design Scene")]
        public static void BackupDesignScene()
        {
            if (!File.Exists(DesignScenePath))
            {
                Debug.LogWarning("No design scene found to backup. Use Cricket Prototype > Make Editable Design Copy first.");
                return;
            }

            BackupAsset(DesignScenePath, "ManualDesignBackup");
        }

        private static void BackupAsset(string sourcePath, string label)
        {
            if (!File.Exists(sourcePath))
            {
                return;
            }

            Directory.CreateDirectory("Assets/Scenes/Backups");
            string fileName = Path.GetFileNameWithoutExtension(sourcePath);
            string timestamp = System.DateTime.Now.ToString("yyyyMMdd_HHmmss");
            string backupPath = $"Assets/Scenes/Backups/{fileName}_{label}_{timestamp}.unity";
            AssetDatabase.CopyAsset(sourcePath, backupPath);
            AssetDatabase.SaveAssets();
            Debug.Log($"Scene backup saved: {backupPath}");
        }

        private static void BuildSceneAtPath(string scenePath)
        {
            Directory.CreateDirectory("Assets/Scenes");
            Directory.CreateDirectory("Assets/Prefabs");
            Directory.CreateDirectory("Assets/Materials");

            Scene scene = EditorSceneManager.NewScene(NewSceneSetup.EmptyScene, NewSceneMode.Single);
            Physics.gravity = new Vector3(0f, -9.81f, 0f);

            CricketMaterialFactory materials = new CricketMaterialFactory();
            CricketSceneFactory sceneFactory = new CricketSceneFactory(materials);
            sceneFactory.Build();

            EditorSceneManager.MarkSceneDirty(scene);
            EditorSceneManager.SaveScene(scene, scenePath);
            AssetDatabase.SaveAssets();
            AssetDatabase.Refresh();
        }
    }
}
```

### `Assets\Editor\SceneBuilder\CricketSceneFactory.cs`

```csharp
using CricketGame;
using UnityEditor;
using UnityEngine;

namespace CricketGame.EditorTools
{
    public sealed class CricketSceneFactory
    {
        private readonly CricketMaterialFactory materials;

        public CricketSceneFactory(CricketMaterialFactory materialFactory)
        {
            materials = materialFactory;
        }

        public void Build()
        {
            GameObject game = new GameObject("Game");
            GameObject managers = CreateRoot("Managers", game.transform);
            GameObject ground = CreateRoot("Ground", game.transform);
            GameObject players = CreateRoot("Players", game.transform);
            GameObject fielders = CreateRoot("Fielders", game.transform);
            GameObject stadium = CreateRoot("Simple Stadium", game.transform);
            GameObject cameras = CreateRoot("Cameras", game.transform);

            CricketGameManager manager = new GameObject("Match Manager").AddComponent<CricketGameManager>();
            manager.transform.SetParent(managers.transform);

            MatchHudController hud = new GameObject("HUD").AddComponent<MatchHudController>();
            hud.transform.SetParent(managers.transform);
            hud.SetGameManager(manager);
            manager.SetHud(hud);

            CreateGround(ground.transform, manager);
            CreateWickets(ground.transform, manager);
            CricketBallController ballPrefab = CreateBallPrefab();
            BowlerController bowler = CreateBowler(players.transform, manager, ballPrefab);
            CreateBatsman(players.transform);
            CreateNonStriker(players.transform);
            BatsmanEntryController batsmanEntry = CreateNextBatsmanEntry(players.transform);
            CreateUmpire(players.transform);
            CreateFielders(fielders.transform, manager);
            CreateSimpleStadium(stadium.transform);
            CreateCamera(cameras.transform, manager);
            CreateLight(game.transform);

            manager.SetBowler(bowler);
            manager.SetBatsmanEntry(batsmanEntry);
        }

        private void CreateGround(Transform parent, CricketGameManager manager)
        {
            GameObject outfield = GameObject.CreatePrimitive(PrimitiveType.Cylinder);
            outfield.name = "Oval Outfield";
            outfield.transform.SetParent(parent);
            outfield.transform.position = new Vector3(0f, -0.06f, 0f);
            outfield.transform.localScale = new Vector3(52f, 0.045f, 58f);
            outfield.GetComponent<Renderer>().sharedMaterial = materials.Grass;
            outfield.AddComponent<GroundSurface>();

            GameObject pitch = GameObject.CreatePrimitive(PrimitiveType.Cube);
            pitch.name = "Pitch";
            pitch.transform.SetParent(parent);
            pitch.transform.position = new Vector3(0f, 0.01f, 0f);
            pitch.transform.localScale = new Vector3(3.05f, 0.08f, 22f);
            pitch.GetComponent<Renderer>().sharedMaterial = materials.Pitch;
            pitch.AddComponent<GroundSurface>();

            CreateLine("Batting Crease", parent, new Vector3(0f, 0.08f, -8.55f), new Vector3(4.2f, 0.035f, 0.05f));
            CreateLine("Bowling Crease", parent, new Vector3(0f, 0.08f, 8.55f), new Vector3(4.2f, 0.035f, 0.05f));
            CreateLine("Center Line", parent, new Vector3(0f, 0.085f, 0f), new Vector3(0.04f, 0.025f, 18.5f));
            CreateCutGrassPattern(parent);

            GameObject boundary = CreateRoot("Boundary", parent);
            CreateBoundaryWall("North Trigger", boundary.transform, new Vector3(0f, 12f, 27f), new Vector3(52f, 24f, 1f), manager);
            CreateBoundaryWall("South Trigger", boundary.transform, new Vector3(0f, 12f, -27f), new Vector3(52f, 24f, 1f), manager);
            CreateBoundaryWall("East Trigger", boundary.transform, new Vector3(25.5f, 12f, 0f), new Vector3(1f, 24f, 60f), manager);
            CreateBoundaryWall("West Trigger", boundary.transform, new Vector3(-25.5f, 12f, 0f), new Vector3(1f, 24f, 60f), manager);

            CreateOvalBoundaryRope("Oval Boundary Rope", boundary.transform, 24.8f, 27.4f, 48);
        }

        private void CreateCutGrassPattern(Transform parent)
        {
            GameObject pattern = CreateRoot("Oval Grass Pattern", parent);
            for (int index = -4; index <= 4; index++)
            {
                Material material = index % 2 == 0 ? materials.Grass : materials.Tree;
                GameObject stripe = GameObject.CreatePrimitive(PrimitiveType.Cube);
                stripe.name = $"Grass Stripe {index + 5}";
                stripe.transform.SetParent(pattern.transform);
                stripe.transform.position = new Vector3(index * 5.2f, 0.012f, 0f);
                stripe.transform.localScale = new Vector3(2.35f, 0.012f, 48f);
                stripe.GetComponent<Renderer>().sharedMaterial = material;
                Object.DestroyImmediate(stripe.GetComponent<Collider>());
            }
        }

        private void CreateWickets(Transform parent, CricketGameManager manager)
        {
            CreateWicketSet("Batting Wicket", parent, new Vector3(0f, 0f, -9.35f), manager);
            CreateWicketSet("Bowling Wicket", parent, new Vector3(0f, 0f, 9.35f), manager);
        }

        private void CreateWicketSet(string name, Transform parent, Vector3 origin, CricketGameManager manager)
        {
            GameObject root = new GameObject(name);
            root.transform.SetParent(parent);
            root.transform.position = origin;

            Wicket wicket = root.AddComponent<Wicket>();
            wicket.SetGameManager(manager);

            for (int index = -1; index <= 1; index++)
            {
                GameObject stump = GameObject.CreatePrimitive(PrimitiveType.Cylinder);
                stump.name = $"Stump {index + 2}";
                stump.transform.SetParent(root.transform);
                stump.transform.localPosition = new Vector3(index * 0.14f, 0.43f, 0f);
                stump.transform.localScale = new Vector3(0.045f, 0.43f, 0.045f);
                stump.GetComponent<Renderer>().sharedMaterial = materials.Bat;
            }

            for (int index = 0; index < 2; index++)
            {
                GameObject bail = GameObject.CreatePrimitive(PrimitiveType.Cube);
                bail.name = $"Bail {index + 1}";
                bail.transform.SetParent(root.transform);
                bail.transform.localPosition = new Vector3(index == 0 ? -0.07f : 0.07f, 0.88f, 0f);
                bail.transform.localScale = new Vector3(0.22f, 0.035f, 0.035f);
                bail.GetComponent<Renderer>().sharedMaterial = materials.Bat;
            }
        }

        private CricketBallController CreateBallPrefab()
        {
            const string prefabPath = "Assets/Prefabs/CricketBall.prefab";

            GameObject ball = GameObject.CreatePrimitive(PrimitiveType.Sphere);
            ball.name = "Cricket Ball";
            ball.transform.localScale = Vector3.one * 0.24f;
            ball.GetComponent<Renderer>().sharedMaterial = materials.Ball;

            Rigidbody body = ball.AddComponent<Rigidbody>();
            body.mass = 0.16f;
            body.linearDamping = 0.03f;
            body.angularDamping = 0.05f;
            body.collisionDetectionMode = CollisionDetectionMode.ContinuousDynamic;

            SphereCollider collider = ball.GetComponent<SphereCollider>();
            collider.sharedMaterial = materials.BallPhysics;

            ball.AddComponent<CricketBallController>();
            GameObject prefab = PrefabUtility.SaveAsPrefabAsset(ball, prefabPath);
            Object.DestroyImmediate(ball);
            return prefab.GetComponent<CricketBallController>();
        }

        private BowlerController CreateBowler(Transform parent, CricketGameManager manager, CricketBallController ballPrefab)
        {
            GameObject bowler = CreateRoot("Bowler", parent);
            bowler.transform.position = new Vector3(0f, 0f, 11.5f);

            CreateVisualPart(PrimitiveType.Capsule, "Body", bowler.transform, new Vector3(0f, 0.95f, 0f), new Vector3(0.48f, 0.95f, 0.48f), materials.Player);
            CreateVisualPart(PrimitiveType.Sphere, "Head", bowler.transform, new Vector3(0f, 2.15f, 0f), new Vector3(0.38f, 0.38f, 0.38f), materials.Trim);

            Transform releasePoint = CreateRoot("Release Point", bowler.transform).transform;
            releasePoint.localPosition = new Vector3(0f, 1.72f, -2.9f);

            Transform target = CreateRoot("Target", bowler.transform).transform;
            target.position = new Vector3(0f, 0.62f, -9.3f);

            BowlerController bowlerController = bowler.AddComponent<BowlerController>();
            bowlerController.SetReferences(manager, ballPrefab, releasePoint, target);
            return bowlerController;
        }

        private void CreateBatsman(Transform parent)
        {
            GameObject batsman = CreateRoot("Batsman", parent);
            batsman.transform.position = new Vector3(0f, 0f, -8.85f);

            CreateVisualPart(PrimitiveType.Capsule, "Body", batsman.transform, new Vector3(-0.18f, 0.95f, 0f), new Vector3(0.5f, 0.95f, 0.5f), materials.Player);
            CreateVisualPart(PrimitiveType.Sphere, "Head", batsman.transform, new Vector3(-0.18f, 2.12f, 0f), new Vector3(0.38f, 0.38f, 0.38f), materials.Trim);
            Transform frontArm = CreateVisualPart(PrimitiveType.Cylinder, "Front Arm", batsman.transform, new Vector3(0.14f, 1.35f, 0.18f), new Vector3(0.11f, 0.42f, 0.11f), materials.Trim).transform;
            Transform backArm = CreateVisualPart(PrimitiveType.Cylinder, "Back Arm", batsman.transform, new Vector3(-0.18f, 1.32f, -0.14f), new Vector3(0.11f, 0.44f, 0.11f), materials.Trim).transform;
            CreateVisualPart(PrimitiveType.Cylinder, "Front Leg", batsman.transform, new Vector3(-0.02f, 0.45f, 0.16f), new Vector3(0.12f, 0.48f, 0.12f), materials.Trim);
            CreateVisualPart(PrimitiveType.Cylinder, "Back Leg", batsman.transform, new Vector3(-0.34f, 0.45f, -0.12f), new Vector3(0.12f, 0.48f, 0.12f), materials.Trim);

            GameObject batRoot = CreateRoot("Bat", batsman.transform);
            batRoot.transform.localPosition = new Vector3(0.16f, 1.1f, 0.2f);

            BatSwingController swing = batRoot.AddComponent<BatSwingController>();

            GameObject handle = CreateVisualPart(PrimitiveType.Cylinder, "Handle", batRoot.transform, new Vector3(0f, 0.46f, 0f), new Vector3(0.05f, 0.26f, 0.05f), materials.Trim);
            handle.transform.localRotation = Quaternion.Euler(0f, 0f, 10f);

            GameObject blade = GameObject.CreatePrimitive(PrimitiveType.Cube);
            blade.name = "Blade";
            blade.transform.SetParent(batRoot.transform);
            blade.transform.localPosition = new Vector3(0f, -0.1f, 0f);
            blade.transform.localScale = new Vector3(0.22f, 1f, 0.12f);
            blade.transform.localRotation = Quaternion.Euler(0f, 0f, 8f);
            blade.GetComponent<Renderer>().sharedMaterial = materials.Bat;

            Rigidbody bladeBody = blade.AddComponent<Rigidbody>();
            bladeBody.isKinematic = true;
            bladeBody.collisionDetectionMode = CollisionDetectionMode.ContinuousSpeculative;

            BoxCollider bladeCollider = blade.GetComponent<BoxCollider>();
            BatContactReporter reporter = blade.AddComponent<BatContactReporter>();
            reporter.SetBat(swing);

            swing.SetReferences(batRoot.transform, bladeCollider, frontArm, backArm);
        }

        private void CreateNonStriker(Transform parent)
        {
            GameObject nonStriker = CreateRoot("Non Striker", parent);
            nonStriker.transform.position = new Vector3(-1.8f, 0f, 6.8f);
            CreateVisualPart(PrimitiveType.Capsule, "Body", nonStriker.transform, new Vector3(0f, 0.95f, 0f), new Vector3(0.46f, 0.92f, 0.46f), materials.Player);
            CreateVisualPart(PrimitiveType.Sphere, "Head", nonStriker.transform, new Vector3(0f, 2.08f, 0f), new Vector3(0.35f, 0.35f, 0.35f), materials.Trim);
            CreateVisualPart(PrimitiveType.Cube, "Bat", nonStriker.transform, new Vector3(0.35f, 0.72f, 0.2f), new Vector3(0.18f, 1f, 0.08f), materials.Bat);
        }

        private BatsmanEntryController CreateNextBatsmanEntry(Transform parent)
        {
            GameObject nextBatsman = CreateRoot("New Batsman Entry", parent);
            Vector3 entryStart = new Vector3(-6.5f, 0f, -30.8f);
            Vector3 creaseTarget = new Vector3(1.55f, 0f, -8.9f);
            nextBatsman.transform.position = entryStart;

            CreateVisualPart(PrimitiveType.Capsule, "Body", nextBatsman.transform, new Vector3(0f, 0.95f, 0f), new Vector3(0.48f, 0.93f, 0.48f), materials.Player);
            CreateVisualPart(PrimitiveType.Sphere, "Head", nextBatsman.transform, new Vector3(0f, 2.08f, 0f), new Vector3(0.36f, 0.36f, 0.36f), materials.Trim);
            CreateVisualPart(PrimitiveType.Cylinder, "Left Leg", nextBatsman.transform, new Vector3(-0.16f, 0.45f, 0f), new Vector3(0.11f, 0.48f, 0.11f), materials.Trim);
            CreateVisualPart(PrimitiveType.Cylinder, "Right Leg", nextBatsman.transform, new Vector3(0.16f, 0.45f, 0f), new Vector3(0.11f, 0.48f, 0.11f), materials.Trim);
            CreateVisualPart(PrimitiveType.Cylinder, "Left Arm", nextBatsman.transform, new Vector3(-0.38f, 1.28f, 0f), new Vector3(0.09f, 0.4f, 0.09f), materials.Trim);
            CreateVisualPart(PrimitiveType.Cylinder, "Right Arm", nextBatsman.transform, new Vector3(0.38f, 1.28f, 0f), new Vector3(0.09f, 0.4f, 0.09f), materials.Trim);
            GameObject bat = CreateVisualPart(PrimitiveType.Cube, "Walking Bat", nextBatsman.transform, new Vector3(0.55f, 0.75f, 0.12f), new Vector3(0.16f, 1.05f, 0.08f), materials.Bat);
            bat.transform.localRotation = Quaternion.Euler(0f, 0f, -13f);

            BatsmanEntryController controller = nextBatsman.AddComponent<BatsmanEntryController>();
            controller.SetRoute(entryStart, creaseTarget, 3.2f);
            return controller;
        }

        private void CreateUmpire(Transform parent)
        {
            GameObject umpire = CreateRoot("Umpire", parent);
            umpire.transform.position = new Vector3(0f, 0f, 7.2f);
            CreateVisualPart(PrimitiveType.Capsule, "Body", umpire.transform, new Vector3(0f, 0.95f, 0f), new Vector3(0.46f, 0.95f, 0.46f), materials.Umpire);
            CreateVisualPart(PrimitiveType.Sphere, "Head", umpire.transform, new Vector3(0f, 2.08f, 0f), new Vector3(0.35f, 0.35f, 0.35f), materials.Trim);
            CreateVisualPart(PrimitiveType.Cylinder, "Hat", umpire.transform, new Vector3(0f, 2.34f, 0f), new Vector3(0.26f, 0.05f, 0.26f), materials.Umpire);
        }

        private void CreateFielders(Transform parent, CricketGameManager manager)
        {
            Vector3[] powerplayPositions =
            {
                new Vector3(-9.5f, 0f, -2f),
                new Vector3(9.5f, 0f, -2f),
                new Vector3(-12f, 0f, 7f),
                new Vector3(12f, 0f, 7f),
                new Vector3(-8f, 0f, 14f),
                new Vector3(8f, 0f, 14f),
                new Vector3(0f, 0f, 17f),
                new Vector3(-20.8f, 0f, 20.5f),
                new Vector3(20.8f, 0f, 20.5f)
            };

            Vector3[] normalPositions =
            {
                new Vector3(-12.5f, 0f, -4f),
                new Vector3(12.5f, 0f, -4f),
                new Vector3(-21.5f, 0f, 2.5f),
                new Vector3(21.5f, 0f, 2.5f),
                new Vector3(-18.5f, 0f, 17.5f),
                new Vector3(18.5f, 0f, 17.5f),
                new Vector3(0f, 0f, 23.5f),
                new Vector3(-7.5f, 0f, 9f),
                new Vector3(7.5f, 0f, 9f)
            };

            for (int index = 0; index < powerplayPositions.Length; index++)
            {
                GameObject fielder = CreateRoot($"Fielder {index + 1}", parent);
                fielder.transform.position = powerplayPositions[index];

                CreateVisualPart(PrimitiveType.Capsule, "Body", fielder.transform, new Vector3(0f, 0.95f, 0f), new Vector3(0.42f, 0.9f, 0.42f), materials.Player);
                CreateVisualPart(PrimitiveType.Sphere, "Head", fielder.transform, new Vector3(0f, 2.04f, 0f), new Vector3(0.32f, 0.32f, 0.32f), materials.Trim);

                FielderController controller = fielder.AddComponent<FielderController>();
                controller.SetFieldPlan(manager, fielder.name, powerplayPositions[index], normalPositions[index]);
            }
        }

        private void CreateSimpleStadium(Transform parent)
        {
            CreateOuterWalls(parent);
            CreateSeating(parent);
            CreateBoundaryBuildings(parent);
            CreateSponsorBoards(parent);
            CreatePavilion(parent);
            CreateEntranceGate(parent);
            CreateWalkInPath(parent);
            CreateFloodlights(parent);
            CreateTrees(parent);
        }

        private void CreateOuterWalls(Transform parent)
        {
            GameObject walls = CreateRoot("Walls", parent);
            CreateOvalWallRing("Outer Oval Wall", walls.transform, 29.4f, 32.1f, 52, 0.85f, 1.55f, materials.Wall);
            CreateOvalWallRing("White Top Ring", walls.transform, 29.4f, 32.1f, 52, 1.68f, 0.18f, materials.Line);
        }

        private void CreateSeating(Transform parent)
        {
            GameObject stands = CreateRoot("Seating Stands", parent);
            CreateOvalStandBase("Oval Stand Base", stands.transform, 28.5f, 31.2f, 52);
            CreateOvalSeatRing("Lower Seat Ring", stands.transform, 28.7f, 31.4f, 52, 1.9f, 0);
            CreateOvalSeatRing("Middle Seat Ring", stands.transform, 30.1f, 32.8f, 56, 2.25f, 1);
            CreateOvalSeatRing("Upper Seat Ring", stands.transform, 31.5f, 34.2f, 60, 2.6f, 2);
        }

        private void CreateSeatRow(string name, Transform parent, Vector3 startPosition, int count, Vector3 direction, float spacing)
        {
            GameObject row = CreateRoot(name, parent);
            for (int index = 0; index < count; index++)
            {
                Material material = index % 2 == 0 ? materials.SeatBlue : materials.SeatYellow;
                Vector3 position = startPosition + direction * spacing * index;
                CreateStadiumPart(PrimitiveType.Cube, $"Seat {index + 1}", row.transform, position, new Vector3(0.55f, 0.28f, 0.45f), Quaternion.identity, material);
            }
        }

        private void CreateOvalWallRing(string name, Transform parent, float radiusX, float radiusZ, int segments, float height, float wallHeight, Material material)
        {
            GameObject ring = CreateRoot(name, parent);
            for (int index = 0; index < segments; index++)
            {
                float angle = index * Mathf.PI * 2f / segments;
                Vector3 radial = new Vector3(Mathf.Cos(angle), 0f, Mathf.Sin(angle));

                if (radial.z < -0.92f && Mathf.Abs(radial.x) < 0.28f)
                {
                    continue;
                }

                Vector3 position = new Vector3(radial.x * radiusX, height, radial.z * radiusZ);
                Quaternion rotation = Quaternion.LookRotation(radial.normalized, Vector3.up);
                CreateStadiumPart(PrimitiveType.Cube, $"Wall Segment {index + 1}", ring.transform, position, new Vector3(3.9f, wallHeight, 0.55f), rotation, material);
            }
        }

        private void CreateOvalStandBase(string name, Transform parent, float radiusX, float radiusZ, int segments)
        {
            GameObject baseRing = CreateRoot(name, parent);
            for (int index = 0; index < segments; index++)
            {
                float angle = index * Mathf.PI * 2f / segments;
                Vector3 radial = new Vector3(Mathf.Cos(angle), 0f, Mathf.Sin(angle));
                Vector3 position = new Vector3(radial.x * radiusX, 1.62f, radial.z * radiusZ);
                Quaternion rotation = Quaternion.LookRotation(radial.normalized, Vector3.up);
                CreateStadiumPart(PrimitiveType.Cube, $"Base Segment {index + 1}", baseRing.transform, position, new Vector3(4.1f, 0.24f, 2.4f), rotation, materials.Trim);
            }
        }

        private void CreateOvalSeatRing(string name, Transform parent, float radiusX, float radiusZ, int count, float height, int offset)
        {
            GameObject ring = CreateRoot(name, parent);
            for (int index = 0; index < count; index++)
            {
                float angle = (index + offset * 0.35f) * Mathf.PI * 2f / count;
                Vector3 radial = new Vector3(Mathf.Cos(angle), 0f, Mathf.Sin(angle));

                if (radial.z < -0.94f && Mathf.Abs(radial.x) < 0.2f)
                {
                    continue;
                }

                Vector3 position = new Vector3(radial.x * radiusX, height, radial.z * radiusZ);
                Quaternion rotation = Quaternion.LookRotation(radial.normalized, Vector3.up);
                Material material = (index + offset) % 2 == 0 ? materials.SeatBlue : materials.SeatYellow;
                CreateStadiumPart(PrimitiveType.Cube, $"Seat {index + 1}", ring.transform, position, new Vector3(0.6f, 0.28f, 0.48f), rotation, material);
            }
        }

        private void CreateBoundaryBuildings(Transform parent)
        {
            GameObject buildings = CreateRoot("Boundary Buildings", parent);
            CreateBuilding("Media Building", buildings.transform, new Vector3(-16f, 3.1f, 36.2f), new Vector3(8.5f, 6.2f, 3.8f), 4);
            CreateBuilding("Club House", buildings.transform, new Vector3(15.5f, 2.85f, 36f), new Vector3(9f, 5.7f, 3.7f), 4);
            CreateBuilding("Left Apartment Block", buildings.transform, new Vector3(-31.8f, 3.35f, 11f), new Vector3(4.4f, 6.7f, 10.5f), 5);
            CreateBuilding("Right Apartment Block", buildings.transform, new Vector3(31.8f, 3.35f, 11f), new Vector3(4.4f, 6.7f, 10.5f), 5);
            CreateBuilding("Food Court", buildings.transform, new Vector3(-13f, 1.55f, -34.8f), new Vector3(7.5f, 3.1f, 2.8f), 2);
            CreateBuilding("Ticket Office", buildings.transform, new Vector3(13f, 1.55f, -34.8f), new Vector3(7.5f, 3.1f, 2.8f), 2);
            CreateCornerTower("North West Corner Tower", buildings.transform, new Vector3(-29.2f, 0f, 31.8f));
            CreateCornerTower("North East Corner Tower", buildings.transform, new Vector3(29.2f, 0f, 31.8f));
            CreateCornerTower("South West Corner Tower", buildings.transform, new Vector3(-29.2f, 0f, -31.8f));
            CreateCornerTower("South East Corner Tower", buildings.transform, new Vector3(29.2f, 0f, -31.8f));
        }

        private void CreateBuilding(string name, Transform parent, Vector3 position, Vector3 size, int floors)
        {
            GameObject building = CreateRoot(name, parent);
            CreateStadiumPart(PrimitiveType.Cube, "Main Block", building.transform, position, size, Quaternion.identity, materials.Building);
            CreateStadiumPart(PrimitiveType.Cube, "Roof", building.transform, position + Vector3.up * (size.y * 0.5f + 0.15f), new Vector3(size.x + 0.4f, 0.22f, size.z + 0.4f), Quaternion.identity, materials.Trim);

            int columns = Mathf.Max(2, Mathf.RoundToInt(size.x / 1.7f));
            float windowStartX = position.x - size.x * 0.35f;
            float windowSpacing = size.x * 0.7f / Mathf.Max(1, columns - 1);

            for (int floor = 0; floor < floors; floor++)
            {
                float y = position.y - size.y * 0.3f + floor * (size.y * 0.6f / Mathf.Max(1, floors - 1));
                for (int column = 0; column < columns; column++)
                {
                    float x = windowStartX + column * windowSpacing;
                    float z = position.z - size.z * 0.51f;
                    CreateStadiumPart(PrimitiveType.Cube, $"Window {floor + 1}-{column + 1}", building.transform, new Vector3(x, y, z), new Vector3(0.5f, 0.35f, 0.06f), Quaternion.identity, materials.Window);
                }
            }
        }

        private void CreateCornerTower(string name, Transform parent, Vector3 basePosition)
        {
            GameObject tower = CreateRoot(name, parent);
            CreateStadiumPart(PrimitiveType.Cylinder, "Tower Body", tower.transform, basePosition + Vector3.up * 1.7f, new Vector3(0.75f, 1.7f, 0.75f), Quaternion.identity, materials.Wall);
            CreateStadiumPart(PrimitiveType.Cylinder, "Tower Top", tower.transform, basePosition + Vector3.up * 3.55f, new Vector3(0.92f, 0.18f, 0.92f), Quaternion.identity, materials.Trim);
        }

        private void CreateSponsorBoards(Transform parent)
        {
            GameObject boards = CreateRoot("Sponsor Boards", parent);
            string[] labels = { "CRICKET", "SIX", "POWER", "STADIUM", "SPORTS" };
            const int count = 32;

            for (int index = 0; index < count; index++)
            {
                float angle = index * Mathf.PI * 2f / count;
                Vector3 radial = new Vector3(Mathf.Cos(angle), 0f, Mathf.Sin(angle));
                Vector3 position = new Vector3(radial.x * 26.6f, 1.55f, radial.z * 29.1f);
                Quaternion rotation = Quaternion.LookRotation(radial.normalized, Vector3.up);
                CreateStadiumPart(PrimitiveType.Cube, $"Oval Board {index + 1}", boards.transform, position, new Vector3(3.05f, 0.75f, 0.12f), rotation, materials.Board);
                CreateTextLabel($"Oval Board Text {index + 1}", boards.transform, labels[index % labels.Length], position - radial * 0.15f, new Vector3(0.13f, 0.13f, 0.13f), rotation);
            }
        }

        private void CreateBoardRow(string name, Transform parent, Vector3 startPosition, int count, Vector3 direction, float spacing, Quaternion textRotation)
        {
            string[] labels = { "CRICKET", "SIX", "POWER", "STADIUM", "SPORTS" };
            GameObject row = CreateRoot(name, parent);

            for (int index = 0; index < count; index++)
            {
                Vector3 position = startPosition + direction * spacing * index;
                CreateStadiumPart(PrimitiveType.Cube, $"Board {index + 1}", row.transform, position, new Vector3(3.2f, 0.9f, 0.12f), textRotation, materials.Board);
                Vector3 textPosition = position + textRotation * Vector3.back * 0.12f;
                CreateTextLabel($"Board Text {index + 1}", row.transform, labels[index % labels.Length], textPosition, new Vector3(0.16f, 0.16f, 0.16f), textRotation);
            }
        }

        private void CreatePavilion(Transform parent)
        {
            GameObject pavilion = CreateRoot("Pavilion", parent);
            CreateStadiumPart(PrimitiveType.Cube, "Pavilion Building", pavilion.transform, new Vector3(0f, 2.15f, 35.4f), new Vector3(12.5f, 3.1f, 3.5f), Quaternion.identity, materials.Trim);
            CreateStadiumPart(PrimitiveType.Cube, "Pavilion Roof", pavilion.transform, new Vector3(0f, 3.9f, 35.2f), new Vector3(14.2f, 0.35f, 4.3f), Quaternion.identity, materials.Line);
            CreateStadiumPart(PrimitiveType.Cube, "Viewing Glass", pavilion.transform, new Vector3(0f, 2.45f, 33.55f), new Vector3(8.8f, 1.1f, 0.16f), Quaternion.identity, materials.SeatBlue);
            CreateStadiumPart(PrimitiveType.Cube, "Scoreboard", pavilion.transform, new Vector3(0f, 5.15f, 33f), new Vector3(7.4f, 2.05f, 0.24f), Quaternion.identity, materials.Scoreboard);
            CreateStadiumPart(PrimitiveType.Cube, "Scoreboard Frame", pavilion.transform, new Vector3(0f, 5.15f, 33.16f), new Vector3(8f, 2.28f, 0.12f), Quaternion.identity, materials.Line);
            CreateTextLabel("Scoreboard Text", pavilion.transform, "MY CRICKET GROUND", new Vector3(0f, 5.15f, 32.84f), new Vector3(0.35f, 0.35f, 0.35f), Quaternion.Euler(0f, 180f, 0f));
            CreateTextLabel("Version Board Text", pavilion.transform, "2 OVER OVAL STADIUM", new Vector3(0f, 4.35f, 32.82f), new Vector3(0.22f, 0.22f, 0.22f), Quaternion.Euler(0f, 180f, 0f));

            for (int index = -2; index <= 2; index++)
            {
                CreateStadiumPart(PrimitiveType.Cylinder, $"Pillar {index + 3}", pavilion.transform, new Vector3(index * 2.5f, 1.55f, 33.25f), new Vector3(0.12f, 1.35f, 0.12f), Quaternion.identity, materials.Line);
            }
        }

        private void CreateEntranceGate(Transform parent)
        {
            GameObject gate = CreateRoot("Entrance Gate", parent);
            CreateStadiumPart(PrimitiveType.Cube, "Gate Left Pillar", gate.transform, new Vector3(-4.4f, 1.35f, -31.4f), new Vector3(0.75f, 2.7f, 0.75f), Quaternion.identity, materials.Trim);
            CreateStadiumPart(PrimitiveType.Cube, "Gate Right Pillar", gate.transform, new Vector3(4.4f, 1.35f, -31.4f), new Vector3(0.75f, 2.7f, 0.75f), Quaternion.identity, materials.Trim);
            CreateStadiumPart(PrimitiveType.Cube, "Gate Sign Board", gate.transform, new Vector3(0f, 2.85f, -31.4f), new Vector3(8.3f, 0.75f, 0.32f), Quaternion.identity, materials.Scoreboard);
            CreateTextLabel("Gate Text", gate.transform, "CRICKET STADIUM", new Vector3(0f, 2.85f, -31.63f), new Vector3(0.28f, 0.28f, 0.28f), Quaternion.identity);

            for (int index = -4; index <= 4; index++)
            {
                CreateStadiumPart(PrimitiveType.Cube, $"Gate Bar {index + 5}", gate.transform, new Vector3(index * 0.55f, 0.95f, -31.65f), new Vector3(0.08f, 1.9f, 0.08f), Quaternion.identity, materials.Umpire);
            }
        }

        private void CreateWalkInPath(Transform parent)
        {
            GameObject path = CreateRoot("Batsman Walk In Path", parent);
            CreateStadiumPart(PrimitiveType.Cube, "Entry Path", path.transform, new Vector3(-3.6f, 0.04f, -22f), new Vector3(1.4f, 0.05f, 16.5f), Quaternion.Euler(0f, -16f, 0f), materials.Pitch);

            for (int index = 0; index < 7; index++)
            {
                Vector3 position = new Vector3(-6.2f + index * 0.82f, 0.1f, -30.2f + index * 3.2f);
                CreateStadiumPart(PrimitiveType.Cube, $"Path Line {index + 1}", path.transform, position, new Vector3(0.45f, 0.04f, 0.08f), Quaternion.Euler(0f, -16f, 0f), materials.Line);
            }
        }

        private void CreateFloodlights(Transform parent)
        {
            GameObject lights = CreateRoot("Flood Lights", parent);
            CreateFloodlightTower("Flood Light 1", lights.transform, new Vector3(-20f, 0f, -22f));
            CreateFloodlightTower("Flood Light 2", lights.transform, new Vector3(20f, 0f, -22f));
            CreateFloodlightTower("Flood Light 3", lights.transform, new Vector3(-20f, 0f, 22f));
            CreateFloodlightTower("Flood Light 4", lights.transform, new Vector3(20f, 0f, 22f));
        }

        private void CreateFloodlightTower(string name, Transform parent, Vector3 basePosition)
        {
            GameObject tower = CreateRoot(name, parent);
            CreateStadiumPart(PrimitiveType.Cylinder, "Pole", tower.transform, basePosition + Vector3.up * 2.9f, new Vector3(0.24f, 2.9f, 0.24f), Quaternion.identity, materials.Floodlight);
            CreateStadiumPart(PrimitiveType.Cube, "Lamp Bar", tower.transform, basePosition + Vector3.up * 5.85f, new Vector3(1.9f, 0.18f, 0.28f), Quaternion.identity, materials.Line);

            for (int index = -1; index <= 1; index++)
            {
                CreateStadiumPart(PrimitiveType.Cube, $"Lamp {index + 2}", tower.transform, basePosition + new Vector3(index * 0.55f, 6.08f, 0f), new Vector3(0.38f, 0.28f, 0.38f), Quaternion.identity, materials.SeatYellow);
            }

            GameObject lamp = new GameObject("Spot Light");
            lamp.transform.SetParent(tower.transform);
            lamp.transform.position = basePosition + new Vector3(0f, 5.9f, 0f);
            lamp.transform.LookAt(new Vector3(0f, 0.3f, 0f));
            Light spot = lamp.AddComponent<Light>();
            spot.type = LightType.Spot;
            spot.color = Color.white;
            spot.intensity = 0.85f;
            spot.range = 36f;
            spot.spotAngle = 42f;
        }

        private void CreateTrees(Transform parent)
        {
            GameObject trees = CreateRoot("Small Trees", parent);
            for (int index = 0; index < 7; index++)
            {
                float x = -18f + index * 6f;
                CreateTree($"Tree {index + 1}", trees.transform, new Vector3(x, 0f, -34.2f));
            }
        }

        private void CreateTree(string name, Transform parent, Vector3 basePosition)
        {
            GameObject tree = CreateRoot(name, parent);
            CreateStadiumPart(PrimitiveType.Cylinder, "Trunk", tree.transform, basePosition + Vector3.up * 0.45f, new Vector3(0.13f, 0.45f, 0.13f), Quaternion.identity, materials.Bat);
            CreateStadiumPart(PrimitiveType.Sphere, "Leaves", tree.transform, basePosition + Vector3.up * 1.25f, new Vector3(0.75f, 0.75f, 0.75f), Quaternion.identity, materials.Tree);
        }

        private void CreateCamera(Transform parent, CricketGameManager manager)
        {
            GameObject rig = CreateRoot("Camera Rig", parent);
            rig.transform.position = new Vector3(0f, 8.2f, -24f);
            rig.transform.rotation = Quaternion.LookRotation(new Vector3(0f, 1.25f, -1f) - rig.transform.position, Vector3.up);

            GameObject camera = new GameObject("Main Camera");
            camera.transform.SetParent(parent);
            camera.tag = "MainCamera";
            camera.transform.position = rig.transform.position;
            camera.transform.rotation = rig.transform.rotation;
            Camera cameraComponent = camera.AddComponent<Camera>();
            cameraComponent.fieldOfView = 68f;
            cameraComponent.nearClipPlane = 0.08f;
            cameraComponent.farClipPlane = 160f;
            camera.AddComponent<AudioListener>();

            CricketCameraController controller = camera.AddComponent<CricketCameraController>();
            controller.SetReferences(manager, rig.transform);
        }

        private void CreateLight(Transform parent)
        {
            RenderSettings.ambientMode = UnityEngine.Rendering.AmbientMode.Flat;
            RenderSettings.ambientLight = new Color(0.55f, 0.58f, 0.62f);

            GameObject light = new GameObject("Light");
            light.transform.SetParent(parent);
            Light sun = light.AddComponent<Light>();
            sun.type = LightType.Directional;
            sun.intensity = 1.15f;
            light.transform.rotation = Quaternion.Euler(50f, -30f, 0f);
        }

        private void CreateBoundaryWall(string name, Transform parent, Vector3 position, Vector3 size, CricketGameManager manager)
        {
            GameObject wall = new GameObject(name);
            wall.transform.SetParent(parent);
            wall.transform.position = position;
            BoxCollider collider = wall.AddComponent<BoxCollider>();
            collider.isTrigger = true;
            collider.size = size;

            BoundaryZone zone = wall.AddComponent<BoundaryZone>();
            zone.SetGameManager(manager);
        }

        private void CreateBoundaryRope(string name, Transform parent, Vector3 position, Vector3 scale)
        {
            GameObject rope = GameObject.CreatePrimitive(PrimitiveType.Cube);
            rope.name = name;
            rope.transform.SetParent(parent);
            rope.transform.position = position;
            rope.transform.localScale = scale;
            rope.GetComponent<Renderer>().sharedMaterial = materials.Line;
            Object.DestroyImmediate(rope.GetComponent<Collider>());
        }

        private void CreateOvalBoundaryRope(string name, Transform parent, float radiusX, float radiusZ, int segments)
        {
            GameObject ropeRoot = CreateRoot(name, parent);
            for (int index = 0; index < segments; index++)
            {
                float angle = index * Mathf.PI * 2f / segments;
                Vector3 radial = new Vector3(Mathf.Cos(angle), 0f, Mathf.Sin(angle));
                Vector3 position = new Vector3(radial.x * radiusX, 0.08f, radial.z * radiusZ);
                Quaternion rotation = Quaternion.LookRotation(radial.normalized, Vector3.up);
                CreateStadiumPart(PrimitiveType.Cube, $"Rope Segment {index + 1}", ropeRoot.transform, position, new Vector3(3.35f, 0.08f, 0.16f), rotation, materials.Line);
            }
        }

        private void CreateLine(string name, Transform parent, Vector3 position, Vector3 scale)
        {
            GameObject line = GameObject.CreatePrimitive(PrimitiveType.Cube);
            line.name = name;
            line.transform.SetParent(parent);
            line.transform.position = position;
            line.transform.localScale = scale;
            line.GetComponent<Renderer>().sharedMaterial = materials.Line;
            Object.DestroyImmediate(line.GetComponent<Collider>());
        }

        private static GameObject CreateRoot(string name, Transform parent)
        {
            GameObject root = new GameObject(name);
            root.transform.SetParent(parent);
            return root;
        }

        private static GameObject CreateStadiumPart(PrimitiveType primitiveType, string name, Transform parent, Vector3 position, Vector3 scale, Quaternion rotation, Material material)
        {
            GameObject part = GameObject.CreatePrimitive(primitiveType);
            part.name = name;
            part.transform.SetParent(parent);
            part.transform.position = position;
            part.transform.rotation = rotation;
            part.transform.localScale = scale;
            part.GetComponent<Renderer>().sharedMaterial = material;
            Object.DestroyImmediate(part.GetComponent<Collider>());
            return part;
        }

        private void CreateTextLabel(string name, Transform parent, string textValue, Vector3 position, Vector3 scale, Quaternion rotation)
        {
            GameObject textObject = new GameObject(name);
            textObject.transform.SetParent(parent);
            textObject.transform.position = position;
            textObject.transform.rotation = rotation;
            textObject.transform.localScale = scale;

            TextMesh text = textObject.AddComponent<TextMesh>();
            text.text = textValue;
            text.anchor = TextAnchor.MiddleCenter;
            text.alignment = TextAlignment.Center;
            text.characterSize = 1f;
            text.fontSize = 42;
            text.color = Color.white;

            MeshRenderer renderer = textObject.GetComponent<MeshRenderer>();
            renderer.sharedMaterial = materials.Line;
        }

        private static GameObject CreateVisualPart(PrimitiveType primitiveType, string name, Transform parent, Vector3 localPosition, Vector3 localScale, Material material)
        {
            GameObject part = GameObject.CreatePrimitive(primitiveType);
            part.name = name;
            part.transform.SetParent(parent);
            part.transform.localPosition = localPosition;
            part.transform.localScale = localScale;
            part.GetComponent<Renderer>().sharedMaterial = material;
            Object.DestroyImmediate(part.GetComponent<Collider>());
            return part;
        }
    }
}
```
