## 0923 三宏科技 FE Senior 筆試 - 系統規劃

### 規劃大綱
* 請依照 [figma](https://www.figma.com/file/qHE5epfwhzZCZOOho46aZp/%E9%9B%BB%E5%BD%B1%E8%B3%BC%E7%A5%A8%E7%B3%BB%E7%B5%B1-(Public)?type=design&node-id=0-1&mode=design&t=yDfPEgqZKhZcEXK9-0) 設計稿 完成電影購票系統的規劃

* 題目：電影購票系統
    1. 使用者可以在首頁看到一個購票表單及上映中的電影清單
        1. 購票表單送出後會進入購票流程[2]
            1. 購票表單
                1. 欄位包含
                    1. 影片
                    1. 影城
                    1. 日期
                    1. 場次
                1. 表單送出後會進入購票流程
            1. 電影清單中點擊任一部電影可以查看電影資訊細節[4]
    1. 購票流程
        1. 選擇電影票張數及餐飲
        1. 選擇座位
        1. 填寫付款資訊
        1. 付款完成後進入購票完成頁[3]
    1. 購票完成頁面
        1. 顯示購票資訊
        1. 影片名稱
        1. 影廳
        1. 觀看日期
        1. 購票張數
        1. 所選餐飲
        1. 座位資訊
        1. 付款資訊
    1. 電影資訊頁面
        1. 包含電影的名稱、介紹等電影資訊，且有購票按鈕
        1. 購票按鈕點擊後會導頁去獨立的購票表單頁面[5]，並預設帶入影片欄位資訊
    1. 購票表單獨立頁面
        1. 單獨顯示購票表單的頁面


* Route design
    * Name & path
    * 頁面權限 (if need)
* API
    * 只需要文字列出需要哪種用途的 API & 可能要注意的部分
    * 範例：取得國家清單的 API
* Components
    * 這一頁會用到哪些 components，請依照自己的習慣來設計即可
* 資料傳遞方式
    * 不同頁面間資料會如何傳遞
* 其他你認為在系統中特別注意在程式內有特殊處理的部分，必要的時候可以提供 pseudo code

## 以下作答

### 1. Route Design

```javascript
{
    path: '/',
    name: 'index',
    meta: {
        title: '首頁',
    },
},
{
    path: '/movie-book-ticket/:id',
    name: 'movie-book-ticket',
    meta: {
        title: '購票流程'
    },
    beforeEach((to, from) => {
        const {
            movie,
            theater,
            date,
            time,
            token,
        } = to.query
        // 如有 token 直導結帳完成頁
        props: route => ({ token: route.query.token })
        
        // 確認在進入頁面前有符合需要的訂票資訊
        if (to.parms.id && movie && theater && date && time) {
            return true
        } else {
            return false
        }
    })
}
{
    path: '/movie-info/:id',
    name: 'movie-info',
    meta: {
        title: '電影資訊頁面'
    },
    beforeEach((to, from) => {
        if (from.parms.id) {
            return true
        } else {
            return false
        }
    })
}
{
    path: '/go-ticket',
    name: 'go-ticket',
    meta: {
        title: '購票表單'
    },
},
{
    path: '/error',
    name: 'error',
    meta: {
        title: 'Error 錯誤'
    },
}
```

### 2. API

#### API 規劃
1. Get - Carousell Api - 輪播資料用來區分需要使用的資料。
    * **Query: type="top、hot、comming"**
3. Get - Movie Data Api - 電影資料搜尋找尋需要的電影資訊。
    * **Query: search**
4. Get - Movie Theater Api - 當前搜尋符合當前有上映電影的影院。
    * **Query: movie**
5. Get - Movie Show Times Api - 當前搜尋符合當前有上映的電影的影院場次。
    * **Query: movie、theater、date**
6. Get - Movie Ticket Type List Api - 電影票種類。
7. Get - Movie Seat Api - 顯示可訂位的資料。
    * **Query: movie、theater、date、time**
        * 撈取帶有座位數字陣列，其中跳過的數字代表已被訂位。
8. Post - Third-party Cash Flow Api 第三方金流、結帳。
    * **Data: movie、theater、date、time、ticket、seat**
    * **Third-party: name、card、passDate、cvv、mail**
9. Get - Ticket Result Api - 結帳後撈取結果資料。
    * **Query:token**
        * 撈取影片名稱、影廳、觀看日期、購票張數、所選餐飲、座位資訊、付款資訊
10. Get - Movie Info Api - 電影資訊。
    * **Parms:id**
        * 包含電影的名稱、介紹等電影資訊
11. Get - Footer List Data Api - 頁腳列表資料。

#### **調用時安排**
* 0-0 首頁 
    * Carousell Api
    * Movie Data Api
    * Movie Theater Api
    * Movie Show Times Api
* 1-1 購票流程-電影與餐點
    * Movie Ticket Type List Api
* 1-2 購票流程-座位
    * Movie Seat Api
* 1-3 購票流程-付款
    * Third-party Cash Flow Api
* 1-4 付款流程-完成購票
    * Ticket Result Api
* 2-1 電影資訊頁面
    * Movie Info Api
* 3-1 購票表單
    * Carousell Api
    * Movie Data Api
    * Movie Theater Api
    * Movie Show Times Api

* All Need Footer List Data Api

### 3. Components
#### **規劃 Page、Components**
* Page 頁面
    * IndexPage.vue - 首頁頁面
    * _TicketPage.vue - 購票流程頁面
        * TicketPageView
            * MovieAndTicket.vue 選擇電影票張數及餐飲頁面
            * MovieTicketSeat.vue 選擇座位頁面
            * Check.vue 填寫付款資訊頁面
            * Result.vue 付款完成後進入購票完成頁
    * _MoviePage.vue - 電影資訊頁面
    * TicketFormPage.vue - 購票表單頁面
* Components 組件
    * Carousell.vue - 輪播組件
    * TicketForm.vue - 訂票表單組件

#### **調用時安排**
* IndexPage.vue(0-0 首頁)
    * Carousell.vue
    * TicketForm.vue
* _TicketPage.vue(1-1~4 購票流程)
    * MovieAndTicket.vue 
    * MovieTicketSeat.vue
    * Check.vue
    * Result.vue
* _MoviePage.vue (2-1 電影資訊頁面)

* TicketFormPage.vue(3-1 單獨顯示購票表單的頁面)
    * TicketForm.vue

### 4. 資料傳遞方式
* 以目前設計稿規模來說不太需要做到 vuex/pinia 層資料互通，各自 query 帶入，Router Middleware 驗證攔截即可。
* 但如果真的要求的話
    * 輪播資料
    * 電影資料
    * 影院資料
    * 電影票種類
    * 頁腳列表資料
* 皆可在 Web Init 時，撈取緩存起來；之後再提供給各頁面。

### 5. 其他你認為在系統中特別注意在程式內有特殊處理的部分，必要的時候可以提供 pseudo code
* 從設計稿上看似乎沒有會員系統，所以只能假設結帳之後的結果會直接寄信到信箱，再由信箱上連結點擊帶入 token 導入 **完成購票頁面呈現**

* 金流付款的部分有沒有**自接金流**處理還是**全交由第三方**，差異在於自接會有發票開立、退貨、退款機制保留需全由團隊開發。
