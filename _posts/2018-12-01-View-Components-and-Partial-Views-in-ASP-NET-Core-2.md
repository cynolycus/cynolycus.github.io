---
title: "View Components and Partial Views in ASP.NET Core: Constructing View Components"
category: Practical-ASP-NET-Core
order: 2
date: 2018-12-01 00:01:00
last_modified_at: 2018-12-22
excerpt: Here we'll touch on how to create a simple View Component and return a Partial View (or not!)
description: Here we'll touch on how to create a simple View Component and return a Partial View (or not!)
applies_to: ASP.NET Core
applies_to_vers: [1.0,1.1,2.0,2.1]
tags: [CSharp, ASP.NET Core,Partial Views, Razor Pages, View Components, MVC]
---
{% include kramdown/ALDs.md %}
{% include post/get-prev.md link_title="What are View Components?" %}
{% include post/get-next.md link_title="Using a View Component" %}

---
* TOC
{:toc}
---

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

Continue to {{next_post}}
Return to {{prev_post}}
*[Future You]: In my experience, usually Next-Monday You.
