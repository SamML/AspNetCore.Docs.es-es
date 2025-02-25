---
title: Introducción a Swashbuckle y ASP.NET Core
author: zuckerthoben
description: Obtenga información sobre cómo agregar Swashbuckle a un proyecto de ASP.NET Core Web API para integrar la interfaz de usuario de Swagger.
ms.author: scaddie
ms.custom: mvc
ms.date: 04/04/2019
uid: tutorials/get-started-with-swashbuckle
ms.openlocfilehash: ebbdfa4cfeb3013cd961167439dd261f2e058a4b
ms.sourcegitcommit: dd9c73db7853d87b566eef136d2162f648a43b85
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 05/06/2019
ms.locfileid: "65087610"
---
# <a name="get-started-with-swashbuckle-and-aspnet-core"></a>Introducción a Swashbuckle y ASP.NET Core

Por [Shayne Boyer](https://twitter.com/spboyer) y [Scott Addie](https://twitter.com/Scott_Addie)

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/tutorials/web-api-help-pages-using-swagger/samples/) ([cómo descargarlo](xref:index#how-to-download-a-sample))

Hay tres componentes principales de Swashbuckle:

* [Swashbuckle.AspNetCore.Swagger](https://www.nuget.org/packages/Swashbuckle.AspNetCore.Swagger/): modelo de objetos de Swagger y middleware para exponer objetos `SwaggerDocument` como puntos de conexión JSON.

* [Swashbuckle.AspNetCore.SwaggerGen](https://www.nuget.org/packages/Swashbuckle.AspNetCore.SwaggerGen/): generador de Swagger que genera objetos `SwaggerDocument` directamente desde las rutas, controladores y modelos. Se suele combinar con el middleware de punto de conexión de Swagger para exponer automáticamente el JSON de Swagger.

* [Swashbuckle.AspNetCore.SwaggerUI](https://www.nuget.org/packages/Swashbuckle.AspNetCore.SwaggerUI/): versión insertada de la herramienta de interfaz de usuario de Swagger. Interpreta el JSON de Swagger para crear una experiencia enriquecida y personalizable para describir la funcionalidad de la API web. Incluye herramientas de ejecución de pruebas integradas para los métodos públicos.

## <a name="package-installation"></a>Instalación del paquete

Se puede agregar Swashbuckle con los métodos siguientes:

### <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

* En la ventana **Consola del Administrador de paquetes**:
  * Vaya a **Vista** > **Otras ventanas** > **Consola del Administrador de paquetes**.
  * Vaya al directorio en el que está *TodoApi.csproj*.
  * Ejecute el siguiente comando:

    ```powershell
    Install-Package Swashbuckle.AspNetCore
    ```

* En el cuadro de diálogo **Administrar paquetes NuGet**:
  * Haga clic con el botón derecho en el proyecto en el **Explorador de soluciones** > **Administrar paquetes NuGet**.
  * Establezca el **origen del paquete** en "nuget.org".
  * Escriba "Swashbuckle.AspNetCore" en el cuadro de búsqueda.
  * Seleccione el paquete "Swashbuckle.AspNetCore" en la pestaña **Examinar** y haga clic en **Instalar**.

### <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio para Mac](#tab/visual-studio-mac)

* Haga clic con el botón derecho en la carpeta *Paquetes* en **Panel de solución** > **Agregar paquetes...**
* Establezca el menú desplegable **Origen** de la ventana **Agregar paquetes** en "nuget.org".
* Escriba "Swashbuckle.AspNetCore" en el cuadro de búsqueda.
* Seleccione el paquete "Swashbuckle.AspNetCore" en el panel de resultados y haga clic en **Agregar paquete**.

### <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Ejecute el siguiente comando en el **terminal integrado**:

```console
dotnet add TodoApi.csproj package Swashbuckle.AspNetCore
```

### <a name="net-core-clitabnetcore-cli"></a>[CLI de .NET Core](#tab/netcore-cli)

Ejecute el siguiente comando:

```console
dotnet add TodoApi.csproj package Swashbuckle.AspNetCore
```

---

## <a name="add-and-configure-swagger-middleware"></a>Agregar y configurar el middleware de Swagger

Importe el siguiente espacio de nombres para que use la clase `Info`:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_InfoClassNamespace)]

Agregue el generador de Swagger a la colección de servicios en el método `Startup.ConfigureServices`:

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_ConfigureServices&highlight=8-11)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Startup2.cs?name=snippet_ConfigureServices&highlight=9-12)]

