官方文档： http://facebook.github.io/react-native/docs/signed-apk-android.html#content

注意本文是在windows环境下测试通过。

### 生成一个签名秘钥
打开cmd, 定位到jdk的bin目录下，比如`C:\Program Files (x86)\Java\jdk1.8.0_31\bin`，运行下面的命令:
```
keytool -genkey -v -keystore my-release-key.keystore -alias my-key-alias -keyalg RSA -keysize 2048 -validity 10000
```
这条命令会要求你输入密钥库（keystore）和对应密钥的密码，然后设置一些发行相关的信息。最后它会生成一个叫做my-release-key.keystore的密钥库文件。在运行上面这条语句之后，密钥库里应该已经生成了一个单独的密钥，有效期为10000天。`--alias`参数后面的别名是你将来为应用签名时所需要用到的，所以记得记录这个别名。

### 设置gradle变量

1. 将`my-release-key.keystore`文件放到react native工程中的`android/app`文件夹下。
2. 编辑android文件夹下的`gradle.properties`,添加如下的代码（注意把其中的****替换为相应密码）
```
MYAPP_RELEASE_STORE_FILE=my-release-key.keystore
MYAPP_RELEASE_KEY_ALIAS=my-key-alias
MYAPP_RELEASE_STORE_PASSWORD=*****
MYAPP_RELEASE_KEY_PASSWORD=*****
```
如图

--img1

### 添加签名到应用的gradle配置文件

编辑你工程目录下的`android/app/build.gradle`，添加如下的内容：
```
...
android {
    ...
    defaultConfig { ... }
    signingConfigs {
        release {
            storeFile file(MYAPP_RELEASE_STORE_FILE)
            storePassword MYAPP_RELEASE_STORE_PASSWORD
            keyAlias MYAPP_RELEASE_KEY_ALIAS
            keyPassword MYAPP_RELEASE_KEY_PASSWORD
        }
    }
    buildTypes {
        release {
            ...
            signingConfig signingConfigs.release
        }
    }
}
...

### 生成发行APK包
如果你在`android/app`下有一个`react.gradle`

只要在终端下运行以下命令：
```
$ cd android && ./gradlew assembleRelease
```