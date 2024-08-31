Originally posted [here](https://gennari.com/posts/htmx-with-razor-components/).

I love the simplicity of [Htmx](https://htmx.org/) and the dev experience of [Razor Components](https://learn.microsoft.com/en-us/aspnet/core/blazor/components/?view=aspnetcore-8.0), so today I'm going to show you how to use both of them together. We're going to be using the standard .NET 8 `webapp` template, [Razor Pages](https://learn.microsoft.com/en-us/aspnet/core/razor-pages/?view=aspnetcore-8.0&tabs=visual-studio) and layouts for the main pages, and Razor Components for the Htmx interactivity. This gives us the best of both simplicity worlds, using Razor pages for our overall app navigation and logic, and the templating an in-line code of Razor Components for our UX components.

Let's start with a basic `webapp` templates:

```bash
dotnet new webapp -o HtmxRazorComponents
```

The first thing we're going to do is add two lines to the `Program.cs` file:

```csharp
builder.Services.AddRazorComponents(); // Registers services required for server-side rendering of Razor Components.

/* ... */

app.MapRazorComponents<HtmxRazorComponents.App>(); // Maps the page components defined in the specified App to the given assembly and renders the component specified by App when the route matches.
```

### Routing

Now we have Razor Component support, but you may notice `App` isn't defined. That's because there isn't a `App` class to direct routes to. For that, we'll need a new `App.razor` file. Because we want our Razor components. In this file is normally a full HTML layout, but because we're using Razor Components to return partial HTML within our Htmx calls, we'll remove all that and just leave a `Router` component. This will allow us to use the `@page` directive to route calls to our components.

```razor
@namespace HtmxRazorComponents // Avoid issues in your program.cs by declaring a namespace here
@using Microsoft.AspNetCore.Components.Routing

<Router AppAssembly="@typeof(App).Assembly">
    <Found Context="routeData">
        <RouteView RouteData="@routeData" />
    </Found>
</Router>
```

### Layout for Razor Pages

 Next we need to define the layout for our Razor Pages. Our Razor Pages will be the basis of our application, including all the navigation, so unifying the look-and-feel is important. We'll start by simplifying the `_Layout.cshtml` that comes int the `webapp` template. Let's replace it completely with a simple template:

```cshtml
<html>
<head>
    <meta charset="UTF-8">
    <title>@ViewData["Title"] - HtmxRazorComponents</title>
    <meta name="viewport" content="width =device-width, initial-scale=1.0">
    <script src="https://unpkg.com/htmx.org@1.9.6" integrity="sha384-FhXw7b6AlE/jyjlZH5iHa/tTe9EpJ1Y55RjcgPbjeWMskSxZt1v9qkxLJWNJaGni" crossorigin="anonymous"></script>
</head>
<body>
    @RenderBody()
</body>
</html>
```

Here we're just referencing the Htmx package and adding the @RenderBody() tag. Next, let's simplify `Index.cshtml` by replacing it with the following:

```cshtml
@page
@model IndexModel
@{
    ViewData["Title"] = "Weather";
}

<h1>Home</h1>
<div hx-get="/weather" hx-trigger="load" hx-swap="innerHTML">

</div>
```

As you can see, we've got our first hint at some Htmx goodness with our `hx-get`, `hx-trigger` and `hx-swap` attributes.

### Our First Component ☀️

Let's add the current weather to our home page. `Index.cshtml` is asking Htmx to swap the contents from an Ajax call to `/weather` into that `div`. I like to create a new folder for components, so I'll create `Components` as a top-level folder. From there, I'll create a new file called `Weather.razor`:

```razor
@page "/weather"

@code 
{
    double _sanFranciscoTemp, _newYorkTemp, _miamiTemp;

    protected override Task OnInitializedAsync()
    {
        var rand = new Random();
        _sanFranciscoTemp = rand.Next(400,800) / 10.0;
        _newYorkTemp = rand.Next(200,500) / 10.0;
        _miamiTemp = rand.Next(700,900) / 10.0;

        return base.OnInitializedAsync();
    }
}

<h2>Current Weather</h2>

<ul>
    <li>San Francisco: @_sanFranciscoTemp</li>
    <li>New York: @_newYorkTemp</li>
    <li>Miami: @_miamiTemp</li>
</ul>
```

Let's run it! And ... uh-oh, exception:

>  InvalidOperationException: Endpoint /weather (/weather) contains anti-forgery metadata, but a middleware was not found that supports anti-forgery.
Configure your application startup by adding app.UseAntiforgery() in the application startup code. If there are calls to app.UseRouting() and app.UseEndpoints(...), the call to app.UseAntiforgery() must go between them. Calls to app.UseAntiforgery() must be placed after calls to app.UseAuthentication() and app.UseAuthorization(). 
{.danger}

So we have two options, either add the global `app.UseAntiforgery();` after `app.UseRouting();`, or disable Antiforgery on our Razor components:

```csharp
app.UseRouting();
app.UseAntiforgery(); // Added

/* ... or ... */

app.MapRazorComponents<HtmxRazorComponents.App>()
    .DisableAntiforgery();
```

And just like that, we have our weather! You can see below that the browser is making a call to `/weather` and just returning the partial rendering from the Razor Component.
