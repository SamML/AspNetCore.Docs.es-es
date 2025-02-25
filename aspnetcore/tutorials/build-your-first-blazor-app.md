---
title: Creación de la primera aplicación Blazor
author: guardrex
description: Cree una aplicación Blazor paso a paso.
monikerRange: '>= aspnetcore-3.0'
ms.author: riande
ms.custom: mvc
ms.date: 06/12/2019
uid: tutorials/first-blazor-app
ms.openlocfilehash: df27dad17133f287b1c73dc308b4cc69426e0a63
ms.sourcegitcommit: 739a3d7ca4fd2908ea0984940eca589a96359482
ms.translationtype: HT
ms.contentlocale: es-ES
ms.lasthandoff: 06/13/2019
ms.locfileid: "67040710"
---
# <a name="build-your-first-blazor-app"></a>Creación de la primera aplicación Blazor

Por [Daniel Roth](https://github.com/danroth27) y [Luke Latham](https://github.com/guardrex)

En este tutorial se muestra cómo crear y modificar una aplicación de Blazor.

Siga las instrucciones del artículo <xref:blazor/get-started> para crear un proyecto de Blazor en este tutorial.

## <a name="build-components"></a>Creación de componentes

1. Vaya a cada una de las tres páginas de la aplicación en la carpeta *Pages*: Home (Inicio), Counter (Contador) y Fetch data (Recuperar datos). Estas páginas se implementan mediante los archivos de componente de Razor *Index.razor*, *Counter.razor* y *FetchData.razor*.

1. En la página Contador, seleccione el botón **Click me** para aumentar el contador sin una actualización de página. Aumentar un contador en una página web suele requerir la escritura de JavaScript, pero Blazor proporciona una mejor manera de usar C#.

1. Examine la implementación del componente Counter en el archivo *Counter.razor*.

   *Pages/Counter.razor*:

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Counter1.razor)]

   la interfaz de usuario del componente Counter se define mediante HTML. La lógica de la representación dinámica (por ejemplo, bucles, instrucciones condicionales, expresiones) se agrega mediante una sintaxis de C# insertada denominada [Razor](xref:mvc/views/razor). El marcado HTML y la lógica de representación de C# se convierten en una clase de componente en tiempo de compilación. El nombre de la clase de .NET generada coincide con el nombre del archivo.

   Los miembros de la clase de componente se definen en un bloque `@code`. En el bloque `@code`, se especifica el estado del componente (propiedades, campos) y los métodos para el tratamiento de eventos o para definir otra lógica del componente. Estos miembros se utilizan como parte de la lógica de representación del componente y para el tratamiento de eventos.

   Al seleccionarse el botón **Click me**:

   * Se llama al controlador `onclick` registrado del componente Counter (el método `IncrementCount`).
   * El componente Counter regenera su árbol de representación.
   * El nuevo árbol de representación se compara con el anterior.
   * Únicamente se aplican modificaciones en Document Object Model (DOM). Se actualiza el recuento mostrado.

1. Modifique la lógica de C# del componente Counter para hacer que el recuento se incremente en dos en lugar de uno.

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Counter2.razor?highlight=14)]

1. Recompile y ejecute la aplicación para ver los cambios. Seleccione el botón **Hacer clic aquí**. El contador se incrementa en dos.

## <a name="use-components"></a>Uso de componentes

Incluya un componente en otro componente mediante una sintaxis HTML.

1. Agregue el componente Counter al componente Index de la aplicación; para ello, agregue un elemento `<Counter />` al componente Index (*Index.razor*).

   Si usa Blazor para esta experiencia, hay un componente Survey Prompt (elemento `<SurveyPrompt>`) en el componente Index. Reemplace el elemento `<SurveyPrompt>` por el elemento `<Counter>`. Si usa una aplicación de servidor de Blazor para esta experiencia, agregue el elemento `<Counter>` al componente Index:

   *Pages/Index.razor*:

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Index1.razor?highlight=7)]

1. Recompile y ejecute la aplicación. El componente Index tiene su propio contador.

## <a name="component-parameters"></a>Parámetros del componente

Los componentes también pueden tener parámetros. Los parámetros del componente se definen mediante propiedades privadas en la clase de componentes decorada con `[Parameter]`. Use atributos para especificar argumentos para un componente en el marcado.

1. Actualice el código de C# `@code` del componente:

   * Agregue una propiedad `IncrementAmount` decorada con el atributo `[Parameter]`.
   * Cambie el método `IncrementCount` para usar `IncrementAmount` al aumentar el valor de `currentCount`.

   *Pages/Counter.razor*:

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Counter.razor?highlight=13,17)]

<!-- Add back when supported.
   > [!NOTE]
   > From Visual Studio, you can quickly add a component parameter by using the `para` snippet. Type `para` and press the `Tab` key twice.
-->

1. Especifique un parámetro `IncrementAmount` en el elemento `<Counter>` del componente Index mediante un atributo. Establezca el valor para incrementar el contador en diez.

   *Pages/Index.razor*:

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Index2.razor?highlight=7)]

1. Vuelva a cargar el componente Index. El contador se incrementa en diez cada vez que se selecciona el botón **Click me**. El contador del componente Counter sigue incrementándose en uno.

## <a name="route-to-components"></a>Enrutamiento a los componentes

La directiva `@page` en la parte superior del archivo *Counter.razor* especifica que el componente Counter es un punto de conexión de enrutamiento. El componente Counter controla las solicitudes enviadas a `/counter`. Sin la directiva `@page`, el componente no controla las solicitudes enrutadas, pero otros componentes aún pueden usar el componente.

## <a name="dependency-injection"></a>Inserción de dependencias

