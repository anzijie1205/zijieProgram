<template>
  <div class="energy-container">
        <div :class="this.$store.state.station.stationInfo.isGf ? 'statistics-top1' : 'statistics-top2'">
            <div class="substation-pannel ele-box box1">
                <template>
                        <i class="borderCorner topLeft"></i>
                        <i class="borderCorner topRight"></i>
                        <i class="borderCorner bottomLeft"></i>
                        <i class="borderCorner bottoRight"></i>
                </template>
                <div class="info-content">
                    <el-form size="mini" label-position="left">
                        <el-form-item label="市电电量:">{{decimalHandle(storageInfo.cityPowerSupplyKwhDay,1)}}kWh</el-form-item>
                        <el-form-item label="直流负载电量:">{{decimalHandle(storageInfo.dcLoadKwhDay,1)}}kWh</el-form-item>
                        <el-form-item label="开关电源直流输出电量:">{{decimalHandle(storageInfo.switchPowerOutputKwhDay,1)}}kWh</el-form-item>
                    </el-form>
                </div>
            </div>
            <div class="substation-pannel ele-box box2" v-if="this.$store.state.station.stationInfo.isGf">
                <template>
                        <i class="borderCorner topLeft"></i>
                        <i class="borderCorner topRight"></i>
                        <i class="borderCorner bottomLeft"></i>
                        <i class="borderCorner bottoRight"></i>
                </template>
                <div class="info-content">
                    <el-form size="mini" label-position="left">
                        <el-form-item label="光伏发电量:">{{decimalHandle(storageInfo.solarOutputKwhDay,2)}}kWh</el-form-item>
                        <el-form-item label="光伏发电时长:">{{decimalHandle(storageInfo.solarOutputTimeDay,2)}}h</el-form-item>
                    </el-form>
                    <el-form size="mini" label-position="left">
                        <el-form-item label="日峰值功率:">{{decimalHandle(storageInfo.solarPeakPowerDay,2)}}kW</el-form-item>
                        <el-form-item label="月峰值功率:">{{decimalHandle(storageInfo.solarPeakPowerMonth,2)}}kW</el-form-item>
                    </el-form>
                </div>
            </div>
            <div class="substation-pannel ele-box box3">
                <template>
                        <i class="borderCorner topLeft"></i>
                        <i class="borderCorner topRight"></i>
                        <i class="borderCorner bottomLeft"></i>
                        <i class="borderCorner bottoRight"></i>
                    </template>
                    <div class="info-content">
                    <el-form size="mini" label-position="left">
                        <el-form-item label="储能削尖电量:">{{decimalHandle(storageInfo.batteryTopDischargeKwhDay,2)}}kWh</el-form-item>
                        <el-form-item label="储能削峰电量:">{{decimalHandle(storageInfo.batteryPeakDischargeKwhDay,1)}}kWh</el-form-item>
                        <el-form-item label="储能填谷电量:">{{decimalHandle(storageInfo.batteryValleyChargeKwhDay,1)}}kWh</el-form-item>
                    </el-form>
                    <el-form size="mini" label-position="left">
                        <el-form-item label="充电时长:">{{decimalHandle(storageInfo.batteryChargeTimeDay,2)}}h</el-form-item>
                        <el-form-item label="充电次数:">{{storageInfo.batteryChargeCountDay}}次</el-form-item>
                        <el-form-item label="储能充电量:">{{decimalHandle(storageInfo.batteryChargeKwhDay,1)}}kWh</el-form-item>
                    </el-form>
                    <el-form size="mini" label-position="left">                      
                        <el-form-item label="放电时长:">{{decimalHandle(storageInfo.batteryDischargeTimeDay,2)}}h</el-form-item>
                        <el-form-item label="放电次数:">{{storageInfo.batteryDischargeCountDay }}次</el-form-item>
                        <el-form-item label="储能放电量:">{{decimalHandle(storageInfo.batteryDischargeKwhDay,1)}}kWh</el-form-item>
                    </el-form>
                </div>
            </div>
        </div>
        <div class="statistics-bottom trend-chart-container">
                <h4 class="trend-data-title">储能功率日趋势</h4>
                <trendDatePick :chartParams="chartParam" @updateChartData="updateByDate" />
                <div class="trend-chart-box">
                        <div class="trend-chart-box-inner">
                            <template v-if="showChartData">
                                <doubleYLineChart :chartData="chartData" :height="'580px'"/>
                            </template>
                            <template v-else>
                                <div class="monitor-no-data-box" style="height: 100%">
                                    <div class="no-data"></div>
                                    <span>暂无数据</span>
                                </div>
                            </template>
                        </div>
                    </div>
            </div>
  </div>
</template>

