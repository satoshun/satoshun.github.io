+++
date = "Sat Feb 20 09:06:26 UTC 2021"
title = "KMM: CocoaPodsのPrivate Spec Repoを使う"
tags = ["kmm", "kotlin"]
blogimport = true
type = "post"
draft = false
+++

(この記事は、前提知識として、KMMにおけるCocoapodsの知識が必要です。)


最初に、今回の記事で試した、サンプルプロジェクトを載せておきます。

- https://github.com/satoshun-android-example/KMMExternalSourceCocoapods
- https://github.com/satoshun-android-example/KMMCocoapodsSpecs

---

KMMでは、CocoaPodsプラグインを使うと、podspecファイルを生成してくれます。このpodfileはローカルでは動かすことが出来るんですが、Private Spec Repoにアップロードするには、これを編集する必要があります。

以下、編集したところを載せます。

```ruby
Pod::Spec.new do |spec|
    ...
    spec.source                   = { :git => "https://github.com/satoshun-android-example/KMMExternalSourceCocoapods.git", :tag => "0.0.2" }
    ...
    spec.preserve_paths           = "**/*.*"
    ...
    spec.script_phases = [
        {
            :name => 'Build cocoapodsshared',
            :execution_position => :before_compile,
            :shell_path => '/bin/sh',
            :script => <<-SCRIPT
                set -ev
                REPO_ROOT="$PODS_TARGET_SRCROOT"
                "$REPO_ROOT/gradlew" -p "$REPO_ROOT" :cocoapodsshared:syncFramework \
                    -Pkotlin.native.cocoapods.target=$KOTLIN_TARGET \
                    -Pkotlin.native.cocoapods.configuration=$CONFIGURATION \
                    -Pkotlin.native.cocoapods.cflags="$OTHER_CFLAGS" \
                    -Pkotlin.native.cocoapods.paths.headers="$HEADER_SEARCH_PATHS" \
                    -Pkotlin.native.cocoapods.paths.frameworks="$FRAMEWORK_SEARCH_PATHS"
            SCRIPT
        }
    ]
```

1. spec.source

spec.sourceの、GitHubとtagを指定します

2. spec.preserve_paths

これを指定すると、sourceのコードを保持してくれます。

3. spec.script_phases

gradlewのパスを調整する必要があります。

---

私の環境だと、この3つを追記、編集すればPrivate Spec Repoを使うことが出来ました。
