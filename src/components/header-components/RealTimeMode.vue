<template>
    <div class="container">  
        <div class="date-select">
            <button class="download-button" @click="generateRealTimeData()">Start collecting data</button>
            <button class="stop-btn" @click="downloadGeneratedData()">Download collected data</button>
        </div>
        <div class="progress-line" :class="progressLineClass">{{ progressLineText }}</div>  
        <div class="interval-input">
            <input type="range" min="1" max="100" v-model="intervalValue" />
            {{ intervalValue }}
        </div>
        <vue-highcharts
        v-if="chartOptions"
        type="chart"
        :options="chartOptions"
        />
    </div>
</template>

<script>
import exportToCsv from "@/jsComponents/csvExport"

import VueHighcharts from 'vue2-highcharts';

export default{
    name: 'RealTimeMode',
    components: {
        VueHighcharts
    },
    data(){
        return{
            realTimeData: [],
            intervalValue: 1,
            myInterval: "",
            processing: false,
            apiError: false,
            completedProcessing: false,
            errorText: "",
            timeStamp: null,

            chartData: {
                chart: {
                    type: 'line',
                    maxWidth: 1000,
                    style: {
                        margin: "auto"
                    }
                },
                title: {
                    text: 'Number of project stars',
                },
                xAxis: {
                    categories: [0]
                },
                yAxis: {
                    title: {
                        text: 'Number of stars',
                    },
                },
                series: [
                    {
                        name: 'pm1.0_cf_1_a',
                        data: [0],
                    },
                    {
                        name: 'pm2.5_cf_1_a',
                        data: [0],
                    },
                    {
                        name: 'pm10.0_cf_1_a',
                        data: [0],
                    },
                    {
                        name: 'voc',
                        data: [0],
                    }
                ],
            }
        }
    }, 
    methods: {
        generateRealTimeData: function() {
            const vm = this
            this.processing = true

            const url = `https://api.purpleair.com/v1/sensors/${process.env.VUE_APP_SENSOR_ID}?fields=${process.env.VUE_APP_FIELDS}`
            

            this.myInterval = setInterval(getData, this.intervalValue*1000);

            function getData() {
                fetch(url, {
                    method: "GET",
                    withCredentials: true,
                    headers: {
                        "X-API-Key": process.env.VUE_APP_API_KEY
                    }
                })
                .then(resp => resp.json())
                .then(resp => {
                    if (resp&&resp.error) {
                        throw resp.description
                    }

                    vm.realTimeData.push({ timestamp: resp.time_stamp, data: resp.sensor })

                    vm.chartData.xAxis.categories.push(resp.time_stamp)

                    vm.chartData.series[0].data.push(resp.sensor["pm1.0_cf_1_a"])
                    vm.chartData.series[1].data.push(resp.sensor["pm2.5_cf_1_a"])
                    vm.chartData.series[2].data.push(resp.sensor["pm10.0_cf_1_a"])
                    vm.chartData.series[3].data.push(resp.sensor["voc"])

                    vm.timeStamp = new Date;
                    this.$set(vm,'timeStamp',new Date)
                })
                .catch(error=>{
                    this.processing = false
                    this.apiError = true
                    this.errorText = error
                    return false
                })
            }
        },
        downloadGeneratedData: function() {
            this.processing = false
            this.completedProcessing = true
            clearInterval(this.myInterval)
            const newData = this.realTimeData.map(el=>{
                const res = []
                res.push(el.timestamp);
                res.push(el.data["pm1.0_cf_1_a"]);
                res.push(el.data["pm2.5_cf_1_a"]);
                res.push(el.data["pm10.0_cf_1_a"]);
                res.push(el.data["voc"]);
                return(res)
            })
            
            return exportToCsv(newData, "data.csv")
        }
    },
    computed: {
        chartOptions() {
            return this.timeStamp && this.chartData
        },
        progressLineClass() { 
            return {
                "error-warning": this.apiError,
                "processing": this.processing,
                "done": this.completedProcessing
            }
        },
        progressLineText() {
            if(this.apiError){
                return this.errorText
            }
            if(this.processing){
                return "Collecting data"
            }
            if(this.completedProcessing){
                return "Done!"
            }
            return 'Press the "Start collecting data" button to start'
        }
    }
}
</script>

<style lang="scss">

.stop-btn{
    background-color: #4caf50;
    border: none;
    border-radius: 20px;
    padding: 12px 10px;
    width: 30%;
    cursor: pointer;
    transition: 0.2s all ease;
    &:hover{
        background-color: $darkgreen;
    }
    @include responsive($sm){
        width: 250px;
        margin-top: 20px;
    }
}
.interval-input{
    display: flex;
    justify-content: center;
    font-weight: bold;
    margin-top: 25px;
    input{
        cursor: pointer;
    }
}
</style>