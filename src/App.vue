<script setup>
import { reactive, ref, watch, watchEffect, computed, onMounted, onUpdated, onBeforeUpdate} from 'vue';
import todolist from './components/todolist.vue'
import weather from './components/weather.vue'
// demo資料
// import todosDemo from '/json/todosDemo.json'
// 節氣資料
import solarTerms from '/json/solarTerms.json'
// todosDemo.forEach(todo => todo.month=new Date().getMonth()); //保持在當下月份
const message = reactive({})
const showCalendar = ref(true)
const showCalendarName = ref('')
const todayCalendar = reactive({}) //當下月份
const calendar = reactive(JSON.parse(localStorage.getItem('calendar') ||'{}')) //萬年曆查詢月份
const today = new Date()
watchEffect(()=>{
    // localStorage.setItem('todos', JSON.stringify(todosDemo))
    localStorage.setItem('calendar',JSON.stringify(calendar))
})
const originalDate=()=>{  //初始化今天的日期
    todayCalendar.year= today.getFullYear()
    todayCalendar.month= today.getMonth()
    todayCalendar.date= today.getDate()
    todayCalendar.day= today.getDay()
    if (Object.keys(calendar).length === 0 && calendar.constructor === Object){ //判斷calendar是否為空物件
        calendar.year= today.getFullYear()
        calendar.month= today.getMonth()
        calendar.date= today.getDate()
        calendar.day= today.getDay()
    }
}
// 把點擊到萬年曆的日期給todolist子組件
const fortodolist = reactive({
    year: '',
    month: '',
    date: '',
    day: ''
})

const clickDate=({year, month, date, day, lunar_year, lunar_month, lunar_date})=>{
    fortodolist.year= year
    fortodolist.month= month
    fortodolist.date= calendar.date= date
    fortodolist.day= calendar.day= day
    fortodolist.lunar_year= lunar_year
    fortodolist.lunar_month= lunar_month
    fortodolist.lunar_date= lunar_date
    listToggle.value = true
}
//將todolist子組件和萬年曆的顯示日期都回到當天日期
const backToday=()=>{
    showCalendar.value = false
    fortodolist.year= calendar.year= today.getFullYear()
    fortodolist.month= calendar.month= today.getMonth() //1月-12月 => 0-11
    fortodolist.date= calendar.date=today.getDate()
    fortodolist.day= calendar.day=today.getDay() //星期日-六 => 0-6
}
//切換萬年曆月份
const setMonth=(change)=>{
    showCalendarName.value = change === 1 ? 'nextCalendar' : 'prevCalendar'
    showCalendar.value = false
    calendar.month += change
    if(calendar.month >= 12){
        calendar.year += 1
        calendar.month = 0;
    }else if(calendar.month < 0){
        calendar.year -= 1
        calendar.month = 11;
    }
}
//切換萬年曆年份
const setYear=(change)=>{
    showCalendarName.value = change === 1 ? 'nextYCalendar' : 'prevYCalendar'
    showCalendar.value= false
    calendar.year+= change
    if(calendar.year <= 1)calendar.year = 1
}
//動畫完成後再切回來
const show=()=>{
    showCalendar.value= true
}

const lunar_date_transfer = (date) =>{
    const arr = ['','初一','初二','初三','初四','初五','初六','初七','初八','初九','初十',
            '十一','十二','十三','十四','十五','十六','十七','十八','十九','二十',
            '廿一','廿二','廿三','廿四','廿五','廿六','廿七','廿八','廿九','三十']
    return arr[date]
}

