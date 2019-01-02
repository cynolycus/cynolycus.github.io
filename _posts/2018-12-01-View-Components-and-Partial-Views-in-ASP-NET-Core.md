---
title: "Introduction to View Components and Partial Views in ASP.NET Core"
subtitle: "A technical overview"
category: asp-net-core
order: 1
date: 2018-12-01T00:00:00-5:00
last_modified_at: 2019-01-01
excerpt: Modularize all the things! New to ASP.NET Core, View Components are a great tool for modularizing your MVC or Razor Pages site and empowering your Partial Views.
description: Introduced with the advent of ASP.NET Core, View Components are a great tool for modularizing your MVC or Razor Pages site and empowering your Partial Views. Their versatility makes View Components a great addition to any MVC application, but they aren't necessarily appropriate for every situation.
applies_to: ASP.NET Core
applies_to_vers: [1.0,1.1,2.0,2.1,2.2]
tags: [csharp, asp-net-core, partial-view, razor-page, view-component, mvc]
redirect_from: blog/practical-asp-net-core/View-Components-and-Partial-Views-in-ASP-NET-Core
---
{% include kramdown/ALDs.md %}
{% include post/get-next.md link_title="Using a View Component" %}
---
* TOC
{:toc}
---
{{ site.content_seperator }}
## What are View Components?
Introduced with the advent of ASP.NET Core, View Components are a great tool for modularizing your MVC or Razor Pages site and empowering your Partial Views. A View Component is essentially a partial View with embeded business logic. If you prefer, you can think of them instead as a miniature Controller with a very specific purpose and associated partial View. Their versatility makes View Components a great addition to any MVC application, but they aren't necessarily appropriate for every situation. If you've worked with MVC and partial Views before ASP.NET Core, View Components will seem familiar at best, or redundant at worst. They're very simple in principle, and very powerful if leveraged appropriately.
### What they are
A View Component usually consists of two parts: a class which inherits from `ViewComponent`{:cs cl token} and contains some kind of logic, and an associated partial View. The latter isn't strictly required, but we will revisit that later in the article. Like a Partial View, View Components return a chunk of HTML, which is to say *just* its corresponding View.[^1] Like Controllers, they encapsulate potentially complex business logic, support constructor dependency injection, and are independently testable.
### What they *aren't*
While these are convenient abstractions, there's some critical differences:
1. View Components are internal constructs which cannot be accessed by external requests *directly*. In other words, they cannot be reached by and **do not handle HTTP requests**.
2. **View Components do not participate in the Controller lifecycle**. This means they cannot leverage the Filter pipeline[^2], nor do they use model binding.
3. Rather than binding parameters from HTTP request values, **View Component parameters are passed as an anonymous object during invocation**.

### What's the point?
As with any other tool in your .NET Core arsenal, View Components are as useful as they are appropriate for the problem at hand. The name implies reusability and cohesion, and it's best to think of a View Component as a module or widget that is part of one or more larger Views/Pages/Layouts within your ASP.NET Core project. Depending on your business requirements and design guidelines, your Views could be largely driven by multiple View Components which are fed parameters from the "parent" View's View Model. With that in mind, View Components are most useful when you have **chunks of a web page that require additional processing independent of the caller**.

View Components supersede a simple Partial View, which usually has minimal or no logic, and even then only *with respect to how the View Model is displayed*. Developers can often run in to hang-ups when it comes to the `ViewDataDictionary` and passing a View Model different than the parent View's; this issue is sidestepped using a View Component. They also precede a full-blown dedicated Controller returning a Partial View, which has the same or greater capabilities; a Controller may be more overhead than necessary, or there may be no intention for the View Component to be accessible by external requests.

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

### The View() Method

This is really where View Components start to shine, but before we can dive into how [`View()`{:cs m token}](#the-view-method) can be leveraged, we need to discuss what happens behind-the-scenes. One of the selling points for View Components is that the associated Partial View requires no special considerations; in fact, [it isn't even necessary](#the-content-method). That said, for most use cases you'll find `ViewComponent`{:cs cl}.`View()`{:cs m} sufficient. This is the bread-and-butter method for passing data or a View Model to a partial View from your View Component. If no view name is specified, this method returns a Partial View file named `Default.cshtml`{:razor} in a predefined directory.

When a ViewComponent is invoked and the `View()`{:cs m} method returned, the following locations are checked for the Patial View, in order:
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

Like with our class naming, consider the project at hand and what makes the most sense to you and your team. However, there is another approach, which I prefer: using a *named partial view* (in this case `Mini.cshtml`{:razor}) and placing it in the [recommended directory](#recommended-path).

#### View() Overloads
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

### The Content() Method
In some cases, you may not need or have a partial view to return from your View Component. Some examples include, but are not limited to:
* Returning HTML from an external source, such as a database, file, or web API.
* Returning a string response, where a separate View file and/or View Model is unnecessary overhead.
* Conditionally returning a default response, such as unauthorized access or invalid input parameters.
* Conditionally returning an empty response without dealing with a `null` View Model in the partial View.

In these cases, the View Component can return a `ContentViewComponentResult`{:cs cl token} using the `Content()`{:cs m token} method, which will HTML encode any string passed to it. This can be sufficient when combined with database lookups, string interpolation, and other means of constructing meaningful return values without requiring the additional complexity of a dedicated View, such as a Message of the Day, or a user message if a particular Component is unavailable.

## {{next_post_name}}
Continue to **[{{next_post_name}}]({{next_post_href}})**


## Footnotes
[^1]: By default, a View Component's `Layout`{:razor} is null. Consequently, `section`{:razor k} blocks will never be rendered, nor throw exceptions. While this can be overridden, it's outside of the scope of this post. It will be covered more in-depth in a future submission.
[^2]: Specifically `OnActionExecuting()`{:cs m token} and `OnActionExecuted()`{:cs m token}. As will be covered later in this series, it is still possible to leverage *some* filters.
[^3]: This overload is technically not part of Microsoft.AspNetCore.Mvc.IViewComponentHelper, but an extension method provided by Microsoft.AspNetCore.Mvc.Rendering.ViewComponentHelperExtensions.

*[Future You]: In my experience, usually Next-Monday You.
