## Prerequisites

If Docker is not available, the below commands/tools/libraries are required.

- Python 3.9.0
- Bazelisk (tested against Bazel 5.0.0)
- GCC/G++ 8.0.0 (Linux, macOS)
- NuGet (tested against 5.10.0.7240)

Please go to the article for each OS for more details.

> :bell: Run commands at the project root if not specified otherwise.

- [Linux](#linux)
   - [Arch Linux](#arch-linux)
- [macOS](#macos)
   - [Intel Mac](#intel-mac)
   - [M1 Mac](#m1-mac)
- [Windows](#windows)


# Supported Platforms

> :warning: GPU mode is not supported on macOS and Windows.

|                             |       Editor       |   Linux (x86_64)   |   macOS (x86_64)   |   macOS (ARM64)    |  Windows (x86_64)  |      Android       |        iOS         | WebGL |
| :-------------------------: | :----------------: | :----------------: | :----------------: | :----------------: | :----------------: | :----------------: | :----------------: | :---: |
|     Linux (AMD64) [^1]      | :heavy_check_mark: | :heavy_check_mark: |                    |                    |                    | :heavy_check_mark: |                    |       |
|          Intel Mac          | :heavy_check_mark: |                    | :heavy_check_mark: |                    |                    | :heavy_check_mark: | :heavy_check_mark: |       |
|         M1 Mac [^2]         | :heavy_check_mark: |                    |                    | :heavy_check_mark: |                    | :heavy_check_mark: | :heavy_check_mark: |       |
| Windows 10/11 (AMD64) [^3]  | :heavy_check_mark: |                    |                    |                    | :heavy_check_mark: | :heavy_check_mark: |                    |       |

[^1]: Tested on Arch Linux.\
[^2]: Experimental, because MediaPipe does not support M1 Mac.\
[^3]: Running MediaPipe on Windows is [experimental](https://google.github.io/mediapipe/getting_started/install.html#installing-on-windows).


## Linux

> :warning: If the GNU libc version in the target machine is less than the version of it in the machine where `libmediapipe_c.so` (a native library for Linux) is built, `libmediapipe_c.so` won't work.\
> For the same reason, if your target machine's GNU libc version is less than 2.27, you cannot use Docker[^5].

[^5]: You can still use Docker, but you need to write `Dockerfile` by yourself.

### Docker

Docker is removed, since it's useless in the projects which are being used.

### Arch Linux

If you are using another distribution, please replace some of the commands.

> :warning: If your GNU libc version in the target machine is compatible with it in the container, always prefer [Docker](#docker).

1. Install [yay](https://github.com/Jguer/yay)

   ```sh
   # Run under your favorite directory

   pacman -S --needed git base-devel
   git clone https://aur.archlinux.org/yay.git
   cd yay
   makepkg -si
   ```

1. Install required packages

   ```sh
   yay -Sy unzip mesa npm
   ```

   It is recommended to configure `npm` here (cf. https://docs.npmjs.com/cli/v8/configuring-npm/npmrc#files).

   ```txt
   # ~/.npmrc
   prefix = ${HOME}/.npm-packages
   ```

   ```txt
   # ~/.bash_profile
   export PATH=${HOME}/.npm-packages/bin:${PATH}
   ```

1. (Optional) If you'd like to link OpenCV dynamically, install OpenCV.

   Skip this step if you want to link OpenCV statically.

   ```sh
   yay -S opencv3-opt
   ```

   By default, it is assumed that OpenCV 3 is installed under `/usr` (e.g. `/usr/lib/libopencv_core.so`).\
   `opencv3-opt` will install OpenCV 3 to `/opt/opencv3`, so you need to edit [WORKSPACE](https://github.com/homuler/MediaPipeUnityPlugin/blob/master/WORKSPACE).

   ```starlark
   # WORKSPACE
   new_local_repository(
       name = "linux_opencv",
       build_file = "@//third_party:opencv_linux.BUILD",
       path = "/opt/opencv3",
   )
   ```

   :bell: If you'd like to use OpenCV 4, you need to edit [third_party/opencv_linux.BUILD](https://github.com/homuler/MediaPipeUnityPlugin/blob/master/third_party/opencv_linux.BUILD), too.

1. Install [Bazelisk](https://github.com/bazelbuild/bazelisk#installation) and [NuGet](https://docs.microsoft.com/en-us/nuget/install-nuget-client-tools#nugetexe-cli)

   ```sh
   npm install -g bazelisk
   yay -S nuget
   ```

1. Install NumPy

   ```sh
   pip install numpy
   ```

1. (For Android) [Install Android SDK and Android NDK](#android-configuration)

1. Run [build command](#build-command)

## macOS

### Intel Mac

1. Install [Homebrew](https://brew.sh)

1. Install OpenCV 3.

   > :bell: It's essentially an optional step, but if you'd like to build libraries for iOS, it's necessary because of a bug.

   ```sh
   brew install opencv@3
   brew uninstall --ignore-dependencies glog
   ```

1. Install Python and NumPy

   If your Python version is **< 3.9.0**, install it here in your favorite way.

   ```sh
   brew install python
   export PATH=$PATH:"$(brew --prefix)/opt/python/libexec/bin"

   # Python version must be >= 3.9.0
   python --version
   # Python 3.9.x

   pip3 install --user six numpy
   ```

1. Install [Bazelisk](https://github.com/bazelbuild/bazelisk#installation) and [NuGet](https://docs.microsoft.com/en-us/nuget/install-nuget-client-tools#nugetexe-cli)

   ```sh
   brew install bazelisk
   brew install nuget
   ```

1. Install Xcode using App Store

   After that, install the Command Line Tools, too.

   ```sh
   sudo xcodebuild -license # if not agreed to the license yet
   sudo xcode-select -s /Applications/Xcode.app
   xcode-select --install
   ```

1. (For iOS) Open [`mediapipe_api/objc/BUILD`](https://github.com/homuler/MediaPipeUnityPlugin/blob/master/mediapipe_api/objc/BUILD#L29) and modify `bundle_id`.

1. (For Android) [Install Android SDK and Android NDK](#android-configuration)

1. Run [build command](#build-command)

### M1 Mac

> :warning: use UnityEditor (Apple silicon) (>= 2021.2.2f1) to open the project.

1. Install [Homebrew](https://brew.sh)

1. Install OpenCV 3.

   > :bell: It's essentially an optional step, but if you'd like to build libraries for iOS, it's necessary because of a bug.

   ```sh
   brew install opencv@3
   brew uninstall --ignore-dependencies glog
   ```

1. Install Python and NumPy

   If your Python version is **< 3.9.0**, install it here in your favorite way.

   ```sh
   brew install python
   export PATH=$PATH:"$(brew --prefix)/opt/python/libexec/bin"

   # Python version must be >= 3.9.0
   python --version
   # Python 3.9.x

   pip3 install six numpy
   ```

1. Install [Bazelisk](https://github.com/bazelbuild/bazelisk#installation)

   ```sh
   brew install bazelisk
   ```

1. Install [NuGet](https://docs.microsoft.com/en-us/nuget/install-nuget-client-tools#nugetexe-cli)

   ```sh
   /usr/sbin/softwareupdate --install-rosetta

   # Install Homebrew using Rosetta 2
   # Before running the command, please check https://brew.sh/ for the correct URL
   arch -x86_64 /bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

   # Install NuGet using Rosetta 2
   arch -x86_64 /usr/local/Homebrew/bin/brew install nuget
   ```

   If you've not included `/usr/local/Homebrew/bin` in `PATH`, do it here.

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

1. Install Xcode using App Store

   After that, install the Command Line Tools, too.

   ```sh
   sudo xcodebuild -license # if not agreed to the license yet
   sudo xcode-select -s /Applications/Xcode.app
   xcode-select --install
   ```

1. (For iOS) Open [`mediapipe_api/objc/BUILD`](https://github.com/homuler/MediaPipeUnityPlugin/blob/master/mediapipe_api/objc/BUILD#L29) and modify `bundle_id`.

1. (For Android) [Install Android SDK and Android NDK](#android-configuration)

1. Run [build command](#build-command)

## Windows
> :warning: You MUST NOT set `core.autocrlf` in `.gitconfig` (run `git config -l`).
To build libraries successfully, you need to check out the source code _as it is_ without changing the line feed code.
>
> :warning: You cannot build libraries for _Android_ with the following steps.
>
> :warning: Run commands using `cmd.exe`. It's known that some commands do not work properly with MSYS2.
>
> :warning: We are all recommend that you use the Windows Subsystem for Linux to build packages for Linux. See [here](aka.ms/wsl) for more infomation. 

1. Install [MSYS2](https://www.msys2.org/) and edit the `%PATH%` environment variable.

   If MSYS2 is installed to `C:\msys64`, add `C:\msys64\usr\bin` to your `%PATH%` environment variable.

1. Install necessary packages

   ```sh
   pacman -S git patch unzip
   ```

1. Install Python >= 3.9.0 and allow the executable to edit the `%PATH%` environment variable.

   Download Python Windows executable from https://www.python.org/downloads/windows/ and install.

1. Install Visual C++ Build Tools 2022 (with Windows SDKs)

   1. Download Build Tools from https://visualstudio.microsoft.com/visual-cpp-build-tools/ and run Visual Studio Installer.

   1. Select `Desktop development with C++` and install it.

   ![Visual Studio Installer](https://user-images.githubusercontent.com/4690128/144782083-7320741b-3ca4-4442-95ae-40421c7725ae.png)

1. Install [Bazelisk](https://docs.bazel.build/versions/main/install-bazelisk.html) and add the location of the executable to `%PATH%` environment variable.

   Note that you need to rename the executable to `bazel.exe`.

1. (Optional) Set Bazel variables.

   If you have installed multiple Visual Studios or Win SDKs, set environment variables here.\
   Learn more details about [“Build on Windows”](https://docs.bazel.build/versions/main/windows.html#build-on-windows) in the Bazel official documentation.

   ```bat
   Rem Please find the exact paths and version numbers from your local version.

   set BAZEL_VS=C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools
   set BAZEL_VC=C:\Program Files (x86)\Microsoft Visual Studio\2019\BuildTools\VC
   set BAZEL_VC_FULL_VERSION=<Your local VC version>
   set BAZEL_WINSDK_FULL_VERSION=<Your local WinSDK version>
   ```

1. (Optional) If you'd like to link OpenCV dynamically, install OpenCV.

   You can skip this step if you want to link OpenCV statically.

   > :bell: It can take a very long time to link OpenCV statically because you need to build OpenCV on your local machine.\
     For reference, it takes about 5 - 6 minutes on Ryzen 3900x (12-core, 24-thread).

   By default, it is assumed that OpenCV 3.4.16 is installed under `C:\opencv`.\
   If your version or path is different, please edit [third_party/opencv_windows.BUILD](https://github.com/homuler/MediaPipeUnityPlugin/blob/master/third_party/opencv_windows.BUILD) and [WORKSPACE](https://github.com/homuler/MediaPipeUnityPlugin/blob/master/WORKSPACE).

1. Install [NuGet](https://docs.microsoft.com/en-us/nuget/install-nuget-client-tools#nugetexe-cli), and add the location of the NuGet executable to the `%PATH%` environment variable.

   ```bat
   nuget
   ```

1. Install NumPy

   ```bat
   pip install numpy --user
   ```

1. Run [build command](#build-command)

   ```bat
   python build.py build --desktop cpu --opencv=cmake -v
   Rem or if you'd like to use local OpenCV
   python build.py build --desktop cpu -v
   ```

## Android Configuration

1. Install [Android Studio](https://developer.android.com/studio/install)

1. Install Android SDK and NDK

   Open **SDK Manager > SDK Tools** and install Android SDK Build Tools and NDK.

   :bell: Note the following 2 points:

   - Bazel will use the newest version of Build Tools found automatically, but the latest Bazel (4.2.1) does not support Build Tools >= 31.0.0, so you need to uncheck these versions.
   - Bazel does not support NDK >= r22 yet

   ![Android Studio (SDK Tools)](https://user-images.githubusercontent.com/4690128/144735652-21339ab0-5a45-4277-b7ee-39d106b5e1e6.png)

1. Set environment variables

   - Linux/macOS

     ```sh
     # Set ANDROID_HOME
     # This directory should contain directories such as `platforms` and `platform-tools`.
     export ANDROID_HOME=/path/to/SDK

     # Set ANDROID_NDK_HOME
     # This is usually like `$ANDROID_HOME/ndk/21.4.7075529
     export ANDROID_NDK_HOME=/path/to/NDK
     ```

   - Windows

     Set them using GUI

1. Set API Level

   To support older Android, you need to specify the API level for NDK when running `build.py`.\
   Otherwise, some symbols in `libmediapipe_jni.so` cannot be resolved and `DllNotFoundException` will be thrown at runtime.

   ```sh
   python build.py build --android arm64 --android_ndk_api_level 21 -vv
   ```

# Build Script

`build.py` supports the following commands.

|   Command   |                              Description                              |
| :---------: | :-------------------------------------------------------------------: |
|   `build`   | Build and install required files (libraries, model files, C# scripts) |
|   `clean`   |             Clean cache directories (`build`, `bazel-*`)              |
| `uninstall` |                         Remove install files                          |

## Build Command

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

# Build for Desktop, Android, and iOS
python build.py build --desktop cpu --android arm64 --ios arm64 -vv

# Specify Android NDK level
python build.py build --android arm64 --android_ndk_api_level 21 -vv

# Specify required solutions
python build.py build --desktop gpu --solutions face_mesh hands pose -vv
```

You can also specify compilation mode and linker options.

```sh
# Build with debug symbols.
python build.py build -c dbg --android arm64 -vv

# Omit all symbol information.
# This can significantly reduce the library size.
python build.py build --android arm64 --linkopt=-s -vv
```

# Known Issues

1. (iOS) When building native libraries for iOS, OpenCV must be installed on your Mac.

   To be precise, `/usr/local/opt/opencv@3` (Intel Mac) or `/opt/homebrew/opt/opencv@3` directory must exist (even if it's empty).

1. (iOS) In some situations, the iOS Framework won't be updated.

   Due to [an issue with bazel](https://github.com/bazelbuild/bazel/issues/12389), we cannot determine the output path beforehand.\
   If you have built this plugin across versions, you may encounter this problem.\
   In that case, run `python build.py clean` and try your build command again.

# Troubleshooting

## DllNotFoundException

This error can occur for a variety of reasons, so it is necessary to isolate the cause.

1. Native libraries are not built yet

   If native libraries (`libmediapipe_c.{so,dylib,dll} / mediapipe_android.aar / MediaPipeUnity.Framework`) are not built yet, this error can occur because Unity cannot load them.

   If they don't exist under `Packages/com.github.homuler.mediapipe/Runtime/Plugins`, run [build command](#build-command) first, and make sure that the command finishes successfully.

1. Native libraries are incompatible with your device

   1. Libraries built on Linux machines won't work on your Windows machine (see [Supported Platforms](#supported-platforms)).
   1. If it occurs on your Android device, maybe
      - architecture is wrong (`--android armv7`) or
      - API Level is not compatible (see [Android Configuration](#android-configuration))

1. Dependent libraries are not linked

   This error occurs when some symbols in the libraries are undefined and cannot be resolved at runtime, which is very typical when OpenCV is not configured properly.\
   See `opencv_linux.BUILD` / `opencv_windows.BUILD` and check if the path is correct (if not, edit the BUILD file).\
   You can also build and link OpenCV statically with `--opencv cmake` option instead.

   **Tips**\
   In this case, when you check on [Load on startup](https://docs.unity3d.com/Manual/PluginInspector.html) and click the `Apply` button, error logs like the following will be output.

   ```txt
   Plugins: Couldn't open Packages/com.github.homuler.mediapipe/Runtime/Plugins/libmediapipe_c.so, error:  Packages/com.github.homuler.mediapipe/Runtime/Plugins/libmediapipe_c.so: undefined symbol: _ZN2cv8fastFreeEPv
   ```

1. Dependent libraries do not exist or are not loaded

   When you build an app and copy it to another machine, you need to bundle dependent libraries with it.

   For example, when you build `libmediapipe_c.so` with `--opencv local`, OpenCV is dynamically linked to `libmediapipe_c.so`.\
   To use this on another machine, OpenCV must be installed on the machine too.\
   In this case, the recommended way is to build `libmediapipe_c.so` with `--opencv cmake` (or bundle OpenCV with your app).

   If you are unsure of the cause, try checking the dependent libraries using such as `ldd` command.

1. When you cannot identify the cause...

   If the error occurs on UnityEditor, try loading `{lib}mediapipe_c.{so,dylib,dll}` on startup.

   ![load-on-startup](https://user-images.githubusercontent.com/4690128/135591282-8a2011b1-9ae8-4a6a-a5fb-cc3a21f1125f.png)

   `DllNotFoundException` should be thrown right after that, and if you're lucky, the error message will be more verbose now.

   Otherwise, try loading the library (on your target device) by yourself using tools or code like below.

   <details>
   <summary>Android</summary>

   ```cs
   public void LoadLibrary()
   {
       using (var system = new AndroidJavaClass("java.lang.System"))
       {
           system.CallStatic("loadLibrary", "mediapipe_jni");
       }
   }
   ```
   </details>

   <details>
   <summary>Windows</summary>
      
   ```cs
   public void LoadLibrary()
   {
   #if UNITY_EDITOR_WIN
       var path = Path.Combine("Packages", "com.github.homuler.mediapipe", "Runtime", "Plugins", "libmediapipe_c.dll");
   #elif UNITY_STANDALONE_WIN
       var path = Path.Combine(Application.dataPath, "Plugins", "libmediapipe_c.dll");
   #endif

       var handle = LoadLibraryW(path);

       if (handle != IntPtr.Zero)
       {
           // Success
           if (!FreeLibrary(handle))
           {
               Debug.LogError($"Failed to unload {path}: {Marshal.GetLastWin32Error()}");
           }
       }
       else
       {
           // Error
           // cf. https://docs.microsoft.com/en-us/windows/win32/debug/system-error-codes--0-499-
           var errorCode = Marshal.GetLastWin32Error();
           Debug.LogError($"Failed to load {path}: {errorCode}");

           if (errorCode == 126)
           {
               Debug.LogError("Check missing dependencies using [Dependencies](https://github.com/lucasg/Dependencies). If you're sure that required libraries exist, open the plugin inspector for those libraries and check `Load on startup`.");
           }
       }   
   }

   [DllImport("kernel32", SetLastError = true, ExactSpelling = true, CharSet = CharSet.Unicode)]
   private static extern IntPtr LoadLibraryW(string path);

   [DllImport("kernel32", SetLastError = true, ExactSpelling = true)]
   [return: MarshalAs(UnmanagedType.Bool)]
   private static extern bool FreeLibrary(IntPtr handle);
   ```
   </details>

   <details>
   <summary>Linux/macOS</summary>

   ```cs
   public void LoadLibrary()
   {
   #if UNITY_EDITOR_LINUX
       var path = Path.Combine("Packages", "com.github.homuler.mediapipe", "Runtime", "Plugins", "libmediapipe_c.so");
   #elif UNITY_STANDALONE_LINUX
       var path = Path.Combine(Application.dataPath, "Plugins", "libmediapipe_c.so");
   #elif UNITY_EDITOR_OSX
       var path = Path.Combine("Packages", "com.github.homuler.mediapipe", "Runtime", "Plugins", "libmediapipe_c.dylib");
   #elif UNITY_STANDALONE_OSX
       var path = Path.Combine(Application.dataPath, "Plugins", "libmediapipe_c.dylib");
   #endif

       var handle = dlopen(path, 2);

       if (handle != IntPtr.Zero)
       {
           // Success
           var result = dlclose(handle);

           if (result != 0)
           {
               Debug.LogError($"Failed to unload {path}");
           }
       }    
       else 
       {
           Debug.LogError($"Failed to load {path}: {Marshal.GetLastWin32Error()}");
           var error = Marshal.PtrToStringAnsi(dlerror());
           // TODO: release memory

           if (error != null)
           {
               Debug.LogError(error);
           }
       }
   }

   [DllImport("dl", SetLastError = true, ExactSpelling = true)]
   private static extern IntPtr dlopen(string name, int flags);

   [DllImport("dl", ExactSpelling = true)]
   private static extern IntPtr dlerror();

   [DllImport("dl", ExactSpelling = true)]
   private static extern int dlclose(IntPtr handle);
   ```
   </details>

## /bin/bash: line 1: $'\r': command not found
Probably you've set `core.autocrlf=true` in `.gitconfig` (run `git config -l`).\
Check out the source code _as it is_ without changing the line feed code.

See https://git-scm.com/book/en/v2/Customizing-Git-Git-Configuration#_formatting_and_whitespace for more details.

## ERROR: PKGBUILD contains CRLF characters and cannot be sourced.
See the above.

## No matching toolchains found for types @bazel_tools//tools/android:sdk_toolchain_type
To build libraries for Android, you need to set `ANDROID_HOME` and `ANDROID_NDK_HOME`.
See [Android Configuration](#android-configuration) for more details.

## java.io.IOException: ERROR: src/main/native/windows/process.cc(202): CreateProcessW("_command_" ...
_command_ is often `git`.\
This error occurs when you're building libraries on Windows and the _command_ path is not resolved.\
Make sure the path to the desired command is set in your `%PATH%`.

## java.io.IOException: Error downloading [...]: Checksum was xxx but wanted yyy
If you changed some URLs in the WORKSPACE file, you may have forgotten to change the corresponding `sha256sum` value.\
Otherwise, the contents may have been tampered with.

## cc_toolchain_suite '@local_config_cc//:toolchain' does not contain a toolchain for cpu 'ios_arm64'

This error means bazel could not find Xcode on your machine.\
If you've installed Xcode, check its path.

```sh
xcode-select --path
```

If the output is like `/Library/Developer/CommandLineTools`, then run the following command and try again.

```sh
# Change the path to suit your environment
sudo xcode-select --switch /Applications/Xcode.app/Contents/Developer
python3 build.py clean
```

## Undefined symbols for architecture arm64: "_glReadPixels"
Add `OpenGLES` to the Framework Dependencies.\

## AttributeError: module 'argparse' has no attribute 'BooleanOptionalAction'

Install Python >= 3.9.0

## SyntaxError: invalid syntax

Install Python >= 3.9.0
