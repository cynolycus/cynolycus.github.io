---
title: "Using View Components and Partial Views in ASP.NET Core"
category: asp-net-core
order: 2
date: 2018-12-01T00:02:00-5:00
last_modified_at: 2019-01-01
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
<small>Return to **[{{prev_post_name}}]({{prev_post_href}})**<small>

---
* TOC
{:toc}
---
{{ site.content_seperator }}
## Using View Components
In order to use a View Component, it must be invoked in code. In most cases, this will be done in a Razor View; less frequently, from a Controller action. Either method accepts the name of the View Component and an optional anonymous object containing parameters.

### Invoking a View Component from a Razor View
For invocation via Razor, the syntax is simply:
<span>
`@`{:razor}`await`{:cs k} `Component`{:cs cl}.`InvokeAsync`{:cs m}("`ViewComponent Name`{:token ph}", `new`{:cs k} {`ViewComponent Params`{:token ph}})
</span>
Alternatively, the concrete implementation of the View Component can be passed as generic type `TComponent`{:cs g}: [^3]
<span>
`@(`{:razor}`await`{:cs k} `Component`{:cs cl}.`InvokeAsync`{:cs m}<`ViewComponent Name`{:token ph}>(`new`{:cs k} {`ViewComponent Params`{:token ph}})`)`{:razor}
</span>
>**TIP**<br/>
> The Razor block must be wrapped in parentheses to avoid compilation errors due to the generic argument.

### Invoking a View Component from a Controller Action
While less common, invoking a View Component from a Controller action isn't unheard of. In fact, it's a common way of refreshing a View Component via AJAX without requesting the whole page and cherry picking the desired element(s). The `Controller`{:cs cl token} method `ViewComponent()`{:cs m token} returns `ViewComponentResult`{:cs cl token}, which implements `IActionResult`{:cs g token}, so any Controller action returning the latter can leverage it:

>`using`{:cs k} Microsoft.AspNetCore.Mvc;
>
>`using`{:cs k} System.Threading.Tasks;
>
>`public class`{:cs k} `ComponentController`{:cs ucl} : `Controller`{:cs cl}
>
>{
>>`public async`{:cs k} `Task`{:cs cl}<`IActionResult`{:cs g}> `GetSidebarComponent`{:cs m}()
>>
>>{
>>>`return`{:cs k} ViewComponent(`"Sidebar"`{:cs cm});
>>
>>}
>
>}
{:.o-code__block}

Similarly to its Razor counterpart, `ViewComponent()`{:cs m token} has several overloads, accepting a required View Component name *or* type, and an optional anonymous object containing the View Component's parameters to be passed. While Model Binding is applicable to calling the Controller action itself, this still does not extend to invoking the View Component, which requires its parameters to be passed manually. All overloads return `ViewComponentResult`{:cs cl token}.
#### ViewComponent() Overloads

| Overload | Usage|
| :--- | :--- |
| `ViewComponent`{:cs m}(`Type`{:cs cl} componentType)|`return`{:cs k} `ViewComponent`{:cs m}(`typeof`{:cs k}(`ViewComponent Type`{:token ph}))|
| `ViewComponent`{:cs m}(`string`{:cs k} componentName)|`return`{:cs k} `ViewComponent`{:cs m}("`ViewComponent Name`{:token ph}")|
| `ViewComponent`{:cs m}(`Type`{:cs cl} componentType), `object`{:cs k} args)|`return`{:cs k} `ViewComponent`{:cs m}(`typeof`{:cs k}(`ViewComponent Type`{:token ph}), `new`{:cs k} {`ViewComponent Params`{:token ph}})|
| `ViewComponent`{:cs m}(`string`{:cs k} componentName, `object`{:cs k} args)|`return`{:cs k} `ViewComponent`{:cs m}("`ViewComponent Name`{:token ph}", `new`{:cs k} {`ViewComponent Params`{:token ph}})|
{:.o-code--table}

### Bringing it together
Let's build on our Sidebar example to demonstrate what we've learned so far. Keep in mind this is a simplified example, and some of the specifics will require suspension of disbelief or "Best Practices".

Suppose we have the business requirements we've lined out above: our ASP.NET Core website requires a sidebar with dynamic content, and at least one page which requires specialized markup for the same content. For variety, we'll also include a Title and a MotD

