# Correct the drift: data guide

All data is **synthetic**. It was generated to behave like real low Earth orbit (LEO) satellites, so you can work without any aerospace background.

## Files

| File | Rows | What it is |
| --- | --- | --- |
| `train.csv` | 20,440 | 40 satellites, 7 prediction runs each, with the true position and the error |
| `test.csv` | 16,060 | Prediction runs you have not seen. Same columns, but no truth and no error |
| `space_weather.csv` | 385 | Hourly solar activity for the whole 16-day period |
| `sample_submission.csv` | 16,060 | The exact format we score. Replace the zeros with your predictions |

## How the data is organised

A **prediction run** (`arc_id`) starts at an epoch (`epoch_utc`) and predicts the satellite's position every hour for 72 hours (`hours_since_epoch` 0 to 72). Each satellite has several runs starting on different days.

For every hour you get:

- where SGP4 (Simplified General Perturbations 4, the standard public prediction model) says the satellite is
- in `train.csv` only: where it actually was, and the difference between the two

## Columns

| Column | Unit | Meaning |
| --- | --- | --- |
| `sat_id`, `arc_id` | | Satellite and prediction-run identifiers |
| `epoch_utc`, `time_utc` | UTC | Start of the run, and the time of this row |
| `hours_since_epoch` | hours | How far into the prediction this row is |
| `mean_altitude_km` | km | Average height above Earth at the start of the run |
| `inclination_deg` | degrees | Tilt of the orbit relative to the equator |
| `eccentricity` | | How far from circular the orbit is (0 is a perfect circle) |
| `raan_deg`, `arg_perigee_deg`, `mean_anomaly_deg` | degrees | Angles describing the orbit's orientation and where the satellite is along it at the start of the run |
| `bstar` | 1/Earth radii | SGP4's drag term: how strongly SGP4 thinks drag slows this satellite |
| `f107_at_epoch` | solar flux units | Solar activity at the start of the run. More activity heats the upper atmosphere and increases drag |
| `ap_at_epoch` | index | Geomagnetic activity at the start of the run. Same effect, and it spikes during solar storms |
| `sgp4_x_km`, `sgp4_y_km`, `sgp4_z_km` | km | SGP4's predicted position, in an Earth-centred frame |
| `sgp4_vx_kms`, `sgp4_vy_kms`, `sgp4_vz_kms` | km/s | SGP4's predicted velocity, same frame |
| `true_x_km`, `true_y_km`, `true_z_km` | km | Train only. The true position, with about 30 m of tracking noise |
| `err_radial_km` | km | Train only. Error in the up/down direction (true minus SGP4) |
| `err_along_km` | km | Train only. Error along the direction of travel. This is usually the biggest |
| `err_cross_km` | km | Train only. Error sideways, out of the orbit plane |

The three error columns are the target. Predicting them and subtracting from SGP4 gives a corrected position.

## Submission

A CSV with exactly these columns, one row per row of `test.csv`:

`arc_id, hours_since_epoch, pred_err_radial_km, pred_err_along_km, pred_err_cross_km`

We score the distance between the true position and your corrected position, compared with SGP4 alone, broken down by how far ahead the prediction is.

## A note on `space_weather.csv`

It covers the full period, including the times your test runs predict into. In real operations you would not know future solar activity at the start of a prediction. Whether and how you use this file is your call. Say what you did and why in your README.
