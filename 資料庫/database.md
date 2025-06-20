# 資料庫系統完整學習筆記

## 目錄
- [資料庫系統架構](#資料庫系統架構)
- [完整性法則](#完整性法則)
- [SQL 語言分類](#sql-語言分類)
- [ACID 特性](#acid-特性)
- [資料庫安全機制](#資料庫安全機制)
- [同步控制機制](#同步控制機制)
- [ER Model](#er-model)
- [關聯正規化](#關聯正規化)
- [分散式資料庫系統](#分散式資料庫系統)
- [NoSQL 資料庫](#nosql-資料庫)
- [大數據處理](#大數據處理)
- [儲存系統](#儲存系統)
- [檔案類型](#檔案類型)
- [關聯代數運算](#關聯代數運算)
- [EER Model (Enhanced Entity-Relationship Model)](#eer-model-enhanced-entity-relationship-model)
- [效能調校](#效能調校)

---

## 資料庫系統架構
[⬆️ 回到目錄](#目錄)

### 三大層次架構（Three-Schema Architecture）

**外部層 (External Level / View Level)**
- 提供不同使用者所能看到的視圖
- 例如：某部門員工只能看到姓名與部門，而不是薪資
- 提供資料安全性和客製化介面

**概念層 (Conceptual Level / Logical Level)**
- 定義整個資料庫的邏輯架構和資料關聯
- 描述資料庫中有哪些實體、屬性和關係
- 獨立於物理儲存方式

**內部層 (Internal Level / Physical Level)**
- 描述資料如何實際存儲在磁碟中
- 包含索引結構、檔案組織方式
- 與系統效能直接相關

---

## 完整性法則
[⬆️ 回到目錄](#目錄)

### 實體完整性 (Entity Integrity)
- **規則**：主鍵 (Primary Key) 不可為空值 (NULL)
- **目的**：確保每筆記錄都能被唯一識別

### 參考完整性 (Referential Integrity)
外來鍵的處理方式：

**Restricted (限制)**
- 當主要表的記錄被刪除或主鍵被更新時，系統會報錯
- 保護資料的完整性，但可能造成操作不便

**Cascade (串聯)**
- 自動更新或刪除相關的外來鍵資料
- 方便但需謹慎使用，避免意外刪除重要資料

**Set Null**
- 將外來鍵設為 NULL（補充）

**Set Default**
- 將外來鍵設為預設值（補充）

### 定義域完整性 (Domain Integrity)
- 所有資料欄位中的數值都符合該欄位被定義的限制條件
- 包含資料型態、長度、範圍等限制

---

## SQL 語言分類
[⬆️ 回到目錄](#目錄)

### 資料定義語言 (Data Definition Language, DDL)

**表格操作**
```sql
CREATE TABLE    -- 建立表格
ALTER TABLE     -- 修改表格結構
DROP TABLE      -- 刪除表格
```

**索引操作**
```sql
CREATE INDEX    -- 建立索引（加快查詢速度的資料結構）
ALTER INDEX     -- 修改索引
DROP INDEX      -- 刪除索引
```

### 資料處理語言 (Data Manipulation Language, DML)
```sql
INSERT    -- 新增資料
UPDATE    -- 修改資料
DELETE    -- 刪除資料
```

### 資料查詢語言 (Data Query Language, DQL)
```sql
SELECT    -- 查詢資料
```

### 資料控制語言 (Data Control Language, DCL)
```sql
GRANT     -- 授予使用者權限
REVOKE    -- 取消使用者權限
DENY      -- 拒絕權限，防止透過 GRANT 取得權限
```

### 交易管理指令 (Transaction Control Language, TCL)
```sql
COMMIT        -- 確認資料庫交易
ROLLBACK      -- 復原資料庫交易
SET TRANSACTION  -- 為交易命名
SAVEPOINT     -- 建立交易記號，方便 Rollback
```

---

## ACID 特性
[⬆️ 回到目錄](#目錄)

### 原子性 (Atomicity)
- 交易必須為「全部執行」或「全部不執行」
- 不允許部分完成的交易

### 一致性 (Consistency)
- 交易執行前後，資料庫都必須處於一致狀態
- 不違反任何完整性約束

### 隔離性 (Isolation)
- 交易過程中使用的資料，不允許其他交易讀取或寫入
- 避免交易間的干擾

### 持久性 (Durability)
- 交易一旦確認完成，對資料庫的變更就是永久有效
- 即使系統當機也不會遺失

---

## 資料庫安全機制
[⬆️ 回到目錄](#目錄)

### 自由選擇存取控制 (Discretionary Access Control, DAC)
- 由資料擁有者設定資料權限
- 每個資料都有存取控制清單 (Access Control List, ACL)
- **ACL**：列出哪個角色對某個資源可以執行哪些操作

### 角色存取控制 (Role-Based Access Control, RBAC)
- 透過角色來分配權限
- 使用者擁有角色，角色擁有權限
- 集中式管理，適合大型系統

### 強制存取控制 (Mandatory Access Control, MAC)
- 系統集中控制，使用者無法自行更改存取權限
- 每個使用者都有不同的安全等級：
  - Top Secret
  - Secret
  - Confidential
  - Unclassified

---

## 同步控制機制
[⬆️ 回到目錄](#目錄)

### 鎖定機制 (Locking)

**Binary Locking**
- 要存取資料 X 時，將資料 X 鎖定

**共享互斥鎖定 (Shared/Exclusive Locking)**
- **Shared Lock (讀取鎖)**：其他交易可以讀取，但不能寫入
- **Exclusive Lock (寫入鎖)**：只有該交易可以獨占使用
- **Unlock**：釋放鎖定

**鎖定升級/降級**
- **Upgrade**：Unlock → Read Lock → Write Lock
- **Downgrade**：Write Lock → Read Lock → Unlock

### 時間戳記排序 (Timestamp Ordering)
- 每個交易進入系統時給予時間戳記
- 依據時間戳記決定執行順序
- 不會發生死結 (Deadlock)

### 多版本同步控制 (Multi-Version Concurrency Control, MVCC)
- 每次資料修改時保留舊版本
- 執行時選擇適當版本，避免資料不一致

### 兩階段鎖定 (Two-Phase Locking, 2PL)

**階段**
- **擴展階段 (Expanding)**：可以加鎖，不能解鎖
- **收縮階段 (Shrinking)**：可以解鎖，不能加鎖

**類型**
- **基本型**：逐漸鎖定需要的項目
- **保守型**：交易開始前先鎖定所有需要的項目
- **嚴格型**：直到交易結束才釋放所有鎖定

**優點**：資料一致性高
**缺點**：可能發生死結

### 樂觀同步控制 (Optimistic Concurrency Control)
- 假設衝突很少發生
- 允許所有交易自由讀取和修改
- 在 COMMIT 時才檢查衝突

**階段**
1. **讀取階段**：讀取和修改資料，但不提交
2. **驗證階段**：檢查是否有其他交易修改相關資料
3. **寫入階段**：如果驗證成功則提交

### 分散式交易協定

**兩階段提交協定 (Two-Phase Commit, 2PC)**
1. **準備階段**：協調者詢問所有參與者是否準備好提交
2. **提交階段**：根據回應決定提交或回滾

**三階段提交協定 (Three-Phase Commit, 3PC)**
1. **CanCommit**：詢問是否準備好提交
2. **PreCommit**：進入預提交狀態
3. **DoCommit**：正式提交

---

## ER Model
[⬆️ 回到目錄](#目錄)

### 基本組成

**實體 (Entity)**
- 可以想像成資料表
- 代表現實世界中的物件或概念

**屬性 (Attribute)**
- 可以想像成資料表中的欄位
- 描述實體的特性

**關係 (Relationship)**
- 描述實體間的關聯
- 一對一、一對多、多對多

### 實體類型

**強實體 (Strong Entity)**
- 有自己的主鍵，能獨立存在
- 不依賴其他實體

**弱實體 (Weak Entity)**
- 沒有自己的主鍵
- 依賴強實體存在

### 特殊關係

**遞迴關係 (Recursive Relationship)**
- 實體與自身建立的關係
- 例如：員工-管理者關係

### 各種鍵值

**Super Key (超鍵)**
- 能唯一識別記錄的欄位集合
- 例如：{student_id}, {email}, {student_id, email}

**Candidate Key (候選鍵)**
- 最小的 Super Key
- 移除任何欄位都會失去唯一性

**Primary Key (主鍵)**
- 從 Candidate Key 中選擇一個作為主要識別

**Foreign Key (外來鍵)**
- 參考其他表格主鍵的欄位

---

## 關聯正規化
[⬆️ 回到目錄](#目錄)

### 目的
- 降低資料重複
- 減少更新異常
- 提高資料完整性

### 正規化層次

**第一正規化 (1NF)**
- 每個欄位只能存放單一值
- 不能有重複的欄位群組
- 例如：將「興趣：籃球,游泳,跑步」拆分成多筆記錄

**第二正規化 (2NF)**
- 符合 1NF
- 非主鍵欄位必須完全依賴主鍵
- 消除部分依賴：如果主鍵是複合鍵 AB，則不能有 B → C 的依賴

**第三正規化 (3NF)**
- 符合 2NF
- 消除遞移依賴：非主鍵欄位不能依賴其他非主鍵欄位
- 例如：學生編號 → 科系 → 科系主任（應拆分）

**Boyce-Codd 正規化 (BCNF)**
- 符合 3NF
- 消除非候選鍵對其他欄位的依賴
- 避免更新、插入、刪除異常

### 正規化的權衡
- **優點**：減少資料重複，提高一致性
- **缺點**：增加 JOIN 操作，可能影響查詢效能

---

## 分散式資料庫系統
[⬆️ 回到目錄](#目錄)

### 同質性分散資料庫 (Homogeneous Distributed Database)
**特性**
- 所有節點使用相同的 DBMS
- 相同的查詢語言和資料結構
- 由上而下的設計

**優點**
- 系統相容性高
- 系統整合容易
- 維護簡單

**缺點**
- 缺乏彈性
- 不利於系統多樣化

### 異質性分散資料庫 (Heterogeneous Distributed Database)
**特性**
- 各節點可使用不同的 DBMS
- 不同的資料模型和結構
- 由下而上的設計
- 需要中間層進行溝通

**優點**
- 靈活性高
- 可擴展性佳
- 可利用既有系統

**缺點**
- 查詢整合複雜
- 維護成本高
- 資料同步困難

### 資料分割 (Database Partitioning)

**水平分割 (Horizontal Partitioning)**
- 依照資料列 (Row) 進行分割
- 例如：2022年資料放第一區，2023年資料放第二區

**垂直分割 (Vertical Partitioning)**
- 依照欄位 (Column) 進行分割
- 以主鍵作為分割依據

**優點**
- 提高查詢效能
- 易於水平擴展
- 減少 I/O 負擔

**缺點**
- 系統設計複雜
- 跨分割查詢困難

---

## NoSQL 資料庫
[⬆️ 回到目錄](#目錄)

### 特性
- 非關聯式資料庫
- 強調彈性的資料模型
- 高可用性和高效能
- 優秀的水平擴展能力

### 類型

**鍵值資料庫 (Key-Value)**
- 最簡單的 NoSQL 模型
- 例如：Redis (記憶體資料庫)
- 適用：快取、會話儲存

**文件型資料庫 (Document)**
- 儲存 JSON/BSON 格式文件
- 例如：MongoDB
- 適用：內容管理、商品目錄

**圖形資料庫 (Graph)**
- 儲存節點和邊的關係
- 例如：Neo4j
- 適用：社交網路、推薦系統

**欄位型資料庫 (Column-Family)**
- 每列可有不同欄位
- 例如：Cassandra
- 適用：大數據分析、時間序列

### CAP 理論
在分散式系統中，無法同時滿足三個特性：

**一致性 (Consistency)**
- 所有節點同時看到相同資料

**可用性 (Availability)**
- 系統持續提供服務

**分區容忍性 (Partition Tolerance)**
- 系統在網路分割時仍能運作

### NoSQL 的限制
- JOIN 操作難以實現
- 資料量不夠大時，效能可能不如關聯式資料庫
- 缺乏標準化查詢語言
- 事務支援較弱

---

## 大數據處理
[⬆️ 回到目錄](#目錄)

### Hadoop 生態系統
**Apache Hadoop**
- 開源分散式運算框架
- 處理 PB 等級的大數據
- 在普通硬體上運行
- 具備高容錯性和可擴展性

### MapReduce 程式模型
**架構**
1. **Map**：將大數據切割成 key-value 對
2. **Shuffle**：將相同 key 的資料分組到同一個 Reduce 節點
3. **Reduce**：合併相同 key 的資料，產生最終結果

**特點**
- 適合批次處理
- 自動處理節點失效
- 程式設計模型簡單

### HDFS (Hadoop Distributed File System)
**特性**
- 分散式檔案系統
- 自動將大檔案切分儲存
- 提供容錯機制
- 支援超大檔案 (TB 等級)

**架構**
- **NameNode**：管理檔案系統元數據
- **DataNode**：實際儲存資料
- **Secondary NameNode**：備份 NameNode

---

## 儲存系統
[⬆️ 回到目錄](#目錄)

### NAS (Network Attached Storage)
**特性**
- 透過網路提供檔案儲存服務
- 專門的檔案伺服器
- 使用標準網路協定 (NFS, CIFS)

**適用場景**
- 檔案共享
- 備份儲存
- 多媒體儲存

### SAN (Storage Area Network)
**特性**
- 專用高速網路 (通常是光纖)
- 區塊層級的資料存取
- 高效能、低延遲

**適用場景**
- 資料庫儲存
- 資料倉儲
- 關鍵業務應用

### 比較
| 特性 | NAS | SAN |
|------|-----|-----|
| 存取方式 | 檔案層級 | 區塊層級 |
| 網路類型 | 標準乙太網路 | 專用光纖網路 |
| 效能 | 中等 | 高 |
| 成本 | 較低 | 較高 |
| 管理複雜度 | 簡單 | 複雜 |

---

## 檔案類型
[⬆️ 回到目錄](#目錄)

### 主檔案 (Master File)
- 儲存長期且較少更新的資料
- 例如：員工基本資料、產品資訊

### 交易檔案 (Transaction File)
- 內容具有實時性
- 資料經常變動
- 例如：銷售記錄、庫存異動

### 報表檔案 (Report File)
- 由系統產生的報表資料
- 用於決策支援和分析

### 歷史檔案 (History File)
- 有保存期限的歷史資料
- 用於趨勢分析和稽核

### 備份檔案 (Backup File)
- 為避免資料損毀而製作的複製檔
- 定期備份，確保資料安全

### 歸檔檔案 (Archive File)
- 保存不常使用或已下線的資料
- 長期保存，偶爾存取

---

## 關聯代數運算
[⬆️ 回到目錄](#目錄)

### 基本運算

**選擇 (Selection, σ)**
- 依照條件篩選資料列
- σ條件(關聯)

**投影 (Projection, π)**
- 選擇特定欄位
- π欄位(關聯)

**聯集 (Union, ∪)**
- 合併兩個關聯的所有資料
- R1 ∪ R2

**差集 (Difference, -)**
- 存在 R1 但不存在 R2 的資料
- R1 - R2

**交集 (Intersection, ∩)**
- 同時存在於 R1 和 R2 的資料
- R1 ∩ R2

**笛卡爾積 (Cartesian Product, ×)**
- 所有可能的配對組合
- R1 × R2

### 進階運算

**合併 (Join, ⋈)**
- **等值合併 (Equi-Join)**：使用等號條件
- **自然合併 (Natural Join)**：自動根據相同欄位名稱合併
- **外部合併 (Outer Join)**：保留未匹配的資料
  - Left Join：保留左邊所有資料
  - Right Join：保留右邊所有資料
  - Full Join：保留兩邊所有資料

**半合併 (Semi-Join, ⋉)**
- 只返回第一個關聯中與第二個關聯匹配的資料

**除法 (Division, ÷)**
- 找出包含另一關聯所有值的資料

---

## EER Model (Enhanced Entity-Relationship Model)
[⬆️ 回到目錄](#目錄)

### 物件導向特性

**特化 (Specialization)**
- 從父類延伸出子類
- **完全特化 (Total)**：所有父類實體都屬於某個子類(例如，每個人都要選隊伍，沒有中立者)
- **部分特化 (Partial)**：部分父類實體不屬於任何子類(例如，有些人有加入社團，有些人沒有)

**概化 (Generalization)**
- 從多個子類歸納出父類
- 抽象化的過程

**繼承 (Inheritance)**
- 子類繼承父類的屬性和關係

### 聚合關係

**組合 (Composition)**
- 強聚合關係
- 部分不能獨立於整體存在

**聚集 (Aggregation)**
- 弱聚合關係
- 部分可以獨立存在

---

## 效能調校
[⬆️ 回到目錄](#目錄)

### 索引策略
**索引類型**
- **主索引 (Primary Index)**：基於主鍵
- **輔助索引 (Secondary Index)**：基於非主鍵欄位
- **複合索引 (Composite Index)**：多欄位索引
- **唯一索引 (Unique Index)**：確保值的唯一性

**索引設計原則**
- 經常用於 WHERE 條件的欄位
- 經常用於 JOIN 的欄位
- 避免在經常異動的欄位建立索引
- 考慮索引維護成本

### 查詢最佳化
**成本導向最佳化**
- 評估不同執行計畫的成本
- 選擇成本最低的執行方式

**規則導向最佳化**
- 應用預定義的最佳化規則
- 例如：將選擇運算儘早執行

---