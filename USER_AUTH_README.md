# ユーザー認証・権限管理機能 セットアップガイド

## 📋 概要

このシステムには、ユーザー認証とロールベースのアクセス制御（RBAC）機能が追加されました。

### 主な機能

✅ **ユーザー認証**
- ログイン/ログアウト
- パスワードのハッシュ化（SHA-256）
- セッション管理

✅ **ロールベースのアクセス制御**
- ユーザーごとに複数のロール割り当て可能
- ロールに基づいたページアクセス制御
- タブレベルの細かい権限設定

✅ **ユーザー管理**
- ユーザーの登録・編集・削除
- ロールの割り当て
- 管理者権限の設定

---

## 🚀 セットアップ手順

### 1. 前提条件

- **データベース**: MySQL 5.7以上
- **Pythonライブラリ**: `pymysql`がインストールされていること

```bash
# pymysqlのインストール（未インストールの場合）
pip install pymysql
```

### 2. データベース設定確認

`config.py`でMySQL接続情報が正しく設定されているか確認:

```python
@dataclass
class DatabaseConfig:
    host: str = 'localhost'
    user: str = 'root'
    password: str = 'your_password'
    database: str = 'kubota_db'
    port: int = 3306
```

### 3. データベースマイグレーション実行

認証機能用のテーブルを作成します。

```bash
# プロジェクトルートディレクトリで実行
cd d:\ts_pm_a

# マイグレーション実行
python migrations/add_user_auth_tables.py
```

**実行後の確認メッセージ:**
```
✅ ユーザー認証・権限管理テーブルを作成しました
✅ デフォルト管理者ユーザー: admin / admin123
```

**作成されるテーブル:**
- `users` - ユーザー情報
- `roles` - ロール定義
- `user_roles` - ユーザーとロールの紐付け
- `page_permissions` - ページ権限
- `tab_permissions` - タブ権限

### 4. アプリケーション起動

```bash
streamlit run main.py
```

### 5. 初回ログイン

ブラウザで `http://localhost:8501` を開き、デフォルト管理者でログイン:

- **ユーザー名**: `admin`
- **パスワード**: `admin123`

⚠️ **セキュリティのため、初回ログイン後にパスワードを変更してください**

---

## 👥 デフォルトロール

マイグレーション実行時に以下の4つのロールが自動作成されます:

| ロール名 | 説明 | アクセス可能なページ |
|---------|------|-------------------|
| **管理者** | 全ての機能にアクセス可能 | 全ページ（編集可） |
| **生産管理者** | 生産計画・製造工程の管理 | ダッシュボード、製品管理、生産計画、制限設定 |
| **配送管理者** | 配送便計画・納入進度の管理 | ダッシュボード、配送便計画、納入進度、会社カレンダー |
| **閲覧者** | 全画面の閲覧のみ可能 | 全ページ（閲覧のみ） |

---

## 🔧 ユーザー管理手順

### 新規ユーザーの作成

1. 管理者でログイン
2. サイドバーから「ユーザー管理」を選択
3. 「➕ 新規登録」タブを開く
4. ユーザー情報を入力して登録

### ロールの割り当て

1. 「ユーザー管理」→「🎭 ロール管理」タブ
2. ユーザーとロールを選択
3. 「➕ ロール割り当て」ボタンをクリック

### ユーザー情報の編集

1. 「ユーザー管理」→「👤 ユーザー一覧」タブ
2. 編集するユーザーを選択
3. 情報を修正して「💾 更新」

---

## 🗂️ データベーステーブル構造

### users（ユーザー情報）

| カラム名 | 型 | 説明 |
|---------|---|------|
| id | INTEGER | 主キー |
| username | TEXT | ユーザー名（ユニーク） |
| password_hash | TEXT | パスワードハッシュ |
| full_name | TEXT | 氏名 |
| email | TEXT | メールアドレス |
| is_active | BOOLEAN | 有効/無効 |
| is_admin | BOOLEAN | 管理者フラグ |
| last_login | TIMESTAMP | 最終ログイン日時 |

### roles（ロール定義）

| カラム名 | 型 | 説明 |
|---------|---|------|
| id | INTEGER | 主キー |
| role_name | TEXT | ロール名 |
| description | TEXT | 説明 |

### page_permissions（ページ権限）

| カラム名 | 型 | 説明 |
|---------|---|------|
| id | INTEGER | 主キー |
| role_id | INTEGER | ロールID |
| page_name | TEXT | ページ名 |
| can_view | BOOLEAN | 閲覧権限 |
| can_edit | BOOLEAN | 編集権限 |

