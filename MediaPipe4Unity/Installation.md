# MediaPipe4Unity Installation Guide

> :warning:
>
> - This repository doesn't contain required (e.g. `libmediapipe_c.so`, `Google.Protobuf.dll`, etc), so you need to build them first.
> - Libraries that can be built differ depending on your environment.
> - In `maton.net`, we don't build for *Android* or *iOS* because of the lack of compability

## Supported Platforms

> :warning:
>
> - GPU mode is not supported on macOS and Windows.

|                             |       Editor       |   Linux (x86_64)   |   macOS (x86_64)   |   macOS (ARM64)    |  Windows (x86_64)  |      Android       |        iOS         | WebGL |
| :-------------------------: | :----------------: | :----------------: | :----------------: | :----------------: | :----------------: | :----------------: | :----------------: | :---: |
|     Linux (AMD64) [^1]      | :heavy_check_mark: | :heavy_check_mark: |                    |                    |                    | :heavy_check_mark: |                    |       |
|          Intel Mac          | :heavy_check_mark: |                    | :heavy_check_mark: |                    |                    | :heavy_check_mark: | :heavy_check_mark: |       |
|         M1 Mac [^2]         | :heavy_check_mark: |                    |                    | :heavy_check_mark: |                    | :heavy_check_mark: | :heavy_check_mark: |       |
| Windows 10/11 (AMD64) [^3]  | :heavy_check_mark: |                    |                    |                    | :heavy_check_mark: | :heavy_check_mark: |                    |       |

