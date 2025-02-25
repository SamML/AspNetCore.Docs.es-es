---
title: Comprobaciones de estado en ASP.NET Core
author: guardrex
description: Obtenga información sobre cómo configurar las comprobaciones de estado para la infraestructura de ASP.NET Core, como las aplicaciones y las bases de datos.
monikerRange: '>= aspnetcore-2.2'
ms.author: riande
ms.custom: mvc
ms.date: 04/23/2019
uid: host-and-deploy/health-checks
ms.openlocfilehash: 5119267a8da5c950989b14b7c2e818aa22806506
ms.sourcegitcommit: 5b0eca8c21550f95de3bb21096bd4fd4d9098026
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2019
ms.locfileid: "64887930"
---
# <a name="health-checks-in-aspnet-core"></a>Comprobaciones de estado en ASP.NET Core

Por [Luke Latham](https://github.com/guardrex) y [Glenn Condron](https://github.com/glennc)

ASP.NET Core ofrece el Middleware de comprobaciones de estado y bibliotecas para informar sobre el estado de los componentes de la infraestructura de la aplicación.

Una aplicación se encarga de exponer las comprobaciones de estado como puntos de conexión HTTP. Los puntos de conexión de las comprobaciones de estado pueden configurarse para diversos escenarios de supervisión en tiempo real:

* Los orquestadores de contenedores y los equilibradores de carga pueden utilizar los sondeos de estado para comprobar el estado de una aplicación. Por ejemplo, para responder a una comprobación de estado con errores, es posible que un orquestador de contenedores detenga una implementación en curso o reinicie un contenedor. Como respuesta a una aplicación con estado incorrecto, es posible que un equilibrador de carga enrute el tráfico al margen de la instancia con errores hacia una instancia con estado correcto.
* El uso de la memoria, el disco y otros recursos del servidor físico puede supervisarse para determinar si el estado es correcto.
* Las comprobaciones de estado pueden probar las dependencias de una aplicación, como las bases de datos y los puntos de conexión de servicio externo, para confirmar la disponibilidad y el funcionamiento normal.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/host-and-deploy/health-checks/samples) ([cómo descargarlo](xref:index#how-to-download-a-sample))

La aplicación de muestra incluye ejemplos de los escenarios descritos en este tema. Para ejecutar la aplicación de ejemplo para un escenario determinado, use el comando [dotnet run](/dotnet/core/tools/dotnet-run) desde la carpeta del proyecto en un shell de comandos. Para obtener información sobre cómo utilizar la aplicación de ejemplo, consulte el archivo *README.md* de la aplicación de ejemplo y las descripciones de escenarios de este tema.

## <a name="prerequisites"></a>Requisitos previos

Normalmente, las comprobaciones de estado se usan con un servicio de supervisión externa o un orquestador de contenedores para comprobar el estado de una aplicación. Antes de agregar comprobaciones de estado a una aplicación, debe decidir en qué sistema de supervisión se va a usar. El sistema de supervisión determina qué tipos de comprobaciones de estado se deben crear y cómo configurar sus puntos de conexión.

Haga referencia al [metapaquete Microsoft.AspNetCore.App](xref:fundamentals/metapackage-app) o agregue una referencia de paquete al paquete [Microsoft.AspNetCore.Diagnostics.HealthChecks](https://www.nuget.org/packages/Microsoft.AspNetCore.Diagnostics.HealthChecks).

La aplicación de ejemplo proporciona código de inicio para mostrar las comprobaciones de estado para varios escenarios. En el escenario de [sondeo de base de datos](#database-probe) se comprueba el estado de una conexión de base de datos mediante [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks). El escenario [sondeo de DbContext](#entity-framework-core-dbcontext-probe) comprueba una base de datos mediante un elemento `DbContext` de EF Core. Para explorar los escenarios de la base de datos, la aplicación de ejemplo:

* Crea una base de datos y proporciona su cadena de conexión en el archivo *appsettings.json*.
* Tiene las siguientes referencias de paquete en su archivo de proyecto:
  * [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/)
  * [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/)

> [!NOTE]
> [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) es un puerto de [BeatPulse](https://github.com/xabaril/beatpulse) y Microsoft no lo mantiene ni lo admite.

Otro escenario de comprobación de estado muestra cómo filtrar las comprobaciones de estado por un puerto de administración. La aplicación de ejemplo requiere la creación de un archivo *Properties/launchSettings.json* que incluya la dirección URL de administración y el puerto de administración. Para obtener más información, consulte la sección [Filtrado por puerto](#filter-by-port).

## <a name="basic-health-probe"></a>Sondeo de estado básico

Para muchas aplicaciones, una configuración de sondeo de estado básico que notifique la disponibilidad de la aplicación para procesar las solicitudes (*ejecución*) es suficiente para detectar el estado de la aplicación.

La configuración básica registra los servicios de comprobación de estado y llama al Middleware de comprobaciones de estado para responder a un punto de conexión de dirección URL con una respuesta de estado. De forma predeterminada, no se registran comprobaciones de estado específicas para probar cualquier dependencia o subsistema concretos. La aplicación se considera correcta si es capaz de responder en la dirección URL de punto de conexión de estado. El escritor de respuesta predeterminado escribe el estado (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) como respuesta de texto no cifrado al cliente, que indica un estado [HealthStatus.Healthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus), [HealthStatus.Degraded](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) o [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus).

Registre los servicios de comprobación de estado con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> de `Startup.ConfigureServices`. Agregue el Middleware de comprobaciones de estado con <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> en la canalización de procesamiento de solicitudes de `Startup.Configure`.

En la aplicación de ejemplo, el punto de conexión de la comprobación de estado se crea en `/health` (*BasicStartup.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/BasicStartup.cs?name=snippet1&highlight=5,10)]

Para ejecutar el escenario de configuración básica mediante la aplicación de ejemplo, ejecute el comando siguiente desde la carpeta del proyecto en un shell de comandos:

```console
dotnet run --scenario basic
```

### <a name="docker-example"></a>Ejemplo de Docker

[Docker](xref:host-and-deploy/docker/index) ofrece una directiva de `HEALTHCHECK` integrada que puede utilizarse para comprobar el estado de una aplicación que use la configuración de comprobación de estado básica:

```
HEALTHCHECK CMD curl --fail http://localhost:5000/health || exit
```

## <a name="create-health-checks"></a>Creación de comprobaciones de estado

Las comprobaciones de estado se crean mediante la implementación de la interfaz de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck>. El método <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck.CheckHealthAsync*> devuelve un elemento `Task<` <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> `>` que indica el estado como `Healthy`, `Degraded` o `Unhealthy`. El resultado se escribe como una respuesta de texto no cifrado con un código de estado configurable. (La configuración se describe en la sección [Opciones de comprobación de estado](#health-check-options)). <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> también puede devolver pares clave-valor opcionales.

### <a name="example-health-check"></a>Comprobación de estado de ejemplo

La siguiente clase `ExampleHealthCheck` muestra el diseño de una comprobación de estado:

```csharp
public class ExampleHealthCheck : IHealthCheck
{
    public ExampleHealthCheck()
    {
        // Use dependency injection (DI) to supply any required services to the
        // health check.
    }

    public Task<HealthCheckResult> CheckHealthAsync(
        HealthCheckContext context,
        CancellationToken cancellationToken = default(CancellationToken))
    {
        // Execute health check logic here. This example sets a dummy
        // variable to true.
        var healthCheckResultHealthy = true;

        if (healthCheckResultHealthy)
        {
            return Task.FromResult(
                HealthCheckResult.Healthy("The check indicates a healthy result."));
        }

        return Task.FromResult(
            HealthCheckResult.Unhealthy("The check indicates an unhealthy result."));
    }
}
```

### <a name="register-health-check-services"></a>Registro de los servicios de comprobación de estado

El tipo `ExampleHealthCheck` se agrega a los servicios de comprobación de estado con <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*>:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddCheck<ExampleHealthCheck>("example_health_check");
}
```

La sobrecarga <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> que se muestra en el ejemplo siguiente establece el estado de error (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>) para notificar cuándo la comprobación de estado informa de un error. Si el estado de error se establece en `null` (valor predeterminado), [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) se notifica. Esta sobrecarga es un escenario útil para los creadores de bibliotecas en los que la aplicación ejecuta el estado de error indicado por la biblioteca cuando se produce un error de comprobación de estado si la implementación de la comprobación de estado respeta la configuración.

Las *etiquetas* pueden usarse para filtrar las comprobaciones de estado, que se describen con más detalle en la sección [Filtrado de las comprobaciones de estado](#filter-health-checks).

```csharp
services.AddHealthChecks()
    .AddCheck<ExampleHealthCheck>(
        "example_health_check",
        failureStatus: HealthStatus.Degraded,
        tags: new[] { "example" });
```

<xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> también puede ejecutar una función lambda. En el ejemplo siguiente, el nombre de la comprobación de estado se especifica como `Example` y la comprobación siempre devuelve un estado correcto:

```csharp
public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddCheck("Example", () =>
            HealthCheckResult.Healthy("Example is OK!"), tags: new[] { "example" })
}
```

### <a name="use-health-checks-middleware"></a>Uso del Middleware de comprobaciones de estado

En `Startup.Configure`, llame a <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> en la canalización de procesamiento con la dirección URL del punto de conexión o la ruta de acceso relativa:

```csharp
public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseHealthChecks("/health");
}
```

Si las comprobaciones de estado deben realizar la escucha en un puerto específico, use una sobrecarga de <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> para establecer el puerto, como se describe con más detalle en la sección [Filtrado por puerto](#filter-by-port):

```csharp
app.UseHealthChecks("/health", port: 8000);
```

El Middleware de comprobaciones de estado es un *middleware terminal* en la canalización de procesamiento de solicitudes de la aplicación. El primer punto de conexión de comprobación de estado encontrado que sea una coincidencia exacta con la dirección URL de la solicitud ejecuta y cortocircuita el resto de la canalización de middleware. Cuando se produce el cortocircuito, no se ejecuta ningún middleware que siga la comprobación de estado coincidente.

## <a name="health-check-options"></a>Opciones de comprobación de estado

El elemento <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions> ofrece una oportunidad para personalizar el comportamiento de las comprobaciones de estado:

* [Filtrado de las comprobaciones de estado](#filter-health-checks)
* [Personalización del código de estado HTTP](#customize-the-http-status-code)
* [Supresión de los encabezados de caché](#suppress-cache-headers)
* [Personalización del resultado](#customize-output)

### <a name="filter-health-checks"></a>Filtrado de las comprobaciones de estado

De forma predeterminada, el Middleware de comprobaciones de estado ejecuta todas las comprobaciones de estado registradas. Para ejecutar un subconjunto de comprobaciones de estado, proporcione una función que devuelva un valor booleano para la opción <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate>. En el ejemplo siguiente, la etiqueta (`bar_tag`) de la comprobación de estado `Bar` la filtra, en la instrucción condicional de la función, donde `true` solo se devuelve si la propiedad <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckRegistration.Tags> de la comprobación de estado coincide con `foo_tag` o `baz_tag`:

```csharp
using System.Threading.Tasks;
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using Microsoft.Extensions.Diagnostics.HealthChecks;

public void ConfigureServices(IServiceCollection services)
{
    services.AddHealthChecks()
        .AddCheck("Foo", () =>
            HealthCheckResult.Healthy("Foo is OK!"), tags: new[] { "foo_tag" })
        .AddCheck("Bar", () =>
            HealthCheckResult.Unhealthy("Bar is unhealthy!"), tags: new[] { "bar_tag" })
        .AddCheck("Baz", () =>
            HealthCheckResult.Healthy("Baz is OK!"), tags: new[] { "baz_tag" });
}

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseHealthChecks("/health", new HealthCheckOptions()
    {
        // Filter out the 'Bar' health check. Only Foo and Baz execute.
        Predicate = (check) => check.Tags.Contains("foo_tag") ||
            check.Tags.Contains("baz_tag")
    });
}
```

### <a name="customize-the-http-status-code"></a>Personalización del código de estado HTTP

Use <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResultStatusCodes> para personalizar la asignación del estado de mantenimiento de los códigos de estado HTTP. Las siguientes asignaciones de <xref:Microsoft.AspNetCore.Http.StatusCodes> son los valores predeterminados que el middleware utiliza. Cambie los valores de código de estado para satisfacer sus necesidades.

```csharp
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using Microsoft.Extensions.Diagnostics.HealthChecks;

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseHealthChecks("/health", new HealthCheckOptions()
    {
        // The following StatusCodes are the default assignments for
        // the HealthStatus properties.
        ResultStatusCodes =
        {
            [HealthStatus.Healthy] = StatusCodes.Status200OK,
            [HealthStatus.Degraded] = StatusCodes.Status200OK,
            [HealthStatus.Unhealthy] = StatusCodes.Status503ServiceUnavailable
        }
    });
}
```

### <a name="suppress-cache-headers"></a>Supresión de los encabezados de caché

<xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.AllowCachingResponses> controla si el Middleware de comprobaciones de estado agrega encabezados HTTP a una respuesta de sondeo para evitar el almacenamiento en caché de respuesta. Si el valor es `false` (valor predeterminado), el middleware establece o invalida los encabezados `Cache-Control`, `Expires` y `Pragma` para evitar el almacenamiento en caché de respuesta. Si el valor es `true`, el middleware no modifica los encabezados de caché de la respuesta.

```csharp
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using Microsoft.Extensions.Diagnostics.HealthChecks;

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseHealthChecks("/health", new HealthCheckOptions()
    {
        // The default value is false.
        AllowCachingResponses = false
    });
}
```

### <a name="customize-output"></a>Personalización del resultado

La opción <xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.ResponseWriter> obtiene o establece un delegado que se usa para escribir la respuesta. El delegado predeterminado escribe una respuesta de texto no cifrado mínima con el valor de cadena [HealthReport.Status](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthReport.Status).

```csharp
using Microsoft.AspNetCore.Diagnostics.HealthChecks;
using Microsoft.Extensions.Diagnostics.HealthChecks;

public void Configure(IApplicationBuilder app, IHostingEnvironment env)
{
    app.UseHealthChecks("/health", new HealthCheckOptions()
    {
        // WriteResponse is a delegate used to write the response.
        ResponseWriter = WriteResponse
    });
}

private static Task WriteResponse(HttpContext httpContext,
    HealthReport result)
{
    httpContext.Response.ContentType = "application/json";

    var json = new JObject(
        new JProperty("status", result.Status.ToString()),
        new JProperty("results", new JObject(result.Entries.Select(pair =>
            new JProperty(pair.Key, new JObject(
                new JProperty("status", pair.Value.Status.ToString()),
                new JProperty("description", pair.Value.Description),
                new JProperty("data", new JObject(pair.Value.Data.Select(
                    p => new JProperty(p.Key, p.Value))))))))));
    return httpContext.Response.WriteAsync(
        json.ToString(Formatting.Indented));
}
```

## <a name="database-probe"></a>Sondeo de bases de datos

Una comprobación de estado puede especificar que una consulta de base de datos se ejecute como una prueba booleana para indicar si esta responde con normalidad.

En la aplicación de ejemplo se usa [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks), una biblioteca de comprobaciones de estado para las aplicaciones ASP.NET Core, que permite realizar una comprobación de estado en una base de datos de SQL Server. `AspNetCore.Diagnostics.HealthChecks` ejecuta una consulta `SELECT 1` en la base de datos para confirmar que la conexión a la base de datos es correcta.

> [!WARNING]
> Al comprobar la conexión de una base de datos a una consulta, elija una consulta que se devuelva rápidamente. El enfoque de la consulta plantea el riesgo de sobrecargar la base de datos y degradar el rendimiento. En la mayoría de los casos, no es necesario ejecutar una consulta de prueba. Simplemente, realizar una conexión correcta a la base de datos es suficiente. Si resulta necesario ejecutar una consulta, elija una consulta SELECT sencilla, como `SELECT 1`.

Incluya una referencia de paquete a [AspNetCore.HealthChecks.SqlServer](https://www.nuget.org/packages/AspNetCore.HealthChecks.SqlServer/).

Proporcione una cadena de conexión a base de datos válida en el archivo *appsettings.json* de la aplicación de ejemplo. La aplicación usa una base de datos de SQL Server denominada `HealthCheckSample`:

[!code-json[](health-checks/samples/2.x/HealthChecksSample/appsettings.json?highlight=3)]

Registre los servicios de comprobación de estado con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> de `Startup.ConfigureServices`. La aplicación de ejemplo llama al método `AddSqlServer` con la cadena de conexión de la base de datos (*DbHealthStartup.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/DbHealthStartup.cs?name=snippet_ConfigureServices)]

Llame al Middleware de comprobaciones de estado en la canalización de procesamiento de la aplicación en `Startup.Configure`:

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/DbHealthStartup.cs?name=snippet_Configure)]

Para ejecutar el escenario de sondeo de base de datos mediante la aplicación de ejemplo, ejecute el comando siguiente desde la carpeta del proyecto en un shell de comandos:

```console
dotnet run --scenario db
```

> [!NOTE]
> [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) es un puerto de [BeatPulse](https://github.com/xabaril/beatpulse) y Microsoft no lo mantiene ni lo admite.

## <a name="entity-framework-core-dbcontext-probe"></a>Sondeo de DbContext de Entity Framework Core

La comprobación `DbContext` confirma que la aplicación puede comunicarse con la base de datos configurada para un elemento `DbContext` de EF Core. La comprobación `DbContext` se admite en las aplicaciones que:

* Usan [Entity Framework (EF) Core](/ef/core/).
* Incluyen una referencia de paquete a [Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore](https://www.nuget.org/packages/Microsoft.Extensions.Diagnostics.HealthChecks.EntityFrameworkCore/).

`AddDbContextCheck<TContext>` registra una comprobación de estado para un elemento `DbContext`. El elemento `DbContext` se proporciona como `TContext` en el método. Hay disponible una sobrecarga para configurar el estado de error, las etiquetas y una consulta de prueba personalizada.

De manera predeterminada:

* `DbContextHealthCheck` llama al método `CanConnectAsync` de EF Core. Se puede personalizar qué operación se ejecuta al comprobar el estado con sobrecargas del método `AddDbContextCheck`.
* El nombre de la comprobación de estado es el nombre del tipo `TContext`.

En la aplicación de ejemplo, `AppDbContext` se proporciona para `AddDbContextCheck` y se registra como un servicio en `Startup.ConfigureServices`.

*DbContextHealthStartup.cs*:

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/DbContextHealthStartup.cs?name=snippet_ConfigureServices)]

En la aplicación de ejemplo, `UseHealthChecks` agrega el Middleware de comprobaciones de estado en `Startup.Configure`.

*DbContextHealthStartup.cs*:

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/DbContextHealthStartup.cs?name=snippet_Configure)]

Para ejecutar el escenario de sondeo de `DbContext` mediante la aplicación de ejemplo, confirme que la base de datos que la cadena de conexión especifica no exista en la instancia de SQL Server. Si la base de datos existe, elimínela.

Ejecute el comando siguiente desde la carpeta del proyecto en un shell de comandos:

```console
dotnet run --scenario dbcontext
```

Cuando la aplicación ya se esté ejecutando, compruebe el estado de mantenimiento mediante una solicitud al punto de conexión `/health` en un explorador. La base de datos y `AppDbContext` no existen, por lo que la aplicación proporciona la respuesta siguiente:

```
Unhealthy
```

Active la aplicación de ejemplo para crear la base de datos. Realice una solicitud a `/createdatabase`. La aplicación responde:

```
Creating the database...
Done!
Navigate to /health to see the health status.
```

Realice una solicitud al punto de conexión `/health`. La base de datos y el contexto existen, por lo que la aplicación responde:

```
Healthy
```

Active la aplicación de ejemplo para eliminar la base de datos. Realice una solicitud a `/deletedatabase`. La aplicación responde:

```
Deleting the database...
Done!
Navigate to /health to see the health status.
```

Realice una solicitud al punto de conexión `/health`. La aplicación proporciona una respuesta incorrecta:

```
Unhealthy
```

## <a name="separate-readiness-and-liveness-probes"></a>Sondeos de preparación y ejecución independientes

En algunos escenarios de hospedaje, se usa un par de comprobaciones de estado que distinguen los dos estados de la aplicación:

* La aplicación está funcionando, pero aún no está preparada para recibir solicitudes. Este estado es la *preparación* de la aplicación.
* La aplicación está funcionando y responde a las solicitudes. Este estado es la *ejecución* de la aplicación.

Normalmente, la comprobación de preparación realiza un conjunto más amplio de comprobaciones, que requieren mucho tiempo, para determinar si están disponibles todos los subsistemas y los recursos de la aplicación. Una comprobación de ejecución simplemente realiza una comprobación rápida para determinar si la aplicación está disponible para procesar las solicitudes. Después de que la aplicación pase la comprobación de preparación, no es necesario sobrecargar aún más la aplicación con el conjunto amplio de comprobaciones de preparación; en las comprobaciones siguientes solo se requiere comprobar la ejecución.

La aplicación de ejemplo contiene una comprobación de estado para notificar la finalización de la tarea de inicio de ejecución prolongada en un [servicio hospedado](xref:fundamentals/host/hosted-services). El elemento `StartupHostedServiceHealthCheck` expone una propiedad, `StartupTaskCompleted`, que el servicio hospedado puede establecer en `true` al terminar su tarea de ejecución prolongada (*StartupHostedServiceHealthCheck.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/StartupHostedServiceHealthCheck.cs?name=snippet1&highlight=7-11)]

Un [servicio hospedado](xref:fundamentals/host/hosted-services) (*Services/StartupHostedService*) se encarga de iniciar la tarea en segundo plano de larga ejecución. Al finalizar la tarea, `StartupHostedServiceHealthCheck.StartupTaskCompleted` se establece en `true`:

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/Services/StartupHostedService.cs?name=snippet1&highlight=18-20)]

La comprobación de estado se registra con <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> en `Startup.ConfigureServices` junto con el servicio hospedado. Dado que el servicio hospedado debe establecer la propiedad en la comprobación de estado, esta también se registra en el contenedor de servicios (*LivenessProbeStartup.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_ConfigureServices)]

Llame al Middleware de comprobaciones de estado en la canalización de procesamiento de la aplicación en `Startup.Configure`. En la aplicación de ejemplo, se crean puntos de conexión de la comprobación de estado en `/health/ready` para la comprobación de preparación y en `/health/live` para la comprobación de ejecución. La comprobación de preparación filtra las comprobaciones de estado con la etiqueta `ready`. La comprobación de ejecución filtra el elemento `StartupHostedServiceHealthCheck` y devuelve `false` en [HealthCheckOptions.Predicate](xref:Microsoft.AspNetCore.Diagnostics.HealthChecks.HealthCheckOptions.Predicate). Para obtener más información, consulte [Filtrado de las comprobaciones de estado](#filter-health-checks):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_Configure)]

Para ejecutar el escenario de configuración de la preparación/ejecución mediante la aplicación de ejemplo, ejecute el comando siguiente desde la carpeta del proyecto en un shell de comandos:

```console
dotnet run --scenario liveness
```

En un explorador, visite `/health/ready` varias veces hasta que hayan pasado 15 segundos. La comprobación de estado notifica un estado *Incorrecto* durante los primeros 15 segundos. Pasados 15 segundos, el punto de conexión notifica un estado *Correcto*, lo que indica que el servicio hospedado ya ha finalizado la tarea de ejecución prolongada.

En este ejemplo también se crea un publicador de la comprobación de estado (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> implementación) que ejecuta la primera comprobación de preparación con un retraso de dos segundos. Para obtener más información, consulte la sección [Publicador de la comprobación de estado](#health-check-publisher).

### <a name="kubernetes-example"></a>Ejemplo de Kubernetes

Utilizar comprobaciones de preparación y ejecución independientes es útil en un entorno como [Kubernetes](https://kubernetes.io/). En Kubernetes, es posible que una aplicación deba realizar un trabajo de inicio que requiera mucho tiempo antes de aceptar solicitudes, como una prueba de la disponibilidad de la base de datos subyacente. El hecho de utilizar comprobaciones independientes permite que el orquestador distinga si la aplicación está funcionando, pero aún no esté preparada, o si la aplicación no se ha podido iniciar. Para obtener más información sobre los sondeos de preparación y ejecución en Kubernetes, consulte [Configuración de sondeos de preparación y ejecución](https://kubernetes.io/docs/tasks/configure-pod-container/configure-liveness-readiness-probes/) en la documentación de Kubernetes.

En el ejemplo siguiente, se muestra una configuración de sondeo de preparación de Kubernetes:

```
spec:
  template:
  spec:
    readinessProbe:
      # an http probe
      httpGet:
        path: /health/ready
        port: 80
      # length of time to wait for a pod to initialize
      # after pod startup, before applying health checking
      initialDelaySeconds: 30
      timeoutSeconds: 1
    ports:
      - containerPort: 80
```

## <a name="metric-based-probe-with-a-custom-response-writer"></a>Sondeo basado en métrica con un escritor de respuesta personalizada

La aplicación de ejemplo muestra una comprobación de estado de memoria con un escritor de respuesta personalizada.

`MemoryHealthCheck` notifica un estado degradado si la aplicación usa más de un umbral de memoria determinado (1 GB en la aplicación de ejemplo). El elemento <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckResult> incluye información del recolector de elementos no utilizados (GC) de la aplicación (*MemoryHealthCheck.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/MemoryHealthCheck.cs?name=snippet1)]

Registre los servicios de comprobación de estado con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> de `Startup.ConfigureServices`. En lugar de pasar la comprobación de estado a <xref:Microsoft.Extensions.DependencyInjection.HealthChecksBuilderAddCheckExtensions.AddCheck*> para habilitarla, `MemoryHealthCheck` se registra como servicio. Todos los servicios registrados de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> están disponibles para los servicios de comprobación de estado y middleware. Se recomienda registrar los servicios de comprobación de estado como los servicios de Singleton.

*CustomWriterStartup.cs*:

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_ConfigureServices&highlight=4)]

Llame al Middleware de comprobaciones de estado en la canalización de procesamiento de la aplicación en `Startup.Configure`. Se proporciona un delegado de `WriteResponse` a la propiedad `ResponseWriter` para generar una respuesta JSON personalizada al ejecutarse la comprobación de estado:

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_Configure&highlight=6)]

El método `WriteResponse` da a `CompositeHealthCheckResult` el formato de objeto JSON y suspende el resultado de JSON para la respuesta de comprobación de estado:

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/CustomWriterStartup.cs?name=snippet_WriteResponse)]

Para ejecutar el sondeo basado en métrica con el resultado de un escritor de respuesta personalizada mediante la aplicación de ejemplo, ejecute el comando siguiente desde la carpeta del proyecto en un shell de comandos:

```console
dotnet run --scenario writer
```

> [!NOTE]
> [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) incluye escenarios de comprobación de estado basados en métrica, como comprobaciones de almacenamiento del disco y de ejecución del máximo valor.
>
> [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) es un puerto de [BeatPulse](https://github.com/xabaril/beatpulse) y Microsoft no lo mantiene ni lo admite.

## <a name="filter-by-port"></a>Filtrado por puerto

Una llamada a <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> con un puerto restringe las solicitudes de comprobación de estado al puerto especificado. Esto se usa normalmente en un entorno de contenedor para exponer un puerto para los servicios de supervisión.

La aplicación de ejemplo configura el puerto con el [proveedor de configuración de variable de entorno](xref:fundamentals/configuration/index#environment-variables-configuration-provider). El puerto se establece en el archivo *launchSettings.json* y se pasa al proveedor de configuración a través de una variable de entorno. También debe configurar el servidor para que escuche las solicitudes en el puerto de administración.

Para utilizar la aplicación de ejemplo para que muestre la configuración del puerto de administración, cree el archivo *launchSettings.json* en una carpeta *Propiedades*.

El siguiente archivo *launchSettings.json* no se incluye en los archivos de proyecto de la aplicación de ejemplo y debe crearse manualmente.

*Properties/launchSettings.json*:

```json
{
  "profiles": {
    "SampleApp": {
      "commandName": "Project",
      "commandLineArgs": "",
      "launchBrowser": true,
      "environmentVariables": {
        "ASPNETCORE_ENVIRONMENT": "Development",
        "ASPNETCORE_URLS": "http://localhost:5000/;http://localhost:5001/",
        "ASPNETCORE_MANAGEMENTPORT": "5001"
      },
      "applicationUrl": "http://localhost:5000/"
    }
  }
}
```

Registre los servicios de comprobación de estado con <xref:Microsoft.Extensions.DependencyInjection.HealthCheckServiceCollectionExtensions.AddHealthChecks*> de `Startup.ConfigureServices`. La llamada a <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*> especifica el puerto de administración (*ManagementPortStartup.cs*):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/ManagementPortStartup.cs?name=snippet1&highlight=12,18)]

> [!NOTE]
> Para evitar la creación del archivo *launchSettings.json* en la aplicación de ejemplo, configure las direcciones URL y el puerto de administración explícitamente en código. En *Program.cs*, donde se crea <xref:Microsoft.AspNetCore.Hosting.WebHostBuilder>, agregue una llamada a <xref:Microsoft.AspNetCore.Hosting.HostingAbstractionsWebHostBuilderExtensions.UseUrls*> y proporcione el punto de conexión de respuesta normal de la aplicación y el punto de conexión del puerto de administración. En *ManagementPortStartup.cs*, donde se llama a <xref:Microsoft.AspNetCore.Builder.HealthCheckApplicationBuilderExtensions.UseHealthChecks*>, especifique explícitamente el puerto de administración.
>
> *Program.cs*:
>
> ```csharp
> return new WebHostBuilder()
>     .UseConfiguration(config)
>     .UseUrls("http://localhost:5000/;http://localhost:5001/")
>     .ConfigureLogging(builder =>
>     {
>         builder.SetMinimumLevel(LogLevel.Trace);
>         builder.AddConfiguration(config);
>         builder.AddConsole();
>     })
>     .UseKestrel()
>     .UseStartup(startupType)
>     .Build();
> ```
>
> *ManagementPortStartup.cs*:
>
> ```csharp
> app.UseHealthChecks("/health", port: 5001);
> ```

Para ejecutar el escenario de configuración del puerto de administración mediante la aplicación de ejemplo, ejecute el comando siguiente desde la carpeta del proyecto en un shell de comandos:

```console
dotnet run --scenario port
```

## <a name="distribute-a-health-check-library"></a>Distribución de una biblioteca de comprobación de estado

Para distribuir una comprobación de estado como una biblioteca, haga lo siguiente:

1. Escriba una comprobación de estado que implemente la interfaz de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheck> como una clase independiente. La clase puede depender de la [inserción de dependencias (DI)](xref:fundamentals/dependency-injection), de la activación del tipo y de las [opciones denominadas](xref:fundamentals/configuration/options) para acceder a los datos de configuración.

   ```csharp
   using System;
   using System.Threading;
   using System.Threading.Tasks;
   using Microsoft.Extensions.Diagnostics.HealthChecks;

   namespace SampleApp
   {
       public class ExampleHealthCheck : IHealthCheck
       {
           private readonly string _data1;
           private readonly int? _data2;

           public ExampleHealthCheck(string data1, int? data2)
           {
               _data1 = data1 ?? throw new ArgumentNullException(nameof(data1));
               _data2 = data2 ?? throw new ArgumentNullException(nameof(data2));
           }

           public async Task<HealthCheckResult> CheckHealthAsync(
               HealthCheckContext context, CancellationToken cancellationToken)
           {
               try
               {
                   // Health check logic
                   //
                   // data1 and data2 are used in the method to
                   // run the probe's health check logic.

                   // Assume that it's possible for this health check
                   // to throw an AccessViolationException.

                   return HealthCheckResult.Healthy();
               }
               catch (AccessViolationException ex)
               {
                   return new HealthCheckResult(
                       context.Registration.FailureStatus,
                       description: "An access violation occurred during the check.",
                       exception: ex,
                       data: null);
               }
           }
       }
   }
   ```

1. Escriba un método de extensión con los parámetros a los que la aplicación de uso llama en su método `Startup.Configure`. En el ejemplo siguiente, suponga que existe la siguiente firma del método de comprobación de estado:

   ```csharp
   ExampleHealthCheck(string, string, int )
   ```

   La firma anterior indica que la `ExampleHealthCheck` requiere datos adicionales para procesar la lógica de sondeo de la comprobación de estado. Los datos se proporcionan al delegado que se usa para crear la instancia de la comprobación de estado cuando la comprobación de estado se registra con un método de extensión. En el ejemplo siguiente, el autor de llamada especifica los siguientes elementos opcionales:

   * nombre de la comprobación de estado (`name`). En el caso de `null`, se utiliza `example_health_check`.
   * punto de datos de cadena para la comprobación de estado (`data1`).
   * punto de datos enteros para la comprobación de estado (`data2`). En el caso de `null`, se utiliza `1`.
   * estado de error (<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus>). De manera predeterminada, es `null`. Si `null`, [HealthStatus.Unhealthy](xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthStatus) se notifica para un estado de error.
   * etiquetas (`IEnumerable<string>`).

   ```csharp
   using System.Collections.Generic;
   using Microsoft.Extensions.Diagnostics.HealthChecks;

   public static class ExampleHealthCheckBuilderExtensions
   {
       const string NAME = "example_health_check";

       public static IHealthChecksBuilder AddExampleHealthCheck(
           this IHealthChecksBuilder builder,
           string name = default,
           string data1,
           int data2 = 1,
           HealthStatus? failureStatus = default,
           IEnumerable<string> tags = default)
       {
           return builder.Add(new HealthCheckRegistration(
               name ?? NAME,
               sp => new ExampleHealthCheck(data1, data2),
               failureStatus,
               tags));
       }
   }
   ```

## <a name="health-check-publisher"></a>Publicador de la comprobación de estado

Cuando un elemento <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> se agrega al contenedor de servicios, el sistema de comprobación de estado ejecuta periódicamente las comprobaciones de estado y llama a `PublishAsync` con el resultado. Esto es útil en un escenario de sistema de seguimiento de estado basado en inserción en el que se espera que cada proceso llame periódicamente al sistema de seguimiento con el fin de determinar el estado.

La interfaz de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> tiene un único método:

```csharp
Task PublishAsync(HealthReport report, CancellationToken cancellationToken);
```

<xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions> le permiten establecer:

* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay> &ndash; El retraso inicial aplicado tras iniciarse la aplicación antes de ejecutar instancias de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>. El retraso se aplica una vez durante el inicio y no se aplica a las iteraciones posteriores. El valor predeterminado es cinco segundos.
* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period> &ndash; El período de ejecución de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>. El valor predeterminado es 30 segundos.
* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate> &ndash; Si <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Predicate> es `null` (valor predeterminado), el servicio de publicador de la comprobación de estado ejecuta todas las comprobaciones de estado registradas. Para ejecutar un subconjunto de comprobaciones de estado, proporcione una función que filtre el conjunto de comprobaciones. El predicado se evalúa cada período.
* <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Timeout> &ndash; El tiempo de expiración para ejecutar las comprobaciones de estado para todas las instancias de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>. Use <xref:System.Threading.Timeout.InfiniteTimeSpan> para ejecutar sin tiempo de expiración. El valor predeterminado es 30 segundos.

::: moniker range="= aspnetcore-2.2"

> [!WARNING]
> En la versión de ASP.NET Core 2.2, el establecimiento de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Period> no se asigna por medio de la implementación de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>; establece el valor de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherOptions.Delay>. Este problema se corregirá en ASP.NET Core 3.0. Para obtener más información, consulte [HealthCheckPublisherOptions.Period establece el valor de .Delay](https://github.com/aspnet/Extensions/issues/1041).

::: moniker-end

En la aplicación de ejemplo, `ReadinessPublisher` es una implementación de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>. El estado de comprobación de estado se registra en `Entries` y para cada comprobación:

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/ReadinessPublisher.cs?name=snippet_ReadinessPublisher&highlight=20,22-23)]

En el ejemplo `LivenessProbeStartup` de la aplicación de ejemplo, la comprobación de preparación `StartupHostedService` tiene un retraso de inicio de dos segundos y ejecuta la comprobación cada 30 segundos. Para activar la implementación de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher>, el ejemplo registra `ReadinessPublisher` como servicio singleton en el contenedor de [inserción de dependencias (DI)](xref:fundamentals/dependency-injection):

[!code-csharp[](health-checks/samples/2.x/HealthChecksSample/LivenessProbeStartup.cs?name=snippet_ConfigureServices&highlight=12-17,28)]

::: moniker range="= aspnetcore-2.2"

> [!NOTE]
> La siguiente solución alternativa permite la adición de una instancia de <xref:Microsoft.Extensions.Diagnostics.HealthChecks.IHealthCheckPublisher> al contenedor del servicio cuando uno o más servicios hospedados ya se han agregado a la aplicación. Esta solución alternativa no se requerirá con el lanzamiento de ASP.NET Core 3.0. Para obtener más información, consulte: https://github.com/aspnet/Extensions/issues/639.
>
> ```csharp
> private const string HealthCheckServiceAssembly =
>     "Microsoft.Extensions.Diagnostics.HealthChecks.HealthCheckPublisherHostedService";
>
> services.TryAddEnumerable(
>     ServiceDescriptor.Singleton(typeof(IHostedService),
>         typeof(HealthCheckPublisherOptions).Assembly
>             .GetType(HealthCheckServiceAssembly)));
> ```

::: moniker-end

> [!NOTE]
> [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) incluye editores para varios sistemas, como [Application Insights](/azure/application-insights/app-insights-overview).
>
> [AspNetCore.Diagnostics.HealthChecks](https://github.com/Xabaril/AspNetCore.Diagnostics.HealthChecks) es un puerto de [BeatPulse](https://github.com/xabaril/beatpulse) y Microsoft no lo mantiene ni lo admite.
