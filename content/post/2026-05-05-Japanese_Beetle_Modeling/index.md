---
title: "Using a Degree-Day Model to Forecast Adult Japanese Beetle Activity"
author: "Bhim Chaulagain"
date: 2026-05-05T21:13:14-05:00
categories: ["Agronomic Intelligence Lab"]
tags: ["Japanese beetle", "degree-day model", "insect phenology", "pest forecasting", "crop scouting", "agronomic analytics"]
math: false
summary: "A practical walkthrough of using the Ebbenga et al. degree-day model to forecast adult Japanese beetle activity and improve scouting timing."
description: "A practical walkthrough of using the Ebbenga et al. degree-day model to forecast adult Japanese beetle activity and improve scouting timing."
image:
  caption: ""
  focal_point: "Center"
  preview_only: true
---

Last year, Japanese beetles hit the rapeseed vegetables in my backyard hard. After seeing that damage up close, I wanted to learn more about the pest. I wanted to know whether I could anticipate adult activity early enough to scout better and respond before damage built again.

That question pushed me into the literature. I looked through papers on Japanese beetle phenology, broader modeling references, and the USPEST.ORG white paper. What became clear quickly was that these sources were not all trying to answer the same question. Some were aimed at broad life-cycle or climate-suitability questions, while others were much closer to the practical forecasting problem I cared about: if I saw meaningful beetle pressure last season, can I use weather this season to anticipate adult activity earlier?

That is what led me to the paper by Dominique N. Ebbenga, A. A. Hanson, E. C. Burkness, and W. D. Hutchison, *A degree-day model for forecasting adult phenology of Popillia japonica (Coleoptera: Scarabaeidae) in a temperate climate*. There are many useful papers on Japanese beetle phenology, but I use this one as the main foundation because it is directly focused on forecasting adult trap-catch activity. I also looked at the USPEST.ORG white paper because it is useful for broader phenology and climate-suitability thinking, but I do not want to blur those purposes into one model. My goal here is simpler: use seasonal weather data to build an earlier scouting signal for adult Japanese beetle activity.

## Why I Started With This Paper

The paper addresses a real operational gap. Japanese beetle is a major invasive pest in turf, ornamentals, and several agricultural crops, but traps only tell us that beetles are already active. Ebbenga et al. aimed to build an early-warning degree-day model that links ambient temperature accumulation to adult trap-catch phenology under temperate Midwest conditions.

For this post, my immediate goal is narrower: I want a model that helps me estimate when adult beetle activity is becoming important enough to scout and respond. That is why I am leaning on Ebbenga et al. as the core paper here. It is focused on forecasting adult trap-catch from field observations and weather data, rather than trying to simulate the entire life cycle from eggs to larvae to pupae in one model.

## What The Paper Actually Did

The study used field data from four Minnesota locations across 2019-2021. Commercial traps with semiochemical lures were used to monitor adult beetle activity, and daily temperature data were paired with trap-catch observations. The model-building workflow followed Hanson et al.'s concordance-correlation-based approach:

1. split 12 site-years into 6 development and 6 validation datasets,
2. iterate across start dates, lower thresholds, upper thresholds, and degree-day calculation methods,
3. fit logistic regression for cumulative proportion trap-catch against accumulated degree-days,
4. rank candidate models using the concordance correlation coefficient, with AIC used as a secondary fit check across the full emergence curve.

The paper reports that model development evaluated `7,392` parameter combinations:

- start dates of January 1, February 1, March 1, and April 1,
- lower thresholds from `0` to `15 C`,
- upper thresholds from `20` to `37 C` in `0.56 C` increments,
- both a simple average method and a half-day sine-wave method.

That comparison is worth keeping in mind. The paper did not begin by assuming the simple method was automatically best. It tested both a simple average degree-day approach and a half-day sine-wave approach, then selected the model that performed better for its forecasting objective. So it is fair to say the paper gives us a taste of both methods, even though the implementation I keep in the main code path follows the one that was ultimately selected.

The paper selected the following degree-day model specification for forecasting `10%` trap-catch:

