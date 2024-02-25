FROM mcr.microsoft.com/dotnet/sdk:8.0 AS build-env
WORKDIR /app

# Copy the CSPROJ file and restore any NuGet packages.
COPY *.sln .
COPY YourApiProjectName/*.csproj ./YourApiProjectName/
RUN dotnet restore

# Copy the rest of the source code.
COPY . ./

# Build the application.
WORKDIR /app/YourApiProjectName
RUN dotnet publish -c Release -o out

# Build runtime image by using the official Microsoft .NET runtime image.
FROM mcr.microsoft.com/dotnet/aspnet:8.0
WORKDIR /app
COPY --from=build-env /app/YourApiProjectName/out .

# Expose the port the API is listening on.
EXPOSE 80
EXPOSE 443

# Start the API.
ENTRYPOINT ["dotnet", "YourApiProjectName.dll"]