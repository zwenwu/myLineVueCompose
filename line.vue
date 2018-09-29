<template>
  <div ref="dom" @click="handleClick"></div>
</template>

<script>

import echarts from 'echarts'
import tdTheme from './theme.json'
import excel from '@/libs/excel'

echarts.registerTheme('tdTheme', tdTheme)

export default {
  name: 'ChartLine',
  props: {
    text: String,
    subtext: String,
    value: Object,
    xAxis: Array
  },
  data () {
    return {
      echartDom: null,
      number: 1,
      icon: 'path://M281 543q-27 -1 -53 -1h-83q-18 0 -36.5 -6t-32.5 -18.5t-23 -32t-9 -45.5v-76h912v41q0 16 -0.5 30t-0.5 18q0 13 -5 29t-17 29.5t-31.5 22.5t-49.5 9h-133v-97h-438v97zM955 310v-52q0 -23 0.5 -52t0.5 -58t-10.5 -47.5t-26 -30t-33 -16t-31.5 -4.5q-14 -1 -29.5 -0.5 t-29.5 0.5h-32l-45 128h-439l-44 -128h-29h-34q-20 0 -45 1q-25 0 -41 9.5t-25.5 23t-13.5 29.5t-4 30v167h911zM163 247q-12 0 -21 -8.5t-9 -21.5t9 -21.5t21 -8.5q13 0 22 8.5t9 21.5t-9 21.5t-22 8.5zM316 123q-8 -26 -14 -48q-5 -19 -10.5 -37t-7.5 -25t-3 -15t1 -14.5 t9.5 -10.5t21.5 -4h37h67h81h80h64h36q23 0 34 12t2 38q-5 13 -9.5 30.5t-9.5 34.5q-5 19 -11 39h-368zM336 498v228q0 11 2.5 23t10 21.5t20.5 15.5t34 6h188q31 0 51.5 -14.5t20.5 -52.5v-227h-327z'
    }
  },
  methods: {
    handleSet(){
      this.$nextTick(() => {
        let that = this
        //获取名
        let xAxisData = Object.keys(this.value)
        //获取值
        let seriesData = Object.values(this.value)
        //设置series
        function setSeries(data,key){
          var serie = []
          for (var x in key) {
            var red = Math.random()*254+1
            var green = Math.random()*254+1
            var blue = Math.random()*254+1
            var item = {
              name: key[x],
              type: 'line',
              smooth: true,
              itemStyle: {
                color: 'rgb(' + red + ',' + green + ',' + blue + ')'
              },
              areaStyle: {
                color: new echarts.graphic.LinearGradient(0, 0, 0, 1, [{
                    offset: 0,
                    color: 'rgb(' + red + ',' + (green+80) + ',' + (blue-70) + ')'
                }, {
                    offset: 1,
                    color: 'rgb(' + red + ',' + green + ',' + blue + ')'
                }])
              },
              data: data[x]
            }
            serie.push(item)
          }
          return serie;
        }
        //设置legend选中状态
        function selecteds(data,number){
          let sel = {}
          if (number === 0) {
            sel = data
          } else {
            for (var x in data) {
              if (x < 3) {
                continue
              }
              sel[data[x]] = false
            }
          }
          return sel
        }
        //设置时间
        function setDate(time){
          let dateArray = []
          for (var x in time) {
            let date = Date.parse(time[x])/1000 + 3600*8
            let locale = new Date(parseInt(date) * 1000).toLocaleString()
            dateArray[x] = locale
          }
          return dateArray
        }
        //设置option
        let option = {
          //标题
          title: {
            text: this.text,
            subtext: this.subtext,
            x: 'center'
          },
          //提示框
          tooltip : {
            trigger: 'axis',
            //axisPointer: {type: 'cross'},
            //alwaysShowContent: true,
            position: function (pos, params, el, elRect, size) {
                var obj = {top: 10};
                if (pos[0] < size.viewSize[0] / 2) {
                  obj['left'] = 30
                } else {
                  obj['right'] = 250
                }
                //obj[['left', 'right'][+(pos[0] < size.viewSize[0] / 2)]] = 30;
                return obj;
            }
          },
          //工具栏
          toolbox: {
            show : true,
            feature : {
              //选款组件
              //brush  : {},
              //数据视图
              dataView : {
                show: true,
                readOnly: false,
                optionToContent: function(opt) {
                  //自定义格式
                  var axisData = opt.xAxis[0].data;
                  var series = opt.series;
                  var table = '<table style="width:50%;text-align:center"><tbody><tr><td>时间</td>'
                  for (var i in series) {
                    table += '<td>' + series[i].name + '</td>'
                  }
                  table += '</tr>'
                  for (var i in axisData) {
                    table += '<tr>'
                              + '<td>' + axisData[i] + '</td>'
                    for (var j in series) {
                      table += '<td>' + series[j].data[i] + '</td>'
                    }
                    table += '</tr>'
                  }
                  table += '</tbody></table>'
                  return table;
                }
              },
              //图形切换
              magicType : {show: true, type: ['line', 'bar']},
              //还原
              restore : {show: true},
              //下载为图片
              saveAsImage : {show: true},
              //区域缩放
              dataZoom: {show: true},
              //自定义按钮
              myTool1:{
                show:true,
                title:'导出数据',
                icon:that.icon,
                onclick:function (){
                  that.exportExcel(xAxisData,seriesData)
                }
              },
              myTool2: {
                show: true,
                title: '搜索',
                icon: 'image://http://echarts.baidu.com/images/favicon.png',
                onclick: function (){
                  that.$Notice.open({
                    title: 'Notification title',
                    render: h => {
                        return h('input')
                    },
                    duration: 0
                  })
                }
              }
            }
          },
          //图例
          legend: {
            type: 'scroll',
            orient: 'vertical',
            selected: selecteds(xAxisData,this.number),
            //tooltip: {show: true},
            //backgroundColor: 'rgb(128, 128, 128)',
            //borderWidth: 10,
            right: 10,
            top: 40,
            bottom: 25
          },
          //坐标网格
          grid: {
            show: true,
            //backgroundColor: 'rgb(0, 0, 0)',
            top: '15%',
            left: '2%',
            right: '21%',
            bottom: '10%',
            containLabel: true
          },
          //x轴
          xAxis: {
            type: 'category',
            data: setDate(this.xAxis),
            //type: 'time',
            //min: new Date("2015/08/24"),
            //max: new Date("2015/10/30"),
            //maxInterval: 3600 * 24 * 1000,
            splitLine:{show: false}
          },
          //y轴
          yAxis: {
            type: 'category',
            splitNumber: 20,
            boundaryGap: [0, '100%'],
            splitLine:{show: false}
          },
          //区域缩放
          dataZoom: [
            {
              show: true,
              xAxisIndex:[0],
              type: 'slider',
              start: 90,
              end: 100
            },{
              show: true,
              yAxisIndex:[0],
              type: 'slider',
              right: '19%'
            },{
              xAxisIndex:[0],
              type: 'inside',
              start: 90,
              end: 100
            },{
              yAxisIndex:[0],
              type: 'inside'
            }
          ],
          //数据列表
          series: setSeries(seriesData,xAxisData)
        }
        //初始化
        if (!this.echartDom) {
          this.echartDom = echarts.init(this.$refs.dom, 'tdTheme')
        }
        //设置Line表
        this.echartDom.setOption(option,true)
      })
    },
    handleClick(event){
      let that = this
      this.echartDom.on('click', function (param) {
        if (param.componentType === 'series') {
          that.$emit('setData',param.seriesName)
        }
      })
    },
    exportExcel (key,val) {
    console.log(this.dateToSec(this.xAxis))
      let tableData = []
      tableData = this.value
      for (var x in key) {

        tableData.push(line)
      }
      if (val) {
        this.exportLoading = true
        const params = {
          title: key,
          key: key,
          data: tableData,
          autoWidth: true,
          filename: 'host/ip每小时次数'
        }
        excel.export_array_to_excel(params)
        this.exportLoading = false
      } else {
        this.$Message.info('数据为空！')
      }
    },
    dateToSec (time) {
      let second = []
      for (var x in time) {
        second[x] = Date.parse(time[x])/1000 + 3600*8
      }
      return second
    }
  },
  mounted () {
    this.handleSet()
  }
}
</script>

<style lang="less">

</style>
