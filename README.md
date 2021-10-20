# Shizuku

## 背景

在开发需要root权限的应用程序时，最常用的方法是在su shell中运行一些命令。例如，使用“pm enable/disable”命令来启用/禁用组件。

此类做法的缺点在于：

1. **极慢** （会创建多个进程）
2. 需要处理文本来获取结果（**超级不可靠**）
3. 功能受制于可用的指令
4. 即使 adb 有足够权限，应用也要求 root 权限才可使用

Shizuku 采用了完全不同的方式。请参阅下面的详细说明。

## 用户指南 & 下载

<https://shizuku.rikka.app/>

## Shizuku 是如何工作的？

首先，我们需要谈谈应用是如何使用系统API的。例如，当应用想要获取已安装应用列表时，我们都知道应该使用 `PackageManager#getInstalledPackages()` 。这实际上是应用进程和系统服务进程间通信（IPC）的过程，只是Android框架为我们做了内部工作。

Android使用 `binder` 来完成这种类型的IPC。 `Binder` 允许服务端学习客户端的uid和pid，以便系统服务可以检查应用是否具有执行该操作的权限。

通常，如果有一个 "manager" （例如，`PackageManager`）供应用使用，那么系统服务进程中应该有一个 "service" （例如，`PackageManager`）。我们可以简单地认为，如果应用拥有此 "service" 的 `binder` ，它就可以与 "service" 进行通信。应用进程将在启动时接收系统服务的绑定（binders）。

Shizuku app 会引导用户使用 root 或是 adb 方式运行一个进程（Shizuku 服务进程）。当应用启动时，存在于 Shizuku 服务进程的 `binder` 也会发送至应用进程。

Shizuku 所提供的最重要的功能是充当中间人来从应用接收请求，再将它们发送到系统服务，然后返回结果。有关详细信息，您可以在 `rikka.shizuku.server.ShizukuService` 类和 `moe.shizuku.api.ShizukuBinderWrapper` 类中看到 `Transact-Remote` 方法。

So, we reached our goal, to use system APIs with higher permission. And to the app, it is almost identical to the use of system APIs directly.
至此，我们达到了以更高的权限使用系统API的目的。对于应用来说，它与直接调用系统 API 体验几乎一致。

## 开发者指南

### API & 实例

https://github.com/RikkaApps/Shizuku-API

### 从v11之前的版本迁移

> 当然，现有的应用仍然可以工作。

https://github.com/RikkaApps/Shizuku-API#migration-guide-for-existing-applications-use-shizuku-pre-v11

### 注意

1. ADB 权限依然有限

   ADB 仅有有限的权限并且在不同系统中有所区别。您可以在[此处](https://github.com/aosp-mirror/platform_frameworks_base/blob/master/packages/Shell/AndroidManifest.xml)查看ADB被授予的权限。

   在调用API之前，可以使用 `ShizukuService#getUid` 检查 Shizuku 是否在运行用户ADB，或者使用 `ShizukuService#checkPermission` 检查 Shizuku 服务是否具有足够的权限。

2. Android 9 隐藏API的使用限制

   从Android 9开始，普通应用程序对隐藏API的使用受到限制。请使用其他方法（例如 <https://github.com/LSPosed/AndroidHiddenApiBypass>）。

3. Android 8.0 & ADB

   目前，Shizuku 服务获取应用进程的方式是将 `IActivityManager#registerProcessObserver` 和 `IActivityManager#registerUidObserver` (26+) 结合起来，以确保在应用启动时获得应用进程。但是，在API 26上，ADB没有使用e `registerUidObserver`的权限，因此如果您需要在一个可能不是由 Activity（活动） 启动的进程中使用 Shizuku，建议通过启动一个透明活动来触发发送绑定。

4. 直接使用 `transactRemote` 需要注意的地方

   * 不同Android版本下的API可能不同，请务必仔细检查。此外， `android.app.IActivityManager` 在API 26以及更高版本中具有aidl表单，并且 `android.app.IActivityManager$Stub` 仅在API 26上存在。

   * `SystemServiceHelper.getTransactionCode` 可能无法获取正确的Transaction代码，例如 `android.content.pm.IPackageManager$Stub.TRANSACTION_getInstalledPackages` 在API 25上不存在，但存在 `android.content.pm.IPackageManager$Stub.TRANSACTION_getInstalledPackages_47` （这种情况已经得到处理，但不排除可能存在其他情况）。 `ShizukuBinderWrapper` 方法没有遇到这个问题。

## 开发 Shizuku
### Build

- 通过 `git clone --recurse-submodules` 克隆
- 运行 `:manager:assembleDebug` 或者 `:manager:assembleRelease` gradle 任务

`:manager:assembleDebug` 任务会生成一个可调试的服务。您可以将调试器附加到 `shizuku_server` 以调试 Shizuku 服务。

## License

此项目在 Apache-2.0 授权许可协议下可用。

### Exceptions

* **禁止**以任何方式使用下面列出的图像文件（除非用于展示 Shizuku 本身）。

  ```
  manager/src/main/res/mipmap-hdpi/ic_launcher.png
  manager/src/main/res/mipmap-hdpi/ic_launcher_background.png
  manager/src/main/res/mipmap-hdpi/ic_launcher_foreground.png
  manager/src/main/res/mipmap-xhdpi/ic_launcher.png
  manager/src/main/res/mipmap-xhdpi/ic_launcher_background.png
  manager/src/main/res/mipmap-xhdpi/ic_launcher_foreground.png
  manager/src/main/res/mipmap-xxhdpi/ic_launcher.png
  manager/src/main/res/mipmap-xxhdpi/ic_launcher_background.png
  manager/src/main/res/mipmap-xxhdpi/ic_launcher_foreground.png
  manager/src/main/res/mipmap-xxxhdpi/ic_launcher.png
  manager/src/main/res/mipmap-xxxhdpi/ic_launcher_background.png
  manager/src/main/res/mipmap-xxxhdpi/ic_launcher_foreground.png
  ```

* **禁止**将您自己编译的apk（包括修改过的，例如将 "Shizuku" 重命名）分发到任何商店（IBNLT Google Play商店等）。
