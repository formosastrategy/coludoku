# COLODUKU 色獨｜初級版完整規則、36 張卡牌資料及 Codex 驗證任務
版本：2026-10-20｜繁體中文｜本檔可整份貼入 Codex 作為開發規格

## 0. 給 Codex 的核心指示
請依本文件從頭實作一套可重現、可測試的《色獨 Coludoku》初級完整求解器、題庫驗證器、難度統計及選配的互動式網頁。**不要把過往解數視為先驗真理，先跑完窮舉驗證才發布結論。**不得在未確認前自行變更魔石 A/B 內容、Shape 位置、雙鎖角度、Target 配對、禁用規則。先產出差異報告與建議，修改需單獨建立候選方案。

## 1. 遊戲總覽
- 遊戲名稱：COLODUKU 色獨；抽象空間邏輯、九宮格、彩色魔石。
- 道具：9 塊獨特的正方形雙面魔石（編號 1–9）；每塊 2×2 顏色格，正面 a／背面 b；顏色編碼 1–6。
- 題卡：36 張不同 ID 的雙用途卡（ID 01–36），每張卡均有 Shape 資料與 Target 資料。最初以 01–18 與 19–36 分為兩個 18 張小盒，現在可全部混用。
- 玩家任選兩張 **不同 ID** 的卡；指派其中一張為 Shape、另一張為 Target，順序有別。
- Shape 決定九宮格中哪些「板塊位置」需放板塊、哪兩格放指定 Target，以及這兩塊的旋轉角度。
- Target 決定兩個指定魔石編號及兩個普通位置的禁用編號。兩個指定魔石均為 a 或均為 b；**Target 面別同時決定所有非 Target 魔石的面別**。
- 每個魔石編號在一道題目中最多用一次；剩餘未用魔石可留在場外。
- 所有已放入 3×3 板塊棋盤的魔石合計形成 6×6 顏色小格棋盤。每一條 6 格橫列與直行中，**已佔用格子的顏色不能重複**；沒有放板塊的位置等同空白，不參與衝突。即使某一行或列未填滿，也不能出現同色兩次。
- 目標：為全部 active 位置完成合法配置；每一組合法的「板塊編號、面別、方向、位置」配置視為一個解，不可擅自把不同旋轉／不同魔石視為同一解。

### 1.1 3×3 板塊位置編號
```text
1 2 3
4 5 6
7 8 9
```

### 1.2 色碼建議
1=紅色、2=黃色、3=綠色、4=藍色、5=紫色、6=棕／橘色。顏色判定以整數 1–6 為準，不依賴圖片色值。

### 1.3 旋轉角度
- rot 0：不轉；1：順時針 90°；2：180°；3：順時針 270°。
- Shape 指定兩塊 Target 的固定旋轉角度；其餘一般位置可自由旋轉四個方向，但不可翻到與 Target 不同的面。
- 旋轉作用於指定面本身的 2×2 色碼陣列；不能透過旋轉來當作翻面。

## 2. 九塊魔石的精確 A／B 色碼
**以下是權威輸入資料，不得依據視覺猜測而改變。**
```javascript
const BLOCKS = {
  1:{a:[[1,2],[3,4]], b:[[4,3],[6,5]]},
  2:{a:[[3,4],[5,6]], b:[[6,5],[2,1]]},
  3:{a:[[5,6],[1,2]], b:[[2,1],[4,3]]},
  4:{a:[[2,3],[5,6]], b:[[5,4],[2,1]]},
  5:{a:[[4,5],[1,2]], b:[[1,6],[4,3]]},
  6:{a:[[6,1],[3,4]], b:[[3,2],[6,5]]},
  7:{a:[[4,5],[6,1]], b:[[1,6],[3,2]]},
  8:{a:[[6,1],[2,3]], b:[[3,2],[5,4]]},
  9:{a:[[2,3],[4,5]], b:[[5,4],[1,6]]}
};
```

## 3. 完整 36 張 Shape（位置、雙鎖角度及混用解數）
資料欄位 `Target①` / `Target②` 的格式為 `位置/角度`；Target①依 Target 卡的第一枚板塊，Target②依第二枚；兩個角度都鎖定。解數是固定本 Shape 配其他 35 張 Target 時的最小/最大值。

