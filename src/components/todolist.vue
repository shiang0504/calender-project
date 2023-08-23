<script setup>
import { ref, reactive, watchEffect, watch, computed, onMounted } from 'vue'
// demo資料
import todosDemo from '/json/todosDemo.json'
todosDemo.forEach(todo => todo.month=new Date().getMonth());


// v-model綁定
const contant = ref('')
const category = ref('personal')

const done = ref(false)
const show = ref(true)
const input = ref(null)

const showWrap = ref(true)
// const todos = reactive(JSON.parse(localStorage.getItem('todos')) || [])
const todos = reactive(JSON.parse(localStorage.getItem('todos')) || todosDemo)
const todoDate = reactive(JSON.parse(localStorage.getItem('todoDate')) || {})
const doneDisplay = ref(JSON.parse(localStorage.getItem('doneDisplay')) || 'show')
const categoryDisplay = ref(JSON.parse(localStorage.getItem('categoryDisplay')) || 'all')
const listToggle = ref()

watchEffect(()=>{
    localStorage.setItem('todoDate',JSON.stringify(todoDate))
    localStorage.setItem('doneDisplay',JSON.stringify(doneDisplay.value))
    localStorage.setItem('categoryDisplay',JSON.stringify(categoryDisplay.value))
})

const setdate=()=>{  //取得今天的日期塞到資料中
    if (Object.keys(todoDate).length === 0 && todoDate.constructor === Object){ //判斷calendar是否為空物件
        const dateNow = new Date()
        todoDate.year = dateNow.getFullYear()
        todoDate.month = dateNow.getMonth() //1月-12月 => 0-11
        todoDate.date = dateNow.getDate() 
        todoDate.day= dateNow.getDay() //星期日-六 => 0-6
    }
}
const emit = defineEmits(['response','listClose','message'])
//篩選出當天的todo
const filterToday = computed(()=>todos.filter((todo)=>todo.year===todoDate.year && todo.month===todoDate.month && todo.date===todoDate.date))
//切換要顯示的done狀態
const filterSwitch = {
    show: (filterToday)=>filterToday.value,
    hide: (filterToday)=>filterToday.value.filter((todo)=>todo.done===false),
}
// 得到篩選done狀態後的物件
const filterResult1 = computed(()=> {
    return filterSwitch[doneDisplay.value](filterToday)
})
//切換要顯示的category狀態
const filterSwitch2 = {
    all: (filterToday)=>filterToday.value,
    personal: (filterToday)=>filterToday.value.filter((todo)=>todo.category==="personal"),
    business: (filterToday)=>filterToday.value.filter((todo)=>todo.category==="business"),
}
// 得到篩選category狀態後的物件
const filterResult2 = computed(()=> {
    return filterSwitch2[categoryDisplay.value](filterResult1)
})
// 接收來自父組件的fortodolist變數
const props = defineProps({
    clickDate: Object,
    listToggle: Boolean,
})
//子組件點擊日曆後要觸發的事 
watch(props.clickDate, (changeDate) => {
    // 改日期
    todoDate.year= changeDate.year
    todoDate.month= changeDate.month
    todoDate.date= changeDate.date
    todoDate.day= changeDate.day
    todoDate.lunar_year= changeDate.lunar_year
    todoDate.lunar_month= changeDate.lunar_month
    todoDate.lunar_date= changeDate.lunar_date
    showWrap.value= false
})
//處理list顯示
watchEffect(()=>{
    listToggle.value= props.listToggle
})
const listClose=()=>{
    if(listToggle.value===true){
        listToggle.value = false
        emit('listClose',listToggle.value)
    }
}
//開始監視todos 一有改變就存檔+通知父組件
watch(todos, (newtodos) => {
    localStorage.setItem('todos', JSON.stringify(newtodos));
    emit('response','') //觸發父組件的response事件
})
//點擊新增按鍵要做的事
const submitHandler = (event) => {
    const todo = {
        id: Date.now(),
        year: todoDate.year,
        month: todoDate.month,
        date: todoDate.date,
        day: todoDate.day,
        contant: contant.value.trim(),
        category: category.value,
        done: done.value,
        show: show.value,
    }
    //防空白
    if(!contant.value.trim()){
        return input.value.focus()
    }
    todos.push(todo) 
    contant.value='' //清空輸入欄位
    categoryDisplayToggle('all')
    emit('message','新增成功') //觸發父組件的message事件
    input.value.focus();
}
const dayTransfer=(day)=>{
    const week=['日','一','二','三','四','五','六']
    return week[day]
}
const checkCategory=(category)=>category==="business" ? true : false
const editCategory=(todo)=>{
    if (todo.category==="personal"){todo.category="business"}
    else if (todo.category==="business"){todo.category="personal"}
}
const deleteTodo=(todo)=>{
    todos.splice(todos.indexOf(todo),1)
    emit('message','刪除成功') //觸發父組件的message事件
}
const editTodo=(todo)=>{
    if(todo.contant)return
    deleteTodo(todo)
}