First, let's define the elements of our sidebar as `SidebarViewModel`{:cs ucl token}:
>`using`{:cs k} System.Collections.Generic;
>
>`public class`{:cs k} SidebarViewModel
>
>{
>>`public string`{:cs k} Title {get; set;}
>>
>>`public string`{:cs k} MotD {get; set;}
>>
>>`public string`{:cs k} Content {get; set;}
>>
>>// Assume we'll store a Name/Link as our KV pair
>>
>>`public`{:cs k} `IDictionary`{:cs g}<`string`{:cs k},`string`{:cs k}> MenuItems {get; set;}
>
>}
{:.o-code__block}
Now, if these values were static or driven by the parent View, **a partial View which accepts an instance of** `SidebarViewModel`{:cs ucl token} **as its** `model`{:razor k} **would be sufficient.** In this case, we don't want a sidebar dictated by the current View, and the content is likely to change on a frequent basis, suggesting an external resource.

Nevertheless, we can create our standard-use partial View, `Default.cshtml`{:razor}, and save it in "Views/Shared/Components/Sidebar/":

```html
@model SidebarViewModel
<div id="sidebar" class="c-sidebar">
  <h4 class="c-sidebar__title">@Model.Title</h4>
  <p class="c-sidebar__motd">
    @Model.MotD
  </p>
  <p class="c-sidebar__content">
    @Model.Content
  </p>
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
For the sake of demonstration, assume our CSS has `c-sidebar__title--icon` and matching modifiers of `c-sidebar__icon` of our MenuItem names to drive the icon content. We'll name this partial `Mini.cshtml`{:razor} and store it alongside our `Default.cshtml`{:razor}.

Now, we'll build our View Component incrementally, to showcase some highlights. At a minimum, we need an invocation parameter to drive whether the default View is used or not, and we need to pass our View Model:


>`using`{:cs k} Microsoft.AspNetCore.Mvc;
>
>`using`{:cs k} System.Threading.Tasks;
>
>`public class`{:cs k} `SidebarViewComponent`{:cs ucl}: `ViewComponents`{:cs cl}
>{
>>
>>`public async`{:cs k} `Task`{:cs cl}\<`IViewComponentResult`{:cs g}\> `InvokeAsync`{:cs m}(`bool`{:cs k} useMini)
>>
>>{
>>>
>>>`SidebarViewModel`{:cs ucl} vm = `new`{:cs k} `SidebarViewModel`{:cs ucl}();
>>>
>>>`if`{:cs k} (useMini)
>>>
>>>{
>>>>
>>>>`return`{:cs k} `View`{:cs m}("Mini", vm);
>>>
>>>}
>>>
>>>`else`{:cs k}
>>>
>>>{
>>>>
>>>>`return`{:cs k} `View`{:cs m}(vm);
>>>
>>>}
>>
>>}
>
>}
{:.o-code__block}

Of course, our View Model is `null` here, and we don't want to use static values... Oh, that's right, View Components support constructor dependency injection! Let's pretend we have some kind of application service for the express purpose of populating our sidebar.
We'll need to instantiate our application service:

>`private readonly`{:cs k} `ISidebarContentAppSvc`{:cs g} _sidebarAppSvc;
>
>`public`{:cs k} SidebarViewComponent(`ISidebarContentAppSvc`{:cs g} sidebarAppSvc)
>
>{
>>_sidebarAppSvc = sidebarAppSvc;
>
>}
{:.o-code__block}
Then we can `await`{:cs k} our various methods to populate our View Model properties:
>`// ...`{:cs cm}
>
>`public async`{:cs k} `Task`{:cs cl}<`IViewComponentResult`{:cs g}> `InvokeAsync`{:cs m}(`bool`{:cs k} useMini)
>
>{
>>`SidebarViewModel`{:cs ucl} vm = `new`{:cs k} `SidebarViewModel`{:cs ucl}()
>>
>>{
>>>Title =  `await`{:cs k} _sidebarAppSvc.GetTitleAsync(),
>>>
>>>MotD = `await`{:cs k} _sidebarAppSvc.GetMotDAsync(),
>>>
>>>Content = `await`{:cs k} _sidebarAppSvc.GetContentAsync(),
>>>
>>>MenuItems = `await`{:cs k}  _sidebarAppSvc.GetMenuAsync()
>>
>>};
>>
>>`if`{:cs k} (useMini)
>>
>>{
>>>`return`{:cs k} View(`"Mini"`{:cs cm}, vm);
>>
>>}
>
>`// ...`{:cs cm}
>
>}
{:.o-code__block}
Keep in mind such a service is overkill, but sufficient for our example to demonstrate the "does something" part of a good View Component, as opposed to hard-coding our values in the name of simplicity.

