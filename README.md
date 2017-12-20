
# Use Setup

### Install vue-ueditor-nice


#### 第一步
``` bash
npm install vue-ueditor-nice --save
```

#### 第二步
``` bash
打开 node_modules 下vue-ueditor-nice文件夹 把里边ueditor 拷到 你的项目目录 保证能调用 

比如这里我们把 ueditor 代码包放到了 static 目录，如果你是用 vue-cli 初始化项目结构的，那么我推荐你也和我一样把 ueditor 放在 static 目录。如果你想放在别的目录，也可以，但是，一定要记得给 vue-ueditor-nice 设置ueditorPath为正确的路径如下：

<VueUEditor ueditorPath="/ueditor/"></VueUEditor>

配置生成编辑器后的唯一ID,这里有利于编辑器的自动保存功能，如果你不要自动保存，请忽略此配置，它会自动默认一个叫ueditor
这里我在改造的时候，发现就算id固定了 但是自动保存后，刷新页面还是不会把保存的数据自动显示在页面上，后来我对ueditor.all.js进行了改在 
<VueUEditor  :editorId="editor" ></VueUEditor>

修改 ueditor 代码中的配置文件 ueditor.config.js，设置 ueditor 的根路径

var URL = '/ueditor/' || getUEBasePath();

上传图片，如果你希望你的编辑器内有上传图片功能，修改 ueditor 代码中的配置文件 ueditor.config.js 配置下你的上传图片接口就行了
这里要说明下，为什么不用那些PHP文件上传，第一PHP文件上传需要在你的服务器上支持PHP换机，而且是文件是传到当前项目内，不利于项目的维护，所以我把ueditor源码改了下支持配置上传图片接口

serverUrl: '',

具体的ueditor配置，可以通过ueditorConfig定义，具体配置参阅：百度ueditor官方配置文档http://ueditor.baidu.com/website/index.html

<VueUEditor :ueditorConfig="editorConfig" ></VueUEditor>

为了开发和使用方便，vue-ueditor 并没有修改或包装 UEditor 的任何功能和接口，而是在编辑器初始化成功的时候，将 UEditor 实例作为 ready 事件的参数传递给使用方，这样你就可以通过这个实例调用 UEditor 的各种功能了。

<VueUEditor @ready="editorReady"></VueUEditor>

import ueditor from 'vue-ueditor-nice'
export default {
  data () {
    return {
      content: ''
    }
  }.
  components: {
    ueditor
  },
  methods: {
      editorReady (editorInstance) {
        let self = this
        editorInstance.setContent('Hello UEditor')
        editorInstance.addListener('contentChange', () => {
          // 监测ueditor 发生变化是变化content值
          self.content = editorInstance.getContent()
        })
      }
  }
}
```


### 最后说一下

如果你的vue项目没有进行服务端渲染，遵循上边步骤完全没问题

如果你的项目使用了像nuxt的服务端渲染的话，建议把编辑器引入放入plugins中


``` bash
// plugins/ueditor.js 注意这里不能use只能component
import Vue from 'vue'
import VueUEditor from 'vue-ueditor-nice'
Vue.component(VueUEditor.name, VueUEditor) 
```

然后在nuxt.config.js plugins 并把ssr设置成false
``` bash
module.exports = {
  /*
  ** Headers of the page
  */
  head: {
    title: 'starter',
    meta: [
      { charset: 'utf-8' },
      { hid: 'description', name: 'description', content: 'Nuxt.js project' }
    ],
    link: [
      { rel: 'icon', type: 'image/x-icon', href: '/favicon.ico' }
    ]
  },
  build: {
    /*
     ** Run ESLINT on save
     */
    extend (config, ctx) {
      config.resolve.alias['vue'] = 'vue/dist/vue.common'
      if (ctx.isClient) {
        config.module.rules.push({
          enforce: 'pre',
          test: /\.(js|vue)$/,
          loader: 'eslint-loader',
          exclude: /(node_modules)/
        })
      }
    }
  },
  router: {
  },
  plugins: [{src: '~plugins/editor', ssr: false}]
}
```

本项目是根据vue-ueditor进行的修改优化，只是感觉其很多地方不够完善
