<script setup>
import { ref, reactive, watchEffect, watch, computed, onMounted, onBeforeMount, onBeforeUpdate } from 'vue'

const cityInput= ref(JSON.parse(localStorage.getItem('cityInput')) || 'taipei') // 綁定input
const apiKey= ref('360b737778499306bbed550f52b61428')
const cityEn= ref('')
const cityZh= ref('')
const cityLat= ref('')
const cityLon= ref('')
const cityCountry= ref('')
const weatherDisplay= ref(true)
const errorMessage= ref('城市名稱有誤，請重新輸入')
const fetchData= reactive({})
const fourDayWeather= reactive([])
const firstDayWeather= reactive([])
const weatherTodayDisplay= ref(true)
const weatherMonth= ref('')
const weatherDate= ref('')
const weatherDay= ref('')
const weatherIcon= ref('')
const weatherDescription= ref('')
const weatherTempMax= ref('')
const weatherTempMin= ref('')
const weatherGrndLevel= ref('')
const weatherWind= ref('')
const weatherHumidity= ref('')
const weatherToggle= ref(false)
const props = defineProps({
    weatherToggle: Boolean,
})

watchEffect(()=>{
    weatherToggle.value= props.weatherToggle
})
watchEffect(()=>{
    localStorage.setItem('cityInput',JSON.stringify(cityInput.value))
})
const fetchCity=()=>{ //fetch得到城市的中英文名和國家資訊
    fetch(`https://api.openweathermap.org/geo/1.0/direct?q=${cityInput.value}&limit=5&appid=${apiKey.value}`)
    .then(response=>response.json())
    .then(data=>{
        if(!data.length || !data[0].local_names){
            fetchFail('城市名稱有誤，請重新輸入!')
        }else{
            cityEn.value = data[0].local_names.en
            cityZh.value = data[0].local_names.zh
            cityLat.value = data[0].lat
            cityLon.value = data[0].lon
            cityCountry.value = data[0].country
            cityInput.value = cityEn.value
            fetchWeather()
            getRandomImage()
        }
    })
}
const getRandomImage=()=>{
    const width = Math.round(Math.random()*(1600-1200)+1200)
    document.querySelector(":root").style.setProperty('--weatherBackground', `url('https://source.unsplash.com/random/${width}x800/?${cityEn.value},city') center center / cover `);
}
//fetch得到城市的天氣
const fetchWeather=()=>{
    fetch(`https://api.openweathermap.org/data/2.5/forecast?lat=${cityLat.value}&lon=${cityLon.value}&appid=${apiKey.value}&lang=zh_tw&units=metric`)
    .then(response=>response.json())
    .then(data=>{
        // console.log('2',data)
        if(data.cod=='404'){
            fetchFail('城市名稱有誤，請重新輸入!!')
            cityInput.value ='taipei'
            fetchCity()
        }else{
            dataFilter(data)
        }
    })
}
//如果返回某些訊息時的失敗處理
const fetchFail=(message)=>{
    weatherDisplay.value = false
    errorMessage.value= message
    setTimeout(() => {
        weatherDisplay.value = true
        errorMessage.value='城市名稱有誤，請重新輸入!!!'
        cityInput.value = cityEn.value
    }, 2000)
}
//處理fetch得到的天氣資料,且只要五天各一筆
const dataFilter=(data)=>{
    firstDayWeather.splice(0)
    fourDayWeather.splice(0)
    fetchData.city=data.city
    fetchData.cnt=data.cnt
    fetchData.cod=data.cod
    fetchData.list=data.list
    fetchData.message=data.message
    fetchData.list.forEach((item,index)=>{
        item.UTC8Date= transformUTC8(item.dt_txt)
        item.src = `https://openweathermap.org/img/wn/${item.weather[0].icon}@2x.png`
        if(index===0){  //只撈出第一筆的預報
            firstDayWeather.push(item)
        }
        if(index%8===0 && index!==0){  //撈出每天距離現在時間點最近的預報(02 05 08 11 14 17 20 23其一)
            fourDayWeather.push(item)
        }
    })
    setWeather(firstDayWeather[0],true)

}
const setWeather=(obj,show)=>{
    weatherTodayDisplay.value = show
    weatherMonth.value= obj.UTC8Date.month
    weatherDate.value= obj.UTC8Date.date
    weatherDay.value= dayTransfer(obj.UTC8Date.day)
    weatherIcon.value= obj.src
    weatherDescription.value= obj.weather[0].description
    weatherTempMax.value= Math.ceil(obj.main.temp_max)
    weatherTempMin.value= Math.floor(obj.main.temp_min)
    weatherGrndLevel.value= obj.main.grnd_level
    weatherWind.value= obj.wind.speed
    weatherHumidity.value= obj.main.humidity
}
//處理時區問題
const transformUTC8=(dt_txt)=>{
    dt_txt = dt_txt.replace(/\-/g, "/")
    const getTime = new Date(dt_txt).getTime()+8*60*60*1000
    const UTC8 = new Date(getTime)
    return {
        year: UTC8.getFullYear(),
        month: UTC8.getMonth()+1,
        date: UTC8.getDate(),
        hours: UTC8.getHours(),
        day: UTC8.getDay()
    } 
}
const dayTransfer=(day)=>{
    const week=['Sun','Mon','Tue','Wed','Thu','Fri','Sat']
    return week[day]
}
fetchCity()
</script>

