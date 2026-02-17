GraphiteOS 开源项目结构

我们将 GraphiteOS 整理为标准的开源项目布局，方便社区贡献和协作。以下是完整的目录结构和关键文件内容。

```
graphite-os/
├── LICENSE                # Apache 2.0 许可证
├── README.md              # 项目概述、构建指南、贡献说明
├── CONTRIBUTING.md        # 贡献指南
├── .gitignore             # Git 忽略文件
├── manifest/              # Repo 清单文件
│   └── default.xml
├── kernel/                # GraphiteOS 内核模块
│   ├── graphite-modules/
│   │   ├── Kconfig
│   │   ├── Makefile
│   │   ├── tocttou_lsm.c
│   │   ├── google_shield.c
│   │   ├── ghost_env.c
│   │   ├── secure_exec.c
│   │   ├── hardened_ipc.c
│   │   ├── memory_guard.c
│   │   └── include/
│   └── Android.bp         # 可选，用于集成到 AOSP 内核构建
├── hardware/              # 硬件安全层 GHSL
│   └── ghsl/
│       ├── Android.bp
│       ├── keymaster/
│       │   ├── keymaster_device.cpp
│       │   ├── keymaster_impl.cpp
│       │   └── include/
│       ├── gatekeeper/
│       │   └── ...
│       ├── backends/
│       │   ├── qsee_backend.cpp
│       │   ├── sprd_tee_backend.cpp
│       │   ├── mobicore_backend.cpp
│       │   ├── optee_backend.cpp
│       │   └── fallback_backend.cpp
│       └── include/ghsl/
│           └── ghsl.h
├── frameworks/            # 框架层修改
│   ├── base/
│   │   ├── services/
│   │   │   ├── core/
│   │   │   │   └── java/com/android/server/
│   │   │   │       ├── pm/
│   │   │   │       │   └── PackageManagerService.patch
│   │   │   │       └── am/
│   │   │   │           └── ActivityManagerService.patch
│   │   │   └── grms/     # GraphiteOS 风险监控服务
│   │   │       ├── java/com/android/server/grms/
│   │   │       │   ├── GraphiteRiskMonitorService.java
│   │   │       │   └── IGraphiteRiskMonitor.aidl
│   │   │       └── Android.bp
│   │   └── core/
│   │       └── java/android/os/
│   │           └── IGraphiteRiskMonitor.aidl
│   └── native/
│       └── services/
│           └── sensorservice/
│               └── SensorService.patch
├── packages/              # 系统应用
│   ├── apps/
│   │   ├── GraphiteSecurity/
│   │   │   ├── Android.bp
│   │   │   ├── src/
│   │   │   │   └── org/graphite/security/
│   │   │   │       ├── MainActivity.kt
│   │   │   │       └── SecurityViewModel.kt
│   │   │   ├── res/
│   │   │   └── proguard.flags
│   │   ├── GithubStore/   # 从GitHub获取预编译APK
│   │   │   └── Android.mk  # 用于拷贝APK
│   │   ├── FDroid/
│   │   │   └── Android.mk
│   │   ├── XiaomiMarket/
│   │   │   └── Android.mk
│   │   ├── VivoMarket/
│   │   │   └── Android.mk
│   │   └── TencentMyApp/
│   │       └── Android.mk
│   └── inputmethods/
│       └── SimpleKeyboard/
│           └── Android.mk  # 如果从源码构建
├── device/                # 设备配置
│   └── graphite/
│       ├── common/
│       │   ├── common.mk
│       │   ├── system_prop.mk
│       │   ├── init.graphite.rc
│       │   ├── sepolicy/
│       │   │   ├── graphite.te
│       │   │   ├── graphite_risk.te
│       │   │   ├── graphite_store.te
│       │   │   ├── google_service.te
│       │   │   ├── file_contexts
│       │   │   └── property_contexts
│       │   ├── permissions/
│       │   │   └── graphite-permissions.xml
│       │   ├── config/
│       │   │   └── store_domains.xml
│       │   └── security/
│       │       ├── testkey.pk8
│       │       └── testkey.x509.pem
│       └── umi/            # 示例设备
│           ├── AndroidProducts.mk
│           ├── graphite_umi.mk
│           ├── BoardConfig.mk
│           └── sepolicy/
│               └── file_contexts
├── ota/                   # OTA 更新工具
│   ├── graphite_ota_generator.py
│   ├── update_engine.py
│   ├── update_server_config.json
│   ├── requirements.txt   # Python 依赖
│   └── README.md
├── docs/                  # 文档
│   ├── building.md
│   ├── porting.md
│   ├── security.md
│   └── architecture.md
└── vendor/                # 可选，用于存放预编译二进制
    └── graphite/
        ├── apps/           # 预编译 APK
        └── modules/        # 预编译内核模块
```