| Shape ID | Active 位置 | Target①位置/角度 | Target②位置/角度 | 最少解 | 最多解 |
|---:|---|---|---|---:|---:|
| 01 | 1,3,5,8 | 1 / 0° | 8 / 0° | 10 | 19 |
| 02 | 2,4,6,8 | 2 / 0° | 4 / 0° | 10 | 19 |
| 03 | 3,6,7,8 | 3 / 0° | 7 / 0° | 10 | 19 |
| 04 | 1,4,8 | 1 / 90° | 8 / 0° | 3 | 6 |
| 05 | 2,4,8 | 2 / 90° | 4 / 0° | 3 | 6 |
| 06 | 3,6,7 | 3 / 90° | 7 / 0° | 3 | 6 |
| 07 | 1,2,8 | 1 / 90° | 8 / 0° | 1 | 1 |
| 08 | 1,2,4 | 2 / 90° | 4 / 0° | 1 | 1 |
| 09 | 1,3,7 | 3 / 90° | 7 / 0° | 1 | 1 |
| 10 | 1,4,5,8,9 | 1 / 0° | 8 / 270° | 2 | 12 |
| 11 | 1,2,4,6,8 | 2 / 90° | 4 / 0° | 4 | 15 |
| 12 | 2,3,4,7,9 | 3 / 0° | 7 / 90° | 4 | 15 |
| 13 | 1,4,5,8 | 1 / 0° | 8 / 270° | 1 | 4 |
| 14 | 2,3,4,5 | 2 / 0° | 4 / 90° | 2 | 5 |
| 15 | 2,3,7,8 | 3 / 0° | 7 / 90° | 1 | 4 |
| 16 | 1,2,7,8 | 1 / 0° | 8 / 0° | 1 | 4 |
| 17 | 1,2,4,5 | 2 / 0° | 4 / 0° | 1 | 4 |
| 18 | 1,3,7,9 | 3 / 0° | 7 / 0° | 1 | 4 |
| 19 | 1,3,5,8 | 1 / 0° | 8 / 0° | 10 | 19 |
| 20 | 2,4,6,8 | 2 / 90° | 4 / 90° | 10 | 19 |
| 21 | 3,6,7,8 | 3 / 90° | 7 / 90° | 10 | 19 |
| 22 | 1,4,8 | 1 / 0° | 8 / 0° | 4 | 6 |
| 23 | 2,4,8 | 2 / 0° | 4 / 0° | 4 | 6 |
| 24 | 3,6,7 | 3 / 0° | 7 / 0° | 4 | 6 |
| 25 | 1,2,8 | 1 / 90° | 8 / 0° | 1 | 1 |
| 26 | 1,2,4 | 2 / 90° | 4 / 0° | 1 | 1 |
| 27 | 1,3,7 | 3 / 90° | 7 / 0° | 1 | 1 |
| 28 | 1,4,5,8,9 | 1 / 0° | 8 / 270° | 2 | 12 |
| 29 | 1,2,4,6,8 | 2 / 90° | 4 / 0° | 4 | 15 |
| 30 | 2,3,4,7,9 | 3 / 0° | 7 / 90° | 4 | 15 |
| 31 | 1,4,5,8 | 1 / 0° | 8 / 270° | 1 | 4 |
| 32 | 2,3,4,5 | 2 / 0° | 4 / 270° | 2 | 5 |
| 33 | 2,3,7,8 | 3 / 90° | 7 / 0° | 1 | 4 |
| 34 | 1,2,7,8 | 1 / 0° | 8 / 0° | 1 | 4 |
| 35 | 1,2,4,5 | 2 / 0° | 4 / 0° | 1 | 4 |
| 36 | 1,3,7,9 | 3 / 0° | 7 / 0° | 1 | 4 |

備註：Shape ID 01–18 與 19–36 的原始 A/B 分盒分組只用於商品和標籤，不再決定解題時一般板塊面別。

## 4. 完整 36 張 Target（兩枚指定板塊、禁用板塊及混用解數）
禁用 `x a` 或 `x b` 表示該面的一般位置不能使用此編號；因為 Target 面別決定所有普通魔石的面別，程式內僅需儲存禁用編號。禁止的是 **一般位置**，並非移出整套九塊魔石。

