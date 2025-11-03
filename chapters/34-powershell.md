# Chapter 34: PowerShell - Windows Finally Gets a Real Shell

## The Object-Oriented Shell Revolution

PowerShell was created by Jeffrey Snover at Microsoft, first released in 2006. For years, Windows administrators envied Unix's powerful command-line tools (Bash, AWK, sed, grep). Windows had CMD.EXE—a pathetic joke compared to Unix shells.

Snover's insight: Don't copy Unix. Text parsing is error-prone. Windows is built on .NET objects. Why not pipe *objects* instead of text?

PowerShell was revolutionary: a shell that passes structured data, not strings. Commands output .NET objects with properties and methods. You manipulate data structures, not text.

Initially Windows-only, PowerShell went open-source in 2016 and became cross-platform (PowerShell Core). By 2025, it runs on Linux, macOS, and Windows. It's the standard for Windows automation and increasingly used for cross-platform DevOps.

## The Philosophy

**Object-oriented shell**: Commands pass .NET objects with properties and methods. No parsing output; access data directly.

**Verb-Noun naming convention**: All cmdlets follow Verb-Noun pattern (Get-Process, Set-Location, Remove-Item). Consistent, predictable, discoverable.

**Designed for discovery**: Tab completion, Get-Help, Get-Command, and Get-Member help you explore without documentation.

**Integration with .NET**: Full access to .NET Framework. Call any .NET library from the command line.

**Consistent syntax**: All cmdlets use same parameter patterns. Learn one, learn them all.

**Backward compatibility**: PowerShell runs .NET code, CMD commands, and external executables. You can mix paradigms.

The result: A shell that's as powerful as Bash but designed for the modern, object-oriented world.

## What It Looks Like

### Basic Syntax

```powershell
# Simple command
Write-Host "Hello, World!"

# Variables start with $
$name = "Alice"
$age = 30
$price = 19.99

# String interpolation
Write-Host "Name: $name, Age: $age"
Write-Host "Name: ${name}, Age: ${age}"  # Explicit bracing

# Here-strings (multiline)
$text = @"
Multiple
lines
of text
"@

# Arrays
$numbers = 1, 2, 3, 4, 5
$colors = @("red", "green", "blue")

# Range operator
$range = 1..10

# Hash tables (dictionaries)
$person = @{
    Name = "Alice"
    Age = 30
    City = "NYC"
}

Write-Host $person.Name
Write-Host $person["Age"]
```

### Cmdlets (Commandlets)

```powershell
# Get-Command: Find commands
Get-Command Get-*
Get-Command *-Service

# Get-Help: Built-in documentation
Get-Help Get-Process
Get-Help Get-Process -Examples
Get-Help Get-Process -Detailed

# Get-Member: Explore object properties/methods
Get-Process | Get-Member
$person | Get-Member

# Common cmdlets
Get-Process                    # List processes
Get-Service                    # List services
Get-ChildItem                  # List files (like ls)
Set-Location C:\Users          # Change directory (like cd)
Copy-Item file.txt backup.txt  # Copy file
Remove-Item file.txt           # Delete file
```

### The Power of Object Pipelines

```powershell
# Unix way (text parsing):
# ps aux | grep chrome | awk '{print $2}' | xargs kill

# PowerShell way (objects):
Get-Process -Name chrome | Stop-Process

# Get top 5 CPU-consuming processes
Get-Process |
    Sort-Object CPU -Descending |
    Select-Object -First 5 Name, CPU

# Filter by property (no text parsing!)
Get-Process |
    Where-Object {$_.CPU -gt 100} |
    Select-Object Name, CPU, Memory

# Access properties directly
$procs = Get-Process
foreach ($proc in $procs) {
    if ($proc.WorkingSet -gt 100MB) {
        Write-Host "$($proc.Name): $($proc.WorkingSet / 1MB) MB"
    }
}

# Get services, filter, start them
Get-Service |
    Where-Object {$_.Status -eq "Stopped"} |
    Where-Object {$_.StartType -eq "Automatic"} |
    Start-Service -WhatIf  # -WhatIf shows what would happen
```

### Functions and Scripts

```powershell
# Simple function
function Greet {
    param($Name)
    Write-Host "Hello, $Name!"
}

Greet "Alice"

# Function with types and defaults
function Calculate {
    param(
        [int]$X,
        [int]$Y,
        [string]$Operation = "Add"
    )

    switch ($Operation) {
        "Add" { $X + $Y }
        "Subtract" { $X - $Y }
        "Multiply" { $X * $Y }
        "Divide" { $X / $Y }
        default { Write-Error "Unknown operation" }
    }
}

Calculate -X 10 -Y 5 -Operation "Multiply"  # 50

# Advanced function with proper cmdlet features
function Get-LargeFiles {
    [CmdletBinding()]
    param(
        [Parameter(Mandatory=$true)]
        [string]$Path,

        [Parameter()]
        [int]$MinimumSize = 10MB
    )

    Get-ChildItem -Path $Path -Recurse -File |
        Where-Object {$_.Length -gt $MinimumSize} |
        Select-Object FullName, @{Name="SizeMB";Expression={$_.Length / 1MB}}
}

Get-LargeFiles -Path C:\Users -MinimumSize 100MB
```

