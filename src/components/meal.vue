<template>
  <div>
    <el-container>
      <el-header></el-header>
      <el-main>
        <el-row>
          <el-col :span="12">
            <el-input v-model="input" placeholder="请输入内容"></el-input>
          </el-col>
          <el-col :span="12">
            <el-button>新增供应商</el-button>
          </el-col>

          <el-col>
            <div style="height: 50px"></div>
          </el-col>

          <el-col :span="12">
            <el-select v-model="supplier_list" placeholder="请选择">
              <el-option
                  v-for="item in options"
                  :key="item.supplier"
                  :label="item.supplier"
                  :value="item.supplier">
              </el-option>
            </el-select>
          </el-col>
          <el-col :span="12">
            <el-button>删除供应商</el-button>
          </el-col>

          <el-col>
            <div style="height: 50px"></div>
          </el-col>

          <el-col :span="12">
            <el-button type="text" @click="dialogFormVisible = true">上传订单excel</el-button>

            <el-dialog title="上传EXCEL" :visible.sync="dialogFormVisible">
              <el-form :model="form">
                <el-form-item label="上传日期" :label-width="formLabelWidth">
                  <el-date-picker
                      v-model="uploaddate"
                      type="date"
                      placeholder="选择日期">
                  </el-date-picker>
                </el-form-item>
                <el-form-item label="供应商" :label-width="formLabelWidth">
                  <el-select v-model="supplier_list" placeholder="请选择">
                    <el-option
                        v-for="item in options"
                        :key="item.supplier"
                        :label="item.supplier"
                        :value="item.supplier">
                    </el-option>
                  </el-select>
                </el-form-item>
                <el-form-item>
                  <el-upload
                      class="upload-demo"
                      drag
                      action="https://jsonplaceholder.typicode.com/posts/"
                      multiple>
                    <i class="el-icon-upload"></i>
                    <div class="el-upload__text">将文件拖到此处，或<em>点击上传</em></div>
                    <div class="el-upload__tip" slot="tip">只能上传jpg/png文件，且不超过500kb</div>
                  </el-upload>
                </el-form-item>

              </el-form>
              <div slot="footer" class="dialog-footer">
                <el-button @click="dialogFormVisible = false">取 消</el-button>
                <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
              </div>
            </el-dialog>
          </el-col>


          <el-date-picker
              v-model="daterange"
              type="daterange"
              align="right"
              unlink-panels
              value-format="yyyyMMdd"
              range-separator="至"
              start-placeholder="开始日期"
              end-placeholder="结束日期"
              :picker-options="pickerOptions"
              :change=dateChange()
          >
          </el-date-picker>

          <el-col>
            <span>每日点单数</span>
            <div id="chart1" style="height: 300px;width: 600px"></div>

          </el-col>

          <el-col>
            <span>每日各类菜点单数</span>
            <div id="chart2" style="height: 300px;width: 600px"></div>

          </el-col>

          <el-col>
            <span>供应商菜品点单分布</span>
            <div id="chart3" style="height: 300px;width: 600px"></div>

          </el-col>
          <el-col>
            <span>人员点单数排行</span>
            <div id="chart4" style="height: 300px;width: 600px"></div>

          </el-col>


        </el-row>


      </el-main>

    </el-container>


  </div>
</template>

<script>
import * as echarts from 'echarts/core';
import {
  DatasetComponent,
  TooltipComponent,
  LegendComponent,
  GridComponent,
  TitleComponent
} from 'echarts/components';
import {
  LineChart, BarChart, PieChart
} from 'echarts/charts';
import {
  CanvasRenderer
} from 'echarts/renderers';

echarts.use(
    [DatasetComponent, TooltipComponent, LegendComponent, BarChart, GridComponent, LineChart, CanvasRenderer, TitleComponent, PieChart]
);


