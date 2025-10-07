# SUV 平均レンタル料金

## 📖 問題文（日本語訳）

以下はある自動車レンタル会社でレンタル中の自動車情報を含む CAR_RENTAL_COMPANY_CAR テーブルです。

CAR_RENTAL_COMPANY_CAR テーブルから自動車種類が 'SUV' の自動車の平均日レンタル料金を出力する SQL 文を作成してください。

この時、平均日レンタル料金は小数点第 1 位で四捨五入し、カラム名は AVERAGE_FEE として指定してください。

## 📊 テーブル構造

### CAR_RENTAL_COMPANY_CAR テーブル

CAR_ID、CAR_TYPE、DAILY_FEE、OPTIONS は、それぞれ自動車 ID、自動車種類、日レンタル料金（ウォン）、自動車オプションリストを表します。

自動車種類は「セダン」、「SUV」、「バン」、「トラック」、「リムジン」があります。

自動車オプションリストはカンマ（','）で区切られたキーワードリスト（例：「シートヒーター」、「スマートキー」、「駐車センサー」）で構成されており、キーワード種類は「駐車センサー」、「スマートキー」、「ナビゲーション」、「ベンチレーションシート」、「シートヒーター」、「バックカメラ」、「レザーシート」があります。

| Column name | Type         | Nullable |
| ----------- | ------------ | -------- |
| CAR_ID      | INTEGER      | FALSE    |
| CAR_TYPE    | VARCHAR(255) | FALSE    |
| DAILY_FEE   | INTEGER      | FALSE    |
| OPTIONS     | VARCHAR(255) | FALSE    |

**サンプルデータ:**

| CAR_ID | CAR_TYPE | DAILY_FEE | OPTIONS                                   |
| ------ | -------- | --------- | ----------------------------------------- |
| 1      | セダン   | 16000     | レザーシート,シートヒーター,バックカメラ  |
| 2      | SUV      | 14000     | スマートキー,ナビゲーション,シートヒーター |
| 3      | SUV      | 22000     | 駐車センサー,バックカメラ,レザーシート    |

## 🎯 期待される出力

| AVERAGE_FEE |
| ----------- |
| 18000       |

## 💡 解答

```sql
SELECT
    ROUND(AVG(DAILY_FEE), 0) AS AVERAGE_FEE
FROM CAR_RENTAL_COMPANY_CAR
WHERE CAR_TYPE = 'SUV';
```

## 📝 解説

### アプローチ

1. CAR_RENTAL_COMPANY_CAR テーブルから CAR_TYPE が 'SUV' の車両をフィルタリング
2. AVG 関数で DAILY_FEE の平均を計算
3. ROUND 関数で小数点第 1 位で四捨五入（整数化）
4. カラム名を AVERAGE_FEE として出力

### 解答の流れ

**SUV 車両のレンタル料金：**

- CAR_ID = 2: 14,000 ウォン
- CAR_ID = 3: 22,000 ウォン

**平均計算：**

(14,000 + 22,000) ÷ 2 = 18,000 ウォン

**四捨五入：**

18,000.0 → ROUND(18000.0, 0) → 18,000

### 学習ポイント

- **AVG 関数**: 指定したカラムの平均値を計算
- **ROUND 関数**: 数値を指定した桁数で四捨五入
- **集計関数**: GROUP BY なしでテーブル全体を 1 つのグループとして集計
- **WHERE + 集計関数**: フィルタリング後に集計を実行

### ROUND 関数の使い方

```sql
ROUND(数値, 桁数)
```

**桁数の指定：**

- `0`: 整数（小数点第 1 位で四捨五入）
- `1`: 小数点第 1 位まで（小数点第 2 位で四捨五入）
- `-1`: 10 の位まで（1 の位で四捨五入）

**例：**

```sql
ROUND(18000.7, 0)  -- 結果: 18001
ROUND(18000.4, 0)  -- 結果: 18000
ROUND(18456, -1)   -- 結果: 18460
ROUND(18456, -2)   -- 結果: 18500
```

### 集計関数の実行順序

```sql
WHERE → 集計関数（AVG、SUM、COUNT など）→ SELECT
```

1. WHERE 句でデータをフィルタリング
2. フィルタリング後のデータに対して AVG を計算
3. ROUND で結果を加工
4. AVERAGE_FEE として出力

### AVG 関数の基本

```sql
-- 全体の平均
SELECT AVG(DAILY_FEE) FROM CAR_RENTAL_COMPANY_CAR;

-- 条件付き平均
SELECT AVG(DAILY_FEE) FROM CAR_RENTAL_COMPANY_CAR WHERE CAR_TYPE = 'SUV';

-- 四捨五入した平均
SELECT ROUND(AVG(DAILY_FEE), 0) FROM CAR_RENTAL_COMPANY_CAR WHERE CAR_TYPE = 'SUV';
```

## 🔑 キーワード

- SELECT（集計関数）
- WHERE 句
- AVG 関数
- ROUND 関数
- カラムエイリアス（AS）
