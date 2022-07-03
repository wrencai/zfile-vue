<template>
  <div class='UploadFile' style="display: inline">
    <el-button @click="initUploader" type="primary" size="medium" plain style="margin-left:10px;margin-right: 10px;">
      文件上传
    </el-button>
    <el-button @click="initMkdirDialog" type="primary" size="medium" plain style="margin-right: 10px;">
      新建文件夹
    </el-button>

    <!-- 文件上传面板 -->
    <el-drawer
      title="文件上传"
      size="90%"
      :show-close="true"
      :visible.sync="uploadParam.drawer"
      :with-header="false"
      :append-to-body="true">    
        <!-- 错误提示 -->        
          <el-alert
            v-show="uploadParam.showErrorMsg"
            :title="uploadParam.errorMsg"
            :type="uploadParam.msgType">
          </el-alert>
        <!-- 上传控件 -->
        <div class="zfile-upload-drawer">
          <el-upload
            class="upload-demo"
            ref="upload"
            :multiple="uploadParam.allowMultiFile"
            :limit="uploadParam.successFileCountLimit"
            :action="uploadUrl"
            :on-error="handleUploadError"
            :on-exceed="handleExceedsuccessFileCount"
            :on-change="handleFileChange"
            :on-success="handleUploadSuccess"
            :file-list="uploadParam.fileList"
            :auto-upload="false">
            <el-button slot="trigger" size="small" type="primary">选择文件</el-button>
            <el-button style="margin-left: 10px;" size="small" type="success" @click="submitUpload">上传</el-button>
            <div slot="tip" class="el-upload__tip">一次最多上传{{uploadParam.successFileCountLimit}}个文件</div>
          </el-upload>
        </div>
    </el-drawer>

    <!-- 新建文件夹面板 -->
    <el-dialog title="新建文件夹" :visible.sync="mkdirParam.dialogFormVisible"
        :append-to-body="true"
        width="90%">
      <el-form :model="mkdirParam">
        <el-form-item label="文件夹名称" :label-width="mkdirParam.formLabelWidth">
          <el-input v-model="mkdirParam.dirname" autocomplete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="mkdirParam.dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitMkdir">确 定</el-button>
      </div>
    </el-dialog>
    
  </div>
</template>

<script>
export default {
  name: 'UploadFile',
  data() {
      return {
        // 查询条件
        searchParam: {
            path: '',
            password: ''
        },
        uploadParam: {
          drawer: false,
          showErrorMsg: false,
          errorMsg: "",
          successFileCountLimit: 60,
          allowMultiFile: true,
          fileList: [],
          successFileCount: 0,
          // 提示框类型：error, success
          msgType: "error"
        },
        mkdirParam: {
          dialogFormVisible: false,
          dirname: ""
        }        
      };
  },
  props: {
    // 上传的存储驱动id
    driveId: {
      type: String,
      default: "0"
    }
  },
  components: {},
  created () {},
  mounted () {
    console.log("upload driveId:"+ this.driveId)
  },
  computed: {
      uploadUrl() {        
        let p = this.$route.params.pathMatch;
        p = p ? p : '/';
        // return 'http://172.16.10.64:8081/api/upload/'+1+"?path="+ p
        return '/api/upload/'+this.driveId+"?path="+ p
      }
  },
  methods: { 
    // 初始化上传面板信息
    initUploader() {      
      this.uploadParam.showErrorMsg = false;
      this.uploadParam.msgType="error";
      this.uploadParam.errorMsg = "";
      this.uploadParam.drawer=true;
      this.uploadParam.successFileCount=0;
    },
    // 初始化新建文件夹对话框
    initMkdirDialog() {
      this.mkdirParam.dirname="";
      this.mkdirParam.dialogFormVisible = true
    },
    // 上传文件
    submitUpload() {
      if( this.$refs.upload.uploadFiles.length <= 0) {      
        this.uploadParam.msgType="error";  
        this.uploadParam.errorMsg = "请先选择要上传的文件";
        this.uploadParam.showErrorMsg = true;
        return;
      }
      this.$refs.upload.submit();
    },
    handleUploadError(err, file, fileList) {
      console.log("错误了"+err);
      this.uploadParam.msgType="error";
      this.uploadParam.errorMsg = "上传出错：";
      this.uploadParam.showErrorMsg = true;
    },
    handleExceedsuccessFileCount(files, fileList){
      this.uploadParam.msgType="error";
      this.uploadParam.errorMsg = "一次最多上传"+this.uploadParam.successFileCountLimit+"个文件";
      this.uploadParam.showErrorMsg = true;
    },
    handleFileChange(file, fileList) {
      this.uploadParam.showErrorMsg = false;
      this.uploadParam.errorMsg = "";
    },
    handleUploadSuccess(response, file, fileList) {
      this.uploadParam.successFileCount = this.uploadParam.successFileCount + 1;
      if(this.uploadParam.successFileCount === fileList.length) {
        // this.uploadParam.msgType="error";
        this.$message("上传成功"+this.uploadParam.successFileCount+"个文件")
        // 上传完成后刷新页面
        setTimeout("location.reload()",1500); 
        
      }
    },    
    // 新建文件夹
    submitMkdir() {
      let param = {
          path: this.getPath(),
          name: this.mkdirParam.dirname,
          password: this.getPathPwd(),
      };
      let url = '/api/mkdir/' + this.driveId+"?path="+param.path+"&name="+param.name+"&password="+param.password;
      // let url = 'http://172.16.10.64:8081/api/mkdir/' + this.driveId+"?path="+param.path+"&name="+param.name+"&password="+param.password;
      let requestDriveId = this.driveId;
      this.$http.post(url).then((response) => {
        let currentDriveId = this.driveId;
        if (requestDriveId !== currentDriveId) {
          return;
        }
        // 如果需要密码或密码错误进行提示, 并弹出输入密码的框.
        if (response.data.code !== this.common.responseCode.SUCCESS) {
            this.$message.error('新建文件夹出错：'+response.data.msg);
            return;
        } else {
            this.$message.info('新建文件夹成功');
            this.mkdirParam.dialogFormVisible = false;
            this.mkdirParam.dirname = "";
            // 上传完成后刷新页面
            location.reload();
        }

      });
      
    },
    // 工具方法
    getPath() {
        let p = this.$route.params.pathMatch;
        this.searchParam.path = p ? p : '/';
        return this.searchParam.path;
    },    
    getPathPwd() {
        let pwd = sessionStorage.getItem("zfile-pwd-" + this.searchParam.path);
        return pwd === null ? '' : encodeURI(pwd);
    },
  }
}
</script>

<style scoped>
.zfile-upload-drawer {
       margin-left: 20px;
       margin-top: 20px;
    }
</style>
