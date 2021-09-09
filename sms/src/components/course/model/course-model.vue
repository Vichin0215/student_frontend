<template>
  <el-dialog v-dialogDrag :title="title" :visible.sync="dialog" :close-on-click-modal=false append-to-body width="800px">
    <el-form ref="form" :rules="rules" :model="form" label-width="90px">
      <el-row>
        <el-col :span="12">
          <el-form-item label="课程号:" prop="courseId">
            <el-input v-model="form.courseId" maxlength="15" :disabled="type === 'edit'"></el-input>
          </el-form-item>
        </el-col>
        <el-col :span="12">
          <el-form-item label="课程名:" prop="courseName">
            <el-input v-model="form.courseName" clearable maxlength="30"></el-input>
          </el-form-item>
        </el-col>
      </el-row>
      <el-row>
        <el-col :span="12">
          <el-form-item label="任课教师:" prop="teacherId">
            <el-input v-model="form.teacherId" maxlength="15" clearable></el-input>
          </el-form-item>
        </el-col>
      </el-row>
    </el-form>
    <div slot="footer">
      <el-button type="primary" @click="click('form')" size="small">确定</el-button>
      <el-button @click="cancel" size="small">取消</el-button>
    </div>
  </el-dialog>
</template>

<script>
  export default {
    name: "course-model",
    data() {
      return {
        type: "",
        professionArr: [],
        doneNum: 0,
        title: '',
        termArr: [{
          value: 1,
          label: '上学期'
        }, {
          value: 2,
          label: '下学期'
        }],
        yearArr: [],
        formLabelWidth: '100px',
        dialog: false,
        typeArr: [{
          value: 1,
          label: '必修'
        }, {
          value: 2,
          label: '选修'
        }],
        form: {
          courseName: '',
          courseId: '',
          teacherId: '',
        },
        rules: {
          courseName: [
            { required: true, message: '课程名不能为空', trigger: 'blur' },
          ],
          courseId: [
            { required: true, message: '课程号不能为空', trigger: 'blur' },
          ],
          teacherId: [
            { required: true, message: '任课教师不能为空', trigger: 'blur' },
          ],
        }
      }
    },
    methods: {
      init (obj) {
        console.log(obj)
        this.clearForm();
        this.type = obj.type;
        if (obj.type === 'add') {
          this.title = '新增';
        } else if (obj.type === 'edit') {
          console.log(obj.row)
          console.log(12345)
          this.title = '修改';
          this.form.courseName = obj.row.name;
          this.form.courseId = obj.row.id;
          this.form.teacherId = obj.row.teacherId;
        }
        this.dialog = true;
      },
      clearForm () {
        let year = new Date().getFullYear();
        this.form = {
          courseName: '',
          courseId: '',
          teacherId: '',
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
              console.log(77777)
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
        this.axiosHelper.post(
          '/student_backend/courses', this.form).then(() => {
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
        console.log(12345555)
        this.axiosHelper.put(
          '/student_backend/courses', this.form).then(() => {
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
      },
       getProfession () {
         // this.axiosHelper.get('/api/sms/profession/getProfessionList').then(response => {
         //   let data = response.data;
         //   this.professionArr = data.map(item => {
         //     return {label: item.name,value: item.name}
         //   });
         // })
       }
    },
    created () {
      let today = new Date();
      let year = today.getFullYear();
      for (let i = 2010; i < year + 2; i++) {
        this.yearArr.push(i);
      }
       this.getProfession();
    }
  }
</script>
<style>
  html, body, #app, .login {
    height: 99%;
    padding: 0;
    margin: 0;
  }
</style>
<style scoped>
</style>
