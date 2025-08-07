<style>section h1 { color: #069; }</style>

[Home](/) > [Reading notes](/reading_notes/)

著：（株）ピネアル CTO 藤田拳

マーケ領域で実践されている生成系AIの技術
===

## Vision モデルによる Image 分析 - LLM によるサムネイル画像分析

### 商品詳細画像分析ワークフローのフローチャート

```mermaid
商品カテゴリを指定 --> ランキング上位の商品を検索 --> 商品ページの画像をダウンロード --> 画像中のテキストを文字起こし --> 内部からカテゴリ分類 --> カテゴリ分類の順位を分析
```

### ざっくりとした手順

* User が Spreadsheet にクエリを入力
    * Spreadsheet から サイトのAPI を呼び出し
        * その取得情報を LLM に送信
            * 返ってきた結果を分類

### 商品情報を抽出するためのプロンプト（例）

```XML
<Prompt>
    <Extract>
        <Products>
            <Product>
                <ProductName>Item.itemName</ProductName>
                <Description>Item.itemDescription</Description>
                <Price>Item.itemPrice</Price>
            </Product>
        </Products>
    </Extract>
    <Output format="json">
        {
            "Products": [
                {
                    "ProductName": "{ProductName}",
                    "Description": "{Description}",
                    "Price": "{Price}",
                }
            ]
        }
    </Output>
</Prompt>    
```

### 商品情報の画像を文字起こしして、カテゴリを分類するためのプロンプト（例）

```
<task>ユーザーの入力した画像をOCRで文字起こしし、指定されたカテゴリに分類してください</task>
<rule>
    カテゴリは以下のいずれかです:
    ${categoriesList}

    出力は以下の JSON 形式でお願いします:
    {
        "transcription": "抽出されたテキスト",
        "category": "選択されたカテゴリ",
    }
</rule>
```

### ポイント

* JSON や XML は システムと生成AI系に対して相性が良い
* API のハブとしてミドルウェアを間に置いてうまく使う
