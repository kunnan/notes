**Android 7.1 PackageManagerService 构造函数**

```java
public PackageManagerService(Context context, Installer installer,
       boolean factoryTest, boolean onlyCore) {
       
       ···
       
        mContext = context;
        mFactoryTest = factoryTest; //假定为false，运行在非工厂模式
        mOnlyCore = onlyCore; // 假定为 false，扫描所有 APK
        mMetrics = new DisplayMetrics(); //分辨率相关类
       
        mSettings = new Settings(mPackages);
        mSettings.addSharedUserLPw("android.uid.system", Process.SYSTEM_UID,
                ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
        mSettings.addSharedUserLPw("android.uid.phone", RADIO_UID,
                ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
        mSettings.addSharedUserLPw("android.uid.log", LOG_UID,
                ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
        mSettings.addSharedUserLPw("android.uid.nfc", NFC_UID,
                ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
        mSettings.addSharedUserLPw("android.uid.bluetooth", BLUETOOTH_UID,
                ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
        mSettings.addSharedUserLPw("android.uid.shell", SHELL_UID,
                ApplicationInfo.FLAG_SYSTEM, ApplicationInfo.PRIVATE_FLAG_PRIVILEGED);
       ···
}
```

# Settings 类

## 构造函数

```java
Settings(Object lock) {
   this(Environment.getDataDirectory(), lock);
}

Settings(File dataDir, Object lock) {
   mLock = lock;

   mRuntimePermissionsPersistence = new RuntimePermissionPersistence(mLock);

    //创建目录 data/system
   mSystemDir = new File(dataDir, "system");
   mSystemDir.mkdirs();
   FileUtils.setPermissions(mSystemDir.toString(),
           FileUtils.S_IRWXU|FileUtils.S_IRWXG
           |FileUtils.S_IROTH|FileUtils.S_IXOTH,
           -1, -1);
    // packages.xml 用于描述系统所安装的 Package 信息，packages-backup.xml 是备份文件
   mSettingsFilename = new File(mSystemDir, "packages.xml");
   mBackupSettingsFilename = new File(mSystemDir, "packages-backup.xml");
   
   //packages.list 保存系统中存在的所有非系统自带的APK信息，即 UID 大于1000的apk
   mPackageListFilename = new File(mSystemDir, "packages.list");
   FileUtils.setPermissions(mPackageListFilename, 0640, SYSTEM_UID, PACKAGE_INFO_GID);

   final File kernelDir = new File("/config/sdcardfs");
   mKernelMappingFilename = kernelDir.exists() ? kernelDir : null;

   // Deprecated: Needed for migration
   // packages-stopped.xml 用于描述系统中强行停止运行的package信息
   mStoppedPackagesFilename = new File(mSystemDir, "packages-stopped.xml");
   mBackupStoppedPackagesFilename = new File(mSystemDir, "packages-stopped-backup.xml");
}
```

## addSharedUserLPw 方法

```java
    SharedUserSetting addSharedUserLPw(String name, int uid, int pkgFlags, int pkgPrivateFlags) {
        SharedUserSetting s = mSharedUsers.get(name);
        if (s != null) {
            if (s.userId == uid) {
                return s;
            }
            PackageManagerService.reportSettingsProblem(Log.ERROR,
                    "Adding duplicate shared user, keeping first: " + name);
            return null;
        }
        //构造 SharedUserSetting
        s = new SharedUserSetting(name, pkgFlags, pkgPrivateFlags);
        s.userId = uid;
        if (addUserIdLPw(uid, s, name)) {
            // 存入 map 中
            mSharedUsers.put(name, s);
            return s;
        }
        return null;
    }
```

