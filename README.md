AndroidUSBCamera
============   
AndroidUSBCamera is is a continuation of [jiangdongguo/AndroidUSBCamera](https://github.com/jiangdongguo/AndroidUSBCamera) that is in turn developed based on the [saki4510t/UVCCamera](https://github.com/saki4510t/UVCCamera). The project of USB Camera (UVC equipment) and the use of video data acquisition are highly packaged, and it can help developers using USB Camera devices easily by a few simple APIs. By using AndroidUSBCamera, you can detect and connect to a USB Camera with ease. You can also can use it to take pictures and recording mp4 videos. Some camera controls are available such as switching resolution, getting a h.264/aac/yuv(nv21) stream and setting camera's contrast and brightness.   

Supporting Android 5.0, 6.0, 7.0, 8.0, 9.0

Usage
-------
### 1.Add to your Android Studio project  

Step 1. Add the JitPack repository to your build file.Add it in your root build.gradle at the end of repositories:  
```java
allprojects {
		repositories {
			...
			maven { url 'http://raw.github.com/saki4510t/libcommon/master/repository/' }
			maven { url 'https://jitpack.io' }
		}
	}
```  
Step 2. Add the dependency  
```java
dependencies {
	       implementation 'com.github.jiangdongguo:AndroidUSBCamera:2.3.1'
}
```
### 2. APIs Introduction  
(1) In order to using it correctly,the following four steps must be achieved：  
```java
mUVCCameraView = (CameraViewInterface) mTextureView;
mUVCCameraView.setCallback(mCallback);
mCameraHelper = UVCCameraHelper.getInstance();
// set default preview size
 mCameraHelper.setDefaultPreviewSize(1280,720);
// set default frame format，defalut is UVCCameraHelper.Frame_FORMAT_MPEG
// if using mpeg can not record mp4,please try yuv
// mCameraHelper.setDefaultFrameFormat(UVCCameraHelper.FRAME_FORMAT_YUYV);	
mCameraHelper.initUSBMonitor(this, mUVCCameraView, mDevConnectListener); 
```
   To be attention,mCallback is a object of interface CameraViewInterface.Callback,and it's used to be listenering surfaceView
created or detoryed.mDevConnectListener is a object of interface UVCCameraHelper.OnMyDevConnectListener,and it's used to be listenering to detect and conntect USB device.Here is the coding order:  
```java
private CameraViewInterface.Callback mCallback = new CameraViewInterface.Callback mCallback(){
    @Override
    public void onSurfaceCreated(CameraViewInterface view, Surface surface) {
        // must have
        if (!isPreview && mCameraHelper.isCameraOpened()) {
            mCameraHelper.startPreview(mUVCCameraView);
            isPreview = true;
        }
    }

    @Override
    public void onSurfaceChanged(CameraViewInterface view, Surface surface, int width, int height) {

    }

    @Override
    public void onSurfaceDestroy(CameraViewInterface view, Surface surface) {
        // must have
        if (isPreview && mCameraHelper.isCameraOpened()) {
            mCameraHelper.stopPreview();
            isPreview = false;
        }
    }
}
private UVCCameraHelper.OnMyDevConnectListener listener = new UVCCameraHelper.OnMyDevConnectListener() {

        @Override
        public void onAttachDev(UsbDevice device) {
            // request open permission(must have)
            if (!isRequest) {
                isRequest = true;
                if (mCameraHelper != null) {
                    mCameraHelper.requestPermission(0);
                }
            }
        }

        @Override
        public void onDettachDev(UsbDevice device) {
            // close camera(must have)
            if (isRequest) {
                isRequest = false;
                mCameraHelper.closeCamera();
            }
        }

        @Override
        public void onConnectDev(UsbDevice device, boolean isConnected) {
       	
        }

        @Override
        public void onDisConnectDev(UsbDevice device) {
          
        }
    };
```
![Connecting gif](https://github.com/jiangdongguo/AndroidUSBCamera/blob/master/gifs/detecting.gif)  
(2) Capturing JPG Images  
```java
 mCameraHelper.capturePicture(picPath, new AbstractUVCCameraHandler.OnCaptureListener() {
                    @Override
                    public void onCaptureResult(String path) {
                        Log.i(TAG,"save path：" + path);
                    }
                }); 
```
(3) Recording Mp4,supporting close voice and save file automatic.
```java
RecordParams params = new RecordParams();
                    params.setRecordPath(videoPath);
                    params.setRecordDuration(0);                        // 0,do not cut save
                    params.setVoiceClose(mSwitchVoice.isChecked());    // is close voice
                    mCameraHelper.startPusher(params, new AbstractUVCCameraHandler.OnEncodeResultListener() {
                        @Override
                        public void onEncodeResult(byte[] data, int offset, int length, long timestamp, int type) {
                            // type = 1,h264 video stream
                            if (type == 1) {
//                                FileUtils.putFileStream(data, offset, length);
                            }
                            // type = 0,aac audio stream
                            if(type == 0) {

                            }
                        }

                        @Override
                        public void onRecordResult(String videoPath) {
                            Log.i(TAG,"videoPath = "+videoPath);
                        }
                    });  
// of course,if you only want to getting h.264 and aac stream
// you can do like this
mCameraHelper.startPusher(listener);
```
(4) setting camera's brightness and contrast.  
```java
mCameraHelper.setModelValue(UVCCameraHelper.MODE_BRIGHTNESS,progress);
mCameraHelper.setModelValue(UVCCameraHelper.MODE_CONTRAST,progress);
mCameraHelper.getModelValue(UVCCameraHelper.MODE_BRIGHTNESS);
mCameraHelper.getModelValue(UVCCameraHelper.MODE_CONTRAST);
...
```
![Connecting gif](https://github.com/jiangdongguo/AndroidUSBCamera/blob/master/gifs/brightness.gif)
(5) switch resolutions and camera.  
```java
mCameraHelper.updateResolution(widht, height);
```
![Connecting gif](https://github.com/jiangdongguo/AndroidUSBCamera/blob/master/gifs/2.1.0.gif)  
At last,remember adding permissions:  
```xml
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.RECORD_AUDIO" />  
```     

Other Library about Android Camera
-------
[OkCamera](https://github.com/jiangdongguo/OkCamera) Android Camera univsersally operation.  
[AndroidRecordMp4](https://github.com/jiangdongguo/AndroidRecordMp4) Using MediaCodec realize record mp4.  
[AndroidYuvOsd](https://github.com/jiangdongguo/AndroidYuvOsd) YUV data operation.  
[Lame4Mp3](https://github.com/jiangdongguo/Lame4Mp3) pcm to mp3 and pcm to aac.  

License
-------

    Copyright 2018 Jiangdongguo

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.
