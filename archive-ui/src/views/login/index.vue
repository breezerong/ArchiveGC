<template>
  <div class="login-container pull-height" ref="loginContainer" :style="{height: containerHeight+'px'}" v-title :data-title="$t('application.name')">
    
    <div class="login-border">
        <template v-if="isSSO==false">
          <el-row>
            <el-col :span="12">
              <div>
                <img src="../../assets/images/top_images/logo.png" width="80px" height="80px" alt="logo" style="vertical-align: middle;" />
              </div>
              <div>
                <h2 class="login-info-title">{{$t("application.name")}}</h2>
              </div>
            </el-col>
            <el-col :span="12">
              <h4 class="login-title">用户登录
              </h4>
              <userLogin></userLogin>
            </el-col>
          </el-row>
        </template>
        <template v-else>
          <el-table
            v-loading="true"
            element-loading-text="正在登录，请稍等"
            element-loading-spinner="el-icon-loading"
            element-loading-background="#fff"
          >
          </el-table>
        </template>
      </div>
  </div>
</template>
<script>
import userLogin from './userlogin'
import { mapGetters } from 'vuex'
export default {
  name: 'login',
  components: {
    userLogin
  },
  data() {
    return {
      activeName: 'user',
      isSSO : false,
      containerHeight: window.innerHeight - 1
    }
  },
  created() {
    var query = window.location.href;
    let _self = this;
    if(query.indexOf("?")>-1){
      var queryParam = query.substring(query.indexOf("?")+1);
      try {
          if(queryParam.indexOf("loginName")>-1&&queryParam.indexOf("sign")>-1&&queryParam.indexOf("encrySign")>-1){
          _self.isSSO = true;
          queryParam = decodeURI(queryParam)
          var params = queryParam.split("&");
          var paramMap =  new Map();
          params.forEach(function(value,index){
            paramMap.set(value.split("=")[0],value.split("=")[1]);
          });
          _self.autoLogin(paramMap);
        }
      } catch (error) {
        console.log(error)
        _self.isSSO = false;
        _self.$message(_self.$t("message.SSOloginFailured"));
      }
    }
    // var integrityurl = window.location.href;
    // //是否包含ticket
    // let _self = this;
    // if(integrityurl.indexOf("ticket")>-1){
    //   _self.isSSO = true;
    //   //var thirdPartyInterface = "http://10.100.3.168/sso/proxyValidate";
    //   var thirdPartyInterface = "http://127.0.0.1:8089/sso/proxyValidate";
    //   var m = new Map();
    //   var ticket = integrityurl.substring(integrityurl.lastIndexOf("ticket")+7)
    //   m.set("ticket",ticket);
    //   m.set("service",thirdPartyInterface)
    //   axios.post(thirdPartyInterface,JSON.stringify(m)).then(function(response){
    //     if(response!=null){
    //       var userName = response.data
    //       console.log(userName)
    //       _self.autoLogin(userName);
    //     }
    //   })
    // }else{
    //   console.log("外部登录")
    // }
  },
  mounted() {
    let _self = this;
    window.addEventListener("resize",function(){
      _self.containerHeight = window.innerHeight - 1;
    });
  },
  computed: {
    ...mapGetters(['website'])
  },
  props: [],
  methods: {
    autoLogin(userParamMap) {
      let _self = this;
      var tocomp = _self.$route.query.redirect;
      if (!tocomp) {
        tocomp = "/";
      }
      console.log(userParamMap)
      axios
        .post("/archive/userLogin", JSON.stringify(userParamMap))
        .then(function(response) {
          //console.log(response.data);
          if (response.data.code == 1) {
            _self.setCurrentUser(response.data.data);
            sessionStorage.setItem("access-token", response.data.token);
            sessionStorage.setItem("access-userName", response.data.userName);
            sessionStorage.setItem("access-department",  response.data.department);
            if (_self.rememberInfo) {
              localStorage.setItem("ziecm-rememberInfo", "1");
              localStorage.setItem("ziecm-ass12bn", _self.account.username);
              localStorage.setItem("ziecm-ass12bp", _self.account.password);
            } else {
              localStorage.removeItem("ziecm-rememberInfo");
              localStorage.removeItem("ziecm-ass12bn");
              localStorage.removeItem("ziecm-ass12bp");
            }
            _self.$router.push({ path: tocomp });
          } else if(response.data.code == 2){
            _self.isSSO = false;
            _self.$message(response.data.msg);
          }
          else{
            _self.isSSO = false;
            _self.$message(response.data.msg);
          }
        })
        .catch(function(error) {
          console.log(error);
          _self.isSSO = false;
          _self.$message(_self.$t("message.SSOloginFailured"));
        });
    }
  }
}
</script>

<style lang="scss">
.login-container {
  display: flex;
  align-items: center;
  justify-content: space-around;
  background: rgba(0, 0, 0, 0.2);
  position: relative;
}
.login-container::before {
  z-index: -999;
  content: "";
  position: absolute;
  left: 0;
  top: 0;
  width: 100%;
  height: 100%;
  background-image: url("../../assets/images/top_images/login_bg_cnpe2.jpg");
  background-repeat: no-repeat;

  background-size:100% 100%;
}
.login-info {
  padding-left: 60px;
}
.login-info-title {
  line-height: 90px;
  color: #fff;
}
.login-info-item {
  font-size: 14px;
}
.login-border {
  display: flex;
  justify-content: center;
  flex-direction: column;
  padding: 30px 50px 25px 50px;
  background-color: #fff;
  border-radius: 8px;
  box-shadow: 1px 1px 2px #eee;
  background-image: url("../../assets/images/top_images/login_box.png");
  background-repeat: no-repeat;
  background-size:100% 100%;
}
.login-main {
  border-radius: 3px;
  box-sizing: border-box;
  background-color: #fff;

}
.login-main > h3 {
  margin-bottom: 20px;
}
.login-main > p {
  color: #76838f;
}
.login-title {
  margin: 0 0 20px;
  text-align: center;
  color: #409eff;
  letter-spacing: 3px;
}
.login-submit {
  width: 100%;
  border-radius: 28px;
}
.login-form {
  margin: 10px 0;
  .el-form-item__content {
    width: 200px;
  }
  .el-form-item {
    margin-bottom: 12px;
    margin-left: 10px;
  }
  .el-input {
    input {
      border-color: #dcdcdc;
      border-radius: 3px;
    }
    .el-input__prefix {
      i {
        padding: 0 5px;
        font-size: 16px !important;
      }
    }
  }
}
.login-code {
  display: flex;
  align-items: center;
  justify-content: space-around;
  margin: 0 0 0 10px;
}
.login-code-img {
  margin-top: 2px;
  width: 100px;
  height: 32px;
  background-color: #fdfdfd;
  border: 1px solid #f0f0f0;
  color: #333;
  font-size: 14px;
  font-weight: bold;
  letter-spacing: 5px;
  line-height: 32px;
  text-indent: 5px;
  text-align: center;
}
</style>