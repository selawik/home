![The Selawik Project banner](https://repository-images.githubusercontent.com/184154311/34c05900-7d99-11e9-92b1-c7834ce38096)

[![Our discord server link](https://discordapp.com/api/guilds/547491934055170055/widget.png?style=shield)](https://discord.gg/vZTVBxp) 

Welcome to the Selawik project. This repository is the starting point for developers interested in Selawik and also the home for general project information.

Selawik is an all-new research/experiment platform for developing reliable, scalable, and verifiable applications, heavily inspired by earlier works like [Singularity](https://www.microsoft.com/en-us/research/project/singularity/) and [Midori](http://joeduffyblog.com/2015/11/03/blogging-about-midori/). We want to spread new ideas across the entire software stack — from the programming languages and compilers, to services and applications, to the underlying OS itself. We're building our platform from scratch so that we can have complete control and face no limits implementing our ideas for Selawik. The Selawik platform and language should be the lowest level tools you will ever need.

The project is split into two primary components which are tightly codeveloped:

- Selawik's programming language starts with C# as a base, but radically departs in the name of reliability, scalability, and verifiability. It incorporates many powerful language features (type classes, higher kinded polymorphism, union/intersection types, and extensive pattern matching) with the learnings of the Midori project (permission and ownership, a modern error model, async/await everywhere, and capability-based security).
- As part of Selawik there will also be a proof of concept operating system which is implemented using Selawik's concepts, like capabilities, fine-grained processes, and async activities. It will be built to provide the fastest and most integrated experience for Selawik development and managed programs.  While low level components (e.g. the microkernel) will be written in C, user-mode components like drivers and services will be strictly managed code.

Here are some of our key ideas that we are spreading throughout the Selawik stack: 

- Using many small, software-isolated processes connected through asynchronous message passing
- Fast, non-blocking, and zero-copy I/O, accomplished with async/await everywhere and efficient stream APIs
- Permission and ownership annotations for safe parallelism, side effects understanding, and no data races
- Separating bugs from recoverable errors with design by contract, non-nullable types, and typed exceptions
- 100% memory safe — garbage collected memory, overflow checking by default, and bounds-checked arrays
- Eliminating ambient authority with distributed, capability-based security and no mutable statics 
- Fast, optimized, native code — global code analysis and aggressive compile-time function execution 
- Language features and rich compiler tooling for code transformations, generation, and analysis

## Engaging and Contributing

We welcome contributions and discussion about Selawik. Feel free to ask questions or raise issues on our GitHub repositories. We also have a Discord server that you can join to have quick and more informal conversations. For more information, see our [support guide](https://github.com/selawik/.github/blob/master/SUPPORT.md).

[![Our discord server link](https://discordapp.com/api/guilds/547491934055170055/widget.png?style=banner2)](https://discord.gg/vZTVBxp)

If you are planning to contribute code, please read through our [contributing guidelines](https://github.com/selawik/.github/blob/master/CONTRIBUTING.md) so that it'll adhere with our code style/practices.

This project has adopted the code of conduct defined by the Contributor Covenant to clarify expected behavior in our community. For more information, see our [full code of conduct](https://github.com/selawik/.github/blob/master/CODE_OF_CONDUCT.md).

## Licensing
The Selawik project is free software. Most code repositories will be licensed under the GPL, but often linking will be allowed under the license of your choice. Make sure you read the full license in the repository for full information. Any substantial documentation, such as this repository, is licensed under CC-BY 4.0.
