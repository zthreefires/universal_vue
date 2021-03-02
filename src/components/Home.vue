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
           <el-table :data="appListData" style="width: 100%">
             <el-table-column type="index">
             </el-table-column>
             <el-table-column label="应用列表" prop="appName" width="200px">
             </el-table-column>
             <el-table-column label="应用Id" prop="appId" width="200px">
             </el-table-column>
             <el-table-column label="操作">
               <template slot-scope="scope">
                 <el-button icon="el-icon-star-on" type="primary" plain @click="openApp(scope.row.appId)">进入应用</el-button>
                 <el-button icon="el-icon-s-tools" type="primary" plain @click="getAppConfig(scope.row.appId)">应用设置
                 </el-button>
                 <el-button icon="el-icon-set-up" type="primary" plain @click="deviceTypeDefineMethod(scope.row.appId)">
                   设备定义
                 </el-button>
                 <el-button icon="el-icon-copy-document" type="primary" plain @click="deviceManage(scope.row.appId)">设备列表</el-button>
                 <el-upload
                   style="display: inline; padding: 10px"
                   :show-file-list="false"
                   :on-success="onSuccess"
                   :on-error="onError"
                   :action="uploadAction">
                   <el-button icon="el-icon-upload" type="primary" plain @click="processAction(scope.row.appId)">页面上传</el-button>
                 </el-upload>
                 <el-button icon="el-icon-s-release" type="danger" @click="unBindApp(scope.row.appId)">解除绑定</el-button>
               </template>
             </el-table-column>
           </el-table>
      </el-card>
      <!--添加应用对话框-->
      <el-dialog :visible.sync="addAppVisible" title="新建应用" width="30%">
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
      <el-dialog :visible.sync="bindAppVisible" title="绑定应用" width="30%">
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
      <!--设备定义对话框-->
      <el-dialog :visible.sync="deviceTypeVisible" title="设备定义" width="70%">
        <el-button type="primary" @click="addDeviceTypeVisible = true">添加设备类型</el-button>
        <el-table :data="deviceType" style="width: 100%">
          <el-table-column label="设备类型Id" prop="deviceTypeId">
          </el-table-column>
          <el-table-column label="设备类型" prop="deviceTypeName">
          </el-table-column>
          <el-table-column label="操作">
            <template scope="scope2">
              <el-tooltip effect="dark" content="查看该类型通信协议" placement="top">
                <el-button icon="el-icon-view" type="primary" size="mini"
                           @click="getFieldInfo(scope2.row.deviceTypeId); fieldInfoVisible = true"></el-button>
              </el-tooltip>
              <el-tooltip effect="dark" content="删除设备类型" placement="top">
                <el-button icon="el-icon-delete" type="danger" size="mini"
                           @click="deleteDeviceTypeMethod(scope2.row.deviceTypeId)"></el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
      </el-dialog>
      <!--      添加设备类型对话框-->
      <el-dialog :visible.sync="addDeviceTypeVisible" title="新建设备类型" width="30%">
        <el-form :model="addDeviceType" :rules="addDeviceTypeRules" ref="addDeviceTypeRef" label-width="120px">
          <el-form-item label="设备类型" prop="deviceTypeName">
            <el-input v-model="addDeviceType.deviceTypeName"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button
            @click="addDeviceTypeVisible = false; addDeviceType.appId = null; addDeviceType.deviceTypeName = null">取 消</el-button>
          <el-button type="primary" @click="addDeviceTypeMethod">确 定</el-button>
        </span>
      </el-dialog>
      <!-- 协议定义对话框 -->
      <el-dialog :visible.sync="fieldInfoVisible" title="通信协议定义" width="70%">
        <el-button type="primary" @click="addFieldInfoVisible = true">添加字段</el-button>
        <el-table :data="fieldInfo" style="width: 100%">
          <el-table-column label="字段Key" prop="fieldInfoKey">
          </el-table-column>
          <el-table-column label="字段值类型" prop="fieldInfoValueType">
          </el-table-column>
          <el-table-column label="是否可控" prop="fieldInfoWriteable">
          </el-table-column>
          <el-table-column label="控制该字段需要的最低级别" prop="fieldInfoWriteGrade">
          </el-table-column>
          <el-table-column label="读取该字段需要的最低级别" prop="fieldInfoReadGrade">
          </el-table-column>
          <el-table-column label="操作">
            <template scope="scope3">
              <el-tooltip effect="dark" content="编辑字段" placement="top">
                <el-button icon="el-icon-edit-outline" type="primary" size="mini"
                           @click="$message.info('还没想好要不要有编辑功能')"></el-button>
              </el-tooltip>
              <el-tooltip effect="dark" content="删除字段" placement="top">
                <el-button icon="el-icon-delete" type="danger" size="mini"
                           @click="deleteFieldMethod(scope3.row.fieldInfoId)"></el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
      </el-dialog>
      <!-- 添加字段对话框-->
      <el-dialog :visible.sync="addFieldInfoVisible" title="新建字段" width="40%">
        <el-form :model="addFieldInfo" :rules="addFieldInfoRules" ref="addFieldInfoRef" label-width="160px">
          <el-form-item label="字段关键字" prop="key">
            <el-input v-model="addFieldInfo.key"></el-input>
          </el-form-item>
          <el-form-item label="数据类型" prop="valueType">
            <el-select v-model="addFieldInfo.valueType" placeholder="请选择">
              <el-option v-for="item in valueTypeOptions" :key="item.value" :label="item.label" :value="item.value">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="是否可以控制" prop="writeable">
            <el-switch v-model="addFieldInfo.writeable" active-text="是" inactive-text="否"></el-switch>
          </el-form-item>
          <el-form-item label="控制需要的最低级别" prop="writeGrade" v-show="addFieldInfo.writeable">
            <el-input-number v-model="addFieldInfo.writeGrade" :min="1" :max="10" label="描述文字"></el-input-number>
          </el-form-item>
          <el-form-item label="读取需要的最低级别" prop="readGrade">
            <el-input-number v-model="addFieldInfo.readGrade" :min="1" :max="10" label="描述文字"></el-input-number>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addFieldInfoVisible = false; addFieldInfo = {
                key: null,
                valueType: null,
                writeable: false,
                writeGrade: 11,
                readGrade: null
              }">取 消</el-button>
          <el-button type="primary" @click="addFieldInfoMethod">确 定</el-button>
        </span>
      </el-dialog>
      <!--设备列表对话框-->
      <el-dialog :visible.sync="deviceListVisible" title="设备列表" width="60%">
        <el-button type="primary" @click="addDeviceVisible = true">添加设备</el-button>
        <el-table :data="deviceList" style="width: 100%">
          <el-table-column label="设备类型Id" prop="deviceTypeId">
          </el-table-column>
          <el-table-column label="设备Id" prop="deviceId">
          </el-table-column>
          <el-table-column label="设备名称" prop="deviceName">
          </el-table-column>
        </el-table>
      </el-dialog>
      <!--添加设备对话框-->
      <el-dialog  :visible.sync="addDeviceVisible" title="添加设备" width="40%">
        <el-form :model="addDevice" :rules="addDeviceRules" ref="addDeviceRef">
          <el-form-item label="设备类型" prop="deviceTypeId">
            <el-select v-model="addDevice.deviceTypeId" placeholder="请选择">
              <el-option
                v-for="item in deviceTypeOptions"
                :key="item.deviceTypeId"
                :label="item.deviceTypeName"
                :value="item.deviceTypeId">
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="设备名称" prop="deviceName">
            <el-input v-model="addDevice.deviceName"></el-input>
          </el-form-item>
        </el-form>
        <span slot="footer" class="dialog-footer">
          <el-button @click="addDeviceVisible = false; addDevice.deviceTypeId = null; addDevice.deviceName = ''">取 消</el-button>
          <el-button type="primary" @click="addDeviceMethod">确 定</el-button>
        </span>
      </el-dialog>
    </el-main>
  </el-container>
