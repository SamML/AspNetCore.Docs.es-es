---
title: Imágenes de Docker para ASP.NET Core
author: tdykstra
description: Obtenga información sobre cómo usar las imágenes de Docker de .NET Core publicadas desde el registro de Docker. Extraiga imágenes y cree las suyas propias.
ms.author: tdykstra
ms.custom: mvc
ms.date: 06/18/2019
uid: host-and-deploy/docker/building-net-docker-images
ms.openlocfilehash: ea96ae6d36c7e8320ea49e666a807ece72645865
ms.sourcegitcommit: a1283d486ac1dcedfc7ea302e1cc882833e2c515
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/18/2019
ms.locfileid: "67207789"
---
# <a name="docker-images-for-aspnet-core"></a>Imágenes de Docker para ASP.NET Core

En este tutorial se muestra cómo ejecutar una aplicación ASP.NET Core en contenedores de Docker.

En este tutorial ha:
> [!div class="checklist"]
> * Obtener información sobre las imágenes de Microsoft .NET Core Docker 
> * Descargará una aplicación de ejemplo de ASP.NET Core
> * Ejecutará la aplicación de ejemplo localmente
> * Ejecutará la aplicación de ejemplo en contenedores de Linux
> * Ejecutará la aplicación de ejemplo en contenedores de Windows
> * Realizará compilaciones e implementaciones manualmente

## <a name="aspnet-core-docker-images"></a>Imágenes de Docker para ASP.NET Core

En este tutorial, descargará una aplicación de ejemplo de ASP.NET Core y la ejecutará en contenedores de Docker. El ejemplo funciona con contenedores de Linux y Windows.

