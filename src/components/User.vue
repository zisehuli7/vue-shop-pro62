<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- card卡片区域-->
    <el-card class="box-card">
      <!-- 搜索框 和 添加用户按钮 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入搜索内容"
            v-model="querycdt.query"
            :clearable="true"
            @clear="getUserInfos"
            @keyup.enter.native="getUserInfos"
          >
            <el-button slot="append" icon="el-icon-search" @click="getUserInfos"></el-button>
          </el-input>
        </el-col>
        <el-col :span="7">
          <el-button type="primary" @click="addDialogVisible=true">添加用户</el-button>
        </el-col>
      </el-row>

      <!-- 表格展示数据列表 -->
      <el-table :data="userInfos" border style="width: 100%">
        <el-table-column type="index" label="序号" width="60"></el-table-column>
        <el-table-column prop="username" label="用户名" width="110"></el-table-column>
        <el-table-column prop="mobile" label="手机号" width="110"></el-table-column>
        <el-table-column prop="role_name" label="角色" width="120"></el-table-column>
        <el-table-column prop="email" label="邮箱" width="160"></el-table-column>
        <el-table-column prop="mg_state" label="状态" width="70">
          <!-- 插槽填充 -->
          <el-switch
            v-model="info.row.mg_state"
            slot-scope="info"
            @change="changeState(info.row.id, info.row.mg_state)"
          ></el-switch>
        </el-table-column>
        <el-table-column label="操作" width="230">
          <template slot-scope="info">
            <!-- 修改按钮 -->
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="showEditDialog(info.row.id)"
            ></el-button>

            <!-- 删除按钮 -->
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="delUser(info.row.id)"
            ></el-button>

            <el-tooltip content="分配角色" placement="top" :enterable="false">
              <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 数据分页展示 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querycdt.pagenum"
        :page-sizes="[3, 5, 10, 20]"
        :page-size="querycdt.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="querycdt.tot"
      ></el-pagination>

      <!-- 添加用户对话框 -->
      <el-dialog title="添加用户" :visible.sync="addDialogVisible" width="50%" @close="addDialogClose">
        <!-- :rules 校验 -->
        <el-form :rules="addFormRules" ref="addFormRef" :model="addForm" label-width="80px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addForm.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addForm.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号码" prop="mobile">
            <el-input v-model="addForm.mobile"></el-input>
          </el-form-item>
        </el-form>

        <span slot="footer" class="dialog-footer">
          <el-button @click="addDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="addUser">确 定</el-button>
        </span>
      </el-dialog>

      <!-- 修改用户对话框 -->
      <el-dialog
        title="修改用户"
        :visible.sync="editDialogVisible"
        width="50%"
        @close="editDialogClose"
      >
        <el-form :rules="editFormRules" ref="editFormRef" :model="editForm" label-width="80px">
          <el-form-item label="用户名" prop="username">
            <el-input v-model="editForm.username" :disabled="true"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editForm.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号码" prop="mobile">
            <el-input v-model="editForm.mobile"></el-input>
          </el-form-item>
        </el-form>

        <span slot="footer" class="dialog-footer">
          <el-button @click="editDialogVisible = false">取 消</el-button>
          <el-button type="primary" @click="editUser">确 定</el-button>
        </span>
      </el-dialog>
    </el-card>
  </div>
</template>
      
