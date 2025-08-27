# 不動産ポータルサイト 運用フロー定義書

## システム概要

### 収益モデル
- **成果報酬制**：成約時のみ報酬発生
- **手数料計算不要**：システム内での手数料計算機能は実装しない

### 承認方式
- **月次報告承認**：企業が月単位で提出する報告を弊社が承認
- **個別承認**：各送客のステータス変更を個別に承認

### 重複送客管理
- **企業自主管理**：企業が「営業停止」ステータスで重複を管理
- **システム重複チェック無し**：自動重複検知機能は実装しない

### 必須ファイル管理
- **成約時**：誓約書のアップロード必須
- **着工時**：着工報告書のアップロード必須

---

## Phase 1: 顧客初回アクション

### 1-1. 資料請求
**顧客操作**
```
Webフォームから資料請求送信
├── 企業選択
├── 顧客基本情報入力（氏名、メール、電話）
├── 希望条件入力（物件種別、予算帯、希望エリア）
└── 資料種別選択
```

**システム処理**
```sql
-- document_requestsテーブル INSERT
INSERT INTO document_requests (
    company_id,
    customer_name,
    customer_email,
    customer_phone,
    property_type,
    budget_range,
    area_preference,
    document_type,
    requested_at
) VALUES (...);
```

### 1-2. イベント予約
**顧客操作**
```
イベント予約フォームから予約送信
├── イベント選択
├── 顧客基本情報入力
├── 参加人数入力
└── 関心物件種別選択
```

**システム処理**
```sql
-- event_reservationsテーブル INSERT
INSERT INTO event_reservations (
    company_id,
    event_name,
    event_date,
    event_time,
    customer_name,
    customer_email,
    customer_phone,
    participant_count,
    property_interest,
    reserved_at
) VALUES (...);
```

### 1-3. 物件見学予約
**顧客操作**
```
見学予約フォームから予約送信
├── 物件選択
├── 見学希望日時入力
├── 顧客基本情報入力
├── 見学者数入力
└── 見学目的入力
```

**システム処理**
```sql
-- property_visit_reservationsテーブル INSERT
INSERT INTO property_visit_reservations (
    company_id,
    property_name,
    property_address,
    visit_date,
    visit_time,
    customer_name,
    customer_email,
    customer_phone,
    visitor_count,
    visit_purpose,
    requested_at
) VALUES (...);
```

**重要ポイント**
- 各テーブルにはステータス管理フィールドなし（記録のみ）
- 企業への通知は別システムで実装

---

## Phase 2: 企業による送客作成

### 2-1. 送客判定基準
**企業の判断基準**
```
以下の条件で送客作成を判定：
├── 継続的な営業活動を行う価値がある顧客
├── 規定日数以内（システム設定）
└── 重複送客でない（企業が自主判定）
```

### 2-2. 送客作成処理
**Step 1: 送客管理レコード作成**
```sql
-- customer_referralsテーブル INSERT
INSERT INTO customer_referrals (
    company_id,
    customer_name,
    customer_email,
    customer_phone,
    is_first_referral,           -- 企業が判定・設定
    current_status_id,           -- 1(未対応)で初期化
    referred_at
) VALUES (...);
```

**Step 2: 送客元との関連付け（起因により分岐）**
```sql
-- 資料請求起因の場合
INSERT INTO document_request_referrals (
    document_request_id,
    referral_id
) VALUES (...);

-- イベント起因の場合  
INSERT INTO event_referrals (
    event_reservation_id,
    referral_id
) VALUES (...);

-- 見学起因の場合
INSERT INTO property_visit_referrals (
    property_visit_id,
    referral_id
) VALUES (...);
```

### 2-3. 重複送客の自主管理
**企業の対応方法**
```
重複検知時の企業操作：
├── 「営業停止」（ステータスID: 13）に変更
├── 月次報告で「営業停止」として報告
└── 弊社は承認時にデータ整合性のみチェック
```

**システム対応**
```
重複チェック機能：
├── 自動重複検知機能は実装しない
├── 企業の自主管理に委ねる運用
└── 承認時に明らかな異常があれば手動確認
```

---

## Phase 3: 日常のステータス管理

