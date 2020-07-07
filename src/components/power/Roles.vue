<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加角色按钮 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="dialogVisible = true">添加角色</el-button>
        </el-col>
      </el-row>
      <!-- 角色列表 -->
      <el-table :data="rolesList" border stripe>
        <el-table-column type="expand">
          <template v-slot="scope">
            <!-- 渲染一级权限 -->
            <el-row v-for="(item1,index1) in scope.row.children" :key="item1.idid" :class="['bdbottom',index1 !== 0 ? '': 'bdtop', 'vcenter']">
              <el-col :span="5">
                <el-tag closable @close="removeRightById(scope.row,item1.id)">{{item1.authName}}</el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <el-col :span="19">
                <!-- 渲染二级和三级权限 -->
                <el-row v-for="(item2,index2) in item1.children" :key="item2.id" :class="['bdbottom',index2 === 0 ? '':'bdtop','vcenter']">
                  <el-col :span="8">
                    <el-tag type="success" closable @close="removeRightById(scope.row,item2.id)">{{item2.authName}}</el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="16">
                    <el-tag v-for="(item3,index3) in item2.children" :key="item3.id" type="warning" closable @close="removeRightById(scope.row,item3.id)">{{item3.authName}}</el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <el-table-column type="index" label="#"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template v-slot="scope">
            <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.id)">编辑</el-button>
            <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeRoleById(scope.row.id)">删除</el-button>
            <el-button size="mini" type="warning" icon="el-icon-setting" @click="showRightsDialog(scope.row)">分配权限</el-button>
          </template>
        </el-table-column>
      </el-table>
    </el-card>
    <!-- 添加角色对话框 -->
    <el-dialog title="添加角色"  :visible.sync="dialogVisible" width="50%" @close="addFormClosed">
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改角色对话框 -->
    <el-dialog title="修改角色"  :visible.sync="editDialogVisible" width="50%" @close="editFormClosed">
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px">
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRoles">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog title="分配权限" :visible.sync="rightsDialogVisible" width="50%" @close="rightsDialogclosed">
      <!-- 树组件 -->
      <el-tree :data="rightsList" :props="treeProps" show-checkbox node-key="id" default-expand-all	:default-checked-keys="defKeys" ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="rightsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog> 
  </div>
</template>

