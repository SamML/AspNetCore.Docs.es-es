---
title: Agregar búsqueda a páginas de Razor de ASP.NET Core
author: rick-anderson
description: Muestra cómo agregar la búsqueda a páginas de Razor de ASP.NET Core
ms.author: riande
ms.date: 12/3/2018
uid: tutorials/razor-pages/search
ms.openlocfilehash: d0ce13e87c3f5e66008f308f6258403ea37b8847
ms.sourcegitcommit: 5b0eca8c21550f95de3bb21096bd4fd4d9098026
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 04/27/2019
ms.locfileid: "64884580"
---
# <a name="add-search-to-aspnet-core-razor-pages"></a>Agregar búsqueda a páginas de Razor de ASP.NET Core

Por [Rick Anderson](https://twitter.com/RickAndMSFT)

[!INCLUDE[](~/includes/rp/download.md)]

En las secciones siguientes, se ha agregado la función de buscar películas por *género* o *nombre*.

Agregue las siguientes propiedades resaltadas a *Pages/Movies/Index.cshtml.cs*:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_newProps&highlight=11-999)]

* `SearchString`: contiene el texto que los usuarios escriben en el cuadro de texto de búsqueda. El elemento `SearchString` está decorado con el atributo [`[BindProperty]`](/dotnet/api/microsoft.aspnetcore.mvc.bindpropertyattribute). `[BindProperty]` enlaza los valores del formulario y las cadenas de consulta con el mismo nombre que la propiedad. `(SupportsGet = true)` se necesita para el enlace de las solicitudes GET.
* `Genres`: contiene la lista de géneros. `Genres` permite al usuario seleccionar un género de la lista. `SelectList` requiere `using Microsoft.AspNetCore.Mvc.Rendering;`.
* `MovieGenre`: contiene el género concreto que selecciona el usuario (por ejemplo, "Western").
* `Genres` y `MovieGenre` se utilizan posteriormente en este tutorial.

[!INCLUDE[](~/includes/bind-get.md)]

Actualice el método `OnGetAsync` de la página de índice con el código siguiente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_1stSearch)]

La primera línea del método `OnGetAsync` crea una consulta [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) para seleccionar las películas:

```csharp
// using System.Linq;
var movies = from m in _context.Movie
             select m;
```

En este momento *solo* se define la consulta, **no** se ejecuta en la base de datos.

Si la propiedad `SearchString` no es NULL ni está vacía, la consulta de películas se modifica para filtrar según la cadena de búsqueda:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SearchNull)]

El código `s => s.Title.Contains()` es una [expresión lambda](/dotnet/csharp/programming-guide/statements-expressions-operators/lambda-expressions). Las lambdas se usan en consultas [LINQ](/dotnet/csharp/programming-guide/concepts/linq/) basadas en métodos como argumentos para métodos de operador de consulta estándar, como el método [Where](/dotnet/csharp/programming-guide/concepts/linq/query-syntax-and-method-syntax-in-linq) o `Contains` (usado en el código anterior). Las consultas LINQ no se ejecutan cuando se definen ni cuando se modifican mediante una llamada a un método (como `Where`, `Contains` u `OrderBy`). En su lugar, se aplaza la ejecución de la consulta. Esto significa que la evaluación de una expresión se aplaza hasta que su valor realizado se repite o se llama al método `ToListAsync`. Para más información, vea [Query Execution (Ejecución de consultas)](/dotnet/framework/data/adonet/ef/language-reference/query-execution).

**Nota:** El método [Contains](/dotnet/api/system.data.objects.dataclasses.entitycollection-1.contains) se ejecuta en la base de datos, no en el código de C#. La distinción entre mayúsculas y minúsculas en la consulta depende de la base de datos y la intercalación. En SQL Server, `Contains` se asigna a [SQL LIKE](/sql/t-sql/language-elements/like-transact-sql), que distingue entre mayúsculas y minúsculas. En SQLite, con la intercalación predeterminada, se distingue entre mayúsculas y minúsculas.

Vaya a la página de películas y anexe una cadena de consulta como `?searchString=Ghost` a la dirección URL (por ejemplo, `https://localhost:5001/Movies?searchString=Ghost`). Se muestran las películas filtradas.

![Vista de índice](search/_static/ghost.png)

Si se agrega la siguiente plantilla de ruta a la página de índice, la cadena de búsqueda se puede pasar como un segmento de dirección URL (por ejemplo, `https://localhost:5001/Movies/Ghost`).

```cshtml
@page "{searchString?}"
```

La restricción de ruta anterior permite buscar el título como datos de ruta (un segmento de dirección URL) en lugar de como un valor de cadena de consulta.  El elemento `?` de `"{searchString?}"` significa que se trata de un parámetro de ruta opcional.

![Vista de índice con la palabra Ghost (Fantasma) agregada a la dirección URL y una lista de películas devuelta con dos películas, Ghostbusters y Ghostbusters 2](search/_static/g2.png)

El entorno de ejecución de ASP.NET Core usa el [enlace de modelos](xref:mvc/models/model-binding) para establecer el valor de la propiedad `SearchString` de la cadena de consulta (`?searchString=Ghost`) o de los datos de ruta (`https://localhost:5001/Movies/Ghost`). El enlace de modelos no hace distinción entre mayúsculas y minúsculas.

Sin embargo, no se puede esperar que los usuarios modifiquen la dirección URL para buscar una película. En este paso, se agrega la interfaz de usuario para filtrar las películas. Si ha agregado la restricción de ruta `"{searchString?}"`, quítela.

Abra el archivo *Pages/Movies/Index.cshtml* y agregue el marcado `<form>` resaltado en el siguiente código:

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index2.cshtml?highlight=14-19&range=1-22)]

La etiqueta HTML `<form>` usa los siguientes [Asistentes de etiquetas](xref:mvc/views/tag-helpers/intro):

* [Asistente de etiquetas de formulario](xref:mvc/views/working-with-forms#the-form-tag-helper). Cuando se envía el formulario, la cadena de filtro se envía a la página *Pages/Movies/Index* a través de la cadena de consulta.
* [Asistente de etiquetas de entrada](xref:mvc/views/working-with-forms#the-input-tag-helper)

Guarde los cambios y pruebe el filtro.

![Vista de índice con la palabra Ghost (Fantasma) escrita en el cuadro de texto del filtro de título](search/_static/filter.png)

## <a name="search-by-genre"></a>Búsqueda por género

Actualice el método `OnGetAsync` con el código siguiente:

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SearchGenre)]

El código siguiente es una consulta LINQ que recupera todos los géneros de la base de datos.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_LINQ)]

La `SelectList` de géneros se crea mediante la proyección de los distintos géneros.

[!code-csharp[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/Index.cshtml.cs?name=snippet_SelectList)]

### <a name="add-search-by-genre-to-the-razor-page"></a>Agregar búsqueda por género a la página de Razor

Actualice *Index.cshtml* como se indica a continuación:

[!code-cshtml[](razor-pages-start/sample/RazorPagesMovie22/Pages/Movies/IndexFormGenreNoRating.cshtml?highlight=16-18&range=1-26)]

Pruebe la aplicación al buscar por género, título de la película y ambos.

## <a name="additional-resources"></a>Recursos adicionales

* [Versión en YouTube de este tutorial](https://youtu.be/4B6pHtdyo08)

> [!div class="step-by-step"]
> [Anterior: Actualización de páginas](xref:tutorials/razor-pages/da1)
> [Siguiente: ](xref:tutorials/razor-pages/new-field)Adición de un nuevo campo
