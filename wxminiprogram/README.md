# wxminiprogram

## 微信小程序项目框架一般是什么样

```
.
├── miniprogram
│   ├── app.json 小程序全局配置
│   ├── app.ts 小程序入口
│   ├── app.wxss 小程序全局样式
│   ├── pages 存放各个页面
│   │   ├── index 首页
│   │   │   ├── index.json 页面配置参数
│   │   │   ├── index.ts 页面逻辑
│   │   │   ├── index.wxml 页面结构
│   │   │   └── index.wxss 页面样式
│   │   └── logs 
│   │       ├── logs.json 
│   │       ├── logs.ts
│   │       ├── logs.wxml
│   │       └── logs.wxss
│   └── utils 工具类
│       └── util.ts
├── package.json 项目依赖管理文件
├── project.config.json 微信小程序项目配置文件
├── project.private.config.json 私有项目配置文件，存储不公开的基本信息
├── tsconfig.json typescript的配置文件，例如编译器设置，输出目录···
└── typings typescript类型定义，用于为项目中的 JavaScript 库或外部 API 提供类型提示
    ├── index.d.ts
    └── types
        ├── index.d.ts
        └── wx
            ├── index.d.ts
            ├── lib.wx.api.d.ts
            ├── lib.wx.app.d.ts
            ├── lib.wx.behavior.d.ts
            ├── lib.wx.cloud.d.ts
            ├── lib.wx.component.d.ts
            ├── lib.wx.event.d.ts
            └── lib.wx.page.d.ts
```

所以一般就只需要考虑 app.ts 和 pages/ 即可

## index/ 的几个文件结构都简单说一下？

```json
index.json

{
  "usingComponents": { 声明小程序使用了什么外部组件
  }
}

```

```ts
// index.ts
// 获取应用实例
const app = getApp<IAppOption>()
const defaultAvatarUrl = 'https://mmbiz.qpic.cn/mmbiz/icTdbqWNOwNRna42FI242Lcia07jQodd2FJGIYQfG0LAJGFxM4FbnQP6yfMxBgJ0F3YRqJCJ1aPAK2dQagdusBZg/0'

定义当前页面或组件的行为和数据绑定
Component({
    // 一些数据
  data: {
    motto: 'Hello World',
    userInfo: {
      avatarUrl: defaultAvatarUrl,
      nickName: '',
    },
    hasUserInfo: false,
    canIUseGetUserProfile: wx.canIUse('getUserProfile'),
    canIUseNicknameComp: wx.canIUse('input.type.nickname'),
  },
  // 一些方法
  methods: {
    // 事件处理函数
    bindViewTap() {
      wx.navigateTo({
        url: '../logs/logs',
      })
    },
    onChooseAvatar(e: any) {
      const { avatarUrl } = e.detail
      const { nickName } = this.data.userInfo
      this.setData({
        "userInfo.avatarUrl": avatarUrl,
        hasUserInfo: nickName && avatarUrl && avatarUrl !== defaultAvatarUrl,
      })
    },
    onInputChange(e: any) {
      const nickName = e.detail.value
      const { avatarUrl } = this.data.userInfo
      this.setData({
        "userInfo.nickName": nickName,
        hasUserInfo: nickName && avatarUrl && avatarUrl !== defaultAvatarUrl,
      })
    },
    getUserProfile() {
      // 推荐使用wx.getUserProfile获取用户信息，开发者每次通过该接口获取用户个人信息均需用户确认，开发者妥善保管用户快速填写的头像昵称，避免重复弹窗
      wx.getUserProfile({
        desc: '展示用户信息', // 声明获取用户个人信息后的用途，后续会展示在弹窗中，请谨慎填写
        success: (res) => {
          console.log(res)
          this.setData({
            userInfo: res.userInfo,
            hasUserInfo: true
          })
        }
      })
    },
  },
})

```

```html
<!--index.wxml-->
滚动区域
<scroll-view class="scrollarea" scroll-y type="list">
  <view class="container">
    <!-- 用户信息 -->
    <view class="userinfo">
      <!-- 可以用昵称并且有用户信息 -->
      <block wx:if="{{canIUseNicknameComp && !hasUserInfo}}">
        <button class="avatar-wrapper" open-type="chooseAvatar" bind:chooseavatar="onChooseAvatar">
          <image class="avatar" src="{{userInfo.avatarUrl}}"></image>
        </button>
        <view class="nickname-wrapper">
          <text class="nickname-label">昵称</text>
          <input type="nickname" class="nickname-input" placeholder="请输入昵称" bind:change="onInputChange" />
        </view>
      </block>
      <!-- 如果没有用户信息 -->
      <block wx:elif="{{!hasUserInfo}}">
        <button wx:if="{{canIUseGetUserProfile}}" bindtap="getUserProfile"> 获取头像昵称 </button>
        <view wx:else> 请使用2.10.4及以上版本基础库 </view>
      </block>
      <!-- 否则~ -->
      <block wx:else>
        <image bindtap="bindViewTap" class="userinfo-avatar" src="{{userInfo.avatarUrl}}" mode="cover"></image>
        <text class="userinfo-nickname">{{userInfo.nickName}}</text>
      </block>
    </view>
    <view class="usermotto">
      <text class="user-motto">{{motto}}</text>
    </view>
  </view>
</scroll-view>

```

```css

样式文件，不多说~

/**index.wxss**/
page {
  height: 100vh;
  display: flex;
  flex-direction: column;
}
.scrollarea {
  flex: 1;
  overflow-y: hidden;
}

.userinfo {
  display: flex;
  flex-direction: column;
  align-items: center;
  color: #aaa;
  width: 80%;
}

.userinfo-avatar {
  overflow: hidden;
  width: 128rpx;
  height: 128rpx;
  margin: 20rpx;
  border-radius: 50%;
}

.usermotto {
  margin-top: 200px;
}

.avatar-wrapper {
  padding: 0;
  width: 56px !important;
  border-radius: 8px;
  margin-top: 40px;
  margin-bottom: 40px;
}

.avatar {
  display: block;
  width: 56px;
  height: 56px;
}

.nickname-wrapper {
  display: flex;
  width: 100%;
  padding: 16px;
  box-sizing: border-box;
  border-top: .5px solid rgba(0, 0, 0, 0.1);
  border-bottom: .5px solid rgba(0, 0, 0, 0.1);
  color: black;
}

.nickname-label {
  width: 105px;
}

.nickname-input {
  flex: 1;
}

```