---

关键文件内容示例

根目录：LICENSE

```
Apache 2.0 License
```

根目录：README.md

```markdown
# GraphiteOS - 终极安全操作系统

GraphiteOS 是一个基于 AOSP 的高度安全的开源操作系统，专注于隐私保护、内存安全和硬件无关的通用安全性。它提供比 GrapheneOS 更强的安全机制，同时支持主流国产应用商店。

## 特性

- **内存安全**：基于 hardened_malloc 的 graphite_malloc，提供堆隔离、随机化和释放后写保护。
- **硬件安全抽象层 GHSL**：统一适配高通、联发科、展讯等平台的 TEE，软件回退方案达到商业安全级别。
- **风险监控服务 GRMS**：实时监控应用行为，动态调整风险等级，彻底拦截 Google 服务。
- **Google Shield**：内核级 Google 服务屏蔽模块，主动终止、降级并阻断网络。
- **商店管控**：小米、vivo、腾讯应用宝、GitHub Store、F-Droid 统一纳入 SELinux 隔离域。
- **原子更新**：基于 OSTree 的不可变系统，支持秒级回滚。
- **安全中心**：可视化仪表盘，展示安全指数、实时监控和事件日志。

## 快速开始

### 环境准备
- Ubuntu 22.04 LTS
- Python 3.8+
- repo 工具
- JDK 11

### 下载源码
```bash
mkdir ~/graphite-os
cd ~/graphite-os
repo init -u https://github.com/graphite-os/manifest.git -b main
repo sync -j8
```

构建

```bash
source build/envsetup.sh
lunch graphite_umi-userdebug
make -j$(nproc)
```

刷机

```bash
fastboot flash boot out/target/product/umi/boot.img
fastboot flash dtbo out/target/product/umi/dtbo.img
fastboot flash vendor out/target/product/umi/vendor.img
fastboot flash system out/target/product/umi/system.img
fastboot flash product out/target/product/umi/product.img
fastboot reboot
```

支持设备

· 小米 10 (umi)
· 小米 10 Pro (cmi)
· 小米 11 (venus)
· POCO X3 Pro (vayu)
· 更多设备正在适配中，欢迎贡献。

贡献

请阅读 CONTRIBUTING.md 了解如何参与开发。

许可证

Apache 2.0

```

#### 根目录：CONTRIBUTING.md
```markdown
# 贡献指南

感谢您对 GraphiteOS 的关注！我们欢迎任何形式的贡献，包括代码、文档、问题反馈等。

## 行为准则
请保持友善、尊重和包容。

## 如何贡献
1. Fork 本项目。
2. 创建您的特性分支 (`git checkout -b feature/amazing-feature`)。
3. 提交您的修改 (`git commit -m 'Add some amazing feature'`)。
4. 推送到分支 (`git push origin feature/amazing-feature`)。
5. 打开一个 Pull Request。

## 代码规范
- **C/C++**：遵循 AOSP 代码风格，使用 clang-format。
- **Java/Kotlin**：遵循 AOSP Java 风格。
- **Python**：遵循 PEP 8。
- **内核模块**：遵循 Linux 内核编码风格。

## 提交消息
请使用清晰的提交消息，第一行简短描述，然后空行，再写详细说明。

## 报告问题
请使用 GitHub Issues 报告 bug 或提出新特性建议。
```

根目录：.gitignore

```
# Android build outputs
out/
*.iml
*.ipr
*.iws
.idea/
.gradle/
local.properties

# Python
__pycache__/
*.pyc

# Kernel build
*.o
*.ko
*.mod.c
Module.symvers
modules.order
```

manifest/default.xml

```xml
<?xml version="1.0" encoding="UTF-8"?>
<manifest>
    <remote name="github" fetch="https://github.com/" />
    <remote name="aosp" fetch="https://android.googlesource.com/" />
    
    <default revision="refs/tags/android-15.0.0_r1" remote="aosp" sync-c="true" sync-j="4" />
    
    <!-- AOSP 基础项目 -->
    <project name="platform/build" path="build" groups="pdk" />
    <project name="platform/build/kati" path="build/kati" groups="pdk" />
    <project name="platform/build/soong" path="build/soong" groups="pdk" />
    <!-- 其他 AOSP 项目略 -->
    
    <!-- GraphiteOS 核心模块 -->
    <project path="external/graphite_malloc" name="graphite-os/graphite_malloc" remote="github" revision="main" />
    <project path="hardware/ghsl" name="graphite-os/hardware_ghsl" remote="github" revision="main" />
    <project path="frameworks/base/services/grms" name="graphite-os/frameworks_grms" remote="github" revision="main" />
    <project path="packages/apps/GraphiteSecurity" name="graphite-os/GraphiteSecurity" remote="github" revision="main" />
    <project path="kernel/graphite-modules" name="graphite-os/kernel_modules" remote="github" revision="main" />
    <project path="device/graphite" name="graphite-os/device_graphite" remote="github" revision="main" />
    <project path="ota" name="graphite-os/ota" remote="github" revision="main" />
</manifest>
```

