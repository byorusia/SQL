# SQL基礎概念

## 📚 目次
1. [SELECT文の基礎](#select文の基礎)
2. [WHERE句とフィルタリング](#where句とフィルタリング)
3. [JOIN操作](#join操作)
4. [集計関数](#集計関数)
5. [サブクエリ](#サブクエリ)
6. [ウィンドウ関数](#ウィンドウ関数)

---

## SELECT文の基礎

### 基本構文
```sql
SELECT カラム1, カラム2
FROM テーブル名
WHERE 条件
ORDER BY カラム1 [ASC|DESC];
```

### よく使う句
- **SELECT**: 取得するカラムを指定
- **FROM**: データを取得するテーブルを指定
- **WHERE**: フィルタ条件を指定
- **ORDER BY**: 結果の並び順を指定
- **LIMIT**: 取得する行数を制限

### 例
```sql
-- 全てのカラムを取得
SELECT * FROM employees;

-- 特定のカラムのみ取得
SELECT first_name, last_name FROM employees;

-- 重複を除外
SELECT DISTINCT department_id FROM employees;
```

---

## WHERE句とフィルタリング

### 比較演算子
- `=` : 等しい
- `<>` または `!=` : 等しくない
- `>`, `<`, `>=`, `<=` : 大小比較

### 論理演算子
- `AND` : かつ
- `OR` : または
- `NOT` : 否定

### その他の演算子
- `BETWEEN A AND B` : A以上B以下
- `IN (値1, 値2, ...)` : リストのいずれかに一致
- `LIKE` : パターンマッチング
- `IS NULL` / `IS NOT NULL` : NULL判定

### 例
```sql
-- 給与が400万円以上の従業員
SELECT * FROM employees
WHERE salary >= 4000000;

-- 営業部または開発部の従業員
SELECT * FROM employees
WHERE department_id IN (1, 2);

-- 名前が「山」で始まる従業員
SELECT * FROM employees
WHERE last_name LIKE '山%';
```

---

## JOIN操作

### JOIN の種類

#### INNER JOIN（内部結合）
両方のテーブルに一致するレコードのみを取得

```sql
SELECT e.first_name, d.department_name
FROM employees e
INNER JOIN departments d ON e.department_id = d.department_id;
```

#### LEFT JOIN（左外部結合）
左テーブルの全レコード + 右テーブルの一致するレコード

```sql
SELECT e.first_name, d.department_name
FROM employees e
LEFT JOIN departments d ON e.department_id = d.department_id;
```

#### RIGHT JOIN（右外部結合）
右テーブルの全レコード + 左テーブルの一致するレコード

#### FULL OUTER JOIN（完全外部結合）
両方のテーブルの全レコード

### 自己結合（SELF JOIN）
同じテーブルを結合

```sql
-- 従業員とその上司を取得
SELECT e.first_name AS 従業員, m.first_name AS 上司
FROM employees e
LEFT JOIN employees m ON e.manager_id = m.employee_id;
```

---

## 集計関数

### 基本的な集計関数
- `COUNT()`: 行数をカウント
- `SUM()`: 合計
- `AVG()`: 平均
- `MAX()`: 最大値
- `MIN()`: 最小値

### GROUP BY と HAVING

```sql
-- 部署ごとの平均給与
SELECT department_id, AVG(salary) as 平均給与
FROM employees
GROUP BY department_id;

-- 平均給与が450万円以上の部署のみ
SELECT department_id, AVG(salary) as 平均給与
FROM employees
GROUP BY department_id
HAVING AVG(salary) >= 4500000;
```

### WHERE vs HAVING
- **WHERE**: 集計前のフィルタリング
- **HAVING**: 集計後のフィルタリング

---

## サブクエリ

### WHERE句のサブクエリ
```sql
-- 平均給与以上の従業員
SELECT first_name, salary
FROM employees
WHERE salary >= (SELECT AVG(salary) FROM employees);
```

### FROM句のサブクエリ
```sql
-- 部署ごとの平均給与を持つ仮想テーブル
SELECT d.department_name, avg_sal.平均給与
FROM departments d
JOIN (
    SELECT department_id, AVG(salary) as 平均給与
    FROM employees
    GROUP BY department_id
) avg_sal ON d.department_id = avg_sal.department_id;
```

### 相関サブクエリ
外側のクエリの値を使用するサブクエリ

```sql
-- 各部署で最高給与の従業員
SELECT first_name, department_id, salary
FROM employees e1
WHERE salary = (
    SELECT MAX(salary)
    FROM employees e2
    WHERE e1.department_id = e2.department_id
);
```

---

## ウィンドウ関数

### 基本構文
```sql
関数名() OVER (
    [PARTITION BY カラム]
    [ORDER BY カラム]
    [ROWS/RANGE 句]
)
```

### よく使うウィンドウ関数

#### ROW_NUMBER()
各行に連番を付与

```sql
SELECT first_name, salary,
       ROW_NUMBER() OVER (ORDER BY salary DESC) as 順位
FROM employees;
```

#### RANK() / DENSE_RANK()
ランキングを付与

```sql
-- RANK: 同順位の場合、次の順位が飛ぶ (1, 2, 2, 4)
-- DENSE_RANK: 飛ばない (1, 2, 2, 3)
SELECT first_name, salary,
       RANK() OVER (ORDER BY salary DESC) as 順位,
       DENSE_RANK() OVER (ORDER BY salary DESC) as 密順位
FROM employees;
```

#### LAG() / LEAD()
前後の行の値を取得

```sql
-- 前月比を計算
SELECT sale_date, revenue,
       LAG(revenue) OVER (ORDER BY sale_date) as 前月売上,
       revenue - LAG(revenue) OVER (ORDER BY sale_date) as 増減
FROM sales;
```

#### 集計関数 + OVER句
累積合計や移動平均

```sql
-- 累積売上
SELECT sale_date, revenue,
       SUM(revenue) OVER (ORDER BY sale_date) as 累積売上
FROM sales;
```

---

## 💡 学習のコツ

1. **実際に手を動かす**: 理論だけでなく、実際にSQLを書いて実行する
2. **エラーを恐れない**: エラーメッセージから学ぶことが多い
3. **小さく始める**: 複雑なクエリは段階的に構築する
4. **実行計画を確認**: `EXPLAIN`を使ってクエリの動作を理解する
5. **サンプルデータを活用**: `schemas/common_schemas.sql`を使って練習

---

**最終更新**: 2025-10-06
