# iOS Fastlane+CI自动化构建

> 作者: Nine

## 摘要

在公司时，都是由一套完整的工具链完成，开发-&gt;构建-&gt;分发-&gt;测试，等流程 而作为个人开发时，失去了这套工具链，自然感觉到十分不便 所以笔者思考，能否自己搭建一套这样的工具链呢

## 需求列表

搭建一套工具，首先要明确我想实现的效果 我目前发布一个版本到TestFlight内测，需要以下几个步骤

* 手动改Xcode的build号格式为当天日期+次数，如09051601
* 手动archive，等待漫长过程，然后上传appstore
* 等待30分钟左右机器审核，然后手动选择组
* 重复输入更新内容，再点分发测试

如果需要发一个版本到AppStore 除了上述几个内容，我还需要

* 选择编译版本
* 添加版本更新内容
* 选择有无加密app
* 有无增加广告标识号

对于我而说，我希望上述两个过程，全部可以分别通过一行命令来进行 我希望我只需要**git push**一下，这些操作会自动在背后ci构建

## 最终实现效果

本地git push后

* 如果当前在master分支，ci自动打包发布TestFlight，版本号为当前日期+次数，并打tag提交。
* 如果当前在release/xxx分支，ci自动发布AppStore，版本号为当前日期+次数，并打tag提交。

  **流程示意图**

  ![](https://img.wxz.name/15583584051012.jpg)

  **Ci结果图**

  ![](https://img.wxz.name/15583584946186.jpg)

## 实现步骤

### 选用工具

* github作为存储代码的仓库
* travis-ci作为构建的ci
* fastlane作为构建工具

  > 注意，如果你的仓库是私有仓库的话，travis-ci将不再免费，不过如果通过了学生认证的话，依旧是免费的，笔者因为目前是学生，所以并没有单独购买。

### 期间遇到的坑

* travis-ci 的KeyChain导致不能编译
* apple的两步验证导致流程不能通过
* build号自增

### 1.配置Fastlane

首先进入工程目录，安装并对工程目录初始化

```text
    # 安装fastlane
    sudo gem install fastlane -NV
    # 初始化工程
    fastlane init
```

接着修改Fastfile文件 可以参照我的[Fastfile文件](https://gist.github.com/isnine/d4aacf8fcc7ecae50c3c8390651b893f)

### 1.1 更新版本号配置

在这一步中，我会先从iTunes Connect上获取最新的版本号，然后拉取下来，如果日期相同，则加一，否则重置为当天日期+01.比如19055101

```ruby
    # 更新版本号
def updateProjectBuildNumber
   currentTime = Time.new.strftime("%y%m%d")
   build = latest_testflight_build_number(version: get_version_number)
   puts("*************| Tag: #{build} |*************")
   buildStr = build.to_s
   if buildStr.include?"#{currentTime}"
      # => 为当天版本 计算迭代版本号
      lastStr = buildStr[buildStr.length-2..buildStr.length-1]
      lastNum = lastStr.to_i
      lastNum = lastNum + 1
      lastStr = lastNum.to_s
      if lastNum < 10
         lastStr = lastStr.insert(0,"0")
      end
      build = "#{currentTime}#{lastStr}"
   else
      # => 非当天版本 build 号重置
      build = "#{currentTime}01"
   end
   puts("*************| 更新build #{build} |*************")
   # => 更改项目 build 号
   increment_build_number(
   build_number: "#{build}"
   )
end
```

### 1.3 上传testFlight配置

在这一部中，我会首先编译当前的app，然后上传到testFlight,最后更新版本tag，push到远端，再上传dysm到fabric中。Changelog.txt里会存放我这个版本的变更内容。

```ruby
platform :ios do
   lane :beta do
      # 编译app
      buildApp
      # 上传
      upload_to_testflight(
      beta_app_feedback_email: "wxz@wxz.name",
      beta_app_description: "",
      notify_external_testers: false,
      distribute_external: true,
      changelog: File.read("Changelog.txt"),
      groups: ["testflight.top","Price Tag","其他渠道"],
      )
      # 更新版本tag
      add_git_tag
      if is_ci?
        system "git push https://isnine:${GH_Token}@github.com/user/repo.git --tags"
     else
        push_to_git_remote
     end
      # 上传dysm
      # crashlytics(api_token: "xxxxx",
      # build_secret: "xxxx")
   end
   # 编译当前app
def buildApp
   # 更改build号
   updateProjectBuildNumber
   # 编译
   build_app(workspace: "easy.xcworkspace", scheme: "easy")
end
```

### 1.4 上传appStore配置

这一步和上一步也类似，只是apple store的选项比较多

```ruby
platform :ios do
   lane :release do
      # 编译app
      buildApp
      # 上传app
      upload_to_app_store(
      skip_screenshots: true,
      skip_metadata: false,
      reject_if_possible: true,
      # skip_binary_upload: true,
      force: true,
      app_review_information: {
         first_name: 'W',
         last_name: 'XZ',
         phone_number: '+86 1760000000',
         email_address: 'user@example.com',
         demo_user: '',
         demo_password: '',
         notes: ''
      },
      submit_for_review: true,
      submission_information: {
         add_id_info_limits_tracking: true,
         add_id_info_serves_ads: false,
         add_id_info_tracks_action: true,
         add_id_info_tracks_install: true,
         add_id_info_uses_idfa: true,
         content_rights_has_rights: true,
         content_rights_contains_third_party_content: true,
         export_compliance_platform: 'ios',
         export_compliance_compliance_required: false,
         export_compliance_encryption_updated: false,
         export_compliance_app_type: nil,
         export_compliance_uses_encryption: false,
         export_compliance_is_exempt: false,
         export_compliance_contains_third_party_cryptography: false,
         export_compliance_contains_proprietary_cryptography: false,
         export_compliance_available_on_french_store: false
      },
      release_notes: {'default' => File.read("Changelog.txt"),
                      'zh-Hans' => File.read("Changelog.txt"),
                      'en-US' => File.read("Changelog.txt")}
      )
      # 更新版本tag
      add_git_tag
      if is_ci?
        system "git push https://isnine:${GH_Token}@github.com/user/repo.git --tags"
     else
        push_to_git_remote
     end
      # 上传dysm
      # crashlytics(api_token: "xxxx",
      # build_secret: "xxx")
   end
```

### 2 ci配置

完成了上面两步，其实已经可以通过fastlane beta和fastlanbe release在本地发布了，因为本地非常方面，其实是不需要考虑以下问题。

* 证书导入
* 私钥导入
* github push权限
* apple connect权限
* 两步认证

  而CI因为每次启动都是一个新的环境，这些全部要考虑一遍。

  首先需要再fastlane/certs/目录下存放好相关的p12和描述文件,如图所示:

  ![](https://img.wxz.name/15584097255132.jpg)

  接着我在fastlane目录下，写了个导入描述文件的脚本:

  load\_provision.sh

  ```bash
    mkdir -p ~/Library/MobileDevice/Provisioning\ Profiles
    cp ./certs/*.mobileprovision ~/Library/MobileDevice/Provisioning\ Profiles/
  ```

  最后Fastfile部分如下

  ```ruby
  # 记得提前在CI中设置 https://travis-ci.com/{user}/{app}/settings
   # - GH_Token //Github的token
   # - Cert_PassWord //证书密码
   # - FASTLANE_PASSWORD // Apple 账号密码
   # - FASTLANE_APPLE_APPLICATION_SPECIFIC_PASSWORD //不开两步认证忽略，PassWord apple专用密码
   # - Apple_Session //appleSession，不开两步认证忽略，通过fastlane spaceauth -u user@example.com获得
   lane :ci do
      branch = git_branch
      puts("*************| 当前branch #{branch} |*************")
      # 配置环境
      keychain_name = "easy-build"
      keychain_password = "travis"
      # 设置session 规避两步验证码，不开两步认证忽略
      # ENV["FASTLANE_SESSION"] = File.read("session.txt")
      # 创建临时钥匙串
      create_keychain(
      name: keychain_name,
      password: keychain_password,
      default_keychain: true,
      unlock: true,
      timeout: 3600,
      add_to_search_list: true
      )
      # 导入私钥
      import_certificate(
      certificate_path: "./fastlane/certs/dist.p12",
      certificate_password: ENV["Cert_PassWord"],
      keychain_name: keychain_name,
      keychain_password: keychain_password
      )
      import_certificate(
      certificate_path: "./fastlane/certs/dev.p12",
      certificate_password: ENV["Cert_PassWord"],
      keychain_name: keychain_name,
      keychain_password: keychain_password
      )
      # 拉取证书
      system "sh ./load_provision.sh"
      # sigh(app_identifier: "bundle.id",
      #   username: "user@exampe.com")
      if branch.start_with? "master"
         beta
      elsif branch.start_with? "release"
         release
      end
   end
  ```

  **2.1 Travis-ci配置**

  > 这部分我以新开一个开发者账号的方式讲，我强烈推荐这种方式。 你可以在后文中找到，如果不开新账号，用原来带有两步认证的开发者账号怎么实现。

这里我们一共需要配置四个环境变量

* GH\_Token //Github的token
* Cert\_PassWord //证书密码
* FASTLANE\_USER // Apple 账号
* FASTLANE\_PASSWORD // Apple 账号密码

  ![](https://img.wxz.name/15584100323855.jpg)

  **GH\_Token**

  这一步是为了解决push权限的问题

  ![](https://img.wxz.name/15584101532733.jpg)

  ![](https://img.wxz.name/15584101656036.jpg)

  然后将创建的token复制过来作为GH\_Token的值即可

  > 记得前面Fastfile里面的仓库地址也要改
  >
  > #### FASTLANE\_USER和FASTLANE\_PASSWORD
  >
  > [https://appstoreconnect.apple.com/](https://appstoreconnect.apple.com/) 这个很好理解，你的apple账号和密码。我踩了无数坑了，强烈建议重新创一个没有两步认证的apple小号。 然后在大号中邀请进来. 个人开发者账号虽然邀请进来的人不能拉取证书，但是一样有权限发布上传App ![](https://img.wxz.name/15584105581907.jpg) 接着在ci中把FASTLANE\_USER和FASTLANE\_PASSWORD这两项配置好

#### Cert\_PassWord

我们在keychain中，导出时设置的p12文件的密码，就是这一项 ![](https://img.wxz.name/15584106259902.jpg)

## 踩过的坑

### 1.为什么不直接用带有两步认证的账号

如果用带两步认证的账号，需要配置apple专用密码，并且还要加上session。 而session特别容易过期，网上说是30天，笔者测试下来一天就要重新设置一次，十分痛苦 如果一定要用的话，可以在刚刚的流程结束完，在ci中设置一个新的环境变量 FASTLANE\_APPLE\_APPLICATION\_SPECIFIC\_PASSWORD 可以在这里面设置[https://appleid.apple.com/account/manage](https://appleid.apple.com/account/manage) ![](https://img.wxz.name/15584109865524.jpg) 同时因为登录会话的问题，如果想在ci上直接绕过两步认证，还需要配置好session

```text
    fastlane spaceauth -u user@example.com #你的apple账号
```

然后复制session，我这里是在fastlane文件夹下创建了一个session.txt 将内容复制到里面去。 在脚本中再通过代码设置出来

```ruby
ENV["FASTLANE_SESSION"] = File.read("session.txt")
```