::: moniker-end

En el método `Startup.Configure`, habilite el middleware para servir el documento JSON generado y la interfaz de usuario de Swagger:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup2.cs?name=snippet_Configure&highlight=4,8-11)]

La llamada de método `UseSwaggerUI` anterior habilita el [middleware de archivos estáticos](xref:fundamentals/static-files). Si el destino es .NET Framework o .NET Core 1.x, agregue el paquete NuGet [Microsoft.AspNetCore.StaticFiles](https://www.nuget.org/packages/Microsoft.AspNetCore.StaticFiles/) al proyecto.

Inicie la aplicación y vaya a `http://localhost:<port>/swagger/v1/swagger.json`. El documento generado en el que se describen los puntos de conexión aparecerá según se muestra en la [especificación de Swagger (swagger.json)](xref:tutorials/web-api-help-pages-using-swagger#swagger-specification-swaggerjson).

La interfaz de usuario de Swagger se encuentra en `http://localhost:<port>/swagger`. Explore la API a través de la interfaz de usuario de Swagger e incorpórela a otros programas.

> [!TIP]
> Para servir la interfaz de usuario de Swagger en la raíz de la aplicación (`http://localhost:<port>/`), establezca la propiedad `RoutePrefix` en una cadena vacía:
>
> [!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup3.cs?name=snippet_UseSwaggerUI&highlight=4)]

Si usa directorios con IIS o un proxy inverso, establezca el punto de conexión de Swagger en una ruta de acceso relativa mediante el prefijo `./`. Por ejemplo: `./swagger/v1/swagger.json`. El uso de `/swagger/v1/swagger.json` indica a la aplicación que debe buscar el archivo JSON en la verdadera raíz de la dirección URL (junto con el prefijo de ruta, si se usa). Por ejemplo, use `http://localhost:<port>/<route_prefix>/swagger/v1/swagger.json` en lugar de `http://localhost:<port>/<virtual_directory>/<route_prefix>/swagger/v1/swagger.json`.

## <a name="customize-and-extend"></a>Personalizar y ampliar

Swagger proporciona opciones para documentar el modelo de objetos y personalizar la interfaz de usuario para que coincida con el tema.

### <a name="api-info-and-description"></a>Información y descripción de la API

La acción de configuración que se pasa al método `AddSwaggerGen` agrega información, como el autor, la licencia y la descripción:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup4.cs?name=snippet_AddSwaggerGen)]

La interfaz de usuario de Swagger muestra información de la versión:

![Interfaz de usuario de Swagger con información de la versión: descripción, autor y el vínculo See more (Ver más)](web-api-help-pages-using-swagger/_static/custom-info.png)

### <a name="xml-comments"></a>comentarios XML

Los comentarios XML se pueden habilitar con los métodos siguientes:

#### <a name="visual-studiotabvisual-studio"></a>[Visual Studio](#tab/visual-studio)

::: moniker range=">= aspnetcore-2.0"

* Haga clic con el botón derecho en el **Explorador de soluciones** y seleccione **Editar <nombre_de_proyecto>.csproj**.
* Agregue manualmente las líneas resaltadas al archivo *.csproj*:

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

* Haga clic con el botón derecho en el proyecto en el **Explorador de soluciones** y seleccione **Propiedades**.
* Active la casilla **Archivo de documentación XML** en la sección **Salida** de la pestaña **Compilar**.

::: moniker-end

#### <a name="visual-studio-for-mactabvisual-studio-mac"></a>[Visual Studio para Mac](#tab/visual-studio-mac)

::: moniker range=">= aspnetcore-2.0"