```java
    private boolean addUserIdLPw(int uid, Object obj, Object name) {
        if (uid > Process.LAST_APPLICATION_UID) {
            return false;
        }
        //普通APK的uid  Process.FIRST_APPLICATION_UID = 1000
        if (uid >= Process.FIRST_APPLICATION_UID) {
            int N = mUserIds.size();
            final int index = uid - Process.FIRST_APPLICATION_UID;
            while (index >= N) {
                mUserIds.add(null);
                N++;
            }
            if (mUserIds.get(index) != null) {
                PackageManagerService.reportSettingsProblem(Log.ERROR,
                        "Adding duplicate user id: " + uid
                        + " name=" + name);
                return false;
            }
            mUserIds.set(index, obj);
        } else {
            if (mOtherUserIds.get(uid) != null) {
                PackageManagerService.reportSettingsProblem(Log.ERROR,
                        "Adding duplicate shared id: " + uid
                                + " name=" + name);
                return false;
            }
            mOtherUserIds.put(uid, obj);
        }
        return true;
    }
```

## SharedUserSettings 类

```java
/**
 * Settings data for a particular shared user ID we know about.
 */
final class SharedUserSetting extends SettingBase {
    final String name;

    int userId;

    // flags that are associated with this uid, regardless of any package flags
    int uidFlags;
    int uidPrivateFlags;

    final ArraySet<PackageSetting> packages = new ArraySet<PackageSetting>();

    final PackageSignatures signatures = new PackageSignatures();

    SharedUserSetting(String _name, int _pkgFlags, int _pkgPrivateFlags) {
        super(_pkgFlags, _pkgPrivateFlags);
        uidFlags =  _pkgFlags;
        uidPrivateFlags = _pkgPrivateFlags;
        name = _name;
    }

    @Override
    public String toString() {
        return "SharedUserSetting{" + Integer.toHexString(System.identityHashCode(this)) + " "
                + name + "/" + userId + "}";
    }

    void removePackage(PackageSetting packageSetting) {
        if (packages.remove(packageSetting)) {
            // recalculate the pkgFlags for this shared user if needed
            if ((this.pkgFlags & packageSetting.pkgFlags) != 0) {
                int aggregatedFlags = uidFlags;
                for (PackageSetting ps : packages) {
                    aggregatedFlags |= ps.pkgFlags;
                }
                setFlags(aggregatedFlags);
            }
            if ((this.pkgPrivateFlags & packageSetting.pkgPrivateFlags) != 0) {
                int aggregatedPrivateFlags = uidPrivateFlags;
                for (PackageSetting ps : packages) {
                    aggregatedPrivateFlags |= ps.pkgPrivateFlags;
                }
                setPrivateFlags(aggregatedPrivateFlags);
            }
        }
    }

    void addPackage(PackageSetting packageSetting) {
        if (packages.add(packageSetting)) {
            setFlags(this.pkgFlags | packageSetting.pkgFlags);
            setPrivateFlags(this.pkgPrivateFlags | packageSetting.pkgPrivateFlags);
        }
    }
}
```


```java
public PackageManagerService(Context context, Installer installer,
       boolean factoryTest, boolean onlyCore) {
  ···
  
  ···      
}
```


```java
public PackageManagerService(Context context, Installer installer,
       boolean factoryTest, boolean onlyCore) {
  ···
  
  ···      
}```


```java
public PackageManagerService(Context context, Installer installer,
       boolean factoryTest, boolean onlyCore) {
  ···
  
  ···      
}```


```java
.......
//构造函数传入的InstallerService，与底层Installd通信
mInstaller = installer;
mPackageDexOptimizer = new PackageDexOptimizer(installer, mInstallLock, context,
        "*dexopt*");

//定义一些回调函数
mMoveCallbacks = new MoveCallbacks(FgThread.get().getLooper());

mOnPermissionChangeListeners = new OnPermissionChangeListeners(
        FgThread.get().getLooper());

//存储显示信息
getDefaultDisplayMetrics(context, mMetrics);

//获取系统配置信息
SystemConfig systemConfig = SystemConfig.getInstance();
//将系统配置信息，存储到PKMS中
mGlobalGids = systemConfig.getGlobalGids();
mSystemPermissions = systemConfig.getSystemPermissions();
mAvailableFeatures = systemConfig.getAvailableFeatures();
..........
```

# 读取 XML 系统配置信息

frameworks/base/core/java/com/android/server/SystemConfig.java

