# Chapter 34: PowerShell - Windows Finally Gets a Real Shell

## The Object-Oriented Shell

PowerShell was created by Microsoft in 2006 to give Windows administrators a powerful command-line interface. Unlike Bash (which pipes text), PowerShell pipes objects—structured data between commands.

By 2025, PowerShell is cross-platform (runs on Linux and macOS) and the standard for Windows automation and system administration.

## The Philosophy

**Object-oriented**: Commands pass .NET objects, not text.

**Consistent naming**: Verb-Noun pattern (Get-Process, Set-Location).

**Discovery**: Tab completion and Get-Help make it learnable.

**Integration**: Deep Windows integration, but now cross-platform.

## What It Looks Like

```powershell
# Simple commands
Write-Host "Hello, World!"

# Variables
$name = "Alice"
$numbers = 1..10

# Pipelines (passing objects!)
Get-Process | Where-Object {$_.CPU -gt 100} | Sort-Object -Property CPU -Descending

# Functions
function Greet {
    param($Name)
    Write-Host "Hello, $Name!"
}

# Objects
$person = @{
    Name = "Alice"
    Age = 30
}

# Loops
foreach ($num in $numbers) {
    Write-Host $num
}

# Conditionals
if ($x -gt 0) {
    Write-Host "Positive"
} elseif ($x -lt 0) {
    Write-Host "Negative"
} else {
    Write-Host "Zero"
}
```

## The Verdict

PowerShell is the Windows administrator's language. Object-oriented pipelines are powerful once you learn them. The verb-noun naming convention is consistent and discoverable.

Essential for Windows automation. Cross-platform support makes it viable on Linux/macOS. Not for general programming, but excellent for system administration.

---

**Next**: [Chapter 35: Assembly - As Close to Metal as It Gets](35-assembly.md)
