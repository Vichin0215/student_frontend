<template>
    <el-dialog v-dialogDrag title="修改密码" :visible.sync="dialog" :close-on-click-modal=false append-to-body class="level-one-dialog" width="500px">
      <el-form ref="form" :model="form" :rules="rules">
        <el-form-item label="旧密码：" prop="password" :label-width="formLabelWidth">
          <el-input type="password" show-password v-model="form.password" maxlength="15" clearable placeholder="请输入旧密码"></el-input>
        </el-form-item>
        <el-form-item label="新密码：" prop="newPassword" :label-width="formLabelWidth">
              <el-input type="password" show-password v-model="form.newPassword" maxlength="15" clearable placeholder="请输入新密码"></el-input>
        </el-form-item>
        <el-form-item label="确认密码：" prop="passwordAgain" :label-width="formLabelWidth">
          <el-input  type="password" show-password v-model="form.passwordAgain" maxlength="15" clearable placeholder="请再次输入新密码"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer">
        <el-button type="primary" @click="click">确定</el-button>
        <el-button>取消</el-button>
      </div>
    </el-dialog>
</template>

<script>
    export default {
        name: "password",
        data () {
          const validPassword = (rule, value, callback) => {
            if(value === this.form.password){
              callback(new Error('新密码不可与旧密码一致'))
            } else if (value !== this.form.newPassword){
              callback(new Error('两次密码不一致'))
            }else{
              callback()
            }
          };
          const validNewPassword = (rule, value, callback) => {
            if(value === this.form.password){
              callback(new Error('新密码不可与旧密码一致'))
            }else{
              callback()
            }
          };
          return {
            formLabelWidth: '120px',
            dialog: false,
            userInfo:{},
            form: {
              username: '',
              password: '',
              newPassword: '',
              passwordAgain: ''
            },
            rules: {
              password: [
                {required: true, message: '密码不可为空', trigger: 'blur'}
              ],
              newPassword:[
                {required: true, message: '新密码不可为空', trigger: 'blur'},
                {validator: validNewPassword, trigger: 'blur'}
              ],
              passwordAgain:[
                {required: true, message: '新密码不可为空', trigger: 'blur'},
                {validator: validPassword, trigger: 'blur'}
              ]
            }
          }
        },
        methods: {
          init () {
            this.dialog = true;
            this.clearForm();
            let userInfo = JSON.parse(localStorage.userInfo);
            this.form.username = userInfo.username;
            this.form.level = userInfo.level;
            this.form.userID = userInfo.userId;
          },
          clearForm () {
            this.form =  {
              username: '',
              userID:'',
              password: '',
              newPassword: '',
              passwordAgain: '',
              level: null
            };
          },
          click () {
            console.log(this.form)
            this.$refs['form'].validate((valid) => {
              if (valid) {
                if(this.userInfo.password === this.form.password) {
                  let userID = '';
                  if(this.userInfo.level === 0){
                    userID = this.userInfo.adminId
                  } else if (this.userInfo.level === 1) {
                    userID = this.userInfo.teacherId
                  } else if (this.userInfo.level === 2) {
                    userID = this.userInfo.studentId
                  }

                  this.axiosHelper.put('/student_backend/password?level='+this.userInfo.level + '&newPassword=' + this.form.newPassword + '&userID=' + userID,{}).then(
                    response => {
                      if (response.data) {
                        this.$message.success({
                          message: '密码更改成功，请重新登录'
                        });
                        this.$router.push("/login");
                        // 退出登录时，清除身份信息
                        localStorage.clear();
                        this.dialog = false;
                      } else {
                        this.$message.error({
                          message: '旧密码输入不正确'
                        })
                      }
                    }
                  )
                } else {
                  this.$message.error({
                    message: '旧密码输入不正确'
                  })
                }
              } else {
                return false;
              }
            })
          }
        },

      mounted(){
        this.userInfo = JSON.parse(localStorage.userInfo);
      }
    }

</script>

<style scoped>

</style>