| Target ID | 指定① | 指定② | 一般位置禁用① | 一般位置禁用② | 最少解 | 最多解 |
|---:|---|---|---|---|---:|---:|
| 01 | 1a | 5a | 3a | 7a | 1 | 14 |
| 02 | 2a | 6a | 4a | 8a | 1 | 12 |
| 03 | 3a | 4a | 5a | 9a | 1 | 12 |
| 04 | 1a | 6a | 2a | 7a | 1 | 14 |
| 05 | 2a | 4a | 3a | 8a | 1 | 14 |
| 06 | 3a | 5a | 1a | 9a | 1 | 14 |
| 07 | 1a | 8a | 3a | 9a | 1 | 19 |
| 08 | 2a | 9a | 1a | 7a | 1 | 19 |
| 09 | 3a | 7a | 2a | 8a | 1 | 19 |
| 10 | 1a | 9a | 2a | 8a | 1 | 19 |
| 11 | 2a | 7a | 3a | 9a | 1 | 19 |
| 12 | 3a | 8a | 1a | 7a | 1 | 19 |
| 13 | 4a | 8a | 2a | 5a | 1 | 12 |
| 14 | 5a | 9a | 3a | 6a | 1 | 12 |
| 15 | 6a | 7a | 1a | 4a | 1 | 12 |
| 16 | 4a | 9a | 3a | 8a | 1 | 14 |
| 17 | 5a | 7a | 1a | 9a | 1 | 14 |
| 18 | 6a | 8a | 2a | 7a | 1 | 14 |
| 19 | 1b | 5b | 4b | 7b | 1 | 12 |
| 20 | 2b | 6b | 5b | 8b | 1 | 12 |
| 21 | 3b | 4b | 6b | 9b | 1 | 12 |
| 22 | 1b | 6b | 5b | 7b | 1 | 12 |
| 23 | 2b | 4b | 6b | 8b | 1 | 12 |
| 24 | 3b | 5b | 4b | 9b | 1 | 12 |
| 25 | 1b | 8b | 3b | 9b | 1 | 19 |
| 26 | 2b | 9b | 1b | 7b | 1 | 19 |
| 27 | 3b | 7b | 2b | 8b | 1 | 19 |
| 28 | 1b | 9b | 2b | 8b | 1 | 19 |
| 29 | 2b | 7b | 3b | 9b | 1 | 19 |
| 30 | 3b | 8b | 1b | 7b | 1 | 19 |
| 31 | 4b | 8b | 2b | 5b | 1 | 12 |
| 32 | 5b | 9b | 3b | 6b | 1 | 12 |
| 33 | 6b | 7b | 1b | 4b | 1 | 12 |
| 34 | 4b | 9b | 3b | 5b | 1 | 12 |
| 35 | 5b | 7b | 1b | 6b | 1 | 12 |
| 36 | 6b | 8b | 2b | 4b | 1 | 12 |

## 5. 組合流程／玩家規則
1. 洗混 36 張卡，抽兩張 **不同 ID** 的卡。決定一張當 Shape，一張當 Target。不要按 A/B 牌組排除跨面配對。
2. 看 Shape：在 3×3 的板塊棋盤標出 active 位置；兩個固定 Target 位置依順序對應 Target 卡上的第一、第二枚魔石。
3. 看 Target：取出兩枚指定魔石，**以 Target 的面別 a 或 b 朝上**，依 Shape 的鎖定方向放入兩個 Target 格。
4. Target 卡上指定的兩個禁用編號 **不可使用於其他一般位置**。已作 Target 的魔石也不得重複使用。
5. 其他 active 位置只能擺入與 Target 同面的魔石，每個編號最多使用一次；可以旋轉 0/90/180/270°。
6. 直到所有 active 位置填滿且在展開的 6×6 色格中，每個橫列與直行的已填色碼都沒有重複，即完成。
7. 非 active 的位置必須維持空白；不要求九個位置全部填滿。