kernel/graphite-modules/Makefile

```make
obj-$(CONFIG_GRAPHITE_TOCTTOU)      += graphite_tocttou.o
obj-$(CONFIG_GRAPHITE_GOOGLE_SHIELD) += graphite_google_shield.o
obj-$(CONFIG_GRAPHITE_GHOST_ENV)     += graphite_ghost_env.o
obj-$(CONFIG_GRAPHITE_SECURE_EXEC)   += graphite_secure_exec.o
obj-$(CONFIG_GRAPHITE_HARDENED_IPC)  += graphite_hardened_ipc.o
obj-$(CONFIG_GRAPHITE_MEMORY_GUARD)  += graphite_memory_guard.o

graphite_tocttou-y := tocttou_lsm.o
graphite_google_shield-y := google_shield.o
graphite_ghost_env-y := ghost_env.o
graphite_secure_exec-y := secure_exec.o
graphite_hardened_ipc-y := hardened_ipc.o
graphite_memory_guard-y := memory_guard.o

ccflags-y := -Wall -Werror -O2 -DGRAPHITEOS_KERNEL_MODULE

KERNEL_MAJOR := $(shell uname -r | cut -d. -f1)
KERNEL_MINOR := $(shell uname -r | cut -d. -f2)

ifeq ($(shell expr $(KERNEL_MAJOR) \>= 5), 1)
    ccflags-y += -DCONFIG_KERNEL_5X
endif

default:
    $(MAKE) -C $(KERNEL_SRC) M=$(PWD) modules

clean:
    $(MAKE) -C $(KERNEL_SRC) M=$(PWD) clean

install:
    $(MAKE) -C $(KERNEL_SRC) M=$(PWD) modules_install
    depmod -a
```

hardware/ghsl/Android.bp

```bp
cc_library_shared {
    name: "ghsl.keymaster",
    vendor: true,
    srcs: [
        "keymaster/keymaster_device.cpp",
        "keymaster/keymaster_impl.cpp",
    ],
    shared_libs: [
        "libhardware",
        "liblog",
        "libcutils",
        "libkeymaster_messages",
    ],
    static_libs: ["libghsl_backends"],
    cflags: ["-DGRAPHITEOS_HARDWARE_SECURITY"],
}

cc_library_static {
    name: "libghsl_backends",
    vendor: true,
    srcs: [
        "backends/qsee_backend.cpp",
        "backends/sprd_tee_backend.cpp",
        "backends/mobicore_backend.cpp",
        "backends/optee_backend.cpp",
        "backends/fallback_backend.cpp",
    ],
    local_include_dirs: ["include"],
    shared_libs: [
        "libQSEEComAPI",
        "libMcClient",
        "libteec",
    ],
}
```

frameworks/base/services/grms/Android.bp

```bp
android_app {
    name: "GraphiteRiskMonitor",
    srcs: ["java/**/*.java"],
    manifest: "AndroidManifest.xml",
    static_libs: [
        "androidx.annotation_annotation",
    ],
    privileged: true,
    certificate: "platform",
}
```

packages/apps/GraphiteSecurity/Android.bp

```bp
android_app {
    name: "GraphiteSecurity",
    srcs: ["src/**/*.kt"],
    resource_dirs: ["res"],
    manifest: "AndroidManifest.xml",
    static_libs: [
        "androidx.core_core",
        "androidx.appcompat_appcompat",
        "androidx.lifecycle_lifecycle-viewmodel-ktx",
        "androidx.compose.ui_ui",
        "androidx.compose.material3_material3",
        "androidx.navigation_navigation-compose",
    ],
    privileged: true,
    certificate: "platform",
    optimize: {
        proguard_flags_files: ["proguard.flags"],
    },
}
```

packages/apps/GithubStore/Android.mk

