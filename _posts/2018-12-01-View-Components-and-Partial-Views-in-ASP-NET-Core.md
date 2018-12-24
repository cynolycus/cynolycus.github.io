---
title: "View Components and Partial Views in ASP.NET Core"
category: Practical-ASP-NET-Core
order: 1
date: 2018-12-01T00:00:00-5:00
last_modified_at: 2018-12-22
excerpt: Modularize all the things! New to ASP.NET Core, View Components are a great tool for modularizing your MVC or Razor Pages site and empowering your Partial Views.
description: New to ASP.NET Core, View Components are a great tool for modularizing your MVC or Razor Pages site and empowering your Partial Views.
applies_to: ASP.NET Core
applies_to_vers: [1.0,1.1,2.0,2.1]
tags: [CSharp, ASP.NET Core, Partial Views, Razor Pages, View Components, MVC]

---
{% include kramdown/ALDs.md %}
---
* TOC
{:toc}
---
## What are View Components?
Introduced with the advent of ASP.NET Core, a View Component is essentially a partial View with business logic, or conversely a miniature Controller. Their versatility makes View Components a great tool for taking a modular approach in designing your MVC application, but they aren't necessarily appropriate for every situation. If you've worked with MVC and partial Views before .NET Core, View Components will seem familiar at best, or redundant at worst. They're very simple in principle, and very powerful if leveraged appropriately.
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
{%- include post/get-next.md link_title="Constructing a View Component" -%}
Continue to {{next_post}}

### Footnotes
[^1]: By default, a View Component's `Layout`{:razor} is null. Consequently, `section`{:razor k} blocks will never be rendered, nor throw exceptions. While this can be overridden, it's outside of the scope of this post. It will be covered more in-depth in a future submission.
[^2]: Specifically `OnActionExecuting()`{:cs m token} and `OnActionExecuted()`{:cs m token}. As will be covered later in this series, it is still possible to leverage *some* filters.

*[Future You]: In my experience, usually Next-Monday You.
