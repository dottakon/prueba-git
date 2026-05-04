# OpenCode Installation & Verification Guide (Windows + PowerShell)

## Overview
This guide installs OpenCode on Windows using PowerShell, configures PATH correctly, verifies functionality, and includes troubleshooting and cleanup steps.

## 1) Prerequisites
- PowerShell
- `winget` available
- Internet access

Install (or update) Git for Windows:
```powershell
winget install --id Git.Git -e --source winget --accept-package-agreements --accept-source-agreements
```

## 2) Configure PATH for Git Bash and OpenCode
Add both required locations to **User PATH** and current session PATH:
- `C:\Program Files\Git\bin`
- `$HOME\.opencode\bin`

```powershell
$pathsToAdd = @(
  'C:\Program Files\Git\bin',
  "$HOME\.opencode\bin"
)

$userPath = [Environment]::GetEnvironmentVariable('Path','User')
if (-not $userPath) { $userPath = '' }

$currentUserParts = $userPath -split ';' | Where-Object { $_ -ne '' }
foreach ($p in $pathsToAdd) {
  if ($currentUserParts -notcontains $p) {
    $userPath = ($userPath.TrimEnd(';') + ';' + $p).Trim(';')
    $currentUserParts += $p
  }
}
[Environment]::SetEnvironmentVariable('Path', $userPath, 'User')

$sessionParts = $env:Path -split ';'
foreach ($p in $pathsToAdd) {
  if ($sessionParts -notcontains $p) {
    $env:Path += ";$p"
  }
}
```

## 3) Install OpenCode
In PowerShell, avoid `curl -fsSL ... | bash` because `curl` is an alias for `Invoke-WebRequest`.

Use this instead:
```powershell
$scriptPath = Join-Path $env:TEMP 'opencode-install.sh'
Invoke-WebRequest -Uri 'https://opencode.ai/install' -OutFile $scriptPath
bash $scriptPath
```

## 4) Basic verification
```powershell
bash --version
opencode --version
opencode --help
```

Expected:
- `bash` returns a version string
- `opencode --version` returns installed version (for example `1.14.33`)
- `opencode --help` shows command list

## 5) Functional verification in a real project
Run OpenCode against a project and ask a basic prompt:

```powershell
$project = 'C:\path\to\project'
Push-Location $project
opencode run "Explain what app.py does"
Pop-Location
```

Expected:
- Exit code `0`
- A valid explanation response (not command-not-found errors)

## 6) Optional end-to-end test project
Create and test a minimal project:

```powershell
$proj = Join-Path $HOME 'opencode-test'
New-Item -ItemType Directory -Path $proj -Force | Out-Null

@'
def greet(name: str) -> str:
    return f"Hello, {name}!"

if __name__ == "__main__":
    print(greet("OpenCode Test"))
'@ | Set-Content -Path (Join-Path $proj 'app.py')
Push-Location $proj
opencode run "Explain what app.py does"
Pop-Location
```

## 7) Optional interactive test
```powershell
opencode C:\path\to\project
```

## 8) Maintenance commands
```powershell
opencode upgrade
opencode uninstall
```

## 9) Cleanup (if you created test project)
```powershell
$proj = Join-Path $HOME 'opencode-test'
if (Test-Path $proj) { Remove-Item -Path $proj -Recurse -Force }
```

## 10) Troubleshooting
- **`bash` not recognized**  
  Ensure `C:\Program Files\Git\bin` is in PATH, then open a new PowerShell window.
- **`curl -fsSL` fails in PowerShell**  
  Use `Invoke-WebRequest -OutFile` + `bash <script>` (Step 3).
- **`opencode` not recognized**  
  Ensure `$HOME\.opencode\bin` is in PATH, then restart terminal.
- **First run is slow**  
  One-time database migration can occur on first OpenCode invocation.

## Final checklist
- [ ] `bash --version` works
- [ ] `opencode --version` works
- [ ] `opencode --help` works
- [ ] `opencode run "<prompt>"` returns expected output when run inside a project directory
- [ ] Commands still work in a newly opened PowerShell session
