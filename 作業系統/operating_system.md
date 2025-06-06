# 作業系統筆記

## 目錄
1. [CPU 與 I/O 設備通信](#cpu-與-io-設備通信)
2. [並發與並行](#並發與並行)
3. [作業系統類型](#作業系統類型)
4. [核心架構](#核心架構)
5. [中斷處理](#中斷處理)
6. [記憶體階層](#記憶體階層)
7. [I/O 架構](#io-架構)
8. [系統保護](#系統保護)
9. [行程管理](#行程管理)
10. [CPU 排程](#cpu-排程)
11. [死結 (Deadlock)](#死結-deadlock)
12. [執行緒 (Thread)](#執行緒-thread)
13. [行程間通信 (IPC)](#行程間通信-ipc)
14. [同步問題](#同步問題)
15. [記憶體管理](#記憶體管理)
16. [虛擬記憶體](#虛擬記憶體)
17. [磁碟管理](#磁碟管理)
18. [檔案系統](#檔案系統)
19. [Linux 系統管理](#linux-系統管理)

---

## CPU 與 I/O 設備通信 [⬆ 回到目錄](#目錄)

### Spooling (Simultaneous Peripheral Operations On-Line)
- **定義**：將磁碟當作巨大的緩衝區，在磁碟中設置 spooling area
- **特點**：每個 I/O device 都有自己的 spooling area
- **優點**：
  - 讓不同工作的 CPU computation 和 I/O operation 同時執行
  - 使 CPU 和 I/O 同時保持忙碌狀態
- **缺點**：需要額外的記憶體空間

### Buffering
- **定義**：將記憶體當作緩衝區
- **結構**：
  - **Input Buffer**：負責接收 I/O 傳送過來的資料
  - **Output Buffer**：負責接收 CPU 所傳送的資料
- **問題**：兩個任務重疊執行，但可能產生不正確的結果
- **解決方案**：
  - Lock user memory space（有風險）
  - 將資料先送至記憶體

---

## 並發與並行 [⬆ 回到目錄](#目錄)

### Concurrent (並發)
- 單一時間點只有一個 process 執行
- 只需要一顆 CPU
- 透過快速切換造成同時執行的假象

### Parallel (並行)
- 單一時間點有多個 process 同時執行
- 需要多顆 CPU
- 真正的同時執行

---

## 作業系統類型 [⬆ 回到目錄](#目錄)

### 1. Multiprogramming System
- **定義**：在一段時間內多組 process 同時執行
- **目的**：提升 CPU 的使用率
- **機制**：透過 CPU scheduling 讓 CPU 在不同的 process 間切換（context switch）

### 2. Time Sharing System
- **特點**：
  - Multiprogramming 的一種
  - 透過 spooling 共享 I/O devices
  - 所有 User 共享 memory space
  - 使用 Round Robin CPU scheduling，規定 CPU time quantum
- **適用**：交談式且反應時間短的環境

### 3. Multiprocessor System
- **定義**：將許多 process 或一個 process 上的 subtasks 配置到不同的 CPU 上，以 parallel 方式執行
- **類型**：
  - **Symmetric Processing (SMP)**：
    - 每個 processor 具有相同功能
    - 當某個 CPU 故障仍可執行
    - 強調 load balance
  - **Asymmetric Processing (ASMP)**：
    - 每個 processor 負責不同功能
    - 效能比 SMP 好，但可靠度較差

### 4. Distributed System
- **特點**：
  - 每部機器都有自己的 Processor、Memory space、I/O device
  - 各個 process 以 network 連接
- **優點**：
  - Resource sharing
  - 滿足通訊需求
  - 計算速度高

### 5. Real Time System
- **定義**：有嚴謹的工作時間限制，必須在規定時間內完成
- **類型**：
  - **Hard Real Time System**：
    - OS 設計不能太複雜
    - 有更嚴格的工作時間要求
  - **Soft Real Time System**：
    - 即時性工作有最高優先權
    - 保證高優先權 process 必須先於低優先權 process
    - 支援 CPU scheduling 但不能提供 Aging 技術
- **問題**：Priority Inversion - 高優先權 process 被低優先權 process 把持資源
- **解決**：暫時調高低優先權 process 的優先權，等資源釋放後再調回

### 6. Clustered System
- **定義**：集合多個 CPU 以完成工作
- **類型**：
  - **Asymmetric Clustering**：負責監督某個 server，如果故障就接手
  - **Symmetric Clustering**：多部電腦同時執行應用程式，彼此監督

### 7. Handheld System
- **特點**：受限於實體大小、記憶體、處理器速度較慢
- **例子**：智慧型手機、平板電腦

---

## 核心架構 [⬆ 回到目錄](#目錄)

### Microkernel (微核心)
- **設計理念**：功能最小化，只保留最基本、最核心的功能在核心空間
- **基本功能**：
  - 低階記憶體管理
  - CPU 排程
  - 基本的訊息傳遞
- **特性**：
  - 模組化設計
  - 安全性高（功能隔離）
  - 穩定性高
  - 效能較低（不同模組通訊需要靠 IPC）
- **應用**：嵌入式系統、即時作業系統、分散式系統

### Monolithic Kernel (單體核心)
- **設計理念**：所有核心功能服務都整合到一個大的核心空間中運作
- **包含功能**：記憶體管理、行程管理、檔案管理等
- **特性**：
  - 效能高（直接在核心呼叫，較少使用 IPC）
  - 除錯困難
  - 安全性稍低（錯誤的模組可能影響其他功能）
- **應用**：Linux、傳統 UNIX

---

## 中斷處理 [⬆ 回到目錄](#目錄)

### 中斷機制
- **觸發**：電腦系統發生事件時，由硬體或軟體發送 interrupt
- **處理流程**：
  1. 查詢 interrupt vector
  2. 執行相對應的中斷服務程式（ISR - Interrupt Service Routine）

### 中斷類型
1. **External Interrupt**：CPU 以外所引發（machine check、I/O）
2. **Internal Interrupt**：CPU 所引發（overflow、除以零）
3. **Software Interrupt**：程式呼叫 system call 請求 OS 服務

### I/O 處理模式
1. **Polling I/O**：
   - 不斷檢查資源是否準備就緒
   - 效能差，占用 CPU 資源

2. **Interrupt I/O**：
   - 當 I/O 裝置準備好時，通知 CPU
   - 避免 CPU 資源浪費

3. **DMA Controller**：
   - 負責 memory 和 I/O device 間的資料傳輸
   - 傳輸過程不需要 CPU 監督
   - 需設定：
     - Memory location
     - Counter（傳輸量大小）
     - I/O command
     - Physical device location
   - **Cycle Stealing**：讓 DMA 偶爾使用 CPU 資源，發生衝突時優先給 DMA

---

## 記憶體階層 [⬆ 回到目錄](#目錄)

1. **Register**（速度最快）：
   - Program Counter
   - Instruction Register

2. **Cache Memory**：
   - 改善 CPU 對 main memory 存取速度
   - **Write Through**：Cache 更新立刻寫回 memory（耗時但資料一致）
   - **Write Back**：Cache 被置換時才寫回（快速但可能資料不一致）

3. **Main Memory**：
   - **RAM**：資料可隨意存取，容量大
   - **ROM**：資料只能讀取，容量小

4. **Secondary Storage**：磁碟、SSD

---

## I/O 架構 [⬆ 回到目錄](#目錄)

### Synchronous I/O
- I/O 完成才交還控制權給 User process
- 一段時間只有一個 I/O 運作

### Asynchronous I/O
- 立刻交還控制權給 user
- 不用等 I/O 運作完成
- OS 必須擁有 device status table，記錄各個 device 的使用狀況

---

## 系統保護 [⬆ 回到目錄](#目錄)

### 1. Dual Mode
- **目的**：保護 OS 不受 User program 破壞，保護 User program 不互相干擾
- **模式**：
  - **Monitor Mode (Kernel Mode)**：OS 掌握系統控制權
  - **User Mode**：User program 能對 CPU 進行掌控，不能執行特權指令

### 2. I/O Protection
- 防止 User program 直接使用 I/O devices
- 需經過 OS 提出 I/O request

### 3. 記憶體保護
- **Fence Register**：劃分 User program 和 OS 使用範圍
- **Base Register**：記錄 User program 的起始位置
- **Limit Register**：記錄 User program 的大小

### 4. CPU 保護
- 防止 user program 長期霸占 CPU
- 設定 timer，強迫 process 放棄 CPU

### 特權指令
- I/O 指令
- Timer/set change 指令
- Halt 指令
- 從 User mode 轉換到 Monitor mode
- 關閉中斷系統指令
- 修改暫存器內容

---

## 行程管理 [⬆ 回到目錄](#目錄)

### Process 組成
- Code Section（程式碼）
- Data Section（資料）
- Program Counter（下一個指令位置）
- CPU Register
- Stack

### Process Control Block (PCB)
每個 process 的相關資訊集合，存在 Monitor Area：
- Process ID
- 程序狀態
- 程式計數器
- CPU 暫存器
- 排班資訊
- 記憶體管理資訊
- 帳號資訊
- I/O 狀態

### 排班種類

1. **Long-term Scheduler**：
   - 從 Job queue 挑選合適的 job 載入到 memory
   - 適用於 batch system
   - 可控制 Multiprogramming degree

2. **Short-term Scheduler**：
   - 從 Ready queue 挑選 high priority process
   - 只負責挑選工作，控制權由 dispatcher 執行

3. **Medium-term Scheduler**：
   - 常用於 time sharing system
   - Memory space 不足時，將某些 process swap out 到 disk
   - 可控制 Multiprogramming degree

### Context Switch
- CPU 由一個 Process 切換到另一個 process
- 保存舊的處理程序狀態，載入新的狀態資訊
- 減少負擔的方法：
  - 提供 Multiple Register sets
  - 使用 Thread

### Dispatcher
負責將 CPU 控制權交給 short-term scheduler 所挑選的 process：
- Context switch
- Change to user mode from monitor mode
- 跳到 user process 適當的起始位置

---

## CPU 排程 [⬆ 回到目錄](#目錄)

### 衡量指標
1. **CPU Utilization**：CPU Use time / (CPU use time + CPU idle time)
2. **Throughput**：單位時間內完成的工作數
3. **Waiting Time**：在 Ready queue 等待 CPU 的時間
4. **Turnaround Time**：task 進入系統到完成工作的時間
5. **Response Time**：要求提交到第一個回應的時間

### 排班法則
- **Preemptive**：程序可被強制離開 CPU（效益較高）
- **Non-Preemptive**：程序不可被強制移除 CPU

### 排班演算法

1. **FCFS (First Come First Serve)**
   - 先到先服務
   - Non-preemptive
   - 缺點：效益差，會有 Convoy Effect

2. **SJF (Shortest Job First)**
   - Burst time 越少，越容易取得 CPU
   - 優點：排班效益最佳
   - 缺點：可能發生 starvation，CPU burst time 需預估

3. **SRTF (Shortest Remaining Time First)**
   - Remaining time 越小，越容易取得 CPU
   - Preemptive
   - Context Switch 頻繁
   - 可能發生 Starvation

4. **Priority Scheduling**
   - 優先權較高越能取得 CPU 控制權
   - 可能發生 starvation
   - 解決：Aging（逐漸提高等待程序的優先權）

5. **Round Robin**
   - 規定 CPU time slice (quantum)
   - 常用在 Time-sharing system
   - 沒有 starvation
   - Preemptive
   - 效能取決於 time quantum 大小

6. **Multilevel Queue**
   - 根據 process 特性分成不同等級的 ready queue
   - 每個 queue 可決定自己的排班法則

7. **Multilevel Feedback Queue**
   - 允許 process 在不同 queue 之間移動
   - 避免 starvation

---

## 死結 (Deadlock) [⬆ 回到目錄](#目錄)

### 定義
一組 process 陷入互相等待對方所擁有的資源

### 四個必要條件
1. **Mutual Exclusion**：資源在同一時間點只能有一個 process 使用
2. **Hold and Wait**：Process 持有部分資源並等待其他 process 的資源
3. **No Preemption**：資源不可被搶奪
4. **Circular Waiting**：形成循環等待鏈

### 處理方法

1. **死結預防**：
   - 打破 Circular Wait：賦予資源獨立編號，Process 需以遞增方式申請資源

2. **死結避免**：
   - 銀行家演算法（Banker's Algorithm）

3. **死結偵測**：
   - 建立 finish 陣列
   - 找到 Request ≤ Available，釋放 Allocation 並設定 finish 為 true
   - 重複步驟
   - 若有 finish 為 false 則陷入死結
   - 時間複雜度：O(m×n²)

4. **死結復原**：
   - 終止 process（成本高）
   - 資源搶奪：選擇 victim，剝奪其資源

### Livelock（活鎖）
- 多個進程不斷嘗試解決衝突，但無法取得進展
- 進程持續運行但沒有有效工作
- 消耗 CPU 資源

---

## 執行緒 (Thread) [⬆ 回到目錄](#目錄)

### 定義
OS 分配 CPU Time 的基本單位

### Thread 組成
- Thread ID
- Thread State
- Program Counter
- Register Set
- Stack
- 共享：Code section、Data section、Files

### 優點
1. **Responsiveness**：一個 process 只要還有 thread 在 run，該 process 還可執行
2. **Resource Sharing**：共享記憶體資源
3. **Economy**：建立和切換成本較低
4. **Scalability**：多處理器架構下的平行處理

### Thread 類型

1. **User Thread**：
   - 在 User mode 下執行
   - OS 不知道有哪些 Thread 存在
   - 缺點：blocking system call 會導致整個 process 被 block

2. **Kernel Thread**：
   - 在 Monitor mode 下執行
   - OS 知道並管理這些 Thread
   - Context Switch 較慢

### MultiThreading Models

1. **Many-to-One Model**：
   - 多個 User Threads 對應一個 Kernel Thread
   - 優點：在使用者空間執行，效率高
   - 缺點：任何 User Thread 暫停將造成整個 process 停止

2. **One-to-One Model**（主流）：
   - 每個 User Thread 對應一個 Kernel Thread
   - 更強的並行功能
   - 允許多個 Threads 在多處理器上並行執行
   - 需限制 Thread 個數

3. **Many-to-Many Model**：
   - 多個 User Thread 對應多個 Kernel Thread
   - OS 可產生所需的 Thread 數目
   - 當 Thread 中止時，可安排另一個 Thread 執行

### Thread Pool
- Process 開始執行時產生一些 Thread 放到 Pool
- 優點：限制 Thread 個數，避免系統資源耗盡

---

## 行程間通信 (IPC) [⬆ 回到目錄](#目錄)

### 行程類型
1. **獨立行程**：無法影響其他 process
2. **合作行程**：能夠影響其他 process，有共享資料

### 通信方式

#### 1. Shared Memory
- 利用共享記憶體的存取達到溝通目的
- **Race Condition**：未對共享變數提供同步機制，導致執行結果不一致
- 解決方法：
  - Disable interrupt（只適用 single processor）
  - Critical Section

#### 2. Message Passing
- Process 間需先建立 link，再互傳訊息

**Direct Communication**：
- Communication link 自動產生
- 只需知道 process ID
- 一條 link 只能連接兩個 process
- 缺點：改變 process 名稱需檢查其他 process

**Indirect Communication**：
- 透過 MailBox 傳送訊息
- 一條 link 可有兩個以上的 process

**Link Capacity**：
- **Zero Capacity**：必須有互相等待的同步架構
- **Bounded Capacity**：有限的緩衝區
- **Unbounded Capacity**：無限的緩衝區

---

## 同步問題 [⬆ 回到目錄](#目錄)

### Critical Section
對共享變數存取動作的互斥控制，需滿足：
1. **Mutual Exclusion**：任何時間點最多只允許一個 process 進入 CS
2. **Progress**：不想進入 CS 的 process 不可阻礙其他 process
3. **Bounded Waiting**：等待時間有限

### 解決方案

#### 1. Peterson's Algorithm (兩個 processes)
```
Repeat
    Flag[i] = true;
    Turn = j;
    While(flag[j] and turn = j) do-loop;
        Critical section
    Flag[i] = false;
    Remainder section
Until false;
```

#### 2. Bakery's Algorithm (N processes)
- 每個 process 抽號碼牌
- 最小號碼優先服務
- 不能保證不會抽到相同號碼

#### 3. Hardware Support
**Test and Set**：
```c
int test_and_set(int* target) {
    int temp;
    temp = *target;
    *target = 1;
    return temp;
}
```
- 原子操作，不受干擾
- 不滿足 bounded waiting

#### 4. Semaphore
用來解決 CS 和同步問題的資料型態：
- **Signal(S)**：S = S + 1
- **Wait(S)**：
  ```
  While(S <= 0) do-noop;
  S = S - 1;
  ```

**實作方式**：
1. **Busy Waiting (Spinlocks)**：
   - 不斷檢查鎖是否釋放
   - 浪費 CPU 資源

2. **Suspend/Wakeup**：
   - 無法執行時暫停
   - 主動讓出 CPU 資源

### 經典同步問題

#### 1. 生產者/消費者問題
- **無限緩衝區**：消費者需等待資訊產生
- **有限緩衝區**：生產者需等待空間，消費者需等待資訊

#### 2. 哲學家用餐問題
- 5 位哲學家和 5 支筷子
- 需要左右兩支筷子才能用餐
- 問題：全部拿左邊筷子會造成 deadlock
- 解決方案：
  - 限制同時用餐人數
  - 奇偶數哲學家拿筷子順序不同
  - 使用 semaphore 控制

### Monitor
高階同步結構，具有互斥性質：
- **組成**：
  - 一組 procedures：供外界呼叫
  - 共享資料區：共享變數
  - 初始區：共享變數初值
- **Condition 變數**：
  - `x.wait`：強制 process 暫停，放到 waiting queue
  - `x.signal`：從 waiting queue 取出第一個 process 喚醒

---

## 記憶體管理 [⬆ 回到目錄](#目錄)

### Binding 三個時期

1. **Compile Time**
   - 編譯時期決定程式的記憶體位置
   - 產生絕對位址（absolute code）
   - 程式必須從固定位置開始執行

2. **Loading Time**
   - 由 linking loader 決定，通常會做四件事：
     - Linking
     - Allocation
     - Loading
     - Relocation
   - 程式不一定由固定位置開始執行
   - 每次重新執行，只需在 linking loader 做一次
   - 支援 Relocation
   - **問題**：沒呼叫的模組仍需事先 loading

3. **Execution Time (Dynamic Time)**
   - 程序的起始位置在程式執行期間才決定
   - 需要 base Register 記錄程式起始位置
   - **優點**：彈性高
   - **缺點**：程式執行慢，效能差

### Dynamic Loading
- 在程式執行時，某個模組被真正呼叫時，才載入到 Main Memory 中
- 節省記憶體空間
- 適合大型程式

### AV-list (Available List)
- 管理記憶體空間的資料結構
- 當系統採用動態記憶體分配時，OS 需要追蹤哪些記憶體區塊是「未被使用、可供配置的」
- 這些區塊會被放進 AV-list 中

### 記憶體分區方法

1. **First-fit**
   - 所需記憶體大小為 N，從 AV-list 頭部開始找，直到找到 size ≥ N
   - **缺點**：經過多次配置後，會產生非常小的可用空間，配置機率極低

2. **Next-fit**
   - 從 AV-list 中上次配置之後的下一個 node 開始找
   - 實作方式使用 circular linked list

3. **Best-fit**
   - 在 AV-list 中找到 size ≥ N，且 size - N 值為最小的空間

4. **Worst-fit**
   - 在 AV-list 中找到 size ≥ N，且 size - N 值為最大的空間

### 記憶體碎裂問題

1. **外部碎裂 (External Fragmentation)**
   - 所有 block size 總和大於 process 需求大小
   - 但因為 block size 不連續，無法配置
   - 造成 memory 利用度低

2. **內部碎裂 (Internal Fragmentation)**
   - OS 配置給 Process 的 Memory 空間大於 process 實際所需空間
   - 此差值該 process 用不到，其他 process 也用不到

### 解決 External Fragmentation
**壓縮 (Compaction)**：
- 移動執行中的 process，使不連續的 block 聚集成夠大的連續空間
- **問題**：
  - 選擇最佳壓縮策略很難決定
  - Process 需要 dynamic binding 才可支援

### 分頁式記憶體管理 (Paging)
將 logical Address 轉換成 physical Address

**結構**：
- **Physical Address**：一組 frame 的集合，各頁框大小均相等
- **Logical Address**：User program，page 的集合，頁面大小等同於頁框大小

**優點**：
- 解決外部碎裂問題
- 支援記憶體共享和保護
- 支援 Dynamic loading 及 Virtual Memory 製作

**缺點**：
- Memory 有效存取時間長
- 會有內部碎片問題（page size 越大，越明顯）
- 需要額外硬體支援

**解決方案**：使用 TLB (Translation Lookaside Buffer) 保存常用的 page table

### 分段式記憶體管理 (Segmentation)

**結構**：
- **Physical Memory**：不需事先分配記憶體空間
- **Logical Memory**：User Program 視為 Segment 的集合，各段大小不一
- **Segment**：邏輯單位（code segment、data segment、stack segment）

**優點**：
- 沒有 internal fragmentation
- 支援 memory protection
- 支援 Virtual Memory 的製作

**缺點**：
- 可能有 external fragmentation
- 記憶體存取時間長

### 分頁式分段 (Paged Segmentation)
先分段再分頁

**優點**：沒有 external fragmentation
**缺點**：
- Table 數目過多，佔空間
- Memory access time 更長

### 反轉分頁表 (Inverted Page Table)
記錄某個頁框被哪一個 Process 持有，減少 page table 所需空間

---

## 虛擬記憶體 [⬆ 回到目錄](#目錄)

### 定義
允許 Program Size 大於 Physical Memory Size 的情況下，Program 仍可以運行

### Demand Paging
- 程式執行之初，不將全部的 page 載入到 MM 中
- 其餘 page 全數置於 blocking stores
- **Pure Demand Paging**：執行之初，不預先載入任何 page

### Page Fault
在 Virtual Memory 系統中，若執行中的 process 試圖存取不存在記憶體中的 page，則發生 page fault

### Effective Memory Access Time
公式：(1-p) × ma + p × (page fault process time)
- p：page fault ratio
- ma：正常的 Memory Access Time
- 越小越好

### Page Replacement
當 page fault 發生且 memory 無可用頁框，OS 需選擇犧牲者將其 swap out 到 blocking stores

**節省 swap out 時間方法**：
- 判斷 Victim page 執行之初到被替換，有沒有被修改過
- **Modification bit**：0 代表沒被修改，1 代表被修改，bit 為零不用 swap out

### Page Replacement 演算法

1. **FIFO (First In First Out)**
   - 最先載入的，最先挑選為 Victim page
   - 會有 Belady's Anomaly：可用 Frame 增加，有時 page fault ratio 上升
   - 原因：FIFO 不具備 stack 性質
   - 效益最差

2. **OPT (Optimal)**
   - 將來長期不會使用的 page 視為 Victim page（往後看）
   - 效果最好，page fault ratio 最低
   - 實際上無法實作（需要預知未來）

3. **LRU (Least Recently Used)**
   - 最近不常使用的，視為 Victim page（往前看）
   - 效果不錯
   - 需要硬體支援（Counters、Stack）

4. **LRU 近似法則**
   - **Second Chance**：
     - 以 FIFO 為基礎，搭配 Reference bit 使用
     - 先以 FIFO 挑出 page，檢查 Reference bit
     - 若為 1，放棄 Victim page；若為 0，替換這個頁面
     - 若 Reference bit 皆為 0 或 1，則退化成 FIFO
   
   - **Enhanced Second Chance**：
     - 使用 (Reference bit, Modification bit) 配對值作為挑選依據
     - 優先順序：(0,0) → (0,1) → (1,0) → (1,1)

5. **MFU (Most Frequently Used)**
   - 參考次數最多的 page 作為 Victim page

6. **LFU (Least Frequently Used)**
   - 參考次數最少的 page 作為 Victim page

### Thrashing
- 採用 Global Replacement Policy，Process 會搶奪其他 process 的頁框
- 造成其他 process 都在處理 page fault，CPU 利用度下降
- Process 處理 page fault 的時間遠大於正常執行時間

### 解決 Thrashing

1. **利用 Page Fault Ratio**
   - 規定上限值和下限值
   - Page fault ratio 大於上限值，OS 分配額外的頁框
   - 低於下限值，將多餘的頁框分給其他 Process

2. **Working Set Model**
   - 預估不同時期 Process 所需的頁框數
   - 基於局部性原理：
     - **Temporal Locality**：Process 所存取的記憶體，過不久再度被存取
     - **Spatial Locality**：Process 所存取的記憶體，鄰近區域極有可能被存取
   - 設定 Working set window，設定頁面所需的工作集合
   - **缺點**：
     - 不易設置精確的 Working set
     - 前後 Working set 相差太大，使得 Transfer time 拉長

### TLB Reach
- TLB 所能涵蓋到實體記憶體範圍的總和
- 公式：TLB entries 數量 × 每頁大小（Page Size）
- 擴大 TLB Reach 可以減少 page table 存取頻率

---

## 磁碟管理 [⬆ 回到目錄](#目錄)

### 空間管理方法

1. **位元向量 (Bit Vector)**
   - 該區段 free 表示 0，被占用表示 1
   - 實作簡單
   - 容易找到連續的可用空間
   - 占用大量 memory 空間

2. **Linked List**
   - 利用 linked list 將 Free block 串接，形成 AV List

3. **Combination**
   - 取某個可用編號，記錄其他可用編號，並用 linked list 相連
   - 效率仍然不佳

4. **Counting**
   - 適用於連續區塊較多的情況
   - 在 Free block 中，記錄其連續區塊可用數目
   - 連續區塊多，AV-list 可大幅縮短

### Disk Access Time
1. **Seek Time**（花費時間最多）：將 Read/Write head 移動到指定 Track 所花費的時間
2. **Latency Time**：將要存取的 Sector 轉到 Read/Write head 下方所花費的時間
3. **Transfer Time**：在 Disk 和 memory 之間的傳輸時間

### 磁碟排程演算法

1. **FCFS (First Come First Serve)**
   - 先來先服務
   - 簡單、公平（不會發生 starvation）
   - 效率不佳

2. **SSTF (Shortest Seek Time First)**
   - 距離 R/W head 最近的 track 優先服務
   - 並非是最佳的演算法
   - 不公平，可能發生 starvation

3. **SCAN 演算法 (Elevator)**
   - 讀寫頭來回不停地做掃描
   - 若有 Track 請求則服務，提供雙向服務
   - 遇到起頭和尾端才折返
   - 可能有不必要的磁軌移動

4. **C-SCAN 演算法**
   - SCAN 演算法的變形
   - 提供單向服務，回頭不提供任何服務

5. **LOOK 演算法**
   - SCAN 演算法變形，提供雙向服務
   - 處理完某方向最後一個請求即折返
   - 不需在 Track 尾端或起頭才折返

6. **C-LOOK 演算法**
   - 和 LOOK 幾乎一樣，但提供單向服務

### RAID (Redundant Array of Independent Disks)
多顆磁碟機組成一個陣列，將資料以切割的方式，同時對不同的磁碟做讀寫動作

**優點**：
- 平行 I/O 傳輸
- 安全性、可靠性提升

### RAID Levels

1. **RAID 0**
   - 將資料切割存放到多部硬碟，不會重複存放
   - 效能好，完全沒有容錯能力

2. **RAID 1 (Mirror RAID)**
   - 資料寫入時，同時寫入兩個硬碟
   - 可靠度高，Read 效能好，Write 效能差

3. **RAID 0+1**
   - 結合 RAID 0 和 RAID 1
   - 至少要四顆硬碟
   - 先進行 RAID 0 striping，再由 RAID 1 進行 mirror

4. **RAID 1+0**
   - 先 mirror 再 striping
   - 可靠性較 RAID 0+1 高，但速度較差

5. **RAID 2**
   - 以 Hamming code 進行編碼
   - 不同的位元存儲在不同的硬碟

6. **RAID 3**
   - 同位元檢查機制
   - 允許同一時間有一碟損壞仍可恢復

7. **RAID 4**
   - 和 RAID 3 幾乎一樣，以 block 為切割單位
   - 存取速度比 RAID 3 快

8. **RAID 5**
   - 資料以 block 為切割單位
   - 把同位元分散儲存於陣列中的每個硬碟中

9. **RAID 6**
   - 儲存 2 份同位元資料
   - 即使系統損壞仍能對資料進行回復
   - Write 效能差

---

## 檔案系統 [⬆ 回到目錄](#目錄)

### 檔案配置方法

1. **Contiguous Allocation**
   - 若 file 大小為 n 個 block，OS 必須在 disk 上找到 ≥ n 個連續 block
   - **優點**：
     - 支援循序存取和隨機存取
     - Seek time 較短
   - **缺點**：
     - 容易有 External Fragmentation
     - 檔案大小無法任意增刪
     - 建檔時需事先宣告大小

2. **Linked Allocation**
   - OS 在 Disk 上找到 ≥ n 個可用區塊配置（可以不連續）
   - 各區塊以 link 串接
   - **優點**：
     - 沒有外部碎裂問題
     - 檔案大小可以任意增刪
     - 建檔不需事先宣告大小
   - **缺點**：
     - 不支援 Random Access
     - Seek time 比較長（block 可能散落在不同 track）
     - Link 斷掉，所有資料都不見

3. **FAT (File Allocation Table)**
   - Linked Allocation 變形
   - 將所有配置區塊之間的 link 及 Free link 全部記錄在同一個 table

4. **Index Allocation**
   - 每個 file 會多一些額外的 block 作為 index block
   - 內含各配置的 data block 編號
   - **優點**：
     - 沒有外部碎裂
     - 支援循序存取和隨機存取
     - File 大小可以任意增刪
     - 建檔時不需事先宣告大小
   - **缺點**：
     - Index block 占用空間
     - Index block 大小設計是難題

---

## Linux 系統管理 [⬆ 回到目錄](#目錄)

### 檔案權限

#### 權限三種操作
- **r (read)**：讀取，數值對應為 4
- **w (write)**：檔案寫入，數值對應為 2
- **x (execute)**：執行，數值對應為 1

#### 權限對象
- **u (user)**：檔案的擁有者
- **g (group)**：檔案所屬群組成員
- **o (other)**：其他人

#### 權限表達式
- 有九個字元，每三個一組
- 順序為：使用者、群組、其他人
- 例如：`rwxr-xr--` 表示擁有者可讀寫執行，群組可讀執行，其他人只可讀

### 權限管理指令

#### chmod
設定權限
```bash
# 格式
chmod [誰][+|-|=][權限] 檔案

# 範例
chmod u+x file.sh      # 給 user 加上執行權限
chmod g-w file.sh      # 給 group 移除寫入權限
chmod u=rwx file.sh    # 給擁有者權限設為 rwx（不管原本權限）
chmod 755 file.sh      # 數字表示法：rwxr-xr-x
```

#### chown
更改檔案擁有者和群組
```bash
# 格式
chown [選項] [新擁有者]:[新群組] 檔案名稱/路徑

# 範例
chown user:group file.txt
chown -R user:group directory/  # 遞迴更改
```

#### chgrp
更改群組
```bash
# 格式
chgrp [選項] 新群組 檔案/目錄

# 範例
chgrp developers file.txt
```

### 常見 Linux 指令

- **ls**：列出檔案目錄
- **cat**：查看檔案內容
- **cp**：複製檔案
- **mv**：移動或重新命名
- **rm**：刪除
- **find**：查找檔案
- **ps**：顯示執行中的程式

### grep 指令與正則表達式

#### grep 格式
```bash
grep [選項] [搜尋內容] 檔案名稱
```

#### 正則表達式符號
- **$**：匹配結尾字串
- **^**：匹配開頭字串
- **.** ：匹配任意字符
- **\***：0 次或多次
- **+**：1 次或多次
- **?**：0 次或 1 次

#### 常用選項
- **-i**：忽略大小寫
- **-n**：顯示行號
- **-r**：遞迴搜尋所有檔案
- **-v**：不包含（反向搜尋）
- **-l**：只顯示檔案名稱

### 範例應用
```bash
# 搜尋包含 "error" 的行
grep "error" log.txt

# 搜尋開頭是 "Error" 的行（忽略大小寫）
grep -i "^error" log.txt

# 搜尋結尾是 ".txt" 的檔案
ls | grep "\.txt$"

# 在目錄中遞迴搜尋，顯示行號
grep -rn "TODO" ./src/
```

---

## 補充概念 [⬆ 回到目錄](#目錄)

### Fork 系統呼叫
- 子程序拿到的返回值是 0
- 父程序拿到的是子程序的 PID

### System Call vs Trap
- **Trap**：通知 OS 要求一個服務（舉手）
- **System Call**：通知 OS 要什麼樣的服務（具體要求）

