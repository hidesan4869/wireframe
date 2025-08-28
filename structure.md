# 不動産ポータルサイト 業務フロー要件定義書

## 1. システム概要

### 1.1 ビジネスモデル
- **収益構造**: 成果報酬型（成約時のみ報酬発生）
- **サービス形態**: 不動産会社と顧客をマッチングするポータルサイト
- **管理方式**: 月次報告による進捗管理と個別送客承認制

### 1.2 主要なステークホルダー
- **顧客**: 不動産購入・建築を検討する一般消費者
- **企業**: 登録不動産会社・工務店・ハウスメーカー
- **弊社**: ポータルサイト運営会社

### 1.3 システムの基本原則
- **企業自主管理**: 重複送客の判定と管理は企業の責任
- **月次一括報告**: 企業は月単位で進捗をまとめて報告
- **個別承認制**: 弊社は送客単位で個別に承認判定
- **証跡管理**: 成約・着工時の必須書類管理

---

## 2. 顧客獲得フロー

### 2.1 資料請求機能
**業務要件**
- 顧客がWebフォームから企業の資料を請求
- 物件種別・予算・希望エリア等の詳細情報を収集
- 企業へのリアルタイム通知（メール・管理画面）

**システム要件**
```sql
-- 資料請求データの記録
INSERT INTO document_requests (
    company_id, customer_name, customer_email, customer_phone,
    property_type, budget_range, area_preference, document_type,
    requested_at
) VALUES (...);
```

**画面要件**
- 顧客向け：資料請求フォーム、完了画面
- 企業向け：資料請求一覧、詳細画面、対応状況管理

### 2.2 イベント予約機能
**業務要件**
- 展示会・セミナー・見学会等のイベント予約
- 参加人数・関心物件種別の事前収集
- 予約確認・変更・キャンセル機能

**システム要件**
```sql
-- イベント予約データの記録
INSERT INTO event_reservations (
    company_id, event_name, event_date, event_time,
    customer_name, customer_email, customer_phone,
    participant_count, property_interest,
    reserved_at
) VALUES (...);
```

**画面要件**
- 顧客向け：イベント一覧、予約フォーム、予約確認
- 企業向け：イベント管理、参加者一覧、出席管理

### 2.3 物件見学予約機能
**業務要件**
- モデルハウス・完成物件の見学予約
- 見学日時の調整機能
- 見学目的・要望の事前収集

**システム要件**
```sql
-- 見学予約データの記録
INSERT INTO property_visit_reservations (
    company_id, property_name, property_address,
    visit_date, visit_time, customer_name, customer_email,
    customer_phone, visitor_count, visit_purpose,
    requested_at
) VALUES (...);
```

**画面要件**
- 顧客向け：物件一覧、見学予約フォーム、日時調整
- 企業向け：見学予約管理、スケジュール調整、顧客対応履歴

---

## 3. 送客管理フロー

### 3.1 送客作成業務
**業務要件**
- 企業が顧客アクション（資料請求・イベント・見学）を受けて送客判定
- 継続営業価値のある顧客のみを送客として登録
- 初回送客フラグの企業判定・設定
- 規定期間内（システム設定可能）での送客作成義務

**送客判定基準**
- 継続的な営業活動を行う価値がある顧客
- 明確な購入・建築意向がある顧客
- 予算・エリア・時期等の条件が合致する顧客

**システム要件**
```sql
-- 送客レコードの作成
INSERT INTO customer_referrals (
    company_id, customer_name, customer_email, customer_phone,
    is_first_referral,    -- 企業が判定
    current_status_id,    -- 1:未対応で初期化
    referred_at
) VALUES (...);

-- 送客元との関連付け（起因により分岐）
INSERT INTO document_request_referrals (document_request_id, referral_id) VALUES (...);
INSERT INTO event_referrals (event_reservation_id, referral_id) VALUES (...);
INSERT INTO property_visit_referrals (property_visit_id, referral_id) VALUES (...);
```

### 3.2 重複送客管理
**業務要件**
- **企業責任**: 重複判定は企業が自主的に実施
- **営業停止設定**: 重複と判定した送客は「営業停止」ステータスに変更
- **月次報告**: 営業停止理由を月次報告で説明
- **システム非介入**: 自動重複チェック機能は実装しない

**重複判定基準（企業ガイドライン）**
- 同一顧客からの複数回の問い合わせ
- 他社経由で既に接触済みの顧客
- 既存顧客・過去顧客との重複

**システム要件**
- 重複チェック機能：実装しない
- 営業停止理由：monthly_referral_reports.progress_notesで記録
- 弊社確認：承認時に不自然な営業停止がないかをマニュアルチェック

