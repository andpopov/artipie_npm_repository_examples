# [Deployment of NPM Repositories with Artipie](https://dev.to/andpopov/deployment-of-npm-repositories-with-artipie-30co)

In this article I, am going to demonstrate how to deploy your own NPM-repository server by using Artipie.

I will show how to configure and start a new NPM repository, how to use the standard NPM tool for publishing an NPM package to the Artipie NPM repository, and how to install an NPM package after obtaining it from the Artipie NPM repository.

## Introduction of Artipie
[Artipie](https://github.com/artipie) is a free open-source binary artifact management tool project under an MIT license.

Artipie is a rapidly growing project that was born in 2020. It currently supports a large number of repository types:
* NPM - for storing and sharing Node JS packages.
* Docker - Docker registry for images.
* RPM – a repository of .rpm files for RHEL, CentOs, Fedora, PCLinuxOS, AlmaLinux, openSUSE, OpenMandriva, Oracle Linux, etc.
* Debian - repository packages for Debian-based Linux distros (Debian, Mint, Ubuntu, MX Linux, Raspberry Pi OS, Parrot OS, etc).
* Go – for storing Go packages.
* Maven - Java, Kotlin, Groovy, Scala, Clojure artifacts of types such as .jar, .war, .klib, etc.
* PyPI - Python packages index.
* Anaconda - packages for data science for Python, R, Lua, C, C++, and other languages.
* HexPM - for storing and sharing packages for Elixir and Erlang languages.
* Gem – a hosting service of RubyGem for Ruby language.
* Helm - Helm charts repository.
* NuGet - .NET package hosting service.
* Composer - PHP source packages.
* Binary (files) storage - for hosting any type of file.

Artipie is flexible enough to support custom configurations, so you can use it like a “Lego constructor” by organizing your own repository or multiple repositories for storing your artifacts.

Artipie consists of the following main parts:
* Storage - keeps artifacts in some source of data, for example, it can store artifacts in a file system or in Amazon S3.
* Repository - understands the format of some types of artifacts and organizes work with these types of artifacts. For example, there is an NPM repository type that can work with NPM packages and NPM tools.
* Artipe engine - server of Artipie.
* Artipie frontend - provides a web-based dashboard to control repository configurations.
* Artipie REST API – provides REST services and Swagger UI to control all aspects of Artipie.

Atripie supports the following types of storage:
* File system storage
* Amazon S3 storage
* Redis storage
* Custom storage type

The Artipie engine is designed as a binary artifact’s storage management system for high loads. Artipie engine design follows the principles of a reactive approach that supposes the usage of asynchronous files and network operations.

Artipie provides access control means through users and groups, and permissions can be assigned to users and groups for specific resources and operations.

Artipie provides two kinds of repository layouts:
* **flat**, where all artifacts lay in one repository
* **org**, where artifacts are organized as a set of separate user repositories.

To get more information about Artipie please visit the [github](https://github.com/artipie/artipie) and [wiki](https://github.com/artipie/artipie/wiki) pages.

## Preparation of NPM Repository

Artipie is a java application and there are two ways to launch it:
* As a Java application from a JAR file
* As a container in Docker Engine

In this article, I use Docker Engine as a deployment environment for Artipie NPM-repository.

Therefore, you should ensure that [Docker Engine](https://docs.docker.com/get-docker/) has already been installed on your workstation.

I use the Windows 10 operating system on my workstation, but you can use either Unix/Linux/MacOs operating system that supports Docker Engine.

I am going to store all the configuration files and NPM packages inside the folder `C:\artipie`.

Next, I store two configuration files in the `C:\artipie` folder: one for the Artipie engine and the second one for the NPM repository.

The Artipie engine configuration file is located at the path: `C:\artipie\config\artipie.yml` and defines the following parameters:
* **type: fs**
  The storage type, where `fs` indicates the use of the file storage type for storing artifacts in the file system.
* **path: /var/artipie/repo**
  Specifies the path to the directory that stores configurations of all repositories, including the NPM repository configuration mentioned in this article.
* **layout: flat**
  The artifact’s layout definition, ‘flat’ means to store all artifacts in one directory of storage.

Listing of `C:\artipie\config\artipie.yml`:

```
meta:
  storage:
    type: fs
    path: /var/artipie/repo #path to repository configurations
  layout: flat
```

The NPM repository configuration file is located at the path: `C:\artipie\repo\my-npm.yaml` and it defines the following parameters:
* **type: npm**
  The type of repository here is `npm` for the deployment of the NPM repository.
* **url: http://localhost:8080/my-npm**
  The URL of the repository. It indicates the HTTP endpoint to access the NPM repository by using the `npm` command tool.
* **path: /var/artipie/packages**
  Specifies the path where the published NPM packages should be stored.
* **permissions:**
  Configures access permissions on the NPM repository’s artifacts, including “democratic” permissions that allow everyone to download and publish NPM packages.

Listing of `C:\artipie\repo\my-npm.yaml`:

```
repo:
  type: npm
  url: http://localhost:8080/my-npm
  storage:
    type: fs
    path: /var/artipie/packages
  permissions:
    "*":
      - download
      - publish
```

Resulting file-tree view of configuration files and directories on my local workstation:

```
C:\artipie
  config/
      artipie.yml
  repo/
      my-npm.yaml
  packages/
```

Now we are ready to launch Artipie as a container of the Docker Engine.

Run following command in the terminal:

```
docker run \
-v C:\artipie\config:/etc/artipie/ \
-v C:\artipie\repo:/var/artipie/repo \
-v C:\artipie\packages:/var/artipie/packages \
-p 8081:8080
artipie/artipie:latest
```

The command starts the Artipie engine as a container application inside Docker Engine. The command mounts 3 local directories to the Docker container as volumes, and forwards the local port 8081 to the container’s port 8080:

1. The Artipie engine expects to find its main configuration file inside the `/etc/artipie/` directory. Therefore we mount the local directory `C:\artipie\config` to `/etc/artipie/` container’s directory.
2. The Artipie engine looks up repository configurations inside the directory `/var/artipie/repo` so we mount the local directory `C:\artipie\repo` to it.
3. Published NPM packages are stored in the directory `/var/artipie/packages`. Therefore, we mount the local directory `C:\artipie\packages` to it.

## Publishing of package

Now we have started the Artipie engine and deployed an NPM repository. Let’s create a new NPM package and publish it to the NPM repository.

I created a NodeJS package folder: `C:\workdir\@hello\simple-npm-project\` and stored two files of the package: `index.js` and `package.json`.

Listing of the `index.js`:

```
console.log("Hello world");
```

Listing of `package.json`:
```
{
  "name": "@hello/simple-npm-project",
  "version": "1.0.1",
  "description": "",
  "main": "index.js",
  "scripts": {
    "test": "echo \"Error: no test specified\" && exit 1"
  },
  "author": "",
  "license": "ISC"
}
```

Now go to the `C:\ workdir` folder and run following command in the terminal:
```
npm publish @hello\simple-npm-project --registry http://localhost:8081/my-npm
```
The parameter `--registry` is required to specify Artipie’s NPM repository.

Using the preceding command publishes the ‘@hello/simple-npm-project’ NPM package to Artipie’s NPM repository.

The published NodeJS package can be found inside the folder `C:\artipie\packages`.

## Installing the package
Let’s install the package ‘@hello\simple-npm-project’ by using the standard command ‘npm install’:
```
npm install @hello/simple-npm-project --registry http://localhost:8081/my-npm
```
The parameter `--registry` is required for specifying Artipie’s NPM repository.

The preceding command downloads the NPM package from the Artipie NPM repository and installs a working project.

# Conclusion
The article demonstrates the simplicity and flexibility usage of using Artipie to deploy a custom configuration of the NPM repository and the interaction with the deployed NPM repository by using the standard `npm` tool.

Files used in the article are available [here](https://github.com/andpopov/artipie_npm_repository_examples).
