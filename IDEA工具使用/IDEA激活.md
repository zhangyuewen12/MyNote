## . 下载激活补丁



下载激活用到的补丁文件。

所有补丁文件，我放到了公众号：**Java后端精选**

扫码关注后，发送关键字：**0125**。就可以下载了

下载之后进行解压，会看到如下的目录（**解压之后的目录不能移动，也不能删除！另外目录中不要出现中文**）

**解压之后的目录不能移动，也不能删除！**

**解压之后的目录不zx能移动，也不能删除！！**

**解压之后的目录不能移动，也不能删除！！！**

![image-20220307120143351](/Users/zyw/Library/Application Support/typora-user-images/image-20220307120143351.png)

相比较之前的激活方法，多了 scripts 和 vmoptions 文件

- scripts是放脚本文件的
- vmoptions是放配置文件的（直接会加载这个文件夹下的vmoptions，不用大家手动配置）

## 1. 执行脚本

打开scripts 文件，可以看到有以下几个脚本

![image-20220307120202231](/Users/zyw/Library/Application Support/typora-user-images/image-20220307120202231.png)

**vbs结尾的文件都是windows系统下使用的脚本**

**sh结尾的文件是在linux和mac系统下使用的脚本**

从命名上来看，有install，也有对应的uninstall，非常的贴心。



> windows系统区分多用户，所以作者也是提供了针对不同用户范围的vbs文件，

> 如果你只想要当前登录windows电脑的这个用户使用，那你就运行install-current-user.vbs脚本

> 如果你想要所有登录windows电脑的这个用户都能使用，那你就运行install-all-user.vbs脚本



以我当前Windows为例，双击执行` **install-all-users.vbs**` **，**点击确定

![image-20220307120230295](/Users/zyw/Library/Application Support/typora-user-images/image-20220307120230295.png)

这个脚本会自动配置vmoptions文件，所以耐心等待几秒钟，会弹出一个 Done的提示

![image-20220307120242485](/Users/zyw/Library/Application Support/typora-user-images/image-20220307120242485.png)

这个时候就代表执行成功了。



![image-20220307120322748](/Users/zyw/Library/Application Support/typora-user-images/image-20220307120322748.png)

永久激活、2088年等信息可以通过修改 `config-jetbrains`文件下`mymap.conf` 自定义

![image-20220307120355066](/Users/zyw/Library/Application Support/typora-user-images/image-20220307120355066.png)

## 3. 激活

这里分享两种激活方法

#### 激活方法1: 使用激活码（推荐）

也可以选择通过激活码的方式激活，这个方法可以自定义LicenseName 以及激活时间

前提还是要先执行上面的脚本！

我测试的时候使用的激活码

我测试的时候使用的激活码

也可以通过 http://idea.hicxy.com/  这个网站重新获取

```
A5rS7IHd3d8K3YWppaHVvwrdjb20iLCJhc3NpZ25lZU5hbWUiOiIiLCJhc3NpZ25lZUVtYWlsIjoiIiwibGljZW5zZVJlc3RyaWN0aW9uIjoiIiwiY2hlY2tDb25jdXJyZW50VXNlIjpmYWxzZSwicHJvZHVjdHMiOlt7ImNvZGUiOiJEUE4iLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IkRCIiwicGFpZFVwVG8iOiIyMDIyLTAyLTA0IiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJQUyIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiSUkiLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IlJTQyIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJHTyIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiRE0iLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IlJTRiIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJEUyIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiUEMiLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IlJDIiwicGFpZFVwVG8iOiIyMDIyLTAyLTA0IiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJDTCIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiV1MiLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IlJEIiwicGFpZFVwVG8iOiIyMDIyLTAyLTA0IiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJSUzAiLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IlJNIiwicGFpZFVwVG8iOiIyMDIyLTAyLTA0IiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJBQyIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjpmYWxzZX0seyJjb2RlIjoiUlNWIiwicGFpZFVwVG8iOiIyMDIyLTAyLTA0IiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IkRDIiwicGFpZFVwVG8iOiIyMDIyLTAyLTA0IiwiZXh0ZW5kZWQiOmZhbHNlfSx7ImNvZGUiOiJSU1UiLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6ZmFsc2V9LHsiY29kZSI6IkRQIiwicGFpZFVwVG8iOiIyMDIyLTAyLTA0IiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBEQiIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQV1MiLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUFNJIiwicGFpZFVwVG8iOiIyMDIyLTAyLTA0IiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBQUyIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQQ1dNUCIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQR08iLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUFBDIiwicGFpZFVwVG8iOiIyMDIyLTAyLTA0IiwiZXh0ZW5kZWQiOnRydWV9LHsiY29kZSI6IlBSQiIsInBhaWRVcFRvIjoiMjAyMi0wMi0wNCIsImV4dGVuZGVkIjp0cnVlfSx7ImNvZGUiOiJQU1ciLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6dHJ1ZX0seyJjb2RlIjoiUlMiLCJwYWlkVXBUbyI6IjIwMjItMDItMDQiLCJleHRlbmRlZCI6dHJ1ZX1dLCJtZXRhZGF0YSI6IjAxMjAyMjAxMDVQUEFNMDAwMDA1IiwiaGFzaCI6IjI5NjkzNjI3LzA6LTIwNjkyMjgzNDciLCJncmFjZVBlcmlvZERheXMiOjcsImF1dG9Qcm9sb25nYXRlZCI6ZmFsc2UsImlzQXV0b1Byb2xvbmdhdGVkIjpmYWxzZX0
```



将激活码复制粘贴到如下位置，显示绿色，就证明激活码有效.

![image-20220307131241003](/Users/zyw/Library/Application Support/typora-user-images/image-20220307131241003.png)

然后点击 `Activate`，之后出现如下界面



<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220307131301805.png" alt="image-20220307131301805" style="zoom:50%;" />



永久激活、2088年等信息可以通过修改 `config-jetbrains`文件下`mymap.conf` 自定义、

<img src="/Users/zyw/Library/Application Support/typora-user-images/image-20220307131322932.png" alt="image-20220307131322932" style="zoom:50%;" />

#### 激活方法2: 使用server

执行上面的脚本之后，选择 `License server`，输入 

https://jetbra.in

![image-20220307131041970](/Users/zyw/Library/Application Support/typora-user-images/image-20220307131041970.png)

然后点击 `Avtivate`

![image-20220307131056564](/Users/zyw/Library/Application Support/typora-user-images/image-20220307131056564.png)

如上图所示就成功激活了，只不过LicenseName信息使用的是这台电脑上当前登录的用户名.

## 4. 结束

以上两种都可以达到激活IDEA的目的，但是无论如何下载的补丁，解压之后就不能再移动，也不能删除！
