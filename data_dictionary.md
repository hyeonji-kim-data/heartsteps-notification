# Data dictionary: decision_points.csv

| column | description | source |
| --- | --- | --- |
| user_id | User ID | user.index |
| decision_id | Decision point ID within a user | decision.index |
| decision_time_local | Decision time in local time | sugg.decision.utime + sugg.gmtoff (seconds) |
| study_day | Days since the user's first decision point (0 = first day) | decision_time_local |
| study_week | study_day // 7 (0 = first week) | study_day |
| slot | Scheduled daily time slot, 1 to 5 | sugg.select.slot |
| slot_name | morning commute, lunch, mid-afternoon, evening commute, after dinner | slot |
| weekday | Day of the week (local time) | decision_time_local |
| is_weekend | 1 = Saturday or Sunday | weekday |
| available | 1 = connected, not snoozed, not in transit | avail |
| sent | 1 = notification sent | send |
| send_type | active, sedentary, none, or unknown (sent, type not recorded) | send.active, send.sedentary |
| travel_day | 1 = travel day excluded by the study | decision.index.nogap is missing |
| is_prefetch | 1 = context taken 30 minutes before the decision | is.prefetch |
| location | home, work, or other (missing if unknown) | dec.location.category |
| weather | Weather condition (missing if unknown or a system error) | dec.weather.condition |
| weather_group | clear, cloudy (incl. fog and haze), or precipitation | weather |
| temperature_c | Temperature in Celsius (-1024 set to missing) | dec.temperature |
| precip_chance | Chance of precipitation in 60 minutes, 0-100 (-1 and unknown set to missing) | dec.precipitation.chance |
| screen_on | 1 = an app was open at the decision point (missing if not connected) | front.end.application |
| hours_since_phone_use | Hours since the last phone use before the decision point | sugg.device.since |
| steps_pre30 | Jawbone steps in the 30 minutes before | jbsteps30pre |
| steps_post30 | Jawbone steps in the 30 minutes after (main outcome) | jbsteps30 |
| steps_pre30_zero | steps_pre30 with missing set to 0 | jbsteps30pre.zero |
| steps_post30_zero | steps_post30 with missing set to 0 | jbsteps30.zero |
| log_steps_pre30 | log(steps_pre30 + 0.5) | steps_pre30 |
| log_steps_post30 | log(steps_post30 + 0.5) | steps_post30 |
| log_steps_pre30_zero | log(steps_pre30_zero + 0.5) | steps_pre30_zero |
| log_steps_post30_zero | log(steps_post30_zero + 0.5) | steps_post30_zero |
| response | good, bad, snoozed, no_response, or missing | response |
| engaged | 1 = good, bad, or snoozed; 0 = no_response; missing if not sent or not recorded | response |
