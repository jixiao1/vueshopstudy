<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片区域 -->
    <el-card class="box-card">
     <!-- 输入框和按钮的区域有添加用户的功能 -->
     <el-row :gutter="20">
          <el-col :span="7">
            <!--@clear="useshujuList" 清空input的内容并重置数据  -->
            <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="useshujuList">
              <el-button slot="append" icon="el-icon-search" @click="useshujuList"></el-button>
            </el-input>
          </el-col>
          <el-col :span="3">
             <el-button type="primary" @click="addSAialogVisible=true">添加用户</el-button>
          </el-col>
        </el-row>
        <!-- 数据列表页面 -->
        <el-table :data="userList" stripe border>
          <el-table-column type="index"></el-table-column>
          <el-table-column prop="username" label="用户名"></el-table-column>
          <el-table-column prop="email" label="邮箱"></el-table-column>
          <el-table-column prop="mobile" label="电话"></el-table-column>
          <el-table-column prop="role_name" label="角色"></el-table-column>
          <el-table-column prop="mg_state" label=状态>
            <!-- 作用域插槽 -->
            <template slot-scope="scope">
              <!-- {{scope.row}} -->
              <el-switch
              v-model="scope.row.mg_state" @change="getStateChange(scope.row)">
              </el-switch>
              </template>
          </el-table-column>
          <el-table-column  label="操作" width="180px">
            <!-- 作用域插槽 -->
            <template slot-scope="scope">
              <!-- 修改按钮 -->
              <el-button type="primary" icon="el-icon-edit" size="mini" @click="EditdialogVisibles(scope.row.id)"></el-button>
              <!--  删除按钮 -->
              <el-button type="danger"
              icon="el-icon-delete" size="mini" @click="removeUserInfo(scope.row.id)"></el-button>
              <!-- 分配按钮 -->
              <!-- <el-button type="wraning" icon="el-icon-setting"></el-button> -->
               <el-tooltip class="item" effect="dark" content="分配角色" placement="top">
                 <el-button type="warning" icon="el-icon-setting" size="mini"></el-button>
              </el-tooltip>
            </template>
          </el-table-column>
        </el-table>
        <!--  页码区域 -->
    <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[1, 3, 5, 10]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
    </el-card>
    <!-- 添加用户的对话框 -->
    <el-dialog
    title="添加用户"
    :visible.sync="addSAialogVisible"
    width="50%"
     @close="addSAialogClose">
    <!--  内容主体区域 -->
    <el-form :model="UserruleForm"
    :rules="Userrules"
    ref="ruleForm"
    label-width="100px">
      <el-form-item label="用户名" prop="username">
        <el-input v-model="UserruleForm.username"></el-input>
     </el-form-item>
     <el-form-item label="密码" prop="password">
        <el-input v-model="UserruleForm.password"></el-input>
     </el-form-item>
     <el-form-item label="邮箱" prop="email">
        <el-input v-model="UserruleForm.email"></el-input>
     </el-form-item>
     <el-form-item label="手机号" prop="mobile">
        <el-input v-model="UserruleForm.mobile"></el-input>
     </el-form-item>
    </el-form>
    <!--  底部区域 -->
    <span slot="footer" class="dialog-footer">
    <el-button @click="addSAialogVisible = false">取 消</el-button>
    <el-button type="primary"  @click="addUser">确 定</el-button>
    </span>
    </el-dialog>
    <!--  编辑用户对话框 -->
    <!-- 对话框的关闭事件 -->
    <el-dialog
    title="编辑用户"
    :visible.sync="EditdialogVisible"
    width="50%"
    @close="EditdialogVisibleClose"
    >
    <el-form
    :model="EditFormObject" label-width="80px"
    :rules="Editrules" ref="EditFormObjectRef">
      <el-form-item label="用户名">
      <el-input v-model="EditFormObject.username"  disabled></el-input>
      </el-form-item>
      <el-form-item label="邮箱"  prop="email">
      <el-input v-model="EditFormObject.email"></el-input>
      </el-form-item>
      <el-form-item label="手机号" prop="mobile">
      <el-input v-model="EditFormObject.mobile"></el-input>
      </el-form-item>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="EditdialogVisible = false">取 消</el-button>
      <el-button type="primary"
      @click="EditUserInfo">确 定</el-button>
    </span>
    </el-dialog>
    </div>
</template>

