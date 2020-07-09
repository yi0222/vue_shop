<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 警示区域 -->
      <el-alert title="注意！只允许为第三级分类设置相关参数！" type="warning" :closable="false" show-icon></el-alert>
      <!-- 选择商品分类区域 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader
          v-model="cateKeys"
          :options="cateList"
          :props="cascaderProps"
          @change="handleChange"></el-cascader>
        </el-col>
      </el-row>
      <!-- tabs标签页 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
        <el-tab-pane label="动态参数" name="many">
          <!-- 添加参数按钮 -->
          <el-button type="primary" :disabled="isCatechanged" @click="addParamsDialogVisible= true">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableList" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,index) in scope.row.attr_vals" :key="index" closable @close="handleClosed(index,scope.row)">{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column prop="attr_name" label="参数名称" ></el-table-column>
            <el-table-column  label="操作" >
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" @click="showEditForm(scope.row.attr_id)">修改</el-button>
                <el-button type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
        <el-tab-pane label="静态属性" name="only">
          <!-- 添加属性按钮 -->
          <el-button type="primary" :disabled="isCatechanged" @click="addParamsDialogVisible= true">添加属性</el-button>
          <!-- 静态态参数表格 -->
          <el-table :data="onlyTableList" border stripe>
            <el-table-column type="expand">
              <template slot-scope="scope">
                <el-tag v-for="(item,index) in scope.row.attr_vals" :key="index" closable>{{item}}</el-tag>
                <el-input
                  class="input-new-tag"
                  v-if="scope.row.inputVisible"
                  v-model="scope.row.inputValue"
                  ref="saveTagInput"
                  size="small"
                  @keyup.enter.native="handleInputConfirm(scope.row)"
                  @blur="handleInputConfirm(scope.row)">
                </el-input>
                <el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>
              </template>
            </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column prop="attr_name" label="属性名称" ></el-table-column>
            <el-table-column  label="操作" >
              <template slot-scope="scope">
                <el-button type="primary" icon="el-icon-edit" @click="showEditForm(scope.row.attr_id)">修改</el-button>
                <el-button type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
              </template>
            </el-table-column>
          </el-table>
        </el-tab-pane>
      </el-tabs>
    </el-card>
    <!-- 添加参数对话框 -->
    <el-dialog :title="'添加'+ ParamsName" :visible.sync="addParamsDialogVisible" width="50%" @close="addParamsDialogClosed">
      <!-- form表单 -->
      <el-form :model="addParamsForm" :rules="addParamsFormRules" ref="addParamsFormRef" label-width="100px">
        <el-form-item :label="ParamsName" prop="attr_name">
          <el-input v-model="addParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>  
      <span slot="footer" class="dialog-footer">
        <el-button @click="addParamsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改参数对话框 -->
    <el-dialog :title="'修改'+ ParamsName" :visible.sync="editParamsDialogVisible" width="50%" @close="editParamsDialogClosed">
      <!-- form表单 -->
      <el-form :model="editParamsForm" :rules="editParamsFormRules" ref="editParamsFormRef" label-width="100px">
        <el-form-item :label="ParamsName" prop="attr_name">
          <el-input v-model="editParamsForm.attr_name"></el-input>
        </el-form-item>
      </el-form>  
      <span slot="footer" class="dialog-footer">
        <el-button @click="editParamsDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data(){
    return{
      // 商品分类数据列表
      cateList:[],
      // 级联选择器的分类配置对象
      cascaderProps:{
        expandTrigger: 'hover',
        value:'cat_id',
        label:'cat_name',
        children:'children'
      },
      // 级联选择器双向绑定的id数组
      cateKeys:[],
      // tabs标签页默认
      activeName:'many',
      // 动态参数数据列表
      manyTableList:[],
      // 静态属性数据列表
      onlyTableList:[],
      // 控制添加参数对话框的展示和隐藏
      addParamsDialogVisible : false,
       // 控制修改参数对话框的展示和隐藏
      editParamsDialogVisible : false,
      // 添加参数表单数据对象
      addParamsForm:{
        attr_name:''
      },
      // 修改参数表单数据对象
      editParamsForm:{},
      // 添加参数表单的验证规则
      addParamsFormRules:{
        attr_name:[{
          required:true,message:'请输入参数名称',trigger:'blur'
        }]
      },
      editParamsFormRules:{
         attr_name:[{
          required:true,message:'请输入参数名称',trigger:'blur'
        }]
      }

    }
  },
  created(){
    this.getCateList()
  },
  methods:{
    // 获取商品分类数据列表
  async getCateList(){
      const {data:res} = await this.$http.get('categories')
      if(res.meta.status !== 200){
        return this.$message.error('获取商品分类数据失败！')
      }
      this.cateList = res.data
      
      
    },
    // 监听级联选择器的改变
    async handleChange(){
      this.getParamsList()
  
    },
    handleTabClick(){
      console.log(this.activeName);
      this.getParamsList()
    },
    // 获取参数列表数据
    async getParamsList(){
      // 判断选择是否为三级分类
      if(this.cateKeys.length !==3){
        this.cateKeys =[]
        this.manyTableList = []
        this.onlyTableList = []
        return
      }
      // console.log(this.cateKeys);
      const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:this.activeName}})
      if(res.meta.status !==200){
        return this.$message.error('获取参数失败！')
      }
      res.data.forEach(item =>{
        item.attr_vals = item.attr_vals ? item.attr_vals.split(',') : []
        item.inputVisible = false
        item.inputValue = ''

      })
      console.log(res.data);
      // 判断获取数据动态还是静态
      if(this.activeName ==='many'){
        this.manyTableList = res.data
      }else{
        this.onlyTableList = res.data
      }
    },
    // 监听添加参数对话框的关闭
    addParamsDialogClosed(){
      this.$refs.addParamsFormRef.resetFields()
    },
    // 监听修改参数对话框的关闭
    editParamsDialogClosed(){
      this.$refs.editParamsFormRef.resetFields()
    },
    // 点击按钮 添加参数
    addParams(){
      this.$refs.addParamsFormRef.validate(async valid =>{
        if(!valid) return
        const {data:res} = await this.$http.post(`categories/${this.cateId}/attributes`,{
          attr_name:this.addParamsForm.attr_name,
          attr_sel:this.activeName
        })
        if(res.meta.status !== 201){
          return this.$message.error('添加参数失败！')
        }
        this.$message.success('添加参数成功！')
        this.getParamsList()
        this.addParamsDialogVisible = false
      })
    },
    // 点击按钮 修改参数
    editParams(){
      this.$refs.editParamsFormRef.validate(async valid =>{
        if(!valid) return
        const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editParamsForm.attr_id}`,
        {attr_name: this.editParamsForm.attr_name, attr_sel: this.activeName}
        )
        if(res.meta.status !== 200){
          return this.$message.error('修改参数失败！')
        }
        this.$message.success('修改参数成功！')
        this.getParamsList()
        this.editParamsDialogVisible =false
      })
          
          
      
    },
    // 点击按钮展示修改参数对话框
    async showEditForm(attr_id){
      const {data:res} =await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,{params:{attr_sel:this.activeName}})
      if(res.meta.status !==200){
        return this.$message.error('获取参数失败！')
      }
      // console.log(res);
      
      this.editParamsForm = res.data
      this.editParamsDialogVisible =true
    },
    // 点击按钮删除参数
    async removeParams(attr_id){
      const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err=>err)
      if(confirmResult !== 'confirm'){
        return this.$message.info('删除参数失败！')
      }
      const {data:res} = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
      if(res.meta.status !==200){
        return this.$message.error('删除参数失败！')
      }
      this.$message.success('删除参数成功！')
      this.getParamsList()
    },
    // 控制文本框和tag标签的切换
    showInput(row){
      row.inputVisible =true
       this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        })
    },
    async handleInputConfirm(row){
      if(row.inputValue.trim().length ===0){
        row.inputValue =''
        row.inputVisible = false
        return
      }
      // 没有return证明输入了内容
      row.attr_vals.push(row.inputValue)
      row.inputValue = ''
      row.inputVisible = false
      // 发起请求保存
      this.saveAttrVals(row)
      
      
    },
    //发起请求保存参数值
    async saveAttrVals(row){
      const{data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${row.attr_id}`,
      {attr_name:row.attr_name, attr_sel:row.attr_sel , attr_vals:row.attr_vals.join(',')})
      if(res.meta.status !== 200){
        console.log(res);
        
        return this.$message.error('参数修改失败！')
      }
      this.$message.success('参数修改成功！')
    },
    handleClosed(i ,row){
      row.attr_vals.splice(i,1)
      this.saveAttrVals(row)
    }
  },
  computed:{
    isCatechanged(){
      if(this.cateKeys.length !== 3){
        return true
      }else{
        return false
      }
      
    },
    // 当前选中的三级分类的id
    cateId(){
      if(this.cateKeys.length ===3){
        return this.cateKeys[2]
      }else{
        return null
      }
    },
    ParamsName(){
      if(this.activeName === 'many'){
        return '动态参数'
      }else{
        return '静态属性'
      }
    }
  }
}
</script>

<style lang="less" scoped>
  .cat_opt{
    margin: 15px 0;
  }
  .input-new-tag{
    width: 100px;
  }
  .el-tag {
    margin: 0 5px;
  }
</style>