</template>

<script>
export default {
  data () {
    return {
      // Home页变量
      appListData: null,
      // 新建应用
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
      // 应用设置相关
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
      // 绑定应用
      bindAppVisible: false,
      bindApp: {
        appId: null,
        appPassword: null
      },
      bindAppRules: {
        appId: [
          {
            required: true,
            message: '请输入应用Id',
            trigger: 'blur'
          }
        ],
        appPassword: [
          {
            required: true,
            message: '请输入密码',
            trigger: 'blur'
          },
          {
            min: 6,
            max: 20,
            message: '密码长度在6-20个字符之间',
            trigger: 'blur'
          }
        ]
      },
      // 设备类型管理
      deviceTypeVisible: false,
      addDeviceTypeVisible: false,
      addDeviceType: {
        appId: null,
        deviceTypeName: null
      },
      addDeviceTypeRules: {
        deviceTypeName: [
          {
            required: true,
            message: '请输入设备类型',
            trigger: 'blur'
          }
        ]
      },
      deviceType: null,
      // 协议管理
      addFieldDeviceTypeId: -1,
      fieldInfoVisible: false,
      fieldInfo: null,
      addFieldInfoVisible: false,
      addFieldInfo: {
        key: null,
        valueType: null,
        writeable: false,
        writeGrade: 11,
        readGrade: null
      },
      addFieldInfoRules: {
        key: [
          {
            required: true,
            message: '请输入字段键值',
            trigger: 'blur'
          },
          {
            min: 1,
            max: 10,
            message: '长度小于10个字符',
            trigger: 'blur'
          }
        ],
        valueType: [
          {
            required: true,
            message: '请输入字段数值类型',
            trigger: 'blur'
          }
        ]
      },
      // 参数类型选项
      valueTypeOptions: [
        {
          value: 'int',
          label: 'int'
        },
        {
          value: 'long',
          label: 'long'
        },
        {
          value: 'boolean',
          label: 'boolean'
        },
        {
          value: 'float',
          label: 'float'
        },
        {
          value: 'double',
          label: 'double'
        }
      ],
      // 设备管理
      deviceListVisible: false,
      deviceList: null,
      addDeviceVisible: false,
      addDevice: {
        appId: null,
        deviceTypeId: null,
        deviceName: ''
      },
      addDeviceRules: {
        deviceTypeId: [
          { required: true, message: '请输入设备类型Id', trigger: 'blur' }
        ],
        deviceName: [
          { required: true, message: '请输入设备名称', trigger: 'blur' }
        ]
      },
      uploadAction: 'http://8.133.182.84:8888/fileUpload?file=2&appId=',
      deviceTypeOptions: null
    }
  },
  created () {
    this.getAppList()
  },
  methods: {
    // 登出
    logout () {
      window.sessionStorage.clear()
      this.$router.push('/login')
    },
    // 进入应用
    async openApp (appId) {
      const { data: res } = await this.$http.get('/app/isUpload?appId=' + appId)
      if (res.code === 200) {
        window.location.href = 'http://8.133.182.84:80/app/' + appId + '/index.html'
      } else {
        this.$message.error(res.message)
      }
    },
    // 新建应用
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
    // 获取该用户当前已绑定应用列表
    async getAppList () {
      const { data: res } = await this.$http.get('/user/appList')
      this.appListData = res.data
    },
    // 解除已绑定应用
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
    // 获取该应用应用设置相关信息
    async getAppConfig (Id) {
      this.appConfigVisible = true
      const { data: res } = await this.$http.get('/auth/?appId=' + Id)
      this.appConfig = res.data
      this.appGrade = res.data.appGrade
    },
    // 创建应用绑定级别
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
    // 删除应用绑定级别
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
    // 通过应用Id和对应级别密码绑定应用
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
    },
    // 查看设备类型
    async deviceTypeDefineMethod (appId) {
      const { data: res } = await this.$http.get('/deviceType?appId=' + appId)
      this.addDeviceType.appId = appId
      this.deviceType = res.data
      this.deviceTypeVisible = true
    },
    // 新建设备类型
    addDeviceTypeMethod () {
      this.$refs.addDeviceTypeRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('/deviceType', this.addDeviceType)
        if (res.code === 200) {
          this.$message.success('添加成功！')
          await this.deviceTypeDefineMethod(this.addDeviceType.appId)
          this.addDeviceType.appId = null
          this.addDeviceType.deviceTypeName = null
          this.addDeviceTypeVisible = false
        } else {
          this.$message.error('添加失败：' + res.message)
        }
      })
    },
    // 删除设备类型
    async deleteDeviceTypeMethod (deviceTypeId) {
      this.$confirm('此操作将删除该设备类型, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        // eslint-disable-next-line no-unused-vars
        console.log(deviceTypeId)
        const { data: res } = await this.$http.delete('/deviceType?deviceTypeId=' + deviceTypeId)
        if (res.code === 200) {
          await this.deviceTypeDefineMethod(this.addDeviceType.appId)
          this.$message.success('删除成功')
        } else if (res.code !== 200) {
          this.$message.error('删除失败：' + res.message)
        }
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消操作'
        })
      })
    },
    // 获取协议字段信息
    async getFieldInfo (deviceTypeId) {
      this.addFieldDeviceTypeId = deviceTypeId
      const { data: res } = await this.$http.get('/fieldInfo?deviceTypeId=' + deviceTypeId)
      this.fieldInfo = res.data
      for (let i = 0; i < this.fieldInfo.length; i++) {
        if (this.fieldInfo[i].fieldInfoWriteable === true) {
          this.fieldInfo[i].fieldInfoWriteable = '是'
        } else {
          this.fieldInfo[i].fieldInfoWriteable = '否'
          this.fieldInfo[i].fieldInfoWriteGrade = '该字段不可控'
        }
      }
    },
    // 删除字段
    async deleteFieldMethod (fieldInfoId) {
      const { data: res } = await this.$http.delete('/fieldInfo?deviceTypeId=' + this.fieldInfo[0].fieldInfoDeviceTypeId + '&fieldId=' + fieldInfoId)
      if (res.code === 200) {
        await this.getFieldInfo(this.fieldInfo[0].fieldInfoDeviceTypeId)
        this.$message.success(res.message)
      } else {
        this.$message.error(res.message)
      }
    },
    // 新建字段
    async addFieldInfoMethod () {
      const { data: res } = await this.$http.post('/fieldInfo?deviceTypeId=' + this.addFieldDeviceTypeId, this.addFieldInfo)
      if (res.code === 200) {
        await this.getFieldInfo(this.addFieldDeviceTypeId)
        this.$message.success('添加成功')
      } else {
        this.$message.error('添加失败' + res.message)
      }
      this.addFieldInfoVisible = false
    },
    // 上传页面成功的钩子函数
    onSuccess (response, file, fileList) {
      this.uploadAction = 'http://localhost:8888/fileUpload?file=2&appId='
      this.$message.success('上传成功')
    },
    onError (response, file, fileList) {
      this.uploadAction = 'http://localhost:8888/fileUpload?file=2&appId='
      this.$message.error('上传失败')
    },
    processAction (appId) {
      this.uploadAction = 'http://8.133.182.84:8888/fileUpload?file=2&appId='
      this.uploadAction = this.uploadAction + appId
      console.log(this.uploadAction)
    },
    // 设备管理页面初始数据
    async deviceManage (appId) {
      const { data: res } = await this.$http.get('/device/deviceList?appId=' + appId)
      this.deviceList = res.data
      this.deviceListVisible = true
      this.addDevice.appId = appId
      const { data: res1 } = await this.$http.get('/deviceType?appId=' + appId)
      this.deviceTypeOptions = res1.data
    },
    // 新建设备
    async addDeviceMethod () {
      this.$refs.addDeviceRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('/device', this.addDevice)
        if (res.code === 200) {
          this.$message.success('添加成功')
          await this.deviceManage(this.addDevice.appId)
          this.addDeviceVisible = false
        } else {
          this.$message.error('添加失败：' + res.message)
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
      white-space: nowrap;
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
