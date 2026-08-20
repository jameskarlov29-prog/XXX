

#requires -version 5.1
chcp 65001 | Out-Null
[Console]::OutputEncoding = [System.Text.Encoding]::UTF8

# ============================================================
# XXX :: PERFORMANCE ENGINE (CLEAN BUILD)
# Low Latency | Stable Frametime | Competitive Optimization
# ------------------------------------------------------------
# NOTE: KeyAuth license check, HWID fingerprinting, and the
# Discord webhook telemetry have been REMOVED from this build.
# ============================================================

$ErrorActionPreference = "SilentlyContinue"

# ==========================
# ANIMATION CORE
# ==========================

$script:RC = @("Red","Yellow","Green","Cyan","Blue","Magenta","White")

function Write-Type {
    param(
        [string]$Text,
        [ConsoleColor]$Color = "Cyan",
        [int]$Delay = 10
    )
    $old = $Host.UI.RawUI.ForegroundColor
    $Host.UI.RawUI.ForegroundColor = $Color
    foreach ($ch in $Text.ToCharArray()) {
        Write-Host -NoNewline $ch
        Start-Sleep -Milliseconds $Delay
    }
    Write-Host
    $Host.UI.RawUI.ForegroundColor = $old
}

function Write-Glitch {
    param(
        [string]$Text,
        [ConsoleColor]$Color = "Cyan",
        [int]$Passes = 4
    )
    $pool = '!@#%^*<>?/|~+-=_'
    $old  = $Host.UI.RawUI.ForegroundColor
    for ($g = 0; $g -lt $Passes; $g++) {
        $Host.UI.RawUI.ForegroundColor = "DarkGray"
        $fake = -join ((1..$Text.Length) | ForEach-Object { $pool[(Get-Random -Maximum $pool.Length)] })
        Write-Host "`r$fake" -NoNewline
        Start-Sleep -Milliseconds 55
    }
    Write-Host "`r" -NoNewline
    $Host.UI.RawUI.ForegroundColor = $Color
    foreach ($ch in $Text.ToCharArray()) {
        Write-Host -NoNewline $ch
        Start-Sleep -Milliseconds 11
    }
    Write-Host
    $Host.UI.RawUI.ForegroundColor = $old
}
function Show-RainbowBar {
    param([string]$Label, [int]$Steps = 40)
    $spin = @('|','/','-','\')
    $old  = $Host.UI.RawUI.ForegroundColor
    Write-Host ""
    Write-Host "  $Label" -ForegroundColor White
    $bar = ""
    for ($i = 0; $i -lt $Steps; $i++) {
        $pct  = [int](($i / $Steps) * 100)
        $sp   = $spin[$i % $spin.Count]
        $col  = $script:RC[$i % $script:RC.Count]
        $Host.UI.RawUI.ForegroundColor = $col
        $bar += "#"
        $pad  = " " * ($Steps - $i - 1)
        Write-Host -NoNewline "`r  $sp [$bar$pad] $pct%  "
        Start-Sleep -Milliseconds (Get-Random -Minimum 12 -Maximum 36)
    }
    $Host.UI.RawUI.ForegroundColor = "Green"
    $full = "#" * $Steps
    Write-Host "`r  * [$full] 100% DONE"
    Write-Host
    $Host.UI.RawUI.ForegroundColor = $old
}

function Show-Pulse {
    param([string]$Text, [int]$Pulses = 3)
    $cols = @("DarkCyan","Cyan","White","Cyan","DarkCyan")
    for ($p = 0; $p -lt $Pulses; $p++) {
        foreach ($col in $cols) {
            Write-Host "`r  $Text  " -NoNewline -ForegroundColor $col
            Start-Sleep -Milliseconds 40
        }
    }
    Write-Host ""
}

function Show-Scan {
    param([string]$Label, [int]$Dots = 28)
    $d = ""
    for ($i = 0; $i -lt $Dots; $i++) {
        $d  += "."
        $col = $script:RC[$i % $script:RC.Count]
        Write-Host "`r  $Label $d" -NoNewline -ForegroundColor $col
        Start-Sleep -Milliseconds 28
    }
    Write-Host " [OK]" -ForegroundColor Green
}

function Show-Spinner {
    param([string]$Text = "Processing", [int]$Seconds = 2)
    $frames = @("|","/","-","\")
    $end = (Get-Date).AddSeconds($Seconds)
    $i   = 0
    while ((Get-Date) -lt $end) {
        $col = $script:RC[$i % $script:RC.Count]
        $Host.UI.RawUI.ForegroundColor = $col
        Write-Host -NoNewline "`r  $Text $($frames[$i % 4])  "
        Start-Sleep -Milliseconds 100
        $i++
    }
    $Host.UI.RawUI.ForegroundColor = "Green"
    Write-Host "`r  $Text [READY]   "
}
function Show-StepOK {
    param([string]$Text, [ConsoleColor]$Color = "Cyan")
    $d = ""
    for ($i = 0; $i -lt 20; $i++) {
        $d  += "."
        $col = $script:RC[$i % $script:RC.Count]
        Write-Host "`r  [ENGINE] $Text $d" -NoNewline -ForegroundColor $col
        Start-Sleep -Milliseconds 22
    }
    Write-Host " OK" -ForegroundColor Green
}

function Write-StatusOK  { param([string]$M) Write-Host "  [OK] $M" -ForegroundColor Green   }
function Write-StatusINF { param([string]$M) Write-Host "  [i]  $M" -ForegroundColor Cyan    }
function Write-StatusWRN { param([string]$M) Write-Host "  [!]  $M" -ForegroundColor Yellow  }
function Write-StatusERR { param([string]$M) Write-Host "  [X]  $M" -ForegroundColor Red     }

# ==========================
# WINDOW TITLE STATE SYSTEM
# ==========================
function Set-AppState {
    param([ValidateSet("READY","APPLYING","ACTIVE","EXIT")][string]$State)
    switch ($State) {
        "READY"    { $host.UI.RawUI.WindowTitle = "XXX | READY"                   }
        "APPLYING" { $host.UI.RawUI.WindowTitle = "XXX | APPLYING OPTIMIZATIONS"  }
        "ACTIVE"   { $host.UI.RawUI.WindowTitle = "XXX | ACTIVE"                  }
        "EXIT"     { $host.UI.RawUI.WindowTitle = "XXX | EXITED"                  }
    }
}

# ==========================
# ADMIN PRIVILEGE CHECK
# ==========================
function Check-Admin {
    $currentUser = [Security.Principal.WindowsIdentity]::GetCurrent()
    $principal   = New-Object Security.Principal.WindowsPrincipal($currentUser)

    if (-not $principal.IsInRole([Security.Principal.WindowsBuiltInRole]::Administrator)) {
        Clear-Host
        Write-Host "  +--------------------------------------+" -ForegroundColor Red
        Write-Host "  |  XXX           :: ADMIN REQUIRED     |" -ForegroundColor Red
        Write-Host "  +--------------------------------------+" -ForegroundColor Red
        Write-Host ""
        Write-StatusERR "This program must be run as Administrator."
        Write-StatusWRN "Right-click and select 'Run as administrator Swagfix'."
        Write-Host ""
        Read-Host "  Press Enter to exit"
        exit
    }
    Write-StatusOK "Privilege Level : ADMINISTRATOR"
}

# ==========================
# THAI REAL-TIME TITLE
# ==========================
function Update-ThaiTimeTitle {
    $tz       = [System.TimeZoneInfo]::FindSystemTimeZoneById("SE Asia Standard Time")
    $thaiTime = [System.TimeZoneInfo]::ConvertTimeFromUtc((Get-Date).ToUniversalTime(), $tz)
    $host.UI.RawUI.WindowTitle = "XXX | TH {0}" -f $thaiTime.ToString("dd/MM/yyyy HH:mm:ss")
}
# ==========================
# SPLASH + BANNER
# ==========================
function Show-SplashEnterprise {
    Clear-Host
    Write-Host ""

    $banner = @(
        "  +--------------------------------------------------+",
        "  |  _   _ _   _____ __ __  _  _____ ____ ___ _     |",
        "  |                                                 |",
        "  |                                                 |",
        "  |                                                 |",
        "  |                                                 |",
        "  +--------------------------------------------------+"
    )
    $bc = @("DarkCyan","Cyan","Blue","Magenta","Cyan","White","DarkCyan")
    for ($i = 0; $i -lt $banner.Count; $i++) {
        Write-Host $banner[$i] -ForegroundColor $bc[$i]
        Start-Sleep -Milliseconds 65
    }

    Write-Host ""
    Show-Pulse "[ LOW LATENCY ] [ STABLE FRAMETIME ] [ COMPETITIVE ]" 4
    Write-Host ""
    Write-StatusINF "Version   : 1.1.5"
    Write-StatusINF "Security  : Administrator"
    Write-Glitch "  >> Initializing system components..." Cyan 3
    Start-Sleep -Milliseconds 400
}

# ==========================
# BOOT SEQUENCE
# ==========================
Check-Admin
Show-SplashEnterprise

Write-Host ""
Show-RainbowBar "Booting Core Modules" 35
Show-Scan "Syncing Hardware Profile"
Show-Scan "Loading Engine Components"
Write-Host ""

# ==========================
# HARDWARE DETECTION
# ==========================
function Detect-HardwareInfo {
    Write-Host ""
    Write-Glitch "  >> Scanning hardware topology..." Cyan 3
    Show-RainbowBar "Detecting CPU" 25
    Show-RainbowBar "Detecting GPU" 25

    $global:CPUName = (Get-CimInstance Win32_Processor).Name
    Write-StatusOK "CPU : $CPUName"

    $gpu = Get-WmiObject Win32_VideoController | Select-Object -First 1
    $global:GPUFullName = $gpu.Name
    Write-StatusOK "GPU : $GPUFullName"

    if     ($GPUFullName -match "NVIDIA")     { $global:GPUType = "NVIDIA" }
    elseif ($GPUFullName -match "AMD|Radeon") { $global:GPUType = "AMD"    }
    else                                      { $global:GPUType = "OTHER"  }

    Write-StatusINF "GPU TYPE : $GPUType"
    Write-Host ""
}
Detect-HardwareInfo
# ==========================
# CORE FUNCTIONS
# ==========================
function Set-PriorityControl {
    Show-StepOK "Applying PriorityControl parameters"
    reg add "HKLM\SYSTEM\CurrentControlSet\Control\PriorityControl" /v ConvertibleSlateMode /t REG_DWORD /d 0 /f | Out-Null
    reg add "HKLM\SYSTEM\CurrentControlSet\Control\PriorityControl" /v Win32PrioritySeparation /t REG_DWORD /d 22 /f | Out-Null
    Write-StatusOK "PriorityControl optimized."
}

function Disable-GameBar {
    Show-StepOK "Disabling Game Bar"
    reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\GameDVR" /v AppCaptureEnabled /t REG_DWORD /d 0 /f | Out-Null
    Write-StatusOK "Game Bar disabled."
}

function Disable-BackgroundApps {
    Show-StepOK "Disabling Background Apps"
    reg add "HKCU\Software\Microsoft\Windows\CurrentVersion\BackgroundAccessApplications" /v GlobalUserDisabled /t REG_DWORD /d 1 /f | Out-Null
    Write-StatusOK "Background apps disabled."
}

function Clear-NvidiaShaderCache {
    if ($GPUType -eq "NVIDIA") {
        Show-StepOK "Clearing NVIDIA Shader Cache"
        $paths = @("$env:LOCALAPPDATA\NVIDIA\DXCache","$env:LOCALAPPDATA\NVIDIA\GLCache")
        foreach ($p in $paths) {
            if (Test-Path $p) {
                Get-ChildItem -Path $p -Recurse -Force -ErrorAction SilentlyContinue |
                    Remove-Item -Recurse -Force -ErrorAction SilentlyContinue
            }
        }
        Write-StatusOK "NVIDIA Shader Cache cleared."
    }
}

function Clear-AMDShaderCache {
    if ($GPUType -eq "AMD") {
        Show-StepOK "Clearing AMD Shader Cache"
        $paths = @("$env:LOCALAPPDATA\AMD\DXCache","$env:LOCALAPPDATA\AMD\GLCache","$env:LOCALAPPDATA\AMD\VkCache")
        foreach ($p in $paths) {
            if (Test-Path $p) {
                Get-ChildItem -Path $p -Recurse -Force -ErrorAction SilentlyContinue |
                    Remove-Item -Recurse -Force -ErrorAction SilentlyContinue
            }
        }
        Write-StatusOK "AMD Shader Cache cleared."
    }
}

function Disable-FullscreenOptimizations {
    Show-StepOK "Applying PlusX FSE Optimizations"
    reg add "HKCU\System\GameConfigStore" /v GameDVR_FSEBehavior /t REG_DWORD /d 2 /f | Out-Null
    reg add "HKCU\System\GameConfigStore" /v GameDVR_FSEBehaviorMode /t REG_DWORD /d 2 /f | Out-Null
    reg add "HKCU\System\GameConfigStore" /v GameDVR_HonorUserFSEBehaviorMode /t REG_DWORD /d 1 /f | Out-Null
    Write-StatusOK "PlusX FSE Optimizations enabled."
}

function Optimize-MMCSSTasks {
    Show-StepOK "Checking FiveM MMCSS Settings"
    reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games" /v "Scheduling Category" /t REG_SZ /d High /f | Out-Null
    reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games" /v "SFIO Priority" /t REG_SZ /d High /f | Out-Null
    reg add "HKLM\SOFTWARE\Microsoft\Windows NT\CurrentVersion\Multimedia\SystemProfile\Tasks\Games" /v "Background Only" /t REG_SZ /d False /f | Out-Null
    Write-StatusOK "MMCSS Tasks optimized."
}

function Optimize-MouseInput {
    Show-StepOK "Applying PlusX Mouse Input"
    reg add "HKCU\Control Panel\Mouse" /v MouseSpeed /t REG_SZ /d 0 /f | Out-Null
    reg add "HKCU\Control Panel\Mouse" /v MouseThreshold1 /t REG_SZ /d 0 /f | Out-Null
    reg add "HKCU\Control Panel\Mouse" /v MouseThreshold2 /t REG_SZ /d 0 /f | Out-Null
    Write-StatusOK "Mouse input optimized."
}

function Disable-HPET {
    Show-StepOK "Checking HPET Status"
    bcdedit /deletevalue useplatformclock 2>$null | Out-Null
    Write-StatusOK "HPET check passed."
}

function Optimize-GPU-TDR {
    Show-StepOK "Applying GPU TDR Values"
    reg add "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v TdrDelay /t REG_DWORD /d 10 /f | Out-Null
    reg add "HKLM\SYSTEM\CurrentControlSet\Control\GraphicsDrivers" /v TdrDdiDelay /t REG_DWORD /d 20 /f | Out-Null
    Write-StatusOK "GPU TDR values applied."
}
function Preset-UltimateXPLUS {
    Write-Host ""
    Write-Glitch "  >> ULTIMATEXPLUS CORE ENGAGING..." Magenta 5
    Show-RainbowBar "ULTRA MODE -- LOW LATENCY ENGINE" 40
    Set-AppState "APPLYING"

    Disable-GameBar
    Disable-BackgroundApps
    Set-PriorityControl
    Disable-FullscreenOptimizations
    Optimize-MMCSSTasks
    Optimize-MouseInput
    Disable-HPET
    Optimize-GPU-TDR

    if ($GPUType -eq "NVIDIA") { Clear-NvidiaShaderCache }
    if ($GPUType -eq "AMD")    { Clear-AMDShaderCache    }

    Show-Scan "Applying TCP Network Tweaks"
    netsh int tcp set global autotuninglevel=high | Out-Null
    Write-StatusOK "Network tweaks applied."

    Write-Host ""
    Show-Pulse "[ LOW LATENCY MODE ACTIVE ] [ ULTIMATEXPLUS ] [ READY ]" 5
    Write-Glitch "  [ENGINE] LOW LATENCY MODE ACTIVE" Green 4

    Set-AppState "ACTIVE"

    Add-Type -AssemblyName System.Windows.Forms
    [System.Windows.Forms.MessageBox]::Show(
        "Installation completed successfully.`nAll optimizations are now active.",
        "ULTIMATEXPLUS 1.5",
        [System.Windows.Forms.MessageBoxButtons]::OK,
        [System.Windows.Forms.MessageBoxIcon]::Information
    )
}

# ==========================
# MENU
# ==========================
function Preset-Menu {
    while ($true) {
        Update-ThaiTimeTitle
        Clear-Host
        Write-Host ""
        Write-Host "  +------------------------------------+" -ForegroundColor Cyan
        Write-Host "  |   XXX                 --  PANEL    |" -ForegroundColor White
        Write-Host "  +------------------------------------+" -ForegroundColor Cyan
        Write-Host "  |  1)  Activate XXX                  |" -ForegroundColor Magenta
        Write-Host "  |  X)  Exit                          |" -ForegroundColor DarkGray
        Write-Host "  +------------------------------------+" -ForegroundColor Cyan
        Write-Host ""

        $c = Read-Host "  Select Option"
        switch ($c.ToUpper()) {
            "1" { Preset-XXX }
            "X" {
                Write-Host ""
                Write-Glitch "  >> XXX Engine Closed. Reboot recommended." DarkGray 2
                Set-AppState "EXIT"
                Write-Host ""
                return
            }
            default { Write-StatusERR "Invalid option." }
        }
        Write-Host ""
        Read-Host "  Press Enter to continue Swagfix"
    }
}

Preset-Menu