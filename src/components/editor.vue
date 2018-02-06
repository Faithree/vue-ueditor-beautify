<template>
  <div class="rich-eitor">
    <div class="article-box">
      <div class="toolbar" id="toolbar">
        <div content="标题"
             :class="[toolStates.Paragraph == 1 ? 'tool-bar_selected' : '', toolStates.Paragraph == -1 ? 'tool-bar_disabled' : '' ]">
          <svg-icon v-if="toolStates.Paragraph === 1" icon-class="title" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="ueDisParagraph()"></svg-icon>
          <svg-icon v-else icon-class="title" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="ueParagraph()"></svg-icon>
        </div>
        <div content="加粗"
             :class="[toolStates.bold == 1 ? 'tool-bar_selected' : '', toolStates.bold == -1 ? 'tool-bar_disabled' : '' ]">
          <svg-icon icon-class="bold" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="execCommand('bold')"></svg-icon>
        </div>
        <div content="斜体"
             :class="[toolStates.italic == 1 ? 'tool-bar_selected' : '', toolStates.italic == -1 ? 'tool-bar_disabled' : '' ]">
          <svg-icon icon-class="italic" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="execCommand('italic')"></svg-icon>
        </div>
        <div content="下划线"
             :class="[toolStates.underline == 1 ? 'tool-bar_selected' : '', toolStates.underline == -1 ? 'tool-bar_disabled' : '' ]">
          <svg-icon icon-class="underline" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="execCommand('underline')"></svg-icon>
        </div>
        <div content="无序列表"
             :class="[toolStates.insertunorderedlist == 1 ? 'tool-bar_selected' : '', toolStates.insertunorderedlist == -1 ? 'tool-bar_disabled' : '' ]">
          <svg-icon icon-class="ul" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="insertunorderedlist()"></svg-icon>

        </div>
        <div content="有序列表"
             :class="[toolStates.insertorderedlist == 1 ? 'tool-bar_selected' : '', toolStates.insertorderedlist == -1 ? 'tool-bar_disabled' : '' ]">
          <svg-icon icon-class="ol" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="insertorderedlist()"></svg-icon>
        </div>
        <div content="插入图片">
          <svg-icon icon-class="picture" :autoStyle="{width:'20px',height:'29px'}" @click.native="ueInsertImage()">
          </svg-icon>
          <input type="file" multiple="multiple" ref="multipleImg" style="display: none" @change="imageFile">
        </div>
        <div content="上传视频">
          <svg-icon icon-class="video" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="ueUploadVideo()"></svg-icon>

        </div>
        <div content="上传文件">
          <svg-icon icon-class="direct" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="ueUploadFn()"></svg-icon>
        </div>
        <div content="撤销"
             :class="[toolStates.undo == 1 ? 'tool-bar_selected' : '', toolStates.undo == -1 ? 'tool-bar_disabled' : '' ]">
          <svg-icon icon-class="back" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="execCommand('undo')"></svg-icon>

        </div>
        <div content="重做" placement="top" effect="light"
             :class="[toolStates.redo == 1 ? 'tool-bar_selected' : '', toolStates.redo == -1 ? 'tool-bar_disabled' : '' ]">
          <svg-icon icon-class="go" :autoStyle="{width:'20px',height:'29px'}"
                    @click.native="execCommand('redo')"></svg-icon>

        </div>
        <div class="is-saved">
            <span v-if="isAutoSaved">
              已保存
            </span>
          <span v-else>
            保存中…
            </span>
        </div>
      </div>
      <div ref="editor">
      </div>
    </div>
  </div>
