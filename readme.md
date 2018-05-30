## Install
```shell
npm install vueshowpdf -D
```
## Quick Start
``` javascript
//template
调用的时候使用
<button @click="showPdffun">展示PDF</button>
 <vuepdflook @closepdf="closepdf" v-model="isshowpdf" :pdfurl="pdfurls" @pdferr="pdferr" :maxscale='4' :minscale='0.6' :scale='1.1' ></vuepdflook>
 

 /*
 如果想要兼容IE就进行如下操作：
 首先
 npm i -D babel-polyfill;
 然后；
在webpack.base.conf.js的入口处需改如下;
 entry: {
    app:["babel-polyfill", "./src/main.js"]
  },
 
 */
//javascript
import vuepdflook from 'vuepdflook'

export default {
    data:{
        return:{
             pdfurls:'//cdn.mozilla.net/pdfjs/tracemonkey.pdf',
             isshowpdf:false
        }
       
    },
    methods:{
        closepdf(){
            this.isshowpdf = false
        },
        pdferr(errurl) {
            console.log(errurl);
        },
        showPdffun(){
          this.isshowpdf = true;
        },
    },
    components:{
        vuepdflook
    }

 }
``` 
 ## 参数说明

- closepdf 是关闭pdf的时候的出发的函数
- v-model 是否显示pdf
- pdfurl pdf的文件地址
- pdferr 文件地址解析错误时触发的函数 返回错误的pdf地址
- maxscale 最大放大倍数 默认 2
- minscale 最小放大倍数 默认 0.8
- scale 默认放大倍数 默认1.2

