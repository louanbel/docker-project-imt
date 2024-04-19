FROM --platform=${BUILDPLATFORM} mcr.microsoft.com/dotnet/sdk:7.0 AS build
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM

RUN echo "TARGETPLATFORM=${TARGETPLATFORM}, TARGETARCH=${TARGETARCH}, BUILDPLATFORM=${BUILDPLATFORM}"

WORKDIR /source

COPY *.csproj ./
RUN dotnet restore -a $TARGETARCH

COPY . .

RUN dotnet publish -c release -o /app -a $TARGETARCH --self-contained false --no-restore

FROM --platform=${BUILDPLATFORM} mcr.microsoft.com/dotnet/runtime:7.0 AS runtime
WORKDIR /app

COPY --from=build /app .

EXPOSE 5000

ENTRYPOINT ["dotnet", "Worker.dll"]