```java
public static SystemConfig getInstance() {
   synchronized (SystemConfig.class) {
       if (sInstance == null) {
           sInstance = new SystemConfig();
       }
       return sInstance;
   }
}

```

```java
    SystemConfig() {
        // 从“system”目录读取
        readPermissions(Environment.buildPath(
                Environment.getRootDirectory(), "etc", "sysconfig"), ALLOW_ALL);
        // Read configuration from the old permissions dir
        readPermissions(Environment.buildPath(
                Environment.getRootDirectory(), "etc", "permissions"), ALLOW_ALL);
        // Allow ODM to customize system configs around libs, features and apps
        // 从"/odm"目录下读取
        int odmPermissionFlag = ALLOW_LIBS | ALLOW_FEATURES | ALLOW_APP_CONFIGS;
        readPermissions(Environment.buildPath(
                Environment.getOdmDirectory(), "etc", "sysconfig"), odmPermissionFlag);
        readPermissions(Environment.buildPath(
                Environment.getOdmDirectory(), "etc", "permissions"), odmPermissionFlag);
        // Only allow OEM to customize features
        // 从 "oem" 目录下读取
        readPermissions(Environment.buildPath(
                Environment.getOemDirectory(), "etc", "sysconfig"), ALLOW_FEATURES);
        readPermissions(Environment.buildPath(
                Environment.getOemDirectory(), "etc", "permissions"), ALLOW_FEATURES);
    }
```

```java
    void readPermissions(File libraryDir, int permissionFlag) {
        
        // 检查目录是否可在，是否可读

        ····

        // Iterate over the files in the directory and scan .xml files
        File platformFile = null;
        for (File f : libraryDir.listFiles()) {
            // We'll read platform.xml last
            if (f.getPath().endsWith("etc/permissions/platform.xml")) {
                platformFile = f;
                continue;
            }

            //读取可读的 xml 文件

            readPermissionsFromXml(f, permissionFlag);
        }

        // Read platform permissions last so it will take precedence
        if (platformFile != null) {
            readPermissionsFromXml(platformFile, permissionFlag);
        }
    }
```

```xml
/etc/permissions/android.hardware.nfc.hce.xml
···
/etc/permissions/android.hardware.nfc.xml
/etc/permissions/platform.xml
```


```xml
<?xml version="1.0" encoding="utf-8"?>

<permissions>

    <permission name="android.permission.BLUETOOTH_ADMIN" >
        <group gid="net_bt_admin" />
    </permission>

    <permission name="android.permission.BLUETOOTH" >
        <group gid="net_bt" />
    </permission>

   ···
   ···
   ···

    <!-- This is a list of all the libraries available for application
         code to link against. -->

    <library name="android.test.runner"
            file="/system/framework/android.test.runner.jar" />
    <library name="javax.obex"
            file="/system/framework/javax.obex.jar" />
    <library name="org.apache.http.legacy"
            file="/system/framework/org.apache.http.legacy.jar" />
    <allow-in-power-save package="com.android.providers.downloads" />
    <allow-in-data-usage-save package="com.android.providers.downloads" />

    <system-user-whitelisted-app package="com.android.settings" />

    <system-user-blacklisted-app package="com.android.wallpaper.livepicker" />
</permissions>

```