//萬年曆最終顯示的"42日" (以星期日為第一欄為基準做調整)
const calendarDisplay = computed(()=>{ 
    const datas = reactive([]);
    const Whatday = new Date(calendar.year, calendar.month, 1).getDay() //得到目前月份的1號是星期幾 //得0~6
    const fisrtDateFix = new Date(calendar.year, calendar.month, 1 - Whatday) //修正目前月份的日曆最左上角的日期
    for (let i= 0; i< 42; i++) { //從最左上角日期往後推42個
        const date = new Date(fisrtDateFix.getFullYear(), fisrtDateFix.getMonth(), fisrtDateFix.getDate() + i)
        datas.push({
            year: date.getFullYear(),
            month: date.getMonth(),
            date: date.getDate(),
            day: date.getDay(),
            // lunar: new Date(fisrtDateFix.getFullYear(), fisrtDateFix.getMonth(), fisrtDateFix.getDate() + i).toLocaleString('zh-CN-u-ca-chinese').replace(/(\d+)\s*?年/, (_,y)=>"甲乙丙丁戊己庚辛壬癸".charAt((y-4)%10)+ "子丑寅卯辰巳午未申酉戌亥".charAt((y-4)%12)).replace('月','月 ').split(' '),  // ['癸卯十一月', '14', '00:00:00']
            lunar_year: new Date(fisrtDateFix.getFullYear(), fisrtDateFix.getMonth(), fisrtDateFix.getDate() + i).toLocaleString('zh-CN-u-ca-chinese').replace(/(\d+)\s*?年/, (_,y)=>"甲乙丙丁戊己庚辛壬癸".charAt((y-4)%10)+ "子丑寅卯辰巳午未申酉戌亥".charAt((y-4)%12)).replace('月','月 ').split(' ')[0].slice(0,2), //'癸卯'
            lunar_month: new Date(fisrtDateFix.getFullYear(), fisrtDateFix.getMonth(), fisrtDateFix.getDate() + i).toLocaleString('zh-CN-u-ca-chinese').replace(/(\d+)\s*?年/, (_,y)=>"甲乙丙丁戊己庚辛壬癸".charAt((y-4)%10)+ "子丑寅卯辰巳午未申酉戌亥".charAt((y-4)%12)).replace('月','月 ').split(' ')[0].slice(2,5), //'十一月'
            lunar_date: lunar_date_transfer(new Date(fisrtDateFix.getFullYear(), fisrtDateFix.getMonth(), fisrtDateFix.getDate() + i).toLocaleString('zh-CN-u-ca-chinese').replace(/(\d+)\s*?年/, (_,y)=>"甲乙丙丁戊己庚辛壬癸".charAt((y-4)%10)+ "子丑寅卯辰巳午未申酉戌亥".charAt((y-4)%12)).replace('月','月 ').split(' ')[1]),  //'14'
            todo:[],
            count: {
                totle: 0,
                personal: {
                    done: 0,
                    undone: 0,
                },
                business: {
                    done: 0,
                    undone: 0,
                }
            }
        })
    }
    return datas
})
//子組件按新增後觸發該事件  
const updateTodoInCalendar = () =>{
    const todos = reactive(JSON.parse(localStorage.getItem('todos')) || todosDemo)
    calendarDisplay.value.forEach(date=>{
        date.todo=[]
        date.solar=''
        date.count.totle = date.count.personal.undone = date.count.personal.done = date.count.business.undone = date.count.business.done = 0
    })
    calendarDisplay.value.forEach(date=>{
        // 如果todos資料年月日與月曆相符 就在calendarDisplay加上.todo物件
        todos.forEach(todo => {
            if (todo.year===date.year && todo.month===date.month && todo.date===date.date){
                date.todo.push({
                    id: todo.id,
                    contant: todo.contant,
                    category: todo.category,
                    done: todo.done,
                })
                date.count.totle +=1
                if (todo.category==='personal' && todo.done===false){
                    date.count.personal.undone +=1
                }else if (todo.category==='personal' && todo.done===true) {
                    date.count.personal.done +=1
                }else if(todo.category==='business' && todo.done===false){
                    date.count.business.undone +=1
                }else if (todo.category==='business' && todo.done===true) {
                    date.count.business.done +=1
                }
            }
        })
        // 如果節氣資料年月日與月曆相符 就在calendarDisplay加上
        solarTerms.forEach(solar => {
            if (date.year===solar.year && date.month===solar.month-1 && date.date===solar.date){
                date.solar=solar.contant
            }
        })
    })
}
// -------------綁class的-------------
const prevYear = ref(false)
const nextYear = ref(false)
const prevMonth = ref(false)
const nextMonth = ref(false)

