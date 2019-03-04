---
layout: post
title:  Cookie持久化方案——PersistentCookieStore源码解读。
date:   2017-03-23 20:35:33 +0800
categories: Android
tag: [Cookie, PersistentCookieStore]
---

* content
{:toc}



### Cookie持久化方案——PersistentCookieStore源码解读。

客户端登陆之后一般都会在本地持有某个cookie，在退出登录时将这个cookie清理掉。如果Request的body体中持有这个cookie，服务器就会认为客户端的用户处于登录状态。反之，就会认为用户没有登录。

假设用户一直处于登录状态，如果他关闭了应用，那么他的登录状态应该保存起来。这样的话，在他下次打开应用时，他的状态还是登录状态，不需要再次登录。

如何实现呢？很简单，将有效的cookie保存起来，需要的时候拿出来，塞进请求里面就ok了。

这里已经有前人造好的轮子了——PersistentCookieStore，它是已经过时的async-http-client里面的一个类。虽然已经过时，不过它里面封装的PersistentCookieStore这个类可以自动实现cookie的本地化存储和管理。它可以将请求的response里面的set-cookie里面的数据保存到本地的preferences里面，在请求的时候会将有效的cookie携带上。

这里我们就不研究response的set-cookie是如何调用到PersistentCookieStore这个类了，我们研究下它是如何存储已经获取到的cookie的。

接下来看看这个类里面到底说了什么。先上代码，发现PersistentCookieStore类实现了CookieStore接口。

```
  public class PersistentCookieStore implements CookieStore {
	    ......
    }
```
CookieStore接口：

```
package org.apache.http.client;

import java.util.Date;
import java.util.List;
import org.apache.http.cookie.Cookie;

/** @deprecated */
@Deprecated
public interface CookieStore {
    void addCookie(Cookie var1);

    List<Cookie> getCookies();

    boolean clearExpired(Date var1);

    void clear();
}
```
这个接口是apache包下的，不过它已经被废弃了。先不管废弃不废弃，回归正题。这个接口内部有四个方法，分别是添加单个的Cookie，回去所有的Cookie列表，根据某个Date格式的时间清楚相应的Cookie，清空操作。PersistentCookieStore作为它的实现类，必须实现这几个方法，接下来看看是如何实现的。

