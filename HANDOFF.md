# ゲストポータル プロジェクト引き継ぎ

## プロジェクト概要
西条農業高校のゲスト（他校生徒・見学者）向けファイル共有ポータルサイト。
ログイン不要でブラウザからアクセスするだけで、資料ダウンロード・ファイルアップロード・アンケート回答ができる。

## 技術スタック
- **フロントエンド**: React（CDN経由、Babel standalone）、単一の index.html ファイル
- **ホスティング**: GitHub Pages（リポジトリ名: `guest-portal-site`）
- **バックエンド**: Firebase（Storage + Cloud Firestore）
- **編集環境**: GitHub Codespaces

## 現在の状態

### 完了済み
- index.html の作成済み（React + Firebase SDK 組み込み済み）
- GitHub リポジトリ作成済み・GitHub Pages 有効化済み
- Firebase プロジェクト作成済み（プロジェクト名: `guest-portal-site`）
- Firebase Billing（Blaze プラン）有効化済み

### 未完了（これからやること）
1. **Firebase の設定値を index.html に反映する**
   - Firebase Console → プロジェクト設定 → マイアプリ → SDK の設定と構成
   - index.html 内の `firebase.initializeApp({...})` の6つの値を実際の値に書き換え

2. **Cloud Firestore を有効化してセキュリティルールを設定する**
   - ロケーション: asia-northeast1（東京）
   - ルール:
   ```
   rules_version = '2';
   service cloud.firestore {
     match /databases/{database}/documents {
       match /feedback/{doc} {
         allow create: if request.resource.data.keys().hasAll(['event', 'timestamp'])
                       && request.resource.data.size() <= 10;
         allow read, update, delete: if false;
       }
       match /{document=**} {
         allow read, write: if false;
       }
     }
   }
   ```

3. **Firebase Storage を有効化してセキュリティルールを設定する**
   - ロケーション: asia-northeast1（東京）
   - ルール:
   ```
   rules_version = '2';
   service firebase.storage {
     match /b/{bucket}/o {
       match /events/{eventId}/{fileName} {
         allow read: if true;
         allow write: if false;
       }
       match /uploads/{eventId}/{fileName} {
         allow create: if request.resource.size < 10 * 1024 * 1024;
         allow read, update, delete: if false;
       }
       match /{allPaths=**} {
         allow read, write: if false;
       }
     }
   }
   ```

4. **Firebase Storage に配布資料をアップロード**
   - フォルダ構造: `events/dx-workshop-2025/` の下にファイルを配置
   - index.html の `EVENT_CONFIG.downloads` の path と一致させる

5. **動作確認**
   - GitHub Pages の URL でサイトが表示されるか
   - アンケート送信 → Firestore の feedback コレクションにデータが入るか
   - ファイルアップロード → Storage の uploads/ にファイルが保存されるか
   - 資料ダウンロード → Storage の events/ からダウンロードできるか

## ファイル構成
```
guest-portal-site/
  └── index.html   ← これ1ファイルのみ
```

## index.html の構造
- Firebase SDK（CDN）+ firebase.initializeApp() ← 設定値を要書き換え
- React / ReactDOM / Babel（CDN）
- EVENT_CONFIG オブジェクト ← イベントごとに書き換える部分
- GuestPortal コンポーネント（4タブ: 案内・資料・提出・アンケート）
- handleFileUpload → storage.ref().put() で Firebase Storage へ
- handleDownload → storage.ref().getDownloadURL() で取得
- handleFeedbackSubmit → db.collection("feedback").add() で Firestore へ

## 運用時の変更箇所
イベントを切り替えるときは `EVENT_CONFIG` の以下を変更:
- title, subtitle, date, description
- schedule（タイムテーブル）
- downloads（配布資料リスト。path は Storage 上のパスと一致させる）
- feedbackQuestions（アンケート項目）
