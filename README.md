# DeepRacer

AWS DeepRacer reward functions and training notes from the 2022 season. Each Python file is a `reward_function` pasted into the DeepRacer console. The engineering notebook records how those functions were trained, evaluated in simulation, and raced on the physical car, mostly on the Monza and Oval tracks.

DeepRacer calls `reward_function(params)` once per step and reinforces actions that return a higher float. These versions start from a reward of `1.0` (or a centerline band) and multiply it down when the car heads the wrong way, drifts wide, goes slow, or leaves the track.

## Repository layout

```
Reward functions/       Console-ready reward_function implementations
engineeringNotebook/    Training notes, evaluation numbers, and race-day log
```

## Reward functions

Copy one file into the DeepRacer reward-function editor. Files that need `math` already import it. Match the action space written in the file header or in the matching notebook entry. The straightness functions were trained with a discrete action space.

Parameter names follow the [DeepRacer reward function inputs](https://docs.aws.amazon.com/deepracer/latest/developerguide/deepracer-reward-function-input.html). The functions in this repo read `waypoints`, `closest_waypoints`, `heading`, `speed`, `steering_angle`, `is_offtrack`, `is_reversed`, `distance_from_center`, and `track_width`.

### Centerline, penalize low speed

[`Reward functions/promoteCenterlinePenalizeLowSpeed.py`](Reward%20functions/promoteCenterlinePenalizeLowSpeed.py)

Pays the car for staying near the center of the track, then cuts that reward when speed is low.

| Distance from center | Reward |
| --- | --- |
| ≤ 10% of track width | 1.0 |
| ≤ 25% of track width | 0.5 |
| ≤ 50% of track width | 0.1 |
| Wider than that | 0.001 |

Speed penalties stack. Under 2 m/s the reward is multiplied by 0.90, under 1.5 m/s by another 0.75, and under 0.5 m/s by another 0.5.

The file header records about 45 minutes of training with steering from −30° to 30° and speed from 0.5 to 3 m/s. Three evaluations: 34% completion in 4.193 seconds, 100% in 12.864 seconds, and 34% in 4.193 seconds.

### Promote straightness

[`Reward functions/PromoteStraighness.py`](Reward%20functions/PromoteStraighness.py)

Compares the car's heading with the direction of the centerline between the closest waypoints. A heading more than 10° off that line halves the reward. On top of that:

- Speed under 1 m/s multiplies the reward by 0.75.
- Off track or reversed sets the reward to near zero.
- A steering angle past 20° while the heading is still aligned multiplies the reward by 0.75, which discourages zig-zag corrections on a straight.

The notebook's first straightness model used a discrete space of 0–2.5 m/s in 3 steps and −30° to 30° in 5 steps. It finished in 11.984 seconds in simulation and about 12 seconds on the physical track. Later copies in the same note used 1–3 m/s and, on one clone, a 4 m/s top speed.

### Promote straightness, tighter heading cuts

[`Reward functions/PromoteStriaghtness2.py`](Reward%20functions/PromoteStriaghtness2.py)

Same heading comparison, with cuts at 7° and 14°. The multipliers are reversed from the usual pattern: a 7–14° error is multiplied by 0.25, and an error past 14° is multiplied by 0.5. The file notes that the conditions were wrong and the model still trained.

### Promote straightness and stay on the track

[`Reward functions/PromoteStraightnessStayOnTrack.py`](Reward%20functions/PromoteStraightnessStayOnTrack.py)

Heading cuts at 15° (×0.7) and 25° (×0.3). If the car is more than 40% of the track width from the center, the reward is multiplied by 0.001, which is the margin-for-error term added after physical-track testing. The header notes a discrete space of −30° to 30° in 5 steps and 0–2.5 m/s in 3 steps, and that the last model trained on this function was fast. The [stay-near-center note](engineeringNotebook/PromoteStraightnessStayNearCenter.md) covers the same idea with heading cuts at 10° and 20°, and records what happened as the top speed was raised from 2 m/s to 4 m/s.

## Engineering notebook

Notes are markdown in [`engineeringNotebook/`](engineeringNotebook/).

| Note | What it covers |
| --- | --- |
| [Getting started](engineeringNotebook/GettingStarted.md) | Naming a model, choosing Monza or the Oval, discrete versus continuous action space, the default car, training for about an hour, and evaluating every model. |
| [Reward function](engineeringNotebook/Reward.md) | Keeping the returned reward in a 0–1 range, returning a tiny default so training converges, and the tradeoffs of an all-wheels-on-track reward versus a centerline reward. |
| [Promote straightness](engineeringNotebook/PromoteStraightness.md) | The 15-action straightness models, including the oval-trained copy that stayed on the physical track (12–14 seconds at 95% speed, 13 laps in 3 minutes with 2 mistakes). |
| [Stay near center](engineeringNotebook/PromoteStraightnessStayNearCenter.md) | Straightness plus a centerline term. The 1–3 m/s clone was reliable at about 12 seconds in simulation and on the car. The 1–4 m/s clone posted a 10.80 second virtual lap and was only about half reliable. |
| [Competition day](engineeringNotebook/CompetitionDay.md) | Morning training, Monza and Oval testing, and race runs. Best Monza result written down: 10 laps and a 14.00 second lap. Oval run 1: 9.5 laps. |

## License

MIT. Copyright 2022 Alex Dickhans. See [LICENSE](LICENSE).
