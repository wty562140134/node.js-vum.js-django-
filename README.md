# node.js+vum.js+django前后分离 
## 开发阶段的配置
1.安装Node.js<br>
2.安装vue-cli<br>
3.创建django项目<br>

    django-admin startproject <project_name> <path>
  找到ALLOWED_HOSTS = []改成ALLOWED_HOSTS = ['*']<br>
   
4.django后台跨域
   
   安装django-cors-headers和配置后台跨域.settings.py文件中修改
   
    pip install django-cors-headers
   
    INSTALLED_APPS = [
    'django.contrib.admin',
    #'horizon.apps.HorizonConfig', # 这是一个前段框架中间件这里可以不使用
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'corsheaders', # django解决npm跨域
    ]
    
    MIDDLEWARE = [
    'django.middleware.security.SecurityMiddleware',
    'django.contrib.sessions.middleware.SessionMiddleware',
    'django.middleware.common.CommonMiddleware',  # django解决npm跨域
    # 'django.middleware.csrf.CsrfViewMiddleware', # 这个注释了是防治django组织跨域请求,也可以在代码中使用@csrf_exempt
    'django.contrib.auth.middleware.AuthenticationMiddleware',
    'django.contrib.messages.middleware.MessageMiddleware',
    'django.middleware.clickjacking.XFrameOptionsMiddleware',
    'corsheaders.middleware.CorsMiddleware',  # django解决npm跨域
    ]
    
    ###################################################################
    """
    django解决跨域
    """
    CORS_ALLOW_CREDENTIALS = True

    CORS_ORIGIN_ALLOW_ALL = False  # 必须是False，否则cookie传递不了

    CORS_ORIGIN_WHITELIST = (
        'localhost:8000',
    )  # 这里可以添加白名单

    CORS_ALLOW_METHODS = (
        'DELETE',
        'GET',
        'OPTIONS',
        'PATCH',
        'POST',
        'PUT',
        'VIEW',
    )  # 允许跨域的请求

    CORS_ALLOW_HEADERS = (
        'XMLHttpRequest',
        'X_FILENAME',
        'accept-encoding',
        'authorization',
        'content-type',
        'dnt',
        'origin',
        'user-agent',
        'x-csrftoken',
        'x-requested-with',
        'Pragma',
    )  # 允许跨域请求的http头
    ###################################################################
  
5.在django目录与app同级创建前端Vue项目<br>

    vue init webpack <projectname>
