<template>
  <v-app>
    <v-content>
      <v-container fluid fill-height>
        <v-layout align-center justify-center>
          <v-flex xs12 sm8 md4>
            <v-card class="elevation-12">
              <v-toolbar dark color="primary">
                <v-toolbar-title>学生注册</v-toolbar-title>
                <v-spacer></v-spacer>
              </v-toolbar>
              <v-card-text>
                <v-form ref="registerForm">
                  <v-text-field
                    prepend-icon="person"
                    v-model="username"
                    label="用户名"
                    :error-messages="errorMsg4"
                    v-on:blur="checkUsername"
                    type="text"/>
                  <v-text-field
                    prepend-icon="phone"
                    v-model="phone"
                    label="手机号"
                    :rules="[rules.phone]"
                    type="text"/>
                  <v-text-field
                    prepend-icon="phone"
                    v-model="studentNo"
                    label="学号"
                    :error-messages="errorMsg5"
                    @change="checkStudentNo"
                    type="text"/>
                  <v-text-field
                    prepend-icon="lock"
                    v-model="password"
                    label="密码"
                    id="password"
                    :rules="[rules.passwordRequired, rules.passwordLength]"
                    :append-icon="e1 ? 'visibility' : 'visibility_off'"
                    :append-icon-cb="() => (e1 = !e1)"
                    :type="e1 ? 'text' : 'password'"
                    :error-messages="errorMsg"
                  ></v-text-field>
                  <v-text-field
                    prepend-icon="lock"
                    v-model="checkPassword"
                    label=" 确认密码"
                    id="checkPassword"
                    v-on:blur="cpasswordCheck"
                    :append-icon="e1 ? 'visibility' : 'visibility_off'"
                    :append-icon-cb="() => (e1 = !e1)"
                    :type="e1 ? 'text' : 'password'"
                    :error-messages="errorMsg"
                  ></v-text-field>

                  <v-layout row>
                    <v-flex xs8>
                      <v-text-field
                        prepend-icon="el-icon-edit"
                        label="验证码"
                        v-model="writeCode"
                        v-on:blur="checkWriteCode"
                        :error-messages="errorMsg2"
                      ></v-text-field>
                    </v-flex>
                    <v-flex xs4>
                      <div class="code" @click="refreshCode">
                        <sidentify :identifyCode="identifyCode"></sidentify>
                      </div>
                    </v-flex>
                  </v-layout>

                  <v-layout row>
                    <v-flex xs8>
                      <v-text-field
                        prepend-icon="el-icon-edit"
                        label="手机验证码"
                        v-model="phoneCode"
                        :error-messages="errorMsg3"
                      ></v-text-field>
                    </v-flex>
                    <v-flex xs4>
                      <v-btn color="primary" @click="getPhoneCode">获取手机验证码</v-btn>
                    </v-flex>
                  </v-layout>

                </v-form>
              </v-card-text>
              <v-card-actions>
                <v-spacer></v-spacer>
                <v-btn color="primary" @click="doLogin">返回登录</v-btn>
                <v-btn color="primary" @click="regist">注册</v-btn>
              </v-card-actions>
            </v-card>
          </v-flex>
        </v-layout>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
  import sidentify from '../components/identify'

  export default {
    data: () => ({
      id: "",
      username: "",
      password: "",
      phone: "",
      phoneCode: "",
      studentNo: "",

      checkPassword: "",
      dialog: false,
      e1: false,
      identifyCodes: "1234578qwqertyuipasfghjjklzxcvnmQWERTYUIPASDFGHJKLZXCVBNM",
      identifyCode: "",
      writeCode: "",
      errorMsg: [],
      errorMsg2: [],
      errorMsg3: [],
      errorMsg4: [],
      errorMsg5: [],
      rules: {
        passwordRequired: v => !!v || '密码不能为空',
        passwordLength: v => (v.length >= 4 && v.length <= 16) || '密码要在4~16位',
        phone: v => {
          const phoneReg = /^1[34578]\d{9}$/;
          return phoneReg.test(v) || '手机号格式不正确';
        }
      },
      checkPasswordError: true,
      writeCodeError: true,
      usernameError: true,
      studentNoError: true,
      phoneCodeError: true
    }),
    components: {
      sidentify
    },
    methods: {
      doLogin() {
        this.$router.push("/login");
      },
      //随机数
      randomNum(min, max) {
        return Math.floor(Math.random() * (max - min) + min)
      },
      //说明随机数格式
      refreshCode() {
        this.identifyCode = ''
        this.makeCode(this.identifyCodes, 4)
      },
      //生成随机数
      makeCode(o, l) {
        for (let i = 0; i < l; i++) {
          this.identifyCode += this.identifyCodes[
            this.randomNum(0, this.identifyCodes.length)
            ]
        }
      },
      cpasswordCheck() {

        if (this.password != this.checkPassword) {
          this.errorMsg = ['确认密码与密码不符'];
          this.checkPasswordError = true;
        } else {
          this.errorMsg = [];
          this.checkPasswordError = false;
        }
        return true;
      },
      checkPhoneCode() {

        if (this.writeCode == null) {
          this.errorMsg3 = ['手机验证码不能为空'];
          this.writeCodeError = true;
        } else {
          this.errorMsg3 = [];
          this.writeCodeError = false;
        }

        return true;
      },
      checkUsername() {

        if (this.username == null || this.username == "") {
          this.errorMsg4 = ['用户名不能为空'];
          this.usernameError = true;
          return true;
        } else if (this.username.length < 4 && this.username > 16) {
          this.errorMsg4 = ['用户名要在4~16位'];
          this.usernameError = true;
          return true;

        } else {
          this.$http({
            method: 'get', // 动态判断是POST还是PUT
            url: '/user/check/' + this.username + '/1'
          }).then(() => {
            this.errorMsg4 = [];
            this.usernameError = false;
          }).catch(() => {
            this.errorMsg4 = ['用户名重复'];
            this.usernameError = true;
            return true;

          });
        }

        return true;
      },
      checkWriteCode() {
        if (this.writeCode == null || this.writeCode == "") {
          this.errorMsg2 = ['验证码不能为空'];
          this.writeCodeError = true;
          return true;
        } else if (this.identifyCode.toUpperCase() != this.writeCode.toUpperCase()) {
          this.errorMsg2 = ['验证码错误'];
          this.writeCodeError = true;
          return true;

        } else {
          this.errorMsg2 = [];
          this.writeCodeError = false;
        }
        return true;
      },
      regist() {
        let user = {
          id: this.id,
          username: this.username,
          password: this.password,
          phone: this.phone,
          code: this.phoneCode
        }
        if (this.$refs.registerForm.validate() && !this.checkPasswordError) {
          this.$http({
            method: 'post', // 动态判断是POST还是PUT
            url: '/user/register',
            data: this.$qs.stringify(user)
          }).then(() => {
            this.$router.push("/login");
          }).catch(() => {
            this.errorMsg3 = ['手机验证码错误'];
          });
        }
      },
      getPhoneCode() {
        this.checkWriteCode();
        if (this.$refs.registerForm.validate() && !this.checkPasswordError && !this.writeCodeError && !this.usernameError) {
          this.$http({
            method: 'post', // 动态判断是POST还是PUT
            url: '/user/send',
            data: "phone=" + this.phone
          }).then(() => {
            this.refreshCode();
          });
        }
      },
      checkStudentNo() {
        if (this.studentNo == null || this.studentNo == "") {
          this.errorMsg5 = ['绑定学号不能为空'];
          this.studentNoError = true;
          return true;
        } else {
          this.$http({
            method: 'get',
            url: '/user/student/getIdByStudentNo/' + this.studentNo
          }).then(resp => {
            this.id = resp.data;
            this.$http({
              method: 'get', // 动态判断是POST还是PUT
              url: '/user/check/' + this.id + '/3'
            }).then(() => {
              this.errorMsg5 = [];
              this.usernameError = false;
            }).catch(() => {
              this.errorMsg5 = ['该学号已经被注册'];
              this.usernameError = true;
              return true;

            });
          }).catch(() => {
            this.errorMsg5 = ['请重新检查学号'];
            this.studentNoError = true;
            return true;
          });
        }
        return true;
      }
    }
  };
</script>
