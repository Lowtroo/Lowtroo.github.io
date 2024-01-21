+++
title = "Something related to .Net"
draft = false
date = "2024-01-20"
+++

## .Net Framework
It's Microsoft .Net Framework, which is a **proprietary** and **original implementation** of .Net. It can only run applications on Windows. .Net Framework is based on **Common Language Runtime(CLR)**  and **Base Class Libraries(BCL)** and **Common Intermediate Language(CIL)**.

## .Net Core 
.Net Core is also an **implementation** of .Net. However, it can run **multiple platforms**, and it's **open-sourced** maintained by Microsoft and community with code hosted on Github. It supports some modern application framework such as ML.NET, ASP.NET Core Razor Pages, Blazor, UWP etc. .Net Core is based on **CoreCLR** and **CoreFX**. CoreCLR is a runtime for .NetCore including JIT, GC and several low-level classes. CoreFX consists of several foundation class libraries. The two metioned could respectively deemed as alternatives to CLR and BCL of .NET Framework to some extent. 

## .Net Standard 
There could be multiple **implementations** of .Net, and third-party **libraries** normally aim to be run on all of these diffefent implementions. So providing a **specification** for developing libraries is warranted. .NET Standard supports for Mono platform, Xamarin, Universal Windows Platform, and Unity.

