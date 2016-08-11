删除 aar里面无用的资源
===

有时候我们引入一些第三方aar包的时候发现多了一些无用的资源，但是又会影响我们的时候，一种方法就是重新自己
编译一个，但是这种以后升级的时候又有同步代码的麻烦。

所以我们可以定义一些自定义的 *task* ，所有 *aar* 的资源都保存在 *build/exploded-aar *文件夹里面，
只要我们在 *resource merging* 之前移除这些文件即可

使用方法

```gradle

apply from: 'https://raw.githubusercontent.com/yaming116/Android_Notes/master/apk_copy/delete_res_in_aar.gradle'


dependencies {
    debugCompile ('com.squareup.leakcanary:leakcanary-android:1.3.1')
                .exclude("res/values-v21/values-v21.xml" ,"other res")
    releaseCompile ('com.squareup.leakcanary:leakcanary-android-no-op:1.3.1')
}

```

[来源](http://stackoverflow.com/questions/34397796/how-to-exclude-specific-resources-from-an-aar-depedency/34525498#34525498)