```java
    private void readPermissionsFromXml(File permFile, int permissionFlag) {
        FileReader permReader = null;
        try {
            permReader = new FileReader(permFile);
        } catch (FileNotFoundException e) {
            Slog.w(TAG, "Couldn't find or open permissions file " + permFile);
            return;
        }

        final boolean lowRam = ActivityManager.isLowRamDeviceStatic();

        try {
            XmlPullParser parser = Xml.newPullParser();
            //xml解析器输入fileReader读取的内容
            parser.setInput(permReader);

            int type;
            while ((type=parser.next()) != parser.START_TAG
                       && type != parser.END_DOCUMENT) {
                ;
            }

            if (type != parser.START_TAG) {
                throw new XmlPullParserException("No start tag found");
            }

            if (!parser.getName().equals("permissions") && !parser.getName().equals("config")) {
                throw new XmlPullParserException("Unexpected start tag in " + permFile
                        + ": found " + parser.getName() + ", expected 'permissions' or 'config'");
            }

            boolean allowAll = permissionFlag == ALLOW_ALL;
            boolean allowLibs = (permissionFlag & ALLOW_LIBS) != 0;
            boolean allowFeatures = (permissionFlag & ALLOW_FEATURES) != 0;
            boolean allowPermissions = (permissionFlag & ALLOW_PERMISSIONS) != 0;
            boolean allowAppConfigs = (permissionFlag & ALLOW_APP_CONFIGS) != 0;
            while (true) {
                XmlUtils.nextElement(parser);
                if (parser.getEventType() == XmlPullParser.END_DOCUMENT) {
                    break;
                }

                String name = parser.getName();
                //解析group标签
                if ("group".equals(name) && allowAll) {
                    String gidStr = parser.getAttributeValue(null, "gid");
                    if (gidStr != null) {
                        int gid = android.os.Process.getGidForName(gidStr);
                        mGlobalGids = appendInt(mGlobalGids, gid);
                    } else {
                        Slog.w(TAG, "<group> without gid in " + permFile + " at "
                                + parser.getPositionDescription());
                    }

                    XmlUtils.skipCurrentTag(parser);
                    continue;
                } else if ("permission".equals(name) && allowPermissions) {
                    String perm = parser.getAttributeValue(null, "name");
                    if (perm == null) {
                        Slog.w(TAG, "<permission> without name in " + permFile + " at "
                                + parser.getPositionDescription());
                        XmlUtils.skipCurrentTag(parser);
                        continue;
                    }
                    perm = perm.intern();
                    readPermission(parser, perm);

                } else if ("assign-permission".equals(name) && allowPermissions) {
                    String perm = parser.getAttributeValue(null, "name");
                    if (perm == null) {
                        Slog.w(TAG, "<assign-permission> without name in " + permFile + " at "
                                + parser.getPositionDescription());
                        XmlUtils.skipCurrentTag(parser);
                        continue;
                    }
                    String uidStr = parser.getAttributeValue(null, "uid");
                    if (uidStr == null) {
                        Slog.w(TAG, "<assign-permission> without uid in " + permFile + " at "
                                + parser.getPositionDescription());
                        XmlUtils.skipCurrentTag(parser);
                        continue;
                    }
                    int uid = Process.getUidForName(uidStr);
                    if (uid < 0) {
                        Slog.w(TAG, "<assign-permission> with unknown uid \""
                                + uidStr + "  in " + permFile + " at "
                                + parser.getPositionDescription());
                        XmlUtils.skipCurrentTag(parser);
                        continue;
                    }
                    perm = perm.intern();
                    ArraySet<String> perms = mSystemPermissions.get(uid);
                    if (perms == null) {
                        perms = new ArraySet<String>();
                        mSystemPermissions.put(uid, perms);
                    }
                    perms.add(perm);
                    XmlUtils.skipCurrentTag(parser);

                } else if ("library".equals(name) && allowLibs) {
                    String lname = parser.getAttributeValue(null, "name");
                    String lfile = parser.getAttributeValue(null, "file");
                    if (lname == null) {
                        Slog.w(TAG, "<library> without name in " + permFile + " at "
                                + parser.getPositionDescription());
                    } else if (lfile == null) {
                        Slog.w(TAG, "<library> without file in " + permFile + " at "
                                + parser.getPositionDescription());
                    } else {
                        //Log.i(TAG, "Got library " + lname + " in " + lfile);
                        mSharedLibraries.put(lname, lfile);
                    }
                    XmlUtils.skipCurrentTag(parser);
                    continue;

                } else if ("feature".equals(name) && allowFeatures) {
                    String fname = parser.getAttributeValue(null, "name");
                    int fversion = XmlUtils.readIntAttribute(parser, "version", 0);
                    boolean allowed;
                    if (!lowRam) {
                        allowed = true;
                    } else {
                        String notLowRam = parser.getAttributeValue(null, "notLowRam");
                        allowed = !"true".equals(notLowRam);
                    }
                    if (fname == null) {
                        Slog.w(TAG, "<feature> without name in " + permFile + " at "
                                + parser.getPositionDescription());
                    } else if (allowed) {
                        addFeature(fname, fversion);
                    }
                    XmlUtils.skipCurrentTag(parser);
                    continue;

                } else if ("unavailable-feature".equals(name) && allowFeatures) {
                    String fname = parser.getAttributeValue(null, "name");
                    if (fname == null) {
                        Slog.w(TAG, "<unavailable-feature> without name in " + permFile + " at "
                                + parser.getPositionDescription());
                    } else {
                        mUnavailableFeatures.add(fname);
                    }
                    XmlUtils.skipCurrentTag(parser);
                    continue;

                } else if ("allow-in-power-save-except-idle".equals(name) && allowAll) {
                    String pkgname = parser.getAttributeValue(null, "package");
                    if (pkgname == null) {
                        Slog.w(TAG, "<allow-in-power-save-except-idle> without package in "
                                + permFile + " at " + parser.getPositionDescription());
                    } else {
                        mAllowInPowerSaveExceptIdle.add(pkgname);
                    }
                    XmlUtils.skipCurrentTag(parser);
                    continue;

                } else if ("allow-in-power-save".equals(name) && allowAll) {
                    String pkgname = parser.getAttributeValue(null, "package");
                    if (pkgname == null) {
                        Slog.w(TAG, "<allow-in-power-save> without package in " + permFile + " at "
                                + parser.getPositionDescription());
                    } else {
                        mAllowInPowerSave.add(pkgname);
                    }
                    XmlUtils.skipCurrentTag(parser);
                    continue;

                } else if ("allow-in-data-usage-save".equals(name) && allowAll) {
                    String pkgname = parser.getAttributeValue(null, "package");
                    if (pkgname == null) {
                        Slog.w(TAG, "<allow-in-data-usage-save> without package in " + permFile
                                + " at " + parser.getPositionDescription());
                    } else {
                        mAllowInDataUsageSave.add(pkgname);
                    }
                    XmlUtils.skipCurrentTag(parser);
                    continue;

                } else if ("app-link".equals(name) && allowAppConfigs) {
                    String pkgname = parser.getAttributeValue(null, "package");
                    if (pkgname == null) {
                        Slog.w(TAG, "<app-link> without package in " + permFile + " at "
                                + parser.getPositionDescription());
                    } else {
                        mLinkedApps.add(pkgname);
                    }
                    XmlUtils.skipCurrentTag(parser);
                } else if ("system-user-whitelisted-app".equals(name) && allowAppConfigs) {
                    String pkgname = parser.getAttributeValue(null, "package");
                    if (pkgname == null) {
                        Slog.w(TAG, "<system-user-whitelisted-app> without package in " + permFile
                                + " at " + parser.getPositionDescription());
                    } else {
                        mSystemUserWhitelistedApps.add(pkgname);
                    }
                    XmlUtils.skipCurrentTag(parser);
                } else if ("system-user-blacklisted-app".equals(name) && allowAppConfigs) {
                    String pkgname = parser.getAttributeValue(null, "package");
                    if (pkgname == null) {
                        Slog.w(TAG, "<system-user-blacklisted-app without package in " + permFile
                                + " at " + parser.getPositionDescription());
                    } else {
                        mSystemUserBlacklistedApps.add(pkgname);
                    }
                    XmlUtils.skipCurrentTag(parser);
                } else if ("default-enabled-vr-app".equals(name) && allowAppConfigs) {
                    String pkgname = parser.getAttributeValue(null, "package");
                    String clsname = parser.getAttributeValue(null, "class");
                    if (pkgname == null) {
                        Slog.w(TAG, "<default-enabled-vr-app without package in " + permFile
                                + " at " + parser.getPositionDescription());
                    } else if (clsname == null) {
                        Slog.w(TAG, "<default-enabled-vr-app without class in " + permFile
                                + " at " + parser.getPositionDescription());
                    } else {
                        mDefaultVrComponents.add(new ComponentName(pkgname, clsname));
                    }
                    XmlUtils.skipCurrentTag(parser);
                } else if ("backup-transport-whitelisted-service".equals(name) && allowFeatures) {
                    String serviceName = parser.getAttributeValue(null, "service");
                    if (serviceName == null) {
                        Slog.w(TAG, "<backup-transport-whitelisted-service> without service in "
                                + permFile + " at " + parser.getPositionDescription());
                    } else {
                        ComponentName cn = ComponentName.unflattenFromString(serviceName);
                        if (cn == null) {
                            Slog.w(TAG,
                                    "<backup-transport-whitelisted-service> with invalid service name "
                                    + serviceName + " in "+ permFile
                                    + " at " + parser.getPositionDescription());
                        } else {
                            mBackupTransportWhitelist.add(cn);
                        }
                    }
                    XmlUtils.skipCurrentTag(parser);
                } else if ("disabled-until-used-preinstalled-carrier-associated-app".equals(name)
                        && allowAppConfigs) {
                    String pkgname = parser.getAttributeValue(null, "package");
                    String carrierPkgname = parser.getAttributeValue(null, "carrierAppPackage");
                    if (pkgname == null || carrierPkgname == null) {
                        Slog.w(TAG, "<disabled-until-used-preinstalled-carrier-associated-app"
                                + " without package or carrierAppPackage in " + permFile + " at "
                                + parser.getPositionDescription());
                    } else {
                        List<String> associatedPkgs =
                                mDisabledUntilUsedPreinstalledCarrierAssociatedApps.get(
                                        carrierPkgname);
                        if (associatedPkgs == null) {
                            associatedPkgs = new ArrayList<>();
                            mDisabledUntilUsedPreinstalledCarrierAssociatedApps.put(
                                    carrierPkgname, associatedPkgs);
                        }
                        associatedPkgs.add(pkgname);
                    }
                    XmlUtils.skipCurrentTag(parser);
                } else {
                    XmlUtils.skipCurrentTag(parser);
                    continue;
                }
            }
        } catch (XmlPullParserException e) {
            Slog.w(TAG, "Got exception parsing permissions.", e);
        } catch (IOException e) {
            Slog.w(TAG, "Got exception parsing permissions.", e);
        } finally {
            IoUtils.closeQuietly(permReader);
        }

        // Some devices can be field-converted to FBE, so offer to splice in
        // those features if not already defined by the static config
        if (StorageManager.isFileEncryptedNativeOnly()) {
            addFeature(PackageManager.FEATURE_FILE_BASED_ENCRYPTION, 0);
            addFeature(PackageManager.FEATURE_SECURELY_REMOVES_USERS, 0);
        }

        for (String featureName : mUnavailableFeatures) {
            removeFeature(featureName);
        }
    }
```


