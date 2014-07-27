# Form Objects

在 rails 設計中， form 最基本的用法就是把 model 的 instance 丟到 `form_for`  `simple_form_for` `bootstrap_form_for` 這類的 form helper 中。

並在 model 定義資料驗證和錯誤訊息 `validation`，例如：

1. 產品名稱不得為空白
2. 金額不得少於 0
3. ... etc

偶爾我們可能還會定義一些 `callback` 在資料儲存前後安插一些事情，例如：

1. generate uuid 當作 id
2. User model 儲存後，以 email 寄出歡迎信
3. ... etc

# 問題是 ...

以上的做法在情況單純的時候都不是問題，但假設你的狀況是：

1. 新增和編輯時的檢驗邏輯不同?

    **如先寫入一筆 empty ercord 等待使用者編輯的情況?**

2. 不同角色編輯同一張 model 的檢驗流程、和做的事情不一樣?

    * 訪客註冊 create User (檢驗完整資料) 後寄出 **確認信**
    * 管理者 create User (只需填寫 Email) 後寄出 **邀請信**

3. Tableless Form

    ** 使用者邀請功能**

    * user 填寫朋友 email
    * 需驗證此 email 是否符合格式、若非會員則 show form error
    * 最後只寄出邀請信不寫入任何資料

4. 多個表的操作 + transaction 或整合其他服務的操作

    **如多 vendor EC 系統 + 紅利點數結帳**

    * 扣除產品庫存
    * 檢驗 User 紅利、折讓金額、最後訂單需付款金額
    * 扣除 User 帳戶紅利
    * 寫入訂單
    * 通知購買人
    * 通知廠商出貨

以上的狀況如果不用 Form object，不是 model 就是 controller 會便很髒，而 `Form Object` 的原理簡單來說就是分擔 model 的工作，把驗證邏輯、和儲存邏輯拆出來，讓 model 單純的負責資料。


### 例子

以上的範例你可以拆成：

1. UserForm::Create <= 使用者的角度 Create user
2. AdminForm::CreateUser <= 管理者的角度 Create User
3. UserForm::EditProfile <= 使用者更新 Profile
4. UserForm::Create <= 使用者新增 User
5. UserForm::InviteFriend <= 使用者邀請朋友
6. OrderForm::Create <= 訂單 Form


