# Postrainer

Postrainer is an AI workout coach for people exercising at home. It watches you through your laptop webcam, counts your reps, and gives live posture feedback for push-ups, sit-ups and squats. Each session's score is saved so you can track progress over time.

> The models were trained on a small set of self-recorded clips. Treat the feedback as a guide, not professional coaching.

## How To Use It During A Workout

| Signal | What it means | What to do |
|---|---|---|
| Green border | You're recognised as in position, so reps are counting | Keep going |
| Red border | Not recognised, so reps aren't counting | Turn side-on and get your full body in frame |
| "The camera cannot see you well" | Parts of your body are hidden | Move back to about 6 ft and improve the lighting |
| Form messages (e.g. "go down lower") | A rep didn't reach full range | Adjust on the next rep |
| `YOUR SCORE` | How closely your pose matches good form | Aim for a consistently high score |

**Putting it together**
- **Good set:** the border is green, the counter is climbing, and no form messages appear.
- **Camera problem:** the border is red and the counter is frozen. Fix your setup before changing your technique.
- **Cutting range:** reps count but form messages keep reappearing. Slow down and use full range.

## The Product Flow

1. **Load Workout (Initiation):** reads `workout.txt`, e.g. `PUSHUP 10`, so instructors can publish daily plans without code changes.
2. **Briefing (Setup):** shows the target reps, camera placement and a motivational quote while you get into position.
3. **Pose Detection (Perception):** MediaPipe Pose tracks 33 body landmarks on-device, so no video leaves your laptop.
4. **Position Check (Classification):** a small Keras model per exercise confirms you're actually doing the exercise before anything counts.
5. **Rep Counting & Feedback (Coaching):** joint-angle thresholds count each rep and flag shallow or incomplete movement.
6. **Session Score (Logging):** the average score is appended to `scores.txt`.

## Supported Exercises

| Exercise | Tracks | Camera setup |
|---|---|---|
| Push-up | Chest depth, back straightness | Laptop on floor, ~6 ft away, side-on |
| Sit-up | Hip angle, full range up and down | Laptop on floor, ~6 ft away, side-on |
| Squat | Knee depth (~90°), back angle, pace | Laptop on table, ~6 ft away, side-on |
| Bicep curl | Classifier trained, not yet in workouts | — |

## Project Structure

```
postrainer/
├── models/
│   ├── modeltest1.py       # Main app: runs the workout, counts reps, logs scores
│   ├── preprocessing.py    # Pose extraction + per-exercise features
│   ├── model-trainer.py    # Trains a position classifier
│   ├── model-runner.py     # Live test of all classifiers
│   ├── *-model.keras       # Trained models (push-up, sit-up, squat, curl)
│   └── data/               # Training videos
├── frontend/               # Website: home, download, today's workout, team
├── backend/                # Django backend for instructor uploads (in progress)
└── mppose.py               # Simple webcam pose demo
```

## Getting Started

**Prerequisites:** Python 3.7.9 and a webcam.

```bash
conda create -n postrainer python=3.7.9
conda activate postrainer
pip install tensorflow==2.8.0 mediapipe==0.8.9.1 opencv-python==4.5.5.64 protobuf==3.20.1 numpy
```

Create a workout and run it from `models/`:

```bash
cd models
printf "PUSHUP 10\nSQUAT 15\nSITUP 10\n" > workout.txt
python modeltest1.py
```

Press **Esc** to quit. To check your camera first, run `python ../mppose.py`.

## Troubleshooting

**Reps aren't counting**
- Make sure the border is green. Face side-on, about 6 ft away, with your whole body visible.

**`FileNotFoundError: workout.txt`**
- Run the app from inside `models/`, with `workout.txt` in that folder.

**MediaPipe / protobuf errors**
- Install the pinned versions above, especially `protobuf==3.20.1`.

---

Everything runs locally on your machine; the only thing saved is your score in `scores.txt`.