# 加载签名策略

```java
        synchronized (mPackages) {
            mHandlerThread = new ServiceThread(TAG,
                    Process.THREAD_PRIORITY_BACKGROUND, true /*allowIo*/);
            mHandlerThread.start();
            mHandler = new PackageHandler(mHandlerThread.getLooper());
            mProcessLoggingHandler = new ProcessLoggingHandler();
            Watchdog.getInstance().addThread(mHandler, WATCHDOG_TIMEOUT);

            mDefaultPermissionPolicy = new DefaultPermissionGrantPolicy(this);

            File dataDir = Environment.getDataDirectory();
            mAppInstallDir = new File(dataDir, "app");
            mAppLib32InstallDir = new File(dataDir, "app-lib");
            mEphemeralInstallDir = new File(dataDir, "app-ephemeral");
            mAsecInternalPath = new File(dataDir, "app-asec").getPath();
            mDrmAppPrivateInstallDir = new File(dataDir, "app-private");

            sUserManager = new UserManagerService(context, this, mPackages);

            // Propagate permission configuration in to package manager.
            ArrayMap<String, SystemConfig.PermissionEntry> permConfig
                    = systemConfig.getPermissions();
            for (int i=0; i<permConfig.size(); i++) {
                SystemConfig.PermissionEntry perm = permConfig.valueAt(i);
                BasePermission bp = mSettings.mPermissions.get(perm.name);
                if (bp == null) {
                    bp = new BasePermission(perm.name, "android", BasePermission.TYPE_BUILTIN);
                    mSettings.mPermissions.put(perm.name, bp);
                }
                if (perm.gids != null) {
                    bp.setGids(perm.gids, perm.perUser);
                }
            }

            ArrayMap<String, String> libConfig = systemConfig.getSharedLibraries();
            for (int i=0; i<libConfig.size(); i++) {
                mSharedLibraries.put(libConfig.keyAt(i),
                        new SharedLibraryEntry(libConfig.valueAt(i), null));
            }

            mFoundPolicyFile = SELinuxMMAC.readInstallPolicy();
```

