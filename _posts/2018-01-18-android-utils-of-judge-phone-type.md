---
layout: post
title:  判断手机类型的工具类——适配8.0手机
date:   2018-01-18 13:48:10 +0800
categories: Android
tag: [判断手机类型]
---

* content
{:toc}



### 判断手机类型的工具类——适配8.0手机
#### 1、需求：在做集成推送方案的时候，需要根据不同的手机类型来启用不同的推送方案。
①手机类型：小米、华为、其他手机

②三种推送方案的注册时机：

友盟推送是在Applicaiton#onCreate中，不区分进程。

小米推送是在Applicaiton#onCreate中，只在主进程。

华为推送是在启动页，StartActivity#onCreate中。

代码如下：通过读取build.prop文件，获取相关信息

```
/**
 * Created by tiny on 17/4/27.
 */

public class BuildProperties {
    private final Properties properties;

    private BuildProperties() throws IOException {
        properties = new Properties();
        properties.load(new FileInputStream(new File(Environment.getRootDirectory(), "build.prop")));
    }

    public boolean containsKey(final Object key) {
        return properties.containsKey(key);
    }

    public boolean containsValue(final Object value) {
        return properties.containsValue(value);
    }

    public Set<Map.Entry<Object, Object>> entrySet() {
        return properties.entrySet();
    }

    public String getProperty(final String name) {
        return properties.getProperty(name);
    }

    public String getProperty(final String name, final String defaultValue) {
        return properties.getProperty(name, defaultValue);
    }

    public boolean isEmpty() {
        return properties.isEmpty();
    }

    public Enumeration<Object> keys() {
        return properties.keys();
    }

    public Set<Object> keySet() {
        return properties.keySet();
    }

    public int size() {
        return properties.size();
    }

    public Collection<Object> values() {
        return properties.values();
    }

    public static BuildProperties newInstance() throws IOException {
        return new BuildProperties();
    }
}
```

```
/**
 * Desc:    获取操作系统，华为，小米，其他。
 * Created by tiny on 2018/1/10.
 * Time: 21:02
 * Version:
 */

public class OSUtils {
    private static final String PREFIX_HUAWEI = "HUAWEI";
    private static final String PREFIX_XIAOMI = "XIAOMI";
    private static final String PREFIX_OTHERS = "OTHERS";

    //MIUI标识
    private static final String KEY_MIUI_VERSION_CODE = "ro.miui.ui.version.code";
    private static final String KEY_MIUI_VERSION_NAME = "ro.miui.ui.version.name";
    private static final String KEY_MIUI_INTERNAL_STORAGE = "ro.miui.internal.storage";

    //EMUI标识
    private static final String KEY_EMUI_VERSION_CODE = "ro.build.version.emui";
    private static final String KEY_EMUI_API_LEVEL = "ro.build.hw_emui_api_level";
    private static final String KEY_EMUI_CONFIG_HW_SYS_VERSION = "ro.confg.hw_systemversion";

    public enum ROM_TYPE {
        MIUI(PREFIX_XIAOMI),
        EMUI(PREFIX_HUAWEI),
        OTHER(PREFIX_OTHERS);

        private String prefix;

        ROM_TYPE(String prefix) {
            this.prefix = prefix;
        }

        public String getPrefix() {
            return prefix;
        }
    }

    /**
     * @param
     * @return ROM_TYPE ROM类型的枚举
     * @description获取ROM类型: MIUI_ROM, EMUI_ROM, OTHER_ROM
     */

    public static ROM_TYPE getRomType() {
        ROM_TYPE rom_type = ROM_TYPE.OTHER;
        try {
            BuildProperties buildProperties = BuildProperties.newInstance();

            if (buildProperties.containsKey(KEY_EMUI_VERSION_CODE) || buildProperties.containsKey(KEY_EMUI_API_LEVEL) || buildProperties.containsKey(KEY_EMUI_CONFIG_HW_SYS_VERSION)) {
                return ROM_TYPE.EMUI;
            }
            if (buildProperties.containsKey(KEY_MIUI_VERSION_CODE) || buildProperties.containsKey(KEY_MIUI_VERSION_NAME) || buildProperties.containsKey(KEY_MIUI_INTERNAL_STORAGE)) {
                return ROM_TYPE.MIUI;
            }
        } catch (IOException e) {
            e.printStackTrace();
        }

        return rom_type;
    }

}
```

```
if (OSUtils.ROM_TYPE.EMUI.name().equals(OSUtils.getRomType().name())) {
    //是华为手机，初始化华为推送。
}
```
-------------------
#### 2、存在问题：
①获取手机类型的工具类需要读取文件的权限，而在Application第一次启动时，6.0以上的手机还未获取权限，所以第一次启动时都会注册为其他类型的推送。

②在6.0中申请sd卡读写权限时只需申请该权限组的任意一个即可，这里我申请的事写权限，没有申请读权限。

这样的做法在8.0以上的手机上会失效，读权限没有专门申请，会抛出相应的异常，导致读取build.prop文件失败，从而获取手机确切类型也会失败。

参考：[https://juejin.im/post/5991476f5188254898192ab9](https://juejin.im/post/5991476f5188254898192ab9)

#### 3、优化方案：
不再依赖于build.prop文件了，依赖的是Build.BRAND。根据它来判断手机类型，这样的话权限问题也就不再影响手机类型判断了。

```
/**
 * Desc:    获取操作系统，华为，小米，其他。
 * Created by tiny on 2018/1/10.
 * Time: 21:02
 * Version:
 */

public class OSUtils {
    private static final String PREFIX_HUAWEI = "HUAWEI";
    private static final String PREFIX_XIAOMI = "XIAOMI";
    private static final String PREFIX_OTHERS = "OTHERS";

    //MIUI标识
    public static final String BRAND_MIUI = "xiaomi";

    //EMUI标识
    public static final String BRAND_EMUI1 = "huawei";
    public static final String BRAND_EMUI2 = "honor";

    public enum ROM_TYPE {
        MIUI(PREFIX_XIAOMI),
        EMUI(PREFIX_HUAWEI),
        OTHER(PREFIX_OTHERS);

        private String prefix;

        ROM_TYPE(String prefix) {
            this.prefix = prefix;
        }

        public String getPrefix() {
            return prefix;
        }
    }

    /**
     * @param
     * @return ROM_TYPE ROM类型的枚举
     * @description获取ROM类型: MIUI_ROM, EMUI_ROM, OTHER_ROM
     */

    public static ROM_TYPE getRomType() {
        ROM_TYPE rom_type = ROM_TYPE.OTHER;

        String brand = Build.BRAND;
        if (!TextUtils.isEmpty(brand)) {
            if (brand.toLowerCase().contains(BRAND_EMUI1) || brand.toLowerCase().contains(BRAND_EMUI2)) {
                return ROM_TYPE.EMUI;
            }
            if (brand.toLowerCase().contains(BRAND_MIUI)) {
                return ROM_TYPE.MIUI;
            }
        }
        return rom_type;
    }

}
```
---------

如有错误，还望指正。

参考：

[https://juejin.im/post/5991476f5188254898192ab9](https://juejin.im/post/5991476f5188254898192ab9)

[http://blog.csdn.net/lovelyelfpop/article/details/65440420](http://blog.csdn.net/lovelyelfpop/article/details/65440420)