const doneDisplayToggle=()=>{
    if(doneDisplay.value==='show'){
        doneDisplay.value='hide'
    }else if(doneDisplay.value==='hide') {
        doneDisplay.value='show'
    }
}
const categoryDisplayToggle=(item)=>{
    categoryDisplay.value= item
}

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
        return
    }
    if (Math.abs(startX-endX)<Math.abs(startY-endY)){
        if(startY-endY >0 ){
            listClose()
        }
    }
}
onMounted(()=>{
    setdate()
})
</script>

<template>
    <transition  name="showtodos" @after-leave="showWrap= true">
    <div class="todolistWrap" v-if="showWrap" :class="{show: listToggle}">
        <div class="tool" @click="listClose"><i class="fa-solid fa-rotate-left"></i></div>
        <div class="section" @touchstart="touchstartHandler" @touchend="touchendHandler">
            <h2>{{todoDate.year}}年{{todoDate.month+1}}月{{todoDate.date}}日({{dayTransfer(todoDate.day)}})</h2>
            <h3>{{todoDate.lunar_year}}年{{todoDate.lunar_month}}{{todoDate.lunar_date}}</h3>
            <form @submit.prevent="submitHandler">
                <div class="inputBox">
                    <input autofocus @vnode-mounted="({ el }) => el.focus()" type="text" name="contant" id="contant" v-model="contant" ref="input">
                    <label>請輸入待辦事項</label>
                </div>
                <div class="options">
                    <label for="category1">
                        <input type="radio" name="category" id="category1" value="personal" v-model="category">
                        <span class="bubble"></span>
                        <div><i class="fa-solid fa-tag"></i> 個人</div>
                    </label>
                    <label for="category2">
                        <input type="radio" name="category" id="category2" value="business" v-model="category">
                        <span class="bubble business"></span>
                        <div><i class="fa-solid fa-tag"></i> 工作</div>
                    </label>
                </div>
                <input class="create" id="create" type="submit" value="新增">
            </form>
            <div class="filterBlockWrap" v-if="filterToday.length">
                <div class="filterBlock">
                    <h3>待辦事項</h3>
                    <div class="filter">
                        <div class="doneFilter">
                            <i class="fa-solid fa-filter"></i>
                            <span @click="doneDisplayToggle" :class="{active:doneDisplay==='hide'}">隱藏已完成</span>
                        </div>
                        <div class="categoryFilter">
                            <i class="fa-solid fa-filter"></i>
                            <span @click="categoryDisplayToggle('all')" :class="{active:categoryDisplay==='all'}"><i class="fa-solid fa-tag"></i><i class="fa-solid fa-tag"></i></span>
                            <span @click="categoryDisplayToggle('personal')" :class="{active:categoryDisplay==='personal'}"><i class="fa-solid fa-tag"></i></span>
                            <span @click="categoryDisplayToggle('business')" :class="{active:categoryDisplay==='business'}"><i class="fa-solid fa-tag"></i></span>
                        </div>
                    </div>
                </div>
            </div>
            <div v-else-if="!filterToday.length">
                <h3>目前這天沒有待辦事項，來新增第一筆吧!</h3>
            </div>
        </div>
        <div class="todolist">
            <transition name="showtodo" class="todo" v-for="todo in filterResult2" :key="todo.key"  @after-leave="deleteTodo(todo)">
            <div v-if="todo.show">
                <input type="checkbox" v-model="todo.done">
                <span class="bubble" :class="{ business: checkCategory(todo.category) }" @click="todo.done= !todo.done"></span>
                <input type="text" :class="{ business: checkCategory(todo.category) }" v-model="todo.contant" @blur="editTodo(todo)">
                <button class="edit" @click="editCategory(todo)"><i class="fa-solid fa-arrows-rotate"></i></button>
                <button class="delete" @click="todo.show = !todo.show"><i class="fa-solid fa-trash-can"></i></button>
            </div>
            </transition>
        </div>
    </div>
    </transition> 
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
$bgColor-main: rgb(238, 239, 244, 0.2);
$bgColor-main1: rgb(238, 239, 244);
$button-shadow: rgb(188, 189, 194);
$bubble-personal-color: rgb(108, 154, 139);
$bubble-business-color: rgb(232, 153, 141);
$create-button-color: rgb(243, 87, 170);
$edit-button-color: rgb(96, 77, 83);
$delete-button-color: rgb(162, 44, 41);
$button-text-color: white;
$lable-color: white;
$bgColor: rgb(238, 239, 244);