### Control Flow

```powershell
# If-ElseIf-Else
if ($x -gt 0) {
    Write-Host "Positive"
} elseif ($x -lt 0) {
    Write-Host "Negative"
} else {
    Write-Host "Zero"
}

# Comparison operators (note the -)
# -eq, -ne, -gt, -lt, -ge, -le
# -like, -notlike, -match, -notmatch
if ($name -eq "Alice") { }
if ($name -like "Ali*") { }
if ($email -match ".*@example\.com") { }

# Switch statement
switch ($status) {
    "Running" { Write-Host "Active" }
    "Stopped" { Write-Host "Inactive" }
    default { Write-Host "Unknown" }
}

# ForEach loop
foreach ($item in $collection) {
    Write-Host $item
}

# ForEach-Object (in pipeline)
1..10 | ForEach-Object { $_ * 2 }

# While loop
while ($count -lt 10) {
    $count++
}

# For loop
for ($i = 0; $i -lt 10; $i++) {
    Write-Host $i
}
```

### Error Handling

```powershell
# Try-Catch-Finally
try {
    Get-Content "nonexistent.txt" -ErrorAction Stop
    # Do something
} catch {
    Write-Host "Error occurred: $_"
    Write-Host "Exception: $($_.Exception.Message)"
} finally {
    Write-Host "Cleanup code here"
}

# Error actions
Get-Process -Name "doesnotexist" -ErrorAction SilentlyContinue
Get-Process -Name "doesnotexist" -ErrorAction Stop
Get-Process -Name "doesnotexist" -ErrorAction Ignore
```

### Working with .NET

```powershell
# Access .NET classes
[System.DateTime]::Now
[System.Environment]::MachineName
[System.Math]::PI
[System.Math]::Sqrt(16)

# Create .NET objects
$datetime = New-Object System.DateTime 2025, 1, 1
$random = New-Object System.Random
$random.Next(1, 100)

# Static methods
[System.IO.File]::Exists("C:\test.txt")
[System.IO.Path]::GetFileName("C:\Users\Alice\file.txt")

# Regular expressions
$text = "Email: alice@example.com"
if ($text -match "([a-zA-Z0-9._%+-]+@[a-zA-Z0-9.-]+\.[a-zA-Z]{2,})") {
    Write-Host "Email found: $($Matches[1])"
}
```

### Real-World Examples

```powershell
# Backup script
$source = "C:\Important"
$destination = "D:\Backups\$(Get-Date -Format 'yyyy-MM-dd')"

if (!(Test-Path $destination)) {
    New-Item -ItemType Directory -Path $destination
}

Copy-Item -Path $source\* -Destination $destination -Recurse
Write-Host "Backup completed to $destination"

# System monitoring
function Get-SystemInfo {
    [PSCustomObject]@{
        ComputerName = $env:COMPUTERNAME
        OS = (Get-WmiObject Win32_OperatingSystem).Caption
        MemoryGB = [math]::Round((Get-WmiObject Win32_ComputerSystem).TotalPhysicalMemory / 1GB, 2)
        CPUUsage = (Get-WmiObject Win32_Processor).LoadPercentage
        DiskFree = (Get-PSDrive C | Select-Object -ExpandProperty Free) / 1GB
    }
}

Get-SystemInfo

# Web scraping
$response = Invoke-WebRequest -Uri "https://example.com"
$response.StatusCode
$response.Content
$response.Links | Select-Object href

# REST API calls
$weather = Invoke-RestMethod -Uri "https://api.weather.com/data"
$weather.temperature

# Send email (if configured)
Send-MailMessage -To "admin@example.com" `
    -From "alert@example.com" `
    -Subject "Server Alert" `
    -Body "CPU usage is high" `
    -SmtpServer "smtp.example.com"
```

### Modules

```powershell
# Import module
Import-Module ActiveDirectory
Import-Module Pester  # Testing framework

# List available modules
Get-Module -ListAvailable

# Find modules online
Find-Module -Name "*Azure*"

# Install module from PowerShell Gallery
Install-Module -Name PSScriptAnalyzer
Install-Module -Name Pester

# Create your own module
# MyModule.psm1
function Get-Greeting {
    param($Name)
    "Hello, $Name from MyModule!"
}

Export-ModuleMember -Function Get-Greeting

# Use module
Import-Module .\MyModule.psm1
Get-Greeting "Alice"
```

### Remoting (Run Commands on Remote Machines)

```powershell
# Enable remoting (one-time setup)
Enable-PSRemoting -Force

# Run command on remote computer
Invoke-Command -ComputerName Server01 -ScriptBlock {
    Get-Process
}

# Interactive remote session
Enter-PSSession -ComputerName Server01
# You're now in the remote shell
Get-Service
Exit-PSSession

# Run command on multiple computers
Invoke-Command -ComputerName Server01, Server02, Server03 -ScriptBlock {
    Restart-Service -Name Spooler
}

# Copy files to remote machines
Copy-Item -Path C:\local\file.txt -Destination C:\remote\ -ToSession $session
```

