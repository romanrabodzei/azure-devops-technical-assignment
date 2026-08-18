# Security Considerations

> Candidate deliverable — required by README §1.3 ("Document any
> assumptions and security considerations"). Replace every prompt below.

## Secret handling

- Where does the `external-api-key` value originate for each environment,
  and how does it reach the Key Vault without ever being committed to Git
  or written to a plain-text config file?
- How is the pipeline itself prevented from printing the secret value to
  logs (§2.5 / §2.7)? Name the specific Azure DevOps mechanism(s) used.

## Identity & access

- What managed identity is used by the App Service, and what is it
  granted access to?
- What is the *minimum* role/permission granted for:
  - reading the Key Vault secret;
  - accessing the designated Storage Account/container?
- Are Storage Account keys or connection strings used anywhere? If not,
  what replaces them?

## Key Vault

- RBAC or access policies, and why?
- Is soft-delete / purge protection considered? Any other Key Vault
  hardening you applied or intentionally skipped?

## Network exposure

- Are any resources publicly reachable? If so, is that an accepted
  assumption for this assignment, or would you change it for a real
  production deployment?

## Pipeline security

- How are Azure DevOps service connections scoped (subscription-wide vs.
  resource-group-scoped, least privilege)?
- Any secrets stored in variable groups / Key Vault-linked variable
  groups vs. plain pipeline variables — and why?

## Things you'd do differently for production

Be honest about tradeoffs made for the sake of the assignment's scope
(e.g. simplified network rules, no private endpoints, single subscription)
that you would not accept in a real production environment.
