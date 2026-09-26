# Financial Goals performance check

Date: 2026-09-26. Branch: `015-financial-goals`.

The backend scale regression seeds 100 owned active goals, five linked accounts, and 1,000 mixed allocation/withdrawal events. It asserts accurate projections and bounded database reads: summary at most six queries, 50-goal list plus projection at most six, and detail at most four. The test uses the SQLite test connection. It passes as `GoalReadScaleTest`.

The browser check uses a fixture of 100 goals and 1,000 activity records with paged API responses, then measures navigation until the overview first goal or detail progress is visible. It takes one warmup and 20 measured navigations per route at each viewport. Chrome preview results:

| CSS viewport | Overview p95 | Detail p95 | Samples per route |
| --- | ---: | ---: | ---: |
| 1280 px | 255 ms | 234 ms | 20 |
| 320 px | 214 ms | 229 ms | 20 |

Both browser fixture timings are below 2,000 ms. They measure the UI with mocked API latency; they do not measure a deployed MySQL service or network latency. The backend test separately bounds SQL query growth. **SC-005 and T071 remain pending** until overview and detail p95 are measured through a representative MySQL-backed API and network path with 100 owned goals and 1,000 activities.
