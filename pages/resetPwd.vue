<template>
  <div class="resetPwd">
    <div class="reset-step">
      <h3 class="title">重置密码</h3>
      <el-steps :active="active" finish-status="success">
        <el-step title="接受验证码"> </el-step>
        <el-step title="重置密码"></el-step>
      </el-steps>
      <div class="reset-con">
        <div v-if="active == 0">
          <el-form
            ref="resetForm1"
            :model="resetForm1"
            :rules="resetForm1Rules"
            class="reset-form"
          >
            <el-form-item prop="email">
              <el-input
                v-model="resetForm1.email"
                type="text"
                auto-complete="off"
                placeholder="请输入邮箱"
              >
                <svg-icon
                  slot="prefix"
                  icon-class="user"
                  class="el-input__icon input-icon"
                />
              </el-input>
            </el-form-item>
            <el-form-item prop="code" v-if="captchaEnabled">
              <el-input
                v-model="resetForm1.code"
                auto-complete="off"
                placeholder="验证码"
                style="width: 63%"
              >
                <svg-icon
                  slot="prefix"
                  icon-class="validCode"
                  class="el-input__icon input-icon"
                />
              </el-input>
              <div class="reset-code">
                <img :src="codeUrl" @click="getCode" class="reset-code-img" />
              </div>
            </el-form-item>
            <el-form-item style="width: 100%">
              <el-button
                style="width: 100%"
                :loading="loading"
                type="success"
                @click.native.prevent="handleEmail"
                >下一步</el-button
              >
            </el-form-item>
          </el-form>
        </div>
        <div v-if="active == 1">
          <el-form
            ref="resetForm2"
            :model="resetForm1"
            :rules="resetForm2Rules"
            class="reset-form"
          >
            <el-form-item prop="email">
              <el-input
                v-model="resetForm1.email"
                type="text"
                auto-complete="off"
                placeholder="请输入邮箱"
                :disabled="true"
              >
                <svg-icon
                  slot="prefix"
                  icon-class="user"
                  class="el-input__icon input-icon"
                />
              </el-input>
            </el-form-item>

            <el-form-item prop="password">
              <el-input
                v-model="resetForm1.password"
                type="password"
                auto-complete="off"
                placeholder="请输入新密码"
              >
                <svg-icon
                  slot="prefix"
                  icon-class="password"
                  class="el-input__icon input-icon"
                />
              </el-input>
            </el-form-item>

            <el-form-item prop="code" v-if="captchaEnabled">
              <el-input
                v-model="resetForm1.code"
                auto-complete="off"
                placeholder="邮箱验证码"
                style="width: 63%"
              >
                <svg-icon
                  slot="prefix"
                  icon-class="validCode"
                  class="el-input__icon input-icon"
                />
              </el-input>
              <div class="reset-code"></div>
            </el-form-item>

            <el-form-item style="width: 100%">
              <el-button
                style="width: 100%"
                :loading="loading"
                type="success"
                @click.native.prevent="handlePwd"
                >修改密码</el-button
              >
            </el-form-item>
          </el-form>
        </div>
      </div>
    </div>
  </div>
</template>

<script>
import { getCodeImg } from "@/api/login";
import userInfoApi from "@/api/userInfo";
import { getToken, setToken, removeToken } from "@/utils/auth";

export default {
  name: "resetPwd",
  data() {
    return {
      active: 0,
      resetForm1: {
        code: "",
        email: "",
        uuid: "",
      },
      resetForm1Rules: {
        email: [
          { required: true, message: "请输入邮箱地址", trigger: "blur" },
          {
            type: "email",
            message: "请输入正确的邮箱地址",
            trigger: ["blur", "change"],
          },
        ],
        code: [{ required: true, trigger: "change", message: "请输入验证码" }],
      },
      resetForm2Rules: {
        code: [{ required: true, trigger: "change", message: "请输入验证码" }],
        password: [
          { required: true, trigger: "blur", message: "请输入您的密码" },
        ],
      },

      codeUrl: "",
      captchaEnabled: true,
      loading: false,
    };
  },
  created() {
    this.getCode();
  },
  methods: {
    getCode() {
      getCodeImg().then((res) => {
        this.captchaEnabled =
          res.captchaEnabled === undefined ? true : res.captchaEnabled;
        if (this.captchaEnabled) {
          this.codeUrl = "data:image/gif;base64," + res.img;
          this.resetForm1.uuid = res.uuid;
        }
      });
    },
    handleEmail() {
      this.$refs.resetForm1.validate((valid) => {
        if (valid) {
          userInfoApi
            .sendResetPwdCode(this.resetForm1)
            .then((respose) => {
              this.$message({
                type: "success",
                message: "正在发送验证码到邮箱!",
              });
              this.resetForm1.code = "";
              this.active = 1;
            })
            .catch((response) => {
              this.getCode();
            });
        }
      });
    },
    handlePwd() {
      this.$refs.resetForm2.validate((valid) => {
        if (valid) {
          userInfoApi
            .changePwd(this.resetForm1)
            .then((respose) => {
              this.$message({
                type: "success",
                message: "修改成功!",
              });
              removeToken();
              // this.resetForm1.code = "";
              // this.active = 0;
              var count = 3; //赋值多少秒
              var times = setInterval(() => {
                count--; //递减
                if (count <= 0) {
                  clearInterval(times);
                  this.$router.push({ path: "/userlogin" });
                } else {
                  this.$message.warning(
                    "将再 " + count + " 秒后跳转到登录页面"
                  );
                }
              }, 1000); //1000毫秒后执行
            })
            .catch((response) => {});
        }
      });
    },
  },
};
</script>

<style>
.resetPwd {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 100%;
}

.reset-step {
  border-radius: 6px;
  background: #ffffff;
  width: 400px;
  padding: 25px 25px 5px 25px;
}
.title {
  margin: 0px auto 30px auto;
  text-align: center;
  color: #707070;
}
.reset-con {
  margin-top: 20px;
  padding: 20px;
}

.el-input input {
  height: 38px;
}
.reset-code {
  width: 33%;
  height: 38px;
  float: right;
}
.reset-code img {
  cursor: pointer;
  vertical-align: middle;
  height: 38px;
}
</style>