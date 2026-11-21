# Reading the Serve: Computer Vision in Tennis

**Can a model tell a flat, slice and kick serve apart from video?**

This repository holds the slide deck, data and analysis for a study of how well computer vision models recognise tennis strokes. We compare **frontier multimodal models used zero-shot** against **models trained on tennis data**, using the THETIS dataset. We look closely at the hardest case: telling the three serve types apart.

**Authors:** Salim Oyinlola · Kayode Makinde

---

## Motivation

Computer vision already officiates tennis. Electronic line calling replaced line judges at the Australian Open in 2021, across the ATP Tour in 2025, and at Wimbledon in 2025. *In or out?* is a solved problem.

*Which serve was that?* is still open. The serve starts every point, and knowing which serve a player picks (on break point, out wide, on the deuce side) is useful for scouting. Today that tagging is done by hand. Flat, slice and kick serves look almost identical in body motion. They differ mainly in racket angle at contact and in the ball's spin, both of which happen within a split second.

## Hypotheses

1. **Shot type is easy.** Every model should separate serves, volleys, groundstrokes and overheads, because the whole-body motion is very different.
2. **Serve type is hard.** Flat, slice and kick differ mostly in spin and racket angle, and THETIS has no visible ball.
3. **Trained models beat frontier models.** Models trained on THETIS should beat zero-shot frontier models. Pose-based models should be strongest where the stroke is defined by body motion.

## Dataset

**THETIS** (Three-dimensional TEnnis Shots) is a human action dataset with 12 stroke classes recorded with Kinect by 55 amateur players. Players swing a racket **without a ball**, so there is no spin, trajectory or bounce in the data.

| Broad category | Classes |
|---|---|
| Groundstroke | backhand, two-hand backhand, backhand slice, forehand flat, forehand open stance, forehand slice |
| Volley | backhand volley, forehand volley |
| Serve | flat serve, kick serve, slice serve |
| Overhead | smash |

> Gourgari, S., Goudelis, G., Karpouzis, K., & Kollias, S. (2013). *THETIS: Three-dimensional tennis shots, a human action dataset.* In Proceedings of the IEEE Conference on Computer Vision and Pattern Recognition Workshops (pp. 676–681).

## Models

| Group | Model | Input | Test set |
|---|---|---|---|
| Frontier (zero-shot) | Claude Opus 5.5 | RGB video | Shared 60 clips |
| Frontier (zero-shot) | ChatGPT GPT-6 | RGB video | Shared 60 clips |
| Frontier (zero-shot) | Gemini Hydrogen | RGB video | Shared 60 clips |
| Trained | Pose CNN (1D CNN on MediaPipe Holistic pose sequences) | Pose landmarks | Shared 60 clips |
| Trained | X3D RGB | RGB video | 396 held-out clips |
| Trained | X3D Skeleton 3D | 3D skeleton | 250 held-out clips |

**Shared 60-clip set:** 5 clips per class from players p9, p18, p27, p36, p45 and p55. Each clip was renamed to a random 8-character ID so the filename could not reveal the label. All three frontier models and the Pose CNN classify exactly the same clips, so their results can be compared directly.

## Results

| Model | 12-class accuracy | 4-category accuracy | n |
|---|---:|---:|---:|
| **Pose CNN** | **68%** | **90%** | 60 |
| X3D RGB | 62% | 83% | 396 |
| X3D Skeleton 3D | 53% | 76% | 250 |
| Claude Opus 5.5 | 52% | 80% | 60 |
| ChatGPT GPT-6 | 37% | 87% | 60 |
| Gemini Hydrogen | 27% | 72% | 60 |

### Key findings

- **Body motion finds the shot family.** Every model scores much higher on the broad category than on the exact stroke. Pose CNN places all 30 groundstrokes and all 15 serves in the correct family.
- **Serve subtypes are the hardest boundary.** Slice serve has the lowest mean recall across models (17%), followed by flat serve (33%). Every model stays below 50% recall on both.
- **Both model families merge the three serve types into one cluster.** Flat vs slice serve is the hardest pair overall (32% mean pairwise confusion), followed by kick vs slice (27%).
- **Pose helps on body-defined strokes, not on spin-defined ones.** The three frontier models share 15 exact-class failures. Pose CNN fixes 7 of them, including 3 of 4 backhand volleys and a smash that every frontier model called a serve. It fixes **none** of the 4 shared slice-serve failures.
- **Frontier models lose track of the body side.** They make more forehand-vs-backhand errors than the trained models do.

### Limitations

- **Small shared sample:** with 5 clips per class, one clip moves class recall by 20 points.
- **Different test sets:** X3D models are scored on their own held-out splits, so ranking them against the 60-clip models is indicative, not exact.
- **No ball in THETIS:** the cues that separate serves (spin, trajectory, bounce) are missing. Serve-type accuracy may be limited by the data, not by the models.
- **Single run:** each frontier model was evaluated with one prompt and one run.

## Next phase: Unity proof of concept

We extracted MediaPipe Holistic 3D landmarks from difficult THETIS serves and replayed the motion in **Unity**, viewing each serve from the front, the side and a three-quarter overhead angle. This shows that real THETIS motion can be moved into a controllable 3D environment. We have not yet generated enough Unity data to retrain the models.

Planned work:

1. Vary camera angle systematically.
2. Add controlled ball flight and spin.
3. Export labelled sequences, then retrain and evaluate the models on the current failure cases, with and without the ball cue.

## Repository structure

```
vision-tennis/
├── index-tennis.html        # Final presentation deck (open in a browser)
├── serve-deck.html          # Earlier, shorter draft of the deck
├── serve-deck-long.html     # Extended draft with sports-analytics background
├── media/                   # Video clips and images used in the slides
│                            #   (THETIS serves, line-calling clips, Unity renders)
├── raw/                     # Raw model outputs
│   ├── model_comparison.csv       # Claude vs GPT predictions on the 60 clips
│   ├── gemini_preds.csv           # Gemini predictions
│   ├── pose_cnn_preds.csv         # Pose CNN predictions
│   └── confusion_matrix_x3d_*.pdf # X3D RGB and skeleton confusion matrices
└── output/                  # Cleaned data and analysis
    ├── predictions_tidy.csv                         # All predictions in long format
    ├── confusion_long.csv                           # Confusion counts in long format
    ├── tennis_classification_results_consolidated.xlsx
    └── full_analysis/
        ├── data/      # Per-model, per-class and pairwise metrics (CSV + JSON summary)
        ├── figures/   # Accuracy, recall heatmap, confusion and recovery charts
        └── tennis_classification_full_analysis.xlsx
```

## Viewing the presentation

Open `index-tennis.html` in a modern browser. No build step or server is needed. Keep the `media/` folder next to the HTML file so the videos load.

| Key | Action |
|---|---|
| `←` `→` / `Space` | Previous / next slide |
| `O` | Slide overview |
| `N` | Speaker notes |
| `F` | Fullscreen |

You can also drag an image or video onto any media slot while the deck is open to replace it. The browser remembers the replacement.

## Authors

- **Salim Oyinlola**
- **Kayode Makinde**

## License

Released under the [MIT License](LICENSE).
