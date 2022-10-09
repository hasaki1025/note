# Vue

- 入门案例

  - 添加依赖

    ```html
    <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.js"></script>
    ```

  - 创建html标签

    ```html
     <div id="app">
            {{message}}
        </div>
    ```

  - 编写Vue程序

    ```html
     <script>
            var app=new Vue( 
                {
                    el:'#app',
                    data:{
                        message:'hello vue'
                    }
                }
            )
        </script> 
    ```

  - Vue作用范围，在id所指的标签内部所有匹配元素都可以

    ```html
    <div id="app">
            {{message}}
            <p>{{message}}</p>
        </div>
    ```

  - el挂载点

    - Vue可以使用class选择器，也可以使用标签选择器

  - data数据对象

    - data添加数组和对象

      ```javascript
      var app=new Vue( 
                  {
                      el:".gg",
                      data:{
                          message:'hello vue',
                          javaBook:['Spring Boot项目实战','Redis实战'],
                          Book:{
                              name:'hello world',
                              type:'小说',
                          }
                      }
                  }
              )
      ```

  - Vue指令

    - v-text

      ```html
      <div v-text="message"></div>
      ```

    - v-html

      ```html
      <div v-html="html"></div>
      ```

    - v-on:事件名称

      ```javascript
      el:"#kk",
          methods: {
              dolt:function() {
                  alert('傻逼');
              }
          },
      ```

      ```html
      <input type="button" value="1" @click="dolt">
      <input type="button" value="1" v-on:click="dolt">
      ```

  - 制作一个简单的计数器

    ```javascript
     var count=new Vue({
                el:'#count',
                data:{
                    nums:0
                },
                methods:{
                    add:function() {
                        
                        if(this.nums<10)
                        {
                            this.nums++;
                        }
                    },
                    sub:function() {
                        if(this.nums>0)
                        {
                            this.nums--;
                        }
                        
                    }
                }
                
            })
    ```

    ```html
    <div id="count">
            <button @click="sub">
                -
            </button>
            <span v-text="nums"></span>
            <button @click="add">
                +
            </button>
        </div>
    
    ```

  - v-show指令

    ```html
    <img src="e24c89ac1e688f148af4d23482ec5e17.jpg" v-show="javaBook.length<0">
    <img src="e24c89ac1e688f148af4d23482ec5e17.jpg" v-show="变量名">
    <img src="e24c89ac1e688f148af4d23482ec5e17.jpg" v-show="true">
    ```

    - 只要是bool表达式就可以了，v-if的使用和这个相同，但是v-if可以支持其他标签

  - v-bind指令绑定属性

    ```html
     <img :src="src" width="1000" height="500" v-show="isshow"><br>
    ```

    - src是vue中的值

  - 完成一个图片轮播图

    ```html
    <p id="imgshow">
            <img :src="img[index]" width="1000" height="500"><br>
            <button @click="next">下一张</button>
            <button @click="last">上一张</button>
        </p>
    ```

    ```javascript
    
            var p2=new Vue({
                el:imgshow,
                data:{
                        img:[
                            "img/3b16b5f610a735b18e3b5100fa502caf.jpg",
                            "img/6e0c18b29250a09bdd49786305a6a25b.jpg",
                            "img/36ca489592e4ed4b9e45327efb8c11c0.jpg",
                            "img/09108f0d7f193b31b010f4d0a01f4f34.jpg",
                            "img/e24c89ac1e688f148af4d23482ec5e17.jpg"],
                        index:0
                    },
                methods: {
                    next:function() {
                        var index=this.index;
                        var img=this.img;
                        this.index=(index+1)%img.length
                    },
                    last:function() {
                        var index=this.index;
                        var img=this.img;
                        this.index=(index-1)>0 ? index-1 : img.length-1;
                    }
                }
            })
    ```

  - v-for指令(遍历数组显示)

    ```html
     <select  id="book">
                <option value="1" v-for="(bookname,index) in name"> {{index}} {{bookname}}</option>
        </select>
    ```

    - bookname代表数组元素，名称可变，index可表示索引名称可变，in为关键字固定，name表示数组名称

    ```javascript
    
            var seletlist=new Vue({
                el:book,
                data:{
                    name:['Spring5','Mybatis','SpringMVC','Maven']
                }
            })
    ```

  - v-model指令

    ```html
    <span  id="text">
            <input type="text" v-model="message">
            <p>{{ message }}</p>
        </span>
    ```

    ```javascript
     var textarea=new Vue({
                el:"#text",
                data: {
                    message:"傻逼"
                }
            })
    ```

    - v-model实现双向绑定，在上述例子中只要message改变或者文本框中改变都会导致另一方的修改，两边始终是同步的

