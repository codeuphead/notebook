# Android Java Config

## AS 的 vmoptions 配置

编辑 `.AndroidStuidoX.X/studio64.exe.vmoptions` 文件

```groovy
-Xms2048m
-Xmx2048m
-XX:MaxPermSize=512m
-XX:ReservedCodeCacheSize=512m
-XX:+UseConcMarkSweepGC
-Dfile.encoding=UTF-8
```

## gradle jvm 配置

`.gradle/gradle.properties`

```groovy
org.gradle.jvmargs=-Xms2048m -Xmx2048m -XX\:MaxPermSize\=512m -XX\:+HeapDumpOnOutOfMemoryError -Dfile.encoding\=UTF-8
```

## aapt 查看 apk 信息

`aapt d badging <apkfile> | find "package"`

```bash
adb shell pm list packages
adb shell pm path [pakcage name]
adb pull
aapt dump badging xxx.apk
```

## monkey 命令

`adb shell monkey`

- 停止 monkey

```bash
adb shell ps | grep monkey
adb shell kill pid
```

- 参数

```bash
-p 指定包名
--pct-touch 指定触摸事件的百分比
--pct-motion 指定手势百分比
-s 随机种子数
--ignore-crashes
--ignore-timeouts
--throttle 指定动作间隔
```

## keytool 密钥管理

```bash
//生成密钥
keytool -genkey -v -keystore my-release-key.jks -keyalg RSA -keysize 2048 -validity 10000 -alias my-alias

keytool -list -v -keystore xxx
```

## zipalign

```bash
//zip 对齐，可以确保所有未压缩的数据的开头均相对于文件开头部分执行特定的字节对齐，这样可减少应用消耗的 RAM 量。
zipalign -v -p 4 my-app-unsigned.apk my-app-unsigned-aligned.apk

//验证 zip 对齐
zipalign -c -v 4 application.apk
```

```bash
zipalign [-f] [-v] <alignment> infile.apk outfile.apk

zipalign -c -v <alignment> existing.apk

-f : overwrite existing outfile.zip
-v : verbose output
-p : outfile.zip should use the same page alignment for all shared object files within infile.zip
-c : confirm the alignment of the given file
```

## apksigner

```bash
//签名 APK
apksigner sign --ks my-release-key.jks --out my-app-release.apk my-app-unsigned-aligned.apk

apksigner sign --key release.pk8 --cert release.x509.pem app.apk

//验证签名方式
apksigner verify <apk>

//验证是否 15 SDK min
apksigner verify --min-sdk-version 15 app.apk

apksigner sign
        + " --ks "
        + storeFile
        + " --ks-key-alias "
        + keyAlias
        + " --ks-pass pass:"
        + storePWD
        + " --key-pass pass:"
        + keyPWD
        + " --out "
        + signedFile
        + " "
        + alignedFile;
```

### 参考

```bash
--print-certs

//输出文件
--out <apk-filename>

//默认使用 manifest 中的
--min-sdk-version <integer>

//默认使用最高的
--max-sdk-version <integer>

//默认根据 --min-sdk-version 和 --max-sdk-version 来决定是否启用
--v1-signing-enabled <true|false>

//默认根据 --min-sdk-version 和 --max-sdk-version 来决定是否启用
--v2-signing-enabled <true|false>

--ks <filename>
--ks-key-alias <alias>
--ks-pass <input-format>
    pass:<passowrd>
    env:<name>
    file:<filename>
    stdin
--pass-encoding <charset>
--key-pass <input-format>
--ks-type <algorithm>
--ks-provider-name <name>
--ks-provider-class <class-name>
--ks-provider-arg <value>
--key <filename>
--cert <filename> //必须 X.509 PEM || DER 格式
```