#### PersistentCookieStore:
先来看一眼它的结构：
![PersistentCookieStore的Structure](https://tinytongtong-1255688482.cos.ap-beijing.myqcloud.com/PersistentCookieStore1.png)

##### 1、那么我们就从成员变量看起：

```
public class PersistentCookieStore implements CookieStore {
    private static final String LOG_TAG = "PersistentCookieStore";
    private static final String COOKIE_PREFS = "CookiePrefsFile";
    private static final String COOKIE_NAME_STORE = "names";
    private static final String COOKIE_NAME_PREFIX = "cookie_";
    private boolean omitNonPersistentCookies = false;
    private final ConcurrentHashMap<String, Cookie> cookies;
    private final SharedPreferences cookiePrefs;
    ......
}
```

解释一下这些参数：

LOG_TAG是这个类的tag标记；

COOKIE_PREFS是用来存储cookie的preferences的文件名；

COOKIE_NAME_STORE是preferences中存储cookie的name的key；

COOKIE_NAME_PREFIX是存储cookie时给name添加的前缀；

omitNonPersistentCookies这个参数是为了避免我们将非持久的cookie存储起来，默认为false，这个参数还提供了set方法；

cookies是一个ConcurrentHashMap对象，是Cookie的容器；

cookiePrefs是存储cookie的preferences对象。

##### 2、一般我们PersistentCookieStore是这么使用的，两行代码搞定。
```
PersistentCookieStore pcs = new PersistentCookieStore(this);
HttpClientUtils.setCookieStore(pcs);
```

接下来看构造方法：

```
public PersistentCookieStore(Context context) {
        this.cookiePrefs = context.getSharedPreferences("CookiePrefsFile", 0);
        this.cookies = new ConcurrentHashMap();
        String storedCookieNames = this.cookiePrefs.getString("names", (String)null);
        if(storedCookieNames != null) {
            String[] cookieNames = TextUtils.split(storedCookieNames, ",");
            String[] var4 = cookieNames;
            int var5 = cookieNames.length;

            for(int var6 = 0; var6 < var5; ++var6) {
                String name = var4[var6];
                String encodedCookie = this.cookiePrefs.getString("cookie_" + name, (String)null);
                if(encodedCookie != null) {
                    Cookie decodedCookie = this.decodeCookie(encodedCookie);
                    if(decodedCookie != null) {
                        this.cookies.put(name, decodedCookie);
                    }
                }
            }

            this.clearExpired(new Date());
        }

    }
```

①初始化一个preferences对象，文件名为CookiePrefsFile，mode为0，也就是PRIVETE_MODE.

②初始化cookies为一个空的ConcurrentHashMap.

③从preferences对象中获取key为“name”对应的字符串。

④如果获取到的name对应的字符串不为空，就使用TextUtils.split(storedCookieNames, ",")工具类将其拆分为String数组，然后遍历这个数组。

⑤在遍历String数组的过程中，会给每个String加前缀"cookie_"，然后以拼接后的字符串为Key，在preferences对象中获取相应的String类型的值。

⑥接下来对获取到的String进行解码，使用类里面定义的decodeCookie方法，这样就获取到了响应的Cookie对象了。

⑦然后将这个获取到的Cookie对象添加进cookies这个ConcurrentHashMap中。

⑧在循环完毕以后，调用 clearExpired(Date var1)方法，清理过期的cookie.

稍微总结下，在我们实例化PersistentCookieStore的过程中，我们将之前存储的Cookie对象全部存preferences中读取了出来，然后将他们添加进cookies这个ConcurrentHashMap中，最后调用clearExpired(Date var1)方法，清理过期的cookie。

##### 3、接下来我们重点PersistentCookieStore接口里面方法的具体实现：
###### ①void addCookie(Cookie var1);

```
public void addCookie(Cookie cookie) {
        if(!this.omitNonPersistentCookies || cookie.isPersistent()) {
            String name = cookie.getName() + cookie.getDomain();
            if(!cookie.isExpired(new Date())) {
                this.cookies.put(name, cookie);
            } else {
                this.cookies.remove(name);
            }

            Editor prefsWriter = this.cookiePrefs.edit();
            prefsWriter.putString("names", TextUtils.join(",", this.cookies.keySet()));
            prefsWriter.putString("cookie_" + name, this.encodeCookie(new SerializableCookie(cookie)));
            prefsWriter.commit();
        }
    }
```
如果omitNonPersistentCookies的值为false(它默认为false)，并且cookie.isPersistent()为true，我们就可以对它进行存储操作了。先将cookie的name和domain进行拼接，然后对cookie.isExpire(new Date())的值进行判断，如果为false（未过期），就存储进cookies这个map里面，存储的规则是以刚才拼接的name+domain为key，cookie对象为value；如果为true（已过期），就从cookies里面remove掉。然后将cookies这个map里面所有的数据存储进preferences，具体规则是将map中所有的key用","进行拼接，然后以"names"为key，以拼接好的String为value，存进preferences；接着通过encodeCookie方法将这个cookie对象编码为String类型，然后以"cookie_" + name为key存储进preferences。
		总的来说，addCookie（Cookie）方法的作用就是将合法有效的cookie，同时添加进map和preferences里面。
		
###### ②List<Cookie> getCookies()方法：

```
public List<Cookie> getCookies() {
        return new ArrayList(this.cookies.values());
    }
```
这个方法很直白，直接返回map中所有的cookie对象。

###### ③ boolean clearExpired(Date var1);

```
public boolean clearExpired(Date date) {
        boolean clearedAny = false;
        Editor prefsWriter = this.cookiePrefs.edit();
        Iterator var4 = this.cookies.entrySet().iterator();

        while(var4.hasNext()) {
            Entry entry = (Entry)var4.next();
            String name = (String)entry.getKey();
            Cookie cookie = (Cookie)entry.getValue();
            if(cookie.isExpired(date)) {
                this.cookies.remove(name);
                prefsWriter.remove("cookie_" + name);
                clearedAny = true;
            }
        }

        if(clearedAny) {
            prefsWriter.putString("names", TextUtils.join(",", this.cookies.keySet()));
        }

        prefsWriter.commit();
        return clearedAny;
    }
```
这个方法的中，遍历cookies这个map中所有的cookie，然后根据传入的Date参数清除掉map中所有的已过期的cookie，并同步更新preferences里面的数据。

###### ④void clear();这个方法也很直白，顾名思义，就是清楚掉所有的cookie.

```
public void clear() {
        Editor prefsWriter = this.cookiePrefs.edit();
        Iterator var2 = this.cookies.keySet().iterator();

        while(var2.hasNext()) {
            String name = (String)var2.next();
            prefsWriter.remove("cookie_" + name);
        }

        prefsWriter.remove("names");
        prefsWriter.commit();
        this.cookies.clear();
    }
```
将map和cookies里面所有的数据全部清空。

##### 4、接下来看看剩余的方法：
deleteCookie(Cookie cookie) --> 删除某个cookie.

encodeCookie(SerializableCookie cookie)、decodeCookie(String cookieString)、byteArrayToHexString(byte[] bytes)、hexStringToByteArray(String hexString)，他们都是对成双成对的，是对cookie进行编码和解码的操作。

###### ①deleteCookie(Cookie cookie) --> 删除某个cookie.
```
 public void deleteCookie(Cookie cookie) {
        String name = cookie.getName() + cookie.getDomain();
        this.cookies.remove(name);
        Editor prefsWriter = this.cookiePrefs.edit();
        prefsWriter.remove("cookie_" + name);
        prefsWriter.commit();
    }
```
###### ②encodeCookie(SerializableCookie cookie)和byteArrayToHexString(byte[] bytes)：将cookie对象转换成为十六进制的字节码

```
protected String encodeCookie(SerializableCookie cookie) {
        if(cookie == null) {
            return null;
        } else {
            ByteArrayOutputStream os = new ByteArrayOutputStream();

            try {
                ObjectOutputStream e = new ObjectOutputStream(os);
                e.writeObject(cookie);
            } catch (IOException var4) {
                Log.d("PersistentCookieStore", "IOException in encodeCookie", var4);
                return null;
            }

            return this.byteArrayToHexString(os.toByteArray());
        }
    }
```

```
protected String byteArrayToHexString(byte[] bytes) {
        StringBuilder sb = new StringBuilder(bytes.length * 2);
        byte[] var3 = bytes;
        int var4 = bytes.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            byte element = var3[var5];
            int v = element & 255;
            if(v < 16) {
                sb.append('0');
            }

            sb.append(Integer.toHexString(v));
        }

        return sb.toString().toUpperCase(Locale.US);
    }
```
###### ③decodeCookie(String cookieString)和hexStringToByteArray(String hexString)：将十六进制的字节码转换为cookie对象。

```
protected Cookie decodeCookie(String cookieString) {
        byte[] bytes = this.hexStringToByteArray(cookieString);
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
        Cookie cookie = null;

        try {
            ObjectInputStream e = new ObjectInputStream(byteArrayInputStream);
            cookie = ((SerializableCookie)e.readObject()).getCookie();
        } catch (IOException var6) {
            Log.d("PersistentCookieStore", "IOException in decodeCookie", var6);
        } catch (ClassNotFoundException var7) {
            Log.d("PersistentCookieStore", "ClassNotFoundException in decodeCookie", var7);
        }

        return cookie;
    }
```

```
protected byte[] hexStringToByteArray(String hexString) {
        int len = hexString.length();
        byte[] data = new byte[len / 2];

        for(int i = 0; i < len; i += 2) {
            data[i / 2] = (byte)((Character.digit(hexString.charAt(i), 16) << 4) + Character.digit(hexString.charAt(i + 1), 16));
        }

        return data;
    }
```

#### 总结
到现在为止，PersistentCookieStore里面所有的方法都介绍完了，这里再总结下。

①在初始化的时候，我们就可以将所有已经存储的cookie读取到内存中。

②我们可以通过addCookie(Cookie)方法将某个Cookie添加进cookies和preferences中。同理，我们可以调用deleteCookie(Cookie)方法删除某个Cookie.另外还有clear方法让我们情况所有cookie.

③cookie如何存储到preferences和如何读取到map中，他的编解码方式和存储规则很值得我们借鉴。

最后，附上PersistentCookieStore的源码：

```
package com.loopj.android.http;

import android.content.Context;
import android.content.SharedPreferences;
import android.content.SharedPreferences.Editor;
import android.text.TextUtils;
import android.util.Log;
import com.loopj.android.http.SerializableCookie;
import java.io.ByteArrayInputStream;
import java.io.ByteArrayOutputStream;
import java.io.IOException;
import java.io.ObjectInputStream;
import java.io.ObjectOutputStream;
import java.util.ArrayList;
import java.util.Date;
import java.util.Iterator;
import java.util.List;
import java.util.Locale;
import java.util.Map.Entry;
import java.util.concurrent.ConcurrentHashMap;
import org.apache.http.client.CookieStore;
import org.apache.http.cookie.Cookie;

public class PersistentCookieStore implements CookieStore {
    private static final String LOG_TAG = "PersistentCookieStore";
    private static final String COOKIE_PREFS = "CookiePrefsFile";
    private static final String COOKIE_NAME_STORE = "names";
    private static final String COOKIE_NAME_PREFIX = "cookie_";
    private boolean omitNonPersistentCookies = false;
    private final ConcurrentHashMap<String, Cookie> cookies;
    private final SharedPreferences cookiePrefs;

    public PersistentCookieStore(Context context) {
        this.cookiePrefs = context.getSharedPreferences("CookiePrefsFile", 0);
        this.cookies = new ConcurrentHashMap();
        String storedCookieNames = this.cookiePrefs.getString("names", (String)null);
        if(storedCookieNames != null) {
            String[] cookieNames = TextUtils.split(storedCookieNames, ",");
            String[] var4 = cookieNames;
            int var5 = cookieNames.length;

            for(int var6 = 0; var6 < var5; ++var6) {
                String name = var4[var6];
                String encodedCookie = this.cookiePrefs.getString("cookie_" + name, (String)null);
                if(encodedCookie != null) {
                    Cookie decodedCookie = this.decodeCookie(encodedCookie);
                    if(decodedCookie != null) {
                        this.cookies.put(name, decodedCookie);
                    }
                }
            }

            this.clearExpired(new Date());
        }

    }

    public void addCookie(Cookie cookie) {
        if(!this.omitNonPersistentCookies || cookie.isPersistent()) {
            String name = cookie.getName() + cookie.getDomain();
            if(!cookie.isExpired(new Date())) {
                this.cookies.put(name, cookie);
            } else {
                this.cookies.remove(name);
            }

            Editor prefsWriter = this.cookiePrefs.edit();
            prefsWriter.putString("names", TextUtils.join(",", this.cookies.keySet()));
            prefsWriter.putString("cookie_" + name, this.encodeCookie(new SerializableCookie(cookie)));
            prefsWriter.commit();
        }
    }

    public void clear() {
        Editor prefsWriter = this.cookiePrefs.edit();
        Iterator var2 = this.cookies.keySet().iterator();

        while(var2.hasNext()) {
            String name = (String)var2.next();
            prefsWriter.remove("cookie_" + name);
        }

        prefsWriter.remove("names");
        prefsWriter.commit();
        this.cookies.clear();
    }

    public boolean clearExpired(Date date) {
        boolean clearedAny = false;
        Editor prefsWriter = this.cookiePrefs.edit();
        Iterator var4 = this.cookies.entrySet().iterator();

        while(var4.hasNext()) {
            Entry entry = (Entry)var4.next();
            String name = (String)entry.getKey();
            Cookie cookie = (Cookie)entry.getValue();
            if(cookie.isExpired(date)) {
                this.cookies.remove(name);
                prefsWriter.remove("cookie_" + name);
                clearedAny = true;
            }
        }

        if(clearedAny) {
            prefsWriter.putString("names", TextUtils.join(",", this.cookies.keySet()));
        }

        prefsWriter.commit();
        return clearedAny;
    }

    public List<Cookie> getCookies() {
        return new ArrayList(this.cookies.values());
    }

    public void setOmitNonPersistentCookies(boolean omitNonPersistentCookies) {
        this.omitNonPersistentCookies = omitNonPersistentCookies;
    }

    public void deleteCookie(Cookie cookie) {
        String name = cookie.getName() + cookie.getDomain();
        this.cookies.remove(name);
        Editor prefsWriter = this.cookiePrefs.edit();
        prefsWriter.remove("cookie_" + name);
        prefsWriter.commit();
    }

    protected String encodeCookie(SerializableCookie cookie) {
        if(cookie == null) {
            return null;
        } else {
            ByteArrayOutputStream os = new ByteArrayOutputStream();

            try {
                ObjectOutputStream e = new ObjectOutputStream(os);
                e.writeObject(cookie);
            } catch (IOException var4) {
                Log.d("PersistentCookieStore", "IOException in encodeCookie", var4);
                return null;
            }

            return this.byteArrayToHexString(os.toByteArray());
        }
    }

    protected Cookie decodeCookie(String cookieString) {
        byte[] bytes = this.hexStringToByteArray(cookieString);
        ByteArrayInputStream byteArrayInputStream = new ByteArrayInputStream(bytes);
        Cookie cookie = null;

        try {
            ObjectInputStream e = new ObjectInputStream(byteArrayInputStream);
            cookie = ((SerializableCookie)e.readObject()).getCookie();
        } catch (IOException var6) {
            Log.d("PersistentCookieStore", "IOException in decodeCookie", var6);
        } catch (ClassNotFoundException var7) {
            Log.d("PersistentCookieStore", "ClassNotFoundException in decodeCookie", var7);
        }

        return cookie;
    }

    protected String byteArrayToHexString(byte[] bytes) {
        StringBuilder sb = new StringBuilder(bytes.length * 2);
        byte[] var3 = bytes;
        int var4 = bytes.length;

        for(int var5 = 0; var5 < var4; ++var5) {
            byte element = var3[var5];
            int v = element & 255;
            if(v < 16) {
                sb.append('0');
            }

            sb.append(Integer.toHexString(v));
        }

        return sb.toString().toUpperCase(Locale.US);
    }

    protected byte[] hexStringToByteArray(String hexString) {
        int len = hexString.length();
        byte[] data = new byte[len / 2];

        for(int i = 0; i < len; i += 2) {
            data[i / 2] = (byte)((Character.digit(hexString.charAt(i), 16) << 4) + Character.digit(hexString.charAt(i + 1), 16));
        }

        return data;
    }
}
```


