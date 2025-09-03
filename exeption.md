# データフロー分析：正常ケースと例外処理

## 1. 正常データフロー

### 1.1 顧客アクション → 送客作成フロー
```
【Step 1】顧客アクション記録
document_requests/event_reservations/property_visit_reservations
    ↓ INSERT
【Step 2】企業による送客判定・作成
customer_referrals (current_status_id = 1:未対応)
    ↓ INSERT
【Step 3】送客元関連付け
document_request_referrals/event_referrals/property_visit_referrals
    ↓ INSERT (中間テーブル)
```

### 1.2 月次報告 → 承認フロー
```
【Step 1】企業月次報告提出
monthly_reports (submitted_at = NOW())
    ↓ INSERT
monthly_referral_reports (各送客の詳細)
    ↓ INSERT (複数レコード)

【Step 2】弊社個別承認処理
monthly_referral_approvals (approval_status_id = 2:承認済み)
    ↓ INSERT (承認対象分のみ)
referral_approval_history (承認履歴)
    ↓ INSERT

【Step 3】送客ステータス確定
customer_referrals.current_status_id
    ↓ UPDATE (承認済み送客のみ)

【Step 4】成約時の特別処理
construction_management (成約承認時のみ)
    ↓ INSERT (ステータス9承認時)
```

### 1.3 ファイル管理フロー
```
【成約時】
media_uploads (file_type = 'pledge_document')
    ↓ INSERT
【着工時】
media_uploads (file_type = 'construction_report')
    ↓ INSERT
【弊社確認】
media_uploads.is_verified = TRUE
    ↓ UPDATE
```

---

## 2. 例外処理のデータフロー

### 2.1 承認却下ケース（approval_status_id = 3）

**発生条件**
- データ不整合（契約金額未入力等）
- ステータス変更の説明不足
- 不自然なステータス遷移

**データフロー**
```sql
-- 却下処理
INSERT INTO monthly_referral_approvals (
    monthly_referral_report_id,
    approval_status_id,        -- 3:却下
    rejection_reason,          -- 却下理由（必須）
    approved_by,
    created_at
) VALUES (...);

-- 却下履歴記録
INSERT INTO referral_approval_history (
    monthly_referral_approval_id,
    previous_status_id,        -- 1:承認待ち
    new_status_id,            -- 3:却下
    changed_by,
    change_reason,
    changed_at
) VALUES (...);

-- 重要：customer_referralsは更新しない
-- 送客ステータスは前回承認時のまま保持
```

**業務影響**
- 企業に却下通知・修正要求
- 該当送客は次回月次報告で再提出必要
- 送客の現在ステータスは変更されない（前回承認時のまま）

### 2.2 修正要求ケース（approval_status_id = 4）

**発生条件**
- 部分的な情報不足
- 軽微なデータ不整合
- 追加説明が必要な案件

**データフロー**
```sql
-- 修正要求処理
INSERT INTO monthly_referral_approvals (
    monthly_referral_report_id,
    approval_status_id,        -- 4:修正要求
    rejection_reason,          -- 修正内容の指示
    approved_by,
    created_at
) VALUES (...);

-- 修正要求履歴記録
INSERT INTO referral_approval_history (
    monthly_referral_approval_id,
    previous_status_id,        -- 1:承認待ち
    new_status_id,            -- 4:修正要求
    changed_by,
    change_reason,
    changed_at
) VALUES (...);
```

### 2.3 再提出処理フロー

**企業による修正・再提出**
```sql
-- 既存の月次送客報告を修正
UPDATE monthly_referral_reports SET
    reported_status_id = ?,
    contract_amount = ?,
    progress_notes = ?,        -- 修正内容を反映
    evidence_documents = ?,
    updated_at = NOW()
WHERE id = ?;

-- 再承認申請（新しい承認レコード作成）
INSERT INTO monthly_referral_approvals (
    monthly_referral_report_id,  -- 同じmonthly_referral_report_id
    approval_status_id,          -- 1:承認待ち（再申請）
    created_at
) VALUES (...);

-- 再申請履歴記録
INSERT INTO referral_approval_history (
    monthly_referral_approval_id,
    previous_status_id,          -- 4:修正要求 または 3:却下
    new_status_id,              -- 1:承認待ち
    changed_by,                 -- 企業担当者
    change_reason,              -- 修正内容
    changed_at
) VALUES (...);
```

### 2.4 データ状態の管理

**承認待ち状態の送客**
```sql
-- 承認待ちの送客一覧
SELECT cr.*, mrr.reported_status_id, mr.submitted_at
FROM customer_referrals cr
JOIN monthly_referral_reports mrr ON cr.id = mrr.referral_id
JOIN monthly_reports mr ON mrr.monthly_report_id = mr.id
LEFT JOIN monthly_referral_approvals mra ON mrr.id = mra.monthly_referral_report_id
WHERE mra.id IS NULL  -- まだ承認処理されていない
   OR mra.approval_status_id IN (3, 4);  -- 却下・修正要求
```

