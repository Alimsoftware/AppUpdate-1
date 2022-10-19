# AppUpdate

In development, we often need to upgrade apk or download a file. So here is a general file download function **ZDown**. Through this article you will see
 - Common framework API interface design
 - The principle and implementation of multi-thread download
 - Background download, after the interface exits, come in and continue to display the principle of the download UI

[Please refer to this blog for the principle](https://blog.csdn.net/u011418943/article/details/85760069)



renderings
<table  align="center">

  <tr>
    <td><a href="url"><img src="https://github.com/LillteZheng/AppUpdate/blob/master/gif/update.gif" ></a></td>
  </tr>

</table>

## configure
```
allprojects {
    repositories {
    ...
    maven { url 'https://jitpack.io' }
    }
}
```
Then write ZDloader:
[![](https://jitpack.io/v/LillteZheng/AppUpdate.svg)](https://jitpack.io/#LillteZheng/AppUpdate)
```
implementation 'com.github.LillteZheng:AppUpdate:v1.4'
```

**Since frameworks such as retrofit and rxjava are used, you also need to add the following associations to your project, otherwise an error will be reported**

```
    implementation 'io.reactivex.rxjava2:rxjava:2.2.4'
    implementation 'io.reactivex.rxjava2:rxandroid:2.1.0'
    implementation 'com.alibaba:fastjson:1.1.70.android'
    implementation 'com.squareup.retrofit2:retrofit:2.4.0'
    implementation 'org.ligboy.retrofit2:converter-fastjson-android:2.1.0'
    implementation 'com.squareup.retrofit2:converter-scalars:2.4.0'
    implementation 'com.squareup.retrofit2:adapter-rxjava2:2.4.0'
```

## 1. Check the version

```
        ZDown.checkWith(this)
                .url(jsonUrlTest)
                .get()
                .listener(new CheckListener<TestBean>() {
                    @Override
                    public void onCheck(final TestBean data) {
                        Log.d(TAG, "zsr onCheck: " + data);
                     }

                    @Override
                    public void onFail(String errorMsg) {
                        Log.d(TAG, "zsr onFail: " + errorMsg);
                    }
                }).check();

```

In the listener, you can write the data to be converted. If you don't want to convert it into an entity bean, you can simply return the original string with String.class.
checkWith also supports writing parameters, using params(..) , supports get() and post()


After checking the version, you can use the following code to download the file：


```
ZDown.with(MainActivity.this)
    .url(fileUrlTest)
    //threads set to 3
    .threadCount(3)
    //ui refresh time is 500ms
    .reFreshTime(500)
    //The path to save the path, the default internal memory
    .filePath(mPath)
    //.allowBackDownload(true)  Whether to allow background updates

 //   .fileName("test.apk") The fileName defaults to root the connection to intercept, or you can write it yourself
    .listener(new TaskListener() {
        @Override
        public void onSuccess(String filePath, String md5Msg) {
            ZCommontUitls.installApk(MainActivity.this,filePath);
            dialog.dismiss();
        }

        @Override
        public void onDownloading(ZBean bean) {
            int progress = (int) bean.progress;
            updateBtn.setVisibility(View.GONE);
            progressBar.setVisibility(View.VISIBLE);
            progressBar.setProgress(progress);

        }

        @Override
        public void onFail(String errorMsg) {
            Log.d(TAG, "zsr onFail: " + errorMsg);
            dialog.dismiss();
        }
    }).down();
    
```

## 2. Confusion
It has been obfuscated internally, but if you use CheckListener to pass in a bean class, the bean class needs to be obfuscated by itself, eg:
```
-keep class com.zhengsr.appupdate.bean.** { *; }
```

ZDown is the program entry, it provides the following methods:

- pause() Pause task
- start() start task
- stopTask() Stop the task, if your apk update is not used in the activity, it is recommended to use this method when the app exits to prevent memory leaks
- stopTaskAndDeleteCache() Stop the task and delete the existing files and databases. When the task fails, you can use
- isTaskExists() Does the task exist
- isRunning() Is it downloading
- updateListener() Return from the background, if the task is downloading, just update the interface directly, the UI will not be messed up
- deleteCacheAndStart() When the task fails, you can use this to delete the cache files and data




