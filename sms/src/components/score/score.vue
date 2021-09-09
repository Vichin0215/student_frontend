<template>
  <div>
    <el-card class="cardStyle" v-loading="loading">
      <el-button v-if="userInfo.level !== 2" @click="change" size="small" style="margin-bottom: 15px">筛选</el-button>
      <el-button @click="addEntry" type="primary" size="small" :disabled="dataTable.length <= 0"
                 style="margin-bottom: 15px" v-if="userInfo.level !== 2">成绩录入
      </el-button>
      <!--过滤框-->
      <el-collapse-transition v-if="userInfo.level !== 2">
        <div v-if="show" style="background-color: white;height: 100px;;box-sizing: border-box">
          <el-form ref="form" :model="form" label-width="80px">
            <el-row>
              <el-col :span="8">
                <el-form-item label="专业：" prop="profession">
                  <el-select v-model="form.professionObj" style="width: 90%"
                             value-key="profession">
                    <el-option v-for="(item,index) in classArr" :key="index" :label="item"
                               :value="item"></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="班级：" prop="grade">
                  <el-select v-model="form.grade" style="width: 90%">
                    <el-option v-for="(item,index) in gradeArr" :key="index" :label="item" :value="item"></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
              <el-col :span="8">
                <el-form-item label="课程名：" prop="profession">
                  <el-select v-model="form.courseName" style="width: 90%">
                    <el-option v-for="(item,index) in courseArr" :key="index" :label="item.name" :value="item.id"></el-option>
                  </el-select>
                </el-form-item>
              </el-col>
            </el-row>
          </el-form>
          <div style="margin: -5px 0px 10px 77%">
            <el-button type="primary" @click="clickAndClose" size="small" style="margin-right: 10px">确定</el-button>
            <el-button size="small" @click="cancel">取消</el-button>
          </div>
        </div>
      </el-collapse-transition>
      <VmBaseTable
        :setTableHigh="true"
        ref="score_table"
        :data="dataTable"
        :columns="dataColumns"
        @page-change="pageChange"
        @on-select-change="select"
        :showCheck="userInfo.level !== 2"
        :tableHigh="tableHigh"
      ></VmBaseTable>
    </el-card>
  </div>
</template>

