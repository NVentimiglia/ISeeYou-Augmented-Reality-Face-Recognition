# ISeeYou-Augmented-Reality-Face-Recognition
Face Recognition Augmented Reality in Unity3d using Metaio SDK

### 1) Get the Metaio SDK
Go install the METAIO SDK.
http://www.metaio.com/products/sdk/

### 2) Create a new project and include the SDK.

### 3) Add SDK assets

- Add a metaioSDK prefab and include your app key
- Set the tracking configuration to "FACE"
- Add a metaioTracker to the scene

### 4) Add a Tracker Camera Script

This script sits metaioTracker prefab. It is the only custom script in the project.

````
using System;
using System.Collections;
using System.IO;
using System.Threading;
using UnityEngine;

public class TrackerCamera : MonoBehaviour
{
    public Camera Cam;

    private float next;

    public float Delay = 5;

    public AudioSource Snap;

    void Awake()
    {
        enabled = false;
    }

    public void TrackerState(bool e)
    {
        enabled = e;
    }

    void Update()
    {
        if (next <= Time.timeSinceLevelLoad)
            TakePicture();
    }
  
    public void TakePicture()
    {
        var file = string.Format("{0}.png", DateTime.UtcNow.Ticks);
        var path = Path.Combine(Application.persistentDataPath, file);
        Debug.Log(path);

#if UNITY_ANDROID
        StartCoroutine(TakePictureAndroid(file));
#else
        Application.CaptureScreenshot(path);
#endif

        //feedback
        if (Snap)
            Snap.Play();

        //update counter
        next = Time.timeSinceLevelLoad + Delay;

        // stop camera
        enabled = false;
    }


    private IEnumerator TakePictureAndroid(string filePath)
    {
        //Wait for graphics to render
        yield return new WaitForEndOfFrame();

        //Create a texture to pass to encoding
        var texture = new Texture2D(Screen.width, Screen.height, TextureFormat.RGB24, false);

        //Put buffer into texture
        texture.ReadPixels(new Rect(0, 0, Screen.width, Screen.height), 0, 0);

        //Split the process
        yield return new WaitForEndOfFrame();


        // TODO Find a better way to save the images.
        // ThreadPool hangs the app. Like hard. 
        // Maybe add textures to a list and save in batch.

        // Use a plugin to save to the gallery on Android
        AndroidPlugin.SaveImageToGallery(texture, filePath);

    }
}

````

### 5) Add components to the tracker game object
- Include an audio source with your favorite snapshot sound
- Include a Quad facing the camera to outline the faces.