//處理list的狀態相關
const listToggle = ref(JSON.parse(localStorage.getItem('listToggle')) || false)
watchEffect(()=>{
    localStorage.setItem('listToggle',JSON.stringify(listToggle.value))
})
const listClose=()=>{
    listToggle.value= false
}
//處理weather的狀態相關
const weatherToggle = ref(JSON.parse(localStorage.getItem('weatherToggle')) || false)
watchEffect(()=>{
    localStorage.setItem('weatherToggle',JSON.stringify(weatherToggle.value))
})
//判斷萬年曆顯示月份是否為當下月份 做出相對應樣式
const todayToggle = ref(false)
const checkToday = watch(calendar,(newCalendar)=>{
    if(newCalendar.year !== todayCalendar.year || newCalendar.month !== todayCalendar.month || newCalendar.date !== todayCalendar.date){
        todayToggle.value = false
    }else if (newCalendar.year === todayCalendar.year && newCalendar.month === todayCalendar.month && newCalendar.date === todayCalendar.date) {
        todayToggle.value = true
    }
})
//判斷萬年曆顯示的"42日"是否為當月(找出不是當月的頭尾)
const isCurrentMonth=(i,j)=> calendarDisplay.value[(i-1)*7+(j-1)].year === calendar.year && calendarDisplay.value[(i-1)*7+(j-1)].month === calendar.month ? true : false
//判斷萬年曆顯示的"42日"是否為當日(找出當月當日)
const isCurrentDate=(i,j)=> calendarDisplay.value[(i-1)*7+(j-1)].year === todayCalendar.year && calendarDisplay.value[(i-1)*7+(j-1)].month === todayCalendar.month && calendarDisplay.value[(i-1)*7+(j-1)].date === todayCalendar.date ? true : false
//判斷萬年曆顯示的"42日"是否為點到的日期
const isClickDate=(i,j)=> calendarDisplay.value[(i-1)*7+(j-1)].year === fortodolist.year && calendarDisplay.value[(i-1)*7+(j-1)].month === fortodolist.month && calendarDisplay.value[(i-1)*7+(j-1)].date === fortodolist.date ? true : false
// -------------綁class的邏輯-------------
const showMessage=(Message)=>{
    message.status= true
    message.content= Message
}
onMounted(()=>{
    originalDate(); //將目前月份 預設為當天的月份
    updateTodoInCalendar() //如果todos資料年月日與月曆相符 就在calendarDisplay加上.todo物件
    if(calendar.year === todayCalendar.year && calendar.month === todayCalendar.month && calendar.date === todayCalendar.date) todayToggle.value = true
})
onUpdated(()=>{
    updateTodoInCalendar()
})

//判斷手機使用者手勢
let startX =''
let startY =''
let endX =''
let endY =''
const touchstartHandler=(event)=>{
    startX = event.touches[0].clientX
    startY = event.touches[0].clientY
}
const touchendHandler=(event)=>{
    endX = event.changedTouches[0].clientX
    endY = event.changedTouches[0].clientY
    if (Math.abs(Math.abs(startX-endX)-Math.abs(startY-endY)) < 20){
        // console.log('滑動距離過短(無效)')
        return
    }
    if (Math.abs(startX-endX)>Math.abs(startY-endY)){
        if(startX-endX >0 ){
            // console.log('向左滑')
            setMonth(+1)
        }else{
            // console.log('向右滑')
            setMonth(-1)
        }
    }
    if (Math.abs(startX-endX)<Math.abs(startY-endY)){
        if(startY-endY >0 ){
            // console.log('向上滑')
            setYear(+1)
        }else{
            // console.log('向下滑')
            setYear(-1)
        }
    }
}

