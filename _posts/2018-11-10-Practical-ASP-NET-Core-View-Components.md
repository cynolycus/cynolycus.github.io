---
layout: post
title: "Practical ASP.NET Core: View Components"
series: "Practical ASP.NET Core"
series_subtitle: "View Components"
date: 2018-11-10
last_modified_at: 2018-12-02
excerpt: Modularize all the things! New to ASP.NET Core, View Components are great for a modular approach in your MVC or Razor Pages website, but they aren't appropriate for every situation.
description: New to ASP.NET Core, a View Component is a great tool for a modular approach in your MVC or Razor Pages website, but they aren't appropriate for every situation.
tags: [csharp,ASP.NET Core,Practical ASP.NET Core,View Components]
applies_to: ASP.NET Core
applies_to_vers: [1.0,1.1,2.0,2.1]
---

{% include kramdown/ALDs.md %}
---
* TOC
{:toc}
---
## What are View Components?
Introduced in ASP.NET Core, a View Component is essentially a partial View with business logic, or conversely a miniature Controller. Their versatility makes View Components a great tool for taking a modular approach in designing your ASP.NET Core application, but they aren't necessarily appropriate for every situation. If you've worked with MVC and partial Views before, View Components will seem familiar at best, or redundant at worst. They're very simple in principle, and very powerful if leveraged appropriately. 
### What they are
A View Component usually consists of two parts: a class which inherits from `ViewComponent`{:cs cl token} and contains some kind of logic, and an associated partial View. The latter isn't strictly required, but we will revisit that later in the article. Like a Partial View, View Components return a chunk of HTML, which is to say *just* its corresponding View.[^1] Like Controllers, they encapsulate potentially complex business logic, support constructor dependency injection, and are independently testable.
### What they *aren't*
While these are convenient abstractions, there's some critical differences:
1. View Components are internal constructs which cannot be accessed by external requests *directly*. In other words, they cannot be reached by and **do not handle HTTP requests**.
2. **View Components do not participate in the Controller lifecycle**. This means they cannot leverage the Filter pipeline[^2], nor do they use model binding.
3. Rather than binding parameters from HTTP request values, **View Component parameters are passed as an anonymous object during invocation**.

### What's the point?
As with any other tool in your .NET Core arsenal, View Components are as useful as they are appropriate for the problem at hand. The name implies reusability and cohesion, and it's best to think of a View Component as a module or widget that is part of one or more larger Views/Pages/Layouts within your ASP.NET Core project. Depending on your business requirements and design guidelines, your Views could be largely driven by multiple View Components which are fed parameters from the "parent" View's View Model. With that in mind, View Components are most useful when you have **chunks of a web page that require additional processing independent of the caller**.

View Components supersede a simple Partial View, which usually has minimal or no logic, and even then chiefly *with respect to how the View Model is displayed*. Developers can often run in to hang-ups when it comes to the `ViewDataDictionary` and passing a View Model different than the parent View's; this issue is sidestepped using a View Component. They also precede a full-blown dedicated Controller returning a Partial View, which has the same or greater capabilities; a Controller may be more overhead than necessary, or there may be no intention for the View Component to be accessible by external requests.

Whether or not it accepts input parameters from the calling View/Controller, a View Component should do *something* when invoked: whether that's accessing another resource (such as a database or web API), validating inputs, or determining rendering logic and which Partial View should be returned. Be careful not to get overzealous about them, though: if a Partial View with some Razor markup is enough for your needs, stick with that. View Components do not *replace* any existing functionality, **they bridge the gap between a pure Partial View and a Controller/View pair**.

## Constructing a View Component
### The View Component Class

>`using`{:cs k} Microsoft.AspNetCore.Mvc;
>
>`using`{:cs k} System.Threading.Tasks;
>
>`public class`{:cs k} `Sidebar`{:cs ucl} : `ViewComponent`{:cs cl}
>
>{
>>`public async`{:cs k} `Task`{:cs cl}<`IViewComponentResult`{:cs g}> `InvokeAsync`{:cs m}()
>>
>>{
>>>`return`{:cs k} View();
>>
>>}
>
>}
{:.o-code__block}

In this barebones example, our View Component `Sidebar`{:cs ucl token} derives from `ViewComponent`{:cs cl token} and implements the required method `InvokeAsync()`{:cs m token} with no parameters, returning its Default View. However, as with most things in ASP.NET Core, there's more than one correct way to create a working View Component.

They can also be defined by naming convention, applying "ViewComponent" as a suffix to the class name:


>`// I'm a View Component!`{:cs cm}
>
>`public class`{:cs k} `SidebarViewComponent`{:cs ucl}
>
>{
>>`// InvokeAsync()`{:cs cm}
>
>}
{:.o-code__block}

Or, by decoration:

>`// Me too!`{:cs cm}
>
>`[ViewComponent]`{:cs g}
>
>`public class`{:cs k} `Sidebar`{:cs ucl}
>
>{
>>`// InvokeAsync()`{:cs cm}
>
>}
{:.o-code__block}

By default, the View Component's name is the prefixed class name; the "ViewComponent" suffix is ignored. This can be overridden by setting the `ViewComponentAttribute.Name`{:cs k token} property of the base `ViewComponent`{:cs cl token}.

**I personally use a combination of the first two approaches**:


>`public class`{:cs k} `SidebarViewComponent`{:cs ucl} : `ViewComponent`{:cs cl}
>
>{
>>`// InvokeAsync()`{:cs cm}
>
>}
{:.o-code__block}

While it may be redundant, it also makes class relationships abundantly obvious (at least to Future You), viz. a `Sidebar`{:cs ucl token} object class or View Model and a `SidebarViewComponent`{:cs ucl token} to dictate lookup/rendering. Ultimately, it's down to preference and consistency; pick a format and stick with it!

