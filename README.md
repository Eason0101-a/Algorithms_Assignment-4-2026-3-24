## 平行離散獅群最佳化演算法 (PDLSO)

## 專案概述
[cite_start]本專案提出「平行離散獅群最佳化演算法」(Parallel Discrete Lion Swarm Optimization, 簡稱 PDLSO) [cite: 1, 3][cite_start]，旨在作為破解旅行推銷員問題 (TSP) 的高效能拓撲策略 [cite: 2][cite_start]。面對屬於 NP-Complete 的無盡迷宮 TSP [cite: 6][cite_start]，傳統的精確解法（如動態規劃、分支界限法）會在龐大的解空間中耗盡運算資源 [cite: 7, 10][cite_start]。本專案的目標轉向如何在最短時間內，獵取精確度最高的近似全局最佳解 [cite: 12]。

[cite_start]PDLSO 的核心應用領域涵蓋電腦網路、物流配送與大規模組合最佳化 [cite: 4]。

---

## 演算法演進與基因突變
[cite_start]原始的獅群演算法 (LSO) 專為連續函數設計 [cite: 14][cite_start]，其移動公式會產生非整數座標 [cite: 22][cite_start]，直接套用於強調順序結構的 TSP 會徹底破壞漢彌爾頓迴路 (Hamiltonian loop) 條件，這是其致命缺陷 [cite: 22][cite_start]。為了解決此問題並突破算力極限，演算法經歷了以下進化 [cite: 24]：

| 特徵 | 連續空間 (LSO) | 離散空間 (DLSO) | 平行空間 (PDLSO) |
| :--- | :--- | :--- | :--- |
| **編碼架構** | [cite_start]實數座標 [cite: 25] | [cite_start]整數離散編碼 (漢彌爾頓迴路) [cite: 25] | [cite_start]多群體獨立編碼 [cite: 25] |
| **移動機制** | [cite_start]連續空間公式加減 [cite: 25] | [cite_start]OX順序交配算子繼承結構 [cite: 25] | [cite_start]平行節點獨立演化 [cite: 25] |
| **局部搜尋** | [cite_start]演算法原生機制 [cite: 25] | [cite_start]結合完整 2-opt (C2-opt) 強化 [cite: 25] | [cite_start]針對菁英個體頻繁執行 C2-opt [cite: 25] |
| **速度與規模** | [cite_start]單機串行運算 [cite: 25] | [cite_start]中小規模高精度求解 [cite: 25] | [cite_start]環狀拓撲，實現大規模數據線性加速 [cite: 25] |

[cite_start]為了征服 TSP，本專案不僅將獅子的基因離散化，更將獅群狩獵的戰術平行化 [cite: 26]。

---

## 核心機制與拓撲策略

### 1. 離散編碼與 OX 順序交配算子
* [cite_start]**漢彌爾頓解碼環**：個體由傳統實數編碼轉變為長度為 n 的整數陣列 [cite: 40][cite_start]，陣列中無重複元素，構成唯一的完美映射漢彌爾頓迴路 [cite: 34, 40][cite_start]。適應度函數即為迴路權重的總和 [cite: 40]。
* [cite_start]**完美繼承 (OX 算子)**：以 OX 算子取代傳統的加減法 [cite: 57][cite_start]。透過定位擷取、核心轉移與序列繞流的步驟 [cite: 43, 46, 48][cite_start]，確保子代能保留親代的相對順序特徵，使得離散移動變得合法且高效 [cite: 57]。

### 2. 階層移動與局部強化
* [cite_start]**階層移動機制**：透過 OX 算子，將獅群的原始狩獵本能完美轉譯為離散空間的最佳化搜尋法則 [cite: 70]。
    * [cite_start]**獅王 (Lion King)**：緊咬全局最佳解的軌跡 ($p_{i}^{t}\circ\times g^{t}$)，小範圍繞行 [cite: 18, 62]。
    * [cite_start]**母獅 (Lioness)**：雙點探索 ($p_{i}^{t}\circ\times p_{c}^{t}$)，結伴移動以擴展未知領域 [cite: 19, 63]。
    * [cite_start]**幼獅 (Lion Cub)**：多元跟隨與反向學習 [cite: 20][cite_start]，當 $1/3<p<2/3$ 時執行 $p_{i}^{t}\circ\times p_{m}^{t}$ 確保族群多樣性 [cite: 65, 66, 67]；否則執行 $p_{i}^{t}\circ\times g^{t}$ [cite: 68, 69]。
