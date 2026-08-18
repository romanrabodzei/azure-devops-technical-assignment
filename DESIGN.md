# Design Notes

> Candidate deliverable. Replace every prompt below with your own content.
> Keep it concise — bullet points are fine. This file exists so a reviewer
> can understand *why* you built it this way without re-reading every file.

## Naming convention

How are resource names constructed (Resource Group, Storage Accounts, Key
Vault, App Service)? Storage Account names must be globally unique,
lowercase alphanumeric, 3–24 characters — explain how you handled that
given the required logical names (`cold`, `standard`, `hot`, `datalake`).

## Resource group strategy

One Resource Group per environment, or shared? Why?

## Configuration-driven resource loops

Where did you use a loop over an object/array to avoid duplicated resource
blocks (Storage Accounts, containers)? Briefly describe the shape of the
configuration structure you designed.

## Key Vault access model

RBAC or access policies? Why? What role(s) is the App Service's managed
identity assigned, and at what scope?

## Environment differences

What changes between Dev / Test / Prod (region, SKU, naming, approval
gates, anything else), and how is that expressed in your parameter files?

## Assumptions

List anything the brief left unspecified that you had to decide on your
own (e.g. network exposure/public access, region defaults, subscription
scope, retention/lifecycle policies). One line each is fine — the goal is
to make implicit decisions explicit.

## Known limitations / what you'd do with more time

Be honest here. Cut corners, things you'd harden for a real production
rollout, anything you didn't get to.
