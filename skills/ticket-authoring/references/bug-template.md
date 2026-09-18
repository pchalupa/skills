# <Area> — <what breaks, in one line>

> Title format: `<Area> — <what breaks>`. Name the screen or service and the broken behaviour, so the title alone is enough to triage. Avoid "doesn't work".

## Summary

> One to three sentences: what the user does, what goes wrong, and who it hits. A reader who stops here still knows whether to pick the ticket up.

## Steps to reproduce

> Numbered, three to eight steps, starting from a known state (fresh install, logged out, seeded account). Every step is one action with literal values — the button label in quotes, the exact input, the exact account. A step someone can perform two different ways is two tickets' worth of confusion.

1. <action>
2. <action>
3. <action>

## Expected result

> The single behaviour that should happen at the last step, stated as an observable fact, plus where that expectation comes from (design, spec, previous release) when it isn't obvious.

## Actual result

> What happens instead, observable and specific: the error text verbatim, the wrong value, the crash, the frozen screen. Facts only; a theory about the cause belongs in the summary.

## Test environment

| Field           | Value                                         |
| --------------- | --------------------------------------------- |
| Platform        | <iOS 26.1 / Android 16 / Chrome 142 on macOS> |
| Device          | <iPhone 17 Pro, physical / Simulator>         |
| App version     | <4.12.0 (1180)>                               |
| Environment     | <development / staging / production>          |
| Account         | <qa+checkout@example.com, role: admin>        |
| Network         | <Wi-Fi / 4G / offline>                        |
| Reproducibility | <5 of 5 attempts / intermittent, 2 of 10>     |

> Drop rows that don't apply; keep platform, app version, environment and reproducibility always. A version range beats a single version when you know the regression window.
