
# PowerShell Script om een EXE-applicatie naar Intune te uploaden

Dit script helpt bij het converteren van een EXE-bestand naar het `.intunewin`-formaat en het uploaden naar Microsoft Intune, zoals beschreven in het voorbeeld op de site [cloudinfra.net](https://cloudinfra.net/how-to-deploy-exe-applications-using-intune/#1-test-silent-install-of-exe-app).

## Vereisten

- **IntuneWinAppUtil.exe** moet gedownload en beschikbaar zijn op het opgegeven pad.
- Microsoft Graph PowerShell-module moet geïnstalleerd zijn.
- Een geldige Azure AD-account met Intune-toegangsrechten.

## PowerShell Script

```powershell
# Configuratie: Vul de volgende variabelen in met jouw specifieke waarden
$AppName = "HandBrake"
$Publisher = "HandBrake"
$AppDescription = "HandBrake Video Converter"
$AppVersion = "1.4.2"
$InstallerFile = "HandBrake-Installer.exe"
$SourceFolder = "C:\Temp\Install\"
$OutputFolder = "C:\Temp\Install\Output\"
$IntuneWinAppUtilPath = "C:\Tools\IntuneWinAppUtil\IntuneWinAppUtil.exe"
$IntuneAppName = "HandBrake Installer"
$DisplayName = "HandBrake Video Converter"
$InformationUrl = "https://handbrake.fr/"
$Owner = "IT Department"
$Notes = "Deploy HandBrake video converter using Intune"
$LogoPath = "C:\Temp\Install\handbrake-logo.png"  # Optioneel: Voeg een logo toe voor de app

# Functie om EXE te converteren naar IntuneWin
function Convert-ToIntuneWin {
    param (
        [Parameter(Mandatory = $true)]
        [string]$SourceFolder,

        [Parameter(Mandatory = $true)]
        [string]$InstallerFile,

        [Parameter(Mandatory = $true)]
        [string]$OutputFolder,

        [Parameter(Mandatory = $true)]
        [string]$IntuneWinAppUtilPath
    )

    # Controleer of IntuneWinAppUtil.exe bestaat
    if (-not (Test-Path $IntuneWinAppUtilPath)) {
        Write-Error "IntuneWinAppUtil.exe niet gevonden op $IntuneWinAppUtilPath. Geef het juiste pad op."
        return
    }

    # Controleer of bronmap bestaat
    if (-not (Test-Path $SourceFolder)) {
        Write-Error "Bronmap niet gevonden op $SourceFolder. Geef een geldig bronmappad op."
        return
    }

    # Controleer of uitvoermap bestaat, zo niet, maak het aan
    if (-not (Test-Path $OutputFolder)) {
        try {
            New-Item -Path $OutputFolder -ItemType Directory -Force
            Write-Output "Uitvoermap aangemaakt op $OutputFolder."
        }
        catch {
            Write-Error "Kon uitvoermap niet maken op $OutputFolder: $_"
            return
        }
    }

    # Stel het commando samen om IntuneWinAppUtil.exe uit te voeren
    $command = "$IntuneWinAppUtilPath -c `"$SourceFolder`" -s `"$InstallerFile`" -o `"$OutputFolder`""

    # Voer het commando uit
    try {
        Write-Output "IntuneWinAppUtil.exe uitvoeren met het volgende commando:"
        Write-Output $command
        Invoke-Expression $command
        Write-Output "Conversie naar .intunewin succesvol voltooid."
    }
    catch {
        Write-Error "Er is een fout opgetreden bij het uitvoeren van IntuneWinAppUtil.exe: $_"
    }
}

# Converteer de EXE naar een .intunewin-bestand
Convert-ToIntuneWin -SourceFolder $SourceFolder -InstallerFile $InstallerFile -OutputFolder $OutputFolder -IntuneWinAppUtilPath $IntuneWinAppUtilPath

# Laad de MS Graph module om met Intune te werken
Install-Module -Name Microsoft.Graph -Force -Scope CurrentUser
Import-Module Microsoft.Graph

# Maak verbinding met Intune
Connect-MgGraph -Scopes "DeviceManagementApps.ReadWrite.All"

# Krijg toegangstoken voor Graph API
$token = (Get-MgContext).AccessToken

# Bestandsupload-instellingen
$win32App = @{
    displayName = $DisplayName
    description = $AppDescription
    publisher = $Publisher
    fileName = "$OutputFolder\HandBrake-Installer.intunewin"
    informationUrl = $InformationUrl
    owner = $Owner
    notes = $Notes
    logo = Get-Content -Path $LogoPath -Encoding Byte
}

# Upload de applicatie naar Intune
Invoke-RestMethod -Uri "https://graph.microsoft.com/v1.0/deviceAppManagement/mobileApps" -Headers @{Authorization = "Bearer $token"} -Method Post -Body ($win32App | ConvertTo-Json) -ContentType "application/json"

Write-Output "Applicatie succesvol geüpload naar Intune."
```

## Wat Dit Script Doet

1. **Configuratie**: Definieer variabelen met jouw specifieke waarden zoals appnaam, uitgever, beschrijving, etc.
2. **Conversie naar `.intunewin`**: Het script gebruikt `IntuneWinAppUtil.exe` om de EXE naar het `.intunewin`-formaat te converteren.
3. **Installatie van MS Graph Module**: Installeert de Microsoft Graph PowerShell-module die nodig is om verbinding te maken met Intune.
4. **Verbinding Maken met Intune**: Maakt verbinding met Intune met behulp van de Graph API.
5. **Uploaden van Applicatie**: Uploadt de geconverteerde applicatie naar Intune.

## Gebruik

- Sla dit script op als een `.ps1`-bestand, bijvoorbeeld `Deploy-AppToIntune.ps1`.
- Voer het script uit in PowerShell als beheerder.
- Zorg ervoor dat je alle vereisten hebt geïnstalleerd en de juiste machtigingen hebt.

Dit script biedt een volledige geautomatiseerde oplossing voor het converteren van een EXE naar het .intunewin-formaat en het uploaden naar Intune. Pas de variabelen aan om te voldoen aan de specifieke vereisten van je organisatie.