# 扫描 Package

```java
// Set flag to monitor and not change apk file paths when
// scanning install directories.
final int scanFlags = SCAN_NO_PATHS | SCAN_DEFER_DEX | SCAN_BOOTING | SCAN_INITIAL;

//指向/system/framework目录
File frameworkDir = new File(Environment.getRootDirectory(), "framework");

// Collect vendor overlay packages. (Do this before scanning any apps.)
// For security and version matching reason, only consider
// overlay packages if they reside in the right directory.
// vendor/overlay
String overlayThemeDir = SystemProperties.get(VENDOR_OVERLAY_THEME_PROPERTY);
if (!overlayThemeDir.isEmpty()) {
 scanDirTracedLI(new File(VENDOR_OVERLAY_DIR, overlayThemeDir), mDefParseFlags
         | PackageParser.PARSE_IS_SYSTEM
         | PackageParser.PARSE_IS_SYSTEM_DIR
         | PackageParser.PARSE_TRUSTED_OVERLAY, scanFlags | SCAN_TRUSTED_OVERLAY, 0);
}
scanDirTracedLI(new File(VENDOR_OVERLAY_DIR), mDefParseFlags
     | PackageParser.PARSE_IS_SYSTEM
     | PackageParser.PARSE_IS_SYSTEM_DIR
     | PackageParser.PARSE_TRUSTED_OVERLAY, scanFlags | SCAN_TRUSTED_OVERLAY, 0);

// Find base frameworks (resource packages without code).
// /system/framework
scanDirTracedLI(frameworkDir, mDefParseFlags
     | PackageParser.PARSE_IS_SYSTEM
     | PackageParser.PARSE_IS_SYSTEM_DIR
     | PackageParser.PARSE_IS_PRIVILEGED,
     scanFlags | SCAN_NO_DEX, 0);

// Collected privileged system packages. system/priv-app 目录
final File privilegedAppDir = new File(Environment.getRootDirectory(), "priv-app");
scanDirTracedLI(privilegedAppDir, mDefParseFlags
     | PackageParser.PARSE_IS_SYSTEM
     | PackageParser.PARSE_IS_SYSTEM_DIR
     | PackageParser.PARSE_IS_PRIVILEGED, scanFlags, 0);

// Collect ordinary system packages. system/app 目录
final File systemAppDir = new File(Environment.getRootDirectory(), "app");
scanDirTracedLI(systemAppDir, mDefParseFlags
     | PackageParser.PARSE_IS_SYSTEM
     | PackageParser.PARSE_IS_SYSTEM_DIR, scanFlags, 0);

// Collect all vendor packages. /vendor/app 目录
File vendorAppDir = new File("/vendor/app");
try {
 vendorAppDir = vendorAppDir.getCanonicalFile();
} catch (IOException e) {
 // failed to look up canonical path, continue with original one
}
scanDirTracedLI(vendorAppDir, mDefParseFlags
     | PackageParser.PARSE_IS_SYSTEM
     | PackageParser.PARSE_IS_SYSTEM_DIR, scanFlags, 0);

// Collect all OEM packages.
final File oemAppDir = new File(Environment.getOemDirectory(), "app");
scanDirTracedLI(oemAppDir, mDefParseFlags
     | PackageParser.PARSE_IS_SYSTEM
     | PackageParser.PARSE_IS_SYSTEM_DIR, scanFlags, 0);
```