### 5.1 明確範例：Shape 01 × Target 19
- Shape 01：active = [1,3,5,8]；Target①位 1／0°、Target②位 8／0°。
- Target 19：指定 1b、5b；一般位置禁用 4b、7b。
- 固定位置 1 放 `1b` 方向 0°，位置 8 放 `5b` 方向 0°；位置 3 和 5 填入其他合法 B 面魔石。
- 位置 3 與 5 可選的魔石編號集合為 {2,3,6,8,9}（全部為 b 面），不可重複，可自由旋轉。
- 此範例的 4/7 禁用與 1/5 已用均須生效；不要錯把 Shape 01 的原分組 a 面用作普通魔石。

## 6. 題目編碼與解數判定
- 有序 pair `(shapeId,targetId)` 是一道題；兩者不得相等。`(1,19)` 與 `(19,1)` 是不同題。
- 全系列共有 36×35 = **1260 題**。其中 A Shape×A Target 306、A×B 324、B×A 324、B×B 306。
- 前次完整窮舉報表：全部 1260 題有解，所有題解數 1–19；其中 327 題恰好唯一解；全部題的解數加總 6699。這是 **待 Codex 獨立複驗的既有基準值**。
- 原始四類唯一解統計：AA=76、AB=87、BA=83、BB=81（共 327）。
- 不要將「一個解」解讀為同色等價排列；請明定等價類檢測作為額外指標，不能取代原始完整解數。

## 7. 求解演算法要求
1. 建立 3×3 大格×2×2 子格映射：board position `(r,c)` 的魔石小格 `(u,v)` 對應全球色格 `(2*r+u,2*c+v)`，r,c,u,v 均為 0-based。
2. 先固定放入 Shape 的兩塊 Target，檢查它們本身的色碼衝突；若不合法則 0 解。
3. 一般位置從未使用編號且不在 Target 禁用集合的候選魔石中挑選，同一面 `target.face`，每次枚舉四種旋轉。
4. DFS／回溯，增量維護六個小格列的 color bitset 及六個小格行的 color bitset；碰到重複色碼立即剪枝。
5. 所有 active 位置填滿即計為一個完整合法解。要可產生至少一組實際解、解數、節點訪問數、每題難度代理指標。
6. 先驗證全部 1260 題，輸出 `.csv` 包含 shapeId、targetId、targetFace、targetTiles、forbidden、locks、solutionCount、sampleSolution、visitedNodes。
7. 檢查每個 Shape 搭其他 35 個 Target 的最小／最大值、每個 Target 搭其他 35 個 Shape 的最小／最大值，與本文表格核對；不一致要列出具體 pair 及反例。
8. 以第二個獨立寫法或測試例交叉驗證，至少含旋轉方向、雙面映射、非連續 active 位置、禁用只針對一般位置、無重複編號及列/行衝突。
9. 將「最少／最多解」與玩家實際推理難度分開統計。

