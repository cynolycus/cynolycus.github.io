---
title: "Using View Components and Partial Views in ASP.NET Core"
category: asp-net-core
order: 2
final: true
date: 2018-12-01T00:02:00-5:00
last_modified_at: 2019-08-01
excerpt: Let's explore the different ways to invoke a View Component from a Razor view or Controller and test what we've learned in our introduction!
description: Let's explore the different ways to invoke a View Component from a Razor view or Controller and build out a working example.
applies_to: ASP.NET Core
applies_to_vers: [1.0,1.1,2.0,2.1]
tags: [csharp, asp-net-core, partial-view, razor-page, view-component, mvc]
redirect_from:
  - blog/practical-asp-net-core/View-Components-and-Partial-Views-in-ASP-NET-Core-2
  - blog/practical-asp-net-core/View-Components-and-Partial-Views-in-ASP-NET-Core-3
---

{% include kramdown/ALDs.md %}
{% include post/get-prev.md link_title="What are View Components?" %}

Already know **[What View Components Are?]({{prev_post_href}})** If so, continue reading!

---
* TOC
{:toc}
---
{{ site.content_seperator }}
## Using View Components
In order to use a View Component, it must be invoked in code. In most cases, this will be done in a Razor View; less frequently, from a Controller action. Either method accepts the name of the View Component and an optional anonymous object containing parameters.

### Invoking a View Component from a Razor View
For invocation via Razor, the syntax is simply:

<span>`@`{:razor}`await`{:.k} `Component`{:.nt}.`InvokeAsync`{:.nf}(`ViewComponent Class`{:ph}, `new`{:.k} {`ViewComponent Params`{:ph}})</span>
{:cs .c-code-block}

Alternatively, the concrete implementation of the View Component can be passed as generic type `TComponent`{:cs g}: [^3]

<span>`@`{:razor}`await`{:.k} `Component`{:.nt}.`InvokeAsync`{:.nf}<`ViewComponent Class`{:ph}>(`new`{:.k} {`ViewComponent Params`{:ph}})</span>
{:cs .c-code-block}

> **TIP**{: .c-blockquote__title}
> The Razor block must be wrapped in parentheses to avoid compilation errors due to the generic argument.

### Invoking a View Component from a Controller Action
While less common, invoking a View Component from a Controller action isn't unheard of. In fact, it's a common way of refreshing a View Component via AJAX without requesting the whole page and cherry picking the desired element(s). The `Controller`{:cs .nc} method `ViewComponent()`{:cs .nf} returns `ViewComponentResult`{:cs .nc}, which implements `IActionResult`{:cs .n}, so any Controller action returning the latter can leverage it:
``` csharp
using Microsoft.AspNetCore.Mvc;
using System.Threading.Tasks;
public class ComponentController : Controller
{
  public async Task<IActionResult> GetSidebarComponent()
  {
    return ViewComponent("Sidebar");
  }
}
```

Similarly to its Razor counterpart, `ViewComponent()`{:cs .nf} has several overloads, accepting a required View Component name *or* type, and an optional anonymous object containing the View Component's parameters to be passed. While Model Binding is applicable to calling the Controller action itself, this still does not extend to invoking the View Component, which requires its parameters to be passed manually. All overloads return `ViewComponentResult`{:cs .nc}.

### Bringing it together
Let's build on our Sidebar example to demonstrate what we've learned so far. Keep in mind this is a simplified example, and some of the specifics will require suspension of disbelief or "Best Practices".

Suppose we have the business requirements we've lined out above: our ASP.NET Core website requires a sidebar with dynamic content, and at least one page which requires specialized markup for the same content. For variety, we'll also include a Title and a MotD

First, let's define the elements of our sidebar as `SidebarViewModel`{:cs .nc}:
``` csharp
using System.Collections.Generic;

public class SidebarViewModel
{
public string Title {get; set;}
public string MotD {get; set;}
public string Content {get; set;}
// Assume we'll store a Name/Link as our KV pair
public IDictionary<string,string> MenuItems {get; set;} 
}
```

If these values were static or driven by the parent View, a partial View which accepts an instance of `SidebarViewModel`{:cs .nc} as its `model`{:razor .k} would be sufficient. In this case, we don't want a sidebar dictated by the current View, and the content is likely to change on a frequent basis, suggesting an external resource.

Instead, we can create our standard-use partial View, `Default.cshtml`{:razor}, and save it in "Views/Shared/Components/Sidebar/":