Right away, one thing should be apparent for you folks playing at home. If you said something like 'we don't need those four extra service calls when we're using our miniature view, since none of those View Model properties will be used!', then you're paying attention! If you also pointed out the pseudo-asynchronous nature of how we're populating our View Model, slow down--you're getting ahead of me!

Let's solve the first problem by splitting our View Model instantiation. Then we can `await`{:cs k} our various methods to populate our View Model properties:
>`// ...`{:cs cm}
>
>`public async`{:cs k} `Task`{:cs cl}<`IViewComponentResult`{:cs g}> `InvokeAsync`{:cs m}(`bool`{:cs k} useMini)
>
>{
>>`if`{:cs k} (useMini)
>>
>>{
>>>`SidebarViewModel`{:cs ucl} vm = `new`{:cs k} `SidebarViewModel`{:cs ucl}()
>>>
>>>{
>>>>MenuItems = `await`{:cs k} _sidebarAppSvc.GetMenuAync()
>>>
>>>};
>>>
>>>`return`{:cs k} View(`"Mini"`{:cs cm}, vm);
>>
>>}
>>
>>`else`{:cs k}
>>
>>{
>>>`SidebarViewModel`{:cs ucl} vm = `new`{:cs k} `SidebarViewModel`{:cs ucl}()
>>>
>>>{
>>>>Title =  `await`{:cs k} _sidebarAppSvc.GetTitleAsync(),
>>>>
>>>>MotD = `await`{:cs k} _sidebarAppSvc.GetMotDAsync(),
>>>>
>>>>Content = `await`{:cs k} _sidebarAppSvc.GetContentAsync(),
>>>>
>>>>MenuItems = `await`{:cs k}  _sidebarAppSvc.GetMenuAsync()
>>>
>>>};
>>>
>>>`return`{:cs k} View(vm);
>>
>>}
>
>}
>
>`// ...`{:cs cm}
{:.o-code__block}

At this point, our View Component is pretty feature complete! Optionally, we can parallelize our default View Model population by populating a collection of tasks from our asynchronous application service methods, and then awaiting those tasks when creating the View Model.

>`using`{:cs k} System.Collections.Generic;
>
>`// ...`{:cs cm}
>
>`IDictionary`{:cs g}<`string`{:cs k},`Task`{:cs cl}> tasks = `new`{:cs k} `Dictionary`{:cs cl}<`string`{:cs k},`Task`{:cs cl}>
>
>{
>>
>>{`"Title"`{:cs cm}, _sidebarAppSvc.GetTitleAsync()},
>>
>>{`"MotD"`{:cs cm}, _sidebarAppSvc.GetMotDAsync()},
>>
>>  {`"Content"`{:cs cm}, _sidebarAppSvc.GetContentAsync()},
>>
>>  {`"MenuItems"`{:cs cm}, _sidebarAppSvc.GetMenuAsync()}
>
>};
>
>`SidebarViewModel`{:cs ucl} vm = new `SidebarViewModel`{:cs ucl}()
>
>{
>>
>>  Title = `await`{:cs k} tasks[`"Title"`{:cs cm}],
>>
>>  MotD = `await`{:cs k} tasks[`"MotD"`{:cs cm}],
>>
>>  Content = `await`{:cs k} tasks[`"Content"`{:cs cm}],
>>
>>  MenuItems = `await`{:cs k} tasks[`"MenuItems"`{:cs cm}]
>
>};
>
>`return`{:cs k} View(vm);
>
>`// ...`{:cs cm}
{:.o-code__block}
>**NOTE**<br/>
>If you aren't familiar with TPL this may seem like unnecessary indirection, but by calling these methods without awaiting them, they are ran in parallel.<br/>While we do `await`{:cs k style="font-size:inherit"} each task when building our View Model, at that point each task is (ostensibly) complete, whereas in the former iteration we would be calling then waiting for each method synchronously. We'll cover asynchronous programming, the `async/await`{:cs k style="font-size:inherit"} pattern, and parallelization in another article.

#### The Finished Product

Finally, our View Component showcases a few things:
  * Constructor dependency injection is used to instantiate a hypothetical application service for returning our sidebar content.
    - The application service contains several asynchronous methods which access data external to our ASP.NET Core web project itself, with inconsistent response time. Therefore, our View Component will call these methods in parallel, where possible.
  * The invocation parameter `useMini` dictates which partial View is used: either the Default, used for the majority of our pages, our a custom "collapsed" View, `Mini`{:razor}
    - If not set, its `bool`{:cs k} type defaults to `false`{:cs k}.
    - If set to `true`{:cs k}, the alternative `Mini`{:razor} partial View will be used, and only `SidebarViewModel`{:cs ucl}.`MenuItems` will be assigned, saving us unnecessary application service calls.

