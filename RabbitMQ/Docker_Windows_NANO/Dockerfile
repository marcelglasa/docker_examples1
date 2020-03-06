# https://github.com/docker-library/rabbitmq/issues/168
FROM mcr.microsoft.com/windows/servercore:1809 as builder
LABEL maintainer "MG"

ARG OTP_VERSION=22.2
ARG RABBITMQ_VERSION=3.8.2

ENV OTP_VERSION=$OTP_VERSION \
  RABBITMQ_VERSION=$RABBITMQ_VERSION \
  #http://erlang.org/download/MD5
  erlang_download_MD5=30dd9f904d6fb43bc3ed98b792399026 \
  erlang_download_url="http://erlang.org/download/otp_win64_22.2.exe" \
  # rabbit_download_ASC="https://github.com/rabbitmq/rabbitmq-server/releases/download/v3.8.2/rabbitmq-server-windows-3.8.2.zip.asc" \
  # https://bintray.com/rabbitmq/all/rabbitmq-server/3.8.2
  rabbit_download_SHA256=a28fff6db7cb7eec5dbbaf120c9d02edff39d289f2cb3e64b5eabc6498ffb74c \
  rabbit_download_url="https://github.com/rabbitmq/rabbitmq-server/releases/download/v${RABBITMQ_VERSION}/rabbitmq-server-windows-${RABBITMQ_VERSION}.zip" \ 
  rabbitplugin_download_url="https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/download/v3.8.0/rabbitmq_delayed_message_exchange-3.8.0.ez" \
  handle_download_url="https://download.sysinternals.com/files/Handle.zip" \
  ERLANG_HOME=C:\\Program\ Files\\erl10.6_otp$OTP_VERSION \
  RABBITMQ_HOME=C:\\Program\ Files\\RabbitMQ\ Server\\rabbitmq_server-$RABBITMQ_VERSION

# setup powershell options for RUN commands
SHELL ["powershell", "-Command", "$ErrorActionPreference = 'Stop'; $ProgressPreference = 'SilentlyContinue';"]

# download and install erlang using silent install option, and remove installer when done
RUN Write-Host "Downloading Erlang OTP version: $($env:OTP_VERSION), from: $($env:erlang_download_url)"; \
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
  Invoke-WebRequest -Uri $env:erlang_download_url -OutFile erlang_install.exe; \
  Write-Host "Verifying MD5 $env:erlang_download_MD5"; \
  if ((Get-FileHash erlang_install.exe -Algorithm md5).Hash -ne $env:erlang_download_MD5) { Write-Host 'FAILED!'; exit 1; }; \
  Start-Process -Wait -NoNewWindow -FilePath .\erlang_install.exe -ArgumentList /S, /D=$env:ERLANG_HOME; \
  Remove-Item -Force erlang_install.exe; \
  Write-Host "Validating installation"; \
  Start-Process "$env:ERLANG_HOME\\bin\\erl" -NoNewWindow -Wait -ArgumentList @('-version');

# create $RABBITMQ_HOME parent dir, download rabbitmq, extract to $RABBITMQ_HOME, and remove zip file when done
RUN Write-Host "Downloading RabbitMQ version: $($env:RABBITMQ_VERSION), from: $($env:rabbit_download_url)"; \
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
  Invoke-WebRequest -Uri $env:rabbit_download_url -OutFile rabbitmq.zip; \
  Write-Host "Verifying SHA256 $env:rabbit_download_SHA256"; \
  if ((Get-FileHash rabbitmq.zip -Algorithm sha256).Hash -ne $env:rabbit_download_SHA256) { Write-Host 'FAILED!'; exit 1; }; \
  New-Item -ItemType Directory -Force -Path (Split-Path $env:RABBITMQ_HOME); \
  Expand-Archive -Path .\rabbitmq.zip -DestinationPath (Split-Path $env:RABBITMQ_HOME); \
  Remove-Item -Force rabbitmq.zip;

# download rabbitmq-delayed-message-exchange plugin 
# ( rabbitmq-delayed-message-exchange build that is compatible with these RabbitMQ versions: 3.8.x, 3.7.x )
# https://github.com/rabbitmq/rabbitmq-delayed-message-exchange/releases/
RUN Write-Host "Downloading RabbitMQ Delayed Message Plugin, from: $($env:rabbitplugin_download_url)"; \
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
  Invoke-WebRequest -Uri $env:rabbitplugin_download_url -OutFile $env:RABBITMQ_HOME\\plugins\\rabbitmq_delayed_message_exchange-3.8.0.ez;

