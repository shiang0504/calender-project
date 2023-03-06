# 行事曆APP


## 專案簡介
[Demo網址](https://shiang0504.github.io/calender-project/)
簡易版本的google日曆，並結合天氣預報，
RWD響應式設計，支援觸控手勢操作(滑動切換行事曆年、月份)

## 開發工具
* Vite
* Vue3 Composition API
* SCSS

## 其他說明
###### 天氣預報
* 使用OpenWeather API查詢五日內城市天氣
* 輸入中、英文城市名稱皆可
* 透過unsplash隨機顯示該城市圖片

###### todolist
* 資料儲存於localstorage
* 可依據標籤及狀態篩選管理

###### 萬年曆
* 與localstorage取得資料
* 統計每天已完成、未完成任務數量
