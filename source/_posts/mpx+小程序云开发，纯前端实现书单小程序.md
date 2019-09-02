> 著作权归作者所有。商业转载请联系luotingv1获得授权，非商业转载请注明出处[务必保留全文，勿做删减]。

### 前言

感谢[roberthuang](https://juejin.im/post/5c341e1d6fb9a049f66c4876)提供的思路。借助云开发和小程序的便利,正值公司项目没有重大更新，趁着这段空闲时间,自己构思了书单分享的小程序原型图和后台数据结构。

![](https://user-gold-cdn.xitu.io/2019/9/2/16cefb979d51ed23?w=2000&h=594&f=jpeg&s=60882)

### 准备工作
> 1.  mpx框架 [mpx官方文档](https://didi.github.io/mpx)
> 2.  小程序·云开发 [小程序·云开发文档](https://link.juejin.im?target=https%3A%2F%2Fdevelopers.weixin.qq.com%2Fminiprogram%2Fdev%2Fwxcloud%2Fbasis%2Fgetting-started.html)

### 项目结构介绍
![](https://user-gold-cdn.xitu.io/2019/9/2/16cefa298e7e6e9d?w=306&h=424&f=png&s=31143)

static目录： 放一些公共资源，如js,css,json.

function目录：云函数存放目录

components目录：组件相关的.vue文件都放在这里

pages目录：所有页面都放在这个目录

store目录：类似vuex的实现

app.json文件：

     {
        "pages": [
          "./pages/index",
          "./pages/publish",
          "./pages/personal",
          "./pages/bookDetail",
          "./pages/myBook",
          "./pages/myPublish",
          "./pages/myCollect",
          "./pages/login",
          "./pages/publishBook",
          "./pages/addBook"
        ],
        "window": {
          "backgroundTextStyle": "light",
          "navigationBarBackgroundColor": "#fff",
          "navigationBarTitleText": "书单圈",
          "navigationBarTextStyle": "black",
          "backgroundColor": "#F6F6F6",
          "backgroundTextStyle": "dark",
          "enablePullDownRefresh": false
        },
        "tabBar": {
          "color": "#5A5A5A",
          "selectedColor": "#5cadff",
          "borderStyle": "white",
          "backgroundColor": "#ffffff",
          "borderStyle": "black",
          "list": [{
              "pagePath": "pages/index",
              "iconPath": "../static/images/shuyou.png",
              "selectedIconPath": "../static/images/shuyou1.png",
              "text": "广场"
            },
            {
              "pagePath": "pages/publish",
              "iconPath": "../static/images/fabu.png",
              "selectedIconPath": "../static/images/fabu1.png",
              "text": "发布"
            },
            {
              "pagePath": "pages/personal",
              "iconPath": "../static/images/my.png",
              "selectedIconPath": "../static/images/my1.png",
              "text": "我的"
            }
          ]
        },
        "usingComponents": {
          "i-button": "./components/iview/button"
        },
        "cloud": true
    }
 
   App.vue文件：本人主要是为了增加项目更新后的提醒，所以在这个文件加了些相关内容，内容如下，

     createApp({
        onLaunch() {
          // 检测小程序是否有新版本更新
          if (wx.canIUse('getUpdateManager')) {
            const updateManager = wx.getUpdateManager()
            updateManager.onCheckForUpdate(function (res) {
              // 请求完新版本信息的回调
              if (res.hasUpdate) {
                updateManager.onUpdateReady(function () {
                  wx.showModal({
                    title: '更新提示',
                    content: '新版本已经准备好，是否重启应用？',
                    success: function (res) {
                      if (res.confirm) {
                        // 新的版本已经下载好，调用 applyUpdate 应用新版本并重启
                        updateManager.applyUpdate()
                      }
                    }
                  })
                })
                // 小程序有新版本，会主动触发下载操作（无需开发者触发）
                wx.getUpdateManager().onUpdateFailed(function () {
                  // 当新版本下载失败，会进行回调
                  wx.showModal({
                    title: '提示',
                    content: '检查到有新版本，下载失败，请检查网络设置',
                    showCancel: false
                  })
                })
              }
            })
          } else { // 版本过低则无法使用该方法
            wx.showModal({
              title: '提示',
              confirmColor: '#5BB53C',
              content: '当前微信版本过低，无法使用该功能，请升级到最新微信版本后重试。'
            })
          }
        }
     })

   
 ### 云开发介绍

 [小程序云开发文档](https://link.juejin.im?target=https%3A%2F%2Fdevelopers.weixin.qq.com%2Fminiprogram%2Fdev%2Fwxcloud%2Fbasis%2Fgetting-started.html)
如果你用mpx云开发 可以看[mpx-cloud](https://juejin.im/post/5c341e1d6fb9a049f66c4876)项目
*   project.config.json文件：
```
"cloudfunctionRoot":"functions"
```

进行云开发首先我们需要找到上面这个文件，在上面这个json文件中加上面这句

 `cloudfunctionRoot` 用于指定存放云函数的目录

 *   app.json文件：
 
    "window": {
        "backgroundTextStyle": "light",
        "navigationBarBackgroundColor": "#fff",
        "navigationBarTitleText": "WeChat",
        "navigationBarTextStyle": "black"
    },
    "cloud": true

 增加字段 `"cloud": true实现云开发能力的兼容性`

*   开通云开发

 在开发者工具工具栏左侧，点击 “云开发” 按钮即可开通云开发
 
![](https://user-gold-cdn.xitu.io/2019/9/2/16cefe58f80d3b90?w=904&h=518&f=png&s=67438)

*   数据库

 云开发提供了一个类似mongodb的数据库
 
 我们必须构建符合我们项目数据存储要求的集合,这里主要提供2个集合的截图
 
![](https://user-gold-cdn.xitu.io/2019/9/2/16cf00c0bbd9bac7?w=1166&h=603&f=png&s=104532)

![](https://user-gold-cdn.xitu.io/2019/9/2/16cf00c757c82e4f?w=1198&h=626&f=png&s=109606)
bookList集合 有收藏操作的记录collect 点赞操作记录 like 
user集合 有收藏记录collect 点赞记录 like 个人信息uesr

*   存储

云开发提供了一块存储空间，提供了上传文件到云端、带权限管理的云端下载能力，开发者可以在小程序端和云函数端通过 API 使用云存储功能。
![](https://user-gold-cdn.xitu.io/2019/9/2/16cefe7debeeaa34?w=813&h=514&f=png&s=74558)

*   云函数

 云函数是一段运行在云端的代码，无需管理服务器，在开发工具内编写、一键上传部署即可运行后端代码。
 > 下面开始讲解使用云开发的过程：

1.  云能力初始化，在小程序端开始使用云能力前，需先调用 `wx.cloud.init` 方法完成云能力初始化

        wx.cloud.init({
          env: '云开发环境ID'
        })
    
 2. 数据库的使用：

 在开始使用数据库 API 进行增删改查操作之前，需要先获取数据库的引用。以下调用获取默认环境的数据库的引用：

 `const db = wx.cloud.database`

要操作一个集合，需先获取它的引用：

`const todos = db.collection`

详情查看云开发小程序端api文档
> 接下来是操作数据库的相关示例：

常见获取当前操作用户的openid

主要是为了获取当前操作用户的openid,获取当前用户的openid方法：
 
     getOpenId() {
      const that = this
      wx.cloud.callFunction({
        name: 'user',
        data: {}
      }).then(res=>{
        that.openId = res.result.openid
        that.getIsExist()
      })
    }

 接着判断当前用户是否已经存在于数据库中，即getIsExist()方法：
 
         getIsExist() {
            let user = this.db.collection('user')
            user.where({
              _openid: this.openId
            }).get().then(res => {
              if (res.data.length === 0) {
                this.addUser()
              }else{
                 wx.navigateBack({
                    delta: 1
                  })
              }
            })
          }
        

如果得到的数组长度为零则添加改用户到数据库，否则则提醒当前用户已经送过祝福

       addUser() {
        let user = this.db.collection('user');
        user.add({
          data: {
            user: this.userInfo
          },
          success:()=>{
              wx.navigateBack({
                delta: 1
              })
          }
        })
      }
    

存入到数据库的信息是这样的：

![](https://user-gold-cdn.xitu.io/2019/1/7/168266611e8d7daa?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)


 > 下面给大家看看云函数的使用方法：
 
我们大部分情况在要获取数据并有分页操作的情况下会使用云函数来获取数据,先给个demo
在云函数指定目录建立一个文件夹类似下面目录
![](https://user-gold-cdn.xitu.io/2019/9/2/16ceff8a0b6a0fab?w=168&h=66&f=png&s=5601)

下面是云函数getClassify的index.js文件：
```
const cloud = require('wx-server-sdk')
cloud.init()
const db = cloud.database()
const MAX_LIMIT = 100
exports.main = async (event, context) => {
  // 先取出集合记录总数
  const countResult = await db.collection('classify').count()
  const total = countResult.total
  // 计算需分几次取
  const batchTimes = Math.ceil(total / 100)
  // 承载所有读操作的 promise 的数组
  const tasks = []
  for (let i = 0; i < batchTimes; i++) {
    const promise = db.collection('classify').skip(i * MAX_LIMIT).limit(MAX_LIMIT).get()
    tasks.push(promise)
  }
  // 等待所有
  return (await Promise.all(tasks)).reduce((acc, cur) => ({
    data: acc.data.concat(cur.data),
    errMsg: acc.errMsg
  }))
}

```

使用云函数前，在开发者工具上，找到getClassify目录，右键如图：

 ![](https://user-gold-cdn.xitu.io/2019/1/7/16826660f26d696a?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

点击上传并部署：云端安装依赖（不上传node_modules）

安装完点击完成就能使用当前云函数了，使用方法即：

       getClassify(context) {
          wx.cloud.callFunction({
            name: 'getClassify'
          }).then(res => {
            let arr=res.result.data;
            arr.unshift({
              id: "",
              name: '请选择分类'
            })
            context.commit('setClassify', arr)
          })
        }

> 常见的翻页操作

 下面是云函数pagination的index.js文件：
    
    const cloud = require('wx-server-sdk')
    cloud.init()
    const db = cloud.database()
    exports.main = async (event, context) => {
    //event 前端上传的集合 
      var dbName = event.dbName; //集合名称
      var filter = event.filter ? event.filter : null; //筛选条件,默认为空 格式{_id:'XXXXXX'}
      var pageIndex = event.pageIndex ? event.pageIndex : 1; //当前第几页,默认为第一页
      var perPage = event.perPage ? event.perPage : 10;
      const countResult = await db.collection(dbName).where(filter).count(); //获取集合中总记录数
      const total = countResult.total; //得到总记录数
      const totalPage = Math.ceil(total / perPage); //计算需要多少页
      var hasMore;//提示前端是否还有数据
      if(pageIndex>totalPage||pageIndex==totalPage){//如果没有数据了,就返回false
        hasMore=false;
      }else{
        hasMore=true;
      }
      // 最后查询数据并返回给前端
      return db.collection(dbName).where(filter).skip((pageIndex-1)*perPage).limit(perPage).get().then(res=>{
        res.hasMore=hasMore;
        return res;
      })
    }

> 常见收藏和点赞操作

    const cloud = require('wx-server-sdk')
    cloud.init()
    const db = cloud.database();
    const _ = db.command;
    exports.main = async (event, context) => {
      try {
        // wxContext内包含用户的openId
        const wxContext = cloud.getWXContext()
        const openId = wxContext.OPENID; //获取openID
        let record = await db.collection('bookLists').doc(event.id).get() //查询符合id的书单记录
        record=record.data;
        let arrName = event.name;
        // 定义空数组
        let arr = [];
        if (record[arrName] && record[arrName].length > 0) {
          // 让定义的数组等于用户操作的当前书单下的like数组
          arr = record[arrName];
          // 定义一个计数变量
          let count = 0
          // 循环遍历，当openId相同时替换like数组中的相同项，并存储对应的type
          arr.forEach((item, index) => {
            if (item.openId === openId) {
              count++;
              arr.splice(index, 1, {
                openId: openId,
                type: event.type,
              })
            }
          })
          // 当计数变量为0时，说明在这条书单中，like数组中未存储过此用户，直接push此用户并存储type
          if (count === 0) {
            arr.push({
              openId: openId,
              type: event.type
            })
          }
        } else {
          // 如果此条书单like数组本身就为空，直接push当前用户并存储type
          arr.push({
            openId: openId,
            type: event.type
          })
        }
        // 通过云开发操作数据库的相关api,即update通过_id来更新集合中某条数据
        let obj = {
          data: {
            [arrName]: arr,
            [arrName + '_num']: event.type == 1 ? _.inc(1) : _.inc(-1)
          }
        };
        let collect=await db.collection('user').where({_openid:openId}).get();//获取用户记录
        //判断用户是否存储过该操作信息
        collect=collect.data[0];
        let arr2=[];
        if(collect[arrName]&&collect[arrName].length>0){
              // 循环遍历 如果有记录删除 如果没有记录添加  因为只有type为1才会添加
              arr2=collect[arrName];
              let count = 0
              arr2.forEach((item, index) => {
                if (item[arrName+'_id'] === event.id) {
                  count++;
                  if(event.type==1){
                    arr2.splice(index, 1,{
                      [arrName+'_id']: event.id
                    })
                  }else{
                    arr2.splice(index, 1)
                  }
                }
              })
              if (count === 0&&event.type==1) {
                arr2.push({
                  [arrName+'_id']: event.id
                })
              }
        }else if(event.type==1){
          arr2.push({
            [arrName+'_id']: event.id
          })
        }
        db.collection('user').doc(collect._id).update({data:{[arrName]:arr2}});
         await db.collection('bookLists').doc(event.id).update(obj);
         return await db.collection('bookLists').doc(event.id).get();
      } catch (e) {
        console.error(e)
      }
    }


### 总结
> 在这里给大家讲解几点细节：

1.  数据库集合名首先必须要建立；
2.  云函数新建后一定要创建并安装依赖；
3.  数据库建立集合后记得将权限放开，否则小程序端无法操作数据库.如下图：

![](https://user-gold-cdn.xitu.io/2019/1/9/168305bc7c22e861?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

> 最后总结：

 除了一些静态资源放在项目中，其他资源建议一律存储在云开发-存储管理中，这样的话，方便更换资源而不用再次提交等待小程序审核，更换后的资源可以立即生效；

**源码地址：** https://github.com/luotingv1/book
