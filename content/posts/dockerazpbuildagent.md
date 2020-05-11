---
title: "Docker azure pipeline build agent"
date: 2020-05-10T16:08:37+10:00
draft: false
---  

Creating a docker build agent for azure pipeline is very easy. There are few simple steps. We will be using chocoletey to create this agent. 

Firstly create a Dockerfile where we can provide docker instructions. In this case I am using windows core 2019 image, but you can use any other images. 

```Dockerfile
FROM mcr.microsoft.com/windows/servercore:ltsc2019
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]
WORKDIR /azp

COPY InstallChoco.ps1 .
RUN .\InstallChoco.ps1 

COPY InstallDotNetCoreSdkChoco.ps1 .
RUN .\InstallDotNetCoreSdkChoco.ps1

COPY start.ps1 .
CMD powershell .\start.ps1

```

1. Start from a base image it could be ltsc2019 or ltsc2016 or ubuntu image.
2. Set Powershll as the shell command 
3. Set WORKDIR - working directory to azp or something of your choice
4. Copy and run 3 powershell scripts `InstallChoco.ps1` , `InstallDotNetCoreSdkChoco.ps1` and `start.ps1` using the COPY and RUN instruction of Dockerfile

Let us add the powershell scripts. Note that you can get latest powershell from microsoft docs https://docs.microsoft.com/en-us/azure/devops/pipelines/agents/docker?view=azure-devops.

Save the below content to `start.ps1` file:

```Powershell
if (-not (Test-Path Env:AZP_URL)) {
  Write-Error "error: missing AZP_URL environment variable"
  exit 1
}

if (-not (Test-Path Env:AZP_TOKEN_FILE)) {
  if (-not (Test-Path Env:AZP_TOKEN)) {
    Write-Error "error: missing AZP_TOKEN environment variable"
    exit 1
  }

  $Env:AZP_TOKEN_FILE = "\azp\.token"
  $Env:AZP_TOKEN | Out-File -FilePath $Env:AZP_TOKEN_FILE
}

Remove-Item Env:AZP_TOKEN

if ($Env:AZP_WORK -and -not (Test-Path Env:AZP_WORK)) {
  New-Item $Env:AZP_WORK -ItemType directory | Out-Null
}

New-Item "\azp\agent" -ItemType directory | Out-Null

# Let the agent ignore the token env variables
$Env:VSO_AGENT_IGNORE = "AZP_TOKEN,AZP_TOKEN_FILE"

Set-Location agent

Write-Host "1. Determining matching Azure Pipelines agent..." -ForegroundColor Cyan

$base64AuthInfo = [Convert]::ToBase64String([Text.Encoding]::ASCII.GetBytes(":$(Get-Content ${Env:AZP_TOKEN_FILE})"))
$package = Invoke-RestMethod -Headers @{Authorization=("Basic $base64AuthInfo")} "$(${Env:AZP_URL})/_apis/distributedtask/packages/agent?platform=win-x64&`$top=1"
$packageUrl = $package[0].Value.downloadUrl

Write-Host $packageUrl

Write-Host "2. Downloading and installing Azure Pipelines agent..." -ForegroundColor Cyan

$wc = New-Object System.Net.WebClient
$wc.DownloadFile($packageUrl, "$(Get-Location)\agent.zip")

Expand-Archive -Path "agent.zip" -DestinationPath "\azp\agent"

try
{
  Write-Host "3. Configuring Azure Pipelines agent..." -ForegroundColor Cyan

  .\config.cmd --unattended `
    --agent "$(if (Test-Path Env:AZP_AGENT_NAME) { ${Env:AZP_AGENT_NAME} } else { ${Env:computername} })" `
    --url "$(${Env:AZP_URL})" `
    --auth PAT `
    --token "$(Get-Content ${Env:AZP_TOKEN_FILE})" `
    --pool "$(if (Test-Path Env:AZP_POOL) { ${Env:AZP_POOL} } else { 'Default' })" `
    --work "$(if (Test-Path Env:AZP_WORK) { ${Env:AZP_WORK} } else { '_work' })" `
    --replace

  # remove the administrative token before accepting work
  Remove-Item $Env:AZP_TOKEN_FILE

  Write-Host "4. Running Azure Pipelines agent..." -ForegroundColor Cyan

  .\run.cmd
}
finally
{
  Write-Host "Cleanup. Removing Azure Pipelines agent..." -ForegroundColor Cyan

  .\config.cmd remove --unattended `
    --auth PAT `
    --token "$(Get-Content ${Env:AZP_TOKEN_FILE})"
}
```

Create a new file named `InstallChoco.ps1`. This will be used to install choco in the docker images

```Powershell


[Net.ServicePointManager]::SecurityProtocol = [Net.ServicePointManager]::SecurityProtocol -bor "Tls12"

Invoke-WebRequest -Uri "https://chocolatey.org/install.ps1" -Outfile "chocoinstall.ps1"

.\chocoinstall.ps1

choco config set cachelocation C:\chococache

```

Once choco is added, you can add any capability by simply installing it with choco. So let us add dotnet core sdk via choco. Create a file named `InstallDotNetCoreSdkChoco.ps1` and add below code. 

``` Powershell
choco install dotnetcore-sdk -version 2.1.4 --confirm --limit-output 
choco install dotnetcore-sdk -version 2.1.7 --confirm --limit-output 
choco install dotnetcore-sdk -version 3.1.0 --confirm --limit-output 

```

# Creating docker container for azp

Build the docker images using docker build command.

```Powershell
docker build -t myazpagent:latest .
```

Once this is done, you can easily create the docker container by running below command. 

```Powershell
docker run -e AZP_URL=https://dev.azure.com/{username} -e AZP_TOKEN=$Env:AZP_TOKEN -e AZP_AGENT_NAME=$Env:AZP_Build_Agent_Name -e AZP_POOL=$Env:AZP_POOL myazpagent:latest

```

Have fun !!