export default {
  name: "meal.vue",
  data() {
    return {
      input: '',
      supplier_list: [],
      dialogFormVisible: false,
      form: {
        name: '',
        region: '',
        date1: '',
        date2: '',
        delivery: false,
        type: [],
        resource: '',
        desc: ''
      },
      uploaddate: new Date(),
      daterange: [20210601, 20210627],
      formLabelWidth: '120px',
      pickerOptions: {
        shortcuts: [
          {
            text: '最近一周',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 7);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近一个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 30);
              picker.$emit('pick', [start, end]);
            }
          }, {
            text: '最近三个月',
            onClick(picker) {
              const end = new Date();
              const start = new Date();
              start.setTime(start.getTime() - 3600 * 1000 * 24 * 90);
              picker.$emit('pick', [start, end]);
            }
          }
        ]
      }
    }
  },
  mounted() {
    this.initCharts1();
    this.initCharts2();
    this.initCharts3();
    this.initCharts4();
  },

  methods: {
    initCharts1() {
      let chartDom = document.getElementById('chart1');
      let myChart = echarts.init(chartDom);
      let option;
      this.$http.get('http://www.yzx.com:8082/meal/api/v1/getmealCount', {
        params: {
          startTime: this.daterange[0],
          endTime: this.daterange[1],
        }
      }).then((response) => {
        let arr1 = [];
        let arr2 = [];
        response.data.result.forEach(function (object) {
          arr1.push(object.date)
          arr2.push(object.count)
        });
        option.xAxis.data = arr1;
        option.series[0].data = arr2;
        myChart.setOption(option)
      }, (response) => {
        console.log(response)
      });

      option = {
        xAxis: {
          type: 'category',
          data: []
        },
        yAxis: {
          type: 'value'
        },
        series: [{
          data: [],
          type: 'line'
        }],
        tooltip: {
          trigger: 'axis',
        },
      };

      option && myChart.setOption(option);
    },
    initCharts2() {

      let chartDom = document.getElementById('chart2');
      let myChart = echarts.init(chartDom);
      let option;
      this.$http.get('http://www.yzx.com:8082/meal/api/v1/getmealclassificationCount', {
        params: {
          startTime: this.daterange[0],
          endTime: this.daterange[1],
        }
      }).then((response) => {
        let obj = {}
        let arr = []
        response.data.result.forEach(function (object) {
          if (object.date in obj) {
            let tmp = {}
            tmp[object.meal] = object.count
            obj[object.date].push(tmp)
          } else {
            let tmp = {}
            tmp[object.meal] = object.count
            obj[object.date] = [tmp]
          }
        });
        console.log(obj)
        for (var o in obj) {
          let set = {}
          set.product = o
          obj[o].forEach(function (object){
            set[Object.keys(object)[0]] = object[Object.keys(object)[0]]
          })
          arr.push(set)
        } console.log(arr)
        option.dataset.source = arr
        myChart.setOption(option)
      }, (response) => {
        console.log(response)
      });


      option = {
        legend: {},
        tooltip: {
          trigger: 'axis',
        },
        dataset: {
          source: [
            {product: 'Matcha Latte', '2015': 43.3, '2016': 85.8, '2017': 93.7},
            {product: 'Milk Tea', '2015': 83.1, '2016': 73.4, '2017': 55.1},
            {product: 'Cheese Cocoa', '2015': 86.4, '2016': 65.2, '2017': 82.5},
            {product: 'Walnut Brownie', '2015': 72.4, '2016': 53.9, '2017': 39.1}
          ]
        },
        xAxis: {type: 'category'},
        yAxis: {},
        // Declare several bar series, each will be mapped
        // to a column of dataset.source by default.
        series: [
          {type: 'bar'},
          {type: 'bar'},
          {type: 'bar'}
        ]
      };

      option && myChart.setOption(option);
    },
    initCharts3() {
      let chartDom = document.getElementById('chart3');
      let myChart = echarts.init(chartDom);
      let option;

      option = {
        title: {
          text: '某站点用户访问来源',
          subtext: '纯属虚构',
          left: 'center'
        },
        tooltip: {
          trigger: 'item'
        },
        legend: {
          orient: 'vertical',
          left: 'left',
        },
        series: [
          {
            name: '访问来源',
            type: 'pie',
            radius: '50%',
            data: [
              {value: 1048, name: '搜索引擎'},
              {value: 735, name: '直接访问'},
              {value: 580, name: '邮件营销'},
              {value: 484, name: '联盟广告'},
              {value: 300, name: '视频广告'}
            ],
            emphasis: {
              itemStyle: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: 'rgba(0, 0, 0, 0.5)'
              }
            }
          }
        ]
      };

      option && myChart.setOption(option);
    },
    initCharts4() {
      let chartDom = document.getElementById('chart4');
      let myChart = echarts.init(chartDom);
      let option;

      option = {
        title: {
          text: '世界人口总量',
          subtext: '数据来自网络'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow'
          }
        },
        legend: {
          data: ['2011年', '2012年']
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: {
          type: 'value',
          boundaryGap: [0, 0.01]
        },
        yAxis: {
          type: 'category',
          data: ['巴西', '印尼', '美国', '印度', '中国', '世界人口(万)']
        },
        series: [
          {
            name: '2011年',
            type: 'bar',
            data: [18203, 23489, 29034, 104970, 131744, 630230]
          },
          {
            name: '2012年',
            type: 'bar',
            data: [19325, 23438, 31000, 121594, 134141, 681807]
          }
        ]
      };

      option && myChart.setOption(option);
    },
    dateChange() {
      // this.initCharts1();
      // this.initCharts2();
      // this.initCharts3();
      // this.initCharts4();
    }
  }
}
</script>


<style scoped>

</style>