## scanDirLI()

```java
    private void scanDirTracedLI(File dir, final int parseFlags, int scanFlags, long currentTime) {
        Trace.traceBegin(TRACE_TAG_PACKAGE_MANAGER, "scanDir");
        try {
            //实际的扫描工作
            scanDirLI(dir, parseFlags, scanFlags, currentTime);
        } finally {
            Trace.traceEnd(TRACE_TAG_PACKAGE_MANAGER);
        }
    }
```

```java
private void scanDirLI(File dir, final int parseFlags, int scanFlags, long currentTime) {
    final File[] files = dir.listFiles();
    ···
    for (File file : files) {
        final boolean isPackage = (isApkFile(file) || file.isDirectory())
                && !PackageInstallerService.isStageName(file.getName());
        if (!isPackage) {
            // Ignore entries which are not packages
            continue;
        }
        try {
                //处理目录下的每一个 Package 文件
            scanPackageTracedLI(file, parseFlags | PackageParser.PARSE_MUST_BE_APK,
                    scanFlags, currentTime, null);
        } catch (PackageManagerException e) {
            Slog.w(TAG, "Failed to parse " + file + ": " + e.getMessage());

            // Delete invalid userdata apps
            if ((parseFlags & PackageParser.PARSE_IS_SYSTEM) == 0 &&
                    e.error == PackageManager.INSTALL_FAILED_INVALID_APK) {
                logCriticalInfo(Log.WARN, "Deleting invalid package at " + file);
                removeCodePathLI(file);
            }
        }
    }
}

private PackageParser.Package scanPackageTracedLI(File scanFile, final int parseFlags,
       int scanFlags, long currentTime, UserHandle user) throws PackageManagerException {
   Trace.traceBegin(TRACE_TAG_PACKAGE_MANAGER, "scanPackage");
   try {
       return scanPackageLI(scanFile, parseFlags, scanFlags, currentTime, user);
   } finally {
       Trace.traceEnd(TRACE_TAG_PACKAGE_MANAGER);
   }
}
```

