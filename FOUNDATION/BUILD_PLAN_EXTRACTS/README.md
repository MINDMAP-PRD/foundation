# BUILD PLAN EXTRACTS

This folder captures the parts of `../BUILD-PLAN/` that belong in the shared foundations instead of remaining only as project-level reference material.

## Purpose

The `BUILD-PLAN/` folder contains a mix of:

- true shared platform patterns
- optional cross-product capabilities
- module-specific feature systems
- meta/planning/training material

This extract folder separates those concerns so the foundations can reuse the right parts without swallowing the entire build-plan corpus.

## Files in this folder

- `BUILD_PLAN_FOUNDATION_PROMOTION_MAP.md`
  - classifies every `BUILD-PLAN` file by foundation fit
  - names the target: `GeneralApp`, `FormBuilder`, optional module, or reference-only
- `FOUNDATION_COMPREHENSIVE_WIRING_PLAN.md`
  - explains what should be promoted into the foundations
  - explains how the promoted parts interconnect through registries, settings, admin surfaces, and adapters

## Rule

These extracts do not replace the original `BUILD-PLAN/` files.

They define:

- what becomes canonical foundation scope
- what remains optional
- what stays module-only
- what should only be treated as reference material