## 8. 需要先實作的資料結構（完整 JS）
以下資料是既有整理版，`shape.face` / `shape.targets[].req` 是**舊分盒的來源標籤**，在 `mixed` 模式中不得被用於限定普通魔石面別。以下 JS 檔片段包括 `SHAPES_BEGINNER`、`TARGETS_BEGINNER` 以及混用配對函式 `getBeginnerPuzzle`。
```javascript
/**
 * COLODUKU: Beginner complete 36-card dataset, with OPTIONAL cross-deck mixed mode.
 * 1-18 A-face cards, 19-36 B-face cards.
 * Shape: positions and target rotation locks. Target: both tile IDs, face, banned non-target IDs.
 * mode='mixed': Target face governs BOTH fixed target pieces AND all non-target pieces.
 * mode='separate': same-deck pairing only; retains existing validated behavior.
 * All 1,260 pairings in mixed mode are solvable under these constraints, with 1..19 solutions.
 * Rotation codes 0,1,2,3 are 0°,90°,180°,270° clockwise.
 */

const SHAPES_BEGINNER = [
  {
    "id": 1,
    "face": "a",
    "active": [
      1,
      3,
      5,
      8
    ],
    "targets": [
      {
        "pos": 1,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 8,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 2,
    "face": "a",
    "active": [
      2,
      4,
      6,
      8
    ],
    "targets": [
      {
        "pos": 2,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 4,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 3,
    "face": "a",
    "active": [
      3,
      6,
      7,
      8
    ],
    "targets": [
      {
        "pos": 3,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 7,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 4,
    "face": "a",
    "active": [
      1,
      4,
      8
    ],
    "targets": [
      {
        "pos": 1,
        "req": "a",
        "rot": 1
      },
      {
        "pos": 8,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 5,
    "face": "a",
    "active": [
      2,
      4,
      8
    ],
    "targets": [
      {
        "pos": 2,
        "req": "a",
        "rot": 1
      },
      {
        "pos": 4,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 6,
    "face": "a",
    "active": [
      3,
      6,
      7
    ],
    "targets": [
      {
        "pos": 3,
        "req": "a",
        "rot": 1
      },
      {
        "pos": 7,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 7,
    "face": "a",
    "active": [
      1,
      2,
      8
    ],
    "targets": [
      {
        "pos": 1,
        "req": "a",
        "rot": 1
      },
      {
        "pos": 8,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 8,
    "face": "a",
    "active": [
      1,
      2,
      4
    ],
    "targets": [
      {
        "pos": 2,
        "req": "a",
        "rot": 1
      },
      {
        "pos": 4,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 9,
    "face": "a",
    "active": [
      1,
      3,
      7
    ],
    "targets": [
      {
        "pos": 3,
        "req": "a",
        "rot": 1
      },
      {
        "pos": 7,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 10,
    "face": "a",
    "active": [
      1,
      4,
      5,
      8,
      9
    ],
    "targets": [
      {
        "pos": 1,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 8,
        "req": "a",
        "rot": 3
      }
    ]
  },
  {
    "id": 11,
    "face": "a",
    "active": [
      1,
      2,
      4,
      6,
      8
    ],
    "targets": [
      {
        "pos": 2,
        "req": "a",
        "rot": 1
      },
      {
        "pos": 4,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 12,
    "face": "a",
    "active": [
      2,
      3,
      4,
      7,
      9
    ],
    "targets": [
      {
        "pos": 3,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 7,
        "req": "a",
        "rot": 1
      }
    ]
  },
  {
    "id": 13,
    "face": "a",
    "active": [
      1,
      4,
      5,
      8
    ],
    "targets": [
      {
        "pos": 1,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 8,
        "req": "a",
        "rot": 3
      }
    ]
  },
  {
    "id": 14,
    "face": "a",
    "active": [
      2,
      3,
      4,
      5
    ],
    "targets": [
      {
        "pos": 2,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 4,
        "req": "a",
        "rot": 1
      }
    ]
  },
  {
    "id": 15,
    "face": "a",
    "active": [
      2,
      3,
      7,
      8
    ],
    "targets": [
      {
        "pos": 3,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 7,
        "req": "a",
        "rot": 1
      }
    ]
  },
  {
    "id": 16,
    "face": "a",
    "active": [
      1,
      2,
      7,
      8
    ],
    "targets": [
      {
        "pos": 1,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 8,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 17,
    "face": "a",
    "active": [
      1,
      2,
      4,
      5
    ],
    "targets": [
      {
        "pos": 2,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 4,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 18,
    "face": "a",
    "active": [
      1,
      3,
      7,
      9
    ],
    "targets": [
      {
        "pos": 3,
        "req": "a",
        "rot": 0
      },
      {
        "pos": 7,
        "req": "a",
        "rot": 0
      }
    ]
  },
  {
    "id": 19,
    "face": "b",
    "active": [
      1,
      3,
      5,
      8
    ],
    "targets": [
      {
        "pos": 1,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 8,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 20,
    "face": "b",
    "active": [
      2,
      4,
      6,
      8
    ],
    "targets": [
      {
        "pos": 2,
        "req": "b",
        "rot": 1
      },
      {
        "pos": 4,
        "req": "b",
        "rot": 1
      }
    ]
  },
  {
    "id": 21,
    "face": "b",
    "active": [
      3,
      6,
      7,
      8
    ],
    "targets": [
      {
        "pos": 3,
        "req": "b",
        "rot": 1
      },
      {
        "pos": 7,
        "req": "b",
        "rot": 1
      }
    ]
  },
  {
    "id": 22,
    "face": "b",
    "active": [
      1,
      4,
      8
    ],
    "targets": [
      {
        "pos": 1,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 8,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 23,
    "face": "b",
    "active": [
      2,
      4,
      8
    ],
    "targets": [
      {
        "pos": 2,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 4,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 24,
    "face": "b",
    "active": [
      3,
      6,
      7
    ],
    "targets": [
      {
        "pos": 3,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 7,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 25,
    "face": "b",
    "active": [
      1,
      2,
      8
    ],
    "targets": [
      {
        "pos": 1,
        "req": "b",
        "rot": 1
      },
      {
        "pos": 8,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 26,
    "face": "b",
    "active": [
      1,
      2,
      4
    ],
    "targets": [
      {
        "pos": 2,
        "req": "b",
        "rot": 1
      },
      {
        "pos": 4,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 27,
    "face": "b",
    "active": [
      1,
      3,
      7
    ],
    "targets": [
      {
        "pos": 3,
        "req": "b",
        "rot": 1
      },
      {
        "pos": 7,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 28,
    "face": "b",
    "active": [
      1,
      4,
      5,
      8,
      9
    ],
    "targets": [
      {
        "pos": 1,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 8,
        "req": "b",
        "rot": 3
      }
    ]
  },
  {
    "id": 29,
    "face": "b",
    "active": [
      1,
      2,
      4,
      6,
      8
    ],
    "targets": [
      {
        "pos": 2,
        "req": "b",
        "rot": 1
      },
      {
        "pos": 4,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 30,
    "face": "b",
    "active": [
      2,
      3,
      4,
      7,
      9
    ],
    "targets": [
      {
        "pos": 3,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 7,
        "req": "b",
        "rot": 1
      }
    ]
  },
  {
    "id": 31,
    "face": "b",
    "active": [
      1,
      4,
      5,
      8
    ],
    "targets": [
      {
        "pos": 1,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 8,
        "req": "b",
        "rot": 3
      }
    ]
  },
  {
    "id": 32,
    "face": "b",
    "active": [
      2,
      3,
      4,
      5
    ],
    "targets": [
      {
        "pos": 2,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 4,
        "req": "b",
        "rot": 3
      }
    ]
  },
  {
    "id": 33,
    "face": "b",
    "active": [
      2,
      3,
      7,
      8
    ],
    "targets": [
      {
        "pos": 3,
        "req": "b",
        "rot": 1
      },
      {
        "pos": 7,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 34,
    "face": "b",
    "active": [
      1,
      2,
      7,
      8
    ],
    "targets": [
      {
        "pos": 1,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 8,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 35,
    "face": "b",
    "active": [
      1,
      2,
      4,
      5
    ],
    "targets": [
      {
        "pos": 2,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 4,
        "req": "b",
        "rot": 0
      }
    ]
  },
  {
    "id": 36,
    "face": "b",
    "active": [
      1,
      3,
      7,
      9
    ],
    "targets": [
      {
        "pos": 3,
        "req": "b",
        "rot": 0
      },
      {
        "pos": 7,
        "req": "b",
        "rot": 0
      }
    ]
  }
];

const TARGETS_BEGINNER = [
  {
    "id": 1,
    "face": "a",
    "blocks": [
      {
        "id": 1,
        "face": "a"
      },
      {
        "id": 5,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      3,
      7
    ]
  },
  {
    "id": 2,
    "face": "a",
    "blocks": [
      {
        "id": 2,
        "face": "a"
      },
      {
        "id": 6,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      4,
      8
    ]
  },
  {
    "id": 3,
    "face": "a",
    "blocks": [
      {
        "id": 3,
        "face": "a"
      },
      {
        "id": 4,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      5,
      9
    ]
  },
  {
    "id": 4,
    "face": "a",
    "blocks": [
      {
        "id": 1,
        "face": "a"
      },
      {
        "id": 6,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      2,
      7
    ]
  },
  {
    "id": 5,
    "face": "a",
    "blocks": [
      {
        "id": 2,
        "face": "a"
      },
      {
        "id": 4,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      3,
      8
    ]
  },
  {
    "id": 6,
    "face": "a",
    "blocks": [
      {
        "id": 3,
        "face": "a"
      },
      {
        "id": 5,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      1,
      9
    ]
  },
  {
    "id": 7,
    "face": "a",
    "blocks": [
      {
        "id": 1,
        "face": "a"
      },
      {
        "id": 8,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      3,
      9
    ]
  },
  {
    "id": 8,
    "face": "a",
    "blocks": [
      {
        "id": 2,
        "face": "a"
      },
      {
        "id": 9,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      1,
      7
    ]
  },
  {
    "id": 9,
    "face": "a",
    "blocks": [
      {
        "id": 3,
        "face": "a"
      },
      {
        "id": 7,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      2,
      8
    ]
  },
  {
    "id": 10,
    "face": "a",
    "blocks": [
      {
        "id": 1,
        "face": "a"
      },
      {
        "id": 9,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      2,
      8
    ]
  },
  {
    "id": 11,
    "face": "a",
    "blocks": [
      {
        "id": 2,
        "face": "a"
      },
      {
        "id": 7,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      3,
      9
    ]
  },
  {
    "id": 12,
    "face": "a",
    "blocks": [
      {
        "id": 3,
        "face": "a"
      },
      {
        "id": 8,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      1,
      7
    ]
  },
  {
    "id": 13,
    "face": "a",
    "blocks": [
      {
        "id": 4,
        "face": "a"
      },
      {
        "id": 8,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      2,
      5
    ]
  },
  {
    "id": 14,
    "face": "a",
    "blocks": [
      {
        "id": 5,
        "face": "a"
      },
      {
        "id": 9,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      3,
      6
    ]
  },
  {
    "id": 15,
    "face": "a",
    "blocks": [
      {
        "id": 6,
        "face": "a"
      },
      {
        "id": 7,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      1,
      4
    ]
  },
  {
    "id": 16,
    "face": "a",
    "blocks": [
      {
        "id": 4,
        "face": "a"
      },
      {
        "id": 9,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      3,
      8
    ]
  },
  {
    "id": 17,
    "face": "a",
    "blocks": [
      {
        "id": 5,
        "face": "a"
      },
      {
        "id": 7,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      1,
      9
    ]
  },
  {
    "id": 18,
    "face": "a",
    "blocks": [
      {
        "id": 6,
        "face": "a"
      },
      {
        "id": 8,
        "face": "a"
      }
    ],
    "forbiddenNonTarget": [
      2,
      7
    ]
  },
  {
    "id": 19,
    "face": "b",
    "blocks": [
      {
        "id": 1,
        "face": "b"
      },
      {
        "id": 5,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      4,
      7
    ]
  },
  {
    "id": 20,
    "face": "b",
    "blocks": [
      {
        "id": 2,
        "face": "b"
      },
      {
        "id": 6,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      5,
      8
    ]
  },
  {
    "id": 21,
    "face": "b",
    "blocks": [
      {
        "id": 3,
        "face": "b"
      },
      {
        "id": 4,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      6,
      9
    ]
  },
  {
    "id": 22,
    "face": "b",
    "blocks": [
      {
        "id": 1,
        "face": "b"
      },
      {
        "id": 6,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      5,
      7
    ]
  },
  {
    "id": 23,
    "face": "b",
    "blocks": [
      {
        "id": 2,
        "face": "b"
      },
      {
        "id": 4,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      6,
      8
    ]
  },
  {
    "id": 24,
    "face": "b",
    "blocks": [
      {
        "id": 3,
        "face": "b"
      },
      {
        "id": 5,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      4,
      9
    ]
  },
  {
    "id": 25,
    "face": "b",
    "blocks": [
      {
        "id": 1,
        "face": "b"
      },
      {
        "id": 8,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      3,
      9
    ]
  },
  {
    "id": 26,
    "face": "b",
    "blocks": [
      {
        "id": 2,
        "face": "b"
      },
      {
        "id": 9,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      1,
      7
    ]
  },
  {
    "id": 27,
    "face": "b",
    "blocks": [
      {
        "id": 3,
        "face": "b"
      },
      {
        "id": 7,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      2,
      8
    ]
  },
  {
    "id": 28,
    "face": "b",
    "blocks": [
      {
        "id": 1,
        "face": "b"
      },
      {
        "id": 9,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      2,
      8
    ]
  },
  {
    "id": 29,
    "face": "b",
    "blocks": [
      {
        "id": 2,
        "face": "b"
      },
      {
        "id": 7,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      3,
      9
    ]
  },
  {
    "id": 30,
    "face": "b",
    "blocks": [
      {
        "id": 3,
        "face": "b"
      },
      {
        "id": 8,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      1,
      7
    ]
  },
  {
    "id": 31,
    "face": "b",
    "blocks": [
      {
        "id": 4,
        "face": "b"
      },
      {
        "id": 8,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      2,
      5
    ]
  },
  {
    "id": 32,
    "face": "b",
    "blocks": [
      {
        "id": 5,
        "face": "b"
      },
      {
        "id": 9,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      3,
      6
    ]
  },
  {
    "id": 33,
    "face": "b",
    "blocks": [
      {
        "id": 6,
        "face": "b"
      },
      {
        "id": 7,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      1,
      4
    ]
  },
  {
    "id": 34,
    "face": "b",
    "blocks": [
      {
        "id": 4,
        "face": "b"
      },
      {
        "id": 9,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      3,
      5
    ]
  },
  {
    "id": 35,
    "face": "b",
    "blocks": [
      {
        "id": 5,
        "face": "b"
      },
      {
        "id": 7,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      1,
      6
    ]
  },
  {
    "id": 36,
    "face": "b",
    "blocks": [
      {
        "id": 6,
        "face": "b"
      },
      {
        "id": 8,
        "face": "b"
      }
    ],
    "forbiddenNonTarget": [
      2,
      4
    ]
  }
];

function getBeginnerPuzzle(shapeId, targetId, mode = 'mixed') {
  const shape = SHAPES_BEGINNER.find(x => x.id === shapeId);
  const target = TARGETS_BEGINNER.find(x => x.id === targetId);
  if (!shape || !target || shapeId === targetId) return null;
  if (mode === 'separate' && shape.face !== target.face) return null;
  if (!['mixed', 'separate'].includes(mode)) throw new Error('Unknown mode');
  return {
    shapeId, targetId, active: [...shape.active],
    fixedTargets: shape.targets.map((slot, index) => ({
      pos: slot.pos, id: target.blocks[index].id,
      face: target.face, rot: slot.rot
    })),
    nonTargetAllowedFaces: [target.face],
    nonTargetAllowedRotations: [0, 1, 2, 3],
    forbiddenNonTarget: [...target.forbiddenNonTarget]
  };
}
```