---

## 4. 月次報告・承認フロー

### 4.1 企業月次報告業務
**業務要件**
- **報告期限**: 毎月末日までに前月分を報告
- **報告単位**: 企業×月単位での一括報告
- **報告内容**: 全送客の進捗状況・ステータス変更・契約情報
- **必須項目**: 送客ID・現在ステータス・進捗メモ
- **任意項目**: 契約金額・着工予定日・完成予定日

**システム要件**
```sql
-- 月次報告の作成
INSERT INTO monthly_reports (
    company_id, report_year, report_month,
    total_referrals_reported, submitted_at
) VALUES (...);

-- 個別送客の詳細報告
INSERT INTO monthly_referral_reports (
    monthly_report_id, referral_id, reported_status_id,
    contract_amount, expected_construction_date, expected_completion_date,
    progress_notes, evidence_documents
) VALUES (...);
```

**画面要件**
- 送客一覧表示（前回報告時からの差分表示）
- ステータス一括変更機能
- 契約情報入力フォーム（成約時）
- 進捗メモ一括編集機能
- 提出前確認画面・提出完了画面

### 4.2 弊社承認業務
**業務要件**
- **承認期限**: 月初10営業日以内での承認完了
- **承認単位**: 送客単位での個別承認
- **承認時間**: 約1分/件での効率的な確認作業
- **承認判定**: データ整合性・ステータス遷移の妥当性確認

**承認確認項目**
- ステータス変更の合理性（前回→今回）
- 契約情報の整合性（成約時）
- 進捗メモの具体性・説明責任
- 不自然な営業停止の有無

**システム要件**
```sql
-- 承認処理の実行
INSERT INTO monthly_referral_approvals (
    monthly_referral_report_id, approval_status_id,
    approved_at, approved_by, rejection_reason
) VALUES (...);

-- 送客ステータスの確定
UPDATE customer_referrals 
SET current_status_id = ?, updated_at = NOW()
WHERE id IN (承認された送客IDリスト);

-- 承認履歴の記録
INSERT INTO referral_approval_history (
    monthly_referral_approval_id, previous_status_id, new_status_id,
    changed_by, change_reason, changed_at
) VALUES (...);
```

**画面要件**
- 企業別月次報告一覧
- 送客別詳細確認画面
- 一括承認・個別承認機能
- 却下理由入力フォーム
- 承認状況ダッシュボード

### 4.3 承認ステータス管理
**承認ステータス定義**
- **1: 承認待ち**: 企業報告提出直後の初期状態
- **2: 承認済み**: 弊社承認完了、送客ステータス確定
- **3: 却下**: 承認拒否、企業に修正・説明を要求
- **4: 修正要求**: 部分的な修正が必要、再提出待ち

**業務ルール**
- 承認済み送客は送客ステータス確定、以後変更不可（例外処理除く）
- 却下・修正要求時は理由を必須入力
- 修正要求への対応期限：10営業日以内

---

## 5. 成約後管理フロー

### 5.1 成約承認時の自動処理
**業務要件**
- ステータス「9:成約済み」承認時の自動的な着工管理レコード作成
- 契約情報（金額・日程）の着工管理への自動転記
- 誓約書アップロード要求の自動通知

**システム要件**
```sql
-- 成約承認時の着工管理自動作成
INSERT INTO construction_management (
    referral_id, contract_date, contract_amount,
    construction_status,           -- 'planned'で初期化
    planned_start_date, planned_completion_date,
    progress_percentage,           -- 0で初期化
    is_verified                    -- FALSEで初期化
) 
SELECT mrr.referral_id, CURDATE(), mrr.contract_amount,
       'planned', mrr.expected_construction_date, mrr.expected_completion_date,
       0, FALSE
FROM monthly_referral_reports mrr
JOIN monthly_referral_approvals mra ON mrr.id = mra.monthly_referral_report_id
WHERE mra.approval_status_id = 2 AND mrr.reported_status_id = 9;
```

### 5.2 着工・工事進捗管理
**業務要件**
- **着工開始時**: 実際着工日・施工業者・建築確認番号の登録
- **定期進捗更新**: 工事進捗率・現在工事段階・完成予定日の更新
- **工事完成時**: 実際完成日・進捗率100%の設定

**進捗管理項目**
- 工事進捗率（0-100%）
- 現在工事段階（基礎工事・上棟・内装工事・外構工事等）
- 検査状況・品質課題
- スケジュール遅延・追加費用
- 次回検査予定日