```java
private PackageParser.Package scanPackageLI(File scanFile, int parseFlags, int scanFlags,
        long currentTime, UserHandle user) throws PackageManagerException {
    if (DEBUG_INSTALL) Slog.d(TAG, "Parsing: " + scanFile);
    //创建PackageParser对象
    PackageParser pp = new PackageParser();
    pp.setSeparateProcesses(mSeparateProcesses);
    pp.setOnlyCoreApps(mOnlyCore);
    pp.setDisplayMetrics(mMetrics);

    if ((scanFlags & SCAN_TRUSTED_OVERLAY) != 0) {
        parseFlags |= PackageParser.PARSE_TRUSTED_OVERLAY;
    }

    Trace.traceBegin(TRACE_TAG_PACKAGE_MANAGER, "parsePackage");
    final PackageParser.Package pkg;
    try {
        // 解析 APK 文件
        pkg = pp.parsePackage(scanFile, parseFlags);
    } catch (PackageParserException e) {
        throw PackageManagerException.from(e);
    } finally {
        Trace.traceEnd(TRACE_TAG_PACKAGE_MANAGER);
    }

    return scanPackageLI(pkg, scanFile, parseFlags, scanFlags, currentTime, user);
}
```

## PackageParser 的 parsePackage函数

```java

```

# 资料

* [Android N -- APK包的安装、卸载和优化（PackageManagerService http://blog.csdn.net/kitty_landon/article/details/46443849）（一）
](http://blog.csdn.net/kitty_landon/article/details/46443849)
* [android APK应用安装过程以及默认安装路径 http://blog.csdn.net/dddxxxx/article/details/75043306](http://blog.csdn.net/dddxxxx/article/details/75043306)