<script>
export default {
  data(){
    return {
      //角色列表
      rolesList:[],
      //权限列表
      rightsList:[],
      // 默认选中的节点Id值数组
      defKeys:[],
      roleId :'',
      dialogVisible: false,
      editDialogVisible: false,
      rightsDialogVisible: false,
      editForm:{},
      addForm:{
        roleName:'',
        roleDesc:''
      },
      //权限树型控件的属性绑定对象
      treeProps:{
        label:'authName',
        children:'children'
      },
      //添加角色表单的验证规则
      addFormRules:{
        roleName:[
          {required:true,message:'请输入角色名称',trigger:'blur'},
          {min:2,max:10,message:'长度在2到10个字符',trigger:'blur'}
        ],
        roleDesc:[
          {required:true,message:'请输入角色描述',trigger:'blur'},
          {min:2,max:10,message:'长度在2到10个字符',trigger:'blur'}
        ]
      },
      //修改角色表单的验证规则
      editFormRules:{
        roleName:[
          {required:true,message:'请输入角色名称',trigger:'blur'},
          {min:2,max:10,message:'长度在2到10个字符',trigger:'blur'}
        ],
        roleDesc:[
          {required:true,message:'请输入角色描述',trigger:'blur'},
          {min:2,max:10,message:'长度在2到10个字符',trigger:'blur'}
        ]
      }
    }
  },
  created(){
    // 获取角色列表数据
    this.getRolesList()
  },
  methods:{
    //// 获取角色列表数据
    async getRolesList(){
      const {data:res} = await this.$http.get('roles')
      if(res.meta.status !==200){
        return this.$message.error('获取角色列表失败！')
      }
      this.rolesList = res.data
      // console.log(this.rolesList);
      
    },
    //添加角色信息并提交数据
    addRoles(){
      this.$refs.addFormRef.validate(async valid =>{
        if(!valid) return
        const {data:res} = await this.$http.post('roles',this.addForm)
        if(res.meta.status !== 201){
          return this.$message.error('添加角色失败！')
        }
        this.$message.success('添加角色成功！')
        //隐藏对话框
        this.dialogVisible = false
        //重新获取角色数据
        this.getRolesList()
      })
    },
    //监听添加角色对话框关闭
    addFormClosed(){
      this.$refs.addFormRef.resetFields()
    },
    //展示修改角色数据对话框
    async showEditDialog(id){
      this.editDialogVisible = true
      const {data:res} = await this.$http.get('roles/'+ id)
      if(res.meta.status !==200){
        return this.$message.error('查询角色信息失败！')
      }
      this.editForm = res.data
    },
     //监听修改角色对话框关闭
    editFormClosed(){
      this.$refs.editFormRef.resetFields()
    },
    //修改角色数据并提交
    editRoles(){
      this.$refs.editFormRef.validate(async valid =>{
        if(!valid) return
        const {data:res} = await this.$http.put('roles/'+ this.editForm.roleId,{
          roleName:this.editForm.roleName,
          roleDesc:this.editForm.roleDesc
        })
        if(res.meta.status !==200){
          console.log(res.meta.status);
          
          return this.$message.error('修改用户信息失败！')
        }
        //关闭对话框
        this.editDialogVisible = false
        //重新获取角色数据
        this.getRolesList()
        //提示修改数据成功
        this.$message.success('修改用户信息成功！')
      })
    },
    //根据角色ID删除用户信息
    async removeRoleById(id){
      const confirmResult = await this.$confirm('此操作将永久删除该角色, 是否继续?', '提示', {
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
      const {data:res}= await this.$http.delete('roles/' + id)
      if(res.meta.status !== 200){
        return this.$message.error('删除角色失败！')
      }
      this.$message.success('删除角色成功！')
      this.getRolesList()
    },
    // 根据id删除权限
    async removeRightById(role,right){
      const confirmResult = await this.$confirm('此操作将永久删除该权限, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
      }).catch(err => err)
      if(confirmResult !== 'confirm'){
        return this.$message.info('已取消删除')
      }
      const {data:res} = await this.$http.delete(`roles/${role.id}/rights/${right}`)
      if(res.meta.status !== 200){
        return this.$message.error('删除权限失败!')
      }
      role.children = res.data
    },
    //展示分配权限的对话框
     async showRightsDialog(role){
       //把当前角色id赋值给roleId
       this.roleId = role.id
      // 获取权限列表数据
      const {data:res} = await this.$http.get('rights/tree')
      if(res.meta.status !==200){
        return this.$message.error('获取权限列表失败!')
      }
      // console.log(res);
      this.rightsList = res.data
      this.getKeys(role,this.defKeys)
      
      this.rightsDialogVisible = true
    },
    //运用递归函数,获取角色下所有三级权限的id,并保存到defKeys中
    getKeys(node,arr){
        if(!node.children){
          return arr.push(node.id)
        }
        node.children.forEach(item => 
          this.getKeys(item,arr)
        );
    },
    //监听分配权限对话框的关闭
    rightsDialogclosed(){
      this.defKeys =[]
    },
    // 给角色分配权限
    async allotRights(){
      const keyArr =[
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ] 
      const idStr = keyArr.join(',')
      console.log(idStr)
      const {data:res} = await this.$http.post(`roles/${this.roleId}/rights`,{rids:idStr})
      if(res.meta.status !==200){
        return this.$message.error('分配权限失败!')
      }
      // 提示分配成功
      this.$message.success('分配权限成功!')
      // 重新获取角色数据
      this.getRolesList()
      // 关闭对话框
      this.rightsDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
  .bdtop {
    border-top: 1px solid #eee;
  }
  .bdbottom {
    border-bottom: 1px solid #eee;
  }
  .el-tag{
    margin: 7px;
  }
  .vcenter {
    display: flex;
    align-items: center;
  }
</style>