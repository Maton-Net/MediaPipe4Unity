# About

This wiki is written by @homuler. We only replace old links and remove unnecessary contents.

# Table of Contents

- [Preparation](#preparation)
  - [Build / Installation](#build--installation)
  - [Test](#test)
  - [Import into your project](#import-into-your-project)
- [Tutorial](#tutorial)
  - [Hello World!](#hello-world)
    - [Send input](#send-input)
    - [Get output](#get-output)
    - [More Tips](#more-tips)
  - [Official Solution](#official-solution)
    - [Setup WebCamTexture](#setup-webcamtexture)
    - [Send ImageFrame](#send-imageframe)
    - [Load model files](#load-model-files)
    - [Set the correct timestamp](#set-the-correct-timestamp)
    - [Get ImageFrame](#get-imageframe)
    - [Coordinate System](#coordinate-system)
    - [Get landmarks](#get-landmarks)
    - [Annotation](#annotation)
    - [More Tips](#more-tips-1)
  - [Official Solution (GPU)](#official-solution-gpu)
  - :construction: Other Topics
    - Side Packet
    - OutputStream API

# Preparation
## Build/Installation
This plugin requires native libraries (e.g. `libmediapipe_c.so`, `mediapipe_c.dll`, `mediapipe_android.aar`, etc...) to work, but they are not included in the repository.

## Test
Before using the plugin in your project, it's strongly recommended that you check if it works in _this project_.

First, open `Assets/MediaPipeUnity/Samples/Scenes/Start Scene.unity`.

![test-start-scene](https://user-images.githubusercontent.com/4690128/163668701-730a1623-3914-4b23-bfaf-75030e12496a.png)

And play the scene.\
If you've built the plugin successfully, the Face Detection sample will start after a while.

![test-start-scene-2](https://user-images.githubusercontent.com/4690128/163668702-26357605-c1f2-4678-8fce-3adc258a9aa1.png)

## Import into your project
Once you've built the plugin, you can import it into your project.
Choose your favorite method from the following options.

### Build and import a unity package
1. Open this project
1. Click `Tools > Export Unitypackage`
   ![export-unity-package](https://user-images.githubusercontent.com/4690128/163669270-2d5365eb-eac1-46b1-aed5-83c28a377090.png)
   - `MediaPipeUnity.[version].unitypackage` file will be created at the project root.
     
1. Open your project
1. [Import the built package](https://docs.unity3d.com/Manual/AssetPackagesImport.html)

### Build and install a local tarball file
1. Install `npm` command
1. Build a tarball file. Don't forget to to replace `[version]` with the version the specific.

   ```sh
   # Make Build folder
   mkdir build
   # Switch active folder
   cd Packages/com.maton-net.mediapipe4unitty
   # Pack the plugin, using Node Package Manager
   npm pack
   # com.github.homuler.mediapipe-[version].tgz will be created
   # Move to the root of project folder
   mv com.github.homuler.mediapipe-[version].tgz ../..
   ```

1. [Install the package from the tarball file](https://docs.unity3d.com/Manual/upm-ui-tarball.html)

### Install from a submodule
> :warning: Development with Git submodules tends to be a bit more complicated.

1. Add a submodule
   
   ```sh
   mkdir Submodules
   cd Submodules
   git submodule add https://github.com/Maton-Net/MediaPipe4Unity
   ```

1. Build the plugin
   
   ```
   cd MediaPipeUnityPlugin
   python build.py build ...
   
   ```

1. [Install the package from `Submodules/MediaPipeUnityPlugin/Packages/com.github.homuler.mediapipe`](https://docs.unity3d.com/Manual/upm-ui-local.html) 

# Tutorial
> :bell: If you are not familiar with MediaPipe, you may want to read the [Framework Concepts](https://google.github.io/mediapipe/framework_concepts/framework_concepts.html) article first.
> 
> :skull_and_crossbones: On Windows, some of the following code may crash your UnityEditor. See [FAQs](https://github.com/Maton-Net/MediaPipe4Unity/wiki/FAQ#unityeditor-crashes) for more details.

## Hello World!
Let's write our first program!

> :bell: The following code is based on [this welcome hello-world code](https://github.com/google/mediapipe/blob/cf101e62a9d49a51be76836b2b8e5ba5c06b5da0/mediapipe/examples/desktop/hello_world/hello_world.cc).
> 
> :bell: You can use [Tutorial/Hello World](https://github.com/Maton-Net/MediaPipe4Unity/tree/main/Assets/MediaPipeUnity/Tutorial/Hello%20World) as a template.

### Send input
To run the Calculators provided by MediaPipe, we usually need to initialize a `CalculatorGraph`, so let's do that first!

> :bell: Each `CalculatorGraph` has its own config ([`CalculatorGraphConfig`](https://google.github.io/mediapipe/framework_concepts/graphs.html#graphconfig)).

```cs
var configText = @"
input_stream: ""in""
output_stream: ""out""
node {
  calculator: ""PassThroughCalculator""
  input_stream: ""in""
  output_stream: ""out1""
}
node {
  calculator: ""PassThroughCalculator""
  input_stream: ""out1""
  output_stream: ""out""
}
";

var graph = new CalculatorGraph(configText);
```

To run a `CalculatorGraph`, call the `StartRun` method.

```cs
graph.StartRun().AssertOk();
```

Note that the `StartRun` method returns a `Status` object, which represents the result.\
`Status#AssertOk` throws iff the result is not OK.

After starting, of course we want to give inputs to the `CalculatorGraph`, right?

Let's say we want to give a sequence of 10 strings ("Hello World!") as input.

```cs
for (var i = 0; i < 10; i++)
{
  // Send input to running graph
}
```

In MediaPipe, input is passed through a class called [`Packet`](https://google.github.io/mediapipe/framework_concepts/framework_concepts.html#packet).

```cs
var input = new StringPacket("Hello World!");
```

To pass an input `Packet` to the `CalculatorGraph`, we can use `CalculatorGraph#AddPacketToInputStream`.\
Note that the only input stream name of _this `CalculatorGraph`_ is `in`.

> :bell: It depends on the `CalculatorGraphConfig`. `CalculatorGraph` can multiple input streams

```cs
for (var i = 0; i < 10; i++)
{
  var input = new StringPacket("Hello World!");
  graph.AddPacketToInputStream("in", input).AssertOk();
}
```

`CalculatorGraph#AddPacketToInputStream` also returns a `Status` object, so let's call `AssertOk` here as well.

After everything is done, we should

1. close input streams
1. dispose of the `CalculatorGraph`

so let's do that.\
Again, note that each method returns a `Status` object.

```cs
graph.CloseInputStream("in").AssertOk();
graph.WaitUntilDone().AssertOk();
graph.Dispose();
```

For now, let's just run the code we've written so far.

Save the following code as `HelloWorld.cs`, attach it to an empty `GameObject` and play the scene.

```cs
using UnityEngine;

namespace Mediapipe.Unity.Tutorial
{
  public class HelloWorld : MonoBehaviour
  {
    private void Start()
    {
      var configText = @"
input_stream: ""in""
output_stream: ""out""
node {
  calculator: ""PassThroughCalculator""
  input_stream: ""in""
  output_stream: ""out1""
}
node {
  calculator: ""PassThroughCalculator""
  input_stream: ""out1""
  output_stream: ""out""
}
";
      var graph = new CalculatorGraph(configText);
      graph.StartRun().AssertOk();

      for (var i = 0; i < 10; i++)
      {
        var input = new StringPacket("Hello World!");
        graph.AddPacketToInputStream("in", input).AssertOk();
      }

      graph.CloseInputStream("in").AssertOk();
      graph.WaitUntilDone().AssertOk();
      graph.Dispose();

      Debug.Log("Done");
    }
  }
}
```

![hello-world-timestamp-error](https://user-images.githubusercontent.com/4690128/163697398-49ede362-6665-4660-8bbe-e087c5743442.png)

It returns the following error:

```txt
MediaPipeException: INVALID_ARGUMENT: Graph has errors: 
; In stream "in", timestamp not specified or set to illegal value: Timestamp::Unset()
  at Mediapipe.Status.AssertOk () [0x00014] in /home/homuler/Development/unity/MediaPipeUnityPlugin/Packages/com.github.homuler.mediapipe/Runtime/Scripts/Framework/Port/Status.cs:50 
  at Mediapipe.Unity.Tutorial.HelloWorld.Start () [0x00025] in /home/homuler/Development/unity/MediaPipeUnityPlugin/Assets/MediaPipeUnity/Tutorial/Hello World/HelloWorld.cs:35 
```



Each input packet [should have a timestamp](https://google.github.io/mediapipe/framework_concepts/packets.html), but it does not appear to be set.\
The initialization creates  a `Packet` as follows.

```cs
// var input = new StringPacket("Hello World!");
var input = new StringPacket("Hello World!", new Timestamp(i));
```

![hello-world-no-output](https://user-images.githubusercontent.com/4690128/163697591-1128ebbb-192b-40b8-92d4-8b01b6eca55a.png)

It works, but data doesn't receive the output from `CalculatorGraph`.

### Get output
To get output, we need to do more work before running the `CalculatorGraph`.\
Note that the only output stream name of _this `CalculatorGraph`_ is `out`.

> :bell: It depends on the `CalculatorGraphConfig`. `CalculatorGraph` can multiple output streams.

```cs
var graph = new CalculatorGraph(configText);

// Initialize an `OutputStreamPoller`.
// NOTE: The type parameter is `string` since the output type is `string`.
var poller = graph.AddOutputStreamPoller<string>("out").Value();

graph.StartRun().AssertOk();
```

`CalculatorGraph#AddOutputStreamPoller<T>` returns a `StatusOr<T>` object.\
`StatusOr<T>` is similar to `Status`, but it can contain a value if the `Status` is OK.

> :warning: In production, you should check if it's OK before calling `StatusOr<V>#Value`.
> 
> ```cs
> var statusOrPoller = graph.AddOutputStreamPoller<string>("out");
> if (statusOrPoller.Ok())
> {
>   var poller = statusOrPoller.Value();
> }  
> ```

Then, we can get output using the `OutputStreamPoller<string>#Next`.\
Like inputs, outputs must be received through packets.

```cs
graph.CloseInputStream("in").AssertOk();

// Initialize an empty packet
var output = new StringPacket();

while (poller.Next(output))
{
  Debug.Log(output.Get());
}

graph.WaitUntilDone().AssertOk();
```

Now, our code would look like this.

```cs
using UnityEngine;

namespace Mediapipe.Unity.Tutorial
{
  public class HelloWorld : MonoBehaviour
  {
    private void Start()
    {
      var configText = @"
input_stream: ""in""
output_stream: ""out""
node {
  calculator: ""PassThroughCalculator""
  input_stream: ""in""
  output_stream: ""out1""
}
node {
  calculator: ""PassThroughCalculator""
  input_stream: ""out1""
  output_stream: ""out""
}
";
      var graph = new CalculatorGraph(configText);
      var poller = graph.AddOutputStreamPoller<string>("out").Value();
      graph.StartRun().AssertOk();

      for (var i = 0; i < 10; i++)
      {
        var input = new StringPacket("Hello World!", new Timestamp(i));
        graph.AddPacketToInputStream("in", input).AssertOk();
      }

      graph.CloseInputStream("in").AssertOk();

      var output = new StringPacket();
      while (poller.Next(output))
      {
        Debug.Log(output.Get());
      }

      graph.WaitUntilDone().AssertOk();
      graph.Dispose();

      Debug.Log("Done");
    }
  }
}
```

![hello-world-output](https://user-images.githubusercontent.com/4690128/163706442-79942043-374e-4b68-96ff-afd79a75932f.png)

### More Tips
#### Validate the config format
What happens if the config format is invalid?

```cs
var graph = new CalculatorGraph("invalid format");
```

![hello-world-invalid-config](https://user-images.githubusercontent.com/4690128/163708332-6a1e46bb-ac30-4dd5-bf94-25441a645910.png)

The code above returns a bug:

```txt
[libprotobuf ERROR external/com_google_protobuf/src/google/protobuf/text_format.cc:335] Error parsing text-format mediapipe.CalculatorGraphConfig: 1:9: Message type "mediapipe.CalculatorGraphConfig" has no field named "invalid".
MediaPipeException: Failed to parse config text. See error logs for more details
  at Mediapipe.CalculatorGraphConfigExtension.ParseFromTextFormat (Google.Protobuf.MessageParser`1[T] _, System.String configText) [0x0001e] in /home/homuler/Development/unity/MediaPipeUnityPlugin/Packages/com.github.homuler.mediapipe/Runtime/Scripts/Framework/CalculatorGraphConfigExtension.cs:21 
  at Mediapipe.CalculatorGraph..ctor (System.String textFormatConfig) [0x00000] in /home/homuler/Development/unity/MediaPipeUnityPlugin/Packages/com.github.homuler.mediapipe/Runtime/Scripts/Framework/CalculatorGraph.cs:33 
  at Mediapipe.Unity.Tutorial.HelloWorld.Start () [0x00000] in /home/homuler/Development/unity/MediaPipeUnityPlugin/Assets/MediaPipeUnity/Tutorial/Hello World/HelloWorld.cs:31 
```

Now set the log handler to track whether the exception appears.

```cs
Protobuf.SetLogHandler(Protobuf.DefaultLogHandler);
var graph = new CalculatorGraph("invalid format");
```

![hello-world-protobuf-logger](https://user-images.githubusercontent.com/4690128/163708754-5566ccd7-9dce-49c9-a2ce-dd67f9c7267e.png)

Don't forget to restore the default `LogHandler` when the application exits.

```cs
void OnApplicationQuit()
{
  Protobuf.ResetLogHandler();
}
```

## Official Solution
In this section, let's try running the Face Mesh Solution.

> :bell: You can use [Tutorial/Official Solution](https://github.com/Maton-Net/MediaPipe4Unity/tree/be96374adcc895dc9f675241333862b22cf61107/Assets/MediaPipeUnity/Tutorial/Official%20Solution) as a template.

### Setup WebCamTexture
First, let's display the Web Camera image on the screen.

> :bell: The code in this section is already saved to [`Tutorial/Official Solution/FaceMesh.cs`](https://github.com/Maton-Net/MediaPipe4Unity/blob/be96374adcc895dc9f675241333862b22cf61107/Assets/MediaPipeUnity/Tutorial/Official%20Solution/FaceMesh.cs)

```cs
using System.Collections;
using UnityEngine;
using UnityEngine.UI;

namespace Mediapipe.Unity.Tutorial
{
  public class FaceMesh : MonoBehaviour
  {
    [SerializeField] private TextAsset _configAsset;
    [SerializeField] private RawImage _screen;
    [SerializeField] private int _width;
    [SerializeField] private int _height;
    [SerializeField] private int _fps;

    private WebCamTexture _webCamTexture;

    private IEnumerator Start()
    {
      if (WebCamTexture.devices.Length == 0)
      {
        throw new System.Exception("Web Camera devices are not found");
      }
      var webCamDevice = WebCamTexture.devices[0];
      _webCamTexture = new WebCamTexture(webCamDevice.name, _width, _height, _fps);
      _webCamTexture.Play();

      yield return new WaitUntil(() => _webCamTexture.width > 16);

      _screen.rectTransform.sizeDelta = new Vector2(_width, _height);
      _screen.texture = _webCamTexture;
    }

    private void OnDestroy()
    {
      if (_webCamTexture != null)
      {
        _webCamTexture.Stop();
      }
    }
  }
}
```

If everything is fine, your screen will look like this.

![web-cam-setup](https://user-images.githubusercontent.com/4690128/163741680-8ce79e55-3011-4dc6-b306-c938f311a899.png)

### Send ImageFrame
Now let's try [`face_mesh_desktop_live.pbtxt`](https://github.com/google/mediapipe/blob/c6c80c37452d0938b1577bd1ad44ad096ca918e0/mediapipe/graphs/face_mesh/face_mesh_desktop_live.pbtxt), the official Face Mesh sample!

> :warning: To run the graph, you must build native libraries with GPU _disabled_.
>
> :bell: `face_mesh_desktop_live.pbtxt` is saved as [`Tutorial/Official Solution/face_mesh_desktop_live.txt`](https://github.com/Maton-Net/MediaPipe4Unity/blob/be96374adcc895dc9f675241333862b22cf61107/Assets/MediaPipeUnity/Tutorial/Official%20Solution/face_mesh_desktop_live.txt).

First, initialize a `CalculatorGraph` as in the Hello World example.

```cs
var graph = new CalculatorGraph(_configAsset.text);
graph.StartRun().AssertOk();
```

In MediaPipe, image data on the CPU is stored in a class called `ImageFrame`.\
Let's initialize an `ImageFrame` instance from the `WebCamTexture` image.

> :bell: On the other hand, image data on the GPU is stored in a class called `GpuBuffer`.

We can initialize an `ImageFrame` instance using `NativeArray<byte>`.\
Here, although not the best from the perspective of the performance, we will copy the `WebCamTexture` data to `Texture2D` to obtain a `NativeArray<byte>`.

```cs
Texture2D inputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
Color32[] pixelData = new Color32[_width * _height];

while (true)
{
  inputTexture.SetPixels32(_webCamTexture.GetPixels32(pixelData));

  yield return new WaitForEndOfFrame();
}
```

Now we can initialize an `ImageFrame` instance using `inputTexture`.

> :warning: In theory, you can build `ImageFrame` instances using various formats, but not all `Calculator`s necessarily support all formats. As for official solutions, they often work only with RGBA32 format.

```cs
var imageFrame = new ImageFrame(ImageFormat.Types.Format.Srgba, _width, _height, _width * 4, inputTexture.GetRawTextureData<byte>());
```

The 4th argument, `widthStep`, may require some explanation.\
It's the byte offset between a pixel value and the same pixel and channel in the next row.\
In most cases, this is equal to the product of the width and the number of channels.

As usual, initialize a `Packet` and send it to the `CalculatorGraph`.\
Note that the input stream name is `"input_video"` and the input type is `ImageFrame` this time.

```cs
graph.AddPacketToInputStream("input_video", new ImageFramePacket(imageFrame)).AssertOk();
```

We should stop the `CalculatorGraph` on the `OnDestroy` event.\
With a little refactoring, the code now looks like this.

```cs
using System.Collections;
using UnityEngine;
using UnityEngine.UI;

namespace Mediapipe.Unity.Tutorial
{
  public class FaceMesh : MonoBehaviour
  {
    [SerializeField] private TextAsset _configAsset;
    [SerializeField] private RawImage _screen;
    [SerializeField] private int _width;
    [SerializeField] private int _height;
    [SerializeField] private int _fps;

    private CalculatorGraph _graph;

    private WebCamTexture _webCamTexture;
    private Texture2D _inputTexture;
    private Color32[] _pixelData;

    private IEnumerator Start()
    {
      if (WebCamTexture.devices.Length == 0)
      {
        throw new System.Exception("Web Camera devices are not found");
      }
      var webCamDevice = WebCamTexture.devices[0];
      _webCamTexture = new WebCamTexture(webCamDevice.name, _width, _height, _fps);
      _webCamTexture.Play();

      yield return new WaitUntil(() => _webCamTexture.width > 16);

      _screen.rectTransform.sizeDelta = new Vector2(_width, _height);
      _screen.texture = _webCamTexture;

      _inputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
      _pixelData = new Color32[_width * _height];

      _graph = new CalculatorGraph(_configAsset.text);
      _graph.StartRun().AssertOk();

      while (true)
      {
        _inputTexture.SetPixels32(_webCamTexture.GetPixels32(_pixelData));
        var imageFrame = new ImageFrame(ImageFormat.Types.Format.Srgba, _width, _height, _width * 4, _inputTexture.GetRawTextureData<byte>());
        _graph.AddPacketToInputStream("input_video", new ImageFramePacket(imageFrame)).AssertOk();

        yield return new WaitForEndOfFrame();
      }
    }

    private void OnDestroy()
    {
      if (_webCamTexture != null)
      {
        _webCamTexture.Stop();
      }

      if (_graph != null)
      {
        try
        {
          _graph.CloseInputStream("input_video").AssertOk();
          _graph.WaitUntilDone().AssertOk();
        }
        finally
        {

          _graph.Dispose();
        }
      }
    }
  }
}
```

Play the scene to try this out

![face-mesh-load-fail](https://user-images.githubusercontent.com/4690128/163748657-a7202100-d39d-43ed-b947-6db4cf88abdd.png)

Well, it's not so easy, is it?

> ```txt
> MediaPipeException: INVALID_ARGUMENT: Graph has errors: 
> Calculator::Open() for node "facelandmarkfrontcpu__facelandmarkcpu__facelandmarksmodelloader__LocalFileContentsCalculator" failed: ; Can't find file: mediapipe/modules/face_landmark/face_landmark_with_attention.tflite
> ```

It looks like `LocalFileContentsCalculator` failed to load `face_landmark_with_attention.tflite`.\
In the next section, we will resolve this error.

> :warning: If you get error messages like the following, go to [Officla Solution (GPU)](https://github.com/Maton-Net/MediaPipe4Unity/wiki/Getting-Started#official-solution-gpu).
> ```txt
> F20220418 11:58:05.626176 230087 calculator_graph.cc:126] Non-OK-status: Initialize(config) status: NOT_FOUND: ValidatedGraphConfig Initialization failed.
> No registered object with name: FaceLandmarkFrontCpu; Unable to find Calculator "FaceLandmarkFrontCpu"
> No registered object with name: FaceRendererCpu; Unable to find Calculator "FaceRendererCpu"
> ```

### Load model files
To load model files on Unity, we need to resolve their paths because they are hardcoded.\
Not only that, we even need to _save_ the file in a specific path because some calculators will read the required resources from the file system.

> :bulb: The path to save is not fixed since we can translate each model path into an arbitrary path.

But don't worry. In most cases, all you need to do is initialize a `ResourceManager` class and call the `PrepareAssetAsync` method in advance.

> :bulb: `PrepareAssetAsync` method will save the specified file under `Application.persistentDataPath`.

For testing purposes, the `LocalResourceManager` class is sufficient.
```cs
var resourceManager = new LocalResourceManager();
yield return resourceManager.PrepareAssetAsync("dependent_asset_name");
```

In development / production, you can choose either `StreamingAssetResourceManager` or `AssetBundleResourceManager`.\
For example, `StreamingAssetResourceManager` will load model files from [`Application.streamingAssetsPath`](https://docs.unity3d.com/Manual/StreamingAssets.html).

> :warning: To use `StreamingAssetResourceManager`, you need to place dependent assets under `Assets/StreamingAssets`. You can usually copy those assets from `Packages/com.github.homuler.mediapipe/Runtime/Resources`.
> ```cs
> var resourceManager = new StreamingAssetsResourceManager();
> yield return resourceManager.PrepareAssetAsync("dependent_asset_name");
> ```
>
> :warning: `ResourceManager` class can be initialized only once. As a consequence, you cannot use both `StreamingAssetResourceManager` and `AssetBundleResourceManager` in one application.

Now, let's get back to the code.

After trial and error, we find that we need to prepare files `face_detection_short_range.tflite` and `face_landmark_with_attention.tflite`.\
Unity does not support [`.tflite`](https://docs.unity3d.com/Manual/class-TextAsset.html)  extension, so this plugin adopts the `.bytes` extension instead.

> :bell: A `.bytes` file has the same contents as the corresponding `.tflite` file, just with a different extension.

Now the entire code will look like this.

```cs
using System.Collections;
using UnityEngine;
using UnityEngine.UI;

namespace Mediapipe.Unity.Tutorial
{
  public class FaceMesh : MonoBehaviour
  {
    [SerializeField] private TextAsset _configAsset;
    [SerializeField] private RawImage _screen;
    [SerializeField] private int _width;
    [SerializeField] private int _height;
    [SerializeField] private int _fps;

    private CalculatorGraph _graph;
    private ResourceManager _resourceManager;

    private WebCamTexture _webCamTexture;
    private Texture2D _inputTexture;
    private Color32[] _pixelData;

    private IEnumerator Start()
    {
      if (WebCamTexture.devices.Length == 0)
      {
        throw new System.Exception("Web Camera devices are not found");
      }
      var webCamDevice = WebCamTexture.devices[0];
      _webCamTexture = new WebCamTexture(webCamDevice.name, _width, _height, _fps);
      _webCamTexture.Play();

      yield return new WaitUntil(() => _webCamTexture.width > 16);

      _screen.rectTransform.sizeDelta = new Vector2(_width, _height);
      _screen.texture = _webCamTexture;

      _inputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
      _pixelData = new Color32[_width * _height];

      _resourceManager = new LocalResourceManager();
      yield return _resourceManager.PrepareAssetAsync("face_detection_short_range.bytes");
      yield return _resourceManager.PrepareAssetAsync("face_landmark_with_attention.bytes");

      _graph = new CalculatorGraph(_configAsset.text);
      _graph.StartRun().AssertOk();

      while (true)
      {
        _inputTexture.SetPixels32(_webCamTexture.GetPixels32(_pixelData));
        var imageFrame = new ImageFrame(ImageFormat.Types.Format.Srgba, _width, _height, _width * 4, _inputTexture.GetRawTextureData<byte>());
        _graph.AddPacketToInputStream("input_video", new ImageFramePacket(imageFrame)).AssertOk();

        yield return new WaitForEndOfFrame();
      }
    }

    private void OnDestroy()
    {
      if (_webCamTexture != null)
      {
        _webCamTexture.Stop();
      }

      if (_graph != null)
      {
        try
        {
          _graph.CloseInputStream("input_video").AssertOk();
          _graph.WaitUntilDone().AssertOk();
        }
        finally
        {

          _graph.Dispose();
        }
      }
    }
  }
}
```

Run the scene to see the result

![face-mesh-resource-loaded](https://user-images.githubusercontent.com/4690128/163800451-32598fec-4eea-430f-9281-746e3146bce7.png)

As the result, the error shows the timestamp needs to be corrected.

### Set the correct timestamp
In the Hello World example, the loop variable `i` was set to the value of `Timestamp`.\
In practice, however, MediaPipe assumes that the value of `Timestamp` is in microseconds ([mediapipe/framework/timestamp.h](https://github.com/google/mediapipe/blob/c6c80c37452d0938b1577bd1ad44ad096ca918e0/mediapipe/framework/timestamp.h#L85-L87)).

> :bell: There are calculators that care about the absolute value of the `Timestamp`. Such calculators will behave unintentionally if the timestamp value is not in microseconds.

Let's initialize a `Timestamp` with a microsecond value from the start.

```cs
using Stopwatch = System.Diagnostics.Stopwatch;

var stopwatch = new Stopwatch();
stopwatch.Start();

var currentTimestamp = stopwatch.ElapsedTicks / (System.TimeSpan.TicksPerMillisecond / 1000);
var timestamp = new Timestamp(currentTimestamp);
```

And the entire code:

```cs
using System.Collections;
using UnityEngine;
using UnityEngine.UI;

using Stopwatch = System.Diagnostics.Stopwatch;

namespace Mediapipe.Unity.Tutorial
{
  public class FaceMesh : MonoBehaviour
  {
    [SerializeField] private TextAsset _configAsset;
    [SerializeField] private RawImage _screen;
    [SerializeField] private int _width;
    [SerializeField] private int _height;
    [SerializeField] private int _fps;

    private CalculatorGraph _graph;
    private ResourceManager _resourceManager;

    private WebCamTexture _webCamTexture;
    private Texture2D _inputTexture;
    private Color32[] _pixelData;

    private IEnumerator Start()
    {
      if (WebCamTexture.devices.Length == 0)
      {
        throw new System.Exception("Web Camera devices are not found");
      }
      var webCamDevice = WebCamTexture.devices[0];
      _webCamTexture = new WebCamTexture(webCamDevice.name, _width, _height, _fps);
      _webCamTexture.Play();

      yield return new WaitUntil(() => _webCamTexture.width > 16);

      _screen.rectTransform.sizeDelta = new Vector2(_width, _height);
      _screen.texture = _webCamTexture;

      _inputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
      _pixelData = new Color32[_width * _height];

      _resourceManager = new LocalResourceManager();
      yield return _resourceManager.PrepareAssetAsync("face_detection_short_range.bytes");
      yield return _resourceManager.PrepareAssetAsync("face_landmark_with_attention.bytes");

      var stopwatch = new Stopwatch();

      _graph = new CalculatorGraph(_configAsset.text);
      _graph.StartRun().AssertOk();
      stopwatch.Start();

      while (true)
      {
        _inputTexture.SetPixels32(_webCamTexture.GetPixels32(_pixelData));
        var imageFrame = new ImageFrame(ImageFormat.Types.Format.Srgba, _width, _height, _width * 4, _inputTexture.GetRawTextureData<byte>());
        var currentTimestamp = stopwatch.ElapsedTicks / (System.TimeSpan.TicksPerMillisecond / 1000);
        _graph.AddPacketToInputStream("input_video", new ImageFramePacket(imageFrame, new Timestamp(currentTimestamp))).AssertOk();

        yield return new WaitForEndOfFrame();
      }
    }

    private void OnDestroy()
    {
      if (_webCamTexture != null)
      {
        _webCamTexture.Stop();
      }

      if (_graph != null)
      {
        try
        {
          _graph.CloseInputStream("input_video").AssertOk();
          _graph.WaitUntilDone().AssertOk();
        }
        finally
        {

          _graph.Dispose();
        }
      }
    }
  }
}
```

![face-mesh-timestamp](https://user-images.githubusercontent.com/4690128/163901150-53a754a8-d7ce-4ad4-9273-be8a0e495338.png)



### Get ImageFrame
In the Hello World example, we initialized `OutputStreamPoller` using `CalculatorGraph#AddOutputStreamPoller`.\
This time, to handle output more easily, let's use the **OutputStream API** provided by the plugin instead!

```cs
var graph = new CalculatorGraph(_configAsset.text);
var outputVideoStream = new OutputStreasm<ImageFramePacket, ImageFrame>(graph, "output_video");
```

This may sound a bit tedious, but both `Packet` type and value type of output must be specified.\
And before running the `CalculatorGraph`, call `StartPolling`.

```cs
// NOTE: StartPolling returns Status
outputVideoStream.StartPolling().AssertOk();
_graph.StartRun().AssertOk();
```

To get the next output, call `TryNext`.\
It returns `true` if the next output is retrieved successfully.

```cs
if (outputVideoStream.TryGetNext(out var outputVideo))
{
  // ...
}
```

This time, let's display the output image directly on the screen.\
We can read the pixel data using `ImageFrame#TryReadPixelData`.

```cs
// NOTE: TryReadPixelData is implemented in `Mediapipe.Unity.ImageFrameExtension`.
// using Mediapipe.Unity;

var outputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
var outputPixelData = new Color32[_width * _height];
_screen.texture = outputTexture;

if (outputVideoStream.TryGetNext(out var outputVideo))
{
  if (outputVideo.TryReadPixelData(outputPixelData))
  {
    outputTexture.SetPixels32(outputPixelData);
    outputTexture.Apply();
  }
}
```

Now our code should look something like this.

```cs
using System.Collections;
using UnityEngine;
using UnityEngine.UI;

using Stopwatch = System.Diagnostics.Stopwatch;

namespace Mediapipe.Unity.Tutorial
{
  public class FaceMesh : MonoBehaviour
  {
    [SerializeField] private TextAsset _configAsset;
    [SerializeField] private RawImage _screen;
    [SerializeField] private int _width;
    [SerializeField] private int _height;
    [SerializeField] private int _fps;

    private CalculatorGraph _graph;
    private ResourceManager _resourceManager;

    private WebCamTexture _webCamTexture;
    private Texture2D _inputTexture;
    private Color32[] _inputPixelData;
    private Texture2D _outputTexture;
    private Color32[] _outputPixelData;

    private IEnumerator Start()
    {
      if (WebCamTexture.devices.Length == 0)
      {
        throw new System.Exception("Web Camera devices are not found");
      }
      var webCamDevice = WebCamTexture.devices[0];
      _webCamTexture = new WebCamTexture(webCamDevice.name, _width, _height, _fps);
      _webCamTexture.Play();

      yield return new WaitUntil(() => _webCamTexture.width > 16);

      _screen.rectTransform.sizeDelta = new Vector2(_width, _height);

      _inputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
      _inputPixelData = new Color32[_width * _height];
      _outputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
      _outputPixelData = new Color32[_width * _height];

      _screen.texture = _outputTexture;

      _resourceManager = new LocalResourceManager();
      yield return _resourceManager.PrepareAssetAsync("face_detection_short_range.bytes");
      yield return _resourceManager.PrepareAssetAsync("face_landmark_with_attention.bytes");

      var stopwatch = new Stopwatch();

      _graph = new CalculatorGraph(_configAsset.text);
      var outputVideoStream = new OutputStream<ImageFramePacket, ImageFrame>(_graph, "output_video");
      outputVideoStream.StartPolling().AssertOk();
      _graph.StartRun().AssertOk();
      stopwatch.Start();

      while (true)
      {
        _inputTexture.SetPixels32(_webCamTexture.GetPixels32(_inputPixelData));
        var imageFrame = new ImageFrame(ImageFormat.Types.Format.Srgba, _width, _height, _width * 4, _inputTexture.GetRawTextureData<byte>());
        var currentTimestamp = stopwatch.ElapsedTicks / (System.TimeSpan.TicksPerMillisecond / 1000);
        _graph.AddPacketToInputStream("input_video", new ImageFramePacket(imageFrame, new Timestamp(currentTimestamp))).AssertOk();

        yield return new WaitForEndOfFrame();

        if (outputVideoStream.TryGetNext(out var outputVideo))
        {
          if (outputVideo.TryReadPixelData(_outputPixelData))
          {
            _outputTexture.SetPixels32(_outputPixelData);
            _outputTexture.Apply();
          }
        }
      }
    }

    private void OnDestroy()
    {
      if (_webCamTexture != null)
      {
        _webCamTexture.Stop();
      }

      if (_graph != null)
      {
        try
        {
          _graph.CloseInputStream("input_video").AssertOk();
          _graph.WaitUntilDone().AssertOk();
        }
        finally
        {

          _graph.Dispose();
        }
      }
    }
  }
}
```

Let's try running!

![face-mesh-upside-down](https://user-images.githubusercontent.com/4690128/163911390-af91cea2-c020-4cb0-a8ae-9fdcfadfefb6.png)

### Coordinate System
In Unity, the pixel data is stored from bottom-left to top-right, whereas MediaPipe assumes the pixel data is stored from top-left to bottom-right.\
Therefore, if you send the pixel data to MediaPipe as is, MediaPipe will receive an upside-down image.

> :bell: `ImageFrame#TryReadPixelData` automatically reads pixels upside down, so the output image is received correctly.

You can flip the input image vertically by yourself, but here we will use `ImageTransformationCalculator`.

```txt
node: {
  calculator: "ImageTransformationCalculator"
  input_stream: "IMAGE:throttled_input_video"
  output_stream: "IMAGE:transformed_input_video"
  node_options: {
    [type.googleapis.com/mediapipe.ImageTransformationCalculatorOptions] {
      flip_vertically: true
    }
  }
}
```

Don't forget to replace `throttled_input_video` with `transformed_input_video`.

```diff
 # Subgraph that detects faces and corresponding landmarks.
 node {
   calculator: "FaceLandmarkFrontCpu"
-  input_stream: "IMAGE:throttled_input_video"
+  input_stream: "IMAGE:transformed_input_video"
   input_side_packet: "NUM_FACES:num_faces"
   input_side_packet: "WITH_ATTENTION:with_attention"
   output_stream: "LANDMARKS:multi_face_landmarks"
```
```diff
 # Subgraph that renders face-landmark annotation onto the input image.
 node {
   calculator: "FaceRendererCpu"
-  input_stream: "IMAGE:throttled_input_video"
+  input_stream: "IMAGE:transformed_input_video"
   input_stream: "LANDMARKS:multi_face_landmarks"
   input_stream: "NORM_RECTS:face_rects_from_landmarks"
   input_stream: "DETECTIONS:face_detections"
```

This time it should work correctly.

![face-mesh-official](https://user-images.githubusercontent.com/4690128/163912868-cbfd86cd-2af7-435f-a208-15671fc46e4b.png)

### Get landmarks
This time, let's try to get landmark positions from the `"multi_face_landmarks"` stream.

The output type is `List<NormalizedLandmarkList>` (`std::vector<NormalizedLandmarkList>` in C++), so we can initialize the `OutputStream` like this.
```cs
var multiFaceLandmarksStream = new OutputStream<NormalizedLandmarkListVectorPacket, List<NormalizedLandmarkList>>(_graph, "multi_face_landmarks");
multiFaceLandmarksStream.StartPolling().AssertOk();
```

As with the `"output_video"` stream, we can receive the result using `OutputStream#TryGetNext`.
```cs
if (multiFaceLandmarksStream.TryGetNext(out var multiFaceLandmarks))
{
  // ...
}
```

Note that the output values are in the Image Coordinate System.\
To convert them to the Unity local coordinates, you can use methods defined in `ImageCoordinateSystem`.

> :bell: Which coordinate system the output is based on depends on the solution. For example, The Objectron Graph uses the Camera Coordinate System instead of the Image Coordinate System.

```cs
// using Mediapipe.Unity.CoordinateSystem;

var screenRect = _screen.GetComponent<RectTransform>().rect;
var position = screenRect.GetPoint(normalizedLandmark);
```

Here is the sample code and the result.
```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;
using Mediapipe.Unity.CoordinateSystem;

using Stopwatch = System.Diagnostics.Stopwatch;

namespace Mediapipe.Unity.Tutorial
{
  public class FaceMesh : MonoBehaviour
  {
    [SerializeField] private TextAsset _configAsset;
    [SerializeField] private RawImage _screen;
    [SerializeField] private int _width;
    [SerializeField] private int _height;
    [SerializeField] private int _fps;

    private CalculatorGraph _graph;
    private ResourceManager _resourceManager;

    private WebCamTexture _webCamTexture;
    private Texture2D _inputTexture;
    private Color32[] _inputPixelData;
    private Texture2D _outputTexture;
    private Color32[] _outputPixelData;

    private IEnumerator Start()
    {
      if (WebCamTexture.devices.Length == 0)
      {
        throw new System.Exception("Web Camera devices are not found");
      }
      var webCamDevice = WebCamTexture.devices[0];
      _webCamTexture = new WebCamTexture(webCamDevice.name, _width, _height, _fps);
      _webCamTexture.Play();

      yield return new WaitUntil(() => _webCamTexture.width > 16);

      _screen.rectTransform.sizeDelta = new Vector2(_width, _height);

      _inputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
      _inputPixelData = new Color32[_width * _height];
      _outputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
      _outputPixelData = new Color32[_width * _height];

      _screen.texture = _outputTexture;

      _resourceManager = new LocalResourceManager();
      yield return _resourceManager.PrepareAssetAsync("face_detection_short_range.bytes");
      yield return _resourceManager.PrepareAssetAsync("face_landmark_with_attention.bytes");

      var stopwatch = new Stopwatch();

      _graph = new CalculatorGraph(_configAsset.text);
      var outputVideoStream = new OutputStream<ImageFramePacket, ImageFrame>(_graph, "output_video");
      var multiFaceLandmarksStream = new OutputStream<NormalizedLandmarkListVectorPacket, List<NormalizedLandmarkList>>(_graph, "multi_face_landmarks");
      outputVideoStream.StartPolling().AssertOk();
      multiFaceLandmarksStream.StartPolling().AssertOk();
      _graph.StartRun().AssertOk();
      stopwatch.Start();

      var screenRect = _screen.GetComponent<RectTransform>().rect;

      while (true)
      {
        _inputTexture.SetPixels32(_webCamTexture.GetPixels32(_inputPixelData));
        var imageFrame = new ImageFrame(ImageFormat.Types.Format.Srgba, _width, _height, _width * 4, _inputTexture.GetRawTextureData<byte>());
        var currentTimestamp = stopwatch.ElapsedTicks / (System.TimeSpan.TicksPerMillisecond / 1000);
        _graph.AddPacketToInputStream("input_video", new ImageFramePacket(imageFrame, new Timestamp(currentTimestamp))).AssertOk();

        yield return new WaitForEndOfFrame();

        if (outputVideoStream.TryGetNext(out var outputVideo))
        {
          if (outputVideo.TryReadPixelData(_outputPixelData))
          {
            _outputTexture.SetPixels32(_outputPixelData);
            _outputTexture.Apply();
          }
        }

        if (multiFaceLandmarksStream.TryGetNext(out var multiFaceLandmarks))
        {
          if (multiFaceLandmarks != null && multiFaceLandmarks.Count > 0)
          {
            foreach (var landmarks in multiFaceLandmarks)
            {
              // top of the head
              var topOfHead = landmarks.Landmark[10];
              Debug.Log($"Unity Local Coordinates: {screenRect.GetPoint(topOfHead)}, Image Coordinates: {topOfHead}");
            }
          }
        }
      }
    }

    private void OnDestroy()
    {
      if (_webCamTexture != null)
      {
        _webCamTexture.Stop();
      }

      if (_graph != null)
      {
        try
        {
          _graph.CloseInputStream("input_video").AssertOk();
          _graph.WaitUntilDone().AssertOk();
        }
        finally
        {

          _graph.Dispose();
        }
      }
    }
  }
}
```

![face-mesh-get-landmarks](https://user-images.githubusercontent.com/4690128/168980348-f6cc6a17-b84c-4918-a8c9-77c22949be0d.png)

### Annotation
You may want to render annotations on the screen to verify that the graph is working correctly.\
This section describes how to do that using the plugin.

First, create an empty object (let us call it `Annotation Layer`) below the `Screen` object.

> :bell: If you want to render multiple annotations, this `Annotation Layer` object will be the parent of all those annotations. 

![face-mesh-create-annotation-layer](https://user-images.githubusercontent.com/4690128/168986273-5c90bf13-233d-4ace-922b-d72137931716.png)

Remove `RectTransform` from `Annotation Layer`, since it will be configured automatically later.
![face-mesh-remove-rect-transform](https://user-images.githubusercontent.com/4690128/168986604-4fe61077-500d-4708-93f1-f60a3efe25bf.png)

Second, drag and drop the `Multi FaceLandmarkList Annotation` object below `Annotation Layer`.

> :bell: Various annotation objects are placed under `Packages/com.github.homuler.mediapipe/Runtime/Objects`. Choose the appropriate one depending on the output type.

![face-mesh-drop-annotation](https://user-images.githubusercontent.com/4690128/168987307-b28d8282-b6b9-484d-9410-4f6bd917a1ac.png)

Finally, attach `MultiFaceLandmarkListAnnotationController` to `Annotation Layer` and set the child `Multi FaceLandmarkList Annotation` to the `Annotation` attribute.
![face-mesh-annotation-controller](https://user-images.githubusercontent.com/4690128/168988492-266b7556-fddd-4206-85a0-697f8fd22a0e.png)

Now you can render annotations using `MultiFaceLandmarkListAnnotationController`.

```cs
// [SerializeField] private MultiFaceLandmarkListAnnotationController _multiFaceLandmarksAnnotationController;

if (multiFaceLandmarksStream.TryGetNext(out var multiFaceLandmarks))
{
  _multiFaceLandmarksAnnotationController.DrawNow(multiFaceLandmarks);
}
else
{
  _multiFaceLandmarksAnnotationController.DrawNow(null);
}
```

It is no longer necessary to view the MediaPipe output image.\
After refactoring, the code should look like this.

```cs
using System.Collections;
using System.Collections.Generic;
using UnityEngine;
using UnityEngine.UI;

using Stopwatch = System.Diagnostics.Stopwatch;

namespace Mediapipe.Unity.Tutorial
{
  public class FaceMesh : MonoBehaviour
  {
    [SerializeField] private TextAsset _configAsset;
    [SerializeField] private RawImage _screen;
    [SerializeField] private int _width;
    [SerializeField] private int _height;
    [SerializeField] private int _fps;
    [SerializeField] private MultiFaceLandmarkListAnnotationController _multiFaceLandmarksAnnotationController;

    private CalculatorGraph _graph;
    private ResourceManager _resourceManager;

    private WebCamTexture _webCamTexture;
    private Texture2D _inputTexture;
    private Color32[] _inputPixelData;

    private IEnumerator Start()
    {
      if (WebCamTexture.devices.Length == 0)
      {
        throw new System.Exception("Web Camera devices are not found");
      }
      var webCamDevice = WebCamTexture.devices[0];
      _webCamTexture = new WebCamTexture(webCamDevice.name, _width, _height, _fps);
      _webCamTexture.Play();

      yield return new WaitUntil(() => _webCamTexture.width > 16);

      _screen.rectTransform.sizeDelta = new Vector2(_width, _height);

      _inputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
      _inputPixelData = new Color32[_width * _height];

      _screen.texture = _webCamTexture;

      _resourceManager = new LocalResourceManager();
      yield return _resourceManager.PrepareAssetAsync("face_detection_short_range.bytes");
      yield return _resourceManager.PrepareAssetAsync("face_landmark_with_attention.bytes");

      var stopwatch = new Stopwatch();

      _graph = new CalculatorGraph(_configAsset.text);
      var multiFaceLandmarksStream = new OutputStream<NormalizedLandmarkListVectorPacket, List<NormalizedLandmarkList>>(_graph, "multi_face_landmarks");
      multiFaceLandmarksStream.StartPolling().AssertOk();
      _graph.StartRun().AssertOk();
      stopwatch.Start();

      var screenRect = _screen.GetComponent<RectTransform>().rect;

      while (true)
      {
        _inputTexture.SetPixels32(_webCamTexture.GetPixels32(_inputPixelData));
        var imageFrame = new ImageFrame(ImageFormat.Types.Format.Srgba, _width, _height, _width * 4, _inputTexture.GetRawTextureData<byte>());
        var currentTimestamp = stopwatch.ElapsedTicks / (System.TimeSpan.TicksPerMillisecond / 1000);
        _graph.AddPacketToInputStream("input_video", new ImageFramePacket(imageFrame, new Timestamp(currentTimestamp))).AssertOk();

        yield return new WaitForEndOfFrame();

        if (multiFaceLandmarksStream.TryGetNext(out var multiFaceLandmarks))
        {
          _multiFaceLandmarksAnnotationController.DrawNow(multiFaceLandmarks);
        }
        else
        {
          _multiFaceLandmarksAnnotationController.DrawNow(null);
        }
      }
    }

    private void OnDestroy()
    {
      if (_webCamTexture != null)
      {
        _webCamTexture.Stop();
      }

      if (_graph != null)
      {
        try
        {
          _graph.CloseInputStream("input_video").AssertOk();
          _graph.WaitUntilDone().AssertOk();
        }
        finally
        {

          _graph.Dispose();
        }
      }
    }
  }
}
```

![face-mesh-annotation](https://user-images.githubusercontent.com/4690128/168990750-92cc31c4-b711-4eb5-a9cf-cccad45a7d62.png)

### More Tips
#### Logging
By default, the plugin outputs INFO level logs.\
You can change the log level at runtime.

```cs
Mediapipe.Unity.Logger.MinLogLevel = Logger.LogLevel.Debug;
```

##### Glog
MediaPipe uses [Google Logging Library(glog)](https://github.com/google/glog) internally.\
You can configure it by [setting flags](https://github.com/google/glog#setting-flags).

```cs
Glog.Logtostderr = true; // when true, log will be output to `Editor.log` / `Player.log` 
Glog.minLogLevel = 0; // output INFO logs
Glog.v = 3; // output more verbose logs
```

To enable those flags, call `Glog.Initialize`.

```cs
Glog.Initialize("MediaPipeUnityPlugin");
```

> :skull_and_crossbones: `Glog.Initialize` can be called only once. The second call will crash your application or UnityEditor.
>
> :bulb: If you look closely at `Editor.log`, you will notice the following warning log output.\
> ```txt
> WARNING: Logging before InitGoogleLogging() is written to STDERR
> ```
> To suppress it, you need to call `Glog.Initialize`.\
> However, without setting `true` to `Glog.Logtostderr`, glog won't output to `Editor.log` / `Player.log`.

#### Load ImageFrame fast
In [this section](https://github.com/Maton-Net/MediaPipe4Unity/wiki/Getting-Started#get-imageframe), we used `ImageFrame#TryReadPixelData` and `Texture2D#SetPixels32` to render the output image.

Using `Texture2D#LoadRawTextureData`, we can do it a little faster.

```cs
_outputTexture.LoadRawTextureData(outputVideo.MutablePixelData(), outputVideo.PixelDataSize());
_outputTexture.Apply();
```

In this case, the orientation of the image must be adjusted as well.

## Official Solution (GPU)
> :warning: To test the code in this section, you need to build native libraries with GPU enabled.

> :note: The code in this section is based on [Official Solution (CPU) - Get ImageFrame](https://github.com/Maton-Net/MediaPipe4Unity/wiki/Getting-Started#get-imageframe).

> :bulb: See [GPU Compute](https://github.com/Maton-Net/MediaPipe4Unity/wiki/API-Overview#gpu-compute) for more details.

When you built native libraries with GPU enabled, you need to initialize GPU resources before running the `CalculatorGraph`.

```cs
yield return GpuManager.Initialize();

if (!GpuManager.IsInitialized)
{
  throw new System.Exception("Failed to initialize GPU resources");
}

_graph = new CalculatorGraph(_configAsset.text);
_graph.SetGpuResources(GpuManager.GpuResources).AssertOk();
```

Note that you need to dispose of GPU resources when the program exits.
```cs
private void OnDestroy()
{
  GpuManager.Shutdown();
}
```

The rest is the same as in the case of the CPU.

Here is a sample code.
Before running, don't forget to set `face_mesh_desktop_live_gpu.txt` to `_configAsset`.

> :bell: `face_mesh_desktop_live_gpu.pbtxt` is saved as [`Tutorial/Official Solution/face_mesh_desktop_live_gpu.txt`](https://github.com/Maton-Net/MediaPipe4Unity/blob/be96374adcc895dc9f675241333862b22cf61107/Assets/MediaPipeUnity/Tutorial/Official%20Solution/face_mesh_desktop_live_gpu.txt).

```cs
using System.Collections;
using UnityEngine;
using UnityEngine.UI;

using Stopwatch = System.Diagnostics.Stopwatch;

namespace Mediapipe.Unity.Tutorial
{
  public class FaceMesh : MonoBehaviour
  {
    [SerializeField] private TextAsset _configAsset;
    [SerializeField] private RawImage _screen;
    [SerializeField] private int _width;
    [SerializeField] private int _height;
    [SerializeField] private int _fps;

    private CalculatorGraph _graph;
    private ResourceManager _resourceManager;

    private WebCamTexture _webCamTexture;
    private Texture2D _inputTexture;
    private Color32[] _inputPixelData;
    private Texture2D _outputTexture;
    private Color32[] _outputPixelData;

    private IEnumerator Start()
    {
      if (WebCamTexture.devices.Length == 0)
      {
        throw new System.Exception("Web Camera devices are not found");
      }
      var webCamDevice = WebCamTexture.devices[0];
      _webCamTexture = new WebCamTexture(webCamDevice.name, _width, _height, _fps);
      _webCamTexture.Play();

      yield return new WaitUntil(() => _webCamTexture.width > 16);
      yield return GpuManager.Initialize();

      if (!GpuManager.IsInitialized)
      {
        throw new System.Exception("Failed to initialize GPU resources");
      }

      _screen.rectTransform.sizeDelta = new Vector2(_width, _height);

      _inputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
      _inputPixelData = new Color32[_width * _height];
      _outputTexture = new Texture2D(_width, _height, TextureFormat.RGBA32, false);
      _outputPixelData = new Color32[_width * _height];

      _screen.texture = _outputTexture;

      _resourceManager = new LocalResourceManager();
      yield return _resourceManager.PrepareAssetAsync("face_detection_short_range.bytes");
      yield return _resourceManager.PrepareAssetAsync("face_landmark_with_attention.bytes");

      var stopwatch = new Stopwatch();

      _graph = new CalculatorGraph(_configAsset.text);
      _graph.SetGpuResources(GpuManager.GpuResources).AssertOk();

      var outputVideoStream = new OutputStream<ImageFramePacket, ImageFrame>(_graph, "output_video");
      outputVideoStream.StartPolling().AssertOk();
      _graph.StartRun().AssertOk();
      stopwatch.Start();

      while (true)
      {
        _inputTexture.SetPixels32(_webCamTexture.GetPixels32(_inputPixelData));
        var imageFrame = new ImageFrame(ImageFormat.Types.Format.Srgba, _width, _height, _width * 4, _inputTexture.GetRawTextureData<byte>());
        var currentTimestamp = stopwatch.ElapsedTicks / (System.TimeSpan.TicksPerMillisecond / 1000);
        _graph.AddPacketToInputStream("input_video", new ImageFramePacket(imageFrame, new Timestamp(currentTimestamp))).AssertOk();

        yield return new WaitForEndOfFrame();

        if (outputVideoStream.TryGetNext(out var outputVideo))
        {
          if (outputVideo.TryReadPixelData(_outputPixelData))
          {
            _outputTexture.SetPixels32(_outputPixelData);
            _outputTexture.Apply();
          }
        }
      }
    }

    private void OnDestroy()
    {
      if (_webCamTexture != null)
      {
        _webCamTexture.Stop();
      }

      if (_graph != null)
      {
        try
        {
          _graph.CloseInputStream("input_video").AssertOk();
          _graph.WaitUntilDone().AssertOk();
        }
        finally
        {

          _graph.Dispose();
        }
      }

      GpuManager.Shutdown();
    }
  }
}
```

Let's try to run the sample scene.
![face-mesh-gpu-packet-type-error](https://user-images.githubusercontent.com/4690128/169201626-5afb2c37-25d6-4cd6-94e1-da048c362e0d.png)

```txt
MediaPipeException: INVALID_ARGUMENT: Graph has errors: 
Packet type mismatch on calculator outputting to stream "input_video": The Packet stores "mediapipe::ImageFrame", but "mediapipe::GpuBuffer" was requested.
  at Mediapipe.Status.AssertOk () [0x00014] in /home/homuler/Development/unity/MediaPipeUnityPlugin/Packages/com.github.homuler.mediapipe/Runtime/Scripts/Framework/Port/Status.cs:149 
  at Mediapipe.Unity.Tutorial.FaceMesh+<Start>d__12.MoveNext () [0x00281] in /home/homuler/Development/unity/MediaPipeUnityPlugin/Assets/MediaPipeUnity/Tutorial/Official Solution/FaceMesh.cs:67 
  at UnityEngine.SetupCoroutine.InvokeMoveNext (System.Collections.IEnumerator enumerator, System.IntPtr returnValueAddress) [0x00020] in /home/bokken/buildslave/unity/buil
d/Runtime/Export/Scripting/Coroutines.cs:17 
```

It seems that the official solution graph expects `mediapipe::GpuBuffer`, but we're putting `mediapipe::ImageFrame` to the input stream.\
So let's convert the input using `ImageFrameToGpuBufferCalculator`.

```txt
node: {
  calculator: "ImageFrameToGpuBufferCalculator"
  input_stream: "throttled_input_video"
  output_stream: "throttled_input_video_gpu"
}
```
We also need to convert the output from `mediapipe::GpuBuffer` to `mediapipe::ImageFrame`.

```txt
node: {
  calculator: "GpuBufferToImageFrameCalculator"
  input_stream: "output_video_gpu"
  output_stream: "output_video"
}
```

> :bell: You need to change some `Calculator` inputs and outputs as well, but this is left as an exercise.

If everything is fine, the result would be like this.

![face-mesh-gpu](https://user-images.githubusercontent.com/4690128/169202939-2d517d24-51c0-4442-84e2-4dde2136441c.png)

See [Coordinate System](https://github.com/Maton-Net/MediaPipe4Unity/wiki/Getting-Started#coordinate-system) for information on how to correct the orientation of the image.
