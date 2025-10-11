# ROOTアイテム検索

## 📖 問題文（日本語訳）

あるゲームで使用されるアイテムはアップグレードが可能です。

**アイテムの階層構造**：
- `ITEM_A` → `ITEM_B` のようにアップグレード可能な場合
- `ITEM_A` を `ITEM_B` の PARENT アイテムと呼ぶ
- PARENT アイテムがないアイテムを ROOT アイテムと呼ぶ

**例**：`ITEM_A` → `ITEM_B` → `ITEM_C` の場合
- `ITEM_C` の PARENT アイテム：`ITEM_B`
- `ITEM_B` の PARENT アイテム：`ITEM_A`
- ROOT アイテム：`ITEM_A`（PARENT がない）

このゲームで使用されるアイテム情報とアイテム関係を管理する2つのテーブルから、ROOT アイテムを検索してください。

**条件**：
- ITEM_ID（アイテムID）、ITEM_NAME（アイテム名）を出力
- 結果はアイテムIDを基準に昇順でソート
- 各アイテムは1つのPARENTアイテムIDのみを持つ
- ROOTアイテムのPARENT_ITEM_IDはNULL
- ROOTアイテムが存在しないケースはない

## 📊 テーブル構造

### ITEM_INFO テーブル（アイテム情報）

| Column name | Type       | Nullable | 説明             |
|-------------|------------|----------|------------------|
| ITEM_ID     | INTEGER    | FALSE    | アイテムID       |
| ITEM_NAME   | VARCHAR(N) | FALSE    | アイテム名       |
| RARITY      | INTEGER    | FALSE    | アイテムの希少度 |
| PRICE       | INTEGER    | FALSE    | アイテムの価格   |

### ITEM_TREE テーブル（アイテム関係）

| Column name      | Type    | Nullable | 説明                 |
|------------------|---------|----------|----------------------|
| ITEM_ID          | INTEGER | FALSE    | アイテムID           |
| PARENT_ITEM_ID   | INTEGER | TRUE     | PARENTアイテムのID   |

**注意事項**：
- 各アイテムは1つのPARENTアイテムIDのみを持つ
- ROOTアイテムのPARENT_ITEM_IDはNULL

### サンプルデータ

**ITEM_INFO テーブル**：

| ITEM_ID | ITEM_NAME | RARITY | PRICE |
|---------|-----------|--------|-------|
| 0       | ITEM_A    | COMMON | 10000 |
| 1       | ITEM_B    | LEGEND | 9000  |
| 2       | ITEM_C    | LEGEND | 11000 |
| 3       | ITEM_D    | UNIQUE | 10000 |
| 4       | ITEM_E    | LEGEND | 12000 |

**ITEM_TREE テーブル**：

| ITEM_ID | PARENT_ITEM_ID |
|---------|----------------|
| 0       | NULL           |
| 1       | 0              |
| 2       | 0              |
| 3       | NULL           |
| 4       | 3              |

**階層構造の可視化**：
```
ITEM_A (0)             ITEM_D (3)
├── ITEM_B (1)         └── ITEM_E (4)
└── ITEM_C (2)
```

## 🎯 期待される出力

| ITEM_ID | ITEM_NAME |
|---------|-----------|
| 0       | ITEM_A    |
| 3       | ITEM_D    |

**説明**：
- ITEM_ID=0（ITEM_A）: PARENT_ITEM_ID=NULL → ROOT
- ITEM_ID=3（ITEM_D）: PARENT_ITEM_ID=NULL → ROOT

## 💡 解答

### 最適化されたSQL

```sql
SELECT
    i.ITEM_ID,
    i.ITEM_NAME
FROM ITEM_INFO i
INNER JOIN ITEM_TREE t ON i.ITEM_ID = t.ITEM_ID
WHERE t.PARENT_ITEM_ID IS NULL
ORDER BY i.ITEM_ID;
```

## 📝 解説

### 解法のポイント

1. **テーブル結合（INNER JOIN）**
   ```sql
   FROM ITEM_INFO i
   INNER JOIN ITEM_TREE t ON i.ITEM_ID = t.ITEM_ID
   ```
   - ITEM_INFOからアイテム名を取得
   - ITEM_TREEから親子関係を取得
   - ITEM_IDで結合

2. **ROOT条件の指定**
   ```sql
   WHERE t.PARENT_ITEM_ID IS NULL
   ```
   - PARENT_ITEM_IDがNULLのアイテムがROOT
   - `IS NULL`演算子でNULL値を検索

3. **テーブルエイリアス**
   - `i`：ITEM_INFO
   - `t`：ITEM_TREE
   - 可読性向上とクエリの簡潔化

4. **ソート**
   ```sql
   ORDER BY i.ITEM_ID
   ```
   - ITEM_ID昇順で結果を整列

### パフォーマンスチューニング

#### インデックス戦略

