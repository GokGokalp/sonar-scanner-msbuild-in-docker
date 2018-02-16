# sonar-scanner-msbuild-in-docker
SonarQube Scanner for MSBuild to .NET Core projects in Docker

https://hub.docker.com/r/gokgokalp/sonar-scanner-dotnetcore-msbuild/

Usage:
-----
Sample Dockerfile for your projects:

```dockerfile
FROM gokgokalp/sonar-scanner-dotnetcore-msbuild:latest AS build-env

ARG NUGET_SOURCE
ARG PROJECT_KEY
ARG PROJECT_NAME
ARG PROJECT_VERSION
ARG SONAR_HOST
ARG SONAR_LOGIN_KEY
ARG NUGET_SOURCE

COPY . .
WORKDIR /build_dir

RUN dotnet restore --source $NUGET_SOURCE

# Sonar scanner start
RUN mono /opt/sonar-scanner-msbuild/SonarQube.Scanner.MSBuild.exe begin /d:sonar.host.url="$SONAR_HOST" /d:sonar.login="$SONAR_LOGIN_KEY" /k:"$PROJECT_KEY" /n:"$PROJECT_NAME" /v:"$PROJECT_VERSION"

RUN dotnet msbuild /t:rebuild ./src/SampleProject.csproj
RUN dotnet test ./tests/SampleProject.Test.csproj

RUN mono /opt/sonar-scanner-msbuild/SonarQube.Scanner.MSBuild.exe end /d:sonar.login="$SONAR_LOGIN_KEY"
# Sonar scanner end

# Publish artifacts
RUN dotnet publish ./src/SampleProject.csproj -o /publish

# Create the image
FROM microsoft/aspnetcore:2.0
COPY --from=build-env /publish /publish
WORKDIR /publish
ENTRYPOINT ["dotnet", "SampleProject.dll"]
```