<script>
import {wzLc7dnHomePVStackMonitor} from '@/api/photovoltaicMonitor/index.js'
import doubleYLineChart from "../../../components/CommonChart/doubleYLineChart.vue"
import trendDatePick from "../../components/trendDatePicker/index"
export default {
    components:{trendDatePick,doubleYLineChart},
    props:{
            stationId:{
                type:String,
                default:'',
            }
        },
    data(){
        return {
            storageInfo:{},
            chartParam:{   //日趋势参数
                  dateType:0,    //0 选择日；1 选择日期范围
                  date:'', 
              },
            showChartData:false,
            chartData:{},
        }
    },
    created() {
    },
    watch: {
        stationId:{
            deep: true,
            handler(val) {
                if (val) {
                    this.updateByDate()
                }
            }
        }
      },
    methods:{
        //储能光伏直流负载开关电源电量
        getStorageInfo() {
            let params = {
                id:this.stationId,
                pvType:1,
                startDate:this.chartParam.date
            }
            wzLc7dnHomePVStackMonitor(params).then(response =>{
                this.storageInfo = response.data
            })
        },
        //储能功率日趋势
        getStorageChart() {
            let params = {
                id:this.stationId,
                pvType:2,
                startDate:this.chartParam.date
            }
            wzLc7dnHomePVStackMonitor(params).then(response =>{
                if(response.data) {
                    let res = response.data
                    let solarOutputKwhDay = []    //光伏
                    let switchPowerOutputKwhDay = []    //开关电源直流输出
                    let batteryDischargeKwhDay = []    //储能放电
                    let batteryChargeKwhDay = []   //储能充电
                    let dcLoadKwhDay = []    //直流负载
                    let electrovalence = []    //电价
                    res.yData.forEach(item => {
                        this.$store.state.station.stationInfo.isGf ? solarOutputKwhDay.push(this.decimalHandle(item.solarOutputKwh,2) ) : ''
                        switchPowerOutputKwhDay.push(this.decimalHandle(item.switchPowerOutputKwh,1) )
                        batteryDischargeKwhDay.push(this.decimalHandle(item.batteryDischargeKwh,1) )
                        batteryChargeKwhDay.push(0-this.decimalHandle(item.batteryChargeKwh,1))
                        dcLoadKwhDay.push(this.decimalHandle(item.dcLoadKwh,1) )
                        electrovalence.push(item.electrovalence )
                    });
                    this.chartData = {
                            title:'储能状态统计图',
                            color:this.$store.state.station.stationInfo.isGf
                                  ? ['#5ad8a6','#b6b6b6','#5b8ff9','#ff9900','#33ff66','#d9001b']
                                  : ['#b6b6b6','#5b8ff9','#ff9900','#33ff66','#d9001b'],
                            legendData: this.$store.state.station.stationInfo.isGf
                                        ? ['光伏','开关电源直流输出','储能放电','储能充电','直流负载','电价']
                                        : ['开关电源直流输出','储能放电','储能充电','直流负载','电价'],
                            xDataName: '时间',
                            xData:res.xData,
                            yAxis:[
                                {
                                    name:'电量(kWh)',
                                    dataMin:Math.round(Math.min(...batteryChargeKwhDay) * 1.2),
                                    negativeHandle:true,   //Y轴负坐标按照正值显示
                                },
                                {
                                    name:'电价(元)',
                                    dataMax:Math.round(Math.max(...electrovalence) + 1)
                                },
                            ],
                            seriesData: [
                                {
                                    name: '光伏',
                                    type: 'bar',
                                    showBackground: true,
                                    unit: 'kWh',
                                    yAxisIndex:0,
                                    data: solarOutputKwhDay,
                                },
                                {
                                    name: '开关电源直流输出',
                                    type: 'bar',
                                    showBackground: true,
                                    unit: 'kWh',
                                    yAxisIndex:0,
                                    data:switchPowerOutputKwhDay,
                                },
                                {
                                    name: '储能放电',
                                    type: 'bar',
                                    showBackground: true,
                                    unit: 'kWh',
                                    yAxisIndex:0,
                                    data:batteryDischargeKwhDay
                                    
                                },
                                {
                                    name: '储能充电',
                                    type: 'bar',
                                    showBackground: true,
                                    unit: 'kWh',
                                    yAxisIndex:0,
                                    data:batteryChargeKwhDay
                                },
                                {
                                    name: '直流负载',
                                    type: 'line',
                                    smooth: true,
                                    unit: 'kWh',
                                    yAxisIndex:0,
                                    data:dcLoadKwhDay
                                },
                                {
                                    name: '电价',
                                    type: 'line',
                                    unit: '元',
                                    step: 'middle',
                                    yAxisIndex:1,
                                    data:electrovalence
                                },
                            ],
                            dataZoom: [
                                {
                                    type: 'inside',
                                    start: 0,
                                    end: 20
                                },
                            ]
                    }
                    !this.$store.state.station.stationInfo.isGf ? this.chartData.seriesData.shift() : ''
                    this.showChartData = this.chartData.xData.length ? true : false
                } else {
                    this.showChartData  = false
                }
            })
        },
        //刷新数据
        updateByDate() {
            this.getStorageInfo()
            this.getStorageChart()
        }

    }
}
</script>

<style lang="scss" scoped>
.energy-container {
    width: 100%;
    height: 100%;
    margin-top:10px;
    position: relative;
    .statistics-top1 {
        display: flex;
        justify-content: space-between;
        height: 22%;
        .box1 {
            width:18%;
        }
        .box2 {
            width:28%;
        }
        .box3 {
            width:43%;
        }
    }
    .statistics-top2 {
        display: flex;
        justify-content: space-between;
        height: 22%;
        .box1 {
            width:18%;
        }
        .box2 {
            width:28%;
        }
        .box3 {
            width:43%;
        }
    }
    .substation-pannel {
        padding-top:0 !important;
    }
    .info-content {
        margin-top:10px;
        padding:34px;
        height: 100%;
        display: flex;
        align-items: center;
        justify-content: space-between;
        >>> .el-form {
            .el-form-item__label {
                font-size: 18px;
                color: #fff;
                float: left;
            }
            .el-form-item__content {
                font-size: 20px;
                font-weight: 600;
                color: #fff;
                float:left;
            }
        }
    }
    .statistics-bottom {
        height: 75%;
        .trend-chart-box-inner {
            height: 600px;
        }
    }
    .trend-chart-container {
        line-height: 60px;
    }
}
</style>