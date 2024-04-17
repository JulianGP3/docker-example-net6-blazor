# README: Ejecutar una imagen Docker

Este archivo README proporciona instrucciones sobre cómo ejecutar una imagen Docker utilizando los comandos adecuados.

## Paso 1: Obtener la imagen

Si no has descargado la imagen Docker previamente, puedes hacerlo ejecutando el siguiente comando:

```bash
docker pull juliangp3/demoblazorserverapp:latest
```

## Paso 2: Ejecutar la imagen con Docker Compose

Puedes utilizar el siguiente archivo `docker-compose.yml` para ejecutar la imagen junto con sus dependencias:

```yaml
version: '3.4'

networks:
  demoblazorapp:

services:
  demoappdb:
    container_name: app-db
    image: mcr.microsoft.com/mssql/server:2019-latest
    ports:
      - 8002:1433
    environment:
      - ACCEPT_EULA=Y
      - SA_PASSWORD=password@12345#
    networks:
      - demoblazorapp
  demoblazorserverapp:
    container_name: demo-blazor-app
    image: juliangp3/demoblazorserverapp
    ports:
      - 8001:80
    depends_on:
      - demoappdb
    environment:
      - DB_HOST=demoappdb
      - DB_NAME=DemoBlazorApp
      - DB_SA_PASSWORD=password@12345#
    networks:
      - demoblazorapp
```

Para ejecutar Docker Compose, utiliza el siguiente comando:

```bash
docker-compose up -d  
```

Después de este comando correrá las imágenes creadas y el contenedor quedará arriba.  
Navega a la siguiente URL: http://localhost:8001

## Paso 3: Dockerfile

El siguiente es el `Dockerfile` utilizado para construir la imagen `juliangp3/demoblazorserverapp`:

```Dockerfile
#See https://aka.ms/containerfastmode to understand how Visual Studio uses this Dockerfile to build your images for faster debugging.

FROM mcr.microsoft.com/dotnet/aspnet:6.0 AS base
WORKDIR /app
EXPOSE 80

FROM mcr.microsoft.com/dotnet/sdk:6.0 AS build
WORKDIR /src
COPY ["DemoBlazorServerApp/DemoBlazorServerApp.csproj", "DemoBlazorServerApp/"]
RUN dotnet restore "DemoBlazorServerApp/DemoBlazorServerApp.csproj"
COPY . .
WORKDIR "/src/DemoBlazorServerApp"
RUN dotnet build "DemoBlazorServerApp.csproj" -c Release -o /app/build

FROM build AS publish
RUN dotnet publish "DemoBlazorServerApp.csproj" -c Release -o /app/publish

FROM base AS final
WORKDIR /app
COPY --from=publish /app/publish .
ENTRYPOINT ["dotnet", "DemoBlazorServerApp.dll"]
```