- biofix date: `January 1`
- method: `simple average degree-day method`
- lower threshold: `15.0 C`
- upper threshold: `21.7 C`

For that selected model, the paper reported:

- development CCC: `0.899`
- validation CCC: `0.785`
- validation `r = 0.837`
- validation `A = 0.938`
- mean error for predicted minus observed 10% trap-catch dates:
  development `-1.5 d`, validation `-1.5 d`

The fitted log-logistic coefficients reported for that model were:

- intercept: `-43.34`
- slope: `7.41`

## The Exact Core Model

The paper's daily degree-day model is not the same as a generic "average temperature above base" shortcut that adjusts `Tmax` and `Tmin` independently before averaging. The authors explicitly say they used the McMaster and Wilhelm simple average interpretation they call `method 1`, and they distinguish it from the alternative `method 2`.

That means the paper-facing daily thermal-time calculation is based on the daily mean:

```text
Tavg = (Tmax + Tmin) / 2
```

with thresholds applied to the mean:

```text
DTT = 0, if Tavg < 15.0
DTT = Tavg - 15.0, if 15.0 <= Tavg <= 21.7
DTT = 21.7 - 15.0, if Tavg > 21.7
```

That daily value is then accumulated from the biofix date of January 1.

Once cumulative degree-days `D` are available, the paper fits proportion trap-catch with a log-logistic regression:

```text
proportion = exp(s * ln(D) + i) / (1 + exp(s * ln(D) + i))
```

where:

- `D` is cumulative degree-days in Celsius,
- `s = 7.41`,
- `i = -43.34`.

This lets us estimate not just one threshold date, but the whole fitted emergence curve in the paper's field-based adult trap-catch sense.

## A Brief Note On The Sine-Wave Method

I still think it is useful to give a taste of the second method the paper evaluated. Ebbenga et al. did not only test a simple average degree-day approach. They also tested a half-day sine-wave approach during model selection.

That matters because it shows the simple model was chosen after comparison, not by default. In the end, the paper selected the simple average model for this forecasting target, so that is the one I keep in the main implementation path. But I also included a half-day sine-wave function in the companion Python script so it is possible to compare the two approaches side by side.

In other words:

- the simple average model is the main model used in this post,
- the half-day sine-wave model is included as a comparison tool,
- the paper's final forecasting target still comes from the selected simple model.

## What The Paper Says About Key Forecast Targets

Table 4 in the paper gives rounded degree-day values associated with several trap-catch proportions for the selected simple model:

- `10%` trap-catch: `257 C degree-days`
- `25%`: `298`
- `50%`: `346`
- `75%`: `401`
- `90%`: `465`

Operationally, `257 DDC` is the important early-warning number because the paper frames `10% trap-catch` as the target useful for alerting growers before peak beetle activity.

## A Prediction Example For This Season

The way I would use this for my own situation is to pair the model with daily weather from a nearby station where I saw beetle damage last year. To make that concrete, I pulled a 2026 local weather example and accumulated degree-days from `January 1` using the paper's selected thresholds of `15.0 C` and `21.7 C`.

As of `May 15, 2026`, the local example reached about `222.9 DDC` under the selected simple average method. That is still below the paper's `257 DDC` benchmark for `10% trap-catch`. If I plug `222.9` into the fitted equation, the predicted trap-catch proportion is only about `3.6%`. So in that example, I would read the season as moving in the right direction for beetle activity, but not yet at the paper's early-warning threshold.

Here I need to be careful. The paper's `257 DDC` threshold comes from Minnesota field data, so I should treat it as a starting benchmark, not as a guaranteed local threshold. But even with that caveat, this kind of calculation is still useful. It gives me a weather-based way to judge whether the season is still early, getting close, or entering the part of the season when adult activity may begin to matter more.

Because my motivation comes from what I saw last season, this is how I would frame the prediction for this season: not as a claim that I know exactly how many beetles will be on each plant, but as an attempt to identify when local weather has become favorable enough that I should expect meaningful adult activity to begin building.