**推奨インデックス**：
```sql
-- ITEM_TREEのPARENT_ITEM_IDにインデックス
CREATE INDEX idx_item_tree_parent ON ITEM_TREE(PARENT_ITEM_ID);

-- ITEM_TREEのITEM_IDにインデックス（PKの場合は不要）
CREATE INDEX idx_item_tree_id ON ITEM_TREE(ITEM_ID);

-- ITEM_INFOのITEM_IDにインデックス（PKの場合は不要）
CREATE INDEX idx_item_info_id ON ITEM_INFO(ITEM_ID);
```

**理由**：
- PARENT_ITEM_IDインデックスでWHERE句を高速化
- JOIN条件のITEM_IDインデックスで結合を最適化
- ORDER BYのITEM_IDインデックスでソートを効率化

**注意**：NULLインデックスの扱い
- MySQLでは、NULLもインデックスに含まれる
- `IS NULL`検索でもインデックスが効く

#### 複合インデックスの検討

```sql
CREATE INDEX idx_item_tree_composite ON ITEM_TREE(PARENT_ITEM_ID, ITEM_ID);
```

**効果**：
- WHERE句とJOIN条件の両方をカバー
- カバリングインデックスとして機能

#### 実行計画の確認

```sql
EXPLAIN SELECT
    i.ITEM_ID,
    i.ITEM_NAME
FROM ITEM_INFO i
INNER JOIN ITEM_TREE t ON i.ITEM_ID = t.ITEM_ID
WHERE t.PARENT_ITEM_ID IS NULL
ORDER BY i.ITEM_ID;
```

**チェックポイント**：
- `type`: `ref` または `eq_ref`（インデックス使用）
- `key`: インデックスが使用されていることを確認
- `Extra`: `Using index` が理想（カバリングインデックス）

### 代替アプローチ

#### オプション1：INNER JOIN（推奨）

```sql
SELECT
    i.ITEM_ID,
    i.ITEM_NAME
FROM ITEM_INFO i
INNER JOIN ITEM_TREE t ON i.ITEM_ID = t.ITEM_ID
WHERE t.PARENT_ITEM_ID IS NULL
ORDER BY i.ITEM_ID;
```

**評価**：
- 最も一般的で可読性が高い
- オプティマイザが最適化しやすい
- **推奨**

#### オプション2：サブクエリ（IN）

```sql
SELECT
    ITEM_ID,
    ITEM_NAME
FROM ITEM_INFO
WHERE ITEM_ID IN (
    SELECT ITEM_ID
    FROM ITEM_TREE
    WHERE PARENT_ITEM_ID IS NULL
)
ORDER BY ITEM_ID;
```

**評価**：
- サブクエリで先にROOTアイテムを抽出
- INリストが小さい場合は効率的
- JOINより若干パフォーマンスが劣る可能性

#### オプション3：サブクエリ（EXISTS）

```sql
SELECT
    i.ITEM_ID,
    i.ITEM_NAME
FROM ITEM_INFO i
WHERE EXISTS (
    SELECT 1
    FROM ITEM_TREE t
    WHERE t.ITEM_ID = i.ITEM_ID
    AND t.PARENT_ITEM_ID IS NULL
)
ORDER BY i.ITEM_ID;
```

**評価**：
- 相関サブクエリを使用
- 各ITEM_INFOの行ごとにサブクエリ実行
- 大量データでは非効率
- 推奨しない

#### オプション4：LEFT JOIN

```sql
SELECT
    i.ITEM_ID,
    i.ITEM_NAME
FROM ITEM_INFO i
LEFT JOIN ITEM_TREE t ON i.ITEM_ID = t.ITEM_ID
WHERE t.PARENT_ITEM_ID IS NULL
ORDER BY i.ITEM_ID;
```

**評価**：
- LEFT JOINでも結果は同じ
- すべてのアイテムがITEM_TREEに存在する前提
- INNER JOINの方が意図が明確
- 推奨しない（INNER JOINを使うべき）

### IS NULL vs = NULL

**正しい**：
```sql
WHERE PARENT_ITEM_ID IS NULL
```

**間違い**：
```sql
WHERE PARENT_ITEM_ID = NULL  -- 常にFALSE
```

**理由**：
- NULLは「不明な値」なので等価比較できない
- `= NULL`は常にFALSEを返す
- 必ず`IS NULL`を使用

## 🔑 キーワード

- `INNER JOIN`：テーブル結合
- `IS NULL`：NULL値の検索
- **階層データ**：親子関係のツリー構造
- **ROOTノード**：親を持たない最上位ノード
- **テーブルエイリアス**：可読性とクエリ簡潔化
- **NULLインデックス**：NULLもインデックスに含まれる
- **相関サブクエリ**：外部クエリの各行に対して実行

## 🎓 学習ポイント

1. INNER JOINで2つのテーブルを結合できる
2. IS NULLでNULL値を正しく検索できる
3. = NULLは使えない（常にFALSE）
4. 階層データ（ツリー構造）の基本概念
5. ROOTノード＝親がないノード
6. テーブルエイリアスで可読性を向上
7. PARENT_ITEM_IDインデックスでIS NULL検索も高速化

