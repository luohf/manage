<template>
  <div>
    <!-- 导航区域 -->
  <el-breadcrumb separator-class="el-icon-arrow-right">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>商品管理</el-breadcrumb-item>
  <el-breadcrumb-item>商品分类</el-breadcrumb-item>
  </el-breadcrumb>

  <!-- 卡片区域 -->
  <el-card>
    <el-row>
      <el-col>
      <el-button type="primary" @click="showAddCateDialog">添加分类</el-button>
      </el-col>
    </el-row>
    <!-- 表格 -->
    <tree-table :data="catelist" :columns="columns" :selection-type="false" :expand-type="false" show-index index-text='#' class="treeTable" border>

<!-- 是否有效的那一列 -->
      <template slot="ifok" slot-scope="scope">
        <i class="el-icon-check" v-if="scope.row.cat_deleted===false" style="color:green;"></i>
        <i class="el-icon-close" v-else style="color:red;"></i>
      </template>

      <!-- 排序 -->
      <template slot="order" slot-scope="scope">
<el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
<el-tag type="success" size="mini" v-else-if="scope.row.cat_level === 1">二级</el-tag>
<el-tag type="warning" size="mini"
v-else>三级</el-tag>
      </template>

<!-- 操作 -->
      <template slot="opt" slot-scope="scope">
<el-button type="primary" icon="el-icon-edit" size="mini">搜索</el-button>
<el-button type="danger" icon="el-icon-delete" size="mini">搜索</el-button>
      </template>


    </tree-table>
    <!-- 分页区域 -->
<el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[3, 5, 10, 15]"
      :page-size=" queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>

  </el-card>

  <!-- 添加分类的弹出框 -->
  <el-dialog
  title="添加分类" @close="addCateDialogClosed"
  :visible.sync="addCateDialogVisible"
  width="50%"
  >

<!-- 添加分类的表单 -->
  <el-form :model="addCateForm" :rules="addCateFormRules" ref="addCateFormRef" label-width="100px" >
  <el-form-item label="分类" prop="cat_name">
    <el-input v-model="addCateForm.cat_name"></el-input>
  </el-form-item>
  <el-form-item label="父级分类" >

    <!-- options用来指定数据源 -->
<!-- props用来指定配置对象 -->
  <el-cascader
    v-model="selectedKeys" 
    :options="parentCateList"
    :props="cascaderProps" 
    clearable 
    @change="parentCateChanged"
    change-on-select >
    </el-cascader>
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
 data() {
   return {
     //查询条件
     queryInfo: {
       type:3,
       pagenum :1,
       pagesize:5,

       
     },
    // 商品分类的数据列表
     catelist:[],
    // 总数据条数
     total:0,
    //  table的列
     columns:[
       {
         label: '分类名称',
         prop:'cat_name'
       },
       {
         label:'是否有效',
// 表示，将当前列定义为模板列
         type:'template',
        //  当前这一列使用的模板名
         template: 'ifok'

       },
      //  排序
        {
         label:'排序',
// 表示，将当前列定义为模板列
         type:'template',
        //  当前这一列使用的模板名
         template: 'order'

       },
        {
         label:'操作',
// 表示，将当前列定义为模板列
         type:'template',
        //  当前这一列使用的模板名
         template: 'opt'

       }

     ],
// 控制添加分类的要不要显现
addCateDialogVisible:false,
// 添加分类
addCateForm:{
  cat_name:'',
  // 分类父id
  cat_pid:0,
// 分类的等级，默认要添加1级分类
  cat_level:0
},
// 添加分类的表单验证规则
addCateFormRules:{
  cat_name: [
    {required:true,message:'请输入分类',trigger:'blur'}
  ]
},
// 父级分类的数据列表
parentCateList:[],
// 指定级联选择器的配置对象
cascaderProps: {
  expandTrigger: 'hover',
  value:'cat_id',
  label:'cat_name',
  children:'children'
},
// 选中的父级分类的Id数组
// 保存一个一级分类和一个二级分类的id数组，
// 一级分类和二级分类是父子级
selectedKeys:[]
   }
 }, 
 created() {
   this.getCateList()
 },
 methods: {
  //  获取商品分类的数据
   async getCateList(){
     const { data:res } = await this.$http.get('categories',{params:this.queryInfo})
     if(res.meta.status !== 200){
       return this.$message.error('获取商品分类失败')
     }
     console.log(res)
     this.catelist = res.data.result
    //  总数据条数  
     this.total = res.data.total
   },
   handleSizeChange(newSize) {
    
     this.queryInfo.pagesize = newSize
     //  根据每页的条数去查询,因为下面的查询要用到，下面的一步不能少
     this.getCateList()
   },

    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      // 根据哪页去查询，因为下面的查询要用到，下面的一步不能少
      this.getCateList()
    },
 
//  添加分类
 showAddCateDialog(){
  //  先获取父级分类的数据列表
   this.getParentCateList()
// 再展现弹出框
   this.addCateDialogVisible = true
 },
 
//  获取父级分类的数据列表
async getParentCateList() {
  const { data:res } = await this.$http.get('categories',{
    // 注意这个写法
    params:{type:2}
  })
  if(res.meta.status !== 200) {
    return this.$message.error('获取父级分类数据失败')
  }
  console.log("3")
  console.log(res)

  this.parentCateList = res.data
  

},
// 选择项发生变化时触发这个函数
parentCateChanged() {
  console.log("8")
  console.log(this.selectedKeys)
  // 如果selectedKeys数组中的length大于0，证明选中的父级分类
  // 反之，就说明没有选中任何父级分类
  if(this.selectedKeys.length > 0) {
    // 父级分类id
    this.addCateForm.cat_pid = this.selectedKeys[this.selectedKeys.length - 1
    ]
    // 为当前分类的等级赋值
    this.addCateForm.cat_level = this.selectedKeys.length
    return
  } else {
    // 父级分类的id
    this.addCateForm.cat_pid = 0
    // 为当前分类的等级赋值
     this.addCateForm.cat_level = 0
  }
},
// 添加分类
addCate() {
  console.log(this.addCateForm)
  this.$refs.addCateFormRef.validate(async valid => {
    if(!valid) return
    const { data: res } = await this.$http.post('categories',
    this.addCateForm)

    if(res.meta.status !== 201) {
      return this.$message.error('添加分类失败')
      }
      this.$message.success('添加分类成功')
      this.getCateList()
      this.addCateDialogVisible = false
    

    
  })

},

// 监听弹出框的关闭
addCateDialogClosed() {
  this.$refs.addCateFormRef.resetFields()
  this.selectedKeys = []
  this.addCateForm.cat_level = 0
  this.addCateForm.cat_pid = 0
}

 },
}
</script>

<style lang="less" scoped>
.treeTable{
  margin-top: 15px;
}
.el-cascader{
  width: 100%;
}
</style>