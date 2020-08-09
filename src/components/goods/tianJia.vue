<template>
  <div>
    <el-breadcrumb separator="/">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>商品管理</el-breadcrumb-item>
  <el-breadcrumb-item>添加商品</el-breadcrumb-item>
  </el-breadcrumb>



<!-- 卡片 -->
  <el-card>
    <!-- 消息提示 -->
 <el-alert
    title="添加商品信息"
    type="info"
    center
    show-icon :closable="false">
  </el-alert>
<!--步骤条 -->

   <el-steps :space="200" :active="activeIndex-0" finish-status="success" align-center>
  <el-step title="基本信息"></el-step>
  <el-step title="商品参数"></el-step>
  <el-step title="商品属性"></el-step>
  <el-step title="商品图片"></el-step>
  <el-step title="商品内容"></el-step>
  <el-step title="完成"></el-step>
</el-steps>

<el-form :model="tianJiaForm" :rules="tianJiarules" label-position="top" ref="tianJiaForm" label-width="100px" >
<el-tabs :tab-position="'left'"  v-model="activeIndex" :before-leave = "tabChange" @tab-click="tabClick">
    <el-tab-pane label="基本信息" name="0">
      <el-form-item label="商品"  prop="goods_name">
        <el-input v-model="tianJiaForm.goods_name"></el-input>
      </el-form-item>
      <el-form-item label="商品
      价格" prop="goods_price" >
        <el-input v-model="tianJiaForm.goods_price" type="number"></el-input>
      </el-form-item>
      <el-form-item label="商品重量" prop="goods_weight" >
        <el-input v-model="tianJiaForm.goods_weight" type="number"></el-input>
      </el-form-item>
      <el-form-item label="商品数量" prop="goods_number" >
        <el-input v-model="tianJiaForm.goods_number" type="number"></el-input>
      </el-form-item>
      <el-form-item label="商品分类" prop="goods_cat">
        <!-- 商品分类的级联选择器 -->
        <el-cascader
    v-model="tianJiaForm.goods_cat"
    :options="catelist"
    :props="cateProps"
    @change="handleChange" ></el-cascader>
      </el-form-item>

      
    </el-tab-pane>
    <el-tab-pane label="商品参数" name="1">

      <el-form-item :label="item.attr_name" v-for="item in dongTaiList" :key="item.attr_id">
        <!-- 商品参数的复选框组 -->
      <el-checkbox-group v-model="item.attr_vals">
    <el-checkbox :label="cb" v-for="(cb,i) in item.attr_vals" :key="i" border></el-checkbox>
    
  </el-checkbox-group>
      </el-form-item>
      </el-tab-pane>
    <el-tab-pane label="商品属性" name="2"><el-form-item :label="item.attr_name" v-for="(item) in onlyTableData" :key="item.attr_id">
      <el-input v-model="item.attr_vals"></el-input>
      </el-form-item></el-tab-pane>
    <el-tab-pane label="商品图片" name="3">
      <!-- 图片的上传 -->
     <el-upload
 
  :action="uploadUrl"
  :on-preview="handlePreview"
  :on-remove="handleRemove"
  
  list-type="picture" :headers="headerObj" :on-success="handleSuccess">
  <el-button size="small" type="primary">点击上传</el-button>
  <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过500kb</div>
</el-upload>
    </el-tab-pane>
    <el-tab-pane label="商品内容" name="4">
      <!-- 富文本编辑器 -->
      <quill-editor v-model=" tianJiaForm. goods_introduce">
      </quill-editor>

      <el-button type="primary" class="tianjiaBtn" @click="tianJia">添加商品</el-button>
    </el-tab-pane>

  </el-tabs>
</el-form>
  </el-card>
   
   <!-- 预览图片 -->
   <el-dialog
  title="预览图片"
  :visible.sync="previewVisible"
  width="50%"
  >
  <img :src = "previewPath" class="preview">
 
</el-dialog>
  </div>