## 9. Codex 交付成果清單
1. README.md：安裝、操作、遊戲規則、數據來源、運行步驟。
2. `data/blocks.js`、`data/shapes-beginner.js`、`data/targets-beginner.js`：分離且不可互相覆寫的版本資料。
3. `src/solver.js`：完整窮舉求解器與取得實際解的 API。
4. `scripts/verify-beginner.*`：可一次驗證所有 1260 題並輸出 CSV/JSON 的測試腳本。
5. `tests/`：至少涵蓋指定雙鎖旋轉、混用 B 面、禁用板塊、Target/Shape 同 ID 不可配、逐列逐行色碼不重複等測試。
6. 可選：一個簡潔的繁體中文遊戲模擬頁面，可選 Shape ID、Target ID，顯示 active 空位、兩枚指定板塊、禁用兩塊、當前面別、合法放置提示、檢查、解答與解數。
7. 驗證完成後產生統計摘要：1260 題有解數、唯一解數、最高解數、全部解數加總、依 ID 最少最多解以及精確唯一解 327 題清單。若不符既有基準值，務必報告，不可為了符合基準而改規則。

## 10. 明確不在本階段範圍內
- 不要與中級 AB 雙面 Target 或高級 AAB/ABB 三枚 Target 混合；本文件**只定義初級**。
- 不要重新最佳化 Shape 雙鎖角度、Target 禁用編號或魔石的 2×2 陣列；要研究改版可另開分支並出具對照。
- 不要把 Target 禁用板塊理解為整副牌永久抽走兩塊。
- 不要讓 Shape 的舊 a/b 標籤決定普通板塊面別；必須由 Target 決定。
- 不要自行添加對角線限制、區塊小宮不重複限制、必須用完 9 塊等不存在的規則。
