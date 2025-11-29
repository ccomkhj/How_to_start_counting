# How to Start Building an Object Counting System

I’m writing this as a quick starting guide because I’ve received a few requests about how to build a video‑based counting system (people, vehicles, etc.).

At a high level, **the core is always: detection + tracking + counting logic**.

---

## 1. Basic Pipeline

1. **Object Detection (per frame)**
   For each frame *t*, run an object detector and find all target objects (e.g., people, cars).

   * In frame *(t)*, your target objects are detected.
   * In frame *(t + n)*, your target objects are detected again.

2. **Tracking (across frames)**
   Between frames *(t)* and *(t + n)*, match detections that belong to the **same physical instance**.
   The goal is to assign a stable ID to each object over time.

3. **Counting Logic (on tracks)**
   Once objects are tracked and have IDs, you define *how* to count based on their trajectories, for example:

   * Line crossing (e.g., count when a track crosses a virtual line from A → B).
   * Zone in/out (e.g., entering or leaving a region).
   * Dwell time (e.g., how long an object stays in an area).

4. **Aggregation & Output**
   Aggregate counts per time window / zone and export them to your analytics, database, or dashboard.

---

## 2. Key Success Factors

To make the system work well in production, focus on:

* **Reliable object detection (especially low false positives)**
  For counting, false positives are very painful since they directly inflate your numbers. It’s often better to sacrifice a bit of recall if it significantly reduces false positives.

* **Tracking algorithms that fit your compute budget**
  Choose tracking methods based on:

  * Where you run them: cloud vs edge device.
  * Mode: real‑time streaming vs offline batch.
  * Scene complexity: occlusion, crowd density, camera motion.

* **Understanding the business requirements**
  Clearly define:

  * What exactly you want to count (which classes, directions, zones).
  * Acceptable error rate (e.g., ±5%, ±10%).
  * Latency requirements (real‑time vs end‑of‑day reports).
    Then derive camera placement, resolution, detector/tracker settings, and counting logic from these requirements.

---

## 3. Recommended Algorithms / Models

### 3.1 Detection

**Robust working versions that I use and can guarantee:**

* **FCOS** – a fully convolutional, anchor‑free one‑stage detector.
* **ATSS‑based setups (e.g., FCOS + ATSS)** – using ATSS (Adaptive Training Sample Selection) to improve sample assignment and robustness.

These have been reliable in practice for me in real projects.

**State‑of‑the‑art (I didn’t implement, but promising from academia):**

* Recent **transformer‑based detectors** (e.g., DETR and its improved variants, DINO, etc.).
  I *didn’t implement these myself*, but they are very promising in the research community and may outperform the above in some scenarios.

---

### 3.2 Tracking

**Robust working versions that I use and can guarantee:**

* **BoT‑SORT** – a strong multi‑object tracker combining motion, appearance features, and camera motion compensation.
* **ByteTrack** – a tracker that effectively uses both high‑ and low‑confidence detections, very strong in practice.

These are my go‑to choices for building stable counting pipelines.

**State‑of‑the‑art (I didn’t implement, but promising from academia):**

* **OC‑SORT / Deep OC‑SORT and similar recent trackers** that focus on better handling of occlusions and non‑linear motion.
  I *didn’t implement these myself*, but they show promising results in academic benchmarks.

---

## 4. Minimal “First Version” Recipe

If you’re starting from scratch, a simple but solid first version could be:

1. Use **FCOS (+ ATSS)** for detection.
2. Use **BoT‑SORT** or **ByteTrack** for tracking.
3. Implement:

   * Detection per frame.
   * Tracking over time (consistent IDs).
   * A basic line‑crossing or zone‑based counting rule.
4. Tune:

   * Detection threshold to reduce false positives.
   * Tracking parameters for your specific camera angles and motion.
   * Counting logic in terms of business definitions (“what exactly counts as an entry/exit?”).

Once this is stable, you can iterate with better models (including newer academic ones) and more complex logic, but the core idea remains the same: **detect → track → count on tracks**.
