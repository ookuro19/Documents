# Unity3D_OculusGO平台对接要点
所用的SDK版本可能偏旧，但授权验证方面暂无问题

## 1. SDK导入
导入时需要注意更新，所用到的物体主要是SDK prefabs的camera，其他基本就没有用到了。
射线检测及其显示所用材质等，需要从小米VR的SDK内引入，相应的还需要导入小米VR SDK的pointer input module和graphic raycast相关脚本。

## 2. 手柄操作
所用到的oculus go的手柄按键及其SDK内对应代码编号如表格所示（对应的enum位于SDK的OVRInput中的controller内的Button）：

|实体按键|SDK对应编号|
|:-: | :-:|
|圆盘键|PrimaryTouchpad|
|扳机键|PrimaryIndexTrigger|
|返回键|Two|

## 3.左右手互换问题
场景内不需要放置SDK内所提供的手柄，可以自建手柄后，通过SDK的OVRInput的一些列静态函数获取关于当前激活手柄的local postion,rotation等。
目前的方案是自建手柄，然后在update中获取激活的手柄的local postion,rotation并进行处理：如果需要显示手柄则直接同步，若显示手部，则需要将x赋值为0，这样手柄将不受系统设置影响。
```
// 获取当前激活的手柄
m_curControllerState = OVRInput.GetActiveController();

// 获取位置
this.transform.localPosition = OVRInput.GetLocalControllerPosition(m_curControllerState);

// 获取旋转
this.transform.localRotation = OVRInput.GetLocalControllerRotation(m_curControllerState);

```
## 4.关于返回按钮
SDK中判断返回按钮是否按下，其调用与其他按键相同，也是通过static get获取bool值，但返回按钮特殊性在于，当它被按下时，所返回的并非是true或false这样的值，具体有待验证，目前先直接调用。
还需要注意的是，oculus官方的审查标准中有一条，需要在按下返回按钮时能够暂停或返回上一级菜单。

## 5. 关于签名
oculus要求启用v1，禁用v2。可在unity中按照一般流程签名后，可参考如下链接实现：
https://blog.csdn.net/qq_32115439/article/details/55520012

所用到的工具是AndroidSDK build tools内的apksigner。其方法为：

1. 签名
```
    进入Android SDK/build-tools/SDK版本, 输入命令
    apksigner sign --ks 密钥库名 --ks-key-alias 密钥别名 xxx.apk

    若密钥库中有多个密钥对,则必须指定密钥别名
    apksigner sign --ks 密钥库名 --ks-key-alias 密钥别名 xxx.apk

    禁用V2签名
    apksigner sign --v2-signing-enabled false --ks 密钥库名 xxx.apk

    参数:
        --ks-key-alias       密钥别名,若密钥库有一个密钥对,则可省略,反之必选
        --v1-signing-enabled 是否开启V1签名,默认开启
        --v2-signing-enabled 是否开启V2签名,默认开启

    例如:
        在debug.keystore密钥库只有一个密钥对
        apksigner sign --ks debug.keystore MyApp.apk

        在debug.keystore密钥库中有多个密钥对,所以必须指定密钥别名
        apksigner sign --ks debug.keystore --ks-key-alias androiddebugkey MyApp.apk
```
2. 验证
```

    进入Android SDK/build-tools/SDK版本, 输入命令
    apksigner verify -v --print-certs xxx.apk

    参数:
        -v, --verbose 显示详情(显示是否使用V1和V2签名)
        --print-certs 显示签名证书信息

    例如:
        apksigner verify -v MyApp.apk

        Verifies
        Verified using v1 scheme (JAR signing): true
        Verified using v2 scheme (APK Signature Scheme v2): true
        Number of signers: 1
```

具体代码可复制：
1. 签名
```
.\apksigner.bat sign  --v2-signing-enabled false --ks F:\SVN\Skiing1.0_OculusGO\Sha
reInfo\hashTech.keystore F:\SVN\Skiing1.0_OculusGO\Builds\FancySKiing_OculusGO_Release.apk
```


2. 验证
```
.\apksigner.bat verify -v --print-certs F:\SVN\Skiing1.0_OculusGO\Builds\FancySKiin
g_OculusGO_Release.apk

```
