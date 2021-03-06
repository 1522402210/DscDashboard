# Dockerfile that describes an Ubuntu18.04 image with PowerShell, SQL Server ODBC Driver and
# MS-SQL tools installed from Microsoft APT Repo
#
# Two additional PowerShel Modules UniversalDashboard.Community and DscDashboard are added
# for the Web Frontend and Backend, respectively

# Start with the Ubuntu 18.04 base image for PowerShell as it is also supported by the 
# Microsoft ODBC Driver 17 for SQL Server
FROM mcr.microsoft.com/powershell:6.1.0-ubuntu-18.04

# DscDashboard Docker Version
ARG VERSION=0.0.1
ARG IMAGE_NAME=dscdashboard:${VERSION}

LABEL maintainer="NetwiZe.be" \
      readme.md="https://github.com/fvanroie/DscDashboard/blob/master/README.md" \
      description="This Dockerfile will an image for hosting DscDashboard." \
      org.label-schema.vcs-url="https://github.com/fvanroie/DscDashboard/" \
      org.label-schema.name="dscdashboard" \
      org.label-schema.vendor="NetwiZe.be" \
      org.label-schema.version=${VERSION} \
      org.label-schema.docker.cmd.devel="docker run ${IMAGE_NAME} pwsh" \
      org.label-schema.docker.cmd="docker run ${IMAGE_NAME}" \
      org.label-schema.vcs-ref=${VCS_REF} \
      org.label-schema.schema-version="1.0" \
      org.label-schema.usage="https://github.com/fvanroie/DscDashboard/master/docker#run-the-docker-image-you-built" \
      org.label-schema.url="https://github.com/fvanroie/DscDashboard/master/docker/README.md" \
      org.label-schema.docker.cmd.test="docker run ${IMAGE_NAME} pwsh" \
      org.label-schema.docker.cmd.help="docker run ${IMAGE_NAME} pwsh --help"

# Suppress Dialog warning from msodbcsql17 and mssql-tools packages because we are running unattended
ENV DEBIAN_FRONTEND noninteractive

# Install msodbcsql17 mssql-tools unixodbc
RUN apt-get update \
    && ACCEPT_EULA=Y apt-get install msodbcsql17 mssql-tools unixodbc -y --no-install-recommends \
    && apt-get dist-upgrade -y \
    && apt-get clean \
    && rm -rf /var/lib/apt/lists/*

# Copy local PowerShell Modules into the image
COPY './DscDashboard/' /usr/local/share/powershell/Modules/DscDashboard
COPY ./UniversalDashboard.Community/ /usr/local/share/powershell/Modules/UniversalDashboard.Community

# Set Environment Variables
ENV DSC_CONTAINER_VERSION ${VERSION}
ENV DSC_CONNECTIONSTRING "DRIVER={ODBC Driver 17 for SQL Server}; Server={0}; UID={1}; PWD={2}; Database=DSC;"

# Check if PowerShell can run and find the modules
RUN pwsh -c "If ((Get-Module 'DscDashboard','UniversalDashboard.Community' -ListAvailable).Count -ne 2) { Throw 'PowerShell Modules are not available. Place the DscDashboard and UniversalDashboard.Community folders in the local path and try again!' }" \
    && pwsh -c 'Write-Host "DSC_CONNECTIONSTRING = $env:DSC_CONNECTIONSTRING"' \
    && ln -s /usr/local/share/powershell/Modules/DscDashboard/dashboard.ps1 /dashboard.ps1

# The container will run the dashboard.ps1 file
ENTRYPOINT [ "pwsh" ]
CMD [ "/dashboard.ps1" ]
