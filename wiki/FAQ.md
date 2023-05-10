# Build/Installation
## Q: Why would you remove the Docker image for MediaPipe?

A: It's useless in Unity3D for Virtual Avatar and its applications.

## Q: How to reduce the plugin size?

A: Strip symbols from native libraries. See [Build Command](https://github.com/homuler/MediaPipeUnityPlugin/wiki/Installation-Guide#build-command) for more details.

# Development
## UnityEditor crashes!
When some errors occur, MediaPipe doesn't throw an exception but aborts the whole program (signals `SIGABRT`).\
It is not fatal in production since the application should crash in such a situation after all, but in a development environment, it is very annoying since the UnityEditor crashes.

On Linux and macOS, this plugin avoids UnityEditor crashing by handling `SIGABRT`, so if UnityEditor crashes, please let us know!\
On Windows, there seem to be no ways to handle `SIGABRT` properly, so if you cannot tolerate this, use a different OS.

## InternalException: INTERNAL: ; eglMakeCurrent() returned error 0x3000

If you encounter an error like below and you use OpenGL Core as the Unity's graphics APIs, please try Vulkan.

```txt
InternalException: INTERNAL: ; eglMakeCurrent() returned error 0x3000_mediapipe/mediapipe/gpu/gl_context_egl.cc:261)
```
