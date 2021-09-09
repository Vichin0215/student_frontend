<template>
  <el-card style="margin: 10px;padding: 15px 10px 10px 10px">
    <el-row style="margin-bottom: 10px">
      <el-button type="primary" size="small" @click="addMethod">新增</el-button>
      <!--<el-button type="danger" size="small" @click="deleteSelect">删除</el-button>-->
      <!--<el-input style="float:right;width:300px;" size="small" v-model="searchValue.code" placeholder="请输入课程名或专业" clearable @keyup.enter.native="filterData">-->
        <!--<el-button slot="append" @click="filterData" type="primary">过滤</el-button>-->
      <!--</el-input>-->
    </el-row>
    <VmBaseTable
      :setTableHigh="true"
      @on-select-change="select"
      ref="course_table"
      :data="dataTable"
      :columns="dataColumns"
      overflow
      @page-change="pageChange"
      showCheck
      showIndex
      :tableHigh="tableHigh"
    ></VmBaseTable>
    <VmCourse ref="course_model" @search="search"></VmCourse>
  </el-card>
</template>

<script>
  import VmCourse from './model/course-model'
  import VmBaseTable from '../../base/base-table'
  export default {
    name: "student",
    components: {
      VmBaseTable, VmCourse
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
            label: '课程号',
            prop: 'id',
            style: 'center',
            minWidth: '60'
          },
          {
            label: '课程名',
            prop: 'name',
            style: 'center',
            minWidth: '120'
          }, {
            label: '任课教师',
            prop: 'teacherId',
            style: 'center',
            minWidth: '60',
          }, {
            label: '操作',
            style: 'center',
            prop: 'id',
            minWidth: '120',
            render: (h, params) => {
              let btns = [];
              btns.push(this.getOpBtn(h, '编辑', 'primary', () => {
                console.log("bianji")
                this.editMethod(params.row)
              }));
              btns.push(this.getOpBtn(h, '删除', 'danger', () => {
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
          '/student_backend/courses',
          {params: that.searchValue}
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
        if (this.$refs['course_table'] !== undefined) {
          this.$refs['course_table'].currentPageToOne();
        }
        this.search()
      },
      addMethod() {
        // this.search();
        let type = 'add';
        let params = {
          type
        };
        this.$refs['course_model'].init(params);
      },
      editMethod(row) {
        let data = Object.assign({}, row);
        let type = 'edit';
        let params = {
          type,
          row: data
        };
        // 跳转到course_model.vue页面
        console.log(params)
        console.log(1222222)
        this.$refs['course_model'].init(params);
      },
      select(selection) {
        this.delBtn = selection.length <= 0;
      },
      deleteSingle (obj) {
        console.log(obj)
        let ids = [obj.id];
        this.deleteTable(ids);
      },
      deleteSelect () {
        let ids = this.table.getIds();
        if (ids.length > 0) {
          this.deleteTable(ids);
        }
      },
      deleteTable(ids) {
        let _this = this;
        this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.deleteMethod(ids, _this);
        }).catch(() => {
          this.$message.info({
            message: '已取消删除'
          });
        });
      },
      deleteMethod(params, _this) {
        _this.axiosHelper.delete(
          '/student_backend/courses?courseId=' + params[0]
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
      },
    },
    mounted () {
      this.table = this.$refs['course_table'];
      this.search();
    }
  }

</script>