<script>
  import VmBaseTable from '../../base/base-table'

  export default {
    name: "score",
    components: {
      VmBaseTable
    },
    data() {
      return {
        loading: false,
        courseArr: [],
        classArr: [],
        professionArr: [],
        gradeArr: [],
        batch: false,
        showInput: '',
        userInfo: '',
        show: false,
        form: {
          professionObj: '',
          grade: '',
          term: '',
          courseName: '',
          year: '',
        },
        yearArr: [],
        termArr: [{
          label: '上',
          value: 1
        }, {
          label: '下',
          value: 2
        }],
        table: null,
        searchValue: {
          $limit: 10,
          $offset: 0,
        },
        selectValue: [],
        selection: [],
        dataTable: [],
        dataColumns: [
          {
            label: '学号',
            prop: 'studentId',
            style: 'center',
            minWidth: '70',
          }, {
            label: '姓名',
            prop: 'studentName',
            style: 'center',
            minWidth: '70',
          }, {
            label: '课程名',
            prop: 'courseName',
            style: 'center',
            minWidth: '70'
          }, {
            label: '专业',
            prop: 'major',
            style: 'center',
            minWidth: '70'
          }, {
            label: '班级',
            prop: 'class1',
            style: 'center',
            minWidth: '80',
          }, {
            label: '成绩',
            prop: 'grade1',
            style: 'center',
            minWidth: '60',
            render: (h, params) => {
              if (this.batch || this.showInput === params.row.studentId) {
                return h('el-input', {
                  props: {
                    value: params.row.grade1,
                    size: 'small',
                  },
                  on: {
                    input(value) {
                      value = value.replace(/[^\d]/g, '') && value.slice(0, 5);
                      params.row.grade1 = value > params.row.score ? params.row.score : value;
                    },
                  }
                })
              } else {
                let colorValue = params.row.grade1 > 59 ? '' : 'red';
                let scoreValue = params.row.grade1 === null ? '未录入' : params.row.grade1;
                return h('div', {style: {color: colorValue}}, scoreValue)
              }
            }
          }, {
            label: '操作',
            style: 'center',
            prop: 'id',
            minWidth: '120',
            render: (h, params) => {
              let btns = [];
              if (this.showInput === params.row.studentId) {
                btns.push(this.getOpBtn(h, '取消', '', () => {
                  this.editMethod(params.row);
                }));
              } else {
                btns.push(this.getOpBtn(h, '编辑', 'primary', () => {
                  this.editMethod(params.row);
                }));
              }
              return h('div', btns)
            }
          }
        ],
      }
    },
    methods: {
      cancel() {
        this.form = {
          profession: '',
          grade: '',
          term: '',
          courseName: ''
        };
        this.show = !this.show;
      },
      change() {
        this.show = !this.show;
        this.axiosHelper.get(
          '/student_backend/classes',
          {}).then(response => {
          console.log(response.data);
          this.gradeArr = response.data
        }).catch(error => {
          this.$message.error({
            message: '获取成绩失败'
          }, error)
        });

        this.axiosHelper.get(
          '/student_backend/courses',
          {}).then(response => {
          console.log(response.data);
          this.courseArr = response.data
        }).catch(error => {
          this.$message.error({
            message: '获取成绩失败'
          }, error)
        });

        this.axiosHelper.get(
          '/student_backend/majors',
          {}).then(response => {
          console.log(response.data);
          this.classArr = response.data;
        }).catch(error => {
          this.$message.error({
            message: '获取专业失败'
          }, error)
        });
      },
      clickSure(data) {
        this.searchValue.$offset = 0;
        // 跳转到第一页
        if (this.$refs['score_table'] !== undefined) {
          this.$refs['score_table'].currentPageToOne();
        }
        this.click(data);
      },
      clickAndClose() {
        this.clickSure(this.searchValue);
        this.show = !this.show
      },
      clickMethod(obj) {
        if (this.userInfo.level === 2) {
          // 调获取学生成绩接口
          this.axiosHelper.get(
            '/student_backend/search',
            {}).then(response => {
            console.log(response.data)
            this.dataTable = response.data;
            this.table.total = response.data.length;
          }).catch(error => {
            this.$message.error({
              message: '获取成绩失败'
            }, error)
          })
        } else if (this.userInfo.level === 1) {
          // 根据 params 参数获取学生成绩接口
          this.axiosHelper.get(
            '/student_backend/search_update',
            {params: obj}).then(response => {
            console.log(response.data);
            this.dataTable = response.data;
            this.table.total = response.data.length;
          }).catch(error => {
            this.$message.error({
              message: '获取成绩失败'
            }, error)
          })
        }
      },
      click(page) {
        let userInfo = JSON.parse(localStorage.userInfo);
        let kind = 1;
        if (this.form.grade !== '') {
          kind = 3;
        } else if (this.form.professionObj !== '') {
          kind = 2;
        }
        let obj = {
          coursetId: this.form.courseName !== '' ? this.form.courseName:101,
          kind: kind,
          major: this.form.professionObj,
          Class1: this.form.grade
        };
        this.clickMethod(obj);
      },
      pageChange(page) {
        this.searchValue.$limit = page.limit;
        this.searchValue.$offset = page.offset;
        this.click(this.searchValue)
      },
      editMethod(data) {
        this.showInput = this.showInput === data.studentId ? '' : data.studentId;
        this.batch = false;
      },
      batchMethod() {
        this.batch = true;
      },
      select(selection) {
        this.selection = selection;
      },
      addEntry() {
        let flag = this.selection.every(data => {
          return data.scoreByUser !== null && data.scoreByUser !== ''
        });
        if (this.selection.length > 0) {
          if (flag) {
            this.loading = true;
            this.axiosHelper.post('/student_backend/search_update', this.selection).then(() => {
              this.$message.success({
                message: '录入成绩成功'
              });
              this.loading = false;
              this.click(this.searchValue);
              this.batch = false;
              this.showInput = false;
            }).catch(() => {
              this.$message.warning({
                message: '录入成绩失败'
              });
              this.loading = false;
            })
          } else {
            this.$message.warning({
              message: '存在未录入成绩人员'
            })
          }
        } else {
          this.$message.warning({
            message: '请先选择人员'
          })
        }
      },
      formatJson(filterVal, jsonData) {
        return jsonData.map(v => filterVal.map(j => v[j]))
      },
    },
    mounted() {
      this.table = this.$refs['score_table'];
      if (this.userInfo.level !== 0) {
        if (this.userInfo.level === 2) {
          this.dataColumns = this.dataColumns.filter(data => {
            return (data.label !== '操作' && data.label !== '学号')
          });
        }
      }
      this.click(this.searchValue);
    },
    created() {
      this.userInfo = JSON.parse(localStorage.userInfo);
    },
    computed: {
      tableHigh() {
        if (this.userInfo.level === 2) {
          return '72vh'
        } else {
          return this.show ? '52vh' : '66vh'
        }
      }
    }
  }
</script>

<style scoped>
  .cardStyle {
    height: 78vh;
    margin: 10px;
    padding: 15px 10px 10px 10px;
  }
</style>
