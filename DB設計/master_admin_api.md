## 一覧

| パラメータ名 | 型 | 必須 | デフォルト値 | 説明 |
|------------|---|-----|------------|-----|
| `page` | integer | No | 1 | ページ番号（1以上） |
| `per_page` | integer | No | 50 | 1ページ当たりの件数（1-100） |
| `keyword` | string | No | "" | フリーワード検索（スペース区切りでAND検索） |
| `company_id` | integer | No | null | 企業ID（空なら全件、IDがあれば該当企業のみ） |
| `order_by` | string | No | DESC | ソート順（ASC: 昇順、DESC: 降順） |

### HTTPメソッド
```
HTTP: GET 
```

### リクエスト
```json
{
    // MEMO
}
```

### レスポンス
```json
{
    "meta": {
        "page": 1,
        "from": 1,
        "to": 2,
        "per_page": 50,
        "total_count": 32,
        "last_page": 1,
        "active_count": 18,
        "inactive_count": 14
    },
    "links": {
        "first": "https://api.example.com/api/admin/catalog-requests?page=1",
        "last": "https://api.example.com/api/admin/catalog-requests?page=3",
        "prev": null,
        "next": "https://api.example.com/api/admin/catalog-requests?page=2"
    },
    "data": [
        {}
    ]
}
```

```json
// エラー時
{
    "success": false,
    "message": "",
    "errors": {}
}
```

---

## 詳細

### HTTPメソッド
```
HTTP: GET 
```

### リクエストパラメータ

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|-----|-----|
| `id` | integer | Yes | ID |

### レスポンス
```json
{}
```

```json
// エラー時
{
    "success": false,
    "message": "",
    "errors": {}
}
```

---

## 編集（更新）

### HTTPメソッド

```json
HTTP: PUT
```

### リクエスト
```json
{}
```

### レスポンス
```json
// 正常
HTTP:200
{
    "success": true,
    "message": "",
    "data": {
        "id": 1000,
    }
}
```

```json
// エラー時
{
    "success": false,
    "message": "",
    "errors": {}
}
```

---

## 登録

### HTTPメソッド

```json
HTTP: POST
```

### リクエスト
```json
{

}
```

### レスポンス
```json
// 正常
HTTP:201
{
    "success": true,
    "message": "",
    "data": {
        "id": 1000,
    }
}
```

```json
// エラー時
{
    "success": false,
    "message": "",
    "errors": {}
}
```

---

## 削除

### HTTPメソッド

```json
HTTP: DELETE
```

```json
{}
```

### レスポンス
```json
// 正常
HTTP:200
{
    "success": true,
    "message": "",
    "data": {
        "id": 1000,
    }
}
```

```json
// エラー時
{
    "success": false,
    "message": "",
    "errors": {}
}
```