* Desde el *Panel de solución*, presione **Control** y haga clic en el nombre del proyecto. Vaya a **Herramientas** > **Editar archivo**.
* Agregue manualmente las líneas resaltadas al archivo *.csproj*:

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

* Abra el cuadro de diálogo **Opciones del proyecto** > **Compilar** > **Compilador**.
* Active la casilla **Generar documentación XML** en la sección **Opciones generales**.

::: moniker-end

#### <a name="visual-studio-codetabvisual-studio-code"></a>[Visual Studio Code](#tab/visual-studio-code)

Agregue manualmente las líneas resaltadas al archivo *.csproj*:

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

#### <a name="net-core-clitabnetcore-cli"></a>[CLI de .NET Core](#tab/netcore-cli)

Agregue manualmente las líneas resaltadas al archivo *.csproj*:

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=1-2,4)]

::: moniker-end

---

Puede habilitar los comentarios XML para proporcionar información de depuración para miembros y tipos públicos sin documentación. Los miembros y tipos que no estén documentados se indican por medio del mensaje de advertencia. Por ejemplo, el siguiente mensaje señala una infracción con el código de advertencia 1591:

```text
warning CS1591: Missing XML comment for publicly visible type or member 'TodoController.GetAll()'
```

Para eliminar las advertencias a nivel de proyecto, defina una lista delimitada por punto y coma de códigos de advertencia que se deban omitir en el archivo de proyecto. Al anexar los códigos de advertencia a `$(NoWarn);`, se aplican también los [valores predeterminados de C#](https://github.com/dotnet/sdk/blob/2eb6c546931b5bcb92cd3128b93932a980553ea1/src/Tasks/Microsoft.NET.Build.Tasks/targets/Microsoft.NET.Sdk.CSharp.props#L16).

::: moniker range=">= aspnetcore-2.0"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=3)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-xml[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/TodoApi.csproj?name=snippet_SuppressWarnings&highlight=3)]

::: moniker-end

Para suprimir las advertencias solo para miembros específicos, incluya el código en directivas de preprocesador de [advertencias #pragma](/dotnet/csharp/language-reference/preprocessor-directives/preprocessor-pragma-warning). Este enfoque es útil para el código que no debe exponerse mediante los documentos de API. En el ejemplo siguiente, se omite el código de advertencia CS1591 para toda la clase `Program`. La ejecución del código de advertencia se restaura al cerrar la definición de clase. Especifique varios códigos de advertencia con una lista delimitada por comas.

```csharp
namespace TodoApi
{
#pragma warning disable CS1591
    public class Program
    {
        public static void Main(string[] args) =>
            BuildWebHost(args).Run();

        public static IWebHost BuildWebHost(string[] args) =>
            WebHost.CreateDefaultBuilder(args)
                .UseStartup<Startup>()
                .Build();
    }
#pragma warning restore CS1591
}
```

Configure Swagger para usar el archivo XML que se generó con las instrucciones anteriores. En Linux o sistemas operativos que no sean Windows, las rutas de acceso y los nombres de archivo pueden distinguir entre mayúsculas y minúsculas. Por ejemplo, un archivo *TodoApi.XML* es válido en Windows, pero no en CentOS.

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Startup.cs?name=snippet_ConfigureServices&highlight=31-33)]

::: moniker-end

::: moniker range="= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup.cs?name=snippet_ConfigureServices&highlight=30-32)]

::: moniker-end

::: moniker range="<= aspnetcore-1.1"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup1x.cs?name=snippet_ConfigureServices&highlight=30-32)]

::: moniker-end

En el código anterior, [Reflection](/dotnet/csharp/programming-guide/concepts/reflection) se usa para crear un nombre de archivo XML que coincida con el proyecto de API web. La propiedad [AppContext.BaseDirectory](xref:System.AppContext.BaseDirectory*) sirve para crear una ruta de acceso al archivo XML. Algunas características de Swagger (por ejemplo, esquemas de parámetros de entrada o métodos HTTP y códigos de respuesta de los atributos respectivos) funcionan sin el uso de un archivo de documentación XML. Para la mayoría de las características, es decir, los resúmenes de los métodos y las descripciones de los parámetros y los códigos de respuesta, el uso de un archivo XML es obligatorio.

