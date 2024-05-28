# Qiita の記事を書く

## 環境構築

```bash
npm install
npx qiita login
```

token は [https://qiita.com/settings/tokens/new?read_qiita=1&write_qiita=1&description=qiita-cli]　で取得

## アップデート

```bash
npm install @qiita/qiita-cli@latest
```

## ローカルサーバー

```bash
npx qiita preview
```

## 記事作成

```bash
npx qiita new 記事のファイルのベース名
```

## 記事公開

```bash
npx qiita publish 記事のファイルのベース名
and
npx qiita publish --all
```
