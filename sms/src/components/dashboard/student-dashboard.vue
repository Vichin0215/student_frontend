<template>
  <div>
    <el-card class="content" style="width: 60%;margin: 12px">
      <div class="title">
          <i class="el-icon-user-solid"></i> 个人信息
      </div>
        <el-row style="color: #666666;padding: 20px">
          <el-col :span="6" style="margin-left: 10px;margin-top: 10px" class="upload" v-loading="loading">
            <el-upload
              action=""
              :http-request="submitUpload"
              :on-change="onchange"
              :show-file-list="false"
              accept=".png,.jpg"
            >
              <div class="header">
                <el-image :src="imageUrl" class="header imgStyle">
                  <div slot="error" class="image-slot">
                    <div style="width: 120px; height: 140px">
                      <i class="el-icon-user-solid"></i>
                    </div>
                  </div>
                </el-image>
              </div>
              <div class="camera" @click="submitUpload"><i class="el-icon-camera"></i></div>
            </el-upload>
          </el-col>
          <el-col :span="16" style="margin-left: 50px">
            <h2 class="distance" style="margin-bottom: 35px;margin-left: 15px">{{userInfo.studentName}}</h2>
            <div  class="distance">
              <i class="el-icon-user" title="学号"> {{userInfo.studentId}}</i>{{'&#12288'}}
              <i class="el-icon-male" v-if="userInfo.studentSex === '男'" title="性别"> {{userInfo.studentSex}}</i>
              <i class="el-icon-female" v-else title="性别"> {{userInfo.studentSex}}</i>{{'&#12288'}}
              <i class="el-icon-reading" title="专业"> {{userInfo.major}}</i>{{'&#12288'}}
              <i class="el-icon-office-building" title="班级"> {{userInfo.class1}}</i>{{'&#12288'}}
            </div>
          </el-col>
        </el-row>
    </el-card>

    <editInfo ref="editInfo_model" @refresh="refresh"></editInfo>
    <VmUpload ref="upload_model" @propUrl="propUrl"></VmUpload>
  </div>
</template>

<script>
import studentSetting from './model/student-setting-model'
import VmUpload from './model/upload-model'
import editInfo from './model/student-edit-model'
export default {
  inject:['reload'],
  name: "dashboard",
    data () {
      this.pieSetting = {
        offsetY: 150,
        radius: 90,
        labelLine: {
          length: 8,
          smooth: false
        },
      };
      return {
        imageUrl: '',
        loading: false,
        showChart: false,
        showPie: false,
        lineHeight: '330px',
        lineWidth: '100%',
        changeChart: true,
        userInfo: {},
        sexName: '男',
        schoolInfo: '',
        myselfInfo: '',
        pieData: {
          columns: ['label', 'value'],
          rows: [],
        },
        histogramData: {
          columns: ['courseName', '分数'],
          rows: []
        },
        lineData: {
          columns: ['courseName', '分数'],
          rows: []
        },
      }
    },
    methods: {
      editInfo () {
        let userInfo = Object.assign({}, this.userInfo);
        this.$refs.editInfo_model.init(userInfo);
      },
      onchange () {
        const event = event || window.event;
        const data = event.target.files[0];
        const reader = new FileReader();
        //转base64
        reader.onload = e => {
          const imageUrl = e.target.result; //将图片路径赋值给src
          this.$refs.upload_model.init(imageUrl, this.form)
        };
        reader.readAsDataURL(data);
      },
      propUrl (url) {
        this.imageUrl = url;
        this.loading = true;
        setTimeout(() => {
          this.loading = false;
        }, 500);
      },
      submitUpload (params) {
      },
      refresh () {
        // 刷新页面
        this.reload();
      },
      setting () {
        this.$refs.setting_model.init();
      },
      changeMethod () {
        this.changeChart = !this.changeChart;
      },
    },
    computed: {
      collapse() {
        return this.$store.state.collapse;
      }
    },
    watch: {
      collapse() {
        if (this.$refs['chart']) {
          setTimeout(() => {
            this.$refs['chart'].resize()
          }, 150)
        }
      }
    },
    mounted() {
      this.userInfo = JSON.parse(localStorage.userInfo);
    },
    components: {
      editInfo, studentSetting, VmUpload
    }
}
</script>

<style scoped>
  .header {
    background-color: #ccc;
    font-size: 80px;
    color: white;
    width: 120px;
    border-radius: 5px;
    height: 140px;
    line-height: 140px;
    text-align: center;
    transition: .2s;
  }
  .upload:hover .imgStyle{
    opacity: 0.2;
  }
  /*“修改我的头像” 样式*/
  .upload:hover .header::after {
    transition: .2s;
    content: "修改我的头像";
    font-size: 13px;
    position: absolute;
    right: 20px;
    top: 30px;
    color: #333;
  }
  /*父组件大小*/
  .upload {
    position: relative;
    top: 0;
    left: 0;
    width: 120px;
  }
  /*照相icon样式*/
  .camera {
    color: #555;
    font-size: 35px;
    position: absolute;
    top: 35px;
    right: 42px;
    opacity: 0;
    cursor: pointer;
  }
  /*显示照相icon*/
  .upload:hover .camera {
    transition: .2s;
    opacity: 1;
  }

  .distance {
    margin: 15px 0px;
  }
  .title {
    font-size: 18px;
    background-color: #f4fcfe;
    border-bottom: 1px solid #dcdcdc;
    height:50px;
    line-height: 50px;
    padding: 0 20px 0 20px;
    color: #0089AB;
  }
  .content {
    height: 300px;
    float: left;
  }
  .content:hover {
    background-color: #F5FAF5;
    box-shadow: 5px 8px 6px 0px rgba(70,88,131,0.2);
  }
  .edit:hover {
    transform:  scale(1.1)
  }
  .settingStyle {
    color: gray;
    margin-left: 450px;
  }
  .settingStyle:hover {
    transform: scale(1.2);
    color: #409EFF;
  }
  .changeChartStyle {
    margin: -4px 0 0 97.5%;
    color: gray;
    font-size: 16px;
  }
  .changeChartStyle:hover {
    transform: scale(1.2);
    color: #409EFF;
  }
</style>
<style>
  #app , body {
    padding: 0;
    margin: 0;
  }
</style>
