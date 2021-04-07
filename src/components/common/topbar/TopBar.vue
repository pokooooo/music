<template>
  <div class="top-bar">
    <div class="title">
      <img src="~assets/image/demo.svg" alt="">
      <div>网易云音乐</div>
      <div><a href=""><i class="el-icon-arrow-left" @click.stop="back"></i></a></div>
      <div>
        <el-input
          size="mini"
          class="inputt"
          placeholder="请输入内容"
          v-model="input"
          clearable
          suffix-icon="el-icon-search"
          @keyup.enter.native="togSearch"
        ></el-input>
      </div>
      <div class="login">
        <img :src="userImg" alt=""style="width: 40px; height: 40px; border-radius: 50%; padding-top: 10px">
        <div style="margin-top: 10px">
          <span style="fontSize:16px; margin-left: 10px" v-if="currentUserInfo">{{currentUserInfo.nickname}}</span>
          <el-button type="text" @click="dialogFormVisible = true" style="margin-left: 10px" v-else>登录</el-button>
          <el-button type="text" @click="logout" v-if="currentUserInfo !== null" style="margin-left: 10px;">退出</el-button>
        </div>
      </div>
      <el-dialog title="手机登录" :visible.sync="dialogFormVisible" width="30%">
        <img src="~assets/image/login.svg" style="display: block;margin: 10px auto"/>
        <el-form :model="form" :rules="loginInfoRules">
          <el-form-item label="手机号" :label-width="formLabelWidth">
            <el-input v-model="form.phone" autocomplete="off"></el-input>
          </el-form-item>
          <el-form-item label="密码" :label-width="formLabelWidth">
            <el-input v-model="form.password" autocomplete="off" type="password"></el-input>
          </el-form-item>
        </el-form>
        <div slot="footer" class="dialog-footer">
          <el-button @click="cancelLogin">取 消</el-button>
          <el-button type="primary" @click="confirmLogin">确 定</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>

<script>
  import {getPhoneLogin, getPhoneLoginOut, getUserSonglist} from "network/login";
  import {USER_SONGLIST} from '@/store/mutationType'

  export default {
    name: "TopBar",
    data() {
      return {
        input: "",// 关联输入框内容
        dialogFormVisible: false, //显示登陆框
        userImg: require('assets/image/logo.png'),//默认用户头像
        currentUserInfo:
          window.localStorage.getItem("currentUserInfo") === "null"
            ? null
            : JSON.parse(window.localStorage.getItem("currentUserInfo")),//查看浏览器是否已存有数据
        loginInfoRules: {
          phone: [{ required: true, message: " ", trigger: "blur" }],
          password: [{ required: true, message: " ", trigger: "blur" }],
        },// 登录格式的校验
        form: {
          phone: "",
          password: "",
        },
        formLabelWidth: "80px", //登录内输入框的宽度
      }
    },
    methods: {
      //返回上一级
      back() {
        this.$router.back()
      },
      //搜索框有数据则跳转页面
      togSearch(){
        if(this.input === ''){
          this.$toast.show("不能为空哦~",2000);
          return;
        };
        this.$router.push('/search' + this.input)
      },
      //获取用户的歌单 放到vuex上
      getUserSonglist(userId) {
        getUserSonglist(userId).then((res) => {
          this.$store.commit(USER_SONGLIST,res.playlist);
        });
      },
      //退出接口登录 告诉接口已退出 并还原数据等操作
      getPhoneLoginOut() {
        getPhoneLoginOut().then((res) => {
          console.log(res);
          //当前用户信息设置为null
          this.currentUserInfo = null;
          //存储用户信息的localstorage设置为null
          window.localStorage.setItem("currentUserInfo", null);
          this.userImg = require("assets/image/logo.png");
          this.$store.commit(USER_SONGLIST,[]);
        });
      },
      //退出登录
      logout() {
        console.log("退出登录");
        //跳转到主页面
        this.$router.push('/home/rtstyle');
        this.getPhoneLoginOut();
      },
      //点击登录界面的 取消
      cancelLogin() {
        this.dialogFormVisible = false;
      },
      //点击登录页面的 确定
      confirmLogin() {
        this.getPhoneLogin(this.form.phone, this.form.password);
      },
      // 验证登录账户和密码
      getPhoneLogin(phone, pwad) {
        getPhoneLogin(phone, pwad).then((res) => {
          //信息不符时
          if (res === undefined) {
            //登录框不消失
            this.dialogFormVisible = true;
            //弹出提示信息
            this.$toast.show("该手机号码不存在或未注册", 2000);
            return;
          }
          //其他错误时
          if (res.code !== 200) {
            this.dialogFormVisible = true;
            //提示对应的错误信息
            this.$toast.show(res.message, 2000);
            return;
          }
          //保存用户信息,退出时删除
          window.localStorage.setItem(
            "currentUserInfo",
            JSON.stringify(res.profile)
          );
          //放入用户的信息 图片 登录框禁止 提示
          this.currentUserInfo = res.profile;
          this.userImg = this.currentUserInfo.avatarUrl;
          this.dialogFormVisible = false;
          this.$toast.show(`欢迎你${this.currentUserInfo.nickname}`, 2000);
          this.getUserSonglist(this.currentUserInfo.userId);
        });
      },
    },
    created() {
      //判断本地是否有用户信息
      if (this.currentUserInfo) {
        this.userImg = this.currentUserInfo.avatarUrl;
        this.getUserSonglist(this.currentUserInfo.userId);
      }
    },
  }
</script>

<style scoped>
  .top-bar{
    background: rgba(0, 0, 255, 0.1);
    height: 60px;
    display: flex;
  }

  .title {
    margin-left: 100px;
    display: flex;
    width: 100%;
    position: relative;
  }

  .title>img{
    margin-top: 5px;
    height: 50px;
    width: 50px;
  }

  .login{
    position: absolute;
    right: 100px;
    display: flex;
  }

  .title>div{
    color: black;
    margin: auto 10px;
    font-size: 25px;
  }



</style>
