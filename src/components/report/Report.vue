<template>
  <div>
    <!-- 面包屑导航区 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>数据统计</el-breadcrumb-item>
      <el-breadcrumb-item>数据报表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 使用 echarts -->
      <div id="main" style="width: 1000px; height: 400px"></div>
    </el-card>
  </div>
</template>

<script>
// 导入echarts 5.0以上引入时需要加入 *as
import * as echarts from 'echarts'


export default {
  name: '',
  data() {
    return {
      // 需要合并的数据
      options: {
          title: {
            text: '用户来源'
          },
          tooltip: {
            trigger: 'axis',
            axisPointer: {
              type: 'cross',
              label: {
                backgroundColor: '#E9EEF3'
              }
            }
          },
          grid: {
            left: '3%',
            right: '4%',
            bottom: '3%',
            containLabel: true
          },
          xAxis: [
            {
              boundaryGap: false
            }
          ],
          yAxis: [
            {
              type: 'value'
            }
          ]
        },
    }
  },
  created() {},
  methods: {},
  async mounted() {
    //   页面上元素渲染完毕后，初始化图表
    var myChart = echarts.init(document.getElementById('main'))
    // 发送请求获取数据
    const { data: res } = await this.$http.get('reports/type/1')
    if (res.meta.status !== 200) {
      return this.$message.error('获取折线图数据失败！')
    }
    console.log(res);
    // 合并两个对象
    const result = {...this.options,...res.data}
    // 展示数据
    myChart.setOption(result)
  },
}
</script>

<style lang="less" scoped>
</style>