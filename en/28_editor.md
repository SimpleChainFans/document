### Remix

*We recommend that you use Remix to develop simple contracts and quickly learn Solidity*

[Remix](https://remix.ethereum.org/) it can be used online without installing anything. If you want to use it offline, press https://github.com/ethereum/browser-solidity/tree/gh-pages download the zip file to use. This page provides further details on how to install the Solidity command line compiler on your computer. If you are just dealing with large contracts or need more compilation options, you should choose to use the command line compiler solc.

### npm / Node.js


Use `npm` can easily install the Solidity compiler solcjs. But `solcjs` function of the program is less than all other options below this page. In `commandline-compiler` chapter, we assume that you are using a full-featured compiler. So if you are from `npm` 
installation `solcjs` ，jump directly [solc-js](https://github.com/ethereum/solc-js) understand .


Note: The solc-js project uses Emscripten to compile from the solc version of the C version to JavaScript across platforms. Therefore, solcjs (like Remix) can be used in JavaScript projects.
For more information, see the solc-js code library.

```bash
npm install -g solc
```
>  In the command, use `solcjs` not `solc` .`solcjs` command options are the same `solc` and some tools（such `geth` )is incompatible, so don't expect `solcjs` energy image`solc` work the same.

### Docker

We provide the latest docker build for the compiler.``stable`` published version is in the warehouse, ``nightly``repository is a version with unstable changes in the development branch.

```bash
docker run ethereum/solc:stable solc --version
```

Currently, docker images only contain solc executable programs, so you need extra work to connect the source code with the output directory.

### Binary package


Can be in [solidity/releases](https://github.com/ethereum/solidity/releases) download `Solidity` binary installation package.

For Ubuntu, we also provide PPAs. You can obtain the latest stable version by using the following command:

```bash
sudo add-apt-repository ppa:ethereum/ethereum
sudo apt-get update
sudo apt-get install solc
```

Of course, you can also install an early version of the developer:

```bash
sudo add-apt-repository ppa:ethereum/ethereum
sudo add-apt-repository ppa:ethereum/ethereum-dev
sudo apt-get update
sudo apt-get install solc
```

At the same time, it also provides installable [All supported Linux versions](https://snapcraft.io/docs/core/install) downloaded [snap package](https://snapcraft.io/) , You can obtain the latest stable version by using the following command:

```bash
sudo snap install solc
```

Or, if you want to test the latest changes under the develop branch, you can install the developer version as follows:

```bash
sudo snap install solc --edge
```

Similarly, `Arch Linux` Installation packages are also available, but only for the latest developer version:

```bash
pacman -S solidity
```

At the time of writing this article, Homebrew did not provide a pre-built binary package (because we migrated from Jenkins to TravisCI). We will provide the binary installation package under homebrew as soon as possible, but at least it works from the source code:

```bash
brew update
brew upgrade
brew tap ethereum/ethereum
brew install solidity
```

If you need a specific version Solidity , you need to start from Github Install one on Homebrew formula, You can consult
[solidity.rb commits on Github](https://github.com/ethereum/homebrew-ethereum/commits/master/solidity.rb) submitted records, to find contains ``solidity.rb`` special submission of file changes. Then use `brew` install:

```bash
brew unlink solidity
# Install 0.4.8
brew install https://raw.githubusercontent.com/ethereum/homebrew-ethereum/77cce03da9f289e5a3ffe579840d3c5dc0a62717/solidity.rb
```

`Gentoo Linux` the installation package is also provided below, which can be used `emerge` Install:

```bash
emerge dev-lang/solidity
```

### Compile from source code

#### Clone code library

Run the following command to clone the source code:

```bash
git clone --recursive https://github.com/ethereum/solidity.git
cd solidity
```

If you want to participate Solidity Development of, you can fork Solidity After the source code library, use your personal Fork Library as the second remote source:

```bash
cd solidity
git remote add personal git@github.com:[username]/solidity.git
```

`Solidity` With `Git` Sub-modules, make sure they are fully loaded:

```bash
git submodule update --init --recursive
```

#### Prerequisite - macOS

In macOS, make sure that the latest version is installed
[Xcode](https://developer.apple.com/xcode/download/)，
Xcode contains [Clang C++ compile](https://en.wikipedia.org/wiki/Clang)， and
[Xcode IDE](https://en.wikipedia.org/wiki/Xcode) and other Apple development tools are `OSX` Compile `C++` required for the application. If you have installed Xcode for the first time or just updated the new version of Xcode, you must agree to the use protocol of Xcode before using the command line to build:
```bash
sudo xcodebuild -license accept
```

Solidity is built under OS X. [Install Homebrew](http://brew.sh)
Package Manager to install dependencies.
If you want to start from scratch, here is[Method for uninstalling Homebrew](https://github.com/Homebrew/homebrew/blob/master/share/doc/homebrew/FAQ.md#how-do-i-uninstall-homebrew>)


#### Prerequisites - Windows

In `Windows` build under `Solidity` , the dependency software package to be downloaded:

Software | Note
-|-
|`Git for Windows`_  | C command line tool for obtaining source code from Github |
| `CMake`_   | Build a file generator across platforms      |
| `Visual Studio 2017 Build Tools`_ | C compiler    |
| `Visual Studio 2017`_  (Optional) | C++ Compiler and development environment |


If you already have `IDE`，only the compiler and related libraries are required, you can install `Visual Studio 2017 Build Tools`。

`Visual Studio 2017` provider `IDE` and necessary compilers and libraries. So if you don't have one `IDE` and want to develop `Solidity`，then `Visual Studio 2017` it will be a simple choice that allows you to get all the tools.

Here is one in `Visual Studio 2017 Build Tools` or `Visual Studio 2017` list of components to be installed in:

* Visual Studio C++ core features
* VC++ 2017 v141 toolset (x86,x64)
* Windows Universal CRT SDK
* Windows 8.1 SDK
* C++/CLI support

#### External dependency

On macOS, Windows, and other Linux distributions, there is a script that can install the required external dependencies with one click. It was originally a multi-step operation that requires manual participation, but now it only needs one line of command:

```bash
./scripts/install_deps.sh
```

**Run in Windows:**

```bash
scripts\install_deps.bat
```

#### Command line construction

**Make sure you have installed external dependencies (see above)**

Solidity uses CMake to configure the build. Linux, macOS, and other Unix systems are built in the same way:

```bash
mkdir build
cd build
cmake .. && make
```

There are also simpler ones:

```bash
#note: 将安装 solc 和 soltest 到 usr/local/bin 目录
./scripts/build.sh
```

For Windows:

```bash
mkdir build
cd build
cmake -G "Visual Studio 15 2017 Win64" ..
```

The last sentence of this set of instructions will create a **solidity.sln** file. After double-clicking, Visual Studio is opened by default. We recommend that you create on **RelWithDebugInfo** configuration file.

Or use the command to create:

```bash
cmake --build . --config RelWithDebInfo
```

### CMake parameter

If you are interested in the CMake command option, you can execute it ``cmake .. -LH`` view

Detailed description of version number string

The Solidity version name consists of four parts:

- Version number
- Pre-release version, usually``develop.YYYY.MM.DD`` or ``nightly.YYYY.MM.DD``
- To ``commit.GITHASH`` submission number displayed in the format.
- Platform identification consisting of several platforms and compiler details

If there is a local modification, the commit part has a suffix ``.mod``。

These parts are combined according to the requirements of Semver. The pre-release version number of Solidity is equivalent to the pre-release version number of Semver. The Solidity submission number and platform identification form the build metadata of Semver.

Release sample: ``0.4.8+commit.60cc1668.Emscripten.clang``.

Pre-release layout sample: ``0.4.9-nightly.2017.1.17+commit.6ecb4aa3.Emscripten.clang``

### Version information details

After the version is released, the patch version will increase because we assume that only patch-level changes will occur later. When changes are merged, the version should be adjusted according to semver and the intensity of the changes. Finally, the release version is always consistent with the version number of the current daily build version, but does not `prerelease` Indicator.

for example:

0. 0.4.0 release Version
1. From now on, build version 0.4.1 every night
2. Introduce non-destructive changes-do not change the version number
3. Introduce disruptive changes-version jump to 0.5.0
4. Version 0.5.0 released

This method and `version pragma` Run well together.






