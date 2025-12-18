# Clean up GitHub Actions workflow runs

## Objective
GitHub Actions workflow might be ran for troubleshooting and debugging the integration with CI/CD tools. It might contain sensitive information or unintended output. Cleaning up failed runs and old runs would reduce the chance of peeking eyes seeing unintended information.

Prioritise using GitHub MCP and GitHub CLI to work on the task.

### GitHub CLI
If using GitHub CLI, ensure that the user has authenticated.

## Sample Scripts

#### PowerShell Script to Delete Failed/Cancelled Runs

Run this script to list and delete failed or cancelled workflow runs:

```powershell
$repo = gh repo view --json nameWithOwner -q .nameWithOwner
if (-not $repo) { Write-Error "Can't determine repo. Run 'gh auth status' and retry."; exit 1 }

$raw = gh api "repos/$repo/actions/runs?per_page=100"
$runs = ($raw | ConvertFrom-Json).workflow_runs |
        Where-Object { $_.conclusion -in @('failure','cancelled') } |
        Select-Object id,name,head_branch,conclusion,html_url

if (-not $runs) { Write-Host "No failed/cancelled runs found."; exit 0 }

$runs | Format-Table id,name,head_branch,conclusion -AutoSize
$choice = Read-Host "Enter run IDs to delete (comma-separated), or type 'all' to delete all, or 'none' to abort"

if ($choice -eq 'none') { Write-Host "Aborted."; exit 0 }
if ($choice -eq 'all') { $ids = $runs.id } else { $ids = $choice -split '\s*,\s*' }

foreach ($id in $ids) {
  Write-Host "Deleting run $id..."
  gh api -X DELETE "repos/$repo/actions/runs/$id"
  if ($LASTEXITCODE -eq 0) { Write-Host "Deleted $id" } else { Write-Host "Failed to delete $id (exit $LASTEXITCODE)" }
}
```

**Usage:**
1. Authenticate: `gh auth login`
2. Run the script in your repository directory
3. Review the listed failed/cancelled runs
4. Enter `all` to delete all, specific IDs (comma-separated) to delete selected runs, or `none` to abort

#### PowerShell Script to Delete Runs Older Than 14 Days

Run this script to list and delete workflow runs older than 14 days:

```powershell
$repo = gh repo view --json nameWithOwner -q .nameWithOwner
if (-not $repo) { Write-Error "Can't determine repo. Run 'gh auth status' and retry."; exit 1 }

$cutoffDate = (Get-Date).AddDays(-14)
$raw = gh api "repos/$repo/actions/runs?per_page=100"
$runs = ($raw | ConvertFrom-Json).workflow_runs |
        Where-Object { [DateTime]$_.created_at -lt $cutoffDate } |
        Select-Object id,name,head_branch,created_at,status

if (-not $runs) { Write-Host "No runs older than 14 days found."; exit 0 }

$runs | Format-Table id,name,head_branch,created_at -AutoSize
Write-Host "`nFound $($runs.Count) run(s) older than 14 days."
$choice = Read-Host "Enter run IDs to delete (comma-separated), or type 'all' to delete all, or 'none' to abort"

if ($choice -eq 'none') { Write-Host "Aborted."; exit 0 }
if ($choice -eq 'all') { $ids = $runs.id } else { $ids = $choice -split '\s*,\s*' }

foreach ($id in $ids) {
  Write-Host "Deleting run $id..."
  gh api -X DELETE "repos/$repo/actions/runs/$id"
  if ($LASTEXITCODE -eq 0) { Write-Host "Deleted $id" } else { Write-Host "Failed to delete $id (exit $LASTEXITCODE)" }
}
```

**Usage:**
1. Authenticate: `gh auth login`
2. Run the script in your repository directory
3. Review the listed old runs (older than 14 days)
4. Enter `all` to delete all, specific IDs (comma-separated) to delete selected runs, or `none` to abort