### tab_permissions（タブ権限）

| カラム名 | 型 | 説明 |
|---------|---|------|
| id | INTEGER | 主キー |
| role_id | INTEGER | ロールID |
| page_name | TEXT | ページ名 |
| tab_name | TEXT | タブ名 |
| can_view | BOOLEAN | 閲覧権限 |

---

## 🔐 セキュリティ機能

### パスワードのハッシュ化

パスワードはSHA-256でハッシュ化され、平文では保存されません。

```python
# services/auth_service.py
def hash_password(password: str) -> str:
    return hashlib.sha256(password.encode()).hexdigest()
```

### セッション管理

Streamlitのセッション状態を使用して認証情報を管理:

```python
st.session_state['authenticated'] = True
st.session_state['user'] = user_info
st.session_state['user_roles'] = roles
```

### アクセス制御

各ページ表示前に権限をチェック:

```python
if not auth_service.can_access_page(user_id, page_name):
    st.error("アクセス権限がありません")
    return
```

---

## 📁 追加ファイル一覧

| ファイル | 説明 |
|---------|------|
| `migrations/add_user_auth_tables.py` | DBマイグレーション |
| `services/auth_service.py` | 認証サービス |
| `ui/pages/login_page.py` | ログイン画面 |
| `ui/pages/user_management_page.py` | ユーザー管理画面 |
| `ui/layouts/sidebar.py` | サイドバー（権限対応） |
| `main.py` | メインアプリ（認証統合） |

---

## 🛠️ カスタマイズ

### 新しいページに権限を設定

1. `migrations/add_user_auth_tables.py`の`pages`リストに追加
2. マイグレーションを再実行
3. 必要に応じてロールごとの権限を設定

### タブレベルの権限を設定

```python
# 例: 生産計画画面の「製造工程」タブを特定ロールのみに制限
tab_permissions = [
    ('生産計画', '🔧 製造工程（加工対象）')
]

for page_name, tab_name in tab_permissions:
    for role in ['管理者', '生産管理者']:
        cursor.execute('''
            INSERT OR IGNORE INTO tab_permissions (role_id, page_name, tab_name, can_view)
            SELECT id, ?, ?, 1 FROM roles WHERE role_name = ?
        ''', (page_name, tab_name, role))
```

---

## ❓ トラブルシューティング

### ログインできない

1. ユーザー名とパスワードを確認
2. MySQLで`users`テーブルに該当ユーザーが存在するか確認
   ```sql
   SELECT * FROM users WHERE username = 'admin';
   ```
3. `is_active`が1（有効）になっているか確認

### ページが表示されない

1. ユーザーにロールが割り当てられているか確認
   ```sql
   SELECT u.username, r.role_name
   FROM users u
   JOIN user_roles ur ON u.id = ur.user_id
   JOIN roles r ON ur.role_id = r.id
   WHERE u.username = 'your_username';
   ```
2. ロールにページ権限が設定されているか確認
   ```sql
   SELECT * FROM page_permissions WHERE role_id = 1;
   ```

### マイグレーションエラー

**エラーが発生した場合:**

```bash
# ロールバック（既存テーブル削除）
python migrations/add_user_auth_tables.py rollback

# 再度マイグレーション実行
python migrations/add_user_auth_tables.py
```

**MySQL接続エラーの場合:**
- `config.py`の接続情報を確認
- MySQLサーバーが起動しているか確認
- データベース`kubota_db`が存在するか確認

### 外部キー制約エラー

テーブル削除時に外部キー制約エラーが出る場合:

```sql
SET FOREIGN_KEY_CHECKS = 0;
DROP TABLE IF EXISTS tab_permissions;
DROP TABLE IF EXISTS page_permissions;
DROP TABLE IF EXISTS user_roles;
DROP TABLE IF EXISTS users;
DROP TABLE IF EXISTS roles;
SET FOREIGN_KEY_CHECKS = 1;
```

---

## 📞 サポート

問題が発生した場合は、以下を確認してください:

1. **Pythonバージョン**: 3.8以上
2. **Streamlitバージョン**: 最新版
3. **MySQLバージョン**: 5.7以上
4. **必要なライブラリ**:
   - `pymysql`
   - `sqlalchemy`
   - `streamlit`
   - `pandas`
5. **データベース接続**: MySQLサーバーが起動し、`kubota_db`が存在するか

---

## 🎉 完了

これで、ユーザー認証・権限管理機能が利用可能になりました！

次のステップ:
- [ ] 管理者パスワードを変更
- [ ] 実際のユーザーを登録
- [ ] ロールの割り当て
- [ ] 権限のテスト
