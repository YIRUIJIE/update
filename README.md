## [Andrioid开发] 实现一个简单通用的APP更新服务

- 已测试适用于安卓6、7、8、9、10。(11、12未测)
- 支持可json远程控制的强制更新和非强制更新。
- 支持浏览器下载和应用内直接下载并跳转安装。
- 支持进入界面检测升级和点击按钮检测升级。
- 支持根据自己需求自定义修改检测更新代码。

|名称|地址  |
|--|--|
|小Demo源码  | [https://gitee.com/yiruijie/update.git](https://gitee.com/yiruijie/update.git) |
|Update_V1.0.apk  | [https://gitee.com/yiruijie/update/attach_files/955256/download/Update_V1.0.apk](https://gitee.com/yiruijie/update/attach_files/955256/download/Update_V1.0.apk) |

### 一.更新弹窗效果图
#### 强制升级
	点击升级弹窗外的其他屏幕部分，不可取消升级弹窗。
	应用内下载完成后自动跳转安装，不可取消进度弹窗。
	点击浏览器下载时销毁软件界面，并直接跳转到浏览器下载。
#### 非强制升级
	点击升级弹窗外的其他屏幕部分，可取消弹窗。
![在这里插入图片描述](https://img-blog.csdnimg.cn/78bb2d4784d94da58f9234c41a03c054.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAeWlyag==,size_20,color_FFFFFF,t_70,g_se,x_16)
### 二. 搭建json格式网页及源码

```java
https://yirj.gitee.io/json1
```

```json
{
  "hasUpdate": true,  //是否有更新
  "NoIgnorable": false, //不可忽略的更新
  "versionCode": 2,   //服务端的版本号
  "versionName": "1.0.2", //服务端的版本名
  "updateLog": "\n1、系统弹窗样式升级。\n2、进入界面检测升级。\n3、点击按钮检测升级。\n4、Json远程控制升级。\n5、远程控制非/强制升级。\n6、应用内直接下载升级。\n7、跳转到浏览器下载升级。", //更新内容
  "apkUrl": "https://gitee.com/yiruijie/update/attach_files/955257/download/Update_V1.0.2.apk",//新版本下载的地址
  "webUrl":"https://gitee.com/yiruijie/update/attach_files/955257/download/Update_V1.0.2.apk",//浏览器下载APK的网址
  "apkSize": "2.61MB" //新版本的大小
}
```
```html
<!DOCTYPE html>
<html lang="en">
<head>
	<meta charset="UTF-8">
	<title>更新检查</title>
</head>
<body>
	<pre id="json"></pre>
</body>
<script type="text/javascript">
    let btn = document.querySelector('#json');
    let data =
    {
        "hasUpdate": true,  //是否有更新
        "NoIgnorable": false, //不可忽略的更新
        "versionCode": 2,   //服务端的版本号
        "versionName": "1.0.2", //服务端的版本名
        "updateLog": "\n1、系统弹窗样式升级。\n2、进入界面检测升级。\n3、点击按钮检测升级。\n4、Json远程控制升级。\n5、远程控制非/强制升级。\n6、应用内直接下载升级。\n7、跳转到浏览器下载升级。", //更新内容
        "apkUrl": "https://gitee.com/yiruijie/update/attach_files/955257/download/Update_V1.0.2.apk",//新版本下载的地址
        "webUrl":"https://gitee.com/yiruijie/update/attach_files/955257/download/Update_V1.0.2.apk",//浏览器下载APK的网址
        "apkSize": "2.61MB" //新版本的大小
    };
    btn.textContent = JSON.stringify(data, null, 2);
</script>
</html>
```
### 三、app/build.gradle添加
```java
dependencies {
    ......
    //app/build.gradle添加androidx支持
    implementation 'androidx.appcompat:appcompat:1.4.1'
}
```
### 四、gradle.properties
```java
//开启AndroidX的使用
android.useAndroidX=true
```
### 五、AndroidManifest.xml

```xml
<!-- 拥有完全的网络访问权限 -->
<uses-permission android:name="android.permission.INTERNET" />
<!-- 修改或删除您的USB存储设备中的内容 -->
<uses-permission android:name="android.permission.WRITE_EXTERNAL_STORAGE" />
<uses-permission android:name="android.permission.READ_EXTERNAL_STORAGE" />
<!-- 查看网络连接 -->
<uses-permission android:name="android.permission.ACCESS_NETWORK_STATE" />
<!--请求安装包权限-->
<uses-permission android:name="android.permission.REQUEST_INSTALL_PACKAGES"/>
<application
        android:allowBackup="true"
        android:icon="@drawable/ic_launcher"
        android:label="@string/app_name"
        android:theme="@style/AppTheme"
        android:requestLegacyExternalStorage="true">
        <provider
            android:name="androidx.core.content.FileProvider"
            android:authorities="${applicationId}.fileprovider"
            android:exported="false"
            android:grantUriPermissions="true">
            <meta-data
                android:name="android.support.FILE_PROVIDER_PATHS"
                android:resource="@xml/file_paths" />
        </provider>
    ............
</application>
```

### 六、创建xml/file_paths.xml文件

```xml
<?xml version="1.0" encoding="utf-8"?>
<paths>
    <external-path name="external_files" path="."/>
</paths>
```

### 七、主界面加入检测升级代码
```java
public class MainActivity extends AppCompatActivity {
    
    private static boolean hasUpdate, NoIgnorable = true; //是否有更新//不可忽略的更新
    private static int versionCode = 0;
    private static String versionName, updateLog, apkSize, apkUrl, webUrl = "";
    private static final String[] upl = new String[]{"https://yirj.gitee.io/json1111", "https://yirj.gitee.io/json1"};//无效 有效轮询


    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
        //解决网络安全异常问题
        StrictMode.ThreadPolicy policy = new StrictMode.ThreadPolicy.Builder().permitAll().build();
        StrictMode.setThreadPolicy(policy);
        //进入界面调用检查更新的方
        checkUpdate();
        //申请存储权限
        getReadPermissions();
    }

    //点击检查更新按钮
    public void BtnCheckUpdate(View view) {
        Toast.makeText(this, "点击了检查更新按钮", Toast.LENGTH_SHORT).show();
        //点击按钮检查更新
        checkUpdate();
    }

    //检查更新的方法
    private void checkUpdate() {
        try {
            GetServerJson();
            //如果检测本程序的版本号小于服务器的版本号，那么提示用户更新
            if (hasUpdate & getVersionCode() < versionCode) {
                showDialogUpdate();//弹出提示版本更新的对话框
            } else {
                Toast.makeText(this, "当前已是最新版本", Toast.LENGTH_LONG).show();
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

    /**
     * 获取当前使用的软件包的版本号
     */
    public int getVersionCode() {
        try {
            //获取packagemanager的实例
            PackageManager packageManager = getPackageManager();
            //getPackageName()是你当前类的包名，0代表是获取版本信息
            PackageInfo packInfo = packageManager.getPackageInfo(getPackageName(), 0);
            Log.e("TAG", "版本号" + packInfo.versionCode);  //更新软件用的是版本号
            return packInfo.versionCode;
        } catch (Exception e) {
            e.printStackTrace();
        }
        return 1;
    }

    /**
     * 提示版本更新的对话框
     */
    public void showDialogUpdate() {
        android.app.AlertDialog.Builder builder = new android.app.AlertDialog.Builder(this);
        if (NoIgnorable) {
            builder.setCancelable(false);//开启强制更新，无法关闭
        }

        builder.setTitle("是否升级到" + versionName + "版本？").
                //设置提示框的图标
                setIcon(R.mipmap.ic_launcher_round).
                setMessage("新版本大小：" + apkSize + "\n" + updateLog)
                .setPositiveButton("更新", (dialog, which) -> {
                    loadNewVersionProgress();//下载最新的版本程序
                })
                /*.setNegativeButton("取消", (dialog, which) -> {
                    //finish();
                })*/
                .setNeutralButton("浏览器下载", (dialog, which) -> {
                    Uri uri = Uri.parse(webUrl);
                    Intent intent = new Intent(Intent.ACTION_VIEW, uri);
                    startActivity(intent);
                    finish(); //销毁
                });
        // 生产对话框
        android.app.AlertDialog alertDialog = builder.create();
        // 显示对话框
        alertDialog.show();
    }

    /**
     * 获取服务端的json数据
     */
    public static void GetServerJson() {
        URL infoUrl;
        InputStream inStream;
        String line;
        try {
            String uurl = checkUrl(upl);//轮询验证可用url
            infoUrl = new URL(uurl);
            URLConnection connection = infoUrl.openConnection();
            HttpURLConnection httpConnection = (HttpURLConnection) connection;
            int responseCode = httpConnection.getResponseCode();
            if (responseCode == HttpURLConnection.HTTP_OK) {
                inStream = httpConnection.getInputStream();
                BufferedReader reader = new BufferedReader(new InputStreamReader(inStream, StandardCharsets.UTF_8));
                StringBuilder strber = new StringBuilder();
                while ((line = reader.readLine()) != null)
                    strber.append(line).append("\n");
                inStream.close();
                int start = strber.indexOf("{");
                int end = strber.indexOf("}");
                String json = strber.substring(start, end + 1);
                //System.out.println("更新JSON:\n" + json);
                try {
                    JSONObject jSONObject = new JSONObject(json);
                    hasUpdate = jSONObject.getBoolean("hasUpdate");
                    NoIgnorable = jSONObject.getBoolean("NoIgnorable");
                    versionCode = jSONObject.getInt("versionCode");
                    versionName = jSONObject.getString("versionName");
                    updateLog = jSONObject.getString("updateLog");
                    apkSize = jSONObject.getString("apkSize");
                    apkUrl = jSONObject.getString("apkUrl");
                    webUrl = jSONObject.getString("webUrl");
                } catch (JSONException e) {
                    e.printStackTrace();
                }
            }
        } catch (Exception e) {
            e.printStackTrace();
        }
    }

	/*
     * 验证Url 单例模式
     */
    public static String checkUrl(String[] ltl) {
        String resultUrl = null;
        for (String url : ltl) {
            resultUrl = url;
            try {
                URL infoUrl = new URL(url);
                URLConnection connection = infoUrl.openConnection();
                HttpURLConnection httpConnection = (HttpURLConnection) connection;
                int code = httpConnection.getResponseCode();
//                System.out.println("Code:" + code+" 链接："+resultUrl); //200 404
                if (code == 200) { //状态码为200证明网址正常，跳出for不再循环，如果不为200，就循环下一个网址。
                    break;
                }
            } catch (Exception e) {
                e.printStackTrace();
            }
        }
        return resultUrl;
    }

    /**
     * 下载新版本程序
     */
    private void loadNewVersionProgress() {
        final String uri = apkUrl;  //测试使用的下载APP直链
        final ProgressDialog pd;    //进度条对话框
        pd = new ProgressDialog(this);
        pd.setProgressStyle(ProgressDialog.STYLE_HORIZONTAL);
        pd.setMessage("下载最新版本到：" + Environment.getExternalStorageDirectory().getAbsolutePath() + "/" + apkUrl.substring(apkUrl.lastIndexOf("/") + 1));
        pd.setCancelable(false); //开启强制更新，触摸屏幕其他位置无法关闭
        pd.show();

        //启动子线程下载任务
        new Thread() {
            @Override
            public void run() {
                Looper.prepare();
                try {
                    File file = getFileFromServer(uri, pd);
                    installApk(file);
//                    pd.dismiss(); //结束掉进度条对话框
                } catch (Exception e) {
                    //下载apk失败
                    Toast.makeText(MainActivity.this, "下载更新失败：请使用浏览器下载", Toast.LENGTH_LONG).show();
                    e.printStackTrace();
                }
                Looper.loop();
            }
        }.start();
    }

    /**
     * 安装apk
     */
    protected void installApk(File file) {
        Intent intent = new Intent();
        intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
        intent.setAction(Intent.ACTION_VIEW);

        if (Build.VERSION.SDK_INT >= 24) {
            Uri apkUri = FileProvider.getUriForFile(this, getApplication().getPackageName() + ".fileprovider", file);
            intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            intent.addFlags(Intent.FLAG_GRANT_READ_URI_PERMISSION);
            intent.setDataAndType(apkUri, "application/vnd.android.package-archive");
        } else {
            intent.setFlags(Intent.FLAG_ACTIVITY_NEW_TASK);
            intent.setDataAndType(Uri.fromFile(file), "application/vnd.android.package-archive");
        }
        startActivity(intent);
    }


    /**
     * 从服务器获取apk文件的代码
     * 传入网址uri，进度条对象即可获得一个File文件
     * （要在子线程中执行哦）
     */
    public static File getFileFromServer(String uri, ProgressDialog pd) throws Exception {
        if (Environment.getExternalStorageState().equals(Environment.MEDIA_MOUNTED)) {
            URL url = new URL(uri);
            HttpURLConnection conn = (HttpURLConnection) url.openConnection();
            conn.setConnectTimeout(5000);
            //获取到文件的大小
            pd.setMax(conn.getContentLength());  //字节的方式显示下载进度
            InputStream is = conn.getInputStream();
            File file = new File(Environment.getExternalStorageDirectory(), apkUrl.substring(apkUrl.lastIndexOf("/", apkUrl.lastIndexOf("")) + 1));
            FileOutputStream fos = new FileOutputStream(file);
            BufferedInputStream bis = new BufferedInputStream(is);
            byte[] buffer = new byte[1024];
            int len;
            int total = 0;
            while ((len = bis.read(buffer)) != -1) {
                fos.write(buffer, 0, len);
                total += len;
                //获取当前下载量
                pd.setProgress(total);//字节方式显示下载量
            }
            fos.close();
            bis.close();
            is.close();
            return file;
        } else {
            return null;
        }
    }

    /**
     * 权限的验证及处理，相关方法
     */
    public void getReadPermissions() {
        if (Build.VERSION.SDK_INT >= Build.VERSION_CODES.M) {
            if (ContextCompat.checkSelfPermission(this, Manifest.permission.WRITE_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED ||
                    ContextCompat.checkSelfPermission(this, Manifest.permission.READ_EXTERNAL_STORAGE) != PackageManager.PERMISSION_GRANTED ||
                    ContextCompat.checkSelfPermission(this, Manifest.permission.REQUEST_INSTALL_PACKAGES) != PackageManager.PERMISSION_GRANTED) {
                ActivityCompat.requestPermissions(this,
                        new String[]{
                                Manifest.permission.WRITE_EXTERNAL_STORAGE,
                                Manifest.permission.READ_EXTERNAL_STORAGE,
                                Manifest.permission.REQUEST_INSTALL_PACKAGES}, 10001);
            }
        }
    }

}
```