Agregar comentarios con la triple barra diagonal a una acción mejora la interfaz de usuario de Swagger en tanto agrega una descripción de la acción al encabezado de la sección. Agregue un elemento [\<summary>](/dotnet/csharp/programming-guide/xmldoc/summary) antes de la acción `Delete`:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Delete&highlight=1-3)]

La interfaz de usuario de Swagger muestra el texto interno del elemento `<summary>` del código anterior:

![Interfaz de usuario de Swagger en la que se muestra el comentario XML "Deletes a specific TodoItem" (Elimina una tarea pendiente específica) para el método DELETE](web-api-help-pages-using-swagger/_static/triple-slash-comments.png)

La interfaz de usuario se controla por medio del esquema JSON generado:

```json
"delete": {
    "tags": [
        "Todo"
    ],
    "summary": "Deletes a specific TodoItem.",
    "operationId": "ApiTodoByIdDelete",
    "consumes": [],
    "produces": [],
    "parameters": [
        {
            "name": "id",
            "in": "path",
            "description": "",
            "required": true,
            "type": "integer",
            "format": "int64"
        }
    ],
    "responses": {
        "200": {
            "description": "Success"
        }
    }
}
```

Agregue un elemento [\<remarks>](/dotnet/csharp/programming-guide/xmldoc/remarks) a la documentación del método de acción `Create`. Este elemento complementa la información especificada en el elemento `<summary>` y proporciona una interfaz de usuario de Swagger más sólida. El contenido del elemento `<remarks>` puede estar formado por texto, JSON o XML.

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=4-14)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=4-14)]

::: moniker-end

Fíjese en las mejoras de la interfaz de usuario con estos comentarios extra:

![Interfaz de usuario de Swagger con comentarios adicionales](web-api-help-pages-using-swagger/_static/xml-comments-extended.png)

### <a name="data-annotations"></a>Anotaciones de datos

Incorpore al modelo atributos que se encuentren en [System.ComponentModel.DataAnnotations](/dotnet/api/system.componentmodel.dataannotations) para controlar los componentes de la interfaz de usuario de Swagger.

Agregue el atributo `[Required]` a la propiedad `Name` de la clase `TodoItem`:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Models/TodoItem.cs?highlight=10)]

La presencia de este atributo cambia el comportamiento de la interfaz de usuario y modifica el esquema JSON subyacente:

```json
"definitions": {
    "TodoItem": {
        "required": [
            "name"
        ],
        "type": "object",
        "properties": {
            "id": {
                "format": "int64",
                "type": "integer"
            },
            "name": {
                "type": "string"
            },
            "isComplete": {
                "default": false,
                "type": "boolean"
            }
        }
    }
},
```

Agregue el atributo `[Produces("application/json")]` al controlador de API. Su propósito consiste en declarar que las acciones del controlador admiten un contenido de respuesta de tipo *application/json*:

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_TodoController&highlight=1)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_TodoController&highlight=1)]

::: moniker-end

En el menú desplegable **Response Content Type** (Tipo de contenido de respuesta) se selecciona este tipo de contenido como el valor predeterminado para las acciones GET del controlador:

![Interfaz de usuario de Swagger con el tipo de contenido de respuesta predeterminado](web-api-help-pages-using-swagger/_static/json-response-content-type.png)

A medida que aumenta el uso de anotaciones de datos en la API web, la interfaz de usuario y las páginas de ayuda de la API pasan a ser más descriptivas y útiles.

### <a name="describe-response-types"></a>Describir los tipos de respuesta

Lo que más preocupa a los desarrolladores que consumen una API web es lo que se devuelve; sobre todo, los tipos de respuesta y los códigos de error (si no son los habituales). Los tipos de respuesta y los códigos de error se indican en las anotaciones de datos y los comentarios XML.

