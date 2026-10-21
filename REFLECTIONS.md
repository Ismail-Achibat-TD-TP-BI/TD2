# REFLECTIONS (TD2)

## Reflection 1
Skipped

## Reflection 2: why the static feed isn't refreshed every 15 minutes
The static GTFS describes the *planned* timetable, which SETRAM only replaces every few months (the current weekday services run from 31 Aug to 30 Oct 2026), so re-downloading it every 15 minutes would produce the same output ~96 times a day. That's wasted compute and needless load on data.gouv.fr for zero new information.

## Reflection 3: auto-typed ID columns and joins
**Prediction:** if one file's `stop_id` is read as numbers and the other's as text, `1001` and `"1001"` aren't equal, so the join would match nothing (or crash), and every stop would end up with zero trips, possibly without any error.

**Compared to the script's comment:** the comment confirms this ("silent int64-vs-object merge failures"). Testing on the real feed showed a nuance. `stop_times.stop_id` became **float64** (blanks turn `1001` into `1001.0`), and `merge()` raised an error, but `isin()` (which this script uses) silently matched 0 of 214,507 stop-times. So the silent failure is real, just not where the comment says. Reading everything with `dtype=str` avoids it, because IDs are labels, not numbers.


## Reflection 4: per-stop vs. per-route logging volume
One snapshot from our pipeline had 4,635 stop-time updates, and 12 weeks of 15-minute snapshots is about 8,000 snapshots, so logging every update would mean roughly 37 million rows (~2 GB), too big for a git-tracked file. One row per route (26 routes) gives about 210,000 rows (~7 MB), roughly 180 times fewer. The cost is location: with per-route averages, a planner can't answer **"where along the route does the delay build up?"** (e.g. is route 5 late everywhere or only after one junction?), which is exactly what's needed to decide where a bus lane or traffic-light priority would help. The average also hides which specific stops are repeatedly skipped, and lets early and late stops cancel out.


## Reflection 5: alternative colour encoding for delay
The current rule paints early buses green, so in our snapshot route 33 (100 s early on average) looks like the best route on the network. But an early bus can be worse for a passenger than a late one: if it leaves before you arrive, you wait for the next one. I'd colour by distance from on-time instead: green inside a punctuality window, then a diverging scale with red for late and blue for early, darker the further from schedule. Because each bar is a route *average* (which already smooths out individual buses), I used a tighter window than the usual per-bus standard (−1/+3 min): ±1 min. A viewer would then see routes 33 and 23 as problems rather than successes, and the colour would point to different fixes: late (red) suggests infrastructure like bus lanes or signal priority, while early (blue) usually means a too-generous timetable that can be fixed cheaply by retiming the schedule. I applied this rule to the dashboard's delay chart.

## Reflection 6: static vs. real-time scheduling mismatch
My Reflection 2 guess covered the first half: the static timetable only changes every few months, so refreshing it every 15 minutes would regenerate identical output ~96 times a day. The real feed adds a nuance: the current weekday services end on 30 Oct, and our monthly job runs on the 1st, so if SETRAM publishes late, the reference stays stale for a month; the right cadence follows the source's publication rhythm (e.g. a weekly check that rebuilds only when the feed has changed), not the calendar. The other direction is worse: GTFS-RT only describes the current moment and keeps no history, so a monthly snapshot would capture one random instant per month (sometimes 3 a.m., sometimes rush hour). It would almost always miss short disruptions like route 5's 32-minute delay, couldn't show daily or weekly patterns, and anything missed is lost forever because there's no archive to go back to. The schedule has to match how fast the source changes, in both directions.

## Reflection 7: sanity check of one delayed route

## Reflection 8: where population misleads as a demand proxy

## Reflection 9: supply metric and a commune where it misleads

## Reflection 10: aggregating stop-level delay events

## Reflection 11: what was answerable today vs. needs history