``` html
@model SidebarViewModel
<div id="sidebar" class="c-sidebar">
  <h4 class="c-sidebar__title">@Model.Title</h4>
  <p class="c-sidebar__motd">@Model.MotD </p>
  <p class="c-sidebar__content">@Model.Content</p>
  <div class="c-sidebar__menu">
    @foreach (var item in @Model.MenuItems)
    {
      <a href="@item.Value">@item.Key</a>
    }
  </div>
</div>
```

While we're at it, let's create our "mini-sidebar" partial View. In this case, the title, MotD, and copy text are pointless since there won't be enough width to display them cleanly. However, we want to keep our navigation ability:

```html
<div id="sidebar" class="c-sidebar">
  <h4><i class="c-sidebar__title--icon"/></h4>
  <div class="c-sidebar__menu">
    @foreach (var item in @Model.MenuItems)
    {
      <a href="@item.Value">
        <i class="c-sidebar__icon--@item.Key.ToLower()"/>
      </a>
    }
  </div>
</div>
```

For the sake of demonstration, assume our CSS has `c-sidebar__title--icon`{:html .s} and matching modifiers of `c-sidebar__icon`{:html .s} of our MenuItem names to drive the icon content. We'll name this partial `Mini.cshtml`{:razor} and store it alongside our `Default.cshtml`{:razor}.

Now, we'll build our View Component incrementally, to showcase some highlights. At a minimum, we need an invocation parameter to drive whether the default View is used or not, and we need to pass our View Model:

``` csharp
using Microsoft.AspNetCore.Mvc;
using System.Threading.Tasks;

public class SidebarViewComponent : ViewComponents
{
  public async Task<IViewComponentResult InvokeAsync(bool useMini)
  {
    SidebarViewModel vm = new SidebarViewModel();
    if (useMini)
    {
      return View("Mini", vm);
    }
    else
    {
      return View(vm);
    }
  }
}
```

Of course, our View Model is `null`{:cs .k} here, and we don't want to use static values... But lucky us, View Components support constructor dependency injection!

Let's pretend we have some kind of application service for the express purpose of populating our sidebar. 
Keep in mind such a service is overkill, but sufficient for our example to demonstrate the "does something" part of a good View Component, as opposed to hard-coding our values.

We'll need to instantiate our application service:

``` csharp
private readonly ISidebarContentAppSvc _sidebarAppSvc;
public SidebarViewComponent(ISidebarContentAppSvc sidebarAppSvc)
{
  _sidebarAppSvc = sidebarAppSvc;
}
```

Then we can `await`{:cs .k} our various methods to populate our View Model properties:
``` csharp
// ...
public async Task<IViewComponentResult> InvokeAsync(bool useMini)
{
  SidebarViewModel vm = new SidebarViewModel()
  {
    Title =  await _sidebarAppSvc.GetTitleAsync(),
    MotD = await _sidebarAppSvc.GetMotDAsync(),
    Content = await _sidebarAppSvc.GetContentAsync(),
    MenuItems = await _sidebarAppSvc.GetMenuAsync()
  };

  if (useMini)
  {
    return View("Mini", vm);
  }

// ...
}
```

Right away, one thing should be apparent: we don't need those four extra service calls when we're using our miniature view, since none of those View Model properties will be used! If you also noticed the synchronous nature of how we're populating our View Model despite having async methods available, slow down--you're getting ahead of me!

Let's solve the first problem by splitting our View Model instantiation. Then we can `await`{:cs .k} our various methods to populate our View Model properties:

``` csharp
// ...
public async Task<IViewComponentResult> InvokeAsync(bool useMini)
{
  if(useMini)
  {
    SidebarViewModel vm = new SidebarViewModel()
    {
      MenuItems = await _sidebarAppSvc.GetMenuAync()
    };

    return View("Mini", vm);
  }
  else
  {
    SidebarViewModel vm = new SidebarViewModel()
    {
      Title =  await _sidebarAppSvc.GetTitleAsync(),
      MotD = await _sidebarAppSvc.GetMotDAsync(),
      Content = await _sidebarAppSvc.GetContentAsync(),
      MenuItems = `await  _sidebarAppSvc.GetMenuAsync()
    };

    return View(vm);
  }
}
// ...
```

At this point, our View Component is pretty feature complete! Optionally, we can parallelize our default View Model population by populating a collection of tasks from our asynchronous application service methods, and then awaiting those tasks when creating the View Model.

using System.Collections.Generic;

``` csharp
// ...
// Let's populate a dictionary with our tasks
// This will make it easy to await the correct Task
// For the corresponding VM property
IDictionary<string,Task> tasks = new Dictionary<string,Task>
{
  {"Title", _sidebarAppSvc.GetTitleAsync()},
  {"MotD", _sidebarAppSvc.GetMotDAsync()},
  {"Content", _sidebarAppSvc.GetContentAsync()},
  {"MenuItems", _sidebarAppSvc.GetMenuAsync()}
};
// Even though we're awaiting these tasks individually,
// Each Task began in parallel as soon as the dictionary was initialized,
// Rather than synchronously as in the previous example
SidebarViewModel vm = new SidebarViewModel()
{
  Title = await tasks["Title"],
  MotD = await tasks["MotD"],
  Content = await tasks["Content"],
  MenuItems = await tasks["MenuItems"]
};

