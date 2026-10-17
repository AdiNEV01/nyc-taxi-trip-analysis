# IDVE Assignment 2: Task Allocation (v2)

This document divides the assignment across four people. Each person owns a question or group of questions, produces the required analysis, and shares outputs that support other parts of the assignment.

**Assignment deadline:** 28 October 2026 (internal target: submit-ready by 27 October, see Timeline).

**Total marks:** 63 (Person 1: 20, Person 2: 12, Person 3: 9, Person 4: 22)

---

## Group-wide conventions (agree on these first)

- **Data:** use only `train.csv` from the NYC Taxi Trip Duration dataset, as the assignment specifies.
- **Random seed:** `SEED = 42` everywhere (sampling, t-SNE, UMAP, clustering).
- **Timestamps:** the data is in NYC local time. Do no timezone conversion.
- **Coordinate order warning:** the PDF labels the Q5 coordinates and the NYC centre point as "(long, lat)", but the values are actually **(lat, long)**. For example, JFK is `(40.647929, -73.777813)`, where 40.6 is latitude and -73.8 is longitude. Everyone must load them as `lat, lon`, otherwise no trips will match.
- **Distance:** use the provided `haversine_np` function, so distances are consistent across the group.
- **Shared schema:** Person 1 publishes one cleaned file (Parquet or CSV) with these column names. Nobody renames columns after this.

| Column | Meaning |
|---|---|
| `id` | original trip id (join key) |
| `pickup_datetime`, `dropoff_datetime` | parsed datetimes |
| `pickup_latitude`, `pickup_longitude`, `dropoff_latitude`, `dropoff_longitude` | original coordinates |
| `trip_duration` | seconds |
| `trip_distance_km` | haversine distance |
| `avg_speed_kmh` | `trip_distance_km / (trip_duration / 3600)` |
| `pickup_hour` | 0-23 |
| `pickup_dow` | 0 = Monday ... 6 = Sunday |
| `pickup_date` | date only (useful for holidays) |
| `is_weekend` | boolean |

- **Airport flag file:** Person 3 publishes `airport_flags` keyed on `id` (see Task 3).
- **Cleaning rules:** Person 1 documents every rule (threshold plus reason) in a short list that is pasted into the report. Others must not apply extra filters silently. If an analysis needs one, state it in that section.

---

## Task 1: Data Cleaning, Feature Generation, and Time-Based Analysis

**Owner:** Person 1
**Assignment questions:** Questions 1, 2, and 3
**Marks:** 20

### Description

Prepare the NYC taxi training dataset, generate additional trip features, and investigate how taxi-trip patterns vary over time.

### What needs to get done

- Identify outliers and justify whether to remove them, considering trip duration, distance, speed, passenger count, and coordinates outside NYC.
- Generate features: trip distance, day of week, average trip speed (plus the extra columns in the shared schema).
- Which day of the week has the most pickups? Support with plots.
- Most popular pickup hour for each day of the week; plot hourly distributions and discuss possible explanations.
- Compare weekday and weekend patterns and explain what accounts for the differences.
- Investigate pickup patterns on the 2016 dates of the five holidays:
  - Martin Luther King Jr. Day: Mon 18 January
  - Valentine's Day: Sun 14 February
  - St. Patrick's Day: Thu 17 March
  - Easter: Sun 27 March
  - Memorial Day: Mon 30 May

  Compare each against typical days of the same weekday, and state whether the pattern changes.
- Plot average trip speed by time of day and identify when trips are fastest.

### Deliverables

- Cleaning code and written justification for each removal rule.
- Feature-generation code.
- Plots and written findings for all parts of Question 3.
- **Cleaned dataset in the shared schema, published early (see Timeline).**

---

## Task 2: Geographic Analysis and Pickup Hotspots

**Owner:** Person 2
**Assignment question:** Question 4
**Marks:** 12

### Description

Explore where pickups occur across NYC and identify hotspots in two specified time periods.

### What needs to get done