## 💡 応用例

### 全階層の深さを計算

```sql
WITH RECURSIVE item_depth AS (
    -- ROOT アイテム（深さ0）
    SELECT
        ITEM_ID,
        PARENT_ITEM_ID,
        0 AS depth
    FROM ITEM_TREE
    WHERE PARENT_ITEM_ID IS NULL

    UNION ALL

    -- 子アイテム（深さ+1）
    SELECT
        t.ITEM_ID,
        t.PARENT_ITEM_ID,
        d.depth + 1
    FROM ITEM_TREE t
    INNER JOIN item_depth d ON t.PARENT_ITEM_ID = d.ITEM_ID
)
SELECT
    i.ITEM_ID,
    i.ITEM_NAME,
    d.depth
FROM ITEM_INFO i
INNER JOIN item_depth d ON i.ITEM_ID = d.ITEM_ID
ORDER BY d.depth, i.ITEM_ID;
```

### 子アイテムの数をカウント

```sql
SELECT
    i.ITEM_ID,
    i.ITEM_NAME,
    COUNT(t.ITEM_ID) AS child_count
FROM ITEM_INFO i
LEFT JOIN ITEM_TREE t ON i.ITEM_ID = t.PARENT_ITEM_ID
WHERE i.ITEM_ID IN (
    SELECT ITEM_ID
    FROM ITEM_TREE
    WHERE PARENT_ITEM_ID IS NULL
)
GROUP BY i.ITEM_ID, i.ITEM_NAME
ORDER BY i.ITEM_ID;
```

### 葉ノード（子がないアイテム）を検索

```sql
SELECT
    i.ITEM_ID,
    i.ITEM_NAME
FROM ITEM_INFO i
INNER JOIN ITEM_TREE t ON i.ITEM_ID = t.ITEM_ID
WHERE NOT EXISTS (
    SELECT 1
    FROM ITEM_TREE child
    WHERE child.PARENT_ITEM_ID = t.ITEM_ID
)
ORDER BY i.ITEM_ID;
```

### ROOTアイテムとその直接の子を表示

```sql
SELECT
    root.ITEM_ID AS ROOT_ID,
    root.ITEM_NAME AS ROOT_NAME,
    child.ITEM_ID AS CHILD_ID,
    child.ITEM_NAME AS CHILD_NAME
FROM ITEM_INFO root
INNER JOIN ITEM_TREE root_tree ON root.ITEM_ID = root_tree.ITEM_ID
LEFT JOIN ITEM_TREE child_tree ON root.ITEM_ID = child_tree.PARENT_ITEM_ID
LEFT JOIN ITEM_INFO child ON child_tree.ITEM_ID = child.ITEM_ID
WHERE root_tree.PARENT_ITEM_ID IS NULL
ORDER BY root.ITEM_ID, child.ITEM_ID;
```

### 階層パスを表示（再帰CTE）

```sql
WITH RECURSIVE item_path AS (
    -- ROOT アイテム
    SELECT
        ITEM_ID,
        PARENT_ITEM_ID,
        CAST(ITEM_ID AS CHAR(100)) AS path
    FROM ITEM_TREE
    WHERE PARENT_ITEM_ID IS NULL

    UNION ALL

    -- 子アイテム（パスに追加）
    SELECT
        t.ITEM_ID,
        t.PARENT_ITEM_ID,
        CONCAT(p.path, ' -> ', t.ITEM_ID)
    FROM ITEM_TREE t
    INNER JOIN item_path p ON t.PARENT_ITEM_ID = p.ITEM_ID
)
SELECT
    i.ITEM_ID,
    i.ITEM_NAME,
    p.path
FROM ITEM_INFO i
INNER JOIN item_path p ON i.ITEM_ID = p.ITEM_ID
ORDER BY i.ITEM_ID;
```

## 📚 関連概念

### ツリー構造の用語

- **ROOT（ルート）**：親を持たない最上位ノード
- **LEAF（葉）**：子を持たない最下層ノード
- **PARENT（親）**：直接の上位ノード
- **CHILD（子）**：直接の下位ノード
- **ANCESTOR（祖先）**：上位のすべてのノード
- **DESCENDANT（子孫）**：下位のすべてのノード
- **SIBLING（兄弟）**：同じ親を持つノード
- **DEPTH（深さ）**：ROOTからの階層レベル

### 階層データの実装パターン

1. **隣接リストモデル（Adjacency List）**
   - 本問題で使用
   - PARENT_ITEM_IDを持つシンプルな構造
   - 再帰クエリが必要

2. **パスエンコーディング**
   - パス全体を文字列で保存（例：`/0/1/2/`）
   - 階層検索が高速

3. **ネステッドセットモデル**
   - 左右の値で階層を表現
   - サブツリー検索が高速

---

**作成日**: 2025-10-08
**難易度**: ⭐ 基本
**カテゴリ**: INNER JOIN, IS NULL, 階層データ, ツリー構造
