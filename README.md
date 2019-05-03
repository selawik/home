
![The Selawik Project banner](https://repository-images.githubusercontent.com/184154311/b977ca00-6aae-11e9-8c52-4e50b5a17952)

[![Our discord server link](https://discordapp.com/api/guilds/547491934055170055/widget.png?style=shield)](https://discord.gg/vZTVBxp) 

Welcome to the Selawik project. This repository is the starting point for developers interested in Selawik and also the home for general project documentation and specifications.

Selawik is a research/experiment platform for developing reliable, scalable, and fast applications. We want to innovate across the entire software stack — from the programming languages and compilers, to services and applications, to the OS itself.  This project in particular has three major components that we will develop and design in tandem, so that we can apply all of our innovations up and down our chain of software.

We are heavily inspired by projects like [Singularity](https://www.microsoft.com/en-us/research/project/singularity/) and [Midori](http://joeduffyblog.com/2015/11/03/blogging-about-midori/). Like these projects, we love .NET and C# — but want to radically depart from them in the name of reliability, scalability, and speed. We want to integrate many novel and functional ideas in a way that is immediately familiar to C# and C++ developers.

The three major components of the Selawik project are:

- *The Selawik OS.* Selawik is an operating system that implements the Selawik platform and its principles natively. While the microkernel and other very low-level components will be written in languages like C++, all user-mode components and programs will be written in S# and will only run programs built on the Selawik platform. 
- *The S# programming language.* S# is the primary programming language of the Selawik platform. We want to take C# and add the learnings of the Midori project (e.g. permission and ownership and a modern error model) and various powerful language features (e.g. higher kinded polymorphism, type classes, and unions/intersections).  
- *The Selawik platform.* The Selawik platform specifies behavior and features that a Selawik environment must implement. The specification will define and explains the functions of the Selawik Intermediate Representation and the base class library. This way, runtimes can be flexible in their implementation but also consistent with each other.

Here are some of our key ideas that we are spreading throughout the Selawik stack: 
- Using many small, software-isolated processes connected through asynchronous message passing
- Fast, non-blocking, and zero-copy I/O, accomplished with async/await everywhere and efficient stream APIs
- Permission and ownership annotations for safe parallelism, side effects understanding, and no data races
- Separating bugs from recoverable errors with design by contract, non-nullable types, and typed exceptions
- 100% memory safe — garbage collected memory, overflow checking by default, and bounds-checked arrays
- Eliminating ambient authority with distributed, capability-based security and no mutable statics 
- Aggressive compile-time function execution, check elimination, and sharing of statically 'prewritten' memory
- Language features and rich compiler tooling for code transformations, generation, and analysis

## Specification
General documentation and Selawik specifications live here. We're planning to write up a specification for the S# language and for the Selawik platform. Specifying the Selawik platform includes writing documentation for the standard library. Repository-specific documenation will be found in the repository wiki.

Currently we're focusing on writing a 'platform overview' — meaning we will be combining the semantics of the OS, the runtime, and the S# language into one document that describes (in more detail than has been done here) all aspects of the Selawik project. When we think we are adequately finished in describing everything about everything, we will start deciding which pieces will end up where.

If you have a suggestion about a new feature (e.g. language feature or API), you should open an issue in the proper repository (or, if there is none, here). If the idea is approved and would require a spec change, the issue and its development will be transferred here.

If you want to raise a question or issue about the spec's existing content, you can open an issue in this repository and we can discuss it here. Otherwise, for conventional issues (i.e. spelling errors or unclear wording), you can just open a pull request with the issues fixed and we will merge it without hassle.

## Repositories
Since no programming work has started yet, there are no other repositories right now. However, once work starts, a new repository will be created and it will be linked here. We do not expect any work to start for a while, as we are still busy fleshing out important components of our design. In the meantime, you are free to contribute ideas and fix issues in the platform overview document.

## Engaging and Contributing
We welcome contributions and discussion about Selawik. Feel free to ask questions or raise issues on our GitHub repositories. We also have a Discord server that you can join to have quick and more informal conversations. For more information, see our [support guide](https://github.com/selawik/.github/blob/master/SUPPORT.md).

[![Our discord server link](https://discordapp.com/api/guilds/547491934055170055/widget.png?style=banner2)](https://discord.gg/vZTVBxp)

If you are planning to contribute code, plase read through our [contributing guidelines](https://github.com/selawik/.github/blob/master/CONTRIBUTING.md) so that it'll adhere with our code style/practices.

This project has adopted the code of conduct defined by the Contributor Covenant to clarify expected behavior in our community. For more information, see our [full code of conduct](https://github.com/selawik/.github/blob/master/CODE_OF_CONDUCT.md).

## Licensing
The Selawik project is free software. Most code repositories will be licensed under the GPL, but often linking will be allowed under the license of your choice. Make sure you read the full license in the repository for full information. Any substantial documentation, such as this repository, is licensed under CC-BY 4.0.