</script>

<template>
    <div class="wrap">
        <Transition name="showMessage" @after-enter="message.status= false">
        <div class="message" v-show="message.status">{{ message.content }}</div>
        </Transition>
        <div class="leftMain">
            <div class="calendarMain">
                <div class="toolAll">
                    <div class="yearTool">
                        <div class="tool setYearToggle" @click="setYear(-1)" @mousedown="prevYear=!prevYear" @mouseup="prevYear=!prevYear" :class="{active: prevYear}"><i class="fa-regular fa-calendar-minus"></i></div>
                        <div class="tool setYearToggle" @click="setYear(+1)" @mousedown="nextYear=!nextYear" @mouseup="nextYear=!nextYear" :class="{active: nextYear}"><i class="fa-regular fa-calendar-plus"></i></div>
                        <div class="tool" @click="backToday" :class="{active: todayToggle}"><i class="fa-solid fa-clock-rotate-left"></i></div>
                    </div>
                    <div class="monthTool">
                        <div class="setMonthToggle" @click="setMonth(-1)" @mousedown="prevMonth=!prevMonth" @mouseup="prevMonth=!prevMonth" :class="{active: prevMonth}"><i class="fa-solid fa-caret-left"></i></div>
                        <h1>{{ calendar.year }}年{{ calendar.month +1 }}月</h1>
                        <div class="setMonthToggle" @click="setMonth(+1)" @mousedown="nextMonth=!nextMonth" @mouseup="nextMonth=!nextMonth" :class="{active: nextMonth}"><i class="fa-solid fa-caret-right"></i></div>
                    </div>
                    <div class="otherTool">
                        <div class="tool" @click="weatherToggle=!weatherToggle" :class="{active: weatherToggle}"><i class="fa-solid fa-cloud-sun"></i></div>
                        <div class="tool listToggle" @click="listToggle=!listToggle" :class="{active: listToggle}"><i class="fa-solid fa-list-check"></i></div>
                    </div>
                </div>
                <Transition :name="showCalendarName" @after-leave="showCalendar= true">
                <div class="calendar" v-if="showCalendar" @touchstart="touchstartHandler" @touchend="touchendHandler">
                    <div class="weekTitle">
                        <div class="wday">Sun</div>
                        <div class="wday">Mon</div>
                        <div class="wday">Tue</div>
                        <div class="wday">Wed</div>
                        <div class="wday">Thu</div>
                        <div class="wday">Fri</div>
                        <div class="wday">Sat</div>
                    </div>
                    <div class="weeks">
                        <div class="week" v-for="i in 6">
                            <div class="day" :class="{ isCurrentMonth: isCurrentMonth(i,j), isClickDate: isClickDate(i,j) }" v-for="j in 7" @click="clickDate(calendarDisplay[(i-1)*7+(j-1)])">
                                <div class="date" :class="{ isCurrentDate: isCurrentDate(i,j)}">{{ calendarDisplay[(i-1)*7+(j-1)].date }} 
                                    <div v-if="calendarDisplay[(i-1)*7+(j-1)].solar" class="solar">{{ calendarDisplay[(i-1)*7+(j-1)].solar }}</div>
                                    <div v-else class="lunar">{{ calendarDisplay[(i-1)*7+(j-1)].lunar_date=='初一'? calendarDisplay[(i-1)*7+(j-1)].lunar_month : calendarDisplay[(i-1)*7+(j-1)].lunar_date }}</div>
                                </div>
                                <span class="undone" v-if="calendarDisplay[(i-1)*7+(j-1)].count.personal.undone+calendarDisplay[(i-1)*7+(j-1)].count.business.undone">{{ calendarDisplay[(i-1)*7+(j-1)].count.personal.undone+calendarDisplay[(i-1)*7+(j-1)].count.business.undone }}</span>
                                <span class="personalDone" v-if="calendarDisplay[(i-1)*7+(j-1)].count.personal.done">{{ calendarDisplay[(i-1)*7+(j-1)].count.personal.done }}</span>
                                <span class="businessDone" v-if="calendarDisplay[(i-1)*7+(j-1)].count.business.done">{{ calendarDisplay[(i-1)*7+(j-1)].count.business.done }}</span>
                                <ul>
                                    <!-- <li v-if="calendarDisplay[(i-1)*7+(j-1)].solar" class="solar"> {{ calendarDisplay[(i-1)*7+(j-1)].solar }}</li> -->
                                    <li :class="{personal: item.category==='personal', business: item.category==='business', done: item.done}" v-for="(item, index) in calendarDisplay[(i-1)*7+(j-1)].todo">{{ item.contant }}</li>
                                </ul>
                            </div>
                        </div>
                    </div>
                </div>
                </Transition>
            </div>
            <weather :weatherToggle="weatherToggle" />
        </div>
        <todolist :clickDate="fortodolist" :listToggle="listToggle" @response="updateTodoInCalendar" @listClose="listClose" @message="showMessage"/>
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