La acción `Create` devuelve un código de estado HTTP 201 cuando se ha ejecutado correctamente. Cuando el cuerpo de solicitud enviado es null, se devuelve un código de estado HTTP 400. Sin la documentación correcta en la interfaz de usuario de Swagger, el consumidor no dispone de la información necesaria de estos resultados esperados. Corrija este problema agregando las líneas resaltadas en el siguiente ejemplo:

::: moniker range="<= aspnetcore-2.0"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=17,18,20,21)]

::: moniker-end

::: moniker range=">= aspnetcore-2.1"

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.1/TodoApi.Swashbuckle/Controllers/TodoController.cs?name=snippet_Create&highlight=17,18,20,21)]

::: moniker-end

Ahora, la interfaz de usuario de Swagger documenta de forma clara los códigos de respuesta HTTP esperados:

![Interfaz de usuario de Swagger, donde se muestra la descripción de la clase de respuesta POST "Returns the newly created item" (Devuelve el elemento recién creado) y "400 - If the item is null" (400 - Si el elemento es nulo) para el código de estado y el motivo en Response Messages (Mensajes de respuesta)](web-api-help-pages-using-swagger/_static/data-annotations-response-types.png)

::: moniker range=">= aspnetcore-2.2"

En ASP.NET Core 2.2 o versiones posteriores, las convenciones se pueden usar como alternativa a la representación explícita de acciones individuales con `[ProducesResponseType]`. Para obtener más información, vea <xref:web-api/advanced/conventions>.

::: moniker-end

### <a name="customize-the-ui"></a>Personalizar la interfaz de usuario

La interfaz de usuario es funcional y tiene un aspecto adecuado. Pero las páginas de documentación de la API deben ostentar su marca o tema. Para incluir la personalización de marca en los componentes de Swashbuckle, se deben agregar los recursos para servir archivos estáticos y generar la estructura de carpetas que hospedará estos archivos.

Si el destino es .NET Framework o .NET Core 1.x, agregue el paquete NuGet [Microsoft.AspNetCore.StaticFiles](https://www.nuget.org/packages/Microsoft.AspNetCore.StaticFiles) al proyecto:

```xml
<PackageReference Include="Microsoft.AspNetCore.StaticFiles" Version="2.0.0" />
```

El paquete NuGet anterior ya estará instalado si el destino es .NET Core 2.x y se usa el [metapaquete](xref:fundamentals/metapackage).

Habilitar middleware de archivos estáticos:

[!code-csharp[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/Startup.cs?name=snippet_Configure&highlight=3)]

Obtenga el contenido de la carpeta *dist* en el [repositorio de GitHub de la interfaz de usuario de Swagger](https://github.com/swagger-api/swagger-ui/tree/master/dist). Esta carpeta contiene los recursos necesarios para la página de interfaz de usuario de Swagger.

Cree una carpeta *wwwroot/swagger/ui* y copie en ella el contenido de la carpeta *dist*.

Cree un archivo *custom.css* en *wwwroot/swagger/ui* con el siguiente código CSS para personalizar el encabezado de página:

[!code-css[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/wwwroot/swagger/ui/custom.css)]

Cree una referencia a *custom.css* en el archivo *index.html* después de cualquier otro archivo CSS:

[!code-html[](../tutorials/web-api-help-pages-using-swagger/samples/2.0/TodoApi.Swashbuckle/wwwroot/swagger/ui/index.html?name=snippet_SwaggerUiCss&highlight=3)]

Vaya a la página *index.html* en `http://localhost:<port>/swagger/ui/index.html`. Escriba `http://localhost:<port>/swagger/v1/swagger.json` en el cuadro de texto del encabezado y haga clic en el botón **Explorar**. La página resultante tiene el siguiente aspecto:

![Interfaz de usuario de Swagger con el título de encabezado personalizado](web-api-help-pages-using-swagger/_static/custom-header.png)

Puede hacer muchas más cosas con la página. Vea todas las capacidades de los recursos de la interfaz de usuario en el [repositorio de GitHub de la interfaz de usuario de Swagger](https://github.com/swagger-api/swagger-ui).