</template>
<script>
// 导入lodash
import _ from 'lodash'
export default {
  data() {
    return {
      activeIndex:'0',
      // 添加商品的表单数据
      tianJiaForm:{
        goods_name:'',
        goods_price:0,
        goods_weight:0,
        goods_number:0,
        // 商品所属的分类数组
        goods_cat:[],

        // 上传图片的临时路径对象
          pics:[],
      
      // 商品的描述
      goods_introduce:'',

        // 动态参数和静态属性的数组
      attrs:[]  
      },
      // 验证规则
     tianJiarules:{
       goods_name: [
         {required:true,message:'请输入商品',trigger:'blur'}
       ],
       goods_price: [
          {required:true,message:'请输入商品价格',trigger:'blur'}
       ],
       goods_weight: [
          {required:true,message:'请输入商品重量',trigger:'blur'}
       ],
       goods_number: [
          {required:true,message:'请输入商品数量',trigger:'blur'}
       ],
       goods_cat: [
         {required:true,message:'请输入商品分类',trigger:'blur'}
       ]
      

     },
    //  商品分类
     catelist:[],

     value:'',
     cateProps:{
       expandTrigger: 'hover',
       value:"cat_id",
       label:"cat_name",
       children:"children"
     },
    //  动态参数列表
    dongTaiList:[],

    // 静态参数列表
    onlyTableData:[],

    uploadUrl:"http://127.0.0.1:8888/api/private/v1/upload",
    // 图片上传组件的headers请求头
    headerObj:{
      Authorization:window.sessionStorage.getItem('token')
    },
    // 预览图的图片
    previewPath:'',
    previewVisible:false,
    attrs:[]
    }
  },
  created() {
    this.getCateList()
   
  },
  methods: {
    // 获取商品分类
    async getCateList() {
      const {data:res} = await this.$http.get('categories')
      if(res.meta.status !== 200) {
        return this.$message.error('获取商品分类数据失败')
      }
      this.catelist = res.data
      console.log(this.catelist)
      
    },
    // 级联选择器选的项变化，会触发这个函数
    handleChange() {
      console.log(this.tianJiaForm.goods_cat)
  // 控制只允许选三级商品分类
      if(this.tianJiaForm.goods_cat.length < 3) {
        this.tianJiaForm.goods_cat = []
      }
    },
    // Tab标签页改变触发
    tabChange(newTab,oldTab) {
      // console.log(oldTab,newTab)
      if(oldTab === '0' && this.tianJiaForm.goods_cat.length !== 3) {
        this.$message.error('请先选择商品分类失败')
        // 阻止Tab标签页改变
        return false
      }
     
    },
    // 在点击标签页触发
    async tabClick() {
      // console.log(this.activeIndex)
      // 获取动态参数
      if(this.activeIndex == '1') {
        
          const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{
        params:{
          sel:'many' 
        }
      })
      if(res.meta.status!==200){
        return this.$message.error("获取参数失败")
      }
      console.log(res.data)
      res.data.forEach(item=> {
        item.attr_vals = 
        item.attr_vals.length === 0?
        [] : item.attr_vals.split(' ')//要两个空格
      })
      this.dongTaiList = res.data
    }
    // 获取静态属性
    else if(this.activeIndex == '2') {
      
         const {data:res} = await this.$http.get(`categories/${this.cateId}/attributes`,{
        params:{
          sel:'only'
        }
      })
      if(res.meta.status!==200){
        return this.$message.error("获取属性失败")
      }
      console.log(res.data)
    
      this.onlyTableData = res.data
      }
      },
      // 点击图片触发
      handlePreview(file) {
       this.previewPath = file.response.data.url
       this.previewVisible = true
        // console.log(file)
      },
      // 移除图片的操作,从数组移除
      handleRemove(file) {
        // console.log(file)
        // 找到图片的临时路径
        const filePath = file.response.data.temp_path
        // 从pics数组，找到图片相应的索引值
        const i = this.tianJiaForm.pics.findIndex(x => {
          x.pic === filePath
        })
        // 调用数组的splice方法，把图片对象，从pics数组移除
        this.tianJiaForm.pics.splice(i,1)
        console.log(this.tianJiaForm  )

      },
      // 图片上传成功
    handleSuccess(response) {
      console.log(response)
      // 1 拼出得到一个图片对象
      const picInfo = {
        pic:response.data.tmp_path
      }
      // 2 将图片对象，添加到数组
      
      
      this.tianJiaForm.pics.push(picInfo)
      console.log(this.tianJiaForm)
    },
    // 添加商品
    tianJia(){
      // console.log(this.tianJiaForm)
      this.$refs.tianJiaForm.validate (async valid => {
        if(!valid) {
          return this.$message.error("请添加必要的表单项")
        }

        // 添加商品的操作
        // lodash的深拷贝函数
        const form = _.cloneDeep(this.tianJiaForm)
        form.goods_cat = form.goods_cat.join(",")
        // console.log(3)
// 取出动态参数的特定项
    this.dongTaiList.forEach(item => {
      const newObj = {
        attr_id:item.attr_id,
        attr_value:item.attr_vals.join(' ')
      }
      console.log("1")
      console.log(this)
      this.tianJiaForm.attrs.push(newObj)
    })
// 取出静态参数的特定项
this.onlyTableData.forEach(item => {
      const newObj = {
        attr_id:item.attr_id,
        attr_value:item.attr_vals
      }
      this.tianJiaForm.attrs.push(newObj)
    })
    form.attrs = this.tianJiaForm.attrs
      console.log(form)

// 添加商品
// 商品要唯一
const {data:res} = await this.$http.post('goods',form)
console.log("---")
// console.log(res)
if(res.meta.status !== 201) {
  return this.$message.error('添加商品失败，请填写完整的参数')
}
this.$message.success('添加商品成功')
this.$router.push('/goods')
      })
    }
    
    
  },
  computed: {
   cateId() {
      if(this.tianJiaForm.goods_cat.length ==3 ) {
        return this.tianJiaForm.goods_cat[2]
    }
    return null
   }
  },
}
</script>
<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.preview{
  width:100%
}
.tianjiaBtn{
  margin-top:10px;
}
</style>