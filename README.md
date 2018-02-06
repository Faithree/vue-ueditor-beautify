### 需求
(PC端)做一个可以使用图片上传、视频上传、文件上传功能的富文本组件，简单的文本编辑发布功能,采用socket方式传输，

### 做法
当时看到这个需求，我觉得是不难的,就去github上找富文本编辑器，因为项目比较急，当时我的想法是能找开箱即用的就找开箱即用的。
![](https://user-gold-cdn.xitu.io/2018/2/5/1616576bbc964482?w=1368&h=808&f=png&s=176492)

> 这几个编辑器都试了一次,最后经过筛选
*  Vue-Quill-Editor：不支持本地视频上传
*  vue-froala-wysiwyg：我想要的功能都有包括图片和视频上传，还额外支持响应式，但是是收费的
*  ueditor:功能很全，但是样式实在是很丑,可能要自己封装
* 剩下的编辑器基本上要么是功能不足或者是移动端的(优点是轻量)

因为没做过富文本的开发，所以当时我测试这些富文本就花了一个下午，后来仔细考虑了一下，最后使用了比较保守的方式，用了ueditor开发，顺便美化了一下。
### 引入ueditor
[下载地址和文档](http://ueditor.baidu.com/website/)
* import '../../static/UEditor/ueditor.config'
* import '../../static/UEditor/ueditor.all.min'
* import '../../static/UEditor/lang/zh-cn/zh-cn'

我们要去ueditor.config.js文件里面去改一下路径配置
![](https://user-gold-cdn.xitu.io/2018/2/6/1616934f9b689418)
### 修改样式
引入ueditor之后，直接让工具栏隐藏起来，然后我们自己新建一个div模拟toolbars

![](https://user-gold-cdn.xitu.io/2018/2/6/16168e88857fa516?w=573&h=271&f=png&s=13519)
隐藏之后的ueditor就是一个类似div加了可编辑属性的既视感
![](https://user-gold-cdn.xitu.io/2018/2/6/16168e9a1edffea9?w=1174&h=543&f=png&s=15606)

接下来我们直接一个div，然后给他一个flex布局，然后去[iconfont](http://www.iconfont.cn/)上面下载一些图标，但是我们需要配置一下webpack使他支持批量处理svg,[参考手摸手系列](https://juejin.im/post/59bb864b5188257e7a427c09)

![](https://user-gold-cdn.xitu.io/2018/2/6/161693efdeb8898d?w=835&h=461&f=png&s=37784)
![](https://user-gold-cdn.xitu.io/2018/2/6/16168ecf9edc8985?w=877&h=311&f=png&s=48132)
现在就大变身了，样子非常好看，跟现代的编辑器没什么差别
![](https://user-gold-cdn.xitu.io/2018/2/6/16168ed6e7c30bcb?w=1190&h=601&f=png&s=20273)
### 给图标绑定点击事件
经过上面的步骤，样式基本上搞定了，剩下来就是使他们点击的时候，触发事件，让他们做出相应的动作就行，例如
```
    execCommand: function (name) {
      this.editor.execCommand(name)
    },
```
插入图片，插入视频，插入文件这种操作，我并没有使用ueditor内置的功能，视频和文件夹我做成了进度条的形式，放在了富文本编辑器的下边，图片上传成功后返回值拼接起来，根据双向绑定，在editor组件内部动态创建图片，点击这三个图标，会把事件抛出来，这样你想用什么上传组件就用哪个，你还可以用socket进行上传等等，这样子，editor组件内部只需要维护编辑器的html文本就可以，职责单一，后期也好维护
```
      editorData: {
        body: '',
        images: []
      },
```

### 本地保存功能
最后添加了一个自动保存的功能,这里可以用定时器或者当内容发生变化的时候存到localStorage里面。
```
    autoSaveBody () {
      if (this.isAutoSaved && this.editorData.body) {
        let storage = {}
        Object.assign(storage, this.editorData)
        const pms = JSON.stringify(storage)
        this.isAutoSaved = false
        setTimeout(() => {
          localStorage.setItem(this.storageKey, pms)
          this.isAutoSaved = true
        }, 500)
      }
    }
```



![](https://user-gold-cdn.xitu.io/2018/2/6/161699c063e31334?w=1245&h=606&f=png&s=20793)



但是必须要考虑一个情况，当前是第一次写还是发布之后进行修改，所以外部使用的时候，你只需要操作innerValue这个属性，这个属性的值你可以通过后台来获取(后台获取的就是修改状态)，然后编辑器就会呈现什么样的数据,内部的实现方式就是加了一个init函数
```
    init: function () {
      // 外部有默认值
      if (this.value.body) {
        this.editor.setContent(this.value.body)
      } else {
        // 有本地缓存
        const storage = localStorage.getItem(this.storageKey)
        if (storage) {
          const storageJson = JSON.parse(storage)
          Object.assign(this.editorData, storageJson)
          if (this.editorData.body) {
            this.editor.setContent(this.editorData.body)
          }
        } else {
          // 没有本地缓存
          Object.assign(this.editorData, this.$options.data().editorData)
        }
      }
      this.autoSaveInterval = setInterval(() => {
        this.autoSaveBody()
      }, 5000)
    },
```
![](https://user-gold-cdn.xitu.io/2018/2/6/16169341b9f27df1)

![](https://user-gold-cdn.xitu.io/2018/2/6/1616905fc037e2e9?w=485&h=310&f=png&s=13989)
### 缺点
组件太大，默认压缩也有389k,开启gzip之后有100k左右
![](https://user-gold-cdn.xitu.io/2018/2/6/16169374e15e4e1e?w=2026&h=1272&f=png&s=163507)
### 优点
功能强大，产品需求可迭代。
### 补充
当然这个是我项目中抽取出来的，不是完整的代码，只是一个思路吧。第一次做富文本，要是说错了大家多多指正，或者大家有更好的思路欢迎一起讨论

[代码地址](https://github.com/Faithree/vue-ueditor-beautify)