Los servicios registrados en el contenedor de servicios de la aplicación están disponibles para los componentes mediante una [inserción de dependencia (DI)](xref:fundamentals/dependency-injection). Inserte servicios en un componente mediante la directiva `@inject`.

Examine las directivas del componente FetchData.

Si trabaja con la aplicación de servidor de Blazor, el servicio `WeatherForecastService` se registra como [singleton](xref:fundamentals/dependency-injection#service-lifetimes), de modo que una instancia del servicio está disponible en toda la aplicación. La directiva `@inject` se usa para insertar la instancia del servicio `WeatherForecastService` en el componente.

*Pages/FetchData.razor*:

[!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/FetchData1.razor?highlight=3)]

El componente FetchData usa el servicio insertado, como `ForecastService`, para recuperar una matriz de objetos `WeatherForecast`:

[!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/FetchData2.razor?highlight=6)]

Si trabaja con la aplicación cliente de Blazor, se inserta `HttpClient` para obtener datos de previsión del tiempo del archivo *weather.json* de la carpeta *wwwroot/sample-data*:

*Pages/FetchData.razor*:

[!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/FetchData1_client.razor?highlight=7-8)]

Se usa un bucle [\@foreach](/dotnet/csharp/language-reference/keywords/foreach-in) para representar cada instancia de previsión como una fila de la tabla de datos meteorológicos:

[!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/FetchData3.razor?highlight=11-19)]


## <a name="build-a-todo-list"></a>Creación de una lista de tareas pendientes

Agregue un nuevo componente a la aplicación que implemente una simple lista de tareas pendientes.

1. Agregue un archivo vacío denominado *Todo.razor* a la aplicación en la carpeta *Pages*:

1. Proporcione el marcado inicial para el componente:

   ```cshtml
   @page "/todo"

   <h1>Todo</h1>
   ```

1. Agregue el componente Todo a la barra de navegación.

   El componente NavMenu (*Shared/NavMenu.razor*) se usa en el diseño de la aplicación. Los diseños son componentes que le permiten impedir la duplicación de contenido en la aplicación. Para obtener más información, vea <xref:blazor/layouts>.

   Agregue un elemento `<NavLink>` a la página Todo mediante la adición del siguiente marcado de elementos de lista debajo de los elementos de lista existentes en el archivo *Shared/NavMenu.razor*:

   ```cshtml
   <li class="nav-item px-3">
       <NavLink class="nav-link" href="todo">
           <span class="oi oi-list-rich" aria-hidden="true"></span> Todo
       </NavLink>
   </li>
   ```

1. Recompile y ejecute la aplicación. Visite la nueva página Todo para confirmar que el vínculo al componente Todo funcione.

1. Agregue un archivo *TodoItem.cs* a la raíz del proyecto para contener una clase que represente un elemento de la lista de tareas. Use el siguiente código de C# para la clase `TodoItem`:

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/TodoItem.cs)]

1. Vuelva al componente Todo (*Pages/Todo.razor*):

   * Agregue un campo a los elementos de tareas pendientes en un bloque `@code`. El componente Todo utiliza este campo para mantener el estado de la lista de tareas pendientes.
   * Agregue el marcado de la lista no ordenada y un bucle `foreach` para que cada elemento de la lista se represente en un elemento de la lista de tareas pendientes.

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo4.razor?highlight=5-10,12-14)]

1. Para agregar elementos de tareas pendientes a la lista, la aplicación requiere elementos de la interfaz de usuario. Agregue una entrada de texto y un botón debajo de la lista:

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo5.razor?highlight=12-13)]

1. Recompile y ejecute la aplicación. Cuando se selecciona el botón **Add todo** (Agregar tarea pendiente), no ocurre nada porque no hay ningún controlador de eventos conectado al botón.

1. Agregue un método `AddTodo` al componente Todo y regístrelo para hacer clic en los botones mediante el atributo `@onclick`:

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo6.razor?highlight=2,7-10)]

   El método `AddTodo` de C# se llama cuando se selecciona el botón.

1. Para obtener el título del nuevo elemento de tarea pendiente, agregue un campo de cadena `newTodo` y enlácelo al valor de la entrada de texto mediante el atributo `bind`:

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo7.razor?highlight=2)]

   ```cshtml
   <input placeholder="Something todo" @bind="@newTodo" />
   ```

1. Actualice el método `AddTodo` para agregar el `TodoItem` con el título especificado a la lista. Borre el valor de la entrada de texto mediante el establecimiento de `newTodo` en una cadena vacía:

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo8.razor?highlight=19-26)]

1. Recompile y ejecute la aplicación. Agregue algunos elementos de tareas pendientes a la lista de tareas pendientes para probar el nuevo código.

1. Se puede hacer que el texto de título de cada elemento de tarea pendiente sea editable y una casilla puede ayudar al usuario a realizar un seguimiento de los elementos completados. Agregue una entrada de casilla a cada elemento de tarea pendiente y enlace su valor a la propiedad `IsDone`. Cambie `@todo.Title` a un elemento `<input>` enlazado a `@todo.Title`:

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/ToDo9.razor?highlight=5-6)]

1. Para comprobar que estos valores están enlazados, actualice el encabezado `<h1>` para mostrar un recuento del número de elementos de la lista de tareas pendientes que no se han completado (`IsDone` es `false`).

   ```cshtml
   <h1>Todo (@todos.Count(todo => !todo.IsDone))</h1>
   ```

1. El componente Todo completado (*Pages/Todo.razor*):

   [!code-cshtml[](build-your-first-blazor-app/samples_snapshot/3.x/Todo.razor)]

1. Recompile y ejecute la aplicación. Agregue elementos de tarea pendiente para probar el nuevo código.

## <a name="publish-and-deploy-the-app"></a>Publicar e implementar la aplicación

Para publicar la aplicación, consulte <xref:host-and-deploy/blazor/index>.
