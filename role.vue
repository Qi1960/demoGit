<template>
  <div>
    <el-card id="search">
      <el-row>
        <el-col :span="20" align="light">
          <el-input v-model="searchModel.roleName" placeholder="角色名称" clearable />
          <el-button type="primary" icon="el-icon-search" @click="getRoleList()">搜索</el-button>
        </el-col>
        <el-col :span="4" align="right">
          <el-button type="primary" @click="openEditUI(null)">新增角色</el-button>
        </el-col>
      </el-row>
    </el-card>
    <!-- 结果列表 -->
    <el-card>
      <el-table :data="roleList" stripe style="width: 100%">
        <el-table-column label="#" width="80">
          <template slot-scope="scope">
            {{ (searchModel.pageNo - 1) * searchModel.pageSize + scope.$index + 1 }}
          </template>
        </el-table-column>
        <el-table-column prop="roleId" label="角色id" />
        <el-table-column prop="roleName" label="角色名称" />
        <el-table-column prop="roleDesc" label="角色描述" />
        <el-table-column label="操作" width="150">
          <template slot-scope="scope">
            <el-button type="text" @click="openEditUI(scope.row.roleId)">编辑</el-button>
            <el-button type="text" @click="deleteRole(scope.row)">删除</el-button>
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
    <!-- 新增/修改用户角色信息弹窗 -->
    <el-dialog :visible.sync="dialogFormVisible" :title="title" @close="clearForm">
      <el-form ref="roleFormRef" :model="roleForm" :rules="rules">
        <el-form-item label="角色名称" prop="roleName" :label-width="formLabelWidth">
          <el-input v-model="roleForm.roleName" autocomplete="off" />
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc" :label-width="formLabelWidth">
          <el-input v-model="roleForm.roleDesc" autocomplete="off" />
        </el-form-item>
        <el-form-item label="权限设置" prop="menuIdList" :label-width="formLabelWidth">
          <el-tree
            ref="menuRef"
            :data="menuList"
            :props="menuProps"
            node-key="menuId"
            show-checkbox
            style="width:85%"
            default-expand-all
          />
        </el-form-item>

      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRole">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import roleApi from '@/api/roleManage'
import menuApi from '@/api/menuManage'

export default {
// 邮箱格式校验
  data() {
    return {
      formLabelWidth: '130px',
      // 权限菜单
      menuList: [],
      // 权限菜单树形结构
      menuProps: {
        children: 'children',
        label: 'title'
      },
      roleForm: {},
      // 弹出框标题
      title: '',
      dialogFormVisible: false,
      total: 0,
      searchModel: {
        pageNo: 1,
        pageSize: 5
      },
      roleList: [],
      // 表单信息检验
      rules: {
        roleName: [
          { required: true, message: '请输入用户角色名称', trigger: 'blur' },
          { min: 2, max: 50, message: '长度在 2 到 50 个字符', trigger: 'blur' }
        ],
        roleDesc: [
          { required: true, message: '请输入角色名称描述', trigger: 'blur' },
          { min: 3, max: 50, message: '长度在 3 到 50 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    this.getRoleList()
    // this.getAllRole()
  },
  methods: {

    // deleteRole(role) {
    //   this.$confirm(`您是否确认删除 ${role.roleName}`, '提示', {
    //     confirmButtonText: '确定',
    //     cancelButtonText: '取消',
    //     type: 'warning'
    //   }).then(() => {
    //     roleApi.deleteRoleById(role.roleId).then(res => {
    //       this.$message({
    //         type: 'success',
    //         message: res.message
    //       })
    //       this.getRoleList()
    //     }).catch(() => {
    //       this.$message({
    //         type: 'info',
    //         message: '已取消删除'
    //       })
    //     })
    //   })
    // },
    // 获取所有权限内容
    getAllMenu() {
      menuApi.getAllMenu().then(res => {
        this.menuList = res.data
      })
    },
    // 删除角色列表
    deleteRole(role) {
      this.$confirm(`您确定删除角色 ${role.roleName} ？`, '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        roleApi.deleteRoleById(role.roleId).then(response => {
          this.$message({
            type: 'success',
            message: response.msg
          })
          this.dialogFormVisible = false
          // 更新列表
          this.getRoleList()
        })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },

    // 保存新增用户角色信息
    saveRole() {
      // 触发表单验证
      const selected = this.$refs.menuRef.getCheckedKeys()
      const halfSelect = this.$refs.menuRef.getHalfCheckedKeys()
      this.roleForm.menuIdList = selected.concat(halfSelect)
      console.log(this.roleForm.menuIdList)
      // 表单验证
      this.$refs.roleFormRef.validate(valid => {
        if (valid) {
          // 提交后台
          roleApi.saveRole(this.roleForm).then(response => {
            // 成功提示
            this.$message({
              message: response.msg,
              type: 'success'
            })
            // 关闭弹窗
            this.dialogFormVisible = false
            // 刷新列表
            this.getRoleList()
          })
        } else {
          console.log('请输入正确的用户角色信息')
          return false
        }
      })
      this.clearForm()
    },
    // 表单回调、清空新增用户角色表单内容
    clearForm() {
      this.roleForm = {
        roleIdList: []
      }
      this.$refs.roleFormRef.clearValidate()
      // 清空末次勾选权限菜单
      this.$refs.menuRef.setCheckedKeys([])
    },

    // 打开新增用户弹窗
    openEditUI(roleId) {
      if (roleId == null) {
        this.title = '新增角色'
        // 显示权限内容
        this.getAllMenu()
      } else {
        this.title = '修改角色信息'
        // 根据用户id查询用户角色信息
        roleApi.getRoleById(roleId).then(response => {
          // 显示用户信息
          this.roleForm = response.data
          // 显示权限内容
          this.$refs.menuRef.setCheckedKeys(response.data.menuIdList)
        })
      }
      this.dialogFormVisible = true
    },

    handleSizeChange(pageSize) {
      this.searchModel.pageSize = pageSize
      this.getRoleList()
    },

    handleCurrentChange(pageNo) {
      this.searchModel.pageNo = pageNo
      this.getRoleList()
    },
    // 获取用户列表
    getRoleList() {
      roleApi.getRoleList(this.searchModel).then(response => {
        this.roleList = response.data.row
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