<script>
export default {
  // 生命周期函数
  created() {
    this.getUserInfos()
  },

  methods: {
    // 修改用户
    // 显示修改用户的对话框
    // 修改用户 收集数据存储
    editUser() {
      // 校验表单
      this.$refs.editFormRef.validate(async valid => {
        if (valid) {
          // 校验成功处理
          // 收集数据存储入库
          const { data: res } = await this.$http.put(
            'users/' + this.editForm.id,
            this.editForm
          )
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          // 修改成功 关闭对话框 成功提示页面数据更新
          this.editDialogVisible = false
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        }
      })
    },
    // param id: 被修改用户的id
    async showEditDialog(id) {
      // 显示对话框
      this.editDialogVisible = true
      // 根据id获得被修改用户的信息
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 把获得到的用户信息 赋予给 editForm 表单数据对象
      this.editForm = res.data
    },
    // 删除用户
    delUser(id) {
      this.$confirm('你确定要删除该用户么?', '删除用户', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(async () => {
          // axios调用api删除数据
          const { data: res } = await this.$http.delete('users/' + id)
          if (res.meta.status !== 200) {
            return this.$message.error(res.meta.msg)
          }
          // 删除成功 消息提示 刷新数据
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        })
        .catch(() => {})
    },

    // 收集数据储存入库 (执行后端api数据接口)
    addUser() {
      // 先校验form表单 validate
      this.$refs.addFormRef.validate(async valid => {
        if (valid) {
          // 校验成功
          const { data: res } = await this.$http.post('users', this.addForm)

          // 添加失败
          if (res.meta.status !== 201) {
            return this.$message.error(res.meta.msg)
          }

          // 添加成功 关闭对话框 成功提示 显示新添加用户
          this.addDialogVisible = false
          this.$message.success(res.meta.msg)
          this.getUserInfos()
        }
      })
    },
    // 对话框关闭回调
    addDialogClose() {
      this.$refs.addFormRef.resetFields()
    },
    // 修改用户状态的方法
    // uid 被修改状态用户的id值
    // state 被修改后的状态信息 true/false
    async changeState(uid, state) {
      const { data: res } = await this.$http.put(
        '/users/' + uid + '/state/' + state
      )
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 提示修改状态成功的信息
      this.$message.success(res.meta.msg)
    },
    // 每页记录条数变化的回调处理
    handleSizeChange(arg) {
      // arg:变化后的记录条数
      this.querycdt.pagesize = arg
      // 重新根据条件获取数据
      this.getUserInfos()
    },
    // 当前页面变化回调处理
    handleCurrentChange(arg) {
      // arg:变化后的当前页码值
      this.querycdt.pagenum = arg
      // 根据变化后的页码重新获得数据
      this.getUserInfos()
    },

    // 获得用于显示的真实用户列表
    async getUserInfos() {
      const { data: res } = await this.$http.get('users', {
        params: this.querycdt
      })

      // 判断获取是否失败
      if (res.meta.status !== 200) {
        return this.$message.error(res.meta.msg)
      }
      // 记录总条数
      this.querycdt.tot = res.data.total
      // 把获得好的数据 赋予给 userInfos成员
      this.userInfos = res.data.users
    }
  },
  data() {
    // 为校验手机号码生成一个函数
    var checkMobile = (rule, value, callback) => {
      // 手机号正则表达式校验
      if (!/^1[35789]\d{9}$/.test(value)) {
        // 校验失败(给页面提示错误信息)
        return callback(new Error('手机格式不正确'))
      }
      // 校验成功继续执行
      callback()
    }
    return {
      // 修改用户相关
      // 控制修改用户对话框是否显示(true:显示 false:隐藏)
      editDialogVisible: false,
      editForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 制作表单域校验规则
      editFormRules: {
        mobile: [
          { required: true, message: '手机号码必填', trigger: 'blur' },
          // 自定义校验手机号码规则
          // { validator: 校验函数, trigger: 'blur' }
          { validator: checkMobile, trigger: 'blur' }
        ]
      },

      // 添加用户相关1
      // 控制添加用户对话框是否显示(true:显示 false:隐藏)
      addDialogVisible: false,
      addForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      // 制作表单域校验规则
      addFormRules: {
        username: [{ required: true, message: '用户名必填', trigger: 'blur' }],
        password: [{ required: true, message: '密码必填', trgger: 'blur' }],
        mobile: [
          { required: true, message: '手机号码必填', trigger: 'blur' },
          // 自定义校验手机号码规则
          // { validator: 校验函数, trigger: 'blur' }
          { validator: checkMobile, trigger: 'blur' }
        ]
      },

      // 搜索关键字
      keywords: '',
      // 用户数据
      userInfos: [],
      // 给获取用户数据设置查询条件
      querycdt: {
        // 查询关键字
        query: '',
        // 当前页码
        pagenum: 1,
        // 每页获取记录条数
        pagesize: 3,
        // 总记录条数
        tot: 0
      }
    }
  }
}
</script>

<style lang="less" scoped>
</style>
