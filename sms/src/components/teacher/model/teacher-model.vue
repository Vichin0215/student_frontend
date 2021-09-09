<template>
  <el-dialog v-dialogDrag :title="title" :visible.sync="dialog" :close-on-click-modal=false append-to-body width="800px">
    <el-form ref="form" :model="form" :rules="rules">
      <el-row>
        <el-col :span="12">
        <el-form-item label="编号：" prop="teacherId" :label-width="formLabelWidth">
          <el-input v-model="form.teacherId" maxlength="15" clearable></el-input>
        </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="姓名：" prop="teacherName" :label-width="formLabelWidth">
            <el-input v-model="form.teacherName" maxlength="15" clearable></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="性别：" :label-width="formLabelWidth">
            <el-select v-model="form.teacherSex" style="width: 100%">
              <el-option v-for="item in sexArr" :key="item.value" :label="item.label" :value="item.value"></el-option>
            </el-select>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="任课专业：" prop="major" :label-width="formLabelWidth">
            <el-input v-model="form.major" maxlength="30" clearable></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="密码：" prop="password" :label-width="formLabelWidth">
            <el-input v-model="form.password" maxlength="15" clearable></el-input>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <div slot="footer">
      <el-button type="primary" @click="click('form')" size="small">确定</el-button>
      <el-button @click="cancel" size="small" >取消</el-button>
    </div>
  </el-dialog>
</template>

<script>
  export default {
    name: "student-model",
    data () {
      const validEmail = (rule, value, callback) => {
        const mailReg = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(.[a-zA-Z0-9_-])+/;
        if (!value) {
          return callback(new Error('电子邮箱不能为空'))
        } else {
          if (mailReg.test(value)) {
            callback()
          } else {
            callback(new Error('请输入正确的邮箱格式'))
          }
        }
      };
      const validPhone = (rule, value, callback) => {
        const phoneReg = /^1[3|4|5|7|8][0-9]{9}$/
        if (!value) {
          return callback(new Error('手机号码不能为空'))
        } else {
          if (!Number.isInteger(+value)) {
            callback(new Error('请输入数字值'))
          } else {
            if (phoneReg.test(value)) {
              callback()
            } else {
              callback(new Error('电话号码格式不正确'))
            }
          }
        }
      };
      return {
        doneNum: 0,
        formLabelWidth: '100px',
        dialog: false,
        type: '',
        title: '',
        form: {
          teacherId: '',
          teacherName: '',
          teacherSex: '',
          major: '',
          password: '',
        },
        sexArr: [{
          value: '男',
          label: '男'
        }, {
          value: '女',
          label: '女'
        }],
        rules: {
          teacherId: [
            { required: true, message: '编号不能为空', trigger: 'blur' },
          ],
          teacherName: [
            {required: true, message: '姓名不能为空', trigger: 'blur'},
          ],
          teacherSex: [
            {required: true, message: '性别不能为空', trigger: 'blur'},
          ],
          major: [
            {required: true, message: '任课专业不能为空', trigger: 'blur'},
          ],
          password: [
            {required: true, message: '密码不能为空', trigger: 'blur'},
          ],
        }
      }
    },
    methods: {
      init (obj) {
        this.clearForm();
        this.type = obj.type;
        if (obj.type === 'add') {
          this.title = '新增';
        } else if (obj.type === 'edit') {
          this.title = '修改';
          this.form = obj.row
        }
        this.dialog = true;
      },
      clearForm () {
        this.type = '';
        this.form = {
          teacherId: '',
          teacherName: '',
          teacherSex: '',
          major: '',
          password: '',
        };
        if(this.$refs['form'] !== undefined) {
          this.$refs['form'].clearValidate();
        }
      },
      click (formName) {
        this.$refs[formName].validate((valid) => {
          if (valid) {
            if (this.type === 'add') {
              this.addMethod();
            } else if (this.type === 'edit') {
              this.editMethod();
            }
          } else {
            return false;
          }
        })
      },
      addMethod () {
        ++this.doneNum;
        if (this.doneNum === 1) {
          this.add();
        }
      },
      add () {
        let obj = {
          kind: 1,
          Class1: this.form.class1,
          teacherName: this.form.teacherName,
          teacherId: this.form.teacherId,
          teacherSex: this.form.teacherSex,
          password: this.form.password,
          major: this.form.major,
        };

        this.axiosHelper.get(
          '/student_backend/add', {params: obj}).then(() => {
          this.doneNum = 0;
          this.$message.success({
            message: '新增成功'
          });
          this.dialog = false;
          this.$emit('search')
        }).catch(() => {
          this.doneNum = 0;
          this.$message.error({
            message: '新增失败'
          });
          this.dialog = false;
        });
      },
      editMethod () {
        this.axiosHelper.put(
          '/api/sms/user/teacher', this.form).then(() => {
          this.$message.success({
            message: '修改成功'
          });
          this.dialog = false;
          this.$emit('search')
        }).catch(() => {
          this.$message.error({
            message: '修改失败'
          });
          this.dialog = false;
        });
      },
      cancel () {
        this.dialog = false;
      }
    },
  }
</script>

<style scoped>

</style>
