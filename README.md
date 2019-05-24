![The Selawik Project banner](https://repository-images.githubusercontent.com/184154311/34c05900-7d99-11e9-92b1-c7834ce38096)

[![Our discord server link](https://discordapp.com/api/guilds/547491934055170055/widget.png?style=shield)](https://discord.gg/vZTVBxp) 

Welcome to the Selawik project. This repository is the starting point for developers interested in Selawik and also the home for general project documentation and specifications.

Selawik is an all-new research/experiment platform for developing reliable, scalable, and fast applications. We want to spread new ideas across the entire software stack — from the programming languages and compilers, to services and applications, to the underlying OS itself. We're building our platform from scratch so that we can have complete control and face no limits implementing our ideas for Selawik.

This project is heavily inspired by earlier works like [Singularity](https://www.microsoft.com/en-us/research/project/singularity/) and [Midori](http://joeduffyblog.com/2015/11/03/blogging-about-midori/). Like these projects, we love .NET and C# — but want to radically depart from them in the name of reliability, scalability, and speed. We want to integrate many novel and functional ideas in a way that is immediately familiar to C# and C++ developers. The Selawik platform and the S# language should be the lowest level tools you will ever need.

Selawik has three primary components that we are codeveloping together so we can spread our work up and down the stack:

- **The Selawik platform.** The Selawik platform defines the behavior of a conforming runtime environment for 'managed' (Selawik platform based) programs. The specification will describe the Selawik Instruction Tree (SIT) assembly format, runtime behavior for services like capabilities and message passing, and standard library APIs. Conforming runtimes can share consistent behavior while also having flexibility in implementation detail.
- **The Selawik OS.** Selawik is an operating system that is implemented in terms the Selawik platform and its concepts, like capabilities and async activities. It will be built to provide the fastest and most integrated experience for Selawik development and managed programs.  While low level components (e.g. the microkernel) will be written in 'native' languages, user-mode components like drivers and services will be strictly managed code (like S#).
- **The S# programming language.** S# is the flagship programming language of the Selawik platform. Starting with C# as a base, it incorporates many powerful language features (type classes, higher kinded polymorphism, union/intersection types, and extensive pattern matching) with the learnings of the Midori project (permission and ownership, a modern error model, async/await everywhere, and capability-based security).

Other work will be done as well, like support for other operating systems, other programming languages, and various libraries and tools for Selawik.

Here are some of our key ideas that we are spreading throughout the Selawik stack: 
- Using many small, software-isolated processes connected through asynchronous message passing
- Fast, non-blocking, and zero-copy I/O, accomplished with async/await everywhere and efficient stream APIs
- Permission and ownership annotations for safe parallelism, side effects understanding, and no data races
- Separating bugs from recoverable errors with design by contract, non-nullable types, and typed exceptions
- 100% memory safe — garbage collected memory, overflow checking by default, and bounds-checked arrays
- Eliminating ambient authority with distributed, capability-based security and no mutable statics 
- Fast, optimized, native code — inter-assembly proof analysis and aggressive compile-time function execution 
- Language features and rich compiler tooling for code transformations, generation, and analysis

## Specification
General documentation and Selawik specifications live here. We're planning to write up a specification for the S# language and for the Selawik platform. Specifying the Selawik platform includes writing documentation for the standard library. Repository-specific documentation can be found in the GitHub wiki on that repository.

Currently, we're focusing on writing a 'platform spec' that describes all aspects of the project — the OS, runtime, and S# — all in one document. When we think we've adequately described everything about everything, we will start deciding what to pull out of the platform spec and into their respective documents.

If you have an idea about a new feature (e.g., language feature or API) which would require a spec change, open a discussion issue here with the 'Feature suggestion' template. Once there is a general idea of what the feature is about, we will either approve or deny it "in principle" before the specific design of the feature is fleshed out. Bug reports should go in their respective repositories.

If you want to raise a question or issue about the spec's existing content, you can open an issue in this repository with the 'Spec issue' template and we'll try and resolve it. Otherwise, for conventional issues (i.e., spelling or grammatical wording), you can just open a pull request with the issues fixed and we will merge it without hassle.

## Repositories
Since no programming work has started yet, there are no other repositories right now. However, once work starts, a new repository will be created and it will be linked here. We do not expect any work to start for a while, as we are still busy fleshing out important components of our design. In the meantime, you are free to contribute ideas and fix issues in the platform spec.

## Engaging and Contributing
We welcome contributions and discussion about Selawik. Feel free to ask questions or raise issues on our GitHub repositories. We also have a Discord server that you can join to have quick and more informal conversations. For more information, see our [support guide](https://github.com/selawik/.github/blob/master/SUPPORT.md).

[![Our discord server link](https://discordapp.com/api/guilds/547491934055170055/widget.png?style=banner2)](https://discord.gg/vZTVBxp)

If you are planning to contribute code, please read through our [contributing guidelines](https://github.com/selawik/.github/blob/master/CONTRIBUTING.md) so that it'll adhere with our code style/practices.

This project has adopted the code of conduct defined by the Contributor Covenant to clarify expected behavior in our community. For more information, see our [full code of conduct](https://github.com/selawik/.github/blob/master/CODE_OF_CONDUCT.md).

## Licensing
The Selawik project is free software. Most code repositories will be licensed under the GPL, but often linking will be allowed under the license of your choice. Make sure you read the full license in the repository for full information. Any substantial documentation, such as this repository, is licensed under CC-BY 4.0.
