# Use the .NET SDK image to build the project
FROM --platform=${BUILDPLATFORM} mcr.microsoft.com/dotnet/sdk:7.0 AS build
ARG TARGETPLATFORM
ARG TARGETARCH
ARG BUILDPLATFORM

# Display build arguments
RUN echo "TARGETPLATFORM=${TARGETPLATFORM}, TARGETARCH=${TARGETARCH}, BUILDPLATFORM=${BUILDPLATFORM}"

# Set the working directory
WORKDIR /source

# Copy csproj file and restore dependencies
COPY *.csproj ./
RUN dotnet restore -a $TARGETARCH

# Copy the rest of the source files
COPY . .

# Publish the application
RUN dotnet publish -c release -o /app -a $TARGETARCH --self-contained false --no-restore

# Use the .NET runtime image to run the project
FROM --platform=${BUILDPLATFORM} mcr.microsoft.com/dotnet/runtime:7.0 AS runtime
WORKDIR /app

# Copy the published application from the build image
COPY --from=build /app .

# Command to run the application
ENTRYPOINT ["dotnet", "Worker.dll"]
