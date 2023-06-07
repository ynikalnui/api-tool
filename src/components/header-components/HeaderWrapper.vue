<template>
    <div class="container">  
        <div>
            <HistoryMode v-if="dataMode === 'history'"/>     
            <RealTimeMode v-if="dataMode === 'realTime'"/>   
        </div>

        <div class="mode-btn-wrapper">
            <button @click="dataMode = 'history'" :class="dataMode === 'history' ? 'active-mode' : ''">History Mode</button>
            <button @click="dataMode = 'realTime'" :class="dataMode === 'realTime' ? 'active-mode' : ''">Real Time Mode</button>
        </div>
    </div>
    
</template>

<script>
import HistoryMode from "./HistoryMode.vue"
import RealTimeMode from "./RealTimeMode.vue"

export default{
    name: 'HeaderWrapper',
    components: {
        HistoryMode,
        RealTimeMode
    },
    data(){
        return{
            dataMode: "history"
        }
    }
}
</script>

<style lang="scss">
// mode select buttons 
.mode-btn-wrapper{
    display: flex;
    justify-content: space-between;
    width: 43%;
    margin: 20px auto 0;
    button{
        background: transparent;
        color: $black;
        border: 2px solid $black;
        padding: 10px 50px;
        border-radius: 20px;
        font-weight: bold;
        cursor: pointer;
        transition: .2s all ease;
        &.active-mode{
            color: #fff;
            background-color: $black;
        }
        @include responsive($xs){
            &:last-child{
                margin-top: 20px;
            }
        }
    }
    @include responsive($md){
        width: 70%;
    }
    @include responsive($sm){
        width: 90%;
    }
    @include responsive($xs){
        flex-direction: column;
    }
}
// different modes variables
.date-select{
    display: flex;
    width: 100%;
    justify-content: space-around;
    padding: 30px 0;
    @include responsive($md){
        justify-content: space-between;
    }
    @include responsive($sm){
        flex-direction: column;
        align-items: center;
    }
}
.download-button{
    background-color: $orange;
    border: none;
    border-radius: 20px;
    padding: 12px 10px;
    width: 30%;
    cursor: pointer;
    transition: .2s all ease;
    &:hover{
        background-color: $active-orange;
    }
    &:active{
        background-color: $orange;
    }
    @include responsive($sm){
        width: 250px;
        margin-top: 20px;
    }
}
.progress-line{
    width: 100%;
    background-color: $dark-blue;
    height: 40px;
    display: flex;
    justify-content: center;
    align-items: center;
    color: #fff;
    text-align: center;
    &.processing{
        background-color: $processing-green;
    }
    &.done{
        background-color: $green;
    }
    &.error-warning{
        background-color: $red;
    }
}
</style>