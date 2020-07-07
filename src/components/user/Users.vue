<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card >
      <!-- 搜索区域 --> 
      <el-row :gutter="20">
        <el-col :span="7">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getUserList"> 
            <el-button slot="append" icon="el-icon-search" @click="getUserList" ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="dialogVisible = true">添加用户</el-button>
        </el-col>
      </el-row>
      <!-- 用户列表区域 -->
      <el-table :data="userList" border stripe>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column prop="username" label="姓名"></el-table-column>
        <el-table-column prop="email" label="邮箱"></el-table-column>
        <el-table-column prop="mobile" label="电话"></el-table-column>
        <el-table-column prop="role_name" label="角色"></el-table-column>
        <el-table-column  label="状态">
          <template v-slot="scope">
            <el-switch v-model="scope.row.mg_state" @change="stateChanged(scope.row)"></el-switch>
          </template>
        </el-table-column>
        <el-table-column  label="操作" width="210px">
          <template v-slot="scope">
            <el-button type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)"></el-button>
            <el-button type="danger" icon="el-icon-delete" @click="removeUserById(scope.row.id)"></el-button>
            <el-tooltip class="item" effect="dark" content="分配角色" placement="top" :enterable="false">
               <el-button type="warning" icon="el-icon-share" @click="setRoles(scope.row)"></el-button>
            </el-tooltip>    
          </template>R
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[1, 2, 5, 10]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
      </el-pagination>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog title="添加用户" :visible.sync="dialogVisible" width="50%" @close="dialogClosed">
      <!-- 主体区域 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item label="用户名" prop="username">
          <el-input v-model="addForm.username"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="password">
          <el-input v-model="addForm.password"></el-input>
        </el-form-item>
        <el-form-item label="邮箱" prop="email">
          <el-input v-model="addForm.email"></el-input>
        </el-form-item>
        <el-form-item label="手机" prop="mobile">
          <el-input v-model="addForm.mobile"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户对话框 -->
    <el-dialog title="修改用户" :visible.sync="editDialogVisible" width="30%" @close="editDialogClosed">
     <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" >
      <el-form-item label="用户名" >
       <el-input v-model="editForm.username" disabled></el-input>
      </el-form-item>
      <el-form-item label="邮箱" prop="email">
       <el-input v-model="editForm.email" ></el-input>
      </el-form-item>
      <el-form-item label="手机号" prop="mobile">
       <el-input v-model="editForm.mobile" ></el-input>
      </el-form-item>
     </el-form>
     <span slot="footer" class="dialog-footer">
       <el-button @click="editDialogVisible = false">取 消</el-button>
       <el-button type="primary" @click="editUserInfo">确 定</el-button>
     </span>
    </el-dialog>
    <!-- 分配角色对话框 -->
    <el-dialog title="分配角色" :visible.sync="rolesDialogVisible" width="50%" @close='rolesDialogClosed'>
      <p>当前的用户:{{userinfo.username}}</p>
      <p>当前的角色:{{userinfo.role_name}}</p>
      <p>
        <el-select v-model="rolesValue" placeholder="请选择">
          <el-option
            v-for="item in rolesList"
            :key="item.id"
            :label="item.roleName"
            :value="item.id">
          </el-option>
        </el-select>
      </p>
      <span slot="footer" class="dialog-footer">
        <el-button @click="rolesDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveRoles">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data(){
    // 验证邮箱的规则
    var checkEmail = (rule,value,cb)=>{
      // 验证邮箱的正则表达式
      const regEmail = /^([a-zA-Z0-9_-])+@([a-zA-Z0-9_-])+(\.[a-zA-Z0-9_-])+/
      if(regEmail.test(value)) {
        return cb()
      }
      cb(new Error('请输入合法的邮箱'))
    }
     // 验证手机号的规则
    var checkMobile = (rule, value, cb) => {
      // 验证手机号的正则表达式
      const regMobile = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/

      if (regMobile.test(value)) {
        return cb()
      }

      cb(new Error('请输入合法的手机号'))
    }

    return{
      queryInfo:{
        query:'',
        pagenum:1,
        pagesize:2
      },
      userinfo:{},
      userList:[],
      total:0,
      //分配角色数据
      rolesList:[],
      // 已选中角色id值
      rolesValue:'',
      // 控制添加用户对话框的显示和隐藏
      dialogVisible:false,
      // 控制修改用户对话框的显示和隐藏
      editDialogVisible:false,
      // 控制分配角色对话框的显示和隐藏
      rolesDialogVisible: false,
      addForm:{
        username:'',
        password:'',
        email:'',
        mobile:'',
      },
      //添加用户表单的验证规则
      addFormRules:{
        username:[
            { required: true, message: '请输入用户名称', trigger: 'blur' },
            { min: 3, max: 5, message: '长度在 3 到 5 个字符', trigger: 'blur' }
        ],
        password:[
            { required: true, message: '请输入用户密码', trigger: 'blur' },
            { min: 6, max: 15, message: '长度在 6 到 15 个字符', trigger: 'blur' }
        ],
        email:[
            { required: true, message: '请输入用户邮箱', trigger: 'blur' },
            { validator:checkEmail, trigger: 'blur' } 
        ],
        mobile:[
            { required: true, message: '请输入用户手机', trigger: 'blur' },
            { validator:checkMobile, trigger: 'blur' }
        ]
      },
      editForm:{},
      //修改用户表单的验证规则
      editFormRules:{
        email:[
            { required: true, message: '请输入用户邮箱', trigger: 'blur' },
            { validator:checkEmail, trigger: 'blur' } 
        ],
        mobile:[
            { required: true, message: '请输入用户手机', trigger: 'blur' },
            { validator:checkMobile, trigger: 'blur' }
        ]
      }
    }
  },
  created(){
    this.getUserList()
  },
  methods:{
    async getUserList(){
      const {data:res} = await this.$http.get('users',{params:this.queryInfo});
      if(res.meta.status !== 200) return this.$message.error('获取用户列表失败！')
      this.userList = res.data.users;
      this.total = res.data.total;
      console.log(res);  
    },
    // 监听pagesize的改变
    handleSizeChange(newSize){
      // console.log(newSize)
      this.queryInfo.pagesize=newSize;
      this.getUserList()
    },
    // 监听页码的改变
    handleCurrentChange(newCurrent){
      // console.log(newCurrent);
      this.queryInfo.pagenum=newCurrent;
      this.getUserList()
    },
    async stateChanged(userinfo){
      console.log(userinfo);
      const {data:res}= await this.$http.put(`users/${userinfo.id}/state/${userinfo.mg_state}`);
      if(res.meta.status !==200){
        userinfo.mg_state =!userinfo.mg_state
        return this.$message.error('更新用户状态失败！')
      }
      this.$message.success('更新用户状态成功！')
    },
    //监听添加用户对话框的关闭
    dialogClosed(){
      this.$refs.addFormRef.resetFields()
    },
    //监听添加用户提交
    addUser(){
      this.$refs.addFormRef.validate(async vaild => {
        if(!vaild) return
        const {data:res} = await this.$http.post('users',this.addForm)
        if(res.meta.status !==201){
          return this.$message.error('添加用户失败！')
        }
        this.$message.success('添加用户成功！')
        //隐藏对话框
        this.dialogVisible = false
        //重新请求用户数据
        this.getUserList()
      })
    },
    //展示修改编辑用户的对话框
    async showEditDialog(id){
      this.editDialogVisible = true
      const {data:res} = await this.$http.get('users/' + id)
      if(res.meta.status !== 200){
        return this.$message.error('查询用户信息失败！')
      }
      this.editForm = res.data
    },
    //监听修改用户对话框的关闭
    editDialogClosed(){
      this.$refs.editFormRef.resetFields()
    },
    //修改用户信息并提交
    editUserInfo(){
      this.$refs.editFormRef.validate(async valid =>{
        if(!valid) return
        //发起修改用户信息的数据请求
        const {data:res} = await this.$http.put('users/'+ this.editForm.id,{
          email:this.editForm.email,
          mobile:this.editForm.mobile
        })
        if(res.meta.status !==200){
          return this.$message.error('更新数据失败')
        }
        //关闭对话框
        this.editDialogVisible = false
        //刷新数据列表
        this.getUserList()
        //提示修改数据成功
        this.$message.success('更新数据成功')
      })
    },
    //根据用户ID删除用户信息
    async removeUserById(id){
      const confirmResult = await this.$confirm('此操作将永久删除该用户, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).catch(err => err)
      //如果用户确认删除 返回字符串 confirm
      //如果用户取消删除 返回字符串 concel
      // console.log(confirmResult);
      if(confirmResult !== 'confirm'){
        return this.$message.info('已取消删除')
      }
      const {data:res}= await this.$http.delete('users/' + id)
      if(res.meta.status !== 200){
        return this.$message.error('删除用户失败！')
      }
      this.$message.success('删除用户成功！')
      this.getUserList()
    },
    //展示分配角色对话框
    async setRoles(user){
      this.userinfo = user
      
      const {data:res} = await this.$http.get('roles')
      if(res.meta.status !==200){
        return this.$message.error('获取角色列表失败！')
      }
      console.log(res.data);
      this.rolesList = res.data
      this.rolesDialogVisible = true
    },
    // 保存提交分配角色数据
    async saveRoles(){
      if(!this.rolesValue){
        return this.$message.error('请分配角色！')
      }
      const {data:res} = await this.$http.put(`users/${this.userinfo.id}/role`,{rid:this.rolesValue})
      if(res.meta.status !== 200){
        return this.$message.error('分配角色失败！')
      }
      this.$message.success('分配角色成功！')
      this.getUserList()
      this.rolesDialogVisible = false

    },
    //监听分配角色对话框的关闭
    rolesDialogClosed(){
      this.rolesValue='',
      this.rolesList=[]
    } 
  }
}
</script>

<style lang="less" scoped>
  .el-table {
    margin-top: 15px;
    font-size: 14px;
  }
  .el-pagination {
    margin-top: 15px;
  }
</style>