#### The Partial View
One of the selling points for View Components is that the associated Partial View requires no special considerations; in fact, [it isn't even necessary](#the-content-method). That said, for most use cases you'll find `ViewComponent`{:cs cl}.`View()`{:cs m} sufficient. This is the bread-and-butter method for passing data or a View Model to a partial View from your View Component.

#### The View() Method
This is really where View Components start to shine, but before we can dive into how [`View()`{:cs m token}](#the-view-method) can be leveraged, we need to discuss what happens behind-the-scenes. If no view name is specified, this return method targets a Partial View file named `Default.cshtml`{:razor} in a predefined directory.

When a ViewComponent is invoked, the following locations are checked, in order:
1. Pages/Components/`ViewComponent Name`{:ph}/
2. Views/`Controller Name`{:ph}/Components/`ViewComponent Name`{:ph}/
3. <span>Views/Shared/Components/`ViewComponent Name`{:ph}/</span>{:#recommended-path} <sup>**`Recommended!`**</sup>
{:#default-paths}
Simple, right? **The last of these is recommended by Microsoft**, and it makes sense: the first two locations are more specific, allowing the selected view to be overridden based on whether the component is called from a Razor Page or from a specific controller  respectively.

In a Razor Pages only project, the "Pages/Components/" path is preferable to keep your project folder structure simple. To illustrate where this would be useful in an MVC or hybrid MVC/Razor Pages application, take our `Sidebar`{:cs ucl token} example:
* My Default view would be created in `Views/Shared/Components/Sidebar/Default.cshtml`.
* Suppose I need to adjust the layout on a particular page of my website that the component appears on, **Catalog**; perhaps I want it in a collapsed configuration to maximize horizontal space, and to remove some extraneous content.
* No problem, I just add my revised view to `Views/Catalog/Components/Default.cshtml`, assuming the Controller for **Catalog** is also named `Catalog`{:cs ucl token}.
* If **Catalog** is instead a Razor Page, I can move my custom view to `Pages/Components/Default.cshtml`, _BUT_ this will apply to *all* Razor Pages calling the same component!

Like our class naming, consider the project at hand and what makes the most sense to you and your team. However, there is another approach, which I prefer: using a *named partial view* (in this case `Mini.cshtml`{:razor}) and placing it in the [recommended directory](#recommended-path).

##### View() Overloads
Now that we understand how [`View()`{:cs m token}](#the-view-method) returns a partial View, we can take a closer look at how View Components utilize named Partial Views and View Models. The method has several overloads, accepting two optional parameters,
`string`{:cs k} `viewName`{:cs} and `TModel`{:cs g} `model`{:cs}:
* As the parameter name implies, `viewName` allows you to change the targeted view from `Default`{:razor} to a specified file name; the file extension `.cshtml`{:razor} is unnecessary in most cases.
  - If only a view name is provided, the [default paths](#default-paths) are searched for it; if a full path is provided, it will be used exclusively.
  - If `viewName` is unspecified or null, `Default`{:razor} is used instead.
* The generic `model` parameter accepts an object to be passed to the invoked partial View; typically this should match the `model`{:razor k} of the partial View. If none is specified, the View's model will be `null`.

| Overload | Usage |
| :--- | :--- |
| `View`{:cs m}()|`return`{:cs k} `View`{:cs m}()|
| `View`{:cs m}(`string`{:cs k} viewName)|`return`{:cs k} `View`{:cs m}("`ViewComponent Name`{:ph}")|
| `View`{:cs m}<`TModel`{:cs g}>(`TModel`{:cs g} model)|`return`{:cs k} `View`{:cs m}(`ViewModel Instance`{:ph})|
| `View`{:cs m}<`TModel`{:cs g}(<`string`{:cs k} viewName, `TModel`{:cs g} model)|`return`{:cs k} `View`{:cs m}("`ViewComponent Name`{:ph}", `ViewModel Instance`{:ph})|

#### The Content() Method
In some cases, you may not need or have a partial view to return from your View Component. Some examples include, but are not limited to:
* Returning HTML from an external source, such as a database, file, or web API.
* Returning a string response, where a separate View file and/or View Model is unnecessary overhead.
* Conditionally returning a default response, such as unauthorized access or invalid input parameters.
* Conditionally returning an empty response without dealing with a `null` View Model in the partial View.

In these cases, the View Component can return a `ContentViewComponentResult`{:cs cl token} using the `Content()`{:cs m token} method, which will HTML encode any string passed to it. This can be sufficient when combined with database lookups, string interpolation, and other means of constructing meaningful return values without requiring the additional complexity of a dedicated View, such as a Message of the Day, or a user message if a particular Component is unavailable.

## Using View Components
In order to use a View Component, it must be invoked in code. In most cases, this will be done in a Razor View; less frequently, from a Controller action. Either method accepts the name of the View Component and an optional anonymous object containing parameters.

### Invoking from a Razor View
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

### Invoking from a Controller Action
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

## Footnotes
[^1]: By default, a View Component's `Layout`{:razor} is null. Consequently, `section`{:razor k} blocks will never be rendered, nor throw exceptions. While this can be overridden, it's outside of the scope of this post. It will be covered more in-depth in a future submission.
[^2]: Specifically `OnActionExecuting()`{:cs m token} and `OnActionExecuted()`{:cs m token}. As will be covered later in this series, it is still possible to leverage *some* filters.
[^3]: This overload is technically not part of Microsoft.AspNetCore.Mvc.IViewComponentHelper, but an extension method provided by Microsoft.AspNetCore.Mvc.Rendering.ViewComponentHelperExtensions.

*[MotD]: Message of The Day
*[Future You]: In my experience, usually Next-Monday You.