```make
LOCAL_PATH := $(call my-dir)
include $(CLEAR_VARS)
LOCAL_MODULE := GithubStore
LOCAL_SRC_FILES := $(LOCAL_PATH)/GithubStore.apk
LOCAL_MODULE_CLASS := APPS
LOCAL_MODULE_TAGS := optional
LOCAL_CERTIFICATE := PRESIGNED
LOCAL_MODULE_SUFFIX := $(COMMON_ANDROID_PACKAGE_SUFFIX)
include $(BUILD_PREBUILT)
```

device/graphite/common/common.mk

```make
# GraphiteOS 通用配置

PRODUCT_PACKAGES += \
    libgraphite_malloc \
    ghsl.keymaster \
    GraphiteRiskMonitor \
    GraphiteSecurity \
    GithubStore \
    FDroid \
    XiaomiMarket \
    VivoMarket \
    TencentMyApp

# 内核模块
BOARD_VENDOR_KERNEL_MODULES += \
    $(KERNEL_MODULES_OUT)/graphite_tocttou.ko \
    $(KERNEL_MODULES_OUT)/graphite_google_shield.ko \
    $(KERNEL_MODULES_OUT)/graphite_ghost_env.ko \
    $(KERNEL_MODULES_OUT)/graphite_secure_exec.ko \
    $(KERNEL_MODULES_OUT)/graphite_hardened_ipc.ko \
    $(KERNEL_MODULES_OUT)/graphite_memory_guard.ko

# 配置文件
PRODUCT_COPY_FILES += \
    $(LOCAL_PATH)/init.graphite.rc:$(TARGET_COPY_OUT_VENDOR)/etc/init/init.graphite.rc \
    $(LOCAL_PATH)/permissions/graphite-permissions.xml:$(TARGET_COPY_OUT_SYSTEM)/etc/permissions/graphite-permissions.xml \
    $(LOCAL_PATH)/config/store_domains.xml:$(TARGET_COPY_OUT_SYSTEM)/etc/graphite/store_domains.xml

# SELinux策略
BOARD_SEPOLICY_DIRS += device/graphite/common/sepolicy
board_Sepolicy_UNION+=\
graphite.te\
graphite_risk.te\
graphite_store.te\
Google_service.te

# 系统属性
product_PROPERTY_OVERRIDES+=\
ro.graphiteos.version=1.0.0\
ro.graphiteos.codename=Black_Diamond\
ro.com.google.gmsversion=禁用

#默认禁用谷歌服务框架
product_PACKAGES+=\
com.google.android.gms.disable

# 签名密钥
product_DEFAULT_DEV_CERTIFICATE：=设备/石墨/通用/安全/测试密钥
```

OTA/README.md

"'降价
#GraphiteOS OTA更新工具

该目录包含用于生成和应用OTA更新的工具.

##依赖
-Python3.8+
-'requests'、'pycryptodome'、'python-json-logger'

安装依赖：
"'bash
PIP安装-r要求.文本
```

生成OTA包

"'bash
Python graphite_OTA_generator py--目标目标文件。zip--输出石墨-ota。拉链
```

增量更新：

"'bash
python graphite_ota_generator。py-target_target_files。拉链--上一个上一个目标文件。zip--输出石墨增量。拉链
```

客户端引擎

update_Engine.py运行在设备上，负责检查更新、下载和应用。

服务端配置

update_server_config.json是OTA服务器的配置文件.

```

####docs/BUILDING.md
"'降价
#构建GraphiteOS

##环境要求
-Ubuntu22.04LTS(64位)
-至少16GB RAM，建议32GB
-至少200GB可用磁盘空间

##安装依赖
"'bash
sudo apt更新
sudo apt安装git-核心gnupg flex bison构建必需的zip curl zlib1g-dev gcc-multilib g++-multilib libc6-dev-i386libncurses5lib32ncur ses5-dev x11proto-core-dev libx11-dev lib32z1-dev libgl1-mesa-dev libxml2-utils xsltproc unzip fontconfig python3python3-pip
```

安装repo

"'bash
mkdir~/bin
path=~/bin：$PATH
卷曲https://storage.googleapis.com/git-repo-downloads/repo>~/bin/repo
chmod a+x~/bin/repo
```

获取源码

"'bash
mkdir石墨-os
CD石墨-os
repo init-u https://github.com/graphite-os/manifest.git-b main
回购同步-j8
```

构建

"'bash
源代码build/envsetup.sh
午餐石墨_umi-userdebug
make-j$(nproc)
```

构建完成后，镜像位于输出/目标/产品/umi/.

```

---

以上内容已整理为可直接在GitHub上创建的开源项目结构。用户可以根据需要调整细节，例如修改许可证、完善文档等。
