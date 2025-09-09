# イベント管理API仕様書

## 目次

1. [イベント一覧API](#イベント一覧api)
2. [イベント詳細API](#イベント詳細api)
3. [イベント登録API](#イベント登録api)
4. [イベント更新API](#イベント更新api)

---

## イベント一覧API

```
HTTP: GET
```

### 概要
登録されているイベントの一覧を取得します。フィルタリング、ページネーション、ソート機能をサポートしています。

### リクエストパラメータ

| パラメータ名 | 型 | 必須 | デフォルト値 | 説明 |
|------------|---|-----|------------|-----|
| `page` | integer | No | 1 | ページ番号（1以上） |
| `per_page` | integer | No | 50 | 1ページ当たりの件数（1-100） |
| `keyword` | string | No | "" | フリーワード検索（スペース区切りでAND検索） |
| `company_id` | integer | No | "" | 企業ID（空なら全件、IDがあれば該当企業のみ） |
| `status` | integer | No | "" | 公開ステータス（0: 非公開、1: 公開、空: 全件） |
| `order_by` | string | No | "DESC" | ソート順（ASC: 昇順、DESC: 降順） |

### リクエスト例
```
HTTP: GET 
```

### レスポンス

#### 成功時（200 OK）
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
        {
            "id": 1000,
            "title": "まちなかジーヴォ緑区桃山見学会",
            "description": "新築戸建て住宅の見学会です。最新の設備や間取りをご確認いただけます。",
            "venue": "ジーヴォ緑区桃山展示場\n名鉄犬山線「岩倉」駅からバス乗車。「三ツ渕東」バス停より徒歩5分",
            "start_date": "2025-01-15",
            "end_date": "2025-01-16",
            "start_time": "10:00:00",
            "end_time": "18:00:00",
            "is_active": 1,
            "postal_code": "458-0021",
            "prefecture_id": 23,
            "prefecture_name": "愛知県",
            "address01": "名古屋市緑区桃山3丁目",
            "address02": "1-15",
            "time_required": "約1時間30分",
            "about_reservations": "事前予約が必要です。当日受付も可能ですが、予約優先となります。",
            "reservation_deadline": "2025-01-14",
            "participation_fee": "無料",
            "access_info": "名古屋市営地下鉄桜通線「神沢」駅より徒歩4分",
            "tmp_save": 0,
            "created_at": "2024-12-01T10:30:00Z",
            "updated_at": "2024-12-15T14:20:00Z",
            "reservations_count": 12,
            "created_at": "2025-09-08T05:38:34.000000Z",
            "updated_at": "2025-09-08T05:38:34.000000Z",
            "company": {
                "id": 10001,
                "name": "株式会社モダン",
                "contact_person": "田中太郎",
                "contact_phone": "052-123-4567",
                "url": "https://example.com",
                "created_at": "2025-09-08T05:38:34.000000Z",
                "updated_at": "2025-09-08T05:38:34.000000Z",
            },
            "features": [
                {
                    "id": 99,
                    "name": "特典あり",
                    "description": "来場特典をご用意しております",
                    "is_active": 1,
                    "sort_no": 1
                },
                {
                    "id": 100,
                    "name": "無料参加",
                    "description": "参加費は無料です",
                    "is_active": 1,
                    "sort_no": 2
                },
                {
                    "id": 101,
                    "name": "個別対応可",
                    "description": "個別でのご案内も可能です",
                    "is_active": 1,
                    "sort_no": 3
                }
            ]
        }
    ]
}
```

#### エラー時（400 Bad Request）
```json
{
    "success": false,
    "message": "エラーが検出されました",
    "errors": {}
}
```

---

## イベント詳細API

### HTTP
```
HTTP: GET
```

### 概要
指定されたIDのイベント詳細情報を取得します。

### パスパラメータ

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|-----|-----|
| `id` | integer | Yes | イベントID |

### レスポンス

#### 成功時（200 OK）
```json
{
    "id": 1000,
    "title": "まちなかジーヴォ緑区桃山見学会",
    "description": "新築戸建て住宅の見学会です。最新の設備や間取りをご確認いただけます。",
    "venue": "ジーヴォ緑区桃山展示場\n名鉄犬山線「岩倉」駅からバス乗車。「三ツ渕東」バス停より徒歩5分",
    "start_date": "2025-01-15",
    "end_date": "2025-01-16",
    "start_time": "10:00:00",
    "end_time": "18:00:00",
    "is_active": 1,
    "postal_code": "458-0021",
    "prefecture_id": 23,
    "prefecture_name": "愛知県",
    "address01": "名古屋市緑区桃山3丁目",
    "address02": "1-15",
    "time_required": "約1時間30分",
    "about_reservations": "事前予約が必要です。当日受付も可能ですが、予約優先となります。",
    "reservation_deadline": "2025-01-14",
    "participation_fee": "無料",
    "tmp_save": 0,
    "created_at": "2024-12-01T10:30:00Z",
    "updated_at": "2024-12-15T14:20:00Z",
    "reservations_count": 12,
    "created_at": "2025-09-08T05:38:34.000000Z",
    "updated_at": "2025-09-08T05:38:34.000000Z",
    "company": {
        "id": 10001,
        "name": "株式会社モダン",
        "contact_person": "田中太郎",
        "contact_phone": "052-123-4567",
        "business_hours": "9:00-18:00",
        "url": "https://example.com",
        "created_at": "2025-09-08T05:38:34.000000Z",
        "updated_at": "2025-09-08T05:38:34.000000Z"
    },
    "genres": [
        {
            "id": 1,
            "name": "見学会",
            "description": "住宅見学会"
        }
    ],
    "features": [
        {
            "id": 99,
            "name": "特典あり",
            "description": "来場特典をご用意しております",
            "is_active": 1,
        }
    ],
    "contexts": [
        {
            "id": 201,
            "title": "住宅見学",
            "description": "最新の住宅設備をご覧いただけます",
        }
    ],
    "flows": [
        {
            "id": 301,
            "title": "受付・案内",
            "description": "受付にて資料をお渡しし、見学ルートをご案内いたします",
            "image_path": "/images/flows/reception.jpg"
        }
    ]
}
```

#### エラー時（404 Not Found）
```json
{
    "success": false,
    "message": "エラーが検出されました",
    "errors": {}
}
```

---

## イベント登録API

### HTTP
```
HTTP: POST
```

### 概要
新しいイベントを登録します。

### リクエストボディ
```json
{
    [〇]"company_id": 10001,
    [〇]"title": "住宅ローンセミナー",
    []"description": "住宅購入時のローンについて詳しく説明いたします",
    []"venue": "ジーヴォ緑区桃山展示場\n名鉄犬山線「岩倉」駅からバス乗車。「三ツ渕東」バス停より徒歩5分", //改行コードを含めてリクエストするので、DBへは文字列のまま登録してください
    []"start_date": "2025-02-01",
    []"end_date": "2025-02-01",
    []"start_time": "14:00:00",
    []"end_time": "16:00:00",
    [〇]"is_active": 1,
    []"postal_code": "450-0002",
    []"prefecture_id": 23,
    []"address01": "名古屋市中村区名駅3-15-1",
    []"address02": "名古屋ビル5F",
    []"time_required": "約2時間",
    []"about_reservations": "事前予約必須",
    []"reservation_deadline": "2025-01-30",
    []"participation_fee": "無料",
    [〇]"tmp_save": 0,
    []"genre_ids": [2],
    []"feature_ids": [100, 101],
    "contexts": [
        {
            [〇]"title": "住宅ローン基礎知識",
            []"description": "住宅ローンの基本的な仕組みについて説明します"
        }
    ],
    "flows": [
        {
            [〇]"title": "受付",
            []"description": "受付でお名前を確認いたします",
            []"sort_order": 1,
            []"image_path": "/images/flows/reception.jpg"
        },
        {
            [〇]"title": "セミナー開始",
            []"description": "住宅ローンについて詳しく説明いたします",
            []"sort_order": 2,
            []"image_path": ""
        }
    ]
}
```

### レスポンス

#### 成功時（201 Created）
```json
{
    "success": true,
    "message": "イベントを登録しました",
    "data": {
        "id": 1001,
    }
}
```

```json
{
    "success": false,
    "message": "エラーが検出されました",
    "errors": {}
}
```

---

## イベント更新API

### HTTP
```
HTTP: PUT
```

### 概要
指定されたIDのイベント情報を更新します。

### パスパラメータ

| パラメータ名 | 型 | 必須 | 説明 |
|------------|---|-----|-----|
| `id` | integer | Yes | イベントID |

### リクエストボディ
```json
{
    [〇]"id": 10001,
    [〇]"title": "住宅ローンセミナー（更新版）",
    []"description": "住宅購入時のローンについて詳しく説明いたします（内容更新）",
    []"venue": "ジーヴォ緑区桃山展示場\n名鉄犬山線「岩倉」駅からバス乗車。「三ツ渕東」バス停より徒歩5分",
    []"start_date": "2025-02-01",
    []"end_date": "2025-02-01",
    []"start_time": "14:00:00",
    []"end_time": "16:00:00",
    [〇]"is_active": 1,
    []"postal_code": "450-0002",
    []"prefecture_id": 23,
    []"address01": "名古屋市中村区名駅3-15-1",
    []"address02": "名古屋ビル5F",
    []"time_required": "約2時間",
    []"about_reservations": "事前予約必須",
    []"reservation_deadline": "2025-01-30",
    []"participation_fee": "無料",
    [〇]"tmp_save": 0,
    []"genre_ids": [2],
    []"feature_ids": [100, 101],
    "contexts": [
        {
            []"id": 201,
            [〇]"title": "住宅ローン基礎知識（更新）",
            []"description": "住宅ローンの基本的な仕組みについて詳しく説明します"
        },
        {
            []"id": "",
            [〇]"title": "住宅ローン基礎知識（更新）",
            []"description": "住宅ローンの基本的な仕組みについて詳しく説明します"
        }
    ],
    "flows": [
        {
            []"id": 301,
            [〇]"title": "受付",
            []"description": "受付でお名前を確認いたします",
            []"sort_order": 1,
            []"image_path": "/images/flows/reception.jpg"
        },
        {
            []"id": "",
            [〇]"title": "セミナー開始",
            []"description": "住宅ローンについて詳しく説明いたします",
            []"sort_order": 2,
            []"image_path": ""
        }
    ]
}
```

### レスポンス

#### 成功時（200 OK）
```json
{
    "success": true,
    "message": "イベント情報を更新しました",
    "data": {
        "id": 1001,
        "updated_at": "2024-12-20T15:30:00Z"
    }
}
```

```json
{
    "success": false,
    "message": "エラーが検出されました",
    "errors": {}
}
```