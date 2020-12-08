<template>
  <el-container>
    <el-header>
      <div>
        <img alt src="../assets/logo.png"/>
        <span>通用物联平台</span>
      </div>
      <el-button type="info" @click="logout">注销</el-button>
    </el-header>
    <el-main>
      <el-card>
        <div slot="header" style="display: flex; justify-content: space-between; align-items: center">
          <span style="font-size: x-large">应用管理</span>
          <div>
            <el-button type="primary" @click="addAppVisible = true">新建应用</el-button>
            <el-button type="primary" @click="bindAppVisible = true">绑定应用</el-button>
          </div>
        </div>
        <el-row :gutter="10">
         <el-col :span="6" :offset="6">
           <el-table :data="appListData" style="width: 100%">
             <el-table-column type="index">
             </el-table-column>
             <el-table-column label="应用列表" prop="appName" width="200px">
             </el-table-column>
             <el-table-column label="应用Id" prop="appId" width="200px">
             </el-table-column>
             <el-table-column label="操作">
               <template slot-scope="scope">
                 <el-button icon="el-icon-star-on" type="primary" plain>进入应用</el-button>
                 <el-button icon="el-icon-s-tools" type="primary" plain @click="getAppConfig(scope.row.appId)">应用设置</el-button>
                 <el-button icon="el-icon-set-up" type="primary" plain>设备定义</el-button>
                 <el-button icon="el-icon-upload" type="primary" plain>页面上传</el-button>
                 <el-button icon="el-icon-copy-document" type="primary" plain>设备列表</el-button>
                 <el-button icon="el-icon-s-release" type="danger" @click="unBindApp(scope.row.appId)">解除绑定</el-button>
               </template>
             </el-table-column>
           </el-table>
         </el-col>
        </el-row>
      </el-card>
      <!--添加应用对话框-->
      <el-dialog :visible.sync="addAppVisible" title="新建应用" width="50%">
        <el-form :model="addApp" :rules="addAppRules" ref="addAppRef" label-width="70px">
          <el-form-item label="应用名" prop="appName">
            <el-input v-model="addApp.appName"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addAppVisible = false; addApp.appName = ''">取 消</el-button>
          <el-button type="primary" @click="addAppMethod">确 定</el-button>
        </span>
      </el-dialog>
      <!--应用设置对话框-->
      <el-dialog :visible.sync="appConfigVisible" title="应用设置" width="40%">
        <el-button type="primary" @click="addAppGradeVisible = true">添加权限级别</el-button>
        <el-table :data="appGrade" style="width: 100%">
          <el-table-column label="权限等级" prop="grade">
          </el-table-column>
          <el-table-column label="注册密码" prop="gradePassword">
          </el-table-column>
          <el-table-column label="操作">
            <template scope="scope1">
              <el-tooltip effect="dark" content="删除对应级别" placement="top">
                <el-button icon="el-icon-delete" type="danger" size="mini" @click="deletePasswordMethod(scope1.row.grade)"></el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
      </el-dialog>
      <!--添加应用权限对话框，输入级别和注册密码-->
      <el-dialog :visible.sync="addAppGradeVisible" title="增加权限级别" width="30%">
        <el-form :model="addGrade" :rules="addGradeRules" ref="addGradeRef">
          <el-form-item label="级别" prop="grade" placeholder="请输入1-10中的一个数字">
            <el-input v-model="addGrade.grade"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="gradePassword" placeholder="请输入密码">
            <el-input v-model="addGrade.gradePassword" type="password"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addAppGradeVisible = false; addGrade.grade = 0; addGrade.gradePassword = ''">取 消</el-button>
          <el-button type="primary" @click="addAppGradeMethod">确 定</el-button>
        </span>
      </el-dialog>
      <!--绑定应用对话框-->
      <el-dialog :visible.sync="bindAppVisible" title="绑定应用" width="50%">
        <el-form :model="bindApp" :rules="bindAppRules" ref="bindAppRef" label-width="90px">
          <el-form-item label="应用Id" prop="appId">
            <el-input v-model="bindApp.appId"></el-input>
          </el-form-item>
          <el-form-item label="应用密码" prop="appPassword" placeholder="请输入密码">
            <el-input v-model="bindApp.appPassword" type="password"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="bindAppVisible = false; bindApp.appId = null; bindApp.appPassword = null">取 消</el-button>
          <el-button type="primary" @click="bindAppMethod">确 定</el-button>
        </span>
      </el-dialog>
    </el-main>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      appListData: null,
      addAppVisible: false,
      addApp: {
        appName: ''
      },
      addAppRules: {
        appName: [
          { required: true, message: '请输入应用名', trigger: 'blur' },
          { min: 3, max: 10, message: '应用名长度在3-10个字符之间', trigger: 'blur' }
        ]
      },
      appConfigVisible: false,
      addAppGradeVisible: false,
      appConfig: null,
      appGrade: null,
      addGrade: {
        grade: null,
        gradePassword: null
      },
      addGradeRules: {
        grade: [
          { required: true, message: '请输入对应级别', trigger: 'blur' },
          { pattern: /^([1-9]|10)$/, message: '级别应为1-10的数字', trigger: 'blur' }
        ],
        gradePassword: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 14, message: '密码长度在6-14个字符之间', trigger: 'blur' }
        ]
      },
      bindAppVisible: false,
      bindApp: {
        appId: null,
        appPassword: null
      },
      bindAppRules: {
        appId: [
          { required: true, message: '请输入应用Id', trigger: 'blur' }
        ],
        appPassword: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 14, message: '密码长度在6-14个字符之间', trigger: 'blur' }
        ]
      }
    }
  },
  created () {
    this.getAppList()
  },
  methods: {
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    addAppMethod () {
      this.$refs.addAppRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('/app?appName=' + this.addApp.appName)
        console.log(res)
        if (res.code !== 200) {
          this.$message.error('添加应用失败！')
        } else {
          this.$message.success('添加应用成功！')
          await this.getAppList()
          this.addApp.appName = ''
          this.addAppVisible = false
        }
      })
    },
    async getAppList () {
      const { data: res } = await this.$http.get('/user/appList')
      this.appListData = res.data
    },
    async unBindApp (appId) {
      this.$confirm('此操作将解除绑定, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        // eslint-disable-next-line no-unused-vars
        const { data: res } = await this.$http.delete('/user/app?appId=' + appId)
        await this.getAppList()
        this.$message({
          type: 'success',
          message: '解绑成功!'
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消操作'
        })
      })
    },
    async getAppConfig (Id) {
      this.appConfigVisible = true
      const { data: res } = await this.$http.get('/auth/?appId=' + Id)
      this.appConfig = res.data
      this.appGrade = res.data.appGrade
    },
    addAppGradeMethod () {
      this.$refs.addGradeRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('/auth/?appId=' + this.appConfig.appId, this.addGrade)
        if (res.code === 415) {
          this.$message(res.message)
          this.addAppGradeVisible = false
        } else if (res.code === 200) {
          await this.getAppConfig(this.appConfig.appId)
          this.addAppGradeVisible = false
        } else {
          this.$message('添加失败')
        }
        this.addGrade.grade = null
        this.addGrade.gradePassword = ''
      })
    },
    async deletePasswordMethod (grade) {
      this.$confirm('此操作将删除该级别, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        // eslint-disable-next-line no-unused-vars
        const { data: res } = await this.$http.delete('/auth/?appId=' + this.appConfig.appId + '&authGrade=' + grade)
        if (res.code === 200) {
          this.$message.success('删除成功')
          await this.getAppConfig(this.appConfig.appId)
        } else {
          this.$message.error('删除失败')
        }
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消操作'
        })
      })
    },
    bindAppMethod () {
      this.$refs.bindAppRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('/user/app', this.bindApp)
        if (res.code === 200) {
          this.$message.success('绑定成功')
          await this.getAppList()
          this.bindAppVisible = false
        }
      })
    }
  }
}
</script>

<style lang="less" scoped>
.el-container {
  height: 100%;
}

.el-header {
  display: flex;
  flex-direction: row;
  justify-content: space-between;
  align-items: center;
  background-color: #409EFF;

  > div {
    display: flex;
    align-items: center;

    img {
      height: 40px;
    }

    span {
      margin-left: 15px;
      color: #FFFFFF;
      font-size: xx-large;
    }
  }

  > el-button {
    height: 40px;
  }
}

.el-main {
  background-color: #F2F6FC;
}
</style>
