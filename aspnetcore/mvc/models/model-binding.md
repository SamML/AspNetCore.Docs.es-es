---
title: Enlace de modelos en ASP.NET Core
author: tdykstra
description: Obtenga información sobre cómo funciona el enlace de modelos en ASP.NET Core y cómo personalizar su comportamiento.
ms.assetid: 0be164aa-1d72-4192-bd6b-192c9c301164
ms.author: tdykstra
ms.date: 05/31/2019
uid: mvc/models/model-binding
ms.openlocfilehash: 7d62ccecdacbd34a38a1fd8c58979a9b09cf86e8
ms.sourcegitcommit: e7e04a45195d4e0527af6f7cf1807defb56dc3c3
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/06/2019
ms.locfileid: "66750198"
---
# <a name="model-binding-in-aspnet-core"></a>Enlace de modelos en ASP.NET Core

En este artículo se explica qué es el enlace de modelos, cómo funciona y cómo personalizar su comportamiento.

[Vea o descargue el código de ejemplo](https://github.com/aspnet/AspNetCore.Docs/tree/master/aspnetcore/mvc/models/model-binding/samples) ([cómo descargarlo](xref:index#how-to-download-a-sample)).

## <a name="what-is-model-binding"></a>Qué es el enlace de modelos

Los controladores y Razor Pages trabajan con datos procedentes de solicitudes HTTP. Por ejemplo, los datos de ruta pueden proporcionar una clave de registro y los campos de formulario publicados pueden proporcionar valores para las propiedades del modelo. La escritura de código para recuperar cada uno de estos valores y convertirlos de cadenas a tipos de .NET sería tediosa y propensa a errores. El enlace de modelos automatiza este proceso. El sistema de enlace de modelos:

* Recupera datos de diversos orígenes, como datos de ruta, campos de formulario y cadenas de consulta.
* Proporciona los datos a los controladores y Razor Pages en parámetros de método y propiedades públicas.
* Convierte datos de cadena en tipos de .NET.
* Actualiza las propiedades de tipos complejos.

## <a name="example"></a>Ejemplo

Imagine que tiene el siguiente método de acción:

[!code-csharp[](model-binding/samples/2.x/Controllers/PetsController.cs?name=snippet_DogsOnly)]

Y que la aplicación recibe una solicitud con esta dirección URL:

```
http://contoso.com/api/pets/2?DogsOnly=true
```

El enlace de modelos realiza los pasos siguientes después de que el sistema de enrutamiento selecciona el método de acción:

* Busca el primer parámetro de `GetByID`, un entero denominado `id`.
* Examina los orígenes disponibles en la solicitud HTTP y busca `id` = "2" en los datos de ruta.
* Convierte la cadena "2" en el entero 2.
* Busca el siguiente parámetro de `GetByID`, un valor booleano denominado `dogsOnly`.
* Examina los orígenes y busca "DogsOnly=true" en la cadena de consulta. La coincidencia de nombres no distingue mayúsculas de minúsculas.
* Convierte la cadena "true" en un valor booleano `true`.

Después, el marco llama al método `GetById` y pasa 2 para el parámetro `id` y `true` para el parámetro `dogsOnly`.

En el ejemplo anterior, los destinos de enlace de modelos son parámetros de método que son tipos simples. Los destinos también pueden ser las propiedades de un tipo complejo. Después de que cada propiedad se haya enlazado correctamente, se produce la [validación de modelos](xref:mvc/models/validation) para esa propiedad. El registro de qué datos se han enlazado al modelo y los errores de enlace o validación se almacenan en [ControllerBase.ModelState](xref:Microsoft.AspNetCore.Mvc.ControllerBase.ModelState) o [PageModel.ModelState](xref:Microsoft.AspNetCore.Mvc.ControllerBase.ModelState). Para averiguar si este proceso se ha realizado correctamente, la aplicación comprueba la marca [ModelState.IsValid](xref:Microsoft.AspNetCore.Mvc.ModelBinding.ModelStateDictionary.IsValid).

## <a name="targets"></a>Destinos

El enlace de modelos intenta encontrar valores para los tipos de destinos siguientes:

* Parámetros del método de acción de controlador al que se enruta una solicitud.
* Parámetros del método de controlador de Razor Pages al que se enruta una solicitud. 
* Propiedades públicas de un controlador o una clase `PageModel`, si se especifican mediante atributos.

### <a name="bindproperty-attribute"></a>Atributo [BindProperty]

Se puede aplicar a una propiedad pública de un controlador o una clase `PageModel` para hacer que el enlace de modelos tenga esa propiedad como destino:

[!code-csharp[](model-binding/samples/2.x/Pages/Instructors/Edit.cshtml.cs?name=snippet_BindProperty&highlight=7-8)]

### <a name="bindpropertiesattribute"></a>Atributo [BindProperties]

Disponible en ASP.NET 2.1 Core y versiones posteriores.  Se pueden aplicar a un controlador o una clase `PageModel` para indicar al enlace de modelos que seleccione como destino todas las propiedades públicas de la clase:

[!code-csharp[](model-binding/samples/2.x/Pages/Instructors/Create.cshtml.cs?name=snippet_BindProperties&highlight=1-2)]

### <a name="model-binding-for-http-get-requests"></a>Enlace de modelos para solicitudes HTTP GET

De forma predeterminada, las propiedades no se enlazan para las solicitudes HTTP GET. Normalmente, todo lo que necesita para una solicitud GET es un parámetro de id. de registro. El id. de registro se usa para buscar el elemento en la base de datos. Por tanto, no es necesario enlazar una propiedad que contiene una instancia del modelo. En escenarios donde quiera propiedades enlazadas a datos de las solicitudes GET, establezca la propiedad `SupportsGet` en `true`:

[!code-csharp[](model-binding/samples/2.x/Pages/Instructors/Index.cshtml.cs?name=snippet_SupportsGet)]

## <a name="sources"></a>Orígenes

De forma predeterminada, el enlace de modelos obtiene datos en forma de pares clave-valor de los siguientes orígenes de una solicitud HTTP:

1. Campos de formulario 
1. El cuerpo de la solicitud (para [controladores que tienen el atributo [ApiController]](xref:web-api/index#binding-source-parameter-inference)).
1. Datos de ruta
1. Parámetros de cadena de consulta
1. Archivos cargados 

Para cada parámetro o propiedad de destino, se examinan los orígenes en el orden indicado en esta lista. Hay algunas excepciones:

* Los datos de ruta y los valores de cadena de consulta solo se usan para tipos simples.
* Los archivos cargados solo se enlazan a tipos de destino que implementan `IFormFile` o `IEnumerable<IFormFile>`.

Si el comportamiento predeterminado no proporciona los resultados adecuados, puede usar uno de los atributos siguientes para especificar el origen que se va a usar para cualquier destino dado. 

* [[FromQuery]](xref:Microsoft.AspNetCore.Mvc.FromQueryAttribute): obtiene valores de la cadena de consulta. 
* [[FromRoute]](xref:Microsoft.AspNetCore.Mvc.FromRouteAttribute): obtiene valores de los datos de ruta.
* [[FromForm]](xref:Microsoft.AspNetCore.Mvc.FromFormAttribute): obtiene valores de los campos de formulario publicados.
* [[FromBody]](xref:Microsoft.AspNetCore.Mvc.FromBodyAttribute): obtiene valores del cuerpo de la solicitud.
* [[FromHeader]](xref:Microsoft.AspNetCore.Mvc.FromHeaderAttribute): obtiene valores de encabezados HTTP.

Estos atributos:

* Se agregan de forma individual a las propiedades del modelo (no a la clase de modelo), como en el ejemplo siguiente:

  [!code-csharp[](model-binding/samples/2.x/Models/Instructor.cs?name=snippet_FromQuery&highlight=5-6)]

* Opcionalmente, acepte un valor de nombre de modelo en el constructor. Esta opción se proporciona en caso de que el nombre de la propiedad no coincida con el valor de la solicitud. Por ejemplo, es posible que el valor de la solicitud sea un encabezado con un guion en el nombre, como en el ejemplo siguiente:

  [!code-csharp[](model-binding/samples/2.x/Pages/Instructors/Index.cshtml.cs?name=snippet_FromHeader)]

### <a name="frombody-attribute"></a>Atributo [FromBody]

Los datos del cuerpo de la solicitud se analizan mediante formateadores de entrada específicos para el tipo de contenido de la solicitud. Los formateadores de entrada se explican [más adelante en este artículo](#input-formatters).

No aplique `[FromBody]` a más de un parámetro por método de acción. El runtime de ASP.NET Core delega la responsabilidad de leer la secuencia de solicitudes al formateador de entrada. Una vez que se ha leído la secuencia de solicitudes, deja de estar disponible para una nueva lectura con el fin de enlazar otros parámetros `[FromBody]`.

### <a name="additional-sources"></a>Orígenes adicionales

Los *proveedores de valores* proporcionan datos de origen al sistema de enlace de modelos. Puede escribir y registrar proveedores de valores personalizados que obtienen datos de otros orígenes para el enlace de modelos. Por ejemplo, es posible que le interesen datos de cookies o del estado de sesión. Para obtener datos desde un origen nuevo:

* Cree una clase que implemente `IValueProvider`.
* Cree una clase que implemente `IValueProviderFactory`.
* Registre la clase de generador en `Startup.ConfigureServices`.

En la aplicación de ejemplo se incluye un [proveedor de valores](https://github.com/aspnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/models/model-binding/samples/2.x/CookieValueProvider.cs) y un [generador](https://github.com/aspnet/AspNetCore.Docs/blob/master/aspnetcore/mvc/models/model-binding/samples/2.x/CookieValueProviderFactory.cs) que obtiene valores de cookies. Este es el código de registro de `Startup.ConfigureServices`:

[!code-csharp[](model-binding/samples/2.x/Startup.cs?name=snippet_ValueProvider&highlight=3)]

En el código mostrado, el proveedor de valor personalizado se coloca después de todos los proveedores de valor integrados.  Para que sea el primero en la lista, llame a `Insert(0, new CookieValueProviderFactory())` en lugar de a `Add`.

## <a name="no-source-for-a-model-property"></a>No hay origen para una propiedad de modelo

De forma predeterminada, si no se encuentra ningún valor para una propiedad de modelo no se crea un error de estado del modelo. La propiedad se establece en NULL o en un valor predeterminado:

* Los tipos simples que aceptan valores NULL se establecen en `null`.
* Los tipos de valor que no aceptan valores NULL se establecen en `default(T)`. Por ejemplo, un parámetro `int id` se establece en 0.
* Para los tipos complejos, el enlace de modelos crea una instancia mediante el constructor predeterminado, sin establecer propiedades.
* Las matrices se establecen en `Array.Empty<T>()`, salvo las matrices `byte[]`, que se establecen en `null`.

Si el estado del modelo se debe invalidar cuando no se encuentra nada en los campos de formulario para una propiedad de modelo, use el [atributo [BindRequired]](#bindrequired-attribute).

Tenga en cuenta que este comportamiento de `[BindRequired]` se aplica al enlace de modelos desde datos de formulario publicados, no a los datos JSON o XML del cuerpo de una solicitud. Los datos del cuerpo de la solicitud se controlan mediante [formateadores de entrada](#input-formatters).

## <a name="type-conversion-errors"></a>Errores de la conversión de tipos

Si se encuentra un origen pero no se puede convertir al tipo de destino, el estado del modelo se marca como no válido. El parámetro o la propiedad de destino se establece en NULL o en un valor predeterminado, como se ha indicado en la sección anterior.

En un controlador de API que tenga el atributo `[ApiController]`, el estado de modelo no válido genera una respuesta HTTP 400 automática.

En una página de Razor, se vuelve a mostrar la página con un mensaje de error:

[!code-csharp[](model-binding/samples/2.x/Pages/Instructors/Create.cshtml.cs?name=snippet_HandleMBError&highlight=3-6)]

La validación del lado cliente detecta la mayoría de los datos incorrectos que en otros casos se enviarían a un formulario de Razor Pages. Esta validación dificulta que se pueda desencadenar el código resaltado anterior. En la aplicación de ejemplo se incluye un botón **Submit with Invalid Date** (Enviar con fecha no válida) que agrega datos incorrectos al campo **Hire Date** (Fecha de contratación) y envía el formulario. Este botón muestra cómo funciona el código para volver a mostrar la página cuando se producen errores de conversión de datos.

Cuando el código anterior vuelve a mostrar la página, no se muestra la entrada no válida en el campo de formulario. El motivo es que la propiedad de modelo se ha establecido en NULL o en un valor predeterminado. La entrada no válida sí aparece en un mensaje de error. Pero si quiere volver a mostrar los datos incorrectos en el campo de formulario, considere la posibilidad de convertir la propiedad de modelo en una cadena y realizar la conversión de datos de forma manual.

Se recomienda la misma estrategia si no quiere que los errores de conversión de tipo generen errores de estado de modelo. En ese caso, convierta la propiedad de modelo en una cadena.

## <a name="simple-types"></a>Tipos simples

Los tipos simples a los que el enlazador de modelos puede convertir las cadenas de origen incluyen los siguientes:

* [Boolean](xref:System.ComponentModel.BooleanConverter)
* [Byte](xref:System.ComponentModel.ByteConverter), [SByte](xref:System.ComponentModel.SByteConverter)
* [Char](xref:System.ComponentModel.CharConverter)
* [DateTime](xref:System.ComponentModel.DateTimeConverter)
* [DateTimeOffset](xref:System.ComponentModel.DateTimeOffsetConverter)
* [Decimal](xref:System.ComponentModel.DecimalConverter)
* [Double](xref:System.ComponentModel.DoubleConverter)
* [Enum](xref:System.ComponentModel.EnumConverter)
* [Guid](xref:System.ComponentModel.GuidConverter)
* [Int16](xref:System.ComponentModel.Int16Converter), [Int32](xref:System.ComponentModel.Int32Converter), [Int64](xref:System.ComponentModel.Int64Converter)
* [Single](xref:System.ComponentModel.SingleConverter)
* [TimeSpan](xref:System.ComponentModel.TimeSpanConverter)
* [UInt16](xref:System.ComponentModel.UInt16Converter), [UInt32](xref:System.ComponentModel.UInt32Converter), [UInt64](xref:System.ComponentModel.UInt64Converter)
* [Uri](xref:System.UriTypeConverter)
* [Versión](xref:System.ComponentModel.VersionConverter)

## <a name="complex-types"></a>Tipos complejos

Un tipo complejo debe tener un constructor público predeterminado y propiedades grabables públicas para enlazar. Cuando se produce el enlace de modelos, se crea una instancia de la clase con el constructor predeterminado público. 

Para cada propiedad del tipo complejo, el enlace de modelos busca entre los orígenes el patrón de nombre *prefijo.nombre_de_propiedad*. Si no se encuentra nada, solo busca *nombre_de_propiedad* sin el prefijo.

Para el enlace a un parámetro, el prefijo es el nombre del parámetro. Para el enlace a una propiedad pública `PageModel`, el prefijo es el nombre de la propiedad pública. Algunos atributos tienen una propiedad `Prefix` que permite invalidar el uso predeterminado del nombre de parámetro o propiedad.

Por ejemplo, imagine que el tipo complejo es la clase `Instructor` siguiente:

  ```csharp
  public class Instructor
  {
      public int ID { get; set; }
      public string LastName { get; set; }
      public string FirstName { get; set; }
  }
  ```

### <a name="prefix--parameter-name"></a>Prefijo = nombre del parámetro

Si el modelo que se va a enlazar es un parámetro denominado `instructorToUpdate`:

```csharp
public IActionResult OnPost(int? id, Instructor instructorToUpdate)
```

El enlace de modelos se inicia mediante el examen de los orígenes para la clave `instructorToUpdate.ID`. Si no se encuentra, busca `ID` sin un prefijo.

### <a name="prefix--property-name"></a>Prefijo = nombre de propiedad

Si el modelo que se va a enlazar es una propiedad denominada `Instructor` del controlador o la clase `PageModel`:

```csharp
[BindProperty]
public Instructor Instructor { get; set; }
```

El enlace de modelos se inicia mediante el examen de los orígenes para la clave `Instructor.ID`. Si no se encuentra, busca `ID` sin un prefijo.

### <a name="custom-prefix"></a>Prefijo personalizado

Si el modelo que se va a enlazar es un parámetro denominado `instructorToUpdate` y un atributo `Bind`, especifica `Instructor` como el prefijo:

```csharp
public IActionResult OnPost(
    int? id, [Bind(Prefix = "Instructor")] Instructor instructorToUpdate)
```

El enlace de modelos se inicia mediante el examen de los orígenes para la clave `Instructor.ID`. Si no se encuentra, busca `ID` sin un prefijo.

### <a name="attributes-for-complex-type-targets"></a>Atributos para destinos de tipo complejo

Existen varios atributos integrados para controlar el enlace de modelos de tipos complejos:

* `[BindRequired]`
* `[BindNever]`
* `[Bind]`

> [!NOTE]
> Estos atributos afectan al enlace de modelos cuando el origen de los valores son datos de formulario publicados. No afectan a los formateadores de entrada, que procesan los cuerpos de la solicitud JSON y XML publicados. Los formateadores de entrada se explican [más adelante en este artículo](#input-formatters).
>
> Vea también la explicación del atributo `[Required]` en [Validación de modelos](xref:mvc/models/validation#required-attribute).

### <a name="bindrequired-attribute"></a>Atributo [BindRequired]

Solo se puede aplicar a propiedades del modelo, no a parámetros de método. Hace que el enlace de modelos agregue un error de estado de modelo si no se puede realizar el enlace para la propiedad de un modelo. Por ejemplo:

[!code-csharp[](model-binding/samples/2.x/Models/InstructorWithCollection.cs?name=snippet_BindRequired&highlight=8-9)]

### <a name="bindnever-attribute"></a>Atributo [BindNever]

Solo se puede aplicar a propiedades del modelo, no a parámetros de método. Impide que el enlace de modelos establezca la propiedad de un modelo. Por ejemplo:

[!code-csharp[](model-binding/samples/2.x/Models/InstructorWithDictionary.cs?name=snippet_BindNever&highlight=3-4)]

### <a name="bind-attribute"></a>Atributo [Bind]

Se puede aplicar a una clase o un parámetro de método. Especifica qué propiedades de un modelo se deben incluir en el enlace de modelos.

En el ejemplo siguiente, solo se enlazan las propiedades especificadas del modelo `Instructor` cuando se llama a cualquier método de acción o controlador:

```csharp
[Bind("LastName,FirstMidName,HireDate")]
public class Instructor
```

En el ejemplo siguiente, solo se enlazan las propiedades especificadas del modelo `Instructor` cuando se llama al método `OnPost`:

```csharp
[HttpPost]
public IActionResult OnPost([Bind("LastName,FirstMidName,HireDate")] Instructor instructor)
```

El atributo `[Bind]` se puede usar para protegerse de la publicación excesiva en escenarios de *creación*. No funciona bien en escenarios de edición porque las propiedades excluidas se establecen en NULL o en un valor predeterminado en lugar de mantenerse sin cambios. Para defenderse de la publicación excesiva, se recomiendan modelos de vista en lugar del atributo `[Bind]`. Para más información, vea [Nota de seguridad sobre la publicación excesiva](xref:data/ef-mvc/crud#security-note-about-overposting).

## <a name="collections"></a>Colecciones

Para los destinos que son colecciones de tipos simples, el enlace de modelos busca coincidencias con *nombre_de_parámetro* o *nombre_de_propiedad*. Si no se encuentra ninguna coincidencia, busca uno de los formatos admitidos sin el prefijo. Por ejemplo:

* Imagine que el parámetro que se va a enlazar es una matriz llamada `selectedCourses`:

  ```csharp
  public IActionResult OnPost(int? id, int[] selectedCourses)
  ```

* Los datos de cadena de consulta o formulario pueden estar en uno de los formatos siguientes:
   
  ```
  selectedCourses=1050&selectedCourses=2000 
  ```

  ```
  selectedCourses[0]=1050&selectedCourses[1]=2000
  ```

  ```
  [0]=1050&[1]=2000
  ```

  ```
  selectedCourses[a]=1050&selectedCourses[b]=2000&selectedCourses.index=a&selectedCourses.index=b
  ```

  ```
  [a]=1050&[b]=2000&index=a&index=b
  ```

* El formato siguiente solo se admite en datos de formulario:

  ```
  selectedCourses[]=1050&selectedCourses[]=2000
  ```

* Para todos los formatos de ejemplo anteriores, el enlace de modelos pasa una matriz de dos elementos al parámetro `selectedCourses`:

  * selectedCourses[0]=1050
  * selectedCourses[1]=2000

  Los formatos de datos que usan números de subíndice (... [0] ... [1] ...) deben asegurarse de que se numeran de forma secuencial a partir de cero. Si hay algún hueco en la numeración de los subíndices, se omiten todos los elementos que aparecen después del hueco. Por ejemplo, si los subíndices son 0 y 2 en lugar de 0 y 1, se omite el segundo elemento.

## <a name="dictionaries"></a>Diccionarios

Para los destinos `Dictionary`, el enlace de modelos busca coincidencias con *nombre_de_parámetro* o *nombre_de_propiedad*. Si no se encuentra ninguna coincidencia, busca uno de los formatos admitidos sin el prefijo. Por ejemplo:

* Imagine que el parámetro de destino es un elemento `Dictionary<string, string>` denominado `selectedCourses`:

  ```csharp
  public IActionResult OnPost(int? id, Dictionary<int, string> selectedCourses)
  ```

* Los datos de cadena de consulta o de formulario publicados pueden ser similares a uno de los ejemplos siguientes:

  ```
  selectedCourses[1050]=Chemistry&selectedCourses[2000]=Economics
  ```

  ```
  [1050]=Chemistry&selectedCourses[2000]=Economics
  ```

  ```
  selectedCourses[0].Key=1050&selectedCourses[0].Value=Chemistry&
  selectedCourses[1].Key=2000&selectedCourses[1].Value=Economics
  ```

  ```
  [0].Key=1050&[0].Value=Chemistry&[1].Key=2000&[1].Value=Economics
  ```

* Para todos los formatos de ejemplo anteriores, el enlace de modelos pasa un diccionario de dos elementos al parámetro `selectedCourses`:

  * selectedCourses["1050"]="Chemistry"
  * selectedCourses["2000"]="Economics"

## <a name="special-data-types"></a>Tipos de datos especiales

Hay algunos tipos de datos especiales que el enlace de modelos puede controlar.

### <a name="iformfile-and-iformfilecollection"></a>IFormFile e IFormFileCollection

Un archivo cargado incluido en la solicitud HTTP.  También se admite `IEnumerable<IFormFile>` para varios archivos.

### <a name="cancellationtoken"></a>CancellationToken

se usa para cancelar la actividad en controladores asincrónicos.

### <a name="formcollection"></a>FormCollection

Se usa para recuperar todos los valores de los datos de formulario publicados.

## <a name="input-formatters"></a>Formateadores de entrada

Los datos del cuerpo de la solicitud pueden estar en XML, JSON u otro formato. Para analizar estos datos, el enlace de modelos usa un *formateador de entrada* configurado para controlar un tipo de contenido determinado. De forma predeterminada, en ASP.NET Core se incluyen formateadores de entrada basados en JSON para el control de los datos JSON. Puede agregar otros formateadores para otros tipos de contenido.

ASP.NET Core selecciona los formateadores de entrada en función del atributo [Consumes](xref:Microsoft.AspNetCore.Mvc.ConsumesAttribute). Si no hay ningún atributo, usa el [encabezado Content-Type](https://www.w3.org/Protocols/rfc1341/4_Content-Type.html).

Para usar los formateadores de entrada XML integrados:

* Instale el paquete NuGet `Microsoft.AspNetCore.Mvc.Formatters.Xml`.

* En `Startup.ConfigureServices`, llame a <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcCoreBuilderExtensions.AddXmlSerializerFormatters*> o a <xref:Microsoft.Extensions.DependencyInjection.MvcXmlMvcCoreBuilderExtensions.AddXmlDataContractSerializerFormatters*>.

  [!code-csharp[](model-binding/samples/2.x/Startup.cs?name=snippet_ValueProvider&highlight=9)]

* Aplique el atributo `Consumes` a las clases de controlador o los métodos de acción que deben esperar XML en el cuerpo de la solicitud.

  ```csharp
  [HttpPost]
  [Consumes("application/xml")]
  public ActionResult<Pet> Create(Pet pet)
  ```

  Para más información, vea [Introducción de la serialización XML](https://docs.microsoft.com/en-us/dotnet/standard/serialization/introducing-xml-serialization).

## <a name="exclude-specified-types-from-model-binding"></a>Exclusión de tipos especificados del enlace de modelos

El comportamiento del enlace de modelos y los sistemas de validación se controla mediante [ModelMetadata](/dotnet/api/microsoft.aspnetcore.mvc.modelbinding.modelmetadata). Puede personalizar `ModelMetadata` mediante la adición de un proveedor de detalles a [MvcOptions.ModelMetadataDetailsProviders](xref:Microsoft.AspNetCore.Mvc.MvcOptions.ModelMetadataDetailsProviders). Los proveedores de detalles integrados están disponibles para deshabilitar el enlace de modelos o la validación para tipos especificados.

Para deshabilitar el enlace de modelos en todos los modelos de un tipo especificado, agregue una instancia de <xref:Microsoft.AspNetCore.Mvc.ModelBinding.Metadata.ExcludeBindingMetadataProvider> en `Startup.ConfigureServices`. Por ejemplo, para deshabilitar el enlace de modelos en todos los modelos del tipo `System.Version`:

[!code-csharp[](model-binding/samples/2.x/Startup.cs?name=snippet_ValueProvider&highlight=4-5)]

Para deshabilitar la validación en propiedades de un tipo especificado, agregue una instancia de <xref:Microsoft.AspNetCore.Mvc.ModelBinding.SuppressChildValidationMetadataProvider> en `Startup.ConfigureServices`. Por ejemplo, para deshabilitar la validación en las propiedades de tipo `System.Guid`:

[!code-csharp[](model-binding/samples/2.x/Startup.cs?name=snippet_ValueProvider&highlight=6-7)]

## <a name="custom-model-binders"></a>Enlazadores de modelos personalizados

Puede ampliar el enlace de modelos si escribe un enlazador de modelos personalizado y usa el atributo `[ModelBinder]` para seleccionarlo para un destino concreto. Más información sobre el [enlace de modelos personalizados](xref:mvc/advanced/custom-model-binding).

## <a name="manual-model-binding"></a>Enlace de modelos manual

El enlace de modelos se puede invocar de forma manual mediante el método <xref:Microsoft.AspNetCore.Mvc.ControllerBase.TryUpdateModelAsync*>. El método se define en las clases `ControllerBase` y `PageModel`. Las sobrecargas de método permiten especificar el prefijo y el proveedor de valores que se van a usar. El método devuelve `false` si se produce un error en el enlace de modelos. Por ejemplo:

[!code-csharp[](model-binding/samples/2.x/Pages/InstructorsWithCollection/Create.cshtml.cs?name=snippet_TryUpdate&highlight=1-4)]

## <a name="fromservices-attribute"></a>Atributo [FromServices]

El nombre de este atributo sigue el patrón de los atributos de enlace de modelos que especifican un origen de datos. Pero no se trata de enlazar datos desde un proveedor de valores. Obtiene una instancia de un tipo desde el contenedor de [inserción de dependencias](xref:fundamentals/dependency-injection). Su objetivo es proporcionar una alternativa a la inserción de constructores cuando se necesita un servicio solo si se llama a un método concreto.

## <a name="additional-resources"></a>Recursos adicionales

* <xref:mvc/models/validation>
* <xref:mvc/advanced/custom-model-binding>
