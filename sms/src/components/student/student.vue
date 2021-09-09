<template>
  <el-card style="margin: 10px;padding: 15px 10px 10px 10px">
    <el-row style="margin-bottom: 10px">
      <el-button type="primary" size="small" @click="addMethod">新增</el-button>
      <!--<el-button type="danger" size="small" @click="deleteSelect">删除</el-button>-->
      <!--<el-input style="float:right;width:300px;" size="small" v-model="searchValue.code" placeholder="请输入学号或姓名或专业" clearable @keyup.enter.native="filterData">-->
        <!--<el-button slot="append" @click="filterData" type="primary">过滤</el-button>-->
      <!--</el-input>-->
    </el-row>
    <VmBaseTable
      :setTableHigh="true"
      @on-select-change="select"
      ref="student_table"
      :data="dataTable"
      :columns="dataColumns"
      @page-change="pageChange"
      showCheck
      :tableHigh="tableHigh"
    ></VmBaseTable>
    <VmStudent ref="student_model" @search="search"></VmStudent>
  </el-card>
</template>

<script>
  import VmStudent from './model/student-model'
  import VmBaseTable from '../../base/base-table'
  export default {
    name: "user",
    components: {
      VmBaseTable, VmStudent
    },
    data () {
      return {
        right: true,
        table: null,
        searchValue: {
          $limit: 10,
          $offset: 0,
          code: ''
        },
        tableHigh: '66vh',
        selectValue: [],
        dataTable: [],
        dataColumns: [
          {
            label: '学号',
            prop: 'studentId',
            style: 'center',
            minWidth: '90',
          } , {
            label: '姓名',
            prop: 'studentName',
            style: 'center',
            minWidth: '70',
          } , {
            label: '性别',
            prop: 'studentSex',
            style: 'center',
            minWidth: '50',
            render: (h, params) => {
              if (params.row.studentSex === '男') {
                return h('div', {}, '男')
              } else if (params.row.studentSex === '女') {
                return h('div', {}, '女')
              }
            }
          }, {
            label: '专业',
            prop: 'major',
            style: 'center',
            minWidth: '100',
          } , {
            label: '班级',
            prop: 'class1',
            style: 'center',
            minWidth: '60',
          }, {
            label: '密码',
            prop: 'password',
            style: 'center',
            minWidth: '60',
          }, {
            label: '操作',
            style: 'center',
            minWidth: '120',
            render: (h, params) => {
              let btns = [];
              // btns.push(this.getOpBtn(h, '编辑', 'primary', () => {
              //   this.editMethod(params.row)
              // }));
              btns.push(this.getOpBtn(h, '删除', 'danger', () => {
                // params.row就是要删除的这一行数据
                this.deleteSingle(params.row)
              }));
              return h('div', btns)
            }
          }
        ],
        delBtn: true
      }
    },
    methods: {
      search() {
        let that = this;
        this.axiosHelper.get(
          '/student_backend/students',
          {}
        ).then(response => {
          this.dataTable = response.data;
          this.table.total = response.data.length
        }).catch(error => {
          this.$message.error({
            message: '失败'
          }, error)
        })
      },
      pageChange(page) {
        this.searchValue.$limit = page.limit;
        this.searchValue.$offset = page.offset;
        this.search();
      },
      filterData() {
        this.searchValue.$offset = 0;
        // 跳转到第一页
        if (this.$refs['student_table'] !== undefined) {
          this.$refs['student_table'].currentPageToOne();
        }
        this.search()
      },
      addMethod() {
        this.search();
        let type = 'add';
        let params = {
          type
        };
        this.$refs['student_model'].init(params);
      },
      editMethod(row) {
        let data = Object.assign({}, row);
        let type = 'edit';
        let params = {
          type,
          row: data
        };
        this.$refs['student_model'].init(params);
      },
      select(selection) {
        this.delBtn = selection.length <= 0;
      },
      deleteSingle (obj) {
        this.deleteTable(obj);
      },
      deleteSelect () {
        let ids = this.table.getIds();
        if (ids.length > 0) {
          this.deleteTable(ids);
        }
      },
      deleteTable(row) {
        let _this = this;
        this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.deleteMethod(row, _this);
        }).catch(() => {
          this.$message.info({
            message: '已取消删除'
          });
        });
      },
      deleteMethod(params, _this) {
        let obj = params;
        obj.kind = 0;
        console.log(obj);
        _this.axiosHelper.get(
          '/student_backend/delete', {params: obj}
        ).then(response => {
          let status = response.status;
          if (status === 200) {
            // 若删除当前页的最后一条时，加载上一页信息
            if ((_this.table.total - params.length) % 10 === 0 && _this.searchValue.$offset !== 0) {
              _this.searchValue.$offset -= 10
            }
            _this.$message.success({
              message: '删除成功'
            });
            _this.search()
          }
        }).catch(error => {
          _this.$message.error({
            message: '删除失败'
          }, error)
        })
      }
    },
    mounted () {
      this.table = this.$refs['student_table'];
      this.search();
    }
  }

</script>