**ステータス不整合の検知**
```sql
-- 現在ステータスと最新報告ステータスの不整合チェック
SELECT cr.id, cr.current_status_id, mrr.reported_status_id
FROM customer_referrals cr
JOIN (
    SELECT referral_id, reported_status_id,
           ROW_NUMBER() OVER (PARTITION BY referral_id ORDER BY created_at DESC) as rn
    FROM monthly_referral_reports
) mrr ON cr.id = mrr.referral_id AND mrr.rn = 1
WHERE cr.current_status_id != mrr.reported_status_id;
```

---

## 3. 例外ケースでの複雑なシナリオ

### 3.1 月をまたいだ再提出

**問題設定**
- 1月分報告で送客A「成約済み」を報告→却下
- 2月分報告で送客A「成約済み」を再報告

**データフロー**
```sql
-- 1月分（却下済み）
monthly_reports (id=1, report_month=1) 
→ monthly_referral_reports (id=10, monthly_report_id=1, referral_id=100, reported_status_id=9)
→ monthly_referral_approvals (id=50, monthly_referral_report_id=10, approval_status_id=3)

-- 2月分（再提出）
monthly_reports (id=2, report_month=2)
→ monthly_referral_reports (id=20, monthly_report_id=2, referral_id=100, reported_status_id=9)
→ monthly_referral_approvals (id=60, monthly_referral_report_id=20, approval_status_id=2)

-- 最終的に送客ステータス更新
UPDATE customer_referrals SET current_status_id = 9 WHERE id = 100;
```

### 3.2 部分承認のケース

**問題設定**
- 企業A：1月分で送客5件報告
- 弊社：3件承認、2件却下

**データフロー**
```sql
-- 承認済み3件
INSERT INTO monthly_referral_approvals (approval_status_id = 2) VALUES (...), (...), (...);
-- 却下2件  
INSERT INTO monthly_referral_approvals (approval_status_id = 3) VALUES (...), (...);

-- 送客ステータス更新（承認済み3件のみ）
UPDATE customer_referrals SET current_status_id = ? WHERE id IN (承認済み3件のID);

-- 結果：却下2件は前回承認時のステータスのまま保持
```

### 3.3 承認取り消し・訂正処理

**業務要件**
- 承認後に誤りを発見した場合の訂正処理
- 承認取り消し後の再審査フロー

**データフロー**
```sql
-- 承認取り消し（新しい履歴レコードで管理）
INSERT INTO referral_approval_history (
    monthly_referral_approval_id,
    previous_status_id,          -- 2:承認済み
    new_status_id,              -- 1:承認待ち（再審査）
    changed_by,                 -- 管理者
    change_reason,              -- 取り消し理由
    changed_at
) VALUES (...);

-- 承認テーブルのステータス更新
UPDATE monthly_referral_approvals SET
    approval_status_id = 1,     -- 承認待ちに戻す
    updated_at = NOW()
WHERE id = ?;

-- 送客ステータスの巻き戻し（前回承認時の状態に戻す）
-- これは複雑なロジックが必要（前回の承認済みステータスを取得）
```

---

## 4. データ整合性の保証

### 4.1 整合性制約

**承認処理の制約**
- 1つのmonthly_referral_reportに対して複数のmonthly_referral_approvalsが存在可能（再提出用）
- customer_referrals.current_status_idは最新の承認済みstatusのみ反映
- 却下・修正要求の場合は送客ステータスを変更しない

**データ検証クエリ**
```sql
-- 承認済み送客のステータス整合性チェック
SELECT cr.id, cr.current_status_id, 
       MAX(CASE WHEN mra.approval_status_id = 2 THEN mrr.reported_status_id END) as latest_approved_status
FROM customer_referrals cr
LEFT JOIN monthly_referral_reports mrr ON cr.id = mrr.referral_id
LEFT JOIN monthly_referral_approvals mra ON mrr.id = mra.monthly_referral_report_id
GROUP BY cr.id, cr.current_status_id
HAVING cr.current_status_id != COALESCE(latest_approved_status, cr.current_status_id);
```

### 4.2 例外処理での運用ルール

**却下・修正要求時の運用**
- 企業への通知：24時間以内
- 修正期限：通知から10営業日以内
- 修正回数：同一月次報告につき最大3回まで

**システム監視項目**
- 長期間承認待ちの送客
- 繰り返し却下される送客
- ステータス整合性の異常

この分析により、正常フローだけでなく例外処理における複雑なデータ状態も適切に管理できる設計となっています。