- Heatmaps (not scatter plots) for weekdays vs weekends, and morning vs evening. Define and justify the hours.
- Describe and interpret the geographic patterns.
- Analyse pickups for **Friday 23:00 to Saturday 02:00** (this window crosses midnight, so handle the day boundary carefully) and **Thursday 17:00 to 20:00**.
- Use DBSCAN (haversine metric, ball tree) to find hotspots in each period.
- **Motivate `eps` and `min_samples` explicitly.** The assignment suggests "at least 15 pickups in an hour" and 50 or 100 m. The data spans about 26 weeks, so pooling every Thursday means a 3-hour window contains roughly 26 × 3 hours of pickups. Choose one approach and explain it:
  - scale `min_samples` to the pooled number of hours (for example 15 per hour × total hours), or
  - run DBSCAN on a single representative date or a per-week average, so that 15 per hour applies directly.

  Show that results are reasonable (for example, try two parameter settings and compare cluster counts).
- Plot clusters on a map (folium or similar) and report the number of clusters found for each period.

### Deliverables

- Required heatmaps and interpretation.
- DBSCAN code with parameter justification.
- Maps for both time periods, cluster counts, written observations.

---

## Task 3: Airport Travel-Time Analysis (and airport flags)

**Owner:** Person 3
**Assignment question:** Question 5
**Marks:** 9

### Description

Estimate average taxi travel times between the Empire State Building area and JFK and Newark airports, and compare how they vary by time of day.

### What needs to get done

- Use the supplied coordinates, remembering they are **(lat, lon)** despite the PDF label.
- Choose and justify a radius around each location (for example, compare GPS noise, the size of the terminal area, and the size of the ESB area, and show the sensitivity of the result to the radius).
- **Direction matters:** the question asks about travel from the Empire State Building *to* the airports. Primary analysis: pickup near ESB and dropoff near the airport. State clearly whether the reverse direction (airport to ESB) is included, shown separately, or excluded.
- Calculate average travel time by time of day for JFK and Newark, and plot it. Include the number of trips behind each hourly point, since some hours may have few trips.
- Compare the two airports and discuss possible explanations (route, tolls and bridges/tunnels, congestion, distance).
- Explain the limitations of inferring a journey from GPS proximity alone.

### Handoff to Person 4: airport flag file

Besides the Q5 analysis, produce a file `airport_flags` with columns `id`, `near_jfk_pickup`, `near_jfk_dropoff`, `near_ewr_pickup`, `near_ewr_dropoff`, and `airport_trip` (true if any of the four). This must cover **all cleaned trips**, so Person 4 can join it to the 20,000-trip sample. Note that the sample may contain few airport trips; report the count.

### Additional workload (rebalancing)

Task 3 is the lightest section, and Task 4 the heaviest, so Person 3 also takes on part of Question 6 (see Task 4): the UMAP parameter grid (Q6.4) and the cluster maps for Q6.5. Person 3 remains the owner of the written answer for Q5, and Person 4 owns the written answers for Q6, using Person 3's outputs.

### Deliverables

- Filtering and travel-time code.
- Radius justification and sensitivity check.
- Plots of travel time by time of day for JFK and Newark.
- Written comparison and limitations.
- `airport_flags` file for the whole cleaned dataset.

---

## Task 4: Dimensionality Reduction and Trip Clustering

**Owner:** Person 4 (with Person 3 supporting, see above)
**Assignment question:** Question 6
**Marks:** 22

### Description

Use PCA, t-SNE, and UMAP to explore whether taxi trips form natural groups, cluster trips in the reduced spaces, and compare the three methods.

### What needs to get done

- **Q6.1:** take a reproducible random sample of 20,000 trips from the cleaned dataset (seed 42). Build a feature matrix with at least pickup and dropoff coordinates, trip distance, duration, average speed, pickup hour, and day of week. Encode pickup hour cyclically (`sin(2πh/24)`, `cos(2πh/24)`) and scale features. Justify the feature, encoding, and scaling choices (for example, why standard scaling, and how skewed variables such as duration and distance are handled). **Save the matrix and sample ids so Person 3 can run UMAP on exactly the same data.**
- **Q6.2 PCA:** scree plot plus cumulative variance; number of components for 90% variance; first two components coloured by pickup hour, weekday/weekend, and trip distance; interpret both components using loadings.
- **Q6.3 t-SNE:** at least three perplexities (for example 5, 30, 100); the three colourings for each; discuss the effect of perplexity.
- **Q6.4 UMAP (Person 3 can run this):** `n_neighbors` in {5, 15, 50} × `min_dist` in {0.0, 0.5}; the same three colourings; discuss the effect of each parameter.
- **Q6.5 Clustering:** DBSCAN or HDBSCAN in each space.
  - PCA: cluster on the number of components that explains 90% of variance; visualise on the first two components.
  - t-SNE and UMAP: cluster on the best 2D embedding.
  - Motivate parameters (for example, a k-distance plot for `eps`).
  - Report cluster counts and the noise fraction for each method.
  - Profile each cluster using **original, unscaled** features (median distance, duration, speed, pickup hour, weekday/weekend split), map its pickup locations (Person 3 can build the maps), and give each cluster a descriptive name.