<script>
import axios from 'axios'
export default {
  created () {
    this.useshujuList()
  },
  data () {
    //  自定义邮箱的校验规则
    var checkEmail = (rule, value, cb) => {
      const regEmail = new RegExp('^[a-z0-9A-Z]+[- | a-z0-9A-Z . _]+@([a-z0-9A-Z]+(-[a-z0-9A-Z]+)?\\.)+[a-z]{2,}$')
      // if(!value) {
      //   return cb(new Error('邮箱不能为空'))
      // }
      if (regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的邮箱'))
    }
    //  自定义手机号的校验规则
    var checkMobile = (rule, value, cb) => {
      let regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/
      if (regMobile.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的手机号'))
    }
    // //  自定义密码的校验规则 密码为8为以上
    // var checkPassWord = (rule, value, cb) => {
    //   const regPassWord = new RegExp('^(?=.*[a-z])(?=.*[A-Z])(?=.*\d)[^]{8,16}$')
    //   if (regPassWord.test(value)) {
    //     return cb()
    //   }
    //   cb(new Error('请输入正确的密码'))
    // }
    return {
      // 获取用户列表的参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 3
      },
      userList: [],
      total: 0, // 数据总条数
      //  控制对话框的隐藏于显示
      addSAialogVisible: false,
      //  需要数据绑定对象
      UserruleForm: {
        username: '',
        password: '',
        email: '',
        mobile: ''
      },
      //  用户名的验证规则对象
      Userrules: {
        username: [
          { required: true, message: '请输入用户名', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        password: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 3, max: 10, message: '长度在 3 到 10 个字符', trigger: 'blur' }
        ],
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      },
      //  编辑用户对话框
      EditdialogVisible: false,
      //  编辑查询到的用户对话框对象
      EditFormObject: {},
      // 编辑对话框校验规则
      Editrules: {
        email: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          { validator: checkEmail, trigger: 'blur' }
        ],
        mobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          { validator: checkMobile, trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    //  获取数据的方法
    useshujuList () {
      axios.get('users', {
        params: this.queryInfo
      })
        .then(result => {
          // console.log(result.data.data)
          this.total = result.data.data.total
          // console.log(result.data.data.users)
          this.userList = result.data.data.users
        })
        .catch(() => {
          alert('数据获取失败')
        })
    },
    //  一页有几条数据的方法
    handleSizeChange (val) {
      // console.log(val)
      this.queryInfo.pagesize = val
      this.useshujuList()
    },
    //  当前是第几页的方法
    handleCurrentChange (val) {
      // console.log(val)
      this.queryInfo.pagenum = val
      this.useshujuList()
    },
    //  改变用户 状态的方法
    getStateChange (uesrinfo) {
      // console.log(uesrinfo)
      // 更改数据接口的改变
      axios.put(`users/${uesrinfo.id}/state/${uesrinfo.mg_state}`)
        .then(response => {
          this.$message.success('数据更新成功')
        })
        .catch(() => {
          uesrinfo.mg_state = !uesrinfo.mg_state
          return this.$message.error('获取数据失败')
        })
    },
    // 添加用户 点击对话框的关闭事件
    addSAialogClose () {
      this.$refs.ruleForm.resetFields()
    },
    //  点击添加用户
    addUser () {
      this.$refs.ruleForm.validate(async vali => {
        // console.log(1111)
        // console.log(vali)
        if (!vali) return
        // console.log(111)
        //  可以添加用户的网络请求
        const { data: res } = await this.$http.post('users', this.UserruleForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加用户失败!')
        }

        // 添加用户成功
        this.$message.success('添加用户成功！')
        //  隐藏添加用户的对话框
        this.addSAialogVisible = false
        //  重新获取数据列表
        this.useshujuList()
      })
    },
    // 控制编辑用户框的显示
    async EditdialogVisibles (id) {
      // console.log(id)
      // 根据 ID 查询用户信息
      // axios.get('users', {
      //   params: id
      // })
      //   .then(response => {
      //     console.log(response)
      //   })
      //   .catch((error) => {
      //     console.log(error)
      //   })
      const { data: res } = await this.$http.get('users/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败')
      }
      // console.log(res)
      console.log(res.data)
      this.EditFormObject = res.data
      this.EditdialogVisible = true
    },
    //  监听修改用户对话框的关闭事件
    EditdialogVisibleClose () {
      //  对整个 编辑表单的校验重置
      this.$refs.EditFormObjectRef.resetFields()
    },
    //  修改用户信息并提交
    EditUserInfo () {
      // console.log(111)
      //  校验表单的规则
      this.$refs.EditFormObjectRef.validate(async val => {
        // console.log(val)
        if (!val) return
        //  发起修改用户数据的请求
        const { data: res } = await this.$http.put('users/' + this.EditFormObject.id, {
          email: this.EditFormObject.email,
          mobile: this.EditFormObject.mobile
        })
        if (res.meta.status !== 200) {
          return this.$message.error('更新用户信息失败')
        }
        //  关闭对话框 刷新数据列表
        this.EditdialogVisible = false
        this.useshujuList()
        this.$message.success('更新用户成功')
      })
    },
    //  通过用户的Id 删除该用户
    removeUserInfo (id) {
      // console.log(id)
      //  调用Messagevbox中的利用confirm 是一个promise对象
      this.$confirm('此操作将永久删除用户, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          axios.delete('users/' + id)
            .then((data) => {
              // console.log(data.request.status)
              if (data.request.status === 200) {
                this.useshujuList()
                this.$message.success('删除用户成功')
              }
            })
        })
        .catch(() => {
          this.$message.error('已取消删除')
        })
    }
  }
}
</script>

<style lang='less' scoped>

</style>
