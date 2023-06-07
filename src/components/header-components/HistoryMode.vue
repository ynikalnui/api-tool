<template>
    <div class="container">  
        <div class="date-select">
            <div class="reset-date" id="startDatePicker">
                <flatPickr 
                v-model="startDate"
                :config="startDateConfig"
                placeholder="Select Start Date"
                class="date-input"
                />
            </div>
            <div class="reset-date" id="endDatePicker">
                <flatPickr 
                v-model="endDate"
                :config="endDateConfig"
                placeholder="Select End Date"
                class="date-input"
                />
            </div>
            <button class="download-button" @click="downloadHistoryData()">Download Selected</button>
        </div>
        <div class="progress-line" :class="progressLineClass">{{ progressLineText }}</div>
    </div>
</template>

<script>
import flatPickr from 'vue-flatpickr-component';
import 'flatpickr/dist/flatpickr.css';
import exportToCsv from "@/jsComponents/csvExport"
import formatDate from "@/jsComponents/formatDate"

export default{
    name: 'HistoryMode',
    components: {
        flatPickr
    },
    data(){
        return{
            startDate: formatDate(new Date(new Date().getTime()-(1*24*60*60*1000))),
            startDateConfig: {
                wrap: true,
                weekNumbers: true,
                defaultDate: new Date(new Date().getTime()-(1*24*60*60*1000))
            },
            endDate: formatDate(new Date()),
            endDateConfig: {
                wrap: true,  
                weekNumbers: true,
                defaultDate: "today"
            },
            maxDelta: 3600*24*2,
            processing: false,
            apiError: false,
            completedProcessing: false,
            errorText: ""
        }
    },
    methods: {
        downloadHistoryData: function() {
            if(this.delta > this.maxDelta || this.endTS<this.startTS){
                return
            }

            this.processing = true
            const url = `https://api.purpleair.com/v1/sensors/${process.env.VUE_APP_SENSOR_ID}/history?start_timestamp=${this.startTS}&end_timestamp=${this.endTS}&average=0&fields=${process.env.VUE_APP_FIELDS}`;
            
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

                exportToCsv(resp.data, `${this.startDate}_${this.endDate}.csv`)
                return true
            })
            .catch(error=>{
                this.processing = false
                this.apiError = true
                this.errorText = error
                return false
            })
            .finally(()=>{
                this.processing = false
                this.completedProcessing = true
            })
        }
    },
    computed: {
        startTS() {
            return parseInt(Date.parse(this.startDate))/1000
        },
        endTS(){
            return parseInt(Date.parse(this.endDate))/1000 + 86399
        },
        delta(){
            return this.endTS - this.startTS
        },
        progressLineClass() { 
            return {
                "error-warning": this.delta > this.maxDelta || this.endTS<this.startTS || this.apiError,
                "processing": this.processing,
                "done": this.completedProcessing
            }
        },
        progressLineText() {
            if(this.delta > this.maxDelta) {
                return "The maximum time range is one day. Please change the dates and try again."
            }
            if(this.endTS<this.startTS) {
                return "The end date can't be less than the start date. Please change the dates and try again."
            }
            if(this.apiError){
                return this.errorText
            }
            if(this.processing){
                return "Processing, please wait!"
            }
            if(this.completedProcessing){
                return "Done!"
            }
            return "Select dates please"
        }
    }
}
</script>

<style lang="scss">

.reset-date{
    width: 20%;
    @include responsive($sm){
        margin: 20px 0;
        width: auto;
    }
}
.date-input {
    border: 2px solid $white;
    border-radius: 30px;
    padding: 12px 10px;
    text-align: center;
    width: 100%;
    @include responsive($sm){
        width: 250px;
    }
}
</style>