[^1]: Tested on Arch Linux.\
[^2]: Experimental, because MediaPipe does not support M1 Mac.\
[^3]: Running MediaPipe on Windows is [experimental](https://google.github.io/mediapipe/getting_started/install.html#installing-on-windows).

## Prerequisites

If Docker is not available, the below commands/tools/libraries are required.

- `Python >= 3.9.0`
- `Bazelisk`
- `GCC/G++ >= 8.0.0`
- `NuGet`

For each platform, check the build instruction.

> :bell:
>
> - If not specified, the run command is under `MediaPipe4Unity` folder

[Linux](#linux)

- [Docker](#docker)
- [Arch Linux](#linux-no-docker)

[macOS](#macos)

- [Intel Mac](#intel-mac)
- [M1 Mac](#m1-mac)

[Windows](#windows)

- [Docker Windows Container](#windows-container)
- [Docker Linux Container](#linux-container)
- [Windows 10](#windows-10)

## Linux

> :warning:
>
> - If the `GNU libc` version in the target machine is less than the version of it in the machine where `libmediapipe_c.so` is built, `libmediapipe_c.so` won't work.

### Docker

> :warning:
>
> - For the best practices, check out the installation instruction [here](https://docs.docker.com/engine/install/)
> - Make sure you can run `docker` instead of `sudo docker`. Leave this when you're using `root`

**1. Build a Docker image**

```sh
docker build --build-arg UID=$(id -u) -t mediapipe_unity:latest . -f docker/linux/x86_64/Dockerfile
```

**2. Run a Docker container**

```sh
# Run with `Packages` directory mounted to the container
docker run \
    --mount type=bind,src=$PWD/Packages,dst=/home/mediapipe/Packages \
    --mount type=bind,src=$PWD/Assets,dst=/home/mediapipe/Assets \
    -it mediapipe_unity:latest
```

**3. Run the [build command](#build-command) inside the container**

```sh
# Build native libraries for Desktop CPU.
# Note that you need to specify `--opencv cmake` because OpenCV is not installed in the container.
python build.py build --desktop cpu --opencv cmake -v

# Build native libraries for Desktop GPU and Android
python build.py build --desktop gpu --android arm64 --opencv cmake -v
```

### Linux (no `docker`)

> :warning:
>
> - These commands are only for Arch-based distribution. If you deploy Mediapipe on Debian-based or other distribution, replace the packages and commands.
> - If your `GNU libc` version in the target machine is compatible with it in the container, always prefer [Docker](#docker).
> - Don't forget to update your distribution first.

**1. Install [`yay`](https://github.com/Jguer/yay)**

```sh
# Run under your favorite directory
pacman -S --needed git base-devel
git clone https://aur.archlinux.org/yay.git
cd yay
makepkg -si
```

**2. Install the required packages**

```sh
yay -Sy unzip mesa npm
```

> :warning:
>
> - It is recommended to configure [`npm`](https://docs.npmjs.com/cli/v8/configuring-npm/npmrc#files)

```txt
# ~/.npmrc
prefix = ${HOME}/.npm-packages
```

```txt
# ~/.bash_profile
export PATH=${HOME}/.npm-packages/bin:${PATH}
```

**3. *(Optional)* If you'd like to link OpenCV dynamically, install OpenCV. Skip this step if you want to link OpenCV statically.**

```sh
yay -S opencv3-opt
```

> :warning:
>
> - By default, it is assumed that OpenCV 3 is installed under `/usr` (e.g. `/usr/lib/libopencv_core.so`).
> - `opencv3-opt` will install OpenCV 3 to `/opt/opencv3`, so you need to edit [WORKSPACE](https://github.com/homuler/MediaPipeUnityPlugin/blob/master/WORKSPACE).
> - If you'd like to use OpenCV 4, you need to edit [third_party/opencv_linux.BUILD](https://github.com/homuler/MediaPipeUnityPlugin/blob/master/third_party/opencv_linux.BUILD), too.

```starlark
# WORKSPACE
new_local_repository(
    name = "linux_opencv",
    build_file = "@//third_party:opencv_linux.BUILD",
    path = "/opt/opencv3",
)
```

**4. Install [Bazelisk](https://github.com/bazelbuild/bazelisk#installation) and [NuGet](https://docs.microsoft.com/en-us/nuget/install-nuget-client-tools#nugetexe-cli)**

```sh
npm install -g bazelisk
yay -S nuget
```

**5. Install NumPy**

```sh
pip install numpy
```

**6. Run the [build command](#build-command)**

## macOS

### Intel Mac

**1. Install [Homebrew](https://brew.sh)**

**2. Install OpenCV 3**

```sh
brew install opencv@3
brew uninstall --ignore-dependencies glog
```

**3. Install Python and NumPy**

> :warning:
>
> - You must have Python 3.9 or later.

```sh
brew install python
export PATH=$PATH:"$(brew --prefix)/opt/python/libexec/bin"
pip3 install --user six numpy
```

**4. Install [Bazelisk](https://github.com/bazelbuild/bazelisk#installation) and [NuGet](https://docs.microsoft.com/en-us/nuget/install-nuget-client-tools#nugetexe-cli)**

```sh
brew install bazelisk
brew install nuget
```

**5. Install Xcode using App Store and CLI tools**

```sh
#If you are still not agreed to the license yet
sudo xcodebuild -license 
sudo xcode-select -s /Applications/Xcode.app
xcode-select --install
```

**6. Run the [build command](#build-command)**

### M1 Mac

> :warning:
>
> - Please use Unity version 2021.2.2f1 to open the project.

**1. Install [Homebrew](https://brew.sh)**

**2. Install OpenCV 3.**

```sh
brew install opencv@3
brew uninstall --ignore-dependencies glog
```

**3. Install Python and NumPy**

```sh
brew install python
export PATH=$PATH:"$(brew --prefix)/opt/python/libexec/bin"
pip3 install --user six numpy
```

**4. Install [Bazelisk](https://github.com/bazelbuild/bazelisk#installation)**

```sh
brew install bazelisk
```

**5. Install [NuGet](https://docs.microsoft.com/en-us/nuget/install-nuget-client-tools#nugetexe-cli)**

```sh
/usr/sbin/softwareupdate --install-rosetta

# Install Homebrew using Rosetta 2
# Before running the command, please check https://brew.sh/ for the correct URL
arch -x86_64 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

# Install NuGet using Rosetta 2
arch -x86_64 /usr/local/Homebrew/bin/brew install nuget
```

> :warning:
>
> - If you've not included `/usr/local/Homebrew/bin` in `PATH`, do it here.

```txt
# If you're using zsh
# ~/.zprofile

export PATH=/usr/local/Homebrew/bin:$PATH

# NOTE: The below command must be executed after `PATH` is exported.
#       Otherwise, when you run `brew`, `/usr/local/Homebrew/bin/brew` (x86_64 version) will be run.
eval "$(/opt/homebrew/bin/brew shellenv)"

# Set Python path
export PATH=$PATH:"$(brew --prefix)/opt/python/libexec/bin"
```

**6. Install Xcode using App Store and CLI tools**

```sh
#If you are still not agreed to the license yet
sudo xcodebuild -license 
sudo xcode-select -s /Applications/Xcode.app
xcode-select --install
```

**7. Run the [build command](#build-command)**

## Windows

> :warning:
>
> - You MUST NOT set `core.autocrlf` in `.gitconfig` (run `git config -l`).
> - To build libraries successfully, you need to check out the source code *as it is* without changing the line feed code.

### Windows Container

> :warning:
>
> - You must either use [WSL](https://aka.ms/wsl) or Hyper-V. By default installation, Docker Desktop will use `WSL`.
> - Once you choose `WSL`, make sure that you're using Windows 11 or Windows Server 2022 22H2 because of Windows 10 WSL brige incompatibility

**1. Install [Docker Desktop](https://docs.docker.com/desktop/windows/install/), then switch to the Windows container**

**2. Build a Docker image**

```bat
docker build -t mediapipe_unity:windows . -f docker/windows/x86_64/Dockerfile
```

> :warning:
>
> - This process will hang when MSYS2 is being installed.
> - If this issue occurs, [remove `C:\ProgramData\Docker\tmp\hcs*\Files\$Recycle.Bin\` manually](https://github.com/docker/for-win/issues/8910).

**3. Run a Docker container**

```bat
Rem Run with `Packages` directory mounted to the container
Rem Specify `--cpus` and `--memory` options according to your machine.
docker run --cpus=16 --memory=32g ^
    --mount type=bind,src=%CD%\Packages,dst=C:\mediapipe\Packages ^
    --mount type=bind,src=%CD%\Assets,dst=C:\mediapipe\Assets ^
    -it mediapipe_unity:windows
```

**4. Run the [build command](#build-command) inside the container**

```bat
python build.py build --desktop cpu --opencv cmake -vv
Rem or if you'd like to link OpenCV dynamically
python build.py build --desktop cpu -vv
```

### Linux Container

**1. Install [Docker Desktop](https://docs.docker.com/desktop/windows/install/) and switch to Linux containers**

**2. Build a Docker image**

```bat
docker build -t mediapipe_unity:linux . -f docker/linux/x86_64/Dockerfile
```

**3. Run a Docker container**

```bat
Rem Run with `Packages` directory mounted to the container
Rem Specify `--cpus` and `--memory` options according to your machine.
docker run --cpus=16 --memory=16g ^
    --mount type=bind,src=%CD%\Packages,dst=/home/mediapipe/Packages ^
    --mount type=bind,src=%CD%\Assets,dst=/home/mediapipe/Assets ^
    -it mediapipe_unity:linux
```

**4. Run the [build command](#build-command) inside the container**

```sh
python build.py build --android arm64 -vv
```

### Windows 10

> :warning:
>
> - Run commands using `cmd.exe`. It's known that some commands do not work properly with MSYS2.

**1. Install [MSYS2](https://www.msys2.org/) and edit the `%PATH%` environment variable.**

**2. Install necessary packages**

```sh
pacman -S git patch unzip
```

**3. Install `Python>=3.9.0`**

**4. Install Visual C++ Build Tools 2019 and Windows SDK**

![Visual Studio Installer](https://user-images.githubusercontent.com/4690128/144782083-7320741b-3ca4-4442-95ae-40421c7725ae.png)

**5. Install [Bazelisk](https://docs.bazel.build/versions/main/install-bazelisk.html) and add the location of the executable to `%PATH%`**

> :warning:
>
> - Note that you need to rename the executable to `bazel.exe`.

**6. Install [NuGet](https://docs.microsoft.com/en-us/nuget/install-nuget-client-tools#nugetexe-cli), and add the location of the NuGet executable to the `%PATH%` environment variable**

```bat
nuget
```

**7. Install NumPy**

```bat
pip install numpy --user
```

**8. Run the [build command](#build-command)**

```bat
python build.py build --desktop cpu --opencv=cmake -v
Rem or if you'd like to use local OpenCV
python build.py build --desktop cpu -v
```

## Build Script

`build.py` supports the following commands.

|   Command   |                              Description                              |
| :---------: | :-------------------------------------------------------------------: |
|   `build`   | Build and install required files (libraries, model files, C# scripts) |
|   `clean`   |             Clean cache directories (`build`, `bazel-*`)              |
| `uninstall` |                         Remove install files                          |

### Build Command

Run `python build.py build --help` for more details.

```sh
# Build for Desktop with GPU enabled (Linux only).
python build.py build --desktop gpu -vv

# If you've not installed OpenCV locally, you need to build OpenCV from sources for Desktop.
python build.py build --desktop gpu --opencv cmake -vv

# Build for Desktop with GPU disabled.
python build.py build --desktop cpu -vv

# Build a Universal macOS Library (macOS only).
python build.py build --desktop cpu --opencv cmake --macos_universal -vv

# Specify required solutions
python build.py build --desktop gpu --solutions face_mesh hands pose -vv
```

You can also specify compilation mode and linker options.

```sh
# Build with debug symbols.
python build.py build -c dbg --desktop gpu -vv

# Omit all symbol information.
# This can significantly reduce the library size.
python build.py build -c dbg --desktop gpu --linkopt=-s -vv
