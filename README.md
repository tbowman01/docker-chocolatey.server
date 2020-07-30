Chocolatey Simple Server
========================

Docker Hub: https://hub.docker.com/r/TheBlackMini/chocolateyserver/

Forked from: takhyon/docker-chocolatey.server

## Overview

Run a chocolatey simple server inside a container.

## How To Use This Image

Download the image:
```
docker pull TheBlackMini/chocolateyserver
```

HTTP: Start the container:
```
docker run -d -p 80:80 --name chocolatey.server theblackmini/chocolateyserver
```

### Better Yet - Seperate storage container

Create a data container to store your nupkg files:
```
docker create -v C:/tools/chocolatey.server/App_Data/Packages --name chocolatey.server-data microsoft/nanoserver
```

HTTP: Start your app container using your data container:
```
docker run -d -p 80:80 --volumes-from chocolatey.server-data --name chocolatey.server theblackmini/chocolateyserver
```

## Get Chocolatey

Push packages:
```
choco push <nupkg_file> --source=<chocolatey.server_url> --api-key=<api_key> --force
```

Install Packages
```
choco install <package_name> --source=<chocolatey.server_url>/chocolatey/
```

## How To Build the SSL Image

Request a certificate (from internal PKI or Self-Signed):

Self-Signed:
```
New-SelfSignedCertificate -DnsName "<DNS Servername>" -CertStoreLocation "Cert:\LocalMachine\My"
Export-PfxCertificate -Password (ConvertTo-SecureString -String "<Password>" -AsPlainText -Force) -FilePath <Export.pfx Path> -Cert Cert:\LocalMachine\My\<Certificate Thumbprint>
```

Build the container:

Download the Dockerfile for the SSL implementation and put in the same folder as the pfx you exported earlier (<Export.pfx Path>)
```
docker build <Export.pfx Path> -t <Tag Name>
```

## How To Use The SSL Image

Start the container:
```
docker run -d -p 443:443 --name chocolatey.server <Tag Name>
```

### Better Yet - Seperate storage container

Create a data container to store your nupkg files:
```
docker create -v C:/tools/chocolatey.server/App_Data/Packages --name chocolatey.server-data microsoft/nanoserver
```

HTTP: Start your app container using your data container:
```
docker run -d -p 443:443 --volumes-from chocolatey.server-data --name chocolatey.server <Tag Name>
```
