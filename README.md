# 【個人專案】JDR 工具簡介
內容：JDR 工具簡介 (Job Dependency Runner)

---
[目錄](https://hackmd.io/Y7i9O4hCQu6xOJ94__PhNg)

[教學筆記 01：編碼、檔案、斷行字元、Excel、其他基礎知識介紹](https://hackmd.io/7-FajauqT62vXpxOViWzBw)
[教學筆記 02：Linux 指令介紹 ](https://hackmd.io/9D_WXaT3TsCaBrsOQKYdTw)
[教學筆記 03：awk 指令介紹](https://hackmd.io/PHZRjtQMRi2v9Z-u6bD6Tg)
[教學筆記 04：文件圖表表達方式的經驗分享](https://hackmd.io/xVM9lnFBSneA6uhIzdkYyA)
[教學筆記 05：以 C 語言開發資料庫存取程式 (ECPG)](https://hackmd.io/4jh_5A51TfieovVipQqoXA)
[推薦書籍](https://hackmd.io/t7T5FxfmT3Kih-b7PRUZrA)
[補充教材](https://hackmd.io/w-BNpl_TSuysG4_qaGtmFg)
個人專案：JDR 工具簡介 (Job Dependency Runner)

---

**※ 前言：**

筆者最近開始學習 Python，因此試著撰寫這個專案來做為練習。概念的發想，是源自於先前幫財政部開發系統時遭遇到困難，所產生的解決方法。在此分享一下。

![](https://i.imgur.com/sg1lCzG.png)
**<center>圖1、JDR 工具外觀</center>**

![](https://i.imgur.com/V1Qri4h.gif)
**<center>圖2、利用 JDR 工具執行與管理程式</center>**

---

## 一、開發動機：
JDR (Job Dependency Runner) 是本專案所開發的一套資料治理小工具，簡言之就是一套「用來協助執行與管理程式的程式」。

在工作上，「執行程式」這個動作，大部分情況下並沒有特別困難，通常就是先把指令編輯好，然後丟進 shell、或者某個介面/平台，然後再坐等結果出來，有時可能也會用 crontab 之類的工具來預做排程。

這種方法，如果規模只是一兩支到十幾支程式的話也許還沒什麼問題，但如果有上百支、上千支程式的話，管理起來就會有困難，其原因在於「數量」與「相依性」所衍生的管理議題。

這些管理議題包含：「程式的當前狀態為何？」、「程式執行的先後順序為何？」、「如果某一支程式需要重跑，會不會影響到哪些下游的相關程式？」當程式的數量越多時，越不可能光憑工程師的記憶來管理。即使以文件輔助記載，維護與查找上也需要時間成本。

且由於近幾年數據分析越來越重要，「程式有沒有按時而正確地執行」這種資料治理 (Data Governance) 議題也越來越被重視。為解決這些議題，我希望在本專案實作出一套工具，讓部分管理議題能夠自動化、儀錶板化，並以視覺化的方式來呈現結果。

也許這個專案在功能上，會跟一些 ETL 工具 (例如：SSIS、Trinity、DataStage、Automation) 有重疊，因為 ETL 工具也有執行與管理程式的功能，但由於目前我還沒有找到一個可以滿足需求的工具，所以這也是我決定想要自行開發的另一個原因。

我希望對於使用者而言，只須維護一份作業清單 (Excel 格式)，再將清單輸入到這套工具之後，就能自動產生圖形化的程式相依性流程圖。所謂的程式相依性流程圖是屬於一種「有向無環圖」 (DAG，Directed Acyclic Graph)。有了圖之後，就衍生出許多如何對其進行操作的議題。我盡可能簡化這些操作，讓這些操作與管理的行為只須在圖形化的介面上做個設定、按個按鈕、看個報表就能輕鬆進行。

歡迎大家使用這套工具，但工具的設計奠基於我個人之前的開發經驗以及自己的想像，因此如果有人覺得不好使用、不方便、不夠彈性的地方，歡迎將這些問題回饋給我，讓我可以做為改進的參考。

本專案的開發，最主要使用下列 python 套件：
* PyQt：視窗程式設計 (GNU GPL 授權)
* graphviz：繪製流程圖 (EPL 授權)
* networkx：graph 的操作 (BSD 授權)

針對本專案軟體授權的選擇，雖然我個人比較意屬最自由的 BSD 授權，但因為 PyQt 套件本身是比較嚴格的 GNU GPL 授權，所以我也只好跟著 GPL。

工具的連結如下：


---

## 二、JDR 工具的運作架構：
 
![](https://i.imgur.com/BHxZVsY.png =70%x70%)

**<center>JDR 工具的運作架構與相關角色</center>**

JDR 工具運作架構的相關角色說明如下：
  * **管理者 (Manager)**：維護作業清單文件的管理者，每當有新增、修改作業，或變動執行規則時，應立即更新作業清單文件。建議由組織中熟知程式執行規則的人擔任，例如專案管理師、資料模型師。
  * **作業清單文件 (Job List Document)**：記錄組織中所有作業的一份清單文件，以 Excel 格式撰寫，由管理者維護。格式請參考「[作業清單文件](#三、作業清單文件)」一節。另外，JDR 工具也提供文件的剔錯檢查，請參考「[作業清單文件的剔錯檢查](#(5).作業清單文件的剔錯檢查)」一節。
  * **開發者 (Developer)**：組織內開發作業的工程師，每一支作業須依據一定規格進行開發，執行時向控制表註冊與更新執行狀態。
  * **作業 (Job)**：由開發者所撰寫的程式，開發時須依據一定規格，執行時向控制表註冊與更新執行狀態。詳細內容請參考「[關於程式、作業、執行項目的定義](#四、關於程式、作業、執行項目的定義)」一節。
  * **控制表資料庫 (Control Table Database)**：用來儲存作業執行時的即時狀態與資訊，並藉由 JDR 工具統整後呈現於畫面供使用者參考。控制表只有一張，詳細內容請參考「[控制表格式與作業開發原則](#五、控制表格式與作業開發原則)」一節。針對控制表的一些彈性設定，則可以參考「[控制表的彈性設定](#(4).控制表的彈性設定)」一節。
  * **JDR 工具**：本專案使用的資料治理工具。依據作業清單中的設定，與讀取控制表中的設定，將各作業之間的相依性與執行順序，以視覺化的方式呈現於畫面，並以動態、互動的方式讓使用者對所有作業進行操作與管理，此外還有提供報表功能，讓使用者檢視執行結果。詳細內容請參考「[JDR工具使用方式](#六、JDR工具使用方式)」一節。

---

## 三、作業清單文件
### (1).欄位說明
使用 JDR 工具之前，必須先維護一份作業清單文件，這份文件採用 Excel 的檔案格式，且必須包含下表1中的所有欄位：

**<center>作業清單文件的欄位說明</center>**
| # | 中文欄位名稱 | 英文欄位名稱 | 說明 |
| --| -------- | -------- | -------- |
| 1 | 編號      | job_no   | 賦予給該作業的任意編號，讓使用者方便查找，使用數字或字串皆可。     |
| 2 | 類別      | job_type | 賦予給該作業的類別，使用者可依據自己的規則任意分類。     |
| 3 | JOB名稱   | job_name | 該作業的名稱。     |
| 4 | 執行頻率   | job_freq | 對於該作業執行時間的規律性描述，若有多種執行頻率請以斷行字元分隔。本欄位與「計畫執行時間」有關，請參考「[關於執行頻率](#(2).關於執行頻率)」一節     |
| 5 | JOB來源   | job_src  | 該作業的上游作業，若有多個上游作業請以斷行字元分隔。本欄位與作業相依性有關，請參考「[關於作業相依性](#(3).關於作業相依性)」一節。     |
| 6 | 非JOB註記 | job_not  | 若該作業實際上並非須執行的作業，請將本欄填寫為「Y」。     |
| 7 | 執行指令  | job_cmd  | 該作業的實際執行指令，提供「${PLANDT}」保留字做為計畫執行時間。     |

以上欄位的使用原則為：
  * 本工具支援中文欄位與英文欄位名稱，中英文皆可使用。
  * 欄位的出現順序可不依照下表排列，但每個欄位都一定要有。

### (2).關於執行頻率
「執行頻率」實際上有兩層含意：
1. **描述規律行為**：例如某程式於設計時規劃將會「每日執行」、「每月1日執行」、「每年1月1日執行」，就代表其具有規律性的執行週期，這種規律的描述語法，在作業清單文件中可用下表1的格式來描述。
2. **決定特定時間**：當「執行頻率」發生在一段時間之內，則可以決定出一些特定時間點。例如某程式「每月1日執行」，則在西元 2023 年一整年內，該程式便會於 2023-01-01、2023-02-01、2023-03-01...2023-12-01 這幾個特定時間點執行，這些特定時間在本專案稱之為「**計劃執行時間**」。

**<center>作業清單文件的「執行頻率」欄位</center>**
| # | 執行頻率表達式範例 | 代表意義 |
| -------- | -------- | -------- |
| 1 | YYYYMMDD            | 每天 (00:00:00)     |
| 2 | YYYYMMDD 080000     | 每天 (08:00:00)     |
| 3 | YYYYMM01            | 每月 1 日 (00:00:00)     |
| 4 | YYYYMM05 120000     | 每月 5 日 (12:00:00)     |
| 5 | YYYYMM$$            | 每月最後一日 (00:00:00)     |
| 6 | YYYYMM$$ 233000     | 每月最後一日 (23:30:00)     |
| 7 | YYYY1025            | 每年 10 月 25 日 (00:00:00)     |
| 8 | YYYY1201 173000     | 每年 12 月 1 日 (17:30:00)     |
| 9 | YYYY02$$            | 每年 2 月最後一日 (00:00:00)     |
| 10 | YYYY02$$ 180000    | 每年 2 月最後一日 (18:00:00)     |
| 11 | 20200101           | 特定時間：2020-01-01 (00:00:00)     |
| 12 | 20221231 235900    | 特定時間：2022-12-31 (23:59:00)     |
| 13 | YYYY1301           | 格式錯誤 (無 13 月)     |
| 14 | YYYYMM32           | 格式錯誤 (無 32 日)     |
| 15 | YYYYMMDD 250000    | 格式錯誤 (無 25 時)     |
| 16 | YYYYMMDD 0800      | 格式錯誤 (時間格式錯誤)     |
| 17 | YYYYMMDD 08        | 格式錯誤 (時間格式錯誤)     |
| 18 | YYYY$$01           | 格式錯誤 ($$ 只能用在表示日期)     |
| 19 | YYYYMMDD-120000    | 格式錯誤 (日期與時間之間不可使用空白以外的分隔字元)     |
| 20 | DDMMYYYY           | 格式錯誤 (描述方式不符定義，僅接受 "年月日時分秒" 的順序)     |

### (3).關於作業相依性
「作業相依性」包含兩種：
1. **作業內相依性**：當單一作業在一段時間之內，須在多個計畫執行時間執行時，則必須依照計畫執行時間的先後次序。
2. **作業間相依性**：不同作業之間的相依性，則依照作業清單文件中的「JOB來源」欄位來設定 (以斷行字元作為分隔)。若某項目其上游作業有多個計畫執行時間，則選擇不晚於且最接近該項目計畫執行時間的上游項目。

### (4).舉例說明
**1. 範例一 (無相依性)**：
下列以 4 支作業為例，彼此之間無任何相依性，可將作業清單文件設定如下：

| 編號 | 類別 | JOB名稱 | 執行頻率 | JOB來源 | 非JOB註記 | 執行指令 |
| ---- | ---- | ---- | -------- | ---- | ---- | ---- |
| 1 | test | Job1 | YYYYMMDD |      |    | ```python runjob.py Job1 ${PLANDT}``` |
| 2 | test | Job2 | YYYYMMDD |      |    | ```python runjob.py Job2 ${PLANDT}``` |
| 3 | test | Job3 | YYYYMMDD |      |    | ```python runjob.py Job3 ${PLANDT}``` |
| 4 | test | Job4 | YYYYMMDD |      |    | ```python runjob.py Job4 ${PLANDT}``` |

將開始與結束時間皆設定為同一日 (本例為 2023-01-01)，則可以得到下圖，此時這 4 支作業各自獨立，執行時沒有先後次序的問題。

![](https://i.imgur.com/Vow73nG.png)

**2. 範例二 (作業內相依性)**：
同樣使用範例一的作業清單文件設定，但將開始與結束時間設定在 2023-01-01 至 2023-01-02 ，則可以得到下圖，此時只有作業內相依性，但沒有作業間相依性。意思是這 4 支作業各自獨立，但都必須等 2023-01-01 的作業執行完成之後才能執行 2023-01-02 的作業。

![](https://i.imgur.com/HVhFhFW.png)



**3. 範例三 (作業間相依性)**：

下列以 4 支作業為例，彼此之間有相依性，可將作業清單文件設定如下：

| 編號 | 類別 | JOB名稱 | 執行頻率 | JOB來源 | 非JOB註記 | 執行指令 |
| ---- | ---- | ---- | -------- | ---- | ---- | ---- |
| 1 | test | Job1 | YYYYMMDD |      |    | ```python runjob.py Job1 ${PLANDT}``` |
| 2 | test | Job2 | YYYYMMDD | Job1 |    | ```python runjob.py Job2 ${PLANDT}``` |
| 3 | test | Job3 | YYYYMMDD | Job1 |    | ```python runjob.py Job3 ${PLANDT}``` |
| 4 | test | Job4 | YYYYMMDD | Job2<br>Job3 |    | ```python runjob.py Job4 ${PLANDT}``` |

將開始與結束時間皆設定為同一日 (本例為 2023-01-01)，則可以得到下圖，此時這 4 支作業執行的順序必須依照相依性的設定：Job1 執行完畢後才能執行 Job2 與 Job3，待 Job2 與 Job3 兩者皆執行完畢後才能執行 Job4。

![](https://i.imgur.com/EXL6PPM.png =60%x60%)


**4. 範例四 (作業間相依性 + 作業內相依性)**：
同樣使用範例三的作業清單文件設定，但將開始與結束時間限定在 2023-01-01 至 2023-01-02 ，則可以得到下圖，此時除了有作業間相依性，也有作業內相依性。

![](https://i.imgur.com/pwxjPuv.png =80%x80%)


---

## 四、關於程式、作業、執行項目的定義
### (1).程式 (Program)
一系列用來完成特定功能的指令，時常使用一些高階程式語言 (如：C/C++、Python) 來撰寫，所以像是學生練習時寫的 Hello World、企業或組織日常執行的程式、本專題中所使用的 JDR 工具，都屬於程式的一種。
### (2).作業 (Job)
「作業」也是程式的一種，但比較偏向於企業或組織在工作上的說法。作業通常用來完成系統中稍微複雜一點的特定任務，例如電信公司可能會有「批價作業」、「出帳作業」、「銷帳作業」這種用來處理帳務金額的程式；半導體製造業可能也會有「排程作業」這種程式來安排各產品在製造產線中的機台順序。
### (3).執行項目 (Execution Item)
「執行項目」則是「作業」依據其「執行頻率」在「特定時間」(即計畫執行時間) 的執行行為。例如電信公司的出帳作業可能是每個月月初執行一次 (執行頻率)，因為客戶是每個月收到一次帳單的。而其中在 2023 年 1 月月初 (計畫執行時間) 執行時，產生的就是 2023 年 1 月份的帳單、2 月初執行則產生 2 月份的帳單，以此類推，N 月初執行則產生 N 月份的帳單。因此「執行項目」可以視為「作業」+「計畫執行時間」的一種概念。
### (4).本工具的目的
本工具的目的是讓各作業依據其執行頻率，以及各作業之間的相依性，以視覺化的方式在畫面上正確安排出執行順序，每一次的執行事件即為畫面中的執行項目。使用者可以對執行項目直接進行管理與操作，例如：以顏色區分執行狀態 (下圖1)、執行該項目、執行該項目與下游項目、產生管理報表、若項目未按照順序執行則產生警示訊息等功能。

![](https://i.imgur.com/SCopRFm.png)
**<center>以顏色區分執行狀態</center>**

* 狀態管理：目前本工具使用以下顏色來表示狀態：
  * initial (未執行)：白色 (#FFFFFF)
  * waiting (等待中)：淡黃色 (#FFFFCC)
  * running (執行中)：黃色 (#FFFF00)
  * success (已成功)：綠色 (#74C126)
  * failure (已失敗)：紅色 (#EFABCD)
  * 此外 not job (非作業) 與 undefined job (未定義作業) 由於不可操作，因此預設為灰色 (#C0C0C0)。

* 狀態流程：狀態具有順序性，除了非作業與未定義作業本身狀態不會改變，其他有效項目的狀態皆由「未執行」開始，然後依序變為「等待中」、「執行中」，最後視執行結果變為「已成功」或「已失敗」。
![](https://i.imgur.com/P72WLVW.png)
**<center>執行狀態的流程</center>**


---

## 五、控制表格式與作業開發原則
「作業」是執行商業邏輯的主要核心，而 JDR 只是輔助用的工具。若要能使 JDR 工具即時讀取作業執行狀態並反應於畫面上，則必須對作業內的執行邏輯進行規範，而記錄作業狀態的控制表也必須互相配合。本工具盡量簡化控制表的設計與其數量，因此只有一張控制表，讓使用者方便利用。

**※ PS：目前 JDR 工具僅支援 PostgreSQL 作為控制表資料庫。**

### (1).控制表DDL
控制表的欄位格式如下：
```
create table job_exec_log (
  job_name  varchar(200),
  plan_dt   timestamp,
  status    varchar(20),
  act_sdt   timestamp,
  act_edt   timestamp,
  data_num  integer
);
```

### (2).控制表欄位介紹
控制表各欄位說明：
| # | 控制表欄位 | 說明 |
| --| -------- | ---- |
| 1 | job_name | 作業名稱        |
| 2 | plan_dt  | 計畫執行時間     |
| 3 | status   | 作業執行狀態，包含：執行中 (running)、已成功 (success)、已失敗 (failure) |
| 4 | act_sdt  | 實際開始執行時間  |
| 5 | act_edt  | 實際結束執行時間  |
| 6 | data_num | 處理資料筆數，但本欄位之值須由作業本身計算之後再填入 |

### (3).作業開發規格
作業的邏輯中須包含存取控制表的部分，以下提供簡略的虛擬碼。每支作業都須依照此原則開發：
```
# 1.執行開始
# 2.將本次執行的狀態註冊至控制表：
  insert into job_exec_log：
  填入欄位包含： job_name, plan_dt, status (請填 'running'), act_sdt
# 3.執行主要作業邏輯
# 4.結束前將本次執行的狀態更新至控制表：
  update job_exec_log：
  異動欄位包含：status (依據執行結果填入 'success' 或 'failure'), act_edt, data_num
  並加入 where 限制條件：job_name, plan_dt, act_sdt
# 5.執行結束
```

---

## 六、JDR工具使用方式
以下開始介紹本工具的操作方法與各種功能：

### (1).介面介紹
JDR 工具的介面外觀如下圖，主要由「設定區」、「項目導航按鈕區」、「主要操作區」、「Log 區」四大部分所組成。
![](https://i.imgur.com/frbr3Ob.png)

1. 設定區：內部還有「Basic」、「DB」、「Function」、「Information」四個子頁籤，提供一般設定、操作功能與資訊彙整。
  * Basic (基本設定)：
    * 設定語系：改變本工具的顯示語言，目前支援英文、中文。
    * 設定作業執行的開始與結束區間，每個作業會依據本身的執行頻率，來決定符合開始與結束區間內的「計畫執行時間」。
    * 讀取作業清單文件，並指定 sheet 名稱。
    * 當上述設定完成後，可在主要操作區中產生流程圖。產製過程中會協助剔錯，將錯誤的設定顯示於 Log 區。
    * 可以自行設定項目在各種狀態的顏色，並可恢復為預設值。
  ![](https://i.imgur.com/6L75dAX.png =50%x50%)

  * DB (資料庫連線)：
    * 控制表資料庫連線設定：包含連線 IP、帳號、密碼、連接埠、資料庫名稱。
    * 控制表設定：由於使用者用來作為控制表的名稱與欄位不一定會與本專案相同，因此提供自行設定的彈性。
    * **注意：目前 JDR 工具僅支援 PostgreSQL 作為控制表資料庫。**
  ![](https://i.imgur.com/RAz4mdc.png =50%x50%)

  * Function (功能)：
    * Run On Time (按時執行)：
    若本核取方塊打勾，則表示所有項目都必須等到實際時間超過項目的計畫執行時間之後才可執行；若本核取方塊未打勾，則表示所有項目都不須考慮實際時間是否超過項目的計畫執行時間，可直接執行。
    * Run All Items (執行所有項目)：
    不論主要操作區中是否有已經執行成功之項目，一律從頭開始執行所有項目。
    * Continue All Items (繼續所有項目)：
    考慮主要操作區中已經執行成功且有依照順序執行之項目，接續於其後面繼續執行所有剩餘的項目。
    * Stop All Items (停止所有項目)：
    當已經執行「Run All Items」或「Continue All Items」功能之後若想要中斷執行，則可執行本功能，執行後會刪除等待中的項目，但已經執行中的項目則無法中斷，只能等待其執行完畢。
    * Show Report (顯示報表)：
    顯示主要操作區中所有項目的彙總報表 (main report)，包含各項目種類與狀態的圓餅圖、比較計畫執行時間與實際執行時間的差距統計、執行時間與處理筆數的統計。
    * Reload From DB (由 DB 載入狀態)：
    因為使用者可能會利用其他方式執行作業，因此提供本功能，讓使用者能夠依據控制表的執行資訊，更新主要操作區上各項目的狀態。
    * Save SVG (儲存 SVG)：
    將主要操作區上的圖形儲存為 SVG 圖檔。
  ![](https://i.imgur.com/yeatnzo.png =50%x50%)

  * Information (資訊)：
    ![](https://i.imgur.com/4pH2lUR.png =50%x50%)
    * 作業資訊之定義：

    | 編號  | 中文名詞 | 英文名詞 | 說明 |
    | ---- | -------- | -------- | -------- |
    | (A1) | 文件作業數量  | Document jobs    | 作業清單文件中的紀錄筆數|
    | (A2) | 有效作業數量  | Available jobs   | 符合時間區間，且非JOB註記不為Y，且作業清單文件中有定義 |
    | (A3) | 無效作業數量  | Unavailable jobs | 不符合時間區間|
    | (A4) | 非作業數量    | Not jobs         | 非JOB註記為Y|
    | (A5) | 未定義作業數量 | Undefined jobs   | 作業清單文件中未定義|

    * 項目資訊之定義：
    
    | 編號  | 中文名詞 | 英文名詞 | 說明 |
    | ---- | -------- | -------- | -------- |
    | (B1) | 所有項目數量  | All items       | 於主要操作區的所有項目 |
    | (B2) | 有效項目數量  | Available items | 符合時間區間，且非JOB註記不為Y，且作業清單文件中有定義 |
    | (B3) | 未執行項目數量 | Initial items   | 狀態為 initial |
    | (B4) | 等待中項目數量 | Waiting items   | 狀態為 waiting |
    | (B5) | 執行中項目數量 | Running items   | 狀態為 running |
    | (B6) | 已成功項目數量 | Success items   | 狀態為 success |
    | (B7) | 已失敗項目數量 | Failure itemss  | 狀態為 failure |

    * 作業資訊與項目資訊之間的關係：
      * (A1) = (A2) + (A3) + (A4)
      * (B2) = (B3) + (B4) + (B5) + (B6) + (B7)
      * (B1) = (B2) + (A4) + (A5)
    ![](https://i.imgur.com/gj6Zwqi.png)
  


2. 項目導航按鈕區：
    當主要操作區產製出流程圖時，會在項目導航按鈕區同時產生出代表該項目的按鈕，按鈕的目的有二：
    * 協助在主要操作區搜尋項目：
      當主要操作區的項目數量越多、流程越複雜時，想要尋找一個特定項目會變得更加麻煩，因此提供本功能讓使用者便於尋找畫面上的項目。當點選一項目的按鈕時，該項目就會顯示在主要操作區的「最左上角」。
    * 協助查看項目的狀態：
      按鈕的顏色與主要操作區的項目顏色相同，因此可以用來快速確認所有項目的狀態。
  ![](https://i.imgur.com/noIZEbT.png =50%x50%)


3. 主要操作區：

  * 提供功能：本區用來顯示流程圖，使用者可在此對項目進行操作 (可操作有效項目，但不可操作非作業與未定義作業)，目前提供三種操作供使用者選擇：
    * Run Single Item (執行單一項目)：
    * Run Dependency Item (執行相依項目)：
    * Show Report (顯示報表)：

  * 以顏色反應狀態：有效項目在執行後，會依據執行狀態改變顏色，預設的顏色為：
    * initial (未執行)：白色 (#FFFFFF)
    * waiting (等待中)：淡黃色 (#FFFFCC)
    * running (執行中)：黃色 (#FFFF00)
    * success (已成功)：綠色 (#74C126)
    * failure (已失敗)：紅色 (#EFABCD)
    * 此外 not job (非作業) 與 undefined job (未定義作業) 由於不可操作，因此預設為灰色 (#C0C0C0)。

  * 執行時間順序檢查功能：由於使用者可任意執行有效項目，因此可能會有執行時間順序錯誤的現象，所以在此提供檢查功能，若上游項目的結束時間晚於下游項目的開始時間，則兩者間的箭頭顏色會變為紅色，於項目導航按鈕區該項目的字型顏色也會變為紅色。

    ![](https://i.imgur.com/sFVi89z.png)



4. Log 區：
    本區用來輸出執行時期的資訊，包括：
    * 環境變數設定值
    * 使用者操作動作
    * 有效項目的執行訊息
    * 各種剔錯訊息，例如：
      * 控制表資料庫的連線是否正常
      * 作業清單文件的設定是否有誤
      * 產製流程圖時的其他剔錯資訊
    
![](https://i.imgur.com/U3zPxth.png =70%x70%)



### (2).使用案例示範
1. 語言切換：
JDR 工具提供多國語言支援，目前支援繁體中文與英文，預設為英文。調整設定區中的 Language 下拉式選單即可。
![](https://i.imgur.com/vKtsC7R.png =50%x50%)

2. 針對單一(或部分)項目的功能操作：
假設一份作業清單文件設定如下：

| 編號 | 類別 | JOB名稱 | 執行頻率 | JOB來源 | 非JOB註記 | 執行指令 |
| ---- | ---- | ---- | -------- | ---- | ---- | ---- |
| 1 | test | Job1 | YYYYMMDD |      |    | ```python runjob.py Job1 ${PLANDT}``` |
| 2 | test | Job2 | YYYYMMDD | Job1 |    | ```python runjob.py Job2 ${PLANDT}``` |
| 3 | test | Job3 | YYYYMMDD | Job1 |    | ```python runjob.py Job3 ${PLANDT}``` |
| 4 | test | Job4 | YYYYMMDD | Job2<br>Job3 |    | ```python runjob.py Job4 ${PLANDT}``` |

  * [1]. 產生作業相依性流程圖
    開啟 JDR 工具，設定起始與結束日期 (皆以 2023-03-01 為例)，然後讀取作業清單文件，選擇要讀取的 sheet，並按下 generate (產製) 按鈕，就會於主要操作區上產生作業相依性流程圖。(以上動作請先確定控制表資料庫的連線正常)
    ![](https://i.imgur.com/x2ynjxU.gif)

  * [2]. 執行單一項目
    產生作業相依性流程圖之後，可以任意選擇狀態為未執行 (initial)、已成功 (success)、已失敗 (failure) 的單一項目來執行。只要將游標移動至欲執行的項目，按下滑鼠右鍵並點選「Run Single Item」功能，就能執行該項目，執行結果也會反應在主要操作區與項目導航按鈕區。
    ![](https://i.imgur.com/35KM8Rd.gif)

  * [3]. 執行相依項目
    產生作業相依性流程圖之後，若欲執行一連串具有上下游關係的項目，可以任意選擇狀態為未執行 (initial)、已成功 (success)、已失敗 (failure) 的最上層項目來執行。只要將游標移動至欲執行的最上層項目，按下滑鼠右鍵並點選「Run Dependency Item」功能，就能按順序執行該項目與其下游項目，執行結果也會反應在主要操作區與項目導航按鈕區。
    ![](https://i.imgur.com/gGbZLdC.gif)
    但必須要注意，執行順序也會受到執行結果與項目所有上游狀態的影響，
    下面的例子表示若上圖中的 Job2 的執行結果為失敗，則下游的 Job4 就不會執行：
    ![](https://i.imgur.com/IpYluf4.gif)
    又例如若對 Job2 點選「Run Dependency Item」功能，但下游的 Job4 並不會執行，這是因為 Job4 的上游 Job3 尚未執行：
    ![](https://i.imgur.com/5AcKo9k.gif)


  * [4]. 若執行時間順序錯誤時，以顏色警告使用者
    由於項目的執行順序是由上層依序執行到下層，因此正常而言，**上層的執行結束時間會早於下層的執行開始時間**。但如果因為使用者任意執行項目，導致違反這個規則的話，就會在主要操作區中將箭頭顏色改變為紅色，項目導航按鈕區也會將下層的按鈕文字改變為紅色。
    ![](https://i.imgur.com/dEgA6Cg.gif)

  * [5]. 顯示項目報表
    在項目的操作功能中，提供了各項目的管理報表 (item report)。只要將游標移動至欲顯示報表的項目，按下滑鼠右鍵並點選「Show Report」功能。報表中包含兩個頁籤：
    * Related Items 頁籤：顯示與該項目有關的所有相關的上游與下游項目，並提供將流程圖另存為 SVG 檔的功能。
    * Execution Info 頁籤：顯示該作業記錄於控制表中的歷史執行資訊，目前提供四種報表供使用者參考，包含：執行狀態統計、計畫執行時間與開始執行時間的比較、執行時間長度統計、處理資料筆數統計。
    ![](https://i.imgur.com/9b4Cj3h.gif)


3. 針對全體項目的功能操作：
假設一份作業清單文件設定如下：

| 編號 | 類別 | JOB名稱 | 執行頻率 | JOB來源 | 非JOB註記 | 執行指令 |
| ---- | ---- | ---- | -------- | ---- | ---- | ---- |
| 1 | test | Job1 | YYYYMMDD |      |    | ```python runjob.py Job1 ${PLANDT}``` |
| 2 | test | Job2 | YYYYMMDD |      |    | ```python runjob.py Job2 ${PLANDT}``` |
| 3 | test | Job3 | YYYYMMDD |      |    | ```python runjob.py Job3 ${PLANDT}``` |
| 4 | test | Job4 | YYYYMMDD | Job1<br>Job2 |    | ```python runjob.py Job4 ${PLANDT}``` |
| 5 | test | Job5 | YYYYMMDD | Job2<br>Job3 |    | ```python runjob.py Job5 ${PLANDT}``` |
| 6 | test | Job6 | YYYYMMDD | Job4<br>Job5 |    | ```python runjob.py Job5 ${PLANDT}``` |

則產生的作業相依性流程圖如下：
![](https://i.imgur.com/z9LKXmr.png)

  * [1]. 執行所有項目
    若使用者覺得一個一個去執行項目太過於麻煩，本工具有提供能夠一次執行所有項目的功能。在產生作業相依性流程圖之後，可在設定區中先切換至「Function (功能)」頁籤，然後執行「Run All Items (執行所有項目)」功能，就能夠由最上層依序執行至最下層，直到所有項目執行完畢，或因執行錯誤導致中斷為止。
    **但必須注意，使用這個功能必須在沒有任何等待中 (waiting) 或執行中 (running) 的項目存在，以避免項目被重複執行**。若有等待中或執行中的項目時，本工具將禁止使用此功能。
    下面的例子可以看到執行本功能後，會同時先執行 Job1, Job2, Job3 三個項目，然後再按照相依性的設定，按照順序直到全部執行完畢，或是因中途發生錯誤而終止。
    ![](https://i.imgur.com/V1Qri4h.gif)


  * [2]. 繼續所有項目
    雖然上面的「Run All Items (執行所有項目)」功能能夠一次執行所有程式，但如果每次都從頭開始執行也很麻煩。因此本工具也提供能夠一次執行所有項目，並且能夠避免重複執行的功能。
    下面的例子為假設 Job2, Job3 已經先執行成功，如果使用者想要全部執行，但又想要跳過這兩個項目時，則可以使用「Continue All Items (繼續所有項目)」功能，結果會直接由 Job1 與 Job5 開始執行。
    **與 Run All Items 功能相同，使用 Continue All Items 功能必須在沒有任何等待中 (waiting) 或執行中 (running) 的項目存在，以避免項目被重複執行**。若有等待中或執行中的項目時，本工具將禁止使用此功能。
    ![](https://i.imgur.com/80QWhCV.gif)
    本功能也可使用在執行時間順序錯誤的時候。下面的例子假設 Job2, Job3, Job5 已經先執行成功，但其中 Job2 的執行時間卻晚於 Job5，造成執行時間順序錯誤 (箭頭也變為紅色)。此時使用本功能會考慮這個情況，因此會選擇由 Job1 與 Job5 開始執行，Job5 在重新執行後，畫面上的資訊也會隨著更新。
    ![](https://i.imgur.com/ekc0OlB.gif)


  * [3]. 停止所有項目
    當使用「Run All Items (執行所有項目)」或「Continue All Items (繼續所有項目)」功能，執行到一半卻想要終止時，則可以使用「Stop All Items (停止所有項目)」功能。本功能會移除正在等待中 (waiting) 的項目，但須注意若有項目正在執行中 (running) 則無法終止，仍必須等待該項目執行結束，結束之後不再繼續執行其下游項目。
    下面的例子為使用者先執行 Run All Items 功能，執行到 Job4 為 waiting、Job5 為 running 的狀態時，按下 Stop All Items 功能，因此 Job4 恢復為原本的 initial 狀態，而 Job5 則繼續 running，直到執行完畢之後才結束。
    ![](https://i.imgur.com/PtN1B4v.gif)


  * [4]. 顯示報表
    使用「Show Report (顯示報表)」功能，就能在主要操作區中產生所有項目的彙總報表 (main report)，目前提供下列六種報表：
    * 作業數量統計
    * 項目數量統計
    * 項目狀態數量統計
    * 比較計畫執行時間與實際執行時間的差距統計
    * 執行時間統計
    * 處理筆數統計
    ![](https://i.imgur.com/zy6uWga.gif)


  * [5]. 由DB載入目前所有項目的狀態 (Reload From DB)
    因為使用者可能會利用其他方式執行作業，因此提供本功能，讓使用者能夠依據控制表的執行資訊，更新主要操作區上各項目的狀態。

  * [6]. 儲存SVG (Save SVG)
  將主要操作區上的圖形儲存為 SVG 圖檔。



### (3).利用「Run On Time」做為排程工具
![](https://i.imgur.com/p8PG5Cm.png  =50%x50%)

在設定區的「Function (功能)」頁籤中，有個「Run On Time (按時執行)」核取方塊，用來限制項目在執行時是否需要依照本身的計畫執行時間來按時執行。

如果 Run On Time 有勾選 (預設)，那麼在使用「Run All Items (執行所有項目)」或「Continue All Items (繼續所有項目)」功能時，就不會執行目前時間尚未到達計畫執行時間的項目，而是先將這些項目的狀態變成 waiting (等待中)，等到目前時間到達計畫執行時間之後，才將這些項目解封，並開始執行。

如果 Run On Time 沒有勾選，那麼就不會去檢查是否已經到達計畫執行時間。這種情形通常會用在測試作業的時候。
因此勾選「Run On Time」之後，可以將 JDR 視為一個排程工具，因為未來的項目將會按時執行，使用者只需等待這些項目依序執行完畢即可。

下面以執行當天為 2023-03-12 為例，並產製出介於 2023-03-12 ~ 2023-03-13 的流程圖。

* 當 Run On Time 有勾選
按下「Run All Items (執行所有項目)」按鈕後，2023-03-12 的 Job1, Job2, Job3, Job4 便開始依序執行完畢，但 2023-03-13 的 Job1 始終保持 waiting 的狀態，這是因為目前是 2023-03-12，尚未到 2023-03-13。
![](https://i.imgur.com/OxaoiUF.gif)

* 當 Run On Time 沒有勾選
按下「Run All Items (執行所有項目)」按鈕後，即使目前尚未到 2023-03-13，所有項目都依序執行完畢。
![](https://i.imgur.com/pOMuoLb.gif)



### (4).控制表的彈性設定
控制表是用來記錄每個項目的執行狀態，不同的公司或組織或許也有類似的設計，但不一定會與本專案的設計相同 (例如控制表名稱不同、欄位名稱不同，或是狀態定義不同...)，因此本工具盡可能提供彈性，讓使用者能夠自行設定，希望這套工具能夠適用於不同的系統。

下面以其他系統為例，並說明如何在 JDR 工具中進行設定。
若其他系統的控制表名稱改為：job_exec_log2，
且對於程式的執行狀態定義為：執行中 (R )、已成功 (S)、已失敗 (F)
且欄位與本專案的設計也不同，例如：
| # | 欄位意義       | 本專案欄位名稱 | 其他系統欄位名稱 |
| --| ------------- | ----------- | ------------- |
| 1 | 作業名稱       | job_name | job_name2 |
| 2 | 計畫執行時間    | plan_dt  | plan_dt2 |
| 3 | 作業執行狀態    | status   | status2   |
| 4 | 實際開始執行時間 | act_sdt  | act_sdt2  |
| 5 | 實際結束執行時間 | act_edt  | act_edt2  |
| 6 | 處理資料筆數    | data_num | data_num2 |

使用者可以在設定區的「DB (資料庫)」頁籤中，設定控制表的名稱與欄位定義：
* Control Table Name：設定為 job_exec_log2
* Column Name (Job Name)：設定為 job_name2
* Column Name (Plan Datetime)：設定為 plan_dt2
* Column Name (Job Status)：設定為 case status2 when 'R' then 'running' when 'S' then 'success' when 'F' then 'failure' end
* Column Name (Actual Start Datetime)：設定為 act_sdt2
* Column Name (Actual End Datetime)：設定為 act_edt2
* Column Name (Data Number)：設定為 data_num2

![](https://i.imgur.com/6GDIFWU.png =50%x50%)

設定後再產製流程圖，便可以順利執行：

![](https://i.imgur.com/7k4ESG4.gif)



### (5).作業清單文件的剔錯檢查
作業清單文件是記錄組織中所有作業的一份清單文件，由管理者來維護，但既然是由人來維護，就有可能會有錯誤，因此本工具提供一些基本剔錯功能，剔錯結果會顯示於 Log 區，而且本工具會視錯誤的種類不同，而給予不同的處理方式。規則如下：

| # | 種類   | 剔錯類型             | 處理方式 |
| --| ----- | ------------------- | ----------- |
| 1 | Error | 文件欄位名稱錯誤       | 無法產生流程圖 |
| 2 | Error | 圖形具有 cycle        | 產生流程圖，但無法操作  |
| 3 | Error | 資料庫無法連線         | 產生流程圖，但無法操作   |
| 4 | Error | 作業未填「執行指令」欄位 | 產生流程圖，但無法操作，且該作業歸屬於無效類型  |
| 5 | Error | 作業未填「執行頻率」欄位 | 產生流程圖，但無法操作，且該作業歸屬於無效類型  |
| 6 | Error | 「執行頻率」欄位格式錯誤 | 產生流程圖，但無法操作，且該作業歸屬於無效類型 |
| 7 | Warning | 「JOB來源」欄位中包含未定義作業 | 產生流程圖，可以操作，但將該作業歸屬於未定義類型 |

下面以一份錯誤的作業清單文件為例，若文件的內容如下：

| 編號 | 類別 | JOB名稱 | 執行頻率 | JOB來源 | 非JOB註記 | 執行指令 |
| ---- | ---- | ---- | -------- | ---- | ---- | ---- |
| 1 | test | Job1 | YYYYMMDD | JobX |    | ```python runjob.py Job1 ${PLANDT}``` |
| 2 | test | Job2 | YYYYMMDD | Job1<br>Job4 |    | ```python runjob.py Job2 ${PLANDT}``` |
| 3 | test | Job3 | YYYYMMDD | Job1 |    | ```python runjob.py Job3 ${PLANDT}``` |
| 4 | test | Job4 | YYYYMMDD | Job2<br>Job3 |    | ```python runjob.py Job4 ${PLANDT}``` |
| 5 | test | Job5 |          |      |    | ```python runjob.py Job5 ${PLANDT}``` |
| 6 | test | Job6 | YYYYMMDD |      |    |    |
| 7 | test | Job7 | YYYYMMDDD |      |    | ```python runjob.py Job7 ${PLANDT}``` |

產生的流程圖如下 (雖然有產生流程圖，但實際上無法對其進行操作)：
![](https://i.imgur.com/GXabBkt.png)

Log 區的訊息如下：

![](https://i.imgur.com/beoHA3H.png =70%x70%)

Log 內容可以找到這些錯誤：
* [Error]：Job5 未填「執行頻率」欄位
* [Error]：Job6 未填「執行指令」欄位
* [Error]：Job7 「執行頻率」格式錯誤
* [Error]：圖形中有一個 cycle：Job2 -> Job4 -> Job2
* [Error]：資料庫無法連線
* [Warning]：「JOB來源」欄位中有個 JobX，視為未定義作業



### (6).項目狀態的顏色調整
JDR 工具本身有為各種執行狀態提供預設的顏色，但如果使用者不喜歡預設值，可以自行設定，設定後也可以改回預設值。
下面的例子示範將 success 狀態的顏色由綠色改為藍色，然後再改回預設值：

![](https://i.imgur.com/aclyPBt.gif)

---

## 七、JDR工具的安裝方式
下載網址：
環境設置：