<template>
    <div class="weather" v-if="weatherDisplay" :class="{show: weatherToggle}">
        <div class="weatherToday" v-for="item in firstDayWeather" >
            <div class="detailInfo1">
                <div class="locationInput">
                    <i class="fa-solid fa-location-dot"></i>
                    <input type="text" name="" id="" v-model="cityInput" @keyup.enter="fetchCity" @vnode-mounted="({ el }) => el.focus()">
                </div>
                <div class="city">{{ cityZh }}, {{ cityCountry }} <i @click="getRandomImage" class="fa-regular fa-image"></i></div>
            </div>
            <transition name="weatherTodayDisplay" @after-leave="weatherTodayDisplay=true">
            <div class="weatherTodayDisplay" v-show="weatherTodayDisplay">
                <div class="detailInfo2">
                    <div class="time">{{ weatherMonth }}/{{ weatherDate }}, {{ weatherDay }}</div>
                    <div class="wrap">
                        <div class="icon"><img :src="weatherIcon" alt=""></div>
                        <div class="description">{{ weatherDescription }}</div>
                    </div>
                    <div class="temp">
                        <div class="maxTemp">{{ weatherTempMax }}°</div>
                        <div class="minTemp">{{ weatherTempMin }}°</div>
                    </div>
                </div>
                <div class="detailInfo3">
                    <div class="grndLevel"><i class="fa-solid fa-hurricane"></i>氣壓：{{ weatherGrndLevel }}hPa</div>
                    <div class="wind"><i class="fa-solid fa-wind"></i>風速：{{ weatherWind }}公尺/秒</div>
                    <div class="humidity"><i class="fa-solid fa-water"></i>濕度：{{ weatherHumidity }}%</div>
                </div>
            </div>
            </transition>
        </div>
        <div class="weatherWeek" v-for="item in fourDayWeather" @mouseenter="setWeather(item,true)" @click="setWeather(item)" @mouseleave="setWeather(firstDayWeather[0],false)">
            <div class="weatherDate">
                <div class="time">{{ dayTransfer(item.UTC8Date.day)}}</div>
                <div class="icon"><img :src="item.src" alt=""></div>
                <div class="temp">
                    <div class="maxTemp">{{ Math.ceil(item.main.temp_max )}}°</div>
                    <div class="minTemp">{{ Math.floor(item.main.temp_min )}}°</div>
                </div>
            </div>
        </div>
    </div>
    <div class="weatherErr" v-else-if="!weatherDisplay">{{ errorMessage }}
    </div>
</template>

<style lang="scss" scoped>
@mixin tablet-768{
    @media(max-width:768px){
        @content;
}
    }
@mixin mobile-576{
    @media(max-width:576px){
        @content;
    }
}
$bgColor-main: rgb(238, 239, 244);
$button-shadow: rgb(188, 189, 194);
$bubble-personal-color: rgb(108, 154, 139);
$bubble-business-color: rgb(232, 153, 141);
$create-button-color: rgb(243, 87, 170);
$edit-button-color: rgb(96, 77, 83);
$delete-button-color: rgb(162, 44, 41);
$button-text-color: white;
$lable-color: white;

