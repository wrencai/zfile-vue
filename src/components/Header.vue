<template>
<div>
    <el-form :inline="true" class="zfile-header" size="mini">
        <el-form-item>
            <el-breadcrumb separator="/" separator-class="el-icon-arrow-right">
                <el-breadcrumb-item :to="{path: '/' + driveId + '/main'}">{{this.$store.state.common.config.siteName ? this.$store.state.common.config.siteName : '首页'}}</el-breadcrumb-item>
                <el-breadcrumb-item v-for="item in breadcrumbData" class="hidden-xs-only"
                                    :to="{path: '/' + driveId + '/main' + item.fullPath}"
                                    :key="item.path">{{item.name}}</el-breadcrumb-item>
            </el-breadcrumb>
        </el-form-item>
            <!-- 上传 -->
            <!-- <UploadFile :drive-id="driveId"></UploadFile> -->
        <div class="zfile-header-drive box animate__animated animate__fadeIn">

          <el-form-item v-show="this.$store.getters.debugMode" label="已开启 DEBUG 模式，使用完请及时关闭" class="zfile-debug-tips" size="small">
            <el-button @click="resetAdminPwd" size="small" type="danger">重置密码</el-button>
          </el-form-item>


            <el-form-item label="图片模式" size="small">
                <el-switch v-model="imgModel"></el-switch>
            </el-form-item>

            <el-select size="small" v-model="currentDriveId" placeholder="请选择驱动器">
                <el-option v-for="item in driveList"
                           :key="item.id"
                           :label="item.name"
                           :value="item.id">
                </el-option>
            </el-select>
            

        </div>
        
    </el-form>
</div>
</template>

<script>
    import path from 'path'
    
    import UploadFile from './UploadFile.vue'

    export default {
        name: "Header",
        props: ['driveId'],
        data() {
            return {activeIndex: '1',
        activeIndex2: '1',
                imgModel: false,
                driveList: [],
                currentDriveId: "",
                search: '',
                breadcrumbData: [],
                searching: false
            }
        },
        components: {
            UploadFile
        },
        created() {
            this.buildBreadcrumbData();
        },
        methods: {
           handleSelect(key, keyPath) {
        console.log(key, keyPath);
      },
            resetAdminPwd() {
                this.$confirm('是否确认重置后台管理员密码？重置后用户名/密码将强制修改为 admin 123456', '提示', {
                    confirmButtonText: '确定',
                    cancelButtonText: '取消',
                    type: 'warning',
                    callback: action => {
                        if (action === 'confirm') {
                            this.$http.get('/debug/resetPwd').then((response) => {
                                if (response.data.code === 0) {
                                    this.$message.success("重置成功，请及时关闭 debug 功能，防止出现安全问题！");
                                } else {
                                    this.$message.error(response.data.msg)
                                }
                            });
                        }
                    }
                });
            },
            buildBreadcrumbData() {
                this.breadcrumbData = [];
                let fullPath = this.$route.params.pathMatch;
                fullPath = fullPath ? fullPath : '/';

                while (fullPath !== '/') {
                    let name = path.basename(fullPath);
                    this.breadcrumbData.unshift({name, fullPath});
                    fullPath = path.resolve(fullPath, "../");
                }
            },
	        refreshCurrentStorageStrategy() {
		        this.driveList.some((item) => {
			        if (item.id === this.currentDriveId) {
				        this.$store.commit('updateCurrentStorageStrategy', item);
			        }
		        });
	        }
        },
        watch: {
            'currentDriveId': function (newVal, oldVal) {
                this.$store.commit('updateOldDriveId', oldVal);
                this.refreshCurrentStorageStrategy();
                if (oldVal !== "") {
                    this.$router.push('/' + newVal + '/main');
                }
            },
            '$store.getters.newImgMode': function (newVal) {
                this.imgModel = newVal;
            },
            'imgModel': function () {
                this.$store.commit('switchImgMode', this.imgModel);
            },
            '$route.fullPath': function () {
                // 当 URL 变化, 则自动重新 build 面包屑
                this.buildBreadcrumbData();
            },
            'search': function (newVal) {
                clearTimeout(this.timer);
                this.timer = setTimeout(() => {
                    this.$store.commit('updateSearchParam', newVal);
                }, 150);
            }
        },
        async mounted() {
            await this.$http.get('/api/drive/list').then((response) => {
            	if (!response.data.data.isInstall) {
		            this.$router.push('/install');
            		return;
	            }

                this.driveList = response.data.data.driveList;
                // 如果当前 URL 参数中有驱动器 ID, 则直接用当前的.
                if (this.driveId) {
                    this.currentDriveId = Number(this.driveId);
                } else if (this.driveList.length > 0) {
                    // 否则读取驱动器列表中的第一个, 并跳转到响应的 URL 中.
                    this.currentDriveId = this.driveList[0].id;
                    this.$router.push('/' + this.driveList[0].id + '/main');
                } else if (this.driveList.length === 0) {
	                this.$confirm('当前无可用驱动器，是否跳转至管理员页面添加驱动器？', '提示', {
		                confirmButtonText: '确定',
		                cancelButtonText: '取消',
		                type: 'info',
		                callback: action => {
			                if (action === 'confirm') {
				                this.$router.push('/login');
			                }
		                }
	                });
                }

	            this.refreshCurrentStorageStrategy();
            });
        }
    }
</script>

<style scoped>

  .el-dropdown {
    vertical-align: top;
  }
  .el-dropdown + .el-dropdown {
    margin-left: 15px;
  }
  .el-icon-arrow-down {
    font-size: 12px;
  }


    .zfile-header {
        display: flex;
        flex-flow: row nowrap;
        justify-content: space-between;
        height: 48px;
        line-height: 48px !important;
        background: #fafafa;
        border-bottom: 1px solid rgba(0,0,0,0.05);
        padding-left: 30px;
    }

    .zfile-header .el-breadcrumb {
        line-height: 48px;
        font-size: 13px;
    }

    .zfile-header .el-input {
        line-height: 48px;
    }


    @media only screen and (max-width:767px) {
        .zfile-header >>> .el-breadcrumb__separator {
            display: none !important;
        }
        .zfile-header >>> .el-form-item__label {
            display: none !important;
        }
        .zfile-header >>> .el-select {
            width: 120px;
        }
    }

    .zfile-header-drive {
        margin-right: 20px;
    }

    .zfile-header-drive >>> .el-form-item__content {
        vertical-align: unset;
    }

    .zfile-debug-tips >>> .el-form-item__label {
      font-weight: bold;
      color: red !important;
    }

</style>