>`using`{:cs k} Microsoft.AspNetCore.Mvc;
>
>`using`{:cs k} System.Collections.Generic;
>
>`using`{:cs k} System.Threading.Tasks;
>
>`public class`{:cs k} `SidebarViewComponent`{:cs ucl}: `ViewComponents`{:cs cl}
>{
>>
>>`private readonly`{:cs k} `ISidebarContentAppSvc`{:cs g} _sidebarAppSvc;
>>
>>`public`{:cs k} SidebarViewComponent(`ISidebarContentAppSvc`{:cs g} sidebarAppSvc)
>>
>>{
>>>_sidebarAppSvc = sidebarAppSvc;
>>
>>}
>>
>>`public async`{:cs k} `Task`{:cs cl}\<`IViewComponentResult`{:cs g}\> `InvokeAsync`{:cs m}(`bool`{:cs k} useMini)
>>
>>{
>>>
>>>`if`{:cs k} (useMini)
>>>
>>>{
>>>>`SidebarViewModel`{:cs ucl} vm = `new`{:cs k} `SidebarViewModel`{:cs ucl}()
>>>>
>>>>{
>>>>>MenuItems = `await`{:cs k} _sidebarAppSvc.GetMenuAync()
>>>>
>>>>};
>>>>
>>>>`return`{:cs k} View(`"Mini"`{:cs cm}, vm);
>>>
>>>}
>>>
>>>`else`{:cs k}
>>>
>>>{
>>>>`IDictionary`{:cs g}<`string`{:cs k},`Task`{:cs cl}> tasks = `new`{:cs k} `Dictionary`{:cs cl}<`string`{:cs k},`Task`{:cs cl}>
>>>>
>>>>{
>>>>>
>>>>>{`"Title"`{:cs cm}, _sidebarAppSvc.GetTitleAsync()},
>>>>>
>>>>>{`"MotD"`{:cs cm}, _sidebarAppSvc.GetMotDAsync()},
>>>>>
>>>>>  {`"Content"`{:cs cm}, _sidebarAppSvc.GetContentAsync()},
>>>>>
>>>>>  {`"MenuItems"`{:cs cm}, _sidebarAppSvc.GetMenuAsync()}
>>>>
>>>>};
>>>>
>>>>`SidebarViewModel`{:cs ucl} vm = new `SidebarViewModel`{:cs ucl}()
>>>>
>>>>{
>>>>>
>>>>>  Title = `await`{:cs k} tasks[`"Title"`{:cs cm}],
>>>>>
>>>>>  MotD = `await`{:cs k} tasks[`"MotD"`{:cs cm}],
>>>>>
>>>>>  Content = `await`{:cs k} tasks[`"Content"`{:cs cm}],
>>>>>
>>>>>  MenuItems = `await`{:cs k} tasks[`"MenuItems"`{:cs cm}]
>>>>
>>>>};
>>>>
>>>>`return`{:cs k} View(vm);
>>>
>>>}
>>
>>}
>
>}
{:.o-code__block}

To use our shiny new View Component, we simply invoke it in our Razor views.

For the Default View:
>`@(`{:razor}`await`{:cs k} `Component`{:cs cl}.`InvokeAsync`{:cs m}<`SidebarViewComponent`{:cs ucl}>()`)`{:razor}
{:.o-code__block}
For the miniature View:
>`@(`{:razor}`await`{:cs k} `Component`{:cs cl}.`InvokeAsync`{:cs m}<`SidebarViewComponent`{:cs ucl}>(`new`{:cs k} {useMini = `true`{:cs k}})`)`{:razor}
{:.o-code__block}

>**DISCLAIMER**<br/>
>This is an example to showcase what we've discussed so far; production-ready code should have substantially more thought put into the "why" and "how", and probably some inline comments for Future You and your team!

## See Also
* Microsoft Docs:
  - [View components in ASP.NET Core](https://docs.microsoft.com/en-us/aspnet/core/mvc/views/view-components?view=aspnetcore-2.2) By [Rick Anderson](https://twitter.com/RickAndMSFT)


*[MotD]: Message of The Day
*[Future You]: In my experience, usually Next-Monday You.