**システム要件**
```sql
-- 着工開始時の更新
UPDATE construction_management SET
    construction_status = 'started',
    actual_start_date = ?,
    contractor_name = ?,
    construction_permit_number = ?,
    updated_at = NOW()
WHERE referral_id = ?;

-- 定期進捗更新
UPDATE construction_management SET
    progress_percentage = ?,
    current_phase = ?,
    inspection_status = ?,
    estimated_completion_date = ?,
    progress_notes = ?,
    updated_at = NOW()
WHERE referral_id = ?;
```

---

## 6. ファイル管理フロー

### 6.1 必須ファイル管理
**成約時必須：誓約書**
- **提出義務**: 成約承認から30日以内
- **ファイル形式**: PDF推奨（画像ファイルも可）
- **確認事項**: 署名・押印・日付の確認

**着工時必須：着工報告書**
- **提出義務**: 着工開始から7日以内
- **ファイル形式**: PDF・Excel・Word等
- **確認事項**: 着工日・施工業者・建築確認番号

**システム要件**
```sql
-- 必須ファイルのアップロード
INSERT INTO media_uploads (
    referral_id, file_type, file_name, file_path,
    file_size, mime_type, upload_date, description
) VALUES (...);
-- file_type: 'pledge_document' または 'construction_report'
```

### 6.2 任意ファイル管理
**工事写真・進捗資料**
- **アップロード**: 企業の任意タイミング
- **分類**: 工事段階別（基礎・上棟・内装・完成等）
- **用途**: 進捗確認・品質管理・顧客報告

**システム要件**
```sql
-- 任意ファイルのアップロード
INSERT INTO media_uploads (
    referral_id, file_type, file_name, file_path,
    description,    -- '基礎工事写真', '上棟式', '完成写真'等
    upload_date
) VALUES (...);
-- file_type: 'construction_photo', 'progress_report', 'other'
```

### 6.3 ファイル確認・承認業務
**業務要件**
- **確認期限**: アップロードから5営業日以内
- **確認内容**: ファイル内容の適切性・必要情報の記載
- **承認処理**: is_verified フラグの更新

**システム要件**
```sql
-- ファイル承認処理
UPDATE media_uploads SET
    is_verified = TRUE,
    verified_at = NOW(),
    verified_by = ?
WHERE id = ?;
```

---

## 7. データ整合性・監査要件

### 7.1 データ整合性チェック
**送客管理の整合性**
- 全送客が月次報告に含まれているか
- ステータス変更履歴の連続性
- 重複送客の適切な管理

**承認処理の整合性**
- 承認済み送客のステータス確定
- 未承認送客の一時的なステータス保持
- 承認履歴の完全性

### 7.2 監査ログ・履歴管理
**変更履歴の記録**
- 送客ステータス変更履歴
- 承認ステータス変更履歴
- ファイルアップロード・承認履歴

**システム要件**
- 全テーブルのcreated_at, updated_at管理
- 重要な変更操作の履歴テーブル記録
- 操作者・操作時刻・変更理由の記録

### 7.3 レポート・分析機能
**運用管理レポート**
- 月次報告提出状況（企業別・期限遵守率）
- 承認処理状況（処理時間・却下率）
- 必須ファイル提出状況

**ビジネス分析レポート**
- 送客数・成約率・成約金額（企業別・月別）
- 工事進捗状況・完成率
- 顧客獲得チャネル分析（資料請求・イベント・見学）

---

## 8. システム運用要件

### 8.1 パフォーマンス要件
- **応答時間**: 画面表示2秒以内、検索処理5秒以内
- **同時利用者数**: 100企業×3名＝300ユーザーの同時アクセス
- **データ保存期間**: 最低7年間の履歴保持

### 8.2 セキュリティ要件
- **認証**: 企業別ログイン・権限管理
- **データ暗号化**: 顧客個人情報の暗号化保存
- **アクセスログ**: 全操作のログ記録・監査対応

### 8.3 バックアップ・復旧要件
- **データバックアップ**: 日次自動バックアップ
- **復旧時間**: 4時間以内での復旧完了
- **データ整合性**: バックアップ時点での完全性保証

### 8.4 通知・アラート機能
**企業向け通知**
- 新規顧客アクション通知（資料請求・予約）
- 月次報告提出期限アラート
- 必須ファイル提出期限アラート

**弊社管理者向け通知**
- 月次報告提出状況アラート
- 承認処理期限アラート
- システムエラー・障害通知

この要件定義書に基づいて、16テーブル構成での不動産ポータルサイトの開発・運用が可能となります。