- # Vue组件

  - 组件

    - 整个页面布局比如顶部是一个组件，其中包含了html，CSS，JS的信息，实现代码复用
    - 单文件组件
      - .Vue结尾，该文件只有一个组件
    - 非单文件组件
      - 一个文件中多个组件，非vue结尾

  - 使用组件三步骤

    - 创建组件

      ```vue
       const hello= Vue.extend({
              template:'<p>{{message}}</p>',
              data(){
                  return {
                      message:'gg1231231'
                  }
              }
          })
      ```

      - 简写方式

        ```javascript
        const hello= {
            template:'<p>{{message}}</p>',
            name:'ttt',
            data(){
                return {
                    message:'666'
                }
            }
        }
        ```

    - 注册组件

      - 局部注册组件

        ```javascript
        const vm=new Vue({
                el:'#aa',
                components:{
                    hello:hello
                }
            })
        ```

      - 全局注册组件

        ```javascript
        Vue.component("Hello",hello);
        ```

      - 关于组件的命名

        - hello组件在Vue中显示是Hello

        - hello-world作为组件名称需要添加引号

        - Hello-World等类似的命名只能在cli中使用，光引入js文件不行

        - 可以使用以下的方式修改Vue开发工具的名称

          ```javascript
          const hello= Vue.extend({
                  template:'<p>{{message}}</p>',
                  name:'ttt',
                  data(){
                      return {
                          message:'666'
                      }
                  }
              })
          ```

    - 使用组件

      ```javascript
      <div id="aa">
          <hello>
          </hello>
      </div>
      ```

      - 使用组件时单标签也可以（但是需要Vue脚手架支持）

    - 组件嵌套

      ```javascript
      const student=Vue.extend({
              template: '<a>123</a>'
          })
      
          const school= Vue.extend({
              template:"<p>傻逼Vue" +
                  "<br>" +
                  "<student></student>" +
                  "</p>",
              data(){
                  return {
                      message:'666'
                  }
              },
              components:{
                  student:student
              }
          })
      ```

  - 组件创建分析

    - 通过将组件输出可以得知组件本身是一个构造函数，称为VueComponent
    - 当页面中识别到我们自定义的组件标签时会调用extend，在extend中调用VueComponent
    - 每次调用extend返回的都是一个全新的VueComponent
    - 在之前定义 Vue对象的方式中this代表是Vue对象而在组件创建中this指的是VueComponent

  - ## 单文件组件

    - 使用.Vue文件完成组件的创建和注册等工作

    - 由于浏览器无法直接识别Vue文件所以可以通过一下渠道将Vue文件转为js文件

      - Webpack（较为麻烦）
      - Vue脚手架

    - 单文件组件基本结构

      ```Vue
      <template>
      <div>this is my Vue</div>
        <!--编写HTML文件-->
      </template>
      
      <script>
      //编写JS代码
      export default {
        name: "MyVue"
      }
      </script>
      
      <style scoped>
      
      /*编写CSS代码*/
      </style>
      ```

    - 编写第一个Vue文件

      ```Vue
      <template>
      <div>this is my Vue</div>
        <!--编写HTML文件-->
        <div id="gg">{{message}}</div>
      </template>
      
      <script>
      //编写JS代码
      export default {
        name: "MyVue",
        data(){
          return {
            message:"abc"
          }
        }
      }
      </script>
      
      <style scoped>
      
      /*编写CSS代码*/
      .gg{
        background: #634eb0;
      }
      </style>
      ```

      - 如果App.Vue要使用该组件则需要将该组件暴露出，一下给出三种暴露方法

        - export暴露

          - 这种方式只能暴露对象，如果要暴露单独的值则需要封装

            ```javascript
            function aa(){
                alert('123');
            }
            export {aa}//使用对象封装该function
            ```

          - 采用这种方式暴露的对象可以通过以下两种方式引入

            ```javascript
            //假设暴露aa的js文件是aa.js
            import {aa} from './aa.js'
            aa();
            
            //或者采用这种方式
            const aa = require('./test1.js');//这样会引入整个暴露的对象并将其存储在const aa中
            ```

        - export default (多使用这种方式)

          - 这种暴露方式只能暴露一个变量

            ```javascript
            function test(){  console.log('test');}
            export default test;
            
            //对于多个值也可以这样写
            const a=2
            function test(){  console.log('test');}
            export default {a,test}
            ```

          - 这种暴露方式只能采用import引入

            ```javascript
            //针对单个引入
            import test from './test.js'
            test();
            //针对对象引入
            import test from './test.js'
            test.test()
            ```

        - exports

          - 只能暴露对象

            ```javascript
            function test(){  console.log('test');}
            exports.test = test;//exports.自定义名称=需要暴露的对象
            ```

            - 同样采用import导入

              ```javascript
              import test from './test.js'
              test.test()
              ```

    - 编写App.Vue

      ```Vue
      <template>
        <img alt="Vue logo" src="./assets/logo.png">
        <HelloWorld msg="Welcome to Your Vue.js App"/>
        <MyVue></MyVue>
      </template>
      
      <script>
      import HelloWorld from './components/HelloWorld.vue'
      import MyVue from './components/MyVue.vue'
      
      export default {
        name: 'App',
        components: {
          HelloWorld,
          MyVue
        }
      }
      </script>
      
      <style>
      
      </style>
      
      ```

      - app.vue是最终最大的组件，也是整个页面，而该组件需要通过main.js进行调用，而在main.js中就是创建Vue的地方

    - 编写Main.js

      ```javascript
      
      import App from './App.vue'
      
      new Vue({
          el:'#root',
          template:'<App></App>',
          components:{
              App
          }
      })
      ```

    - 最终呈现的页面是index.html页面，在index.html中引入main.js文件和vue.js文件

      ```html
      
      <div id="app"></div>
      <script src="vue.js"/>
      <script src="main.js"/>
      ```

    - 以上这些只是大概的形式实际上无法运行，只能通过脚手架运行

  - ## 使用Vue脚手架

    - 安装脚手架

      ```sh
      npm install -g @vue/cli
      #下载较慢则安装淘宝镜像
      npm config set registry https://registry.npm.taobao.org
      ```

    - 切换到在需要创建项目的目录下
    
      ```sh
      vue create ***(项目名称)
      ```

    - 在该项目根目录下执行以下指令运行项目
    
      ```sh
      npm run serve
      ```
    
    - 使用Vue-cli创建的项目的项目结构
    
      - bavel.cfongi.js（ES6转ES5配置文件）
    
      - package-lock.json，package.json(由npm允运行的项目都含有这两个文件，主要用于配置，里面主要包括了一些常用命令)
    
        - 主要命令
          - serve（运行）
          - build（打包）
          - lint（编译js检查）
    
      - main.js
    
        ```javascript
        //引入createApp方法
        import { createApp } from 'vue'
        //引入app组件
        import App from './App.vue'
        
        createApp(App).mount('#app')
        
        ```
    
        只要一执行npm run serve就会直接运行该文件
    
      - assets文件夹
    
        放置静态资源
    
      - compontents
    
        放置组件文件
    
      - public文件夹下的html文件是容器
    
        - index.html文件
    
          ```html
          <!DOCTYPE html>
          <html lang="">
            <head>
              <meta charset="utf-8">
                <!--让IE以最高形式渲染该页面-->
              <meta http-equiv="X-UA-Compatible" content="IE=edge">
                <!--配置移动端-->
              <meta name="viewport" content="width=device-width,initial-scale=1.0">
                <!--配置网站图标,BASE_URL=public文件夹-->
              <link rel="icon" href="<%= BASE_URL %>favicon.ico">
                 <!--配置网站标题，在package.json中的name属性-->
              <title><%= htmlWebpackPlugin.options.tit                                                                                                                                           le %></title>
            </head>
            <body>
                 <!--当浏览器不支持JS时noscript就会被渲染-->
              <noscript>
                <strong>We're sorry but <%= htmlWebpackPlugin.options.title %> doesn't work properly without JavaScript enabled. Please enable it to continue.</strong>
              </noscript>
              <div id="app"></div>
              <!-- built files will be auto injected -->
            </body>
          </html>
          
          ```
    
      - vue.config.js（可选）
    
        vue的配置文件，可修改一些默认的配置如何配置可以在Vue官方网站上查看
    
  - ## Vue组件
  
    - 使用ref代替原生dom
  
      ```vue
      <template>
      
      <div>
        <button ref="gg" @click="show">abc</button>
      </div>
      
      </template>
      
      <script>
      export default {
        name: "School",
        methods:{
          show:function (){
              //这种方式同样适用于组件标签，只不过获取的是组件实例消息而通过Dom的Id获取到的是解析过后的组件
            console.log(this.$refs.gg);
              //使用原生Dom,this.$refs.gg替换为domcument.getElementById(Id)
          }
        }
      
      }
      </script>
      
      ```
  
    - props配置
  
      - 案例
  
        ```vue
        <template>
          <div>
            <div>{{name}}</div>
            <div>{{id}}</div>
            <div>{{email}}</div>
          </div>
        </template>
        
        <script>
        export default {
          name: "Student",
          data() {
              return{
                name: 'zhangsan',
                id: '1',
                email: '123@qq.com'
              }
          }
        }
        </script>
        ```
  
        - 如果要复用以上组件但是要求数据不同可采用props配置
  
          ```vue
          <template>
            <div>
              <div>{{name}}</div>
              <div>{{id}}</div>
              <div>{{email}}</div>
            </div>
          </template>
          
          <script>
          export default {
            name: "Student",
            props:['name','id','email']
          }
          </script>
          
          ```
  
          - 使用（直接通过属性指定）
  
            ```vue
             <Student name="张三" email="123@qq.com" id="1"></Student>
              <hr/>
              <Student name="李四" email="121233@qq.com" id="2"></Student>
              <hr/>
              <Student name="王五" email="1212123133@qq.com" id="3"></Student>
              <hr/>
            ```
  
            - 直接通过属性指定的属性的类型默认都是字符串，如果想要使用其他类型可以采用v-bind
            
              ```vue
              <!--解析过后id为4--> 
              <Student name="王五" email="1212123133@qq.com" :id="3+1"></Student>
              ```
            
          - props的第二种写法(不会出现类型混乱的问题,同时需要配合v-bind使用)
          
            ```vue
            <template>
              <div>
                <div>{{name}}</div>
                <div>{{id}}</div>
                <div>{{email}}</div>
              </div>
            </template>
            
            <script>
            export default {
              name: "Student",
              props:{
                  name:String,
                  email:String,
                  id:Number
              }
            }
            </script>
            
            ```
          
          - props的第三种写法
          
            ```vue
            
            <script>
            export default {
              name: "Student",
              props:{
                name:{
                  required:true
                },
                email:{
                  type:String,
                  default:'123@qq.com'
                }//省略id
              }
            }
            </script>
            
            ```
          
        - props中的值都是可以通过函数修改但是尽量不要修改，如果需要修改props展示的消息可以采用以下写法
        
          ```vue
           <template>
            <div>
          
              <button @click="showThis">增加age</button>
              <div>age:{{Myage}}</div>
            </div>
          </template>
          
          <script>
          export default {
            name: "Student",
            data(){
              return {
                Myage:this.age
              }
            },
            methods:{
              showThis(){
                this.Myage++;
              }
            },props:{
              age:Number
            }
          }
          </script>
          
          ```
        
          - 这是由于props的优先级高于data
    
    - mixin
    
      - 对于同一个组件中相同的部分可以抽离出来放在一个单独的JS文件中（即可以作用于函数也可以作用于数据，如果存在相同的则本地文件的优先，但是mounts例外，mounts会全部执行，只是顺序会有所变化）
    
        ```js
        //js文件
        export const mix={
            methods:{
                showName()
                {
                    alert(this.name);
                }
            }
        }
        
        
        //vue文件（局部混合）
        
        <template>
        
        <div>
          <button ref="gg" @click="showName">abc</button>
          <div>{{name}}</div>
        </div>
        
        </template>
        
        <script>
        import {mix} from '../assets/mix.js'
        export default {
          name: "School",
          data(){
            return{
              name:'张三'
            }
          },
          mixins:[mix]
        
        }
        </script>
        
        ```
    
        - 以上方式只是局部混合
    
      - 全局混合
    
        ```js
        //mix.js
        export const mix={
            methods:{
                showName()
                {
                    alert(this.name);
                }
            }
        }
        
        
        
        
        //main.js
        import App from './App.vue'
        import {mix} from './assets/mix.js'
        import {createApp} from "vue";
        
        
        const app=createApp(App)
        app.mixin(mix)
        app.mount('#app')
        
        ```
    
    
    - Vue定义插件和使用插件
    
      - 定义插件
    
        ```js
        import {mix} from './mix.js'
        export default {
        
            install(Vue)//可传入参数Vue
            {
                Vue.mixin(mix)//可以通过Vue设置mixin
                //还可以在插件中定义全局过滤器
                
            }
        }
        ```
    
      - 使用插件
    
        ```js
        import plugins from './assets/plugins'
        
        const app=createApp(App)
        app.use(plugins)
        app.mount('#app')
        ```
    
    
    - Vue CSS渲染
    
      - 相同选择器的渲染按照引入的先后进行渲染
    
      - 如果想要同一组件的CSS只对同一组件的进行渲染则需要添加Scoped属性
    
        ```vue
        <style scoped>
        </style>
        ```
    
    - TODOLists案例
    
      - 组件开发流程
    
        - 编写静态组件
    
          