### 3-1. ステータス更新タイミング
**現在の運用**
```
即座のステータス更新
└── 資料請求、イベント予約、見学予約のアクションから○○日以内に作成
```

---

## Phase 4: 月次報告・承認フロー

### 4-1. 企業による月次報告提出
**提出期限**
```
毎月末日まで：前月分の進捗報告提出
```

**Step 1: 月次報告レコード作成**
```sql
-- monthly_reportsテーブル INSERT
INSERT INTO monthly_reports (
    company_id,
    report_year,
    report_month,
    total_referrals_reported,    -- 報告対象送客数
    approval_status_id,          -- 1(承認待ち)
    submitted_at
) VALUES (...);
```

**Step 2: 個別送客の詳細報告**
```sql
-- monthly_referral_reportsテーブル INSERT（対象送客分繰り返し）
INSERT INTO monthly_referral_reports (
    monthly_report_id,           -- Step1で作成したID
    referral_id,
    reported_status_id,          -- 報告時点のステータス
    contract_amount,             -- 成約時のみ
    expected_construction_date,  -- 成約時のみ
    expected_completion_date,    -- 成約時のみ
    progress_notes,
    evidence_documents           -- JSON形式で証跡資料情報
) VALUES (...);
```

### 4-2. 弊社による承認処理
**承認業務フロー**
```
月初処理（約1分/件）：
├── 前月分の月次報告一覧確認
├── 各企業の送客ステータス変更履歴確認
├── データ整合性チェック
├── 不自然なステータス変更確認
└── 承認・却下判定
```

**Step 1: 承認履歴記録**
```sql
-- monthly_report_approvalsテーブル INSERT
INSERT INTO monthly_report_approvals (
    monthly_report_id,
    previous_status_id,          -- 1(承認待ち)
    new_status_id,              -- 2(承認済み)
    changed_by,
    change_reason
) VALUES (...);
```

**Step 2: 月次報告承認**
```sql
-- monthly_reportsテーブル UPDATE
UPDATE monthly_reports SET
    approval_status_id = 2,      -- 2(承認済み)
    approved_at = NOW(),
    approved_by = '承認者名'
WHERE id = ?;
```

**Step 3: 送客ステータス確定**
```sql
-- 承認された送客のステータス更新
UPDATE customer_referrals cr
JOIN monthly_referral_reports mrr ON cr.id = mrr.referral_id
SET cr.current_status_id = mrr.reported_status_id,
    cr.updated_at = NOW()
WHERE mrr.monthly_report_id = ?;
```

**Step 4: 成約承認時の特別処理**
```sql
-- ステータスID = 9(成約済み)になった送客について
-- construction_managementテーブル自動作成
INSERT INTO construction_management (
    referral_id,
    contract_date,               -- 月次報告から取得
    contract_amount,             -- 月次報告から取得
    construction_status,         -- 'planned'(予定)
    planned_start_date,          -- expected_construction_dateから取得
    planned_completion_date      -- expected_completion_dateから取得
) 
SELECT mrr.referral_id, ..., ...
FROM monthly_referral_reports mrr
WHERE mrr.monthly_report_id = ? AND mrr.reported_status_id = 9;

-- 誓約書アップロード要求（システム通知）
-- media_uploadsには手動アップロード後に記録
```

---

## Phase 5: 成約後の管理

### 5-1. 成約承認時の着工管理作成
**自動処理**
```
成約承認時（ステータスID = 9）に以下を自動実行：
├── construction_managementレコード自動作成
├── 初期ステータス：'planned'（着工予定）
├── 契約情報：月次報告から自動設定
└── 誓約書アップロード要求通知
```

**着工管理初期値**
```sql
-- 自動作成される初期値
├── construction_status: 'planned'
├── progress_percentage: 0
├── is_verified: FALSE
├── verified_at: NULL
└── その他：月次報告データから転記
```

### 5-2. 着工・工事進捗管理
**着工開始時の企業操作**
```sql
-- construction_managementテーブル UPDATE
UPDATE construction_management SET
    construction_status = 'started',
    actual_start_date = '実際の着工日',
    contractor_name = '施工業者名',
    construction_permit_number = '建築確認番号',
    updated_at = NOW()
WHERE referral_id = ?;

-- 着工報告書アップロード
INSERT INTO media_uploads (
    referral_id,
    file_type,                   -- 'construction_report'
    file_name,
    file_path,
    upload_date
) VALUES (...);
```

