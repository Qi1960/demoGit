<template>
  <div>
    <el-card id="search">
      <el-row>
        <el-col :span="20" align="light">
          <el-input v-model="searchModel.username" placeholder="用户名" clearable />
          <el-input v-model="searchModel.phone" placeholder="电话" clearable />
          <el-button type="primary" icon="el-icon-search" @click="getUserList()">搜索</el-button>
        </el-col>
        <!-- <el-col>
          <el-upload
            class="upload-demo"
            action="http://localhost:8082/redexcel"
            :on-preview="handlePreview"
            :on-remove="handleRemove"
            :on-success="uploadSuccess"
            multiple
            name="multipartFile"
            :limit="3"
          >
            <el-button size="small" type="primary">点击上传</el-button>
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
          </el-upload>

        </el-col> -->
        <el-col :span="4" align="right">
          <el-button type="primary" @click="openEditUI(null)">新增用户</el-button>
        </el-col>
      </el-row>
    </el-card>
    <!-- 结果列表 -->
    <el-card>
      <el-table :data="userList" stripe style="width: 100%">
        <el-table-column label="#" width="80">
          <template slot-scope="scope">
            {{ (searchModel.pageNo - 1) * searchModel.pageSize + scope.$index + 1 }}
          </template>
        </el-table-column>
        <!-- <el-table-column prop="id" label="用户id" /> -->
        <el-table-column prop="username" label="用户名" />
        <el-table-column prop="phone" label="电话" />
        <el-table-column prop="status" label="状态">
          <template slot-scope="scope">
            <el-tag v-if="scope.row.status == 1" type="success">正常</el-tag>
            <el-tag v-else type="danger">禁用</el-tag>
          </template>
        </el-table-column>
        <el-table-column prop="email" label="邮箱" />
        <el-table-column prop="created" label="创建时间" />
        <el-table-column prop="updateTime昵称" />
        <el-table-column prop="avatar" label="头像" />
        <!-- <el-table-column prop="avatar" label="操作" width="150">
          <template slot-scope="scope">
            <img
              style="width: 100px; height: 100px"
              :src="scope.row.avatar"
              :preview-src-list="scope.row.avatar"
            >
          </template>
        </el-table-column> -->
        <el-table-column prop="avatar" label="头像" width="150">
          <template slot-scope="scope">
            <div class="demo-image__preview">
              <el-image
                style="width: 70px; height: 70px"
                :src="scope.row.avatar"
              />
            </div>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="150">
          <template slot-scope="scope">
            <el-button type="text" @click="openEditUI(scope.row.id)">编辑</el-button>
            <el-button type="text" @click="deleteUser(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 分页查询组件 -->
    <el-pagination
      :current-page="searchModel.pageNo"
      :page-sizes="[5, 20, 50, 100]"
      :page-size="searchModel.pageSize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total"
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
    />
    <!-- 新增用户表单
    <el-dialog :visible.sync="dialogFormVisible" title="title" :rules="rules" @click="clearForm">
      <el-form :model="userForm">
        <el-form-item label="用户名" prop="username" :label-width="formLabelWidth">
          <el-input v-model="userForm.username" autocomplete="off" />
          <el-form-item label="用户密码" prop="password" :label-width="formLabelWidth">
            <el-input v-model="userForm.password" type="password" autocomplete="off" />
            <el-form-item label="联系电话" :label-width="formLabelWidth">
              <el-input v-model="userForm.phone" autocomplete="off" />
              <el-form-item label="电子邮件" prop="email" :label-width="formLabelWidth">
                <el-input v-model="userForm.email" autocomplete="off" />
                <el-form-item label="用户状态" :label-width="formLabelWidth">
                  <el-switch
                    v-model="userForm.status"
                    :inactive-value="0"
                    :active-value="1"
                  />
                </el-form-item>
              </el-form-item></el-form-item></el-form-item></el-form-item></el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
      </div>
    </el-dialog> -->
    <!-- 新增用户dialog表单 -->
    <el-dialog :visible.sync="dialogFormVisible" :title="title" @close="clearForm">
      <el-form ref="userFormRef" :model="userForm" :rules="rules">
        <el-form-item label="用户名" prop="username" :label-width="formLabelWidth">
          <el-input v-model="userForm.username" autocomplete="off" />
        </el-form-item>
        <el-form-item v-if="userForm.id == null || userForm.id == undefined" label="用户密码" prop="password" :label-width="formLabelWidth">
          <el-input v-model="userForm.password" type="password" autocomplete="off" />
        </el-form-item>
        <el-form-item label="联系电话" prop="phone" :label-width="formLabelWidth">
          <el-input v-model="userForm.phone" autocomplete="off" />
        </el-form-item>
        <el-form-item label="电子邮件" prop="email" :label-width="formLabelWidth">
          <el-input v-model="userForm.email" autocomplete="off" />
        </el-form-item>
        <el-form-item label="用户状态" prop="status" :label-width="formLabelWidth">
          <el-switch
            v-model="userForm.status"
            :inactive-value="0"
            :active-value="1"
          />
        </el-form-item>
        <el-form-item label="创建时间" prop="status" :label-width="formLabelWidth">
          <el-date-picker
            v-model="userForm.created"
            type="datetime"
            placeholder="选择日期时间"
            default-time="12:00:00"
          />
        </el-form-item>
        <el-form-item label="用户角色" :label-width="formLabelWidth">
          <el-checkbox-group
            v-model="userForm.roleIdList"
            style="width: 85%"
            :max="2"
          >
            <el-checkbox
              v-for="(item) in roleList"
              :key="item.roleId"
              :label="item.roleId"
            >
              {{ item.roleDesc }}
            </el-checkbox>
          </el-checkbox-group>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveUser">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import userApi from '@/api/userManage'
