# OWNER CONTROL CONTRACT

> **Foundation wiring note:** This contract is now a default `GeneralApp` rule. Any retained `BUILD-PLAN` file that defines configurable behavior must satisfy this owner-control traceability model.

This document defines the non-negotiable default for comprehensive builds.

## Core rule

If an element exists in the product and could reasonably need platform-owner control, it must trace back to an owner configuration surface.

That means the implementation is not complete if the element is:

- hardcoded
- renameable in settings but not actually driven by settings
- visually configurable in one place but behaviorally fixed elsewhere
- governed only in code with no owner/operator control path
- controlled inconsistently across pages, components, APIs, and enforcement layers

## What must be owner-configurable by default

Unless explicitly documented as intentionally fixed, the following must be traceable to owner control:

- brand identity
- platform name
- logos and marks
- sidebar/header/footer labels
- navigation structure
- menu items
- buttons
- tabs
- cards
- dialogs
- forms
- form fields
- page sections
- empty states
- help text
- visibility rules
- enabled/disabled states
- role access
- capability access
- feature flags
- notification behavior
- integration behavior
- AI/provider behavior
- billing and plan behavior
- system defaults
- organization overrides

## Traceability requirement

Each meaningful element must have a clear trace like:

`UI element -> config key -> owner UI -> persistence layer -> runtime enforcement/rendering`

If that chain is broken, the element is not comprehensively implemented.

Examples of failure:

- sidebar brand text is hardcoded
- platform name setting exists but changing it does not update runtime UI
- feature flag exists in admin but component behavior ignores it
- permission model exists in data but owner cannot inspect or manage it
- library supports richer control but implementation exposes only a shallow subset

## Permit.io-level expectation

The target owner experience is closer to Permit.io and advanced admin systems than to a basic settings page.

That means:

- real management UI
- real inspection/debug UI
- real audit/history
- real override paths
- real scope hierarchy
- real policy and capability visibility
- real bulk actions where needed
- real explanation of why something is enabled, disabled, visible, hidden, allowed, or denied

## Library rule

Because the system uses proven libraries/frameworks, advanced owner control should be easier, not weaker.

Do not:

- install strong libraries and expose only their simplest features
- hardcode values that should be configuration-backed
- create settings that are only decorative

Do:

- use the full practical control surface of the chosen library/framework
- expose those controls through owner/operator UIs where platform control is required
- keep platform-level controls behind foundation capabilities and adapters

## Build acceptance rule

A build is not comprehensive if:

- owner-facing settings do not actually drive runtime behavior
- important elements appear without an owner control path
- admin pages exist only as display shells
- configuration is cosmetic instead of authoritative

If full owner control is not being built for a surface, that downgrade must be discussed explicitly before implementation proceeds.