**定期的な進捗更新**
```sql
-- 工事進捗更新
UPDATE construction_management SET
    progress_percentage = ?,     -- 0-100%
    current_phase = '現在の工事段階',
    progress_notes = '進捗メモ',
    estimated_completion_date = '更新された完成予定日',
    next_inspection_date = '次回検査予定日',
    updated_at = NOW()
WHERE referral_id = ?;

-- 工事写真アップロード（任意）
INSERT INTO media_uploads (
    referral_id,
    file_type,                   -- 'construction_photo'
    description,                 -- '基礎工事', '上棟', '内装' 等
    ...
) VALUES (...);
```

**完成時の処理**
```sql
UPDATE construction_management SET
    construction_status = 'completed',
    actual_completion_date = '実際の完成日',
    progress_percentage = 100,
    updated_at = NOW()
WHERE referral_id = ?;
```

---

## Phase 6: ファイル管理

### 6-1. 必須ファイル管理
**成約時必須：誓約書**
```sql
INSERT INTO media_uploads (
    referral_id,
    file_type,                   -- 'pledge_document'
    file_name,
    file_path,
    file_size,
    mime_type,
    upload_date,
    description                  -- '誓約書'
) VALUES (...);
```

**着工時必須：着工報告書**
```sql
INSERT INTO media_uploads (
    referral_id,
    file_type,                   -- 'construction_report'  
    file_name,
    file_path,
    upload_date,
    description                  -- '着工報告書'
) VALUES (...);
```

### 6-2. 必須ファイルチェック機能
**システム機能**
```sql
-- 成約済みで誓約書未提出の送客抽出
SELECT cr.id, cr.customer_name, c.company_name
FROM customer_referrals cr
JOIN companies c ON cr.company_id = c.id
WHERE cr.current_status_id = 9  -- 成約済み
AND NOT EXISTS (
    SELECT 1 FROM media_uploads mu 
    WHERE mu.referral_id = cr.id 
    AND mu.file_type = 'pledge_document'
);

-- 着工開始で着工報告書未提出の案件抽出
SELECT cm.id, cr.customer_name, c.company_name
FROM construction_management cm
JOIN customer_referrals cr ON cm.referral_id = cr.id
JOIN companies c ON cr.company_id = c.id
WHERE cm.construction_status = 'started'
AND NOT EXISTS (
    SELECT 1 FROM media_uploads mu 
    WHERE mu.referral_id = cr.id 
    AND mu.file_type = 'construction_report'
);
```

### 6-3. ファイル確認・承認
**弊社業務**
```
アップロードファイル確認：
├── ファイル内容の適切性チェック
├── is_verifiedフラグ更新
├── verified_atに確認日時記録
└── verified_byに確認者記録
```

```sql
-- ファイル確認完了時
UPDATE media_uploads SET
    is_verified = TRUE,
    verified_at = NOW(),
    verified_by = '確認者名'
WHERE id = ?;
```

---

## 運用上の重要ポイント

### データ整合性の確保
1. **月次報告の網羅性**：全送客が月次報告に含まれているかチェック
2. **ステータス遷移の妥当性**：不正なステータス変更がないかチェック  
3. **必須ファイルの完備**：成約・着工時の必須ファイル提出確認

### 承認業務の効率化
1. **バッチ承認**：問題のない報告は一括承認
2. **例外処理**：異常なケースのみ個別確認
3. **督促機能**：未提出・遅延の自動通知

### システム運用の監視
1. **月次報告提出率**：企業別の提出状況監視
2. **ファイルアップロード率**：必須ファイルの提出状況監視
3. **承認処理速度**：承認業務の処理時間監視

### 将来的な機能拡張
1. **リアルタイムステータス更新**：月次報告以外でのステータス更新
2. **自動重複チェック**：システムによる重複送客検知
3. **手数料計算機能**：成果報酬の自動計算
4. **レポート機能**：各種統計レポートの自動生成