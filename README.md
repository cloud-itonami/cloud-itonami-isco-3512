# cloud-itonami-isco-3512

Open Occupation Blueprint for **ISCO-08 3512**: Information and Communications Technology User Support Technicians.

This repository designs a forkable OSS business for an independent IT support technician: a support kiosk robot performs device diagnostics and ticket intake under a governor-gated actor, so the practice keeps its own ticket and resolution records instead of renting a closed helpdesk SaaS.

## Robotics premise

All cloud-itonami verticals are designed on the premise that a **robot performs
the physical domain work**. Here a support kiosk robot performs device diagnostics, cable/peripheral checks and ticket intake at client sites under an actor that proposes
actions and an independent **IT Support Governor** that gates them. The governor never
dispatches hardware itself; `:high`/`:safety-critical` actions (such as
credential resets, or access-permission changes) require human sign-off.

A live sample of the operator console (robotics safety console, shared template) is rendered in [docs/samples/operator-console.html](docs/samples/operator-console.html) — pure-data HTML output of `kotoba.robotics.ui`.

## Core Contract

```text
support ticket + device inventory + access policy
        |
        v
Support Advisor -> IT Support Governor -> resolve, or human sign-off
        |
        v
robot actions (gated) + operating records + audit ledger
```

No automated advice can dispatch a robot action the governor refuses, suppress
an operating record, or disclose sensitive data without governor approval and
audit evidence.

## Capability layer

Resolves via [`kotoba-lang/occupation`](https://github.com/kotoba-lang/occupation)
(ISCO-08 `3512`). Required capabilities:

- :robotics
- :identity
- :forms
- :dmn
- :bpmn
- :audit-ledger

See [`docs/business-model.md`](docs/business-model.md) and
[`docs/operator-guide.md`](docs/operator-guide.md).

## License

AGPL-3.0-or-later.
