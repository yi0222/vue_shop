<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/welcome' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- table表格 -->
      <tree-table :data="cateList" :columns="columns" 
      :selection-type="false" :expand-type="false" 
      show-index index-text="#" border>
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
            <i class="el-icon-success" v-if="scope.row.cat_deleted ===false" style="color:lightgreen"></i>
            <i class="el-icon-error" v-else style="color:red"></i>
        </template>
        <!-- 排序等级 -->
        <template slot="order" slot-scope="scope">
            <el-tag v-if="scope.row.cat_level === 0">一级</el-tag>
            <el-tag type="success" v-else-if="scope.row.cat_level === 1">二级</el-tag>
            <el-tag type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
            <el-button type="primary" icon="el-icon-edit" size="mini">编辑</el-button>
            <el-button type="danger" icon="el-icon-delete" size="mini">删除</el-button>
        </template>
      </tree-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[3, 5, 10, 30]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total">
      </el-pagination> 
    </el-card>
    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCateDialogVisible" width="50%" @close="addCateDialogclosed">
      <!-- 表单 -->
      <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px" class="demo-ruleForm">
        <el-form-item label="分类名称：" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类：">
          <el-cascader v-model="selectedKeys" :options="parentCateLit" :props="cascaderProps" @change="parentCateChanged" clearable></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate">确 定</el-button>
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
      // 总商品分类数据条数
      total:0,
      // 为table指定列的定义
      columns:[
        {
          label:'分类名称',
          prop:'cat_name'
        },
        {
          label:'是否有效',
          // 指定这一列为自定义模板列
          type:'template',
          // 自定义模板名称
          template:'isok'
        },
        {
          label:'排序',
          // 指定这一列为自定义模板列
          type:'template',
          // 自定义模板名称
          template:'order'
        },
        {
          label:'操作',
          // 指定这一列为自定义模板列
          type:'template',
          // 自定义模板名称
          template:'opt'
        }
      ],
      // 商品分类数据请求参数
      queryInfo:{
        type:3,
        pagenum:1,
        pagesize:5
      },
      // 控制添加分类对话框的显示和隐藏
      addCateDialogVisible:false,
      // 添加分类表单的数据对象
      addCateForm:{
        // 添加的分类名称
        cat_name:'',
        // 父级分类Id
        cat_pid:0,
        // 分类等级级，默认是一级
        cat_level:0
      },
      // 分类表单的验证规则
      addCateFormRules:{
        cat_name:[
          {required:true, message:'请输入分类名称', trigger:'blur'}
        ]
      },
      // 父级分类数据列表
      parentCateLit:[],
      // 级联选择器的配置对象
      cascaderProps:{
        expandTrigger: 'hover',
        value:'cat_id',
        label:'cat_name',
        children:'children',
        checkStrictly	: true

      },
      // 级联选择器选中父级分类的id数组
      selectedKeys:[]
    }
  },
  created(){
    this.getCateList()
  },
  methods:{
    // 获取商品分类数据列表
    async getCateList(){
      const {data:res} = await this.$http.get('categories',{params:this.queryInfo})
      if(res.meta.status !==200){
        return this.$message.error('获取商品分类数据列表失败！')
      }
      console.log(res);
      // 把数据列表赋值给cateList
      this.cateList = res.data.result
      // 把数据列表总条数赋值给total
      this.total = res.data.total
    },
    // 监听pagesize改变
    handleSizeChange(newSize){
      this.queryInfo.pagesize = newSize
      this.getCateList()  
    },
    // 监听pagenum改变
    handleCurrentChange(newNum){
      this.queryInfo.pagenum = newNum
      this.getCateList()  
    },
    // 点击添加按钮显示添加分类对话框
    showAddCateDialog(){
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类数据列表
    async getParentCateList(){
      const {data:res} = await this.$http.get('categories',{params:{type:2}})
      if(res.meta.status !== 200){
        return this.$message.error('获取父级分类数据列表失败！')

      }
      // console.log(res);
      this.parentCateLit = res.data
      
    },
    //监听级联选择器发生变化
    parentCateChanged(){
      if(this.selectedKeys.length > 0){
        this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length -1]
        this.addCateForm.cat_level = this.selectedKeys.length
        return
      }else{
        this.addCateForm.cat_pid = 0,
        this.addCateForm.cat_level =0
      }
    },
    // 点击按钮添加新的分类
    addCate(){
      this.$refs.addCateFormRef.validate(async valid=>{
        if(!valid) return
        const {data:res} = await this.$http.post('categories',this.addCateForm)
        if(res.meta.status !==201){
          return this.$message.error('添加分类失败！')
        }
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible =false
      })
      
    },
    // 监听添加分类对话框的关闭
    addCateDialogclosed(){
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys =[]
      this.addCateForm.cat_level =0
      this.addCateForm.cat_pid =0

    }
  }
}
</script>

<style lang="less" scoped>
  .el-cascader{
    width: 100%;
  }
</style>