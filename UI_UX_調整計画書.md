# 注文住宅ポータル管理画面 UI/UX調整計画書

## 1. 概要
DB設計「注文住宅_リファクタリング版.md」に基づいて、管理画面のUI/UXを調整します。
特にカタログ請求管理、イベント予約管理、見学予約管理の3つの機能について、DBのカラムとの整合性を確保し、抜け漏れがないように実装します。

## 2. 調整対象画面と改修内容

### 2.1 カタログ請求管理（注文住宅）

#### 対象ファイル
- `catalog_custom_home_list.html` - カタログ一覧画面
- `catalog_custom_home_register.html` - カタログ登録・編集画面
- `catalog_request.html` - カタログ請求管理画面

#### DB対応テーブル
- `cbh_catalogs` - カタログマスタ
- `cbh_catalog_requests` - カタログ請求データ

#### 必要な調整項目

##### カタログマスタ（cbh_catalogs）
| カラム名 | 現在のUI | 必要な調整 |
|---------|----------|-----------|
| code | なし | カタログコード入力欄を追加 |
| name | あり | - |
| description | あり | - |
| image_path | あり | - |
| service_type_id | なし | サービス種別選択を追加 |
| download_url | なし | PDFダウンロードURL入力欄を追加 |
| page_count | なし | ページ数入力欄を追加 |
| file_size | なし | ファイルサイズ表示を追加 |
| is_active | あり | - |

##### カタログ請求データ（cbh_catalog_requests）
| カラム名 | 現在のUI | 必要な調整 |
|---------|----------|-----------|
| catalog_id | あり | - |
| customer_id | あり | - |
| preferred_contact_time | なし | 希望連絡時間帯を追加 |
| is_house_building_hope | なし | 住宅建築希望チェックボックスを追加 |
| house_building_hope_prefecture_id | なし | 建築希望都道府県選択を追加 |
| house_building_hope_city | なし | 建築希望市区町村入力を追加 |
| house_building_hope_address_line | なし | 建築希望住所詳細入力を追加 |
| is_land_available | なし | 土地所有チェックボックスを追加 |
| is_newsletter_subscribed | なし | メルマガ購読チェックボックスを追加 |

### 2.2 イベント予約管理

#### 対象ファイル
- `all_events.html` - イベント一覧画面
- `event_setting.html` - イベント設定画面
- `event_requests.html` - イベント予約管理画面

#### DB対応テーブル
- `cbh_events` - イベントマスタ
- `cbh_event_reservations` - イベント予約データ
- `cbh_event_flows` - イベントフロー
- `cbh_event_features` - イベント特徴

#### 必要な調整項目

##### イベントマスタ（cbh_events）
| カラム名 | 現在のUI | 必要な調整 |
|---------|----------|-----------|
| title | あり | - |
| start_date/end_date | あり | - |
| explanation_title | なし | 説明タイトル入力欄を追加 |
| explanation_description | なし | 説明文入力欄を追加 |
| date_and_time | あり | - |
| prefecture_id | あり | - |
| city | あり | - |
| address_line | あり | - |
| time_required | なし | 所要時間入力欄を追加 |
| about_reservations | なし | 予約についての説明欄を追加 |
| reservation_deadline | なし | 予約締切入力欄を追加 |
| genre | なし | ジャンル選択を追加 |
| participation_fee | なし | 参加費入力欄を追加 |
| phone_number | あり | - |
| organizer_name | なし | 主催者名入力欄を追加 |
| max_participants | なし | 最大参加者数入力欄を追加 |

##### イベント予約データ（cbh_event_reservations）
| カラム名 | 現在のUI | 必要な調整 |
|---------|----------|-----------|
| event_id | あり | - |
| customer_id | あり | - |
| preferred_date | あり | - |
| preferred_time | あり | - |
| preferred_contact_time | なし | 希望連絡時間帯を追加 |
| question_text | あり | - |
| scheduled_construction_period | なし | 建築予定時期選択を追加 |
| construction_budget | なし | 建築予算選択を追加 |
| scheduled_construction_site | なし | 建築予定地入力を追加 |
| scheduled_construction_site_prefecture_id | なし | 建築予定地都道府県選択を追加 |
| scheduled_construction_site_city | なし | 建築予定地市区町村入力を追加 |
| scheduled_construction_site_address_line | なし | 建築予定地住所詳細入力を追加 |
| construction_details | なし | 建築詳細入力欄を追加 |
| current_situation | なし | 現在の状況選択を追加 |
| is_newsletter_subscribed | なし | メルマガ購読チェックボックスを追加 |
| status | あり | - |

### 2.3 見学予約管理（モデルハウス）

#### 対象ファイル
- `tour_schedule_list.html` - 見学スケジュール一覧
- `tour_schedule_register.html` - 見学スケジュール登録
- `tour_reservation_list.html` - 見学予約一覧
- `tour_reservation_register.html` - 見学予約登録
- `tour_user_reservation_list.html` - ユーザー予約一覧

#### DB対応テーブル
- `cbh_model_houses` - モデルハウスマスタ
- `cbh_model_house_tour_reservations` - 見学予約データ
- `cbh_model_house_reservation_schedules` - 予約スケジュール
- `cbh_model_house_reservation_time_slots` - 予約時間枠

