# Additional Automation

> Candidate deliverable — required by README §3.8. Describe the extra
> PowerShell automation you added beyond `Test-AzureDeployment.ps1`.

## What it does

One or two sentences describing the script and where it lives (e.g.
`scripts/<YourScript>.ps1`).

## Why it's useful

What operational problem does it solve for this infrastructure? (e.g.
drift detection, tag compliance reporting, pre-deployment sanity checks,
deployment summary generation.)

## How to run it

```powershell
./scripts/<YourScript>.ps1 `
    -SubscriptionId "..." `
    -ResourceGroupName "..." `
    -Environment "dev"
```

## Sample output

Paste a short example of what running it looks like.