import roleApi from '@/api/roleManage'

export default {
// 邮箱格式校验
  data() {
    var checkEmail = (rule, value, callback) => {
      var reg = /^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/
      if (!reg.test(value)) {
        return callback(new Error('请输入正确的邮箱格式'))
      } else {
        callback()
      }
    }
    return {
      src: '',
      srcList: [],
      // 图片
      roleList: [],
      userForm: { roleIdList: [] },
      title: '',
      dialogFormVisible: false,
      total: 0,
      searchModel: {
        pageNo: 1,
        pageSize: 5
      },
      userList: [],
      // 表单信息检验
      rules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 2, max: 50, message: '长度在 2 到 50 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 6, max: 16, message: '长度在 6 到 16 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getUserList()
    this.getAllRole()
  },
  methods: {

    getAllRole() {
      roleApi.getAllRole().then(res => {
        console.log(res.data)
        this.roleList = res.data
      })
    },
    deleteUser(user) {
      this.$confirm(`您是否确认删除 ${user.username}`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        userApi.deleteUserById(user.id).then(res => {
          this.$message({
            type: 'success',
            message: res.msg
          })
          this.getUserList()
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
      })
    },
    // 保存新增用户信息
    saveUser() {
      // 触发表单验证
      this.$refs.userFormRef.validate(valid => {
        if (valid) {
          // 提交后台
          userApi.saveUser(this.userForm).then(response => {
            // 成功提示
            this.$message({
              message: response.msg,
              type: 'success'
            })
            // 关闭弹窗
            this.dialogFormVisible = false
            // 刷新列表
            this.getUserList()
          })
        } else {
          console.log('请输入正确的用户信息')
          return false
        }
      })
      this.clearForm()
    },
    // 表单回调、清空新增用户表单内容
    clearForm() {
      this.userForm = {
        roleIdList: []
      }
      this.$refs.userFormRef.clearValidate()
    },
    // 打开新增用户弹窗
    openEditUI(id) {
      if (id == null) {
        this.title = '新增用户'
      } else {
        this.title = '修改用户信息'
        // 根据用户id查询用户信息
        userApi.getUserById(id).then(response => {
          this.userForm = response.data
        })
      }
      this.dialogFormVisible = true
    },
    handleSizeChange(pageSize) {
      this.searchModel.pageSize = pageSize
      this.getUserList()
    },
    handleCurrentChange(pageNo) {
      this.searchModel.pageNo = pageNo
      this.getUserList()
    },
    // 获取用户列表
    getUserList() {
      userApi.getUserList(this.searchModel).then(response => {
        this.userList = response.data.row
        this.total = response.data.total
      })
    }
  }
}
</script>

<style>
#search .el-input {
  width: 200px;
  margin-right: 10px;
}
#search .el-card {
  margin-bottom: 10px;
}
.el-dialog .el-input {
  width: 50%;
}
</style>