$bgColor: rgb(238, 239, 244);
$button-shadow: rgb(188, 189, 194);
$button-light: rgb(250, 251, 255);
$textColor: rgb(20, 20, 20);
$bubble-personal-color: rgb(108, 154, 139);
$bubble-business-color: rgb(232, 153, 141);
$today-color: rgb(161, 104, 58);
$clickDate-color: rgb(255, 238, 224);

.wrap{
    display: flex;
    background: var(--background);
    height: 100vh;
    max-height: -webkit-fill-available;
    overflow: hidden;
    position: relative;
    .leftMain{
        min-width: 0;
        flex: 5;
        display: flex;
        flex-direction: column;
        position: sticky;
        top: 0;
        padding: 0 20px 0 20px;
        .calendarMain{
            order: 1;
            display: flex;
            flex-direction: column;
            flex: 1 1;
            margin-bottom: 50px;
            .toolAll{
                display: flex;
                justify-content: space-evenly;
                align-items: center;
                margin: 5px 0 10px 0;
                .yearTool{
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    .setYearToggle{
                        @include tablet-768{
                            display: none;
                        }
                    }
                    i:hover{
                        transform: scale(3);
                    }
                }
                .otherTool{
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    position: relative;
                    .listToggle{
                        @include tablet-768{
                            display: none;
                        }
                    }
                    i:hover{
                        animation: hover 2s;
                        @keyframes hover {
                            10%,30%{transform: rotate(20deg) scale(2)}
                            20%,40%{transform: rotate(-20deg) scale(2)}
                            0%,50%,100%{transform: rotate(0deg) scale(2)}
                        }
                    }
                }
                .monthTool{
                    flex: 1;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    @include mobile-576{
                        margin: 0 10px 0 10px;
                    }
                    .setMonthToggle{
                        cursor: pointer;
                        transform: scale(2);
                        transition: .3s;
                        &:hover{
                            transform: scale(3);
                        }
                    }
                    h1{
                        width: 180px;
                        text-align: center;
                        @include mobile-576{
                            width: 140px;
                            font-size: 24px;
                        }
                    }
                }
                .tool{
                    flex: 0 0 auto;
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    margin: 0 3px 0 3px;
                    width: 60px;
                    height: 60px;
                    border: 5px solid $button-shadow;
                    border-radius: 50%;
                    font-size: 5px;
                    font-weight: 800;
                    background-color: $bgColor;
                    box-shadow: 0 0 5px 1px $button-shadow inset;
                    cursor: pointer;
                    user-select: none;
                    transition: .5s;
                    @include tablet-768{
                        border: 3px solid $button-shadow;
                        width: 8vw;
                        height: 8vw;
                    }
                    i{
                        transform: scale(2);
                        transition: .5s;
                        color: $button-shadow;
                        @include tablet-768{
                            transform: scale(4);
                        }
                        &:hover{
                            @include tablet-768{
                                animation: none;
                            }
                        }
                    }
                    &.active{
                        border: 5px solid white;
                        border-radius: 50%;
                        font-size: 5px;
                        font-weight: 800;
                        background-color: $bgColor;
                        box-shadow: 0 0 10px 5px $button-light;
                        @include tablet-768{
                            box-shadow: 0 0 10px 0.1px $button-light;
                        }
                        i{
                            transition: .5s;
                            color: white;
                            @include tablet-768{
                                transform: scale(4);
                            }
                        }
                    }
                }
            }
            .calendar{
                flex: 1;
                display: flex;
                flex-direction: column;
                backdrop-filter: blur(2px);
                border: 3px solid $button-shadow;
                border-radius: 20px;
                box-shadow: 3px 3px 3px 3px $button-shadow;              
                .weekTitle{
                display: flex;
                justify-content: center;
                    .wday{
                        flex: 1;
                        text-align: center;
                        padding: 5px 0 5px 0;
                        background-color: rgb(213, 233, 250, 0.5);
                        border: 1px solid $button-shadow;
                        &:first-child{
                            background-color: rgb(255, 196, 187);
                            border-radius: 20px 0 0 0;
                        }
                        &:last-child{
                            background-color: rgb(208, 255, 187);
                            border-radius: 0 20px 0 0;
                        }
                    }
                }
                .weeks{
                    flex: 1;
                    display: flex;
                    flex-direction: column;
                    border-radius: 0 0 20px 20px;
                    cursor: pointer;
                    .week{
                        flex: 1 1;
                        display: flex;
                        overflow: hidden;
                        max-height: 250px;
                        &:last-child{
                            border-radius: 0 0 20px 20px;
                            .day:first-child{
                                border-radius: 0 0 0 20px;
                            }
                            .day:last-child{
                                border-radius: 0 0 20px 0;
                            }
                        }
                        .day{
                            flex: 1;
                            border: 0.1px solid $button-shadow;
                            min-width: 0;
                            background-color:rgba(230, 229, 229, 0.5);
                            transition: all .3s .1s ease-in-out;
                            .date{
                                display: inline-block;
                                position: sticky;
                                background-color:rgb(230, 229, 229, 0.5);
                                top: 0;
                                padding: 3px;
                                margin-right: 5px;
                                margin-bottom: 5px;
                                width: 40px;
                                text-align: center;
                                border-bottom: 1px solid $button-shadow;
                                border-right: 1px solid $button-shadow;
                                border-bottom-right-radius: 20%;
                                transition: all .3s .1s ease-in-out;
                                @include tablet-768{
                                    display: block;
                                }
                                @include mobile-576{
                                }
                                &.isCurrentDate{
                                    background-color: $today-color !important ;
                                    color: white;
                                }
                                .solar{
                                    font-size: 10px;
                                    font-weight: 600;
                                    background: rgb(254, 183, 183);
                                    border-radius: 20px;
                                    color: white;
                                }
                                .lunar{
                                    font-size: 10px;
                                    color: gray;
                                }
                            }
                            span{
                                display: inline-block;
                                margin-left: 2px;
                                width: 16px;
                                height: 16px;
                                padding: 1px;
                                position: sticky;
                                top: 30px;
                                background-color: $button-shadow;
                                color: white;
                                border-radius: 50%;
                                text-align: center;
                                line-height: 16px;
                                font-size: 16px;
                                @include tablet-768{
                                    top: 39px;
                                }
                                @include mobile-576{
                                    width: 12px;
                                    height: 12px;
                                    font-size: 12px;
                                    line-height: 12px;
                                }
                                &.undone{
                                    background-color: $button-shadow;
                                }
                                &.personalDone{
                                    background-color: $bubble-personal-color;
                                }
                                &.businessDone{
                                    background-color: $bubble-business-color;
                                }
                            }
                            li{
                                padding: 0 5px 0 5px;
                                text-overflow: ellipsis;
                                white-space: nowrap;
                                overflow: hidden;
                                font-size: 16px;
                                @include mobile-576{
                                    display: none;
                                }
                                &.personal{
                                    color: $bubble-personal-color;
                                }
                                &.business{
                                    color: $bubble-business-color;
                                }
                                &.done{
                                    text-decoration: line-through;
                                }
                                // &.solar{
                                //     background: rgb(254, 183, 183);
                                //     border-radius: 20px;
                                //     margin: 0 5px;
                                //     color: white;
                                //     font-size: 10px;
                                // }
                            }
                            &.isCurrentMonth{
                                background-color: white;
                                .date{
                                    background-color:white;
                                }
                            }
                            &.isClickDate{
                                background-color: $clickDate-color;
                                overflow-y: overlay;
                                z-index: 10;
                                .date{
                                    background-color: $clickDate-color;
                                    transition: all .3s .5s ease-in-out;
        
                                }
                            }
                            &:hover{
                                overflow-y: overlay;
                                transition: all .3s .5s ease-in-out;
                            }
                        }
                    }
                }
            }
        }
    }
    .message{
        width: 30%;
        height: 50px;
        background: linear-gradient(161deg, rgba(34,193,195,1) 0%, rgba(253,187,45,1) 100%);
        position: absolute;
        border-radius: 20px;
        text-align: center;
        color: white;
        line-height: 50px;
        font-size: 20px;
        top: 50px;
        left: 0;
        right: 0;
        margin: auto;
        z-index: 100;
        @include tablet-768{
            width: 100%;
            border-radius: 0;
            top: 0;
        }
    }
}
// 切換行事曆動畫
.nextCalendar-enter-active, .nextCalendar-leave-active, .prevCalendar-enter-active, .prevCalendar-leave-active, .nextYCalendar-enter-active, .nextYCalendar-leave-active, .prevYCalendar-enter-active, .prevYCalendar-leave-active {
    transition: all 0.3s ease;
}
// 切換行事曆動畫-往左滑動
.nextCalendar-enter-from {
    opacity: 0;
    transform: translateX(150px);
    filter: blur(2px);
}
.nextCalendar-leave-to {
    opacity: 0;
    transform: translateX(-150px);
    filter: blur(2px);
}
// 切換行事曆動畫-往右滑動
.prevCalendar-enter-from {
    opacity: 0;
    transform: translateX(-150px);
    filter: blur(2px);
}
.prevCalendar-leave-to {
    opacity: 0;
    transform: translateX(150px);
    filter: blur(2px);
}
// 切換行事曆動畫-往上滑動

.nextYCalendar-enter-from {
    opacity: 0;
    transform: translateY(150px);
    filter: blur(2px);
}
.nextYCalendar-leave-to {
    opacity: 0;
    transform: translateY(-150px);
    filter: blur(2px);
}
// 切換行事曆動畫-往下滑動
.prevYCalendar-enter-from {
    opacity: 0;
    transform: translateY(-150px);
    filter: blur(2px);
}
.prevYCalendar-leave-to {
    opacity: 0;
    transform: translateY(150px);
    filter: blur(2px);
}
// 新增刪除成功提示動畫
.showMessage-enter-active {
    transition: all 1s ease;
}
.showMessage-leave-active {
    transition: all 10s ease;
}
.showMessage-enter-from, .showMessage-leave-to {
    opacity: 0;
}
.showMessage-enter-to, .showMessage-leave-from{
    opacity: 1;
}
</style>