- **Q6.6 Comparison:** place the best embedding from each method side by side with the same colouring and discuss **all** of the following:
  - groups of trips that appear consistently across all three methods (including whether airport trips from Q5 form their own group, using the `airport_flags` file);
  - what linear PCA reveals or misses compared to the non-linear methods;
  - local vs global structure preservation, and whether distances between clusters and cluster sizes in each plot can be interpreted;
  - runtime (time each method) and sensitivity to hyperparameters **and to the random seed** (rerun at least t-SNE and UMAP with a couple of different seeds and compare);
  - agreement of the clusters, quantified by the adjusted Rand index for each pair of methods (PCA-t-SNE, PCA-UMAP, t-SNE-UMAP), including how noise points are handled in the ARI;
  - whether clusters found in a t-SNE or UMAP embedding can be trusted as real groups in the original data;
  - a final recommendation of which method to use for exploring this dataset, and why.

### Deliverables

- Reproducible sampling, feature preparation, scaling, and reduction code.
- PCA variance plots, component plots, and loading interpretation.
- t-SNE and UMAP embeddings for all required settings.
- Clustering visualisations, parameter justification, cluster counts, noise fractions, profiles, names, and pickup maps.
- ARI comparisons, method evaluation, and recommendation.

---

## Cross-task sharing (comparisons, not assumed results)

| From | To | What |
|---|---|---|
| Person 1 | Everyone | Cleaned dataset, schema, cleaning rules |
| Person 3 | Person 4 | `airport_flags` file and the airport-trip count |
| Person 4 | Person 3 | Feature matrix and sample ids (for the UMAP runs) |
| Person 2 | Person 4 | Hotspot maps, to compare against the cluster pickup maps (the methods use different settings and objectives, so differences are expected) |
| Person 4 | Person 1 | Embeddings coloured by hour and weekday/weekend, as extra context for Question 3 |
| Person 3 | Person 1 | Travel time by hour, as a concrete example of how trip times vary over the day |

These links are useful comparisons. They are not assumptions that one task's result must appear in another.

---

## Report and integration roles

Assign a name to each role at the first meeting:

- **Data owner:** Person 1 (cleaned data, schema).
- **Notebook integrator:** merges everyone's code into a single notebook with clear question-number headings, then runs it top to bottom on a fresh kernel.
- **Report lead:** assembles the PDF in the NeurIPS LaTeX style, makes sure every figure is referenced and every sub-question is answered in the text.
- **Checker:** goes through the assignment PDF subquestion by subquestion and ticks each off against the notebook and the report (this should be someone other than the report lead).

Each person writes their own sections, plots, and interpretations.

---

## Timeline (suggested; adjust as a group)

| Date | Milestone |
|---|---|
| Sat 17 Oct | Meet; agree on schema, seed, conventions, and roles |
| Mon 19 Oct | Person 1 publishes cleaned dataset + features + cleaning rules |
| Tue 20 Oct | Person 3 publishes `airport_flags`; Person 4 publishes feature matrix and sample ids |
| Sat 24 Oct | First drafts of all sections (code, plots, written answers) |
| Sun 25 Oct | Cross-checks between tasks; fix inconsistencies |
| Mon 26 Oct | Notebook merged and run end to end on a clean kernel; report assembled |
| Tue 27 Oct | Checker pass against the assignment PDF; final fixes; submission-ready |
| Wed 28 Oct | Deadline: upload PDF and notebook to Moodle early in the day |

## Final checklist

- [ ] Only `train.csv` used
- [ ] Coordinates loaded as (lat, lon)
- [ ] Seed fixed everywhere
- [ ] Every subquestion answered in the PDF with its plot attached
- [ ] Notebook headings carry the question numbers
- [ ] Notebook runs top to bottom without errors on a fresh kernel
- [ ] PDF in NeurIPS style
- [ ] Submitted to Moodle (PDF + notebook)
