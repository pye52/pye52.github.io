---
title: webview上传文件的坑
date: 2016-12-07 18:07:41
tags:
- Android
- Webview
---

WebView是现阶段混合开发必不可少的组件，但由于google几乎每个版本都在修改其内核，导致有很多坑要踩。

<!-- more -->

直接上代码

参考网址[关于Webview拍照或从相册上传图片处理总结](http://blog.csdn.net/zouzhigang96/article/details/53331689)

```java
@Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        if (mUploadMessage == null && mUploadCallbackAboveL == null) {
            return;
        }

        switch (requestCode) {
            case RESULT_FILE_CHOOSE:
                Uri result = data == null || resultCode != RESULT_OK ? null : data.getData();
                if (mUploadCallbackAboveL != null) {
                    Uri[] results = null;
                    if (data != null) {
                        String dataString = data.getDataString();
                        ClipData clipData = data.getClipData();

                        if (clipData != null) {
                            results = new Uri[clipData.getItemCount()];
                            for (int i = 0; i < clipData.getItemCount(); i++) {
                                ClipData.Item item = clipData.getItemAt(i);
                                results[i] = item.getUri();
                            }
                        }

                        if (dataString != null) {
                            results = new Uri[]{Uri.parse(dataString)};
                        }
                    }

                    mUploadCallbackAboveL.onReceiveValue(results);
                    mUploadCallbackAboveL = null;
                } else if (mUploadMessage != null) {
                    mUploadMessage.onReceiveValue(result);
                    mUploadMessage = null;
                }
                break;
        }
    }

    public class MyWebChromeClient extends WebChromeClient {
        // For Android 3.0+
        public void openFileChooser(ValueCallback<Uri> uploadMsg) {
            mUploadMessage = uploadMsg;
            Intent i = new Intent(Intent.ACTION_GET_CONTENT);
            i.addCategory(Intent.CATEGORY_OPENABLE);
            i.setType("*/*");
            startActivityForResult(Intent.createChooser(i, "File Chooser"), RESULT_FILE_CHOOSE);
        }

        // For Android 3.0+
        public void openFileChooser( ValueCallback uploadMsg, String acceptType ) {
            mUploadMessage = uploadMsg;
            Intent i = new Intent(Intent.ACTION_GET_CONTENT);
            i.addCategory(Intent.CATEGORY_OPENABLE);
            i.setType("*/*");
            startActivityForResult(Intent.createChooser(i, "File Browser"), RESULT_FILE_CHOOSE);
        }
        //For Android 4.1
        public void openFileChooser(ValueCallback<Uri> uploadMsg, String acceptType, String capture){
            mUploadMessage = uploadMsg;
            Intent i = new Intent(Intent.ACTION_GET_CONTENT);
            i.addCategory(Intent.CATEGORY_OPENABLE);
            i.setType("image/*");
            startActivityForResult( Intent.createChooser( i, "File Browser" ), RESULT_FILE_CHOOSE );
        }
        // For Android 5.0+
        public boolean onShowFileChooser (WebView webView, ValueCallback<Uri[]> filePathCallback, WebChromeClient.FileChooserParams fileChooserParams) {
            mUploadCallbackAboveL = filePathCallback;
            Intent i = new Intent(Intent.ACTION_GET_CONTENT);
            i.addCategory(Intent.CATEGORY_OPENABLE);
            i.setType("image/*");
            startActivityForResult(Intent.createChooser(i, "File Browser"), RESULT_FILE_CHOOSE);
            return true;
        }
    }
```