return View(vm)

// ...
```

>**NOTE**{: .c-blockquote__title}
>If you aren't familiar with TPL this may seem like unnecessary indirection, but by calling these methods without awaiting them, they are ran in parallel.<br/>While we do `await`{:k style="font-size:inherit"} each task when building our View Model, at that point each task is (ostensibly) complete, whereas in the former iteration we would be calling then waiting for each method synchronously. We'll cover asynchronous programming, the `async/await`{:k style="font-size:inherit"} pattern, and parallelization in another article.

#### The Finished Product

Finally, our View Component showcases a few things:
  * Constructor dependency injection is used to instantiate a hypothetical application service for returning our sidebar content.
    - The application service contains several asynchronous methods which access data external to our ASP.NET Core web project itself, with inconsistent response time. Therefore, our View Component will call these methods in parallel, where possible.
  * Optional partial views are driven by the invocation parameter `useMini`. This dictates which partial View is used: either the Default, which is acceptable for the majority of our pages, or a custom "collapsed" View `Mini`{:razor}
    - If not set, the parameter's `bool`{:cs .k} type defaults to `false`{:cs .k}.
    - If set to `true`{:cs .k}, the alternative `Mini`{:razor} partial View will be used, and only `SidebarViewModel`{:cs .nc}.`MenuItems` will be assigned, saving us unnecessary application service calls.

``` csharp
using Microsoft.AspNetCore.Mvc;
using System.Collections.Generic;
using System.Threading.Tasks;
public class SidebarViewComponent : ViewComponent
{
  private readonly ISidebarContentAppSvc _sidebarAppSvc;

  public SidebarViewComponent(ISidebarContentAppSvc sidebarAppSvc)
  {
    _sidebarAppSvc = sidebarAppSvc;
  }
  
  public async Task<IViewComponentResult> InvokeAsync(bool useMini)
  {
    if (useMini)
    {
      SidebarViewModel vm = new SidebarViewModel()
      {
        MenuItems = await _sidebarAppSvc.GetMenuAync()
      };
      
      return View("Mini", vm);
    }
    else
    {
      IDictionary<string,Task> tasks = new Dictionary<string,Task>
      {
        {"Title", _sidebarAppSvc.GetTitleAsync()},
        {"MotD", _sidebarAppSvc.GetMotDAsync()},
        {"Content", _sidebarAppSvc.GetContentAsync()},
        {"MenuItems", _sidebarAppSvc.GetMenuAsync()}
      };

      SidebarViewModel vm = new SidebarViewModel()
      {
        Title = await tasks["Title"],
        MotD = await tasks["MotD"],
        Content = await tasks["Content"],
        MenuItems = await tasks["MenuItems"]
      };
      
      return View(vm)
    }
  }
}
```

To use our shiny new View Component, we simply invoke it in our Razor views.

For the Default View:
<span>`@(`{:razor}`await`{:.k} `Component`{:.nc}.`InvokeAsync`{:.nf}<`SidebarViewComponent`{:.nc}>()`)`{:razor}
</span>{:blk cs}
For the miniature View:
<span>`@(`{:razor}`await`{:.k} `Component`{:.nc}.`InvokeAsync`{:.nf}<`SidebarViewComponent`{:.nc}>(`new`{:.k} {useMini = `true`{:.k}})`)`{:razor}
</span>{:blk cs}

>#### **DISCLAIMER**
>This is an example to showcase what we've discussed so far; production-ready code should have substantially more thought put into the "why" and "how", and probably some inline comments for Future You and your team!

## See Also
* Microsoft Docs:
  - [View components in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/view-components?view=aspnetcore-2.2) By [Rick Anderson](https://twitter.com/RickAndMSFT)
  
## Footnotes
[^3]: This overload is technically not part of `Microsoft.AspNetCore.Mvc.IViewComponentHelper`, but an extension method provided by `Microsoft.AspNetCore.Mvc.Rendering.ViewComponentHelperExtensions`.

*[MotD]: Message of The Day
*[Future You]: In my experience, usually Next-Monday You.