.weather{
    -webkit-text-stroke: #fff 0.8px;
    -webkit-text-fill-color:slategrey;
    flex: 0 0 auto;
    height: 0px;
    display: flex;
    margin: 20px 0 20px 0;
    border-radius: 20px;
    box-shadow: none;
    transition: .2s;
    position: relative;
    &:hover::after{
        filter: blur(5px);
        @include mobile-576{
            filter: blur(0px);
        }
    }
    &::after{
        content: '';
        position: absolute;
        top: 0;
        left: 0;
        width: 100%;
        height: 100%;
        border-radius: 20px;
        background: var(--weatherBackground);
        filter: blur(1px);
        opacity: 0.8;
        z-index: -1;
        transition: 1s;
        @include mobile-576{
            filter: blur(0px);
        }
    }
    &.show{
        display: flex;
        flex: 0 0 auto;
        height: 150px;
        transition: .5s;
        box-shadow: 0 0 3px 3px white;
        .weatherToday{
            display: flex;
        }
        .weatherWeek{
            display: flex;
            @include mobile-576{
                display: none;
            }
        }
    }
    .weatherToday{
        min-width: 0;
        flex: 7;
        display: none;
        border-radius: 20px;
        margin: 5px;
        padding: 5px 10px 5px 10px;
        color: white;
        .detailInfo1{
            flex: 1;
            display: flex;
            flex-direction: column;
            padding-left: 5px;
            .locationInput{
                display: flex;
                justify-content: flex-start;
                align-items: center;
                margin: 5px 0px 5px 0;
                border-bottom: 2px white solid;
                i{
                    transform: scale(1.5);
                }
                input[type="text"] {
                    width: 100%; 
                    border: none;
                    padding: 5px;
                    height: 30px;
                    background-color: transparent;
                    outline: none;
                    color: white;
                    font-size: 20px;
                    white-space: nowrap;
                }
            }
            .city{
                font-size: 16px;
                white-space: nowrap;
                i{
                    cursor: pointer;
                }
            }
            
        }
        .weatherTodayDisplay{
            flex: 2;
            display: flex;
            @include tablet-768{
                flex: 1;
            }
            .detailInfo2{
                position: relative;
                min-width: 0;
                flex: 1;
                display: flex;
                justify-content: center;
                align-items: center;
                margin-right: 30px;
                flex-direction: column;
                padding-left: 30px;
                .time{
                    padding-left: 10px;
                    font-size: 30px;
                    position: relative;
                    top: 30px;
                    white-space: nowrap;
                }
                .wrap{
                    display: flex;
                    justify-content: flex-start;
                    align-items: center;                    
                    .icon{
                        width: 100px;
                        img{
                            object-fit: contain;
                            width: 100%;
                            filter: drop-shadow(0 0 1px #ffffff);
                        }
                    }
                    .description{
                        white-space: nowrap;
                    }
                }
                .temp{
                    display: flex;
                    font-size: 30px;
                    position: relative;
                    top: -30px;
                    .maxTemp{
                        margin-right: 10px;
                    }
                }
            }
            .detailInfo3{
                flex: 1;
                display: flex;
                flex-direction: column;
                justify-content: center;
                align-items: flex-start;
                padding-left: 10px;
                font-size: 16px;
                white-space: nowrap;
                @include tablet-768{
                    display: none;
                }
                div{
                    margin: 2px 0 2px 0;
                }
            }
        }
    }
    .weatherWeek{
        display: none;
        flex: 1;
        @include tablet-768{
            flex-direction: column;
            min-width: 0;
        }
        .weatherDate{
            flex: 1;
            display: flex;
            flex-direction: column;
            justify-content: center;
            align-items: center;
            background-color: rgba(255, 255, 255, 0.2);
            backdrop-filter: blur(2px);
            border-radius: 20px;
            margin: 5px;
            padding: 5px;
            color: white;
            .icon{
                width: 50px;
                img{
                    width: 50px;
                    object-fit: cover;
                    filter: drop-shadow(0 0 1px #ffffff);
                }
            }
            .temp{
                display: flex;
                flex-direction: column;
                color: white;
                
            }
        }
    }
}
.weatherErr{
    display: flex;
    justify-content: center;
    align-items: center;
    height: 150px;
    width: 100%;
    font-size: 20px;
    color: white;
    margin: 20px 0 20px 0;
    background: linear-gradient(90deg, rgba(2,0,36,0.3) 0%, rgba(9,9,121,0.3) 35%, rgba(0,212,255,0.3) 100%);
    border-radius: 20px;
}
</style>