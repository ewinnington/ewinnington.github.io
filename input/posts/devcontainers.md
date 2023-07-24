Title: DevContainers - The future of developer environments
Published: 24/07/2023
Tags: [DevContainers, Docker, VSCode, Github, Codespaces, Architecture, Dev] 
---

## History

It's been years now that we've had Infrastructure as Code (IaC), Containers and Desired state Configuration (DsC) tools to do our deployments. But these have been mostly focused on the deployment side of things, with fewer tools on the developer side. On the dev machine, installing and maintaining the development tools and package dependencies has been in flux, both in windows where finally tools like Ninite, Chocolatey and Winget allow management of dev tools, and on the linux side, which was always quite well served with apt - but has also gained Snap, Flatpack and other package management tools. The thing is, sometimes you need more that one version of a particular tool, Python3.10 and Python3.11, Java9 and Java17, Dotnet 4.8 and Dotnet 6, to work on the various projects you have during the day. Sometimes, they work side by side very well and sometimes they don't. And when they don't, it can be a long process to figure out why and also very difficult to get help without resorting to having a clean image refresh and starting again to install your dependencies.

Since the end of the 2010s and the early 2020s, with the rise of web hosted IDEs, there has been a need to define ways to have a base image that contained the environment and tools needed to work. I remember running some in the mid 2010s - Nitrous.IO (2013-16) - that allowed you to use a base container and configure it to do remote development.

## DevContainers
With the arrival of Docker on every desktop, Github's Cloudspaces and Visual Studio Code, there's been a new interest in this type of desired state environments with developer tooling. Microsoft published the [DevContainer specification](https://containers.dev/) in early 2022 to formalize the language.

So how does it help us? Well, with a DevContainer, we can setup a new development environment on Premise (in VSCode), on the cloud VM (Azure+VM) or on a Codespace environment with a single file that ensures that we always have the tools we want and need installed. Starting to work is as easy as openining the connection and cloning the repo we need if the .devcontainer file is located inside.

## DevContainer example
You can find below my [personal DevContainer](https://github.com/ewinnington/DevContainerTemplate/blob/master/.devcontainer/devcontainer.json), it is setup with Git, Node, AzureCLI, Docker control of hose, Dotnet, Terraform, Java with Maven, Python3 and Postgresql. I also have the VSCode extensions directly configured so I can directly start using them when I connect. I also use the "postStartCommand": "nohup bash -c 'postgres &'" to run an instance of Postgresql directly inside the development container, so I can a directly have a DB to run requests against. And yes, this is a bit of a kitchen sink DevContainer, they can be smaller and more tailored to a project with only one or two of these features included, but here I use a generic one add added everything I use apart from the c++ and fortran compilers.

```
{
    "name": "Erics-base-dev-container",
    "image": "mcr.microsoft.com/devcontainers/base:debian",
 
    "features": {
        "ghcr.io/devcontainers/features/git:1": {},
        "ghcr.io/devcontainers/features/node:1": {},
        "ghcr.io/devcontainers/features/azure-cli:1": {}, //azure-cli,
        "ghcr.io/devcontainers/features/docker-outside-of-docker:1": {}, //docker on host
        "ghcr.io/devcontainers/features/dotnet:1": {}, //dotnet installed
        "ghcr.io/devcontainers/features/terraform:1": {},
        "ghcr.io/devcontainers/features/java:1": { "installMaven" : true },
        "ghcr.io/devcontainers-contrib/features/postgres-asdf:1": {}
    },
 
    // Configure tool-specific properties.
    "customizations": {
        // Configure properties specific to VS Code.
        "vscode": {
            "settings": {},
            "extensions": [
                "streetsidesoftware.code-spell-checker",
                "ms-azuretools.vscode-docker",
                "ms-dotnettools.csharp",
                "HashiCorp.terraform",
                "ms-azuretools.vscode-azureterraform",
                "GitHub.copilot",
                "GitHub.copilot-chat",
                "vscjava.vscode-java-pack",
                "ms-python.python"
            ]
        }
    },
 
    // Use 'forwardPorts' to make a list of ports inside the container available locally.
    // "forwardPorts": [3000],
 
    // Use 'portsAttributes' to set default properties for specific forwarded ports.
    // More info: https://containers.dev/implementors/json_reference/#port-attributes
    "portsAttributes": {
        "3000": {
            "label": "Hello Remote World",
            "onAutoForward": "notify"
        }
    },
 
    // Use 'postCreateCommand' to run commands after the container is created.
    "postCreateCommand": "",
 
    "postStartCommand": "nohup bash -c 'postgres &'"
 
    // Uncomment to connect as root instead. More info: https://aka.ms/dev-containers-non-root.
    // "remoteUser": "root"
}
```

## So how do you start with DevContainers?
There are 2 easy ways:

1) (remote) Github Codespaces
By going to my repo, you can click "Create Codespace on Master" and get a running VSCode in the cloud with all those tools setup instantly.

(at first build, the image might take time)

2) (local) Docker + VS Code
Ensure you have the [ms-vscode-remote.remote-containers](https://marketplace.visualstudio.com/items?itemName=ms-vscode-remote.remote-containers) extension installed in VS Code and Docker installed.

Clone the repo https://github.com/ewinnington/DevContainerTemplate.git, then open it with VSCode. It should automatically detect the .devContainer and offer to rebuild the container image and open it up in the IDE for you.



Once that is done, you should have access to a complete environment at the state you specified.

## What's the use for Developers at corporations where computers are locked down?

I think that providing developer windows machine with Git, Docker, WSL2 installed and using VS Code or another IDE that supports DevContainers is an excellent way forwards in providing a good fast and stable environment for developers to work faster and more efficiently. Using this configuration, any person showing up to a Hackathon would be able to start working in minutes after cloning a repository. It would really simplify daily operations, since every repo can provide the correct .DevContainer configuration, or teams can share a DevContainer basic configuration.

This all simplifies operations, makes developer experience more consistent and increases productivity since you can move faster from one development environment to another in minutes. OnPrem → Remote VM → Cloudspace and back in minutes, without any friction.

All in all, I'm convinced it is a tool that both IT support must understand and master how to best provide access to, and for developers to understand the devContainer to benefit from it.


Have you used DevContainers? What is your experience?
