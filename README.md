# Hospital Appointment No-Shows: Monthly Reporting Pack

20.2% of appointments are missed. Appointments booked more than 30 days ahead are missed at 33.4%, against 4.7% for same-day bookings, across 110,323 appointments spanning April–June 2016.

## The question

Which appointments get missed, and what could the clinic do about it?

## The data

- Source: [Medical Appointment No Shows dataset, Kaggle](https://www.kaggle.com/datasets/joniarroba/noshowappointments?select=KaggleV2-May-2016.csv) (~110k rows overall)
- Row count: 110,323 appointments (April, May, and June 2016, combined)
- Date range: April–June 2016
- Four data traps found in the raw file:
  1. `No-show = Yes` actually means the patient **did not attend** (the column name is inverted from what you'd expect).
  2. Negative age values in the data (e.g. `Age = -1`), which are invalid and need to be filtered or flagged.
  3. Misspelled and miscoded columns in the source: `Hipertension` and `Handcap` are misspelled, and `Handcap` is stored as an integer severity code (0–4) rather than a clean yes/no flag.
  4. Unrealistic age outliers — a handful of rows record `Age = 115`, which is technically possible but extreme enough to flag and check rather than take at face value.

## What I built

- Power Query load from a folder, so dropping a new month's CSV into `data/` and refreshing pulls it straight into the model.
- A pivot layer summarizing no-show rate by lead-time band, age band, weekday, and neighbourhood.
- A one-page Summary sheet with KPIs and a slicer for month.
- A reconciliation check that compares the raw row count/total against the summary layer and flags a mismatch — this actually caught two real bugs during testing: a hardcoded pivot cell reference that broke when a new month's row was inserted, and a column-mapping issue that silently nulled out June's No-show values on import.
- Verified with a live test: manually recomputing all KPIs for a new month took 54 minutes; dropping the file in and clicking Refresh All took 15 seconds.
- A Tableau Public dashboard for a no-download view: **[Hospital Appointment No-Shows Dashboard](https://public.tableau.com/app/profile/tanusnigdha.biswas/viz/HospitalAppointmentNo-ShowsDashboard/Overview)**

## Findings

1. **Overall no-show rate is 20.2%** across 110,323 appointments (April–June 2016) — fairly stable across the three months (19.6% in April, 20.8% in May, and slightly lower in June).
2. **Lead time is the strongest single driver of no-shows.** The rate rises from 4.7% for same-day bookings to 33.4% for bookings made more than 30 days out — roughly a 7x increase.
3. **Age matters, but in the opposite direction you might expect.** Young adults are the worst attenders (24.5% no-show), while older adults and elderly patients are the best (~16.2–16.3%). Weekday and repeat-visit history, by contrast, showed almost no effect (weekday range 19.6–21.7%; first-timers 19.7% vs repeat patients 20.8%).
4. **Neighbourhood effects are real but secondary to lead time.** Among neighbourhoods with 500+ appointments, rates range from 15.5% (Santa Martha) to 29.1% (Santos Dumont) — about a 2x spread, with the worst clinics running roughly 1.4x the city average.
5. **The SMS reminder effect is a Simpson's Paradox.** In the raw comparison, patients who received an SMS reminder missed appointments at a higher rate than those who didn't (28.3% vs 17.7%) — reminders look counterproductive. But within every single lead-time band, SMS recipients had a *lower* no-show rate than non-recipients (e.g. Over-30-days: 37.6% without SMS vs 29.9% with SMS). Reminders were simply concentrated on long-lead-time, high-risk bookings, which drove the misleading raw comparison. This is observational data, so it shows the raw comparison is misleading — it doesn't prove reminders reduce no-shows; that would need a randomized test.

![Summary dashboard](images/summary.png)

## Recommendations

1. Cap routine bookings at a shorter lead time where clinically possible.
2. Send reminders closer to the appointment date rather than at the time of booking.
3. Overbook the slots with the highest predicted no-show rate (e.g. long-lead-time, younger-age-band slots).

What I'd test first: reminder timing (send a second reminder 24–48 hours out) via a small randomized trial, since the current data can only show correlation, not causation.

## Limitations

- Three months of appointment data, one city.
- No data on *why* patients missed appointments.
- No information on transport access or cost as barriers.
- Observational data only — no randomized comparison, so causal claims about reminders are not supported by this alone.

## How to reproduce

1. Put the monthly appointment CSVs in `data/`.
2. Open the workbook and click **Data > Refresh All**.
3. The Summary sheet, pivots, charts, and reconciliation check will update automatically — no manual steps required.
