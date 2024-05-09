## Unity Editor Setup

1. **Create a New Unity Project**: Open Unity Hub, create a new project, and select the Unity version.
2. **Import Base Scripts**: Copy `CameraManager.cs`, `ImageObfuscator.cs`, `ImgUtils.cs` and `ImageWriter.cs` into your project's `Assets` folder.
3. **Set Up Camera and UI**: Create a GameObject for the camera and add a `RawImage` component for display. Assign the `RawImage` to the `display` field in `CameraManager`.
4. **Configure Image Obfuscator**: Add `ImageObfuscator` to the camera GameObject and configure its parameters in the Inspector.
5. **Add YOLOv8 Model**: Import a compatible `onnx` YOLOv8 instance segmentation model and assign it to the `yoloV8Asset` field in `ImageObfuscator`.
6. **Configure Runtime Environment**: Ensure [Unity.Sentis](https://docs.unity3d.com/Packages/com.unity.sentis@1.2/manual/index.html) (version `1.2.0-exp.2` or greater) is installed using the Package Manager (com.unity.sentis). Then, adjust the device type and backend in the `ImageObfuscator` script to configure the runtime environment for your project.
7. **Configure Obfuscation Settings**: In CameraManager.cs, create a dictionary that maps class IDs to obfuscation types in the Start() method. Available obfuscation types include `Masking`, `Pixelation`, `Blurring`, or `None` (default value). Customize the settings for your use case by adding or removing entries.
For example:

   ```csharp
      obfuscationTypes = new Dictionary<int, Obfuscation.Type>
         {
         { 0, Obfuscation.Type.Masking },       // person
         { 1, Obfuscation.Type.Blurring},       // bicycle
         { 53, Obfuscation.Type.Pixelation },   // pizza
         { 67, Obfuscation.Type.Masking },      // cell phone
         // Add or remove entries as needed
         };
   ```
**Note**: The example assumes that the YOLOv8 model is trained on the COCO dataset. If your model is trained on a different dataset, you will need to check the ID correspondence for your particular case. For the COCO dataset, you can consult the class to ID mapping

8. **Test the System**: Run the project in Unity Editor to test the obfuscation functionality.
9. **Optimization**: Adjust parameters and optimize performance as needed.


## CameraManager Script

This script manages a camera feed and applies obfuscation to detected objects in real-time. It uses a WebCamTexture to capture video from the device's camera and a RawImage to display the obfuscated video.

#### How it works:

In the `Start()` method, the script initializes the camera and sets up an obfuscation mapping that associates object classes with obfuscation types:
```csharp
void Start()
{
InitializeCamera();

    obfuscationTypes = new Dictionary<int, Obfuscation.Type>
    {
        { 0, Obfuscation.Type.Masking },     // person
        { 1, Obfuscation.Type.Masking },     // bicycle
        { 53, Obfuscation.Type.Pixelation }, // pizza
        { 67, Obfuscation.Type.Masking },    // cell phone
    };
}
```
In the `Update()` method, the script checks if the camera is playing and if a new frame is available. If so, it runs the `imgObfuscator` to apply obfuscation to the detected objects in the frame:
```csharp
void Update()
   {
   if (webCamTexture.isPlaying && webCamTexture.didUpdateThisFrame)
   {
      obfuscatedTexture = imgObfuscator.Run(webCamTexture, obfuscationTypes); // This line is used for obfuscation
      display.texture = obfuscatedTexture;  // In this line the obfuscation is projected in RawImage
   }
}
```
## Repo Structure

```markdown
SafeARUnity/
├── Models/
│   ├── 🧠 yolov8n-seg.onnx       # YOLOv8 instance segmentation model: nano (13.8 Mb)
│   └── 🧠 yolov8s-seg_v2.onnx    # YOLOv8 instance segmentation model: small (47.5 Mb)
├── 📹 CamaraManager.cs           # Manages camera feed and obfuscation
├── 📜 coco80list.txt             # List of 80 COCO object categories
├── 📸 ImageWriter.cs             # Debug Class to check images
├── 🕵️‍♂️ ImgObfuscator.cs           # Obfuscation Class
├── 🛠️ ImgUtils.cs                # Image processing utility methods
└── 📖 README.md                  # Repository information and guide
```