#### Android 8.0适配
android 8.0适配首先将 targetSdkVersion 更新至26

android8.0行为变更 https://developer.android.google.cn/about/versions/oreo/android-8.0-changes 下面介绍其中几项的适配：
#####  1.系统属性 net.dns1、net.dns2、net.dns3 和 net.dns4 不再可用，此项变更可加强平台的隐私性
解决方法：

Now the only way to access the DNS server list is by using the Connectivity Manager though Java. This adds the necessary JNI code to use the Connectivity Manager and pull the DNS server list. The old way using __system_property_get with net.dns# remains for compatibilty.

Using the Connectivity Manager requires the ACCESS_NETWORK_STATE permission to be set on the app. Existing applications most likely are not setting this and keeping the previous method as a fallback will at the very least ensure those apps don't break on older versions of Android. They will need to add this permission for Android 8 compatibility.

```
import android.content.Context;
import android.net.ConnectivityManager;
import android.net.LinkProperties;
import android.net.Network;
import java.net.InetAddress;
import java.util.List;

ConnectivityManager cm = (ConnectivityManager)this.getApplicationContext()
.getSystemService(Context.CONNECTIVITY_SERVICE);
Network an = cm.getActiveNetwork();
LinkProperties lp = cm.getLinkProperties(an);
List<InetAddress> dns = lp.getDnsServers();
for (InetAddress ia: dns) {
   String ha = ia.getHostAddress();
}

对应的c语言实现参考以下commit：
https://github.com/c-ares/c-ares/commit/1abf13ed309b001868bc9c90507ab73939f6b9da#comments
``` 
##### 2. 悬浮窗适配
使用 SYSTEM_ALERT_WINDOW 权限的应用无法再使用以下窗口类型来在其他应用和系统窗口上方显示提醒窗口：

- TYPE_PHONE
- TYPE_PRIORITY_PHONE
- TYPE_SYSTEM_ALERT
- TYPE_SYSTEM_OVERLAY
- TYPE_SYSTEM_ERROR

相反，应用必须使用名为 TYPE_APPLICATION_OVERLAY 的新窗口类型。

使用 TYPE_APPLICATION_OVERLAY 窗口类型显示应用的提醒窗口时，请记住新窗口类型的以下特性：

- 应用的提醒窗口始终显示在状态栏和输入法等关键系统窗口的下面。
- 系统可以移动使用 TYPE_APPLICATION_OVERLAY窗口类型的窗口或调整其大小，以改善屏幕显示效果。
- 通过打开通知栏，用户可以访问设置来阻止应用显示使用 TYPE_APPLICATION_OVERLAY 窗口类型显示的提醒窗口。