</template>
<script>
import '../../static/UEditor/ueditor.config'
import '../../static/UEditor/ueditor.all.min'
import '../../static/UEditor/lang/zh-cn/zh-cn'
export default {
  data () {
    return {
      id: Math.random(10) + 'ueditorId',
      isAutoSaved: true,
      editorData: {
        body: ''
        // images: []
      },
      toolStates: {
        bold: 0,
        italic: 0,
        underline: 0,
        Paragraph: 0,
        insertImage: 0,
        insertvideo: 0,
        insertunorderedlist: 0,
        insertorderedlist: 0,
        undo: 0,
        redo: 0
      }
    }
  },
  props: {
    value: {
      type: Object
    },
    storageKey: {
      type: String,
      default: 'storageKey'
    },
    config: {
      type: Object
    },
    uploadImageList: {
      type: Array,
      default: function () {
        return []
      }
    }
  },
  mounted () {
    // this.$nextTick(function f1 () {
    // 保证 this.$el 已经插入文档
    this.$refs.editor.id = this.id
    this.editor = UE.getEditor(this.id, this.config)
    this.editor.ready(() => {
      this.init()
      this.editor.addListener('contentChange', () => {
        const wordCount = this.editor.getContentLength(true)
        this.editorData.body = this.editor.getContent()
        const plainTxt = this.editor.getPlainTxt()
        this.$emit('input', {
          wordCount: wordCount,
          body: this.editorData.body,
          plainTxt: plainTxt
        })
        this.autoSaveBody()
      })
      this.editor.addListener('selectionchange', () => {
        const stateList = Object.keys(this.toolStates)
        var i = -1
        while (i++ < stateList.length - 1) {
          this.toolStates[stateList[i]] = this.editor.queryCommandState(stateList[i])
          if (stateList[i] === 'Paragraph') {
            console.log('Paragraph')
            console.log(this.editor.queryCommandValue(stateList[i]))
            if (this.editor.queryCommandValue(stateList[i]) === 'h1') {
              this.toolStates[stateList[i]] = 1
            } else {
              this.toolStates[stateList[i]] = 0
            }
          }
        }
      })
      this.$emit('ready', this.editor)
    })
  },
  watch: {
    uploadImageList: function () {
      this.usInsertMImage()
    }
  },
  methods: {
    ueDisParagraph: function () {
      this.editor.execCommand('Paragraph', 'p')
    },
    ueParagraph: function () {
      this.editor.execCommand('Paragraph', 'h1')
    },
    execCommand: function (name) {
      this.editor.execCommand(name)
    },
    insertorderedlist: function () {
      this.editor.execCommand('insertorderedlist', 'decimal')
    },
    insertunorderedlist: function () {
      this.editor.execCommand('insertunorderedlist')
    },
    ueInsertImage: function () {
      this.$refs['multipleImg'].click()
    },
    imageFile (event) {
      // 抛出事件,让外部处理上传图片逻辑
    },
    usInsertMImage () {
      let imageHtml = ''
      for (const i in this.uploadImageList) {
        imageHtml += '<p><img src="' + '/api/' + this.uploadImageList[i].url + '"><br></p>'
        // this.editorData.images.push(this.uploadImageList[i])
      }
      this.editor.execCommand('inserthtml', imageHtml)
    },
    ueUploadVideo () {
      // 抛出事件,让外部处理上传视频逻辑
    },
    // 上传压缩文件
    ueUploadFn () {
      // 抛出事件,让外部处理上传文件逻辑
    },
    // 初始化
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
  }
}
</script>
<style>
  .rich-eitor {
    max-width: 930px;
  }
  .article-box {
    width: 930px;
    border: 1px solid #E7E7E7;
    margin-bottom: 20px;
    position: relative;
  }
  .rich-eitor .edui-editor-toolbarbox {
    display: none;
  }
  .rich-eitor .edui-default .edui-editor {
    border: none;
  }
  .rich-eitor .edui-editor-bottomContainer {
    display: none;
  }

  .toolbar {
    border-bottom: 1px solid #e9e9e9;
    border-radius: 0;
    box-shadow: none;
    padding: 5px 10px 5px 20px;
    z-index: 1000;
    width: 900px;
    display: flex;
  }
  .toolbar > div {
    margin-right: 5px;
  }

  .is-saved {
    position: absolute;
    right: 0;
    margin-top: 7px;
    margin-right: 10px;
    font-family: PingFangSC-Regular;
    font-size: 14px;
    color: #999999;
    font-weight: bold;
    letter-spacing: 0;
  }

  .toolbar .tool-bar_selected svg {
    background-color: #fff;
  }

  .toolbar .tool-bar_disabled svg {
    opacity: 0.1;
  }

  .toolbar svg {
    cursor: pointer;
  }
</style>
