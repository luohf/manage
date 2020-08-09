<template>
  <div>
  <el-breadcrumb separator="/">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>订单管理</el-breadcrumb-item>
  <el-breadcrumb-item>订单列表</el-breadcrumb-item>
  </el-breadcrumb>
  <!-- 卡片 -->
  <el-card>
    <el-row>
      <el-col :span="8">
        <el-input placeholder="请输入内容" >
          <el-button slot="append" icon="el-icon-search"></el-button>
  </el-input>
      </el-col>
    </el-row>
    <!-- 订单列表数据 -->
    <el-table :data="orderList" border stripe>
  <el-table-column type="index">
  </el-table-column>
 <el-table-column label="订单编号" prop="order_number"></el-table-column>
  <el-table-column label="订单价格" prop="order_price" >
  </el-table-column>
   <el-table-column label="是否付款" prop="pay_status" >
     <template slot-scope="scope">
       <el-tag type="success" v-if="scope.row.pay_status === '1'">已付款</el-tag>
       <el-tag type="danger">未付款</el-tag>
     </template>
  </el-table-column>
  <el-table-column label="是否发货" prop="is_send" >
  </el-table-column>
   <!-- <el-table-column label="下单时间"
   prop="create_time" >
   <template slot-scope="scope">
     {{scope.row.create_time | dateFormat}}
   </template>
  </el-table-column> -->
  
   <el-table-column label="操作" >
     <template slot-scope="scope">
       <el-button type="primary" icon="el-icon-edit" size="mini"></el-button>
       <el-button type="success" icon="el-icon-location" size="mini" @click="showGoodsLocation"></el-button>

     </template>
  </el-table-column>
</el-table>

<!-- 分页区域 -->
<el-pagination
      @size-change="handleSizeChange"
      @current-change="handleCurrentChange"
      :current-page="queryInfo.pagenum"
      :page-sizes="[5,10,15]"
      :page-size="queryInfo.pagesize"
      layout="total, sizes, prev, pager, next, jumper"
      :total="total">
    </el-pagination>
  </el-card>
  <!-- 展现物流进度的框 -->
  <el-dialog
  title="物流进度"
  :visible.sync="LocationVisible"
  width="50%"
  :before-close="handleClose">
  <!-- 时间线 -->
 <el-timeline >
    <el-timeline-item
      v-for="(activity, index) in  locationInfo"
      :key="index"
      :timestamp="activity.time">
      {{activity.context}}
    </el-timeline-item>
  </el-timeline>
  <span slot="footer" class="dialog-footer">
    <el-button @click="LocationVisible = false">取 消</el-button>
    <el-button type="primary" @click="LocationVisible = false">确 定</el-button>
  </span>
</el-dialog>

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
      // 订单列表
      orderList:[],
       // 总订单条数
      total:0,
      LocationVisible:false,
      // 物流
      locationInfo:[]

    }
  },
   created() {
    this.getOrderlist() 
  },
  methods: {
      async getOrderlist() {
const {data:res} = await this.$http.get("orders",{
        params:this.queryInfo
      })
      if(res.meta.status !== 200){
        return this.$message.error("获取订单列表失败")
      }
      this.$message.success("获取订单列表成功")
     console.log(res)
    this.orderList = res.data.goods
    this.total = res.data.total
    },
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderlist()
    },
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderlist()

    },
    async showGoodsLocation() {
      const {data:res} = await this.$http.get("/kuaidi/1106975712662")
      // console.log(res)
      if(res.meta.status !== 200) {
        return this.$message.error('获取物流进度失败')
      }
      this.locationInfo = res.data
      console.log(this.locationInfo)
      this.LocationVisible = true
    },
    handleClose() {

    }
  },

}
</script>
<style lang="less" scoped>

</style>