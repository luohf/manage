<template>
  <div>
  <el-breadcrumb separator="/">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>商品管理</el-breadcrumb-item>
  <el-breadcrumb-item>参数列表</el-breadcrumb-item>
  </el-breadcrumb>
<!-- 卡片 -->
<el-card>
  <!-- 警告 -->
   <el-alert
    title="注意：只允许为第三级分类设置参数"
    type="warning" :closable ="false" show-icon>
  </el-alert>

<!-- 选择商品分类 -->
<el-row class="cat_opt">
  <el-col>
    <span >选择商品分类：</span>
    <!-- 选择商品分类的级联选择器 -->
    <el-cascader
    v-model="selectedCateKeys"
    :options="catelist"
    :props="cateProps"
    @change="handleChange"></el-cascader>
  </el-col>
</el-row>
 
 <!-- tab页签区域 -->
<el-tabs v-model="activeName" @tab-click="handleTabClick">
    <el-tab-pane label="动态参数" name="many">
      <el-button type="primary" size="mini" :disabled=" isBtnDisabled" @click ="addDialogVisible=true">添加参数</el-button>
      <!-- 动态参数表 -->
      <el-table :data="manyTableData" border stripe>
        <!-- 展开行 -->
        <el-table-column type="expand" >
          <template slot-scope="scope">
            <!-- 循环渲染tag标签 -->
            <el-tag v-for = "(item,i) in scope.row.attr_vals" :key="i" type="warning" closable>{{item}}</el-tag>
<!-- 输入的文本框 -->
<el-input
class="input-new-tag"
v-if="scope.row.inputVisible"
v-model="scope.row.inputValue"
ref="saveTagInput"
size="small"
@keyup.enter.native="handleInputConfirm"
@blur="handleInputConfirm"
>
</el-input>
<!-- 添加按钮 -->
<el-button v-else class="button-new-tag" size="small" @click="showInput(scope.row)">+ New Tag</el-button>  
          </template>

        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="参数" prop="attr_name"></el-table-column>
         <el-table-column label="操作" >
           <template slot-scope="scope">

    <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>

 <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
           </template>
         </el-table-column>
      </el-table>
    </el-tab-pane>
    <!-- 添加静态属性的面板 -->
    <el-tab-pane label="静态属性" name="only">
      <!-- 添加属性 -->
       <el-button type="primary" size="mini" :disabled=" isBtnDisabled" @click ="addDialogVisible=true">添加属性</el-button>

       <!-- 静态属性表 -->
       
 <el-table :data="onlyTableData" border stripe>
        <!-- 展开行 -->
        <el-table-column type="expand" ></el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="静态属性" prop="attr_name"></el-table-column>
         <el-table-column label="操作" >
           <template slot-scope="scope">

    <el-button size="mini" type="primary" icon="el-icon-edit" @click="showEditDialog(scope.row.attr_id)">编辑</el-button>

 <el-button size="mini" type="danger" icon="el-icon-delete" @click="removeParams(scope.row.attr_id)">删除</el-button>
           </template>
         </el-table-column>
      </el-table>

    </el-tab-pane>
    
   
  </el-tabs>


<!-- 添加参数的弹出框 -->
<el-dialog
  :title="'添加' + titleText"
  :visible.sync="addDialogVisible"
  width="50%" @close="addDialogClosed"
  >
  <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" >
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="addForm.attr_name"></el-input>
  </el-form-item>
  </el-form>
  
  <span slot="footer" class="dialog-footer">
    <el-button @click="addDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="addParams">确 定</el-button>
  </span>
</el-dialog>

<!-- 修改参数的弹出框 -->
<el-dialog
  :title="'修改' + titleText"
  :visible.sync="editDialogVisible"
  width="50%" @close="editDialogClosed"
  >
  <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" >
  <el-form-item :label="titleText" prop="attr_name">
    <el-input v-model="editForm.attr_name"></el-input>
  </el-form-item>
  </el-form>
  
  <span slot="footer" class="dialog-footer">
    <el-button @click="editDialogVisible = false">取 消</el-button>
    <el-button type="primary" @click="editParams">确 定</el-button>
  </span>
</el-dialog>
</el-card>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 商品分类列表
      catelist:[],
      // 级联选择器的配置
      cateProps:{ 
        expandTrigger: 'hover' ,
        value:'cat_id',
        label:'cat_name',
        children:'children'
        
      },
      //  级联选择框双向绑定到的数组
      selectedCateKeys:[],
      // 被激活的页签
      activeName:'many',
      // 动态参数的数据
      manyTableData:[],
// 静态属性的数据
      onlyTableData:[],
      // 控制添加参数弹出框的显现与隐藏
      addDialogVisible:false,
      // 添加参数的表单数据
      addForm:{
        attr_name:''
      },
// 添加参数的验证规则
      addFormRules: {
         attr_name: [
           {required: true,message: '请输入参数',trigger:'blur'}
         ]
      },