## The Good Parts

**Object pipelines**: No text parsing. Access properties directly. Pipe complex data structures between commands.

**Discoverability**: Tab completion, Get-Help, Get-Command, Get-Member. The shell teaches itself.

**Consistent naming**: Verb-Noun pattern. Once you learn the pattern, you can guess command names.

**Full .NET access**: Call any .NET library. Massive ecosystem at your fingertips.

**Cross-platform**: Runs on Windows, Linux, macOS. Write once, run anywhere (mostly).

**Powerful for automation**: System administration, DevOps, Active Directory management. PowerShell excels.

**Remoting built-in**: Run commands on remote machines easily. No SSH needed (though PowerShell over SSH also works).

**Rich type system**: Strong typing when needed. Type enforcement in parameters.

## The Pain Points

**Verbose syntax**: Verb-Noun naming means long command names. `Remove-Item` vs. `rm`.

**Aliases help but can confuse**: `ls` is actually `Get-ChildItem`. Works differently from Unix `ls`.

**Learning curve for Unix users**: If you know Bash, PowerShell feels alien. Different philosophy.

**Windows-centric history**: Still deeply integrated with Windows. Some cmdlets don't work cross-platform.

**Slower than native tools**: Object overhead. PowerShell is slower than Bash for simple text processing.

**Error messages can be cryptic**: .NET exceptions bubble up. Stack traces are verbose.

**Not ideal for quick scripting**: For simple one-liners, Bash is faster to type. PowerShell is verbose.

**Different comparison operators**: `-eq` instead of `==`, `-and` instead of `&&`. Syntactic friction.

## Use Cases

**Windows system administration**: Managing services, processes, registry, Active Directory. PowerShell is essential.

**DevOps and automation**: CI/CD pipelines, deployment scripts, infrastructure as code. Works well with Azure, AWS.

**Configuration management**: Desired State Configuration (DSC) for server configuration.

**Active Directory management**: User management, group policy, queries. PowerShell is the standard tool.

**Exchange and Office 365**: Email administration, mailbox management, automation.

**Azure cloud management**: PowerShell is first-class citizen for Azure. Manage resources, automation.

**Cross-platform scripting**: With PowerShell Core, write scripts that run on Windows, Linux, and macOS.

**Testing**: Pester framework for testing PowerShell code and infrastructure.

## The Ecosystem

**PowerShell Gallery**: Central repository for modules. Like npm for PowerShell.

**Modules**: Thousands of modules for various tasks. Azure, AWS, VMware, Active Directory, etc.

**ISE (Integrated Scripting Environment)**: GUI for writing PowerShell scripts (Windows only, deprecated in favor of VS Code).

**VS Code**: Best editor for PowerShell. PowerShell extension provides IntelliSense, debugging.

**DSC (Desired State Configuration)**: Configuration management tool built on PowerShell.

**Pester**: Testing framework. Write tests for scripts and infrastructure.

**Community**: Large, active community. Microsoft supports it. Good documentation.

## Common Phrases You'll Hear

**"Objects, not text"**: The core philosophy. PowerShell pipes structured data.

**"Verb-Noun"**: Naming convention. Get-Process, Set-Location, Remove-Item.

**"Get-Help is your friend"**: Built-in documentation. Use it liberally.

**"Pipeline everything"**: Chain cmdlets together. Each outputs objects for the next.

**"Aliases are shortcuts"**: `ls`, `cd`, `rm` work, but they're aliases for PowerShell cmdlets.

**"The PowerShell way"**: Do it with objects, not text processing.

## The Verdict

PowerShell is the definitive shell for Windows administrators and increasingly relevant for cross-platform DevOps. Its object-oriented approach is genuinely better than text parsing for structured data.

**Choose PowerShell if**:
- You're a Windows administrator (mandatory)
- You manage Active Directory, Exchange, or Office 365
- You work with Azure or hybrid cloud
- You need cross-platform scripting for DevOps
- You want type-safe, structured automation
- You prefer discoverable, consistent syntax

**Avoid PowerShell if**:
- You're doing simple text processing (Bash is faster)
- You're on Unix and don't need cross-platform scripts
- You value terse syntax over explicitness
- You don't work with Windows systems
- You're writing quick one-liners (Bash is more concise)

PowerShell won't replace Bash for Unix administrators. But for Windows automation, it's essential. For DevOps working across platforms, it's increasingly valuable.

It proved that shells don't have to pipe text. Objects are better for structured data. Other shells are starting to adopt this lesson (Nushell, for example).

PowerShell is verbose, sometimes clunky, but undeniably powerful. For Windows administrators, it transformed the job from clicking through GUIs to writing repeatable, testable automation.

If you administer Windows systems and don't know PowerShell, you're doing it the hard way.

---

**Next**: [Chapter 35: Assembly - As Close to Metal as It Gets](35-assembly.md)