That distinction matters. The Ebbenga model predicts the expected proportion of seasonal adult trap-catch, not an absolute beetle count per plant or per garden bed. Still, that kind of timing signal can be very useful. If cumulative degree-days in the season are moving toward the paper's benchmark, I can treat that as a practical sign that pressure may be building and that I should monitor my plants more closely.

In the Python implementation, I can use `proportion_trap_catch_from_degree_days` to convert cumulative degree-days into predicted trap-catch proportion, and `predict_first_date_for_proportion` to estimate when a target such as `10%` is first reached.

## Python Implementation

The companion Python script follows that paper-facing structure directly:

```python
def calculate_daily_degree_days_method1(
    tmax_c,
    tmin_c,
    lower_threshold_c=15.0,
    upper_threshold_c=21.7,
):
    tavg_c = (float(tmax_c) + float(tmin_c)) / 2.0

    if tavg_c < lower_threshold_c:
        return 0.0
    if tavg_c > upper_threshold_c:
        return upper_threshold_c - lower_threshold_c
    return tavg_c - lower_threshold_c
```

and the fitted trap-catch curve is implemented as:

```python
def proportion_trap_catch_from_degree_days(
    cumulative_degree_days_c,
    intercept=-43.34,
    slope=7.41,
):
    linear_term = slope * math.log(cumulative_degree_days_c) + intercept
    exp_term = math.exp(linear_term)
    return exp_term / (1.0 + exp_term)
```

For comparison, the script also includes a `calculate_daily_degree_days_half_day_sine` function so the second candidate method from the paper can be explored without replacing the selected simple model.

That keeps the implementation aligned with the model I actually want to discuss in this post, while still preserving a place to compare the alternative method the paper evaluated.

## What This Model Can Tell Me For This Season

For my own use, the value of this model is that it gives me an earlier weather-based signal than simply waiting until damage is obvious again. If I use this season's weather and see cumulative degree-days moving toward the paper's `257 DDC` benchmark, that gives me a reason to start watching more carefully before I am caught off guard.

At the same time, I do not want to claim more than the model can really tell me. It does not predict the exact number of beetles on a particular plant in my backyard, and it does not simulate the full biology from eggs to larvae to pupae. It is better understood as a forecasting tool for adult activity, especially for the timing of when that activity may begin to matter.

That still fits my purpose well. I am not trying to build a complete Japanese beetle population simulator in the first step. I am trying to use last season's experience and this season's weather to build a better early-warning signal for scouting and management.

## Practical Use Beyond The Paper

Beyond the paper, a practical workflow is straightforward:

1. start accumulating daily degree-days on `January 1`,
2. use daily `Tmax` and `Tmin` weather data,
3. compute daily thermal time with the selected simple average method,
4. accumulate cumulative degree-days,
5. flag `257 DDC` as the early-warning point for `10%` trap-catch,
6. optionally compute the whole fitted trap-catch proportion curve using Equation 1.

That workflow is practical, but it is still important to remember that the paper was developed from Minnesota field data. In my case, the natural next step would be to use what I observed last season as a starting point, run this season's weather through the model, and then compare the predicted early-activity timing against what I actually see in the field this year. That season-to-season comparison is what would tell me whether the Minnesota-based benchmark is a good fit for my situation or whether I need to adjust my expectations.

## Takeaway

The strongest way to use this Japanese beetle lesson is not to say "here is a general beetle model." It is to say:

1. Ebbenga et al. produced a field-based adult trap-catch forecasting model,
2. the selected model was a simple average degree-day approach with `January 1`, `15.0 C`, and `21.7 C`,
3. the fitted adult trap-catch curve used intercept `-43.34` and slope `7.41`,
4. `257 C degree-days` corresponds to `10%` trap-catch and serves as the early-warning target,
5. broader phenology or climate-suitability models are useful too, but they should be kept clearly separate unless they are being implemented and validated on their own terms.

That framing fits what I am actually trying to do here: use the literature to build a useful, biologically grounded way to anticipate adult Japanese beetle activity.
