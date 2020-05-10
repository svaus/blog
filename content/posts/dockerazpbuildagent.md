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

Let us add the powershell scripts. Note that you can get latest powershell from microsoft docs.

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

