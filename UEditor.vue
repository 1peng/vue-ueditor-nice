<template>
  <div>
    <script :id="id" name="content" type="text/plain"></script>
  </div>
</template>
<script>
export default {
  name: 'VueUEditor',
  props: {
    ueditorPath: {
      // UEditor 代码的路径
      type: String,
      default: '/ueditor/'
    },
    ueditorConfig: {
      // UEditor 配置项
      type: Object,
      default: function () {
        return {}
      }
    },
    editorId: {
      // UEditor 编辑器ID
      type: String,
      default: 'ueditor'
    }
  },
  data () {
    return {
      id: '',
      instance: null,
      // scriptTagStatus -> 0:代码未加载，1:两个代码依赖加载了一个，2:两个代码依赖都已经加载完成
      scriptTagStatus: 0
    }
  },
  created () {
    if (typeof window !== 'undefined') {
      if (window.UE !== undefined) {
        // 如果全局对象存在，说明编辑器代码已经初始化完成，直接加载编辑器
        this.scriptTagStatus = 2
        this.initEditor()
      } else {
        // 如果全局对象不存在，说明编辑器代码还没有加载完成，需要加载编辑器代码
        this.insertScriptTag()
      }
    }
  },
  methods: {
    insertScriptTag () {
      let editorScriptTag = document.getElementById('editorScriptTag')
      let configScriptTag = document.getElementById('configScriptTag')

      // 如果这个tag不存在，则生成相关代码tag以加载代码
      if (editorScriptTag === null) {
        configScriptTag = document.createElement('script')
        configScriptTag.type = 'text/javascript'; configScriptTag.src = this.ueditorPath + 'ueditor.config.js'; configScriptTag.id = 'configScriptTag'

        editorScriptTag = document.createElement('script')
        editorScriptTag.type = 'text/javascript'; editorScriptTag.src = this.ueditorPath + 'ueditor.all.js'; editorScriptTag.id = 'editorScriptTag'
        let s = document.getElementsByTagName('head')[0]
        s.appendChild(configScriptTag)
        s.appendChild(editorScriptTag)
      }

      // 等待代码加载完成后初始化编辑器
      if (configScriptTag.loaded) {
        this.scriptTagStatus++
      } else {
        configScriptTag.addEventListener('load', () => {
          this.scriptTagStatus++
          configScriptTag.loaded = true
          this.initEditor()
        })
      }

      if (editorScriptTag.loaded) {
        this.scriptTagStatus++
      } else {
        editorScriptTag.addEventListener('load', () => {
          this.scriptTagStatus++
          editorScriptTag.loaded = true
          this.initEditor()
        })
      }

      this.initEditor()
    },
    initEditor () {
      this.id = 'editor_' + Math.random().toString(16).slice(-6);
      // scriptTagStatus 为 2 的时候，说明两个必需引入的 js 文件都已经被引入，且加载完成
      if (this.scriptTagStatus === 2 && this.instance === null) {
        // Vue 异步执行 DOM 更新，这样一来代码执行到这里的时候可能 template 里面的 script 标签还没真正创建
        // 所以，我们只能在 nextTick 里面初始化 UEditor
        this.$nextTick(() => {
          this.instance = window.UE.getEditor(this.id, this.ueditorConfig)
          // this.instance.removeListener('ready')
          // 绑定事件，当 UEditor 初始化完成后，将编辑器实例通过自定义的 ready 事件交出去
          this.instance.addListener('ready', () => {
            this.$emit('ready', this.instance)
          })
        })
      }
    }
  }
}
</script>