*{
    margin: 0;
    padding: 0;
    list-style: none;
}
input[type="checkbox"],input[type="radio"]{
    display: none;
}
input{
    outline: none; 
}
i{
    transform: scale(1.1);
}
.todolistWrap{
    flex: 0;
    display: flex;
    flex-direction: column;
    margin: 20px;
    border-radius: 20px;
    background-color: $bgColor-main;
    position: relative;
    transition: 0.3s;
    backdrop-filter: blur(2px);
    box-shadow: none;
    &.show{
        display: flex;
        flex: 3;
        transition: .3s;
        box-shadow: 0 0 3px 3px white;
        @include tablet-768{
            display: flex;
            position: absolute;
            overflow: hidden;
            height: 100vh;
            height: -webkit-fill-available;
            width: 100%;
            border-radius: 0;
            margin: 0px;
            animation: show .3s;
            @keyframes show {
                0%{opacity:0; transform: translateY(30px);}
                100%{opacity:1; transform: translateY(0px);}
            }

        }
        .tool{
            display: none;
            @include tablet-768{
                display: flex;
                background-color: $button-shadow;
            }
        }
        .section{
            display: block;
            @include tablet-768{
                position: sticky;
                top: 0;
                width: calc(100% - 40px);
                background-color: $bgColor-main1;
            }
        }
        .todolist{
            display: block;
            .todo{
                .edit{
                    height: 40px;
                    width: 40px;
                }
                .delete{
                    height: 40px;
                    width: 40px;
                }
            }
        }
    }
    @include tablet-768{
        display: none;
        background-color: $bgColor-main1;
    }
    .tool{
        display: none;
        position: fixed;
        bottom: 20px;
        left: 20px;
        justify-content: center;
        align-items: center;
        padding: 5px;
        width: 40px;
        height: 40px;
        border-radius: 50%;
        font-size: 5px;
        font-weight: 800;
        color: white;
        background-color: $bgColor;
        cursor: pointer;
        user-select: none;
        z-index: 10;
        @include tablet-768{
            display: flex;
        }
        i{
            transform: scale(5);
        }
    }
    .section{
        display: none;
        padding: 50px 20px 0 20px;
        position: sticky;
        top: 0;
        z-index: 1;
        @include tablet-768{
            display: block;
        }
        h2 {
            text-align: center;
            margin: 20px 0 10px 0;
        }
        h3 {
            text-align: center;
            margin-bottom: 30px;
            color: gray;
            font-size: 12px;
        }
        form {
            .inputBox{
                position: relative;
                input[type="text"] { 
                    width: calc(100% - 20px);
                    border: none;
                    border-radius: 5px;
                    height: 30px;
                    padding: 10px;
                    font-size: 20px;
                    margin: 10px 0 10px 0;
                    box-shadow: 3px 3px 3px 0.1px $button-shadow;
                    background-color: $lable-color;
                    transition: .3s linear;
                }
                input[type="text"]:focus { 
                    box-shadow: 0 0 5px 3px $create-button-color;
                }
                label{
                    position: absolute;
                    // top: 22px;
                    // left: 10px;
                    top: -15px;
                    left: 15px;
                    // font-size: 18px;
                    font-size: 10px;                     
                    color: $button-shadow;
                    pointer-events: none;
                    transition: .5s;
                }
                input[type="text"]:focus + label{
                    color: $create-button-color;                    
                }
            }
            .options{
                display: flex;
                label{
                    display: flex;
                    justify-content: center;
                    align-items: center;
                    flex: 1;
                    text-align: center;
                    width: 100px;
                    height: 50px;
                    margin-top: 10px;
                    margin-bottom: 10px;
                    background-color: $lable-color;
                    border-radius: 5px;
                    box-shadow: 3px 3px 3px 0.1px $button-shadow;
                    color: $bubble-business-color;
                    div{
                        margin-left: 10px;
                    }
                    &:first-child{
                        margin-right: 10px;
                        color: $bubble-personal-color;
                    }
                }
            }
            .create{
                width: 100%;
                padding: 10px;
                background-color: $create-button-color;
                color: $button-text-color;
                border: none;
                border-radius: 5px;
                font-size: 20px;
                transition: .5s;
                margin: 10px 0 30px 0;
                box-shadow: 0px 0px 3px $create-button-color;
                cursor: pointer;
                &:hover{
                    opacity: 0.8;
                }
            }
        }
        .filterBlockWrap{
            width: 100%;
            .filterBlock{
                display: flex;
                align-items: center;
                justify-content: flex-start;
                h3{
                    margin: 10px 0 10px 0;
                    color: darkgrey;
                }
                .filter{
                    display: flex;
                    align-items: center;
                    flex-wrap: wrap;
                    .doneFilter{
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        margin-left: 10px;
                        span:nth-child(2){
                            margin-right: 0px;
                            border-radius: 20px;
                        }
                    }
                    .categoryFilter{
                        display: flex;
                        justify-content: center;
                        align-items: center;
                        margin-left: 10px;
                        span:nth-child(2){
                            position: relative;
                            border-radius: 20px 0 0 20px;
                            i:first-child{
                                color: $bubble-business-color;
                                position: absolute;
                                top: 9px;
                                left: 16px;
                            }
                            i:last-child{
                                color: $bubble-personal-color;
                            }
                        }
                        span:nth-child(3){
                            i{
                                color: $bubble-personal-color;
                            }
                        }
                        span:nth-child(4){
                            border-radius: 0 20px 20px 0;
                            i{
                                color: $bubble-business-color;
                            }
                        }
                    }
                    i{
                        color: $button-shadow;
                        margin-right: 5px;
                    }
                    span{
                        cursor: pointer;
                        border: 1px solid $button-shadow;
                        padding: 5px 10px 5px 10px;
                        color: $button-shadow;
                        &.active{
                            background-color: $button-shadow;
                            color: white;
                        }
                    }
                }
            }
        }
    }
    .todolist{
        display: none;
        padding: 0 20px 20px 20px;
        overflow-y: overlay;
        height: 100%;
        @include tablet-768{
            display: block;
        }
        .todo{
            display: flex;
            align-items: center;
            margin: 10px 0 10px 0;
            padding: 8px 12px 8px 12px;
            background-color: $lable-color;
            border-radius: 5px;
            box-shadow: 3px 3px 3px 0.1px $button-shadow;
            &:has(input[type="checkbox"]:checked) input[type="text"]{
                text-decoration: line-through;
                color: $bubble-personal-color;
                &.business{
                    color: $bubble-business-color;
                }
            }
            input[type="text"]{
                flex: 1;
                width: calc(100% - 20px);
                background-color: transparent;
                color: $bubble-personal-color;
                border: none;
                padding: 10px;
                font-size: 20px;
                @include mobile-576{
                    padding: 5px 10px 5px 10px;
                }
                &.business{
                    color: $bubble-business-color;
                }
            }
            .edit{
                display: none;
                width: 30px;
                height: 30px;
                background: linear-gradient(135deg,$bubble-business-color 0%,$bubble-business-color 50%,$bubble-personal-color 50%,$bubble-personal-color 100%);
                color: $button-text-color;
                border: none;
                border-radius: 5px;
                margin-right: 5px;
                cursor: pointer;
                opacity: 1;
                transition: .5s;
                @include tablet-768{
                    display: block;
                    opacity: 1;
                }
            }
            .delete{
                display: none;
                width: 30px;
                height: 30px;
                background-color: $delete-button-color;
                color: $button-text-color;
                border: none;
                border-radius: 5px;
                cursor: pointer;
                opacity: 0;
                transition: .5s;
                @include tablet-768{
                    display: block;
                    opacity: 1;
                }
            }     
            &:hover{
                .edit{
                    display: block;
                    opacity: 0.8;
                }
                .delete{
                    display: block;
                    opacity: 0.8;
                }
            }
        }
    }
}
.bubble{
    display: inline-block;
    width: 14px;
    height: 14px;
    border: 2px $bubble-personal-color solid;
    border-radius: 50%;
    box-shadow: 0px 0px 3px $bubble-personal-color;
    position: relative;
    cursor: pointer;
    &::after{
        content: '';
        width: 0px;
        height: 0px;
        position: absolute;
        background-color: $bubble-personal-color;
        top: 0;
        left: 0;
        right: 0;
        bottom: 0;
        margin: auto;
        border-radius: 50%;
        transition: .3s;
    }
    &.business{
        border: 2px $bubble-business-color solid;
        box-shadow: 0px 0px 3px $bubble-business-color;
        &::after{
            background-color: $bubble-business-color;
        }
    }
}
input:checked ~ .bubble.business::after{
    background-color: $bubble-business-color;
}
input:checked ~ .bubble::after{
    content: '';
    width: 10px;
    height: 10px;
    position: absolute;
    background-color: $bubble-personal-color;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    margin: auto;
    border-radius: 50%;
}


.showtodo-enter-active {
    transition: all 0.5s ease;
}
.showtodo-leave-active {
    transition: all 0.5s ease;
}
.showtodo-enter-from {
    opacity: 1;
}
.showtodo-enter-to {
    opacity: 1;
}
.showtodo-leave-from {
    opacity: 1;
}
.showtodo-leave-to {
    opacity: 0;
}

.showtodos-enter-active {
    transition: all 0.1s ease;
}
.showtodos-leave-active {
    transition: all 0.1s ease;
}
.showtodos-enter-from {
    opacity: 0.3;
}
.showtodos-enter-to {
    opacity: 1;
}
.showtodos-leave-from {
    opacity: 1;
}
.showtodos-leave-to {
    opacity: 0.3;
}
</style>