6.在项目目录下安装依赖包<br>

    npm install axios # axios是vue官方推荐的前后端互通信息的插件
    npm install qs # qs是将表单内容组织成后台能够使用的数据的插件
    npm install element-ui # element-ui是基于vue开发的一套UI组件，可以很快的开发出一个叫好看的网站
   [element-ui官网](http://element-cn.eleme.io/#/zh-CN/component/installation)
    
   若安装依赖中出现以下告警:
   npm WARN ajv-keywords@2.1.1 requires a peer of ajv@^5.0.0 but none is installed. You must install peer dependencies yourself
    
      npm install -g npm-install-peers
   npm WARN optional SKIPPING OPTIONAL DEPENDENCY: fsevents@1.2.7 (node_modules\fsevents):<br>
   
   npm WARN notsup SKIPPING OPTIONAL DEPENDENCY: Unsupported platform for fsevents@1.2.7: wanted {"os":"darwin","arch":"any"} (current: {"os":"win32","arch":"x64"})<br>
   fsevent是mac osx系统的，你是在win或者Linux下使用了 所以会有警告，忽略即可<br>
   
   
7.引入依赖包<br>
   在项目目录下src/main.js中引入依赖包<br> 
   import 'element-ui/lib/theme-chalk/index.css'一定要放在import App from './App'之上不然自己写的样式会被覆盖<br>
   
    import Vue from 'vue'
    import ElementUI from 'element-ui'// 导入element-ui
    import 'element-ui/lib/theme-chalk/index.css'//导入element-ui css
    import App from './App'
    import router from './router'
    import Axios from 'axios'// 导入前端请求发送插件
    import Qs from 'qs'// 导入表单内容组织成后台能够使用的数据的插件
    Axios.defaults.withCredentials = true//这句不写有可能后台拒绝post请求
    Vue.prototype.qs = Qs// 设置全局的表单插件
    Vue.prototype.$ajax = Axios// 设置全局的请求发送插件
    Vue.config.productionTip = false
    Vue.use(ElementUI)// 设置全局使用element-ui
    
   Vue前端跨域index.js中修改:<br>
   在config/index.js中找到proxyTable添加<br>
   
    proxyTable: {
      '/api': {
        target: 'http://127.0.0.1:8000',
        changeOrigin: true,
        pathRewrite: {
          '^/api': ''
        }
      }
    },
   
   关闭useEslint,不然会语法检查错误:
   
    useEslint: false,//关闭eslint检查防止报错

8.Vue前端请求数据方法<br>

   在App.vue中<br>
   这里/api在proxyTable中被pathRewrite:所定义的空字符串代替,就剩下/index<br>
   所以实际被替换为:http://127.0.0.1:8000/index/<br>
   注意:index/参数需要和后台urls中的路由参数一致,不然无法正常发送请求<br>
   
    <template>
        <div id="app">
            <img src="./assets/logo.png">
            <input type="button" value="发送" @click="send"></input>
            <router-view/>
        </div>
    </template>
    
    <script>
        export default {
        name: 'App',
        methods: {
          send: function () {
            console.log("测试按钮")
            //这里/api在proxyTable中被pathRewrite:所定义的空字符串代替,就剩下/index
            // 所以实际被替换为:http://127.0.0.1:8000/index/
            //注意index/参数需要和后台urls中的路由参数一致,不然无法正常发送请求
            this.$ajax.post('/api/index/')
            console.log('测试222')
            }
          }
        }
    </script>
    
9.后端django接收数据方法:<br>
    这里为了方便直接在urls中添加index方法<br>
    
     def index(request):
        print('>>>>>>>>>>>>>>>>>>', request)
        data = {
          'data': '前后分离测试'
        }
        return HttpResponse(simplejson.dumps({'data': data}))
        
    urlpatterns = [
    path('index/', index),  # 测试前后分离
    ]
    
## 部署生产阶段的配置
   1.安装express-generator生成器
        express是基于node.sj开发的可以作为一个前端服务器和反向代理服务器,类似于nginx<br>
        
    npm install express-generator -g
   2.创建一个express项目
   
    express <project_name>
   3.进入expressDemo目录，安装项目依赖
    
    npm install
   4.解决生产阶段跨域<br>
    (1).进入到所创建express项目下修改app.js加入以下配置
        
            /*
            HTTP前端服务器解决跨域
             */
            app.all('*', function (req, res, next) {
                res.header("Access-Control-Allow-Origin", "*");//项目上线后改成页面的地址
                res.header("Access-Control-Allow-Headers", "X-Requested-With,Content-Type");
                res.header("Access-Control-Allow-Methods", "PUT,POST,GET,DELETE,OPTIONS");
                next();
            });

            /*
            HTTP前端服务器请求代理设置,这一段一定要放在处理404前面,不然会导致页面请求无法正常到达后端服务器
             */
            var proxyMiddleWare = require('http-proxy-middleware');
            var proxyOption = {
                target: 'http://127.0.0.1:8000',
                changeOrigoin: true,
                ws: true,
                pathRewrite: {
                    '^/api': ''
                }
            };
            app.use(express.static(__dirname + "public"));
            app.use("/api", proxyMiddleWare(proxyOption));
            app.listen(4000);
   5.把dist目录下的所有文件复制到express项目的public文件夹下后退出到express项目根路径后启动服务
   
    npm start