#### 必要な調整項目

##### モデルハウスマスタ（cbh_model_houses）
| カラム名 | 現在のUI | 必要な調整 |
|---------|----------|-----------|
| name | あり | - |
| title | なし | タイトル入力欄を追加 |
| description | あり | - |
| sub_title | なし | サブタイトル入力欄を追加 |
| sub_description | なし | サブ説明文入力欄を追加 |
| sort_number | なし | 表示順入力欄を追加 |
| business_hours | あり | - |
| holidays | なし | 休業日入力欄を追加 |
| prefecture_id | あり | - |
| postal_code | あり | - |
| city | あり | - |
| address_line | あり | - |
| access | なし | アクセス情報入力欄を追加 |
| reservation_deadline | なし | 予約締切設定を追加 |
| exhibition_venue_name | なし | 展示場名入力欄を追加 |
| has_benefits | なし | 特典有無チェックボックスを追加 |

##### 見学予約データ（cbh_model_house_tour_reservations）
| カラム名 | 現在のUI | 必要な調整 |
|---------|----------|-----------|
| model_house_id | あり | - |
| customer_id | あり | - |
| preferred_date | あり | - |
| preferred_time | あり | - |
| preferred_contact_time | なし | 希望連絡時間帯を追加 |
| question_text | あり | - |
| scheduled_construction_period | なし | 建築予定時期選択を追加 |
| construction_budget | なし | 建築予算選択を追加 |
| scheduled_construction_site | なし | 建築予定地入力を追加 |
| scheduled_construction_site_prefecture_id | なし | 建築予定地都道府県選択を追加 |
| scheduled_construction_site_city | なし | 建築予定地市区町村入力を追加 |
| scheduled_construction_site_address_line | なし | 建築予定地住所詳細入力を追加 |
| construction_details | なし | 建築詳細入力欄を追加 |
| current_situation | なし | 現在の状況選択を追加 |
| is_newsletter_subscribed | なし | メルマガ購読チェックボックスを追加 |
| status | あり | - |

##### 予約スケジュール管理（cbh_model_house_reservation_schedules）
| カラム名 | 現在のUI | 必要な調整 |
|---------|----------|-----------|
| date | あり | - |
| status | あり | ◯△✕のステータス表示を実装 |

##### 予約時間枠（cbh_model_house_reservation_time_slots）
| カラム名 | 現在のUI | 必要な調整 |
|---------|----------|-----------|
| time | あり | - |
| is_main | なし | メイン見学時間フラグを追加 |
| max_capacity | なし | 最大予約可能数設定を追加 |
| current_capacity | なし | 現在の予約数表示を追加 |

## 3. 共通調整項目

### 3.1 顧客情報（users）
全ての予約・請求フォームで以下の項目を統一：
- 氏名（姓・名）
- 氏名かな（姓・名）
- メールアドレス
- 電話番号
- 郵便番号
- 都道府県
- 住所1（市区町村）
- 住所2（番地以降）
- メール通知設定
- 生年月日
- 性別

### 3.2 選択肢マスタの実装
以下のマスタデータを選択肢として実装：
- 都道府県マスタ（prefectures）
- サービス種別マスタ（property_service_types）
- 特徴タグマスタ（cbh_feature_tags）
- 特徴カテゴリマスタ（cbh_feature_categories）

## 4. 実装チケット

### チケット1: カタログ請求管理UI改修
**対象ファイル**: 
- catalog_custom_home_list.html
- catalog_custom_home_register.html

**作業内容**:
1. カタログマスタの不足項目追加
2. カタログ請求フォームの項目追加
3. バリデーション実装
4. 一覧画面の表示項目調整

### チケット2: イベント予約管理UI改修
**対象ファイル**: 
- event_setting.html
- event_requests.html

**作業内容**:
1. イベントマスタの不足項目追加
2. イベント予約フォームの項目追加
3. イベントフロー管理機能追加
4. 予約ステータス管理機能強化

### チケット3: 見学予約管理UI改修
**対象ファイル**: 
- tour_reservation_list.html
- tour_reservation_register.html
- tour_schedule_register.html

**作業内容**:
1. モデルハウスマスタの不足項目追加
2. 見学予約フォームの項目追加
3. 予約スケジュール管理機能強化
4. 時間枠管理機能の実装

### チケット4: 共通コンポーネント実装
**作業内容**:
1. 顧客情報入力フォームコンポーネント
2. 都道府県選択コンポーネント
3. 建築予定情報入力コンポーネント
4. ステータス表示コンポーネント

## 5. 実装優先順位
1. 共通コンポーネント実装（チケット4）
2. カタログ請求管理UI改修（チケット1）
3. イベント予約管理UI改修（チケット2）
4. 見学予約管理UI改修（チケット3）

## 6. 注意事項
- 全ての日付項目にはカレンダーピッカーを実装
- 必須項目には赤いアスタリスク（*）を表示
- フォームのバリデーションはリアルタイムで実施
- 削除機能には確認ダイアログを表示
- ステータス変更は履歴を保持
- メール通知設定は個人情報保護の観点から慎重に扱う