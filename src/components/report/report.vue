<template>
  <div>
   <el-breadcrumb separator="/">
  <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
  <el-breadcrumb-item>数据统计</el-breadcrumb-item>
  <el-breadcrumb-item>数据报表</el-breadcrumb-item>
  </el-breadcrumb>
  <el-card>
     <div id="main" style="width: 600px;height:400px;"></div>
  </el-card>
  </div>
</template>
<script>
import echarts from 'echarts'
export default {
  // 此时，页面上的dom元素，已经被渲染完毕
 mounted() {
   var oneEcharts = echarts.init(document.getElementById('main'))
   var option = {
            title: {
                text: 'ECharts 入门示例'
            },
            tooltip: {},
            legend: {
                data:['销量']
            },
            xAxis: {
                data: ["衬衫","羊毛衫","雪纺衫","裤子","高跟鞋","袜子"]
            },
            yAxis: {},
            series: [{
                name: '销量',
                type: 'bar',
                data: [5, 20, 36, 10, 10, 20]
            }]
        }
        oneEcharts.setOption(option);
 },
 
}
</script>
<style lang="less" scoped>

</style>