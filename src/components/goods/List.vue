<template>
  <div>
  <el-breadcrumb separator="/">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>商品管理</el-breadcrumb-item>
  <el-breadcrumb-item>商品列表</el-breadcrumb-item>
  </el-breadcrumb>
  
    
  <!-- 卡片 -->
  <el-card>
    <el-row :gutter="20">
      <el-col :span="8">
<el-input placeholder="请输入内容"  class="input-with-select" v-model="queryInfo.query " clearable @clear = "clearSearch">
<el-button slot="append" icon="el-icon-search" @click="SearchGoods"></el-button>
  </el-input>
      </el-col>

      <el-col :span="4">
        <el-button type="primary" @click="tianJiaGoods">
          添加商品
        </el-button>
      </el-col>
    </el-row>
<el-table :data="goodslist" border >
  <el-table-column type="index">
  </el-table-column>
 <el-table-column label="商品" prop="goods_name"></el-table-column>
  <el-table-column label="商品价格(元)" prop="goods_price" width="98">
  </el-table-column>
   <el-table-column label="商品重量" prop="goods_weight" width="78">
  </el-table-column>
   <!-- <el-table-column label="创建时间"
   prop="add_time" width="140">
   <template slot-scope="scope">
    {{scope.row.add_time | dateFormat }}
       
   </template>
  </el-table-column> -->
   <el-table-column label="操作" width="130">
     <template slot-scope="scope">
       <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
       <el-button type="danger" icon="el-icon-delete" size="mini" ></el-button>

     </template>
  </el-table-column>
</el-table>

<!-- 分页区域 -->
  <el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5,10,15,20]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total" background>
    </el-pagination>
   
  </el-card>

  </div>
</template>
<script>
export default {
  data() {
    return {
      // 查询参数对象
      queryInfo: {
        query :'',
        pagenum:1,
        pagesize:10
      },
      // 商品列表
      goodslist:[],
      // 总商品条数
      total:0

    }
  },
  created() {
    this.getGoodslist() 
  },
  methods: {
    // 查询商品列表
    async getGoodslist() {
      const {data:res} = await this.$http.get("goods",{
        params:this.queryInfo
      })
      if(res.meta.status !== 200){
        return this.$message.error("获取商品失败")
      }
      this.$message.success("获取商品成功")
    //  console.log(res)
    this.goodslist = res.data.goods
    this.total = res.data.total

    },
    // 触发显示多少条
    handleSizeChange(newSize) {
      // console.log(newSize)
      this.queryInfo.pagesize = newSize
          this.getGoodslist() 

    },
    // 触发显示页
    handleCurrentChange(newPage) {
      // console.log(newPage)
    this.queryInfo.pagenum = newPage
    this.getGoodslist() 

    },
    // 查询商品
    SearchGoods() {
      this.getGoodslist() 
    },
    // 清空搜索
    clearSearch() {
       this.getGoodslist() 
    },
    //添加商品
    tianJiaGoods() {
      // console.log(this)
      this.$router.push('/goods/add')
    }
  },
}
</script>
<style lang="less" scoped>
  
</style>