<template>
  <div class="rich-eitor">
    <editor v-model="innerValue" @ready="ready" :storageKey="storageKey" :config="config"></editor>
  </div>
</template>
<script>
// import '../../static/UEditor/ueditor.config'
// import '../../static/UEditor/ueditor.all'
// import '../../static/UEditor/lang/zh-cn/zh-cn'
import editor from './editor'
export default {
  data () {
    return {
      innerValue: {
        wordCount: 0,
        body: '',
        plainTxt: ''
      },
      storageKey: '',
      config: {
        toolbars: [
          []
        ],
        initialFrameWidth: 930,
        initialFrameHeight: 400
      },
      uploadImageList: [] // 图片上传成功之后返回的图片属性对象，用于插入到富文本编辑器里面。
    }
  },
  watch: {
    innerValue: function () {
      console.log(this.innerValue)
    }
  },
  components: {
    editor
  },
  methods: {
    ready (ue) {
      console.log('初始化完毕')
    }
  }
}
</script>