// 展现要不要修改参数的弹出框
editDialogVisible:false,
// 修改表单的数据对象 
editForm: {},
// 修改表单的验证规则对象 
editFormRules:{
         attr_name: [
           {required: true,message: '请输入参数',trigger:'blur'}
         ]
      },

inputVisible:false,
inputValue:''
    }
  
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取所有的商品分类列表
    async getCateList() {
      const { data:res } = await this.$http.get('categories')  
      if(res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败')
      }
      this.catelist = res.data
      console.log(this.catelist)
    },
    // 级联选择框选中项变化，会触发这个函数
     handleChange() {
     this.getParamsData()
    },
    // tab页签点击事件的处理函数
    handleTabClick() {
      console.log(this.activeName)
       this.getParamsData()
    },
    async getParamsData() {
       console.log(this.selectedCateKeys)
      // 证明选中的不是三级分类
      if(this.selectedCateKeys.length !== 3) {
        this.selectedCateKeys = []
        return 
      }
      // 证明选中的是三级分类
      console.log(this.selectedCateKeys)
      // 根据所选分类的id,和当前所处的面板，获取对应的参数
      const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{params:{sel:this.activeName}})
      if(res.meta.status !== 200){
        return this.$message.error('获取参数列表失败')
      }

      console.log(res)
      if(this.activeName === 'many') {
        this.manyTableData = res.data

      }else {
        this.onlyTableData = res.data
      }
      
      res.data.forEach(item => {item.attr_vals = item.attr_vals ? item.attr_vals.split(' ') : [] 
      
        // 添加控制文本框
        // 控制文本框的显现与隐藏
        item.inputVisible = false

        // 添加文本框
        item.inputValue = ''
      
      }
      
      
    
      
      
      
      )


      // console.log(res.data)
      // 分割字符串，变成数组
      // res.data.forEach(item => {item.attr_vals !== ''?
      //   item.attr_vals = item.attr_vals.split(' '):[]
// 添加控制文本框
        // 控制文本框的显现与隐藏
        // item.inputVisible = false

        // 添加文本框
        // item.inputValue = ''
      // })
    },
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 添加参数
    addParams() {
      // 预校验
      this.$refs.addFormRef.validate( async valid => {
        if(!valid) return
        const {data:res} = await this.$http.post(`categories/${this.cateId}/attributes`,{attr_name:this.addForm.attr_name,
        attr_sel:this.activeName 
        } )
         if(res.meta.status !== 201) {
        return this.$message.error('添加参数失败')
      }
      this.$message.success('添加参数成功')
      this.addDialogVisible=false
      this.getCateList()

      })

 
     
    },
    // 展现修改的弹出框
    async showEditDialog(attr_id) {
      const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`,{
        params:{
          attr_sel:this.activeName
        }
      })
      if(res.meta.status !== 200) {
        return this.$message.error('查询参数失败')
      }

      this.editForm = res.data

      this.editDialogVisible = true
    },

// 重置修改的表单
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
// 修改参数信息
     editParams() {
      this.$refs.editFormRef.validate(async valid => {
        if(!valid) return 

        const {data:res} = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`,{attr_name:this.editForm.attr_name,attr_sel:this.activeName

      })
      if(res.meta.status !== 200) {
        return this.$message.error('修改参数失败')
      }
     this.$message.success('修改参数成功')
     this.getParamsData()
     this.editDialogVisible = false

      })
      

    },
    // 删除参数
     async removeParams(attr_id) {
     const confirmResult = await this.$confirm('此操作将永久删除该参数, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err => err)
        // 用户取消了删除的操作
        console.log(confirmResult)
        if(confirmResult !== 'confirm' ) {
          return this.$message.info('已取消删除')
        }
        // 删除的业务
        const {data:res} = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
        if(res.meta.status !==200){
           return this.$message.error('删除参数失败')
        }
        this.$message.success('删除参数成功')
        this.getParamsData()
    },
    // 失去焦点或按下enter键触发该函数
    handleInputConfirm() {
      console.log('ok')
    },
    // 点击按钮，显示文本输入框
    showInput(row) {
      console.log('到')
    
      row.inputVisible = true
      // 让文本框得焦点
      // $nextTick方法的作用，就是当页面上元素重新渲染后，才会指定回调函数的代码
       this.$nextTick(_ => {
          this.$refs.saveTagInput.$refs.input.focus();
        });
      console.log(row)
      // this.inputVisible = ! this.inputVisible
    }
  },
  computed: {
    // 如果按钮需要被禁用，则返回true,否则返回false
    isBtnDisabled() {
      if(this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类的id
    cateId() {
      if(this.selectedCateKeys.length === 3) {
        return this.selectedCateKeys[2]
      }
      return null
    },
    // 动态算出标题
    titleText() {
      if(this.activeName === 'many') {
        return '动态参数'
      }
      return '静态属性'
    }
  }
}
</script>
<style lang="less" scoped>
.cat_opt{
  margin: 15px 0;
}
.el-tag{
  margin: 10px;
}
.input-new-tag{
  width:130px
}
</style>