El Dockerfile de ejemplo usa la [característica de compilación en varias fases de Docker](https://docs.docker.com/engine/userguide/eng-image/multistage-build/) para la compilación y ejecución en distintos contenedores. Los contenedores de compilación y ejecución se crean a partir de imágenes que proporciona Microsoft en Docker Hub:

* `dotnet/core/sdk`

  En el ejemplo se usa esta imagen para compilar la aplicación. La imagen contiene el SDK de .NET Core, que incluye las herramientas de línea de comandos (CLI). Esta imagen está optimizada para el desarrollo local, la depuración y las pruebas unitarias. Las herramientas instaladas para el desarrollo y la compilación hacen que esta sea una imagen relativamente grande. 

* `dotnet/core/aspnet` 

   En el ejemplo se usa esta imagen para ejecutar la aplicación. La imagen contiene el entorno de ejecución y las bibliotecas de ASP.NET Core y está optimizada para la ejecución de aplicaciones en producción. Diseñada para acelerar la implementación y el inicio de las aplicaciones, la imagen es relativamente pequeña, de forma que se optimiza el rendimiento de la red desde Docker Registry hasta el host de Docker. Solo se copian en el contenedor los archivos binarios y el contenido necesario para ejecutar una aplicación. El contenido está listo para ejecutarse, lo que permite el tiempo más rápido desde `Docker run` hasta el inicio de la aplicación. En el modelo de Docker no se necesita compilación de código dinámico.

## <a name="prerequisites"></a>Requisitos previos

* [SDK de .NET Core 2.2](https://www.microsoft.com/net/core)

* Cliente de Docker 18.03 o posterior

  * Distribuciones de Linux
    * [CentOS](https://docs.docker.com/install/linux/docker-ce/centos/)
    * [Debian](https://docs.docker.com/install/linux/docker-ce/debian/)
    * [Fedora](https://docs.docker.com/install/linux/docker-ce/fedora/)
    * [Ubuntu](https://docs.docker.com/install/linux/docker-ce/ubuntu/)
  * [macOS](https://docs.docker.com/docker-for-mac/install/)
  * [Windows](https://docs.docker.com/docker-for-windows/install/)

* [Git](https://git-scm.com/download)

## <a name="download-the-sample-app"></a>Descarga de la aplicación de ejemplo

* Para descargar el ejemplo, clone el [repositorio de Docker de .NET Core](https://github.com/dotnet/dotnet-docker): 

  ```console
  git clone https://github.com/dotnet/dotnet-docker
  ```

## <a name="run-the-app-locally"></a>Probar la aplicación localmente

* Vaya a la carpeta de proyecto en *dotnet-docker/samples/aspnetapp/aspnetapp*.

* Ejecute el siguiente comando para compilar y ejecutar localmente la aplicación:

  ```console
  dotnet run
  ```

* Vaya a `http://localhost:5000` en un explorador para probar la aplicación.

* Presione Ctrl+C en el símbolo del sistema para detener la aplicación.

## <a name="run-in-a-linux-container"></a>Ejecución en un contenedor de Linux

* En el cliente de Docker, cambie a contenedores de Linux.

* Vaya a la carpeta de Dockerfile en *dotnet-docker/samples/aspnetapp*.

* Ejecute los siguientes comandos para compilar y ejecutar el ejemplo en Docker:

  ```console
  docker build -t aspnetapp .
  docker run -it --rm -p 5000:80 --name aspnetcore_sample aspnetapp
  ```

  Los argumentos del comando `build`:
  * Asigne a la imagen el nombre aspnetapp.
  * Busque el archivo Dockerfile en la carpeta actual (el punto al final).

  Los argumentos del comando de ejecución:
  * Asigne un pseudo-TTY y manténgalo abierto aunque no esté asociado. (El mismo efecto que `--interactive --tty`).
  * Quite automáticamente el contenedor cuando se cierre.
  * Asigne al puerto 5000 de la máquina local el puerto 80 en el contenedor.
  * Asigne al contenedor el nombre aspnetcore_sample.
  * Especifique la imagen aspnetapp.

* Vaya a `http://localhost:5000` en un explorador para probar la aplicación.

## <a name="run-in-a-windows-container"></a>Ejecución en un contenedor de Windows

* En el cliente de Docker, cambie a los contenedores de Windows.

Vaya a la carpeta de archivos de Docker en `dotnet-docker/samples/aspnetapp`.

* Ejecute los siguientes comandos para compilar y ejecutar el ejemplo en Docker:

  ```console
  docker build -t aspnetapp .
  docker run -it --rm --name aspnetcore_sample aspnetapp
  ```

* Para contenedores de Windows, necesita la dirección IP del contenedor (no sirve ir a `http://localhost:5000`):
  * Abra otro símbolo del sistema.
  * Ejecute `docker ps` para ver los contenedores en ejecución. Compruebe que aparece el contenedor "aspnetcore_sample".
  * Ejecute `docker exec aspnetcore_sample ipconfig` para mostrar la dirección IP del contenedor. La salida del comando es similar a este ejemplo:

    ```console
    Ethernet adapter Ethernet:

       Connection-specific DNS Suffix  . : contoso.com
       Link-local IPv6 Address . . . . . : fe80::1967:6598:124:cfa3%4
       IPv4 Address. . . . . . . . . . . : 172.29.245.43
       Subnet Mask . . . . . . . . . . . : 255.255.240.0
       Default Gateway . . . . . . . . . : 172.29.240.1
    ```

* Copie la dirección IPv4 (por ejemplo, 172.29.245.43) del contenedor y péguela en la barra de direcciones del explorador para probar la aplicación.

## <a name="build-and-deploy-manually"></a>Compilaciones e implementaciones manuales

En algunos escenarios, puede que quiera implementar una aplicación en un contenedor mediante la copia de los archivos de aplicación que son necesarios en tiempo de ejecución. En esta sección se muestra cómo realizar una implementación manual.

* Vaya a la carpeta de proyecto en *dotnet-docker/samples/aspnetapp/aspnetapp*.

* Ejecute el comando [dotnet publish](/dotnet/core/tools/dotnet-publish):

  ```console
  dotnet publish -c Release -o published
  ```

  Los argumentos del comando:
  * Compile la aplicación en modo de versión (el valor predeterminado es modo de depuración).
  * Cree los archivos en la carpeta *publicada*.

* Ejecute la aplicación.

  * Windows:

    ```console
    dotnet published\aspnetapp.dll
    ```

  * Linux:

    ```bash
    dotnet published/aspnetapp.dll
    ```

* Vaya a `http://localhost:5000` para ver la página principal.

### <a name="the-dockerfile"></a>El archivo Dockerfile

Aquí el archivo Dockerfile se usa con el comando `docker build` que ejecutó anteriormente.  Se usa `dotnet publish` de la misma manera que en esta sección para realizar compilaciones e implementaciones.  

```console
FROM mcr.microsoft.com/dotnet/core/sdk:2.2 AS build
WORKDIR /app

# copy csproj and restore as distinct layers
COPY *.sln .
COPY aspnetapp/*.csproj ./aspnetapp/
RUN dotnet restore

# copy everything else and build app
COPY aspnetapp/. ./aspnetapp/
WORKDIR /app/aspnetapp
RUN dotnet publish -c Release -o out


FROM mcr.microsoft.com/dotnet/core/aspnet:2.2 AS runtime
WORKDIR /app
COPY --from=build /app/aspnetapp/out ./
ENTRYPOINT ["dotnet", "aspnetapp.dll"]
```

## <a name="additional-resources"></a>Recursos adicionales

* [Comando de compilación de Docker](https://docs.docker.com/engine/reference/commandline/build)
* [Comando de ejecución de Docker](https://docs.docker.com/engine/reference/commandline/run)
* [Ejemplo de Docker de ASP.NET Core](https://github.com/dotnet/dotnet-docker) (el usado en este tutorial).
* [Configurar ASP.NET Core para trabajar con servidores proxy y equilibradores de carga](/aspnet/core/host-and-deploy/proxy-load-balancer)
* [Working with Visual Studio Docker Tools](https://docs.microsoft.com/aspnet/core/publishing/visual-studio-tools-for-docker) (Trabajo con Visual Studio Docker Tools)
* [Depuración con Visual Studio Code](https://code.visualstudio.com/docs/nodejs/debugging-recipes#_debug-nodejs-in-docker-containers) 

## <a name="next-steps"></a>Pasos siguientes

En este tutorial ha:
> [!div class="checklist"]
> * Obtener información sobre las imágenes de Microsoft .NET Core Docker 
> * Descargado una aplicación de ejemplo de ASP.NET Core
> * Ejecutado la aplicación de ejemplo localmente
> * Ejecutado la aplicación de ejemplo en contenedores de Linux
> * Ejecutado el ejemplo con contenedores de Windows
> * Realizado compilaciones e implementaciones manualmente

El repositorio de Git que contiene la aplicación de ejemplo también incluye documentación. Para información general de los recursos disponibles en el repositorio, consulte [el archivo Léame](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/README.md). En concreto, vea cómo implementar HTTPS:

> [!div class="nextstepaction"]
> [Developing ASP.NET Core Applications with Docker over HTTPS](https://github.com/dotnet/dotnet-docker/blob/master/samples/aspnetapp/aspnetcore-docker-https-development.md) (Desarrollo de aplicaciones ASP.NET Core con Docker a través de HTTPS)
