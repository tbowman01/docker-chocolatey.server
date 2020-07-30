FROM microsoft/iis

LABEL Version=1.0

#create a folder on the container to hold the code
RUN powershell -NoProfile -Command \
    New-Item C:\Temp -type directory
#Set the newly created folder in docker as the working directory for subsequent commands
WORKDIR 'C:\Temp'
#Copy everything from where you are on host to the working directory in docker (this folder should contain your SSL cert)
COPY ./ .

RUN powershell -NoProfile -Command \
    Add-WindowsFeature NET-Framework-45-ASPNET; \
    Add-WindowsFeature Web-Net-Ext45; \
    Add-WindowsFeature Web-Asp-Net45; \
    Add-WindowsFeature Web-ISAPI-Ext; \
    Add-WindowsFeature Web-ISAPI-Filter

RUN powershell -NoProfile -Command \
    $env:chocolateyUseWindowsCompression = 'false'; \
    "Invoke-WebRequest https://chocolatey.org/install.ps1 -UseBasicParsing | Invoke-Expression"

RUN powershell -NoProfile -Command \
    choco install chocolatey.server -y; \
    Remove-Item -Path C:\tools\chocolatey.server\App_Data\Packages\Readme.txt

RUN powershell -NoProfile -Command \
    (Get-Content -Path "C:\tools\chocolatey.server\Web.config") -replace ('chocolateyrocks','somethingSecure') ^| Out-File -filepath C:\tools\chocolatey.server\Web.config -Encoding "UTF8"

RUN powershell -NoProfile -Command \
    New-WebAppPool -Name chocolatey.server; \
    Set-ItemProperty "IIS:\AppPools\chocolatey.server" -Name "processModel.loadUserProfile" -Value "True"

RUN powershell -NoProfile -Command \
    Import-Module IISAdministration; \
    Import-Module WebAdministration; \
    $pwd = ConvertTo-SecureString -String 'P@ssw0rd1' -Force -AsPlainText; \
    $cert = Import-PfxCertificate -Exportable -FilePath 'C:\Temp\choco.pfx' -CertStoreLocation Cert:\LocalMachine\My -Password $pwd; \
    Remove-WebSite -Name 'Default Web Site'; \
    New-IISSite -Name "Chocolatey" -PhysicalPath C:\tools\chocolatey.server -BindingInformation "*:443:" -CertificateThumbPrint $cert.Thumbprint -CertStoreLocation "Cert:\LocalMachine\My" -Protocol https; \
    Set-ItemProperty "IIS:\Sites\Chocolatey" ApplicationPool chocolatey.server

EXPOSE 443
