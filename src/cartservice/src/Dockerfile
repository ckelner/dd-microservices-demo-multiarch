# Copyright 2021 Google LLC
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#      http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

# https://mcr.microsoft.com/v2/dotnet/sdk/tags/list
FROM --platform=$BUILDPLATFORM mcr.microsoft.com/dotnet/sdk:8.0-preview-jammy as build

ARG \
    TARGETPLATFORM \
    TARGETARCH \
    BUILDPLATFORM

ARG CACHE_DATE=2016-01-01

WORKDIR /app
# takes too long to run
# COPY . .

COPY cartservice.csproj .
# RUN dotnet restore cartservice.csproj
# but we also need some protos, I don't know why...
# COPY ./protos/Cart.proto ./protos/Cart.proto
# RUN dotnet publish cartservice.csproj -o /out
# RUN dotnet restore cartservice.csproj -a $TARGETARCH
RUN dotnet restore cartservice.csproj -r linux-musl-x64
COPY . .
# RUN dotnet publish cartservice.csproj -p:PublishSingleFile=true \
#     --self-contained true -p:PublishTrimmed=True -p:TrimMode=Link \
#     -a $TARGETARCH -c release -o /out --no-restore
RUN dotnet publish cartservice.csproj -p:PublishSingleFile=true \
    --self-contained true -p:PublishTrimmed=True -p:TrimMode=Link \
    -r linux-musl-x64 -c release -o /out --no-restore

# https://mcr.microsoft.com/v2/dotnet/aspnet/tags/list
# https://mcr.microsoft.com/v2/dotnet/runtime-deps/tags/list
# Building final image used in running container
FROM mcr.microsoft.com/dotnet/aspnet:8.0-preview-jammy AS final

ARG \
    TARGETPLATFORM \
    BUILDPLATFORM

ARG CACHE_DATE=2016-01-01

# RUN apt-get update && apt-get install -y wget
# # renovate: datasource=github-releases depName=grpc-ecosystem/grpc-health-probe
# RUN GRPC_HEALTH_PROBE_VERSION=v0.4.14 && \
#     wget -qO/bin/grpc_health_probe https://github.com/grpc-ecosystem/grpc-health-probe/releases/download/${GRPC_HEALTH_PROBE_VERSION}/grpc_health_probe-linux-amd64 && \
#     chmod +x /bin/grpc_health_probe

WORKDIR /app
COPY --from=build /out .
EXPOSE 7070
ENV DOTNET_EnableDiagnostics=0 \
    ASPNETCORE_URLS=http://*:7070
USER 1000

# RUN chmod +x /app/cartservice

ENTRYPOINT ["/app/cartservice"]
