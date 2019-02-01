<template>
    <div class="container">
        <button open-type="getUserInfo">授权</button>
        <div class="userinfo">
            <img :src="userinfo.avatarUrl" alt="" @click="login"/>
            <p>{{userinfo.nickName}}</p>
        </div>
        <YearProgress></YearProgress>
        <button v-if='userinfo.openId' class="btn"  @click="scanBook">添加图书</button>
    </div>
</template>
<script>
import qcloud from 'wafer2-client-sdk'
import YearProgress from '@/components/YearProgress'
import {get, showSuccess, post, showModal} from '@/util'
import config from '@/config'
export default {
    components:{
        YearProgress
    },
   data(){
       return {
           userinfo:{
                avatarUrl: '../../../../../static/img/unlogin.png',
                nickName: '点击登录'
           }
       }
   },
   onShow(){
        let userinfo = wx.getStorageSync('userinfo')
        console.log('userinfo')
        console.log(userinfo)
        if(userinfo){
            this.userinfo=userinfo
        }
    },
   methods:{
       async addBook(isbn){
           console.log(isbn)
           const res = await post('/weapp/addbook',{
               isbn,
               openid: this.userinfo.openId
           })
           showModal('添加成功',`${res.title}添加成功`)
       },
       scanBook(){
           wx.scanCode({
               success: (res)=>{
                   console.log(res)
                   if(res.result){
                       this.addBook(res.result)
                   }
               }
           })
       },
       login(){
           let user = wx.getStorageSync('userinfo')
            console.log('user')
            console.log(user)
            const self = this
            if (!user) {
                qcloud.setLoginUrl(config.loginUrl)
                qcloud.login({
                    success: function (userinfo) {
                        qcloud.request({
                            url: config.userUrl,
                        login: true,
                        success (userRes) {
                            showSuccess('登录成功')
                            wx.setStorageSync('userinfo', userRes.data.data)
                            self.userinfo = userRes.data.data
                        }
                    })
                }

                })
            }
        },
        
   }
}
</script>
<style lang="scss">
.container{
  padding:0 30rpx;

}  
.userinfo{
  margin-top:100rpx;
  text-align:center;
  img{
    width: 150rpx;
    height:150rpx;
    margin: 20rpx;
    border-radius: 50%;
  }
}
</style>
