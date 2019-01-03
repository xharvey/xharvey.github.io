sdk接入过程中常遇到Andorid6.0已上一些权限获取不到，由于高版本系统对危险权限做了严格限定，对权限做了小结：

targetsdkversion：目标设备sdk版本，Android适配兼容版本
由于线上怒焰版本部分渠道targetsdkversion已经>=23,而渠道上架targetsdkversion只能升不能降(覆盖安装失败)。所以只能动态去获取，获取失败提示去设置中打开。一般sdk中会去申请定位、识别手机识别码、电话短信、读写外部存储等危险权限。

（Android6.0后权限分为普通权限和危险权限）
普通权限：配置在AndroidMnifest.xml中，安装时默认获得此权限，且用户无法主动取消此权限。
危险权限：同样配置在AndroidMnifest.xml中，一般sdk会申请很多危险权限（识别手机状态身份、读写收短信等）。


targetsdkversion ：相当于App兼容适配到的版本，Andoird6.0 API level 为23。
分以下情况：
- ①targetsdkversion < 23 ，用户手机系统低于6.0，安装时默认获得此权限，且用户无法主动取消此权限。
- ②targetsdkversion < 23 ，用户手机系统高于6.0，安装时默认获得此权限，且用户可以主动取消此权限。
- ③targetsdkversion >=23  ,  用户手机系统低于6.0，安装时默认获得此权限，且用户无法主动取消此权限。
- ④targetsdkversion >= 23 ，用户手机系统高于6.0 ,  安装时不会获得此权限，需要在运行时想用户申请此权限，
授权后可以主动取消此权限，但是如果申请时拒绝，系统会默认记住，不会在下次启动时再次申请。需要检测，
并在游戏启动时给与玩家提示去进行授权。

Android6.0后授权是以组的形式，如果已经对一组危险权限中的一个权限授权，其他权限系统会默认授权。
相关API：

//动态权限

```
private static final int MY_PERMISSIONS_REQUEST_READ_EXTERNAL_STORAGE = 1;
private static boolean showedPermissionSetting = false;
String[] permissions = new String[]{
        Manifest.permission.WRITE_EXTERNAL_STORAGE,
        Manifest.permission.READ_PHONE_STATE,
        Manifest.permission.ACCESS_FINE_LOCATION
};
List<String> mPermissionList = new ArrayList<String>();
```


//判断API level：

```
if(Build.VERSION.SDK_INT >= 23) { ... }
```


//检查权限：

```
public void checkPrmission()
{
    if(Build.VERSION.SDK_INT >= 23)
    {
        mPermissionList.clear();
        for (int i = 0; i < permissions.length; i++) {
            if (ContextCompat.checkSelfPermission(this, permissions[i]) != PackageManager.PERMISSION_GRANTED) {
                mPermissionList.add(permissions[i]);
            }
        }
        if (mPermissionList.isEmpty()) {
            //未授予的权限为空，表示都授予了

        } else {//请求权限方法
            String[] permissions = mPermissionList.toArray(new String[mPermissionList.size()]);//将List转为数组
            ActivityCompat.requestPermissions(this, permissions, MY_PERMISSIONS_REQUEST_READ_EXTERNAL_STORAGE);
        }
    }

}
```


//申请授权：

```
ActivityCompat.requestPermissions(thisActivity,
                new String[]{Manifest.permission.READ_CONTACTS},
                MY_PERMISSIONS_REQUEST_READ_CONTACTS);
```


//处理申请授权回调

```
@Override
@Override
public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
    if (requestCode == MY_PERMISSIONS_REQUEST_READ_EXTERNAL_STORAGE) {
        for (int i = 0; i < grantResults.length; i++) {
            if (grantResults[i] != PackageManager.PERMISSION_GRANTED) {
                //判断是否勾选禁止后不再询问
                boolean showRequestPermission = ActivityCompat.shouldShowRequestPermissionRationale(this, permissions[i]);
                if (showRequestPermission && !showedPermissionSetting) {
                    showedPermissionSetting = true;
                    new AlertDialog.Builder(this)
                        .setMessage("请在设置中打开存储、读取本机识别码和位置权限")
                        .setPositiveButton("设置", new DialogInterface.OnClickListener() {
                            @Override
                            public void onClick(DialogInterface dialogInterface, int i) {
                                Intent intent = new Intent(Settings.ACTION_APPLICATION_DETAILS_SETTINGS);
                                intent.setData(Uri.parse("package:" + getPackageName()));
                                startActivity(intent);
                            }
                        })
                        .setNegativeButton("取消", null)
                        .create()
                        .show();
                }
            }
        }
    }
    super.onRequestPermissionsResult(requestCode, permissions, grantResults);
}
```


备注：
- ①应用宝渠道如果不授权READ_PHONE_STATE（识别手机状态和身份）会造成QQ登录失败，魅族等渠道遇到不授权游戏中造成crash。
- ②如果线上版本已经>= 23，降低targetsdkversion 会和降低versioncode一样，造成覆盖安装失败，不能通过降低targetsdkversion 来使用户默认获得必须的权限。