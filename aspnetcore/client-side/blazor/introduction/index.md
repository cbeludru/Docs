---
title: Introduction to Blazor
author: guardrex
description: Learn how Blazor runs in the browser to execute C#/Razor code with WebAssembly and the Mono runtime in this introduction.
manager: wpickett
ms.author: riande
ms.custom: mvc
ms.date: 03/23/2018
ms.prod: asp.net-core
ms.technology: aspnet
ms.topic: article
uid: client-side/blazor/introduction/index
---
# Introduction to Blazor

By [Steve Sanderson](http://blog.stevensanderson.com), [Daniel Roth](https://github.com/danroth27), and [Luke Latham](https://github.com/guardrex)

[!INCLUDE[](~/includes/blazor-preview-notice.md)]

Blazor is a new experimental .NET web framework using C#/Razor and HTML that runs in the browser with [WebAssembly](http://webassembly.org). Blazor provides all of the benefits of a rich single-page application (SPA) platform using .NET on both the server and the client.

## Running .NET in the browser

Running .NET code inside web browsers is made possible by a relatively new technology, WebAssembly (abbreviated *wasm*). WebAssembly is based on open web standards and is supported in web browsers without plugins. WebAssembly is a compact bytecode format optimized for fast download and maximum execution speed.

Security isn't a major concern because WebAssembly isn't ordinary assembly code (for example, x86/x64) &mdash; WebAssembly is a new bytecode format that accesses browser functionality with the same capabilities as JavaScript.

Mono is the official .NET runtime for client platforms, such as native mobile apps and games. Recently, the Mono team added support for WebAssembly.

When a Blazor app is built and run in a browser:

1. C# code files and Razor files are compiled into .NET assemblies.
1. The assemblies and the Mono WebAssembly runtime are downloaded to the browser.
1. Blazor uses JavaScript to bootstrap the .NET runtime and configures the runtime to load the app's assemblies along with any required base class libraries (BCLs). Document object model (DOM) manipulation and browser API calls are handled via JavaScript interoperability (interop).

## Compilation modes

Mono aims run under WebAssembly in two modes: interpreted and ahead-of-time (AOT).

### Interpreted mode

In interpreted mode, the Mono runtime is compiled into WebAssembly bytecode, but the .NET assembly files aren't compiled into bytecode. The browser can load and execute the Mono runtime, which in turn can load and execute standard .NET assemblies (*\*.dll* files) built by the normal .NET compilation toolchain.

![Interpreted mode](index/_static/interpretedmode.png)

Interpreted mode is similar to how the core internals of the desktop Common Language Runtime (CLR) are distributed precompiled into native code, which then loads and executes .NET assembly files. One key difference is that the desktop CLR uses just-in-time (JIT) compilation extensively to make execution faster, while Mono on WebAssembly is closer to a pure interpretation model.

### Ahead-of-time (AOT) compilation mode

In ahead-of-time (AOT) compilation mode, the app's .NET assemblies are transformed into WebAssembly binaries at build time. At runtime, there's no interpretation. The code executes immediately. It's still necessary to load part of the Mono runtime. Features for low-level .NET services are loaded, such as garbage collection. Features for parsing .NET assemblies aren't required and aren't loaded.

![AOT mode](index/_static/aotmode.png)

This is similar to how the ngen tool has historically allowed AOT compilation of .NET binaries into native machine code. More recently, CoreRT provides a complete native AOT .NET runtime for executing .NET binaries.

## Interpreted versus AOT

Interpreted mode provides a faster development cycle than AOT. When code is changed under the interpreted model, the app can be rebuilt and the updated app reloaded in the browser within a few seconds. An AOT rebuild might take several minutes to compile and reload. Because interpreted mode offers faster recompilation, it might become the default mode for compilation during development. More work is required before a conclusion can be drawn on how to best apply interpreted and AOT modes of compilation with WebAssembly.

## Blazor, a SPA framework

Blazor has a growing standard feature set to solve common app requirements, such as UI composition, state management, and routing. Features are designed around the strengths of .NET and the C# language with careful consideration given to tooling support.

Blazor is inspired by today's top SPA frameworks such as React, Vue, and Angular, plus some Microsoft technologies, such as Razor Pages. The goal is to provide SPA features that web developers have found most successful in existing frameworks in a way that capitalizes on .NET's strengths.

## Blazor components

In SPA frameworks, apps are built with *components*. A component usually represents a piece of UI, such as a page, dialog, or data entry form. Components can be nested, reused, and shared between projects.

In Blazor, a component is a .NET class. The class can either be written directly, as a C# class (*\*.cs*), or more commonly in the form of a Razor markup page (*\*.cshtml*).

Razor, which has been around since 2010, is a syntax for combining HTML markup with C# code. Razor is designed for developer productivity, allowing the developer to switch between markup and C# in the same file with IntelliSense support. Here's an example of how to express a simple custom dialog component in a Razor file:

```cshtml
<div>
    <h2>@Title</h2>
    @RenderContent(Body)
    <button onclick=@OnOK>OK</button>
</div>

@functions {
    public string Title { get; set; }
    public Content Body { get; set; }
    public Action OnOK { get; set; }
}
```

When this component is used elsewhere in the app, IntelliSense speeds development with syntax completion and parameter info.

Many design patterns are possible with this simple foundation. This includes patterns recognizable in other SPA frameworks, such as stateful components, functional stateless components, and higher-order components.

Components can be:

* Nested.
* Generated procedurally in code.
* Shared via class libraries.
* Unit tested without requiring a browser DOM.

## Infrastructure

Blazor offers the core facilities that most apps require, including:

* Layouts
* Routing
* Dependency injection
* Lazy loading (loading parts of the app on demand as a user navigates the app)
* Unit testing

As an important design point, all of these features are optional. When one of these features isn't used in an app, the implementation is stripped out of the app when published by the IL linker.

Only a few low-level elements are baked into the framework. For example, routing and layouts aren't built-in. Routing and layouts are implemented in *user space*, code that an app developer can write without using internal APIs. These features can be replaced with different systems to suit the app's requirements. The current layouts prototype is implemented in only about 30 lines of C# code, so a developer can easily understand and replace it if desired.

## Code sharing and .NET Standard

The [.NET Standard](/dotnet/standard/net-standard) is a formal specification of .NET APIs that are intended to be available on all .NET implementations. Mono on WebAssembly supports `netstandard2.0` or higher. .NET Standard class libraries can be shared across backend server code and in browser-based apps.

Browsers support the APIs that developers use to build web apps. Not all .NET APIs make sense in the browser. For example, arbitrary TCP sockets can't be accessed in a browser, so [System.Net.Sockets.TcpListener](/dotnet/api/system.net.sockets.tcplistener) can't perform any useful task. For BCL APIs that don't apply to a given platform, the BCL throws a [PlatformNotSupported](/dotnet/api/system.platformnotsupportedexception) exception.

## JavaScript/TypeScript interop

Sometimes, access to third-party JavaScript libraries and browser APIs is required. WebAssembly is designed to interoperate with JavaScript, so Blazor is capable of using any library or API that JavaScript is able to use.

To call JavaScript libraries or your own custom JavaScript/TypeScript code from .NET, the current approach is to register a named function in a JavaScript/TypeScript file:

```javascript
Blazor.registerFunction('doPrompt', message => {
  return prompt(message);
});
```

Wrap the named function for calls from .NET:

```csharp
public static bool DoPrompt(string message)
{
    return RegisteredFunction.Invoke<bool>("doPrompt", message);
}
```

This approach has the benefit of working with JavaScript build tools, such as [webpack](https://webpack.js.org/).

To save the trouble of registering functions in this way for standard browser APIs, the Mono team is working on a library that exposes standard browser APIs to .NET.

## Optimization

Traditionally, .NET has focused on platforms where the app's binary size isn't a major concern. It doesn't really matter whether a server-side ASP.NET app is 1MB or 50MB. It's only a moderate concern for native desktop or mobile apps. But for browser apps, payload size is critical.

Development efforts are aimed at reducing the download size of the Mono runtime and .NET app assemblies. Here are three phases of size optimization the Blazor engineering team has in mind:

1. Mono runtime stripping

   The Mono runtime contains many desktop-specific features. We hope that the Blazor packages will contain a trimmed version of Mono that is substantially smaller than the full-fat distribution. In an optimization experiment, the Mono runtime was pruned of unnecessary code. Over 70% of the Mono *wasm* file was removed while keeping a basic app working.

1. Publish-time IL stripping

   The .NET IL linker (originally based on the Mono linker) performs static analysis to determine which parts of .NET assemblies can ever get called by an app, then it strips out everything else.

   This is equivalent to *tree shaking* in JavaScript, except the IL linker is much more fine-grained, operating at the level of individual methods. The IL Linker removes all of the system library code that the app isn't using, which often results in a 70% or greater reduction in code size.

1. Compression

   Most web servers support HTTP compression, which typically cuts the remaining payload size by a further 75%.

Overall, a .NET-based browser app is never going to be as tiny as a minimal React app, but the goal is to make it small enough that a typical user with average Internet bandwidth won't notice or care about an app's first load time. After first load, the app's assemblies are fully cached.

## Deployment

A target market for Blazor is ASP.NET developers. For ASP.NET Core apps, [middleware](xref:fundamentals/middleware/index) will offer an easy path to serve a Blazor UI seamlessly from ASP.NET Core. Advanced features, such as server-side prerendering, are possible.

Equally important are developers who don't yet use ASP.NET Core. To make Blazor a viable consideration for developers using Node.js, Rails, PHP, or even for serverless web apps, ASP.NET Core isn't required on the server. When a Blazor app is built, a *dist* directory is produced containing nothing but static files. The contents of the *dist* folder can be hosted on the Azure CDN, GitHub Pages, Node.js servers, and many other servers and services.