# to report used file handles on Windows, handle.exe from sysinternals must be installed in your path.
RUN Write-Host "Downloading handle.exe / sysinternals, from: $($env:handle_download_url)"; \
  [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12; \
  Invoke-WebRequest -Uri $env:handle_download_url -OutFile Handle.zip; \
  Expand-Archive -Path .\Handle.zip -DestinationPath "C:\\Windows\\"; \
  Remove-Item -Force Handle.zip;

# https://www.rabbitmq.com/relocate.html#windows
ARG APPDATA=C:\\Users\\ContainerAdministrator\\AppData\\Roaming
ENV APPDATA=$APPDATA \
  RABBITMQ_BASE=$APPDATA\\RabbitMQ \
  RABBITMQ_HOME=C:\\Program\ Files\\RabbitMQ\ Server\\rabbitmq_server-$RABBITMQ_VERSION
#PATH=C:\Windows\system32;C:\Windows;C:\Windows\System32\Wbem;C:\Windows\System32\WindowsPowerShell\v1.0\;C:\Windows\System32\OpenSSH\;C:\Users\ContainerAdministrator\AppData\Local\Microsoft\WindowsApps;$RABBITMQ_HOME\\bin

#USER Administrator
# error: The user name or password is incorrect

RUN New-Item -ItemType Directory -Force -Path $env:RABBITMQ_BASE
COPY rabbitmq.conf $RABBITMQ_BASE
COPY enabled_plugins $RABBITMQ_BASE

# https://www.rabbitmq.com/clustering.html#erlang-cookie
# https://www.rabbitmq.com/windows-quirks.html 
#COPY .erlang.cookie C:\Users\ContainerAdministrator\.erlang.cookie
#RUN icacls C:\Users\ContainerAdministrator\.erlang.cookie /grant administrator:F
#RUN icacls %USERPROFILE%\.erlang.cookie /grant %USERNAME%:F

# persistent volume for RabbitMQ data
#VOLUME $RABBITMQ_BASE\\db

# Ports
# 4369: epmd, a peer discovery service used by RabbitMQ nodes and CLI tools
# 5672: used by AMQP 0-9-1 and 1.0 clients without TLS
# 5671: used by AMQP 0-9-1 and 1.0 clients with TLS
# 25672: used by Erlang distribution for inter-node and CLI tools communication and is allocated from a dynamic range (limited to a single port by default, computed as AMQP port + 20000).
# 15672: HTTP API clients and rabbitmqadmin (only if the management plugin is enabled)
EXPOSE 4369 5671 5672 25672 15672

# setup working directory
WORKDIR $RABBITMQ_HOME\\sbin

# setup cmd options for RUN commands
SHELL ["cmd", "/S", "/C"]

RUN setx PATH "%PATH%;%RABBITMQ_HOME%\sbin;" /M

RUN rabbitmq-plugins.bat enable --offline rabbitmq_management rabbitmq_delayed_message_exchange rabbitmq_consistent_hash_exchange

# send all logs to TTY
# ENV RABBITMQ_LOGS=- RABBITMQ_SASL_LOGS=-

# run server when container starts - container will shutdown when this process ends
CMD rabbitmq-server.bat
#
# docker container exec CONTAINER_ID cmd /c rabbitmqctl status
#
# Windows Server CORE 1809
##################################################




##################################################
#
# Windows NANO Server 1809
#
FROM mcr.microsoft.com/powershell:nanoserver-1809

# Default Locations on Windows
# https://www.rabbitmq.com/relocate.html#windows
ARG APPDATA=C:\\Users\\ContainerAdministrator\\AppData\\Roaming
ARG OTP_VERSION=22.2
ARG RABBITMQ_VERSION=3.8.2

# https://www.rabbitmq.com/clustering.html#erlang-cookie
# https://www.rabbitmq.com/windows-quirks.html
ENV APPDATA=$APPDATA \
  OTP_VERSION=$OTP_VERSION \
  RABBITMQ_VERSION=$RABBITMQ_VERSION \
  ERLANG_HOME=C:\\Program\ Files\\erl10.6_otp$OTP_VERSION \
  RABBITMQ_HOME=C:\\Program\ Files\\RabbitMQ\ Server\\rabbitmq_server-$RABBITMQ_VERSION \
  RABBITMQ_BASE=$APPDATA\\RabbitMQ \
  HOMEDRIVE=C: \
  HOMEPATH=\\Users\\ContainerAdministrator \
  RABBITMQ_NODENAME=rabbit@localhost

USER ContainerAdministrator

# copy erlang and rabbitmq
COPY --from=builder $ERLANG_HOME $ERLANG_HOME
COPY --from=builder $RABBITMQ_HOME $RABBITMQ_HOME
COPY --from=builder $RABBITMQ_BASE $RABBITMQ_BASE

# persistent volume for RabbitMQ data
# VOLUME $RABBITMQ_BASE\\db

# Ports
# 4369: epmd, a peer discovery service used by RabbitMQ nodes and CLI tools
# 5672: used by AMQP 0-9-1 and 1.0 clients without TLS
# 5671: used by AMQP 0-9-1 and 1.0 clients with TLS
# 25672: used by Erlang distribution for inter-node and CLI tools communication and is allocated from a dynamic range (limited to a single port by default, computed as AMQP port + 20000).
# 15672: HTTP API clients and rabbitmqadmin (only if the management plugin is enabled)
EXPOSE 4369 5671 5672 25672 15672

# https://www.rabbitmq.com/clustering.html#erlang-cookie
# https://www.rabbitmq.com/windows-quirks.html 
#COPY .erlang.cookie $HOMEDRIVE$HOMEPATH
#RUN icacls %HOMEDRIVE%%HOMEPATH%\\.erlang.cookie /grant Administrator:F

# setup working directory
WORKDIR $RABBITMQ_HOME\\sbin

# setup cmd options for RUN commands
SHELL ["cmd", "/S", "/C"]

RUN setx PATH "%PATH%;%RABBITMQ_HOME%\sbin;" /M

# enable plugins and validate installation
RUN rabbitmq-plugins.bat enable --offline rabbitmq_management rabbitmq_delayed_message_exchange rabbitmq_consistent_hash_exchange

# run external command when container starts to allow for additional setup
CMD rabbitmq-server.bat