* **C2-opt 手術刀**：為了解決原生 DLSO 局部收斂速度緩慢的缺陷 [cite: 89]，在每次迭代中強制對 1 隻獅王、2 隻最佳母獅與 1 隻最佳幼獅執行完整 2-opt 運算 [cite: 89]。此雙劍合璧的策略能大幅消除局部交叉路徑，瞬間提升解的精確度 [cite: 89]。

### 3. 環狀拓撲平行運算
* **多群體分裂**：將總體拆分為 N 個子群體 ($SN_{sub}=SN/N$) [cite: 151, 159]，透過 MPI (訊息傳遞介面) 分配至多個處理器核心進行獨立平行運算，以突破單機運算瓶頸 [cite: 159]。
* **智慧遷移法則**：採用環狀拓撲結構 [cite: 161]，每間隔 10 代發動一次遷移 [cite: 164, 165]。系統會提取節點 k 中的全局最佳解 (菁英) [cite: 167, 168]，並順時針覆寫精準替換相鄰節點 k+1 中的最弱個體 [cite: 175]。
* **拓撲優勢**：相較於星狀或全連接拓撲，環狀拓撲收斂較為平緩 [cite: 176]，能保持子群體間的多樣性，避免過早陷入局部最佳解 [cite: 176]。

---

## 效能驗證與實驗結果

### 精確度表現
* 在低於 200 個城市的 TSP 問題中，DLSO 的平均誤差穩定控制在 1% 以下 [cite: 130]。
* 相比於改良型的蟻群演算法 (ACO Variants) [cite: 131]，DLSO 將精確度提升了高達 1% 到 10% [cite: 132] (例如在 KroA100 測試中，DLSO 誤差為 0.41%，而 ACO 為 10.15% [cite: 109, 110])。

### 平行加速與時間縮減
* 面對大於 1000 個城市的規模，單一獅群的計算時間會急遽飆升至超過 34,000 秒 [cite: 143]。
* 導入多進程後，PDLSO 的運算時間呈現近乎完美的比例縮減 [cite: 202]。例如在 Pcb442 測試中，運算時間從串行的 2883 秒下降至 4-process 的 782 秒 [cite: 192, 194, 197]。
* 即使群體切分導致單一子群體個體數減少，最終解的精確度誤差變動仍小於 0.5%，與串行 DLSO 保持高度一致 [cite: 199, 202]。

### 突破通訊瓶頸與網路強健性
* **印證 Gustafson's Law**：隨著測試問題規模變大，平行效率 (EFF) 不降反升 [cite: 216]（在多項測試中維持在 0.90 以上 [cite: 214, 215, 217]），證明問題越龐大，通訊時間佔比越小 [cite: 226]。
* **分散式系統佈署**：雙機網路傳輸的延遲極小 [cite: 246]，且無線與有線網路的表現幾乎沒有顯著差別 [cite: 247]，證明 PDLSO 在鬆散耦合的網路環境中具有極高的強健性 [cite: 247]。

---

## 專案總結
PDLSO 完美結合了掠食法則與拓撲策略 [cite: 249]。透過 OX 算子與離散編碼解決了連續方程式的根本衝突 [cite: 251]；並藉由環狀拓撲與 MPI 技術，在硬體叢集中實現了高達 90% 以上的平行效率，無懼龐大數據擴張 [cite: 253]。PDLSO 不僅重新定義了啟發式演算法的效能邊界，更為大規模組合最佳化問題提供了一套無懈可擊的拓撲策略 [cite: 257]。

---

## 參考文獻
* Z. Daoqing and J. Mingyan, "Parallel discrete lion swarm optimization algorithm for solving traveling salesman problem," in *Journal of Systems Engineering and Electronics*, vol. 31, no. 4, pp. 751-760, Aug. 2020, doi: 10.23919/JSEE.2020.000050.
