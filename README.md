# iOS
日常iOS开发遇到的项目问题，记录下来

### 一：CocoaPods环境配置
#### 1.1：Homebrew
```
// MARK: - 根据提示输入命令：

/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

#### 1.2：是否需要配置环境变量
```
// MARK: - 提示warning：

==> Homebrew is run entirely by unpaid volunteers. Please consider donating:https://github.com/Homebrew/brew#donations
==> Next steps:
- Run these two commands in your terminal to add Homebrew to your PATH:
    (echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> /Users/V606/.zprofile
    eval "$(/opt/homebrew/bin/brew shellenv)"
- Run brew help to get started

// MARK: - 根据提示输入命令,则需要配置环境变量，按照提示执行：Next steps:中的内容，直接copy到终端命令行就行,例如：
 ~ % (echo; echo 'eval "$(/opt/homebrew/bin/brew shellenv)"') >> /Users/V606/.zprofile
 ~ % eval "$(/opt/homebrew/bin/brew shellenv)"
```

#### 1.3：安装brew
```
brew install

brew update
```

#### 1.4：安装ruby
```
brew install ruby

// MARK: - 提示warning：
You may want to add this to your PATH.
If you need to have ruby first in your PATH, run:
    echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc
For compilers to find ruby you may need to set:
    export LDFLAGS="-L/opt/homebrew/opt/ruby/lib"
    export CPPFLAGS="-I/opt/homebrew/opt/ruby/include"

// MARK: - 根据提示输入命令即可
 ~ % echo 'export PATH="/opt/homebrew/opt/ruby/bin:$PATH"' >> ~/.zshrc
 ~ % export LDFLAGS="-L/opt/homebrew/opt/ruby/lib"
 ~ % export CPPFLAGS="-I/opt/homebrew/opt/ruby/include"

// MARK: - 查看源
 gem sources -l

// MARK: - 更改源
 ~ % gem sources --add https://gems.ruby-china.com/ --remove https://rubygems.org/
```

#### 1.5：安装cocopod
```
brew install cocoapods

brew upgrade cocoapods

pod setup
```

### 二：Xcode 编辑错误

#### 2.1：第三方库适配版本，避免一个一个手动修改，在podfile中添加
```
post_install do |installer|
  installer.generated_projects.each do |project|
    project.targets.each do |target|
      target.build_configurations.each do |config|
        config.build_settings['IPHONEOS_DEPLOYMENT_TARGET'] = '16.0'
      end
    end
  end
end
```

#### 2.2：编译报错 'Multiple commands produce .... .Swift文件/ .framwork文件'

##### 2.2.1：重复的文件 Multiple commands produce.......baseViewController.swift
```
1: 报错的 targets
2: Targets -> build Phases
3: Filter,搜索报错的文件，可能会看到好几个相同的文件
4: 点击 （- 号） 移除不属于项目本身的文件，这里的移除也只会移除索引，不会删除文件本身
```

##### 2.2.2：重复的framework文件 Multiple commands produce........framework

```
1: 按照（2。1.1）的方式寻找framework文件
2: 看是不是本地有a.framework 然后在第三方库里面也引入了a.framework
3: 看是不是有framework中又引入了嵌套framework，没有检索出来导致的问题
```

#### 2.3：编译报错 Lipo Erroe! Can’t open input file
```
1: Build setting
2: Valid Architecture add armv7
3: Valid Architecture remove armv7s and arm64
```

#### 2.4：编译报错 Sandbox: rsync.samba(59355) deny(1) file-write-unlink/Build/Products/Debug-iphoneos/603.app/Frameworks
```
1: Targets -> BuildSetting - > Build Options
2: ENABLE_USER_SCRIPT_SANDBOXING -> NO
```

#### 2.5：编译报错 "clang: error: linker command failed with exit code 1 (use -v to see invocation)"
```
1: Targets -> BuildSetting - > Architectures -> Build Active Architectures Only -> NO
2: Targets -> BuildSetting - > Architectures -> Excluded Architectures -> arm64
```

#### 2.6：编译报错 "library not found for -lAFNetworking"
```
Targets -> BuildSetting - > Linking-General -> Other Linker Flage
1: add -ObjC
2: remove -l"Framwork" 的样式
```

#### 2.7：编译报错 "Run custom shell script '[CP]' Embed Pods Frameworks"
```
Targets -> Build Phases - > Embed Pods Frameworks
remove "${SRCROOT}/Pods/Target Support Files/Pods-xxx/Pods-xxxx-frameworks.sh"
```

### 三：Xcode 打包错误

#### 3.1：打包报错 "dyld: Library not loaded: @rpath/FrameworkName.framework"
```
检查podfile格式

inhibit_all_warnings!
platform :ios, '14.0'

target ' appname' do
   use_modular_headers!
   
end

```

#### 3.2：打包报错 "Frameworks/xxxx.framework contains unsupported architectures '[i386, x86_64]"
```
添加Xcode编译脚本
Run Script 选项也要放在 CP Enbed Pods Framework 选项之前



# Type a script or drag a script file from your workspace to insert its path.
APP_PATH="${TARGET_BUILD_DIR}/${WRAPPER_NAME}"

# This script loops through the frameworks embedded in the application and
# removes unused architectures.
find "$APP_PATH" -name '*.framework' -type d | while read -r FRAMEWORK
do
FRAMEWORK_EXECUTABLE_NAME=$(defaults read "$FRAMEWORK/Info.plist" CFBundleExecutable)
FRAMEWORK_EXECUTABLE_PATH="$FRAMEWORK/$FRAMEWORK_EXECUTABLE_NAME"
echo "Executable is $FRAMEWORK_EXECUTABLE_PATH"

EXTRACTED_ARCHS=()

for ARCH in $ARCHS

do
echo "Extracting $ARCH from $FRAMEWORK_EXECUTABLE_NAME"
lipo -extract "$ARCH" "$FRAMEWORK_EXECUTABLE_PATH" -o "$FRAMEWORK_EXECUTABLE_PATH-$ARCH"
EXTRACTED_ARCHS+=("$FRAMEWORK_EXECUTABLE_PATH-$ARCH")
done

echo "Merging extracted architectures: ${ARCHS}"
lipo -o "$FRAMEWORK_EXECUTABLE_PATH-merged" -create "${EXTRACTED_ARCHS[@]}"
rm "${EXTRACTED_ARCHS[@]}"

echo "Replacing original executable with thinned version"
rm "$FRAMEWORK_EXECUTABLE_PATH"
mv "$FRAMEWORK_EXECUTABLE_PATH-merged" "$FRAMEWORK_EXECUTABLE_PATH"
done


```

#### 3.3：上传报错 "Failed to upload app to appstore: Invalid BundleExecutable. The executable file 'xxx.app/Frameworks/xxx.framework/ xxx' contains incomplete bitcode"
```
1: Window -> Organizer -> Archives 找到打包好的上架包
2: 显示包含内容，product -> .app -> 包含内容 -> 找到相关源码
3: 找到报错的framework -> 终端打开
4: 终端输入命令（提示 xxx 为报错的framework前缀）： otool -l xxx | grep __LLVM | wc -l 
5: 如果返回不为 0 ，则终端输入： xcrun bitcode_strip -r xxx -o xxx
```

### 四：Firebase & Facebook 集成问题

#### 4.1：Facebook 推广 -  投放 iOS14+ 选不了当前需要投放的APP
```
1: 集成FB SDK (版本最好用比最新版本低一两个的小版本 16.x.x)
2: iOS版本要支持 iOS 14+
3: FBAudienceNetwork 广告归因也集成进项目（https://developers.facebook.com/docs/audience-network/setting-up/platform-setup/ios/add-sdk?locale=zh_CN）
```

#### 4.2：文件无法找到 - FirebaseCrashlytics/FirebaseCrashlytics-Swift.h' file not found
```
在podfile 中增加脚本

post_install do |installer|
  installer.pod_targets.each do |pod|
    if pod.name.start_with?("Firebase")
      def pod.build_type;
        Pod::BuildType.dynamic_framework
        end
      pod.recursive_dependent_targets.each do |dep_pod|
        def dep_pod.build_type;
          Pod::BuildType.dynamic_framework
        end
      end
    end
  end
end
```















