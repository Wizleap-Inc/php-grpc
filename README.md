# gRPC & Protobuf PHP Extensions - Pre-built Binaries

PHP DockerでgRPCとProtobufをビルドする時間を短縮するため、事前ビルド済みの拡張バイナリを配布するリポジトリです。

## サポート環境

- **OS**: Linux (x86_64, aarch64), macOS (x86_64, arm64)
- **PHP**: 8.0, 8.1, 8.2, 8.3, 8.4

## 使い方

### Dockerfileでの使用例

```dockerfile
FROM php:8.3-cli

# gRPC & Protobuf拡張をダウンロード
ADD https://github.com/wizleap/grpc-php/releases/latest/download/grpc-php8.3-linux-x86_64.so /usr/local/lib/php/extensions/no-debug-non-zts-20230831/grpc.so
ADD https://github.com/wizleap/grpc-php/releases/latest/download/protobuf-php8.3-linux-x86_64.so /usr/local/lib/php/extensions/no-debug-non-zts-20230831/protobuf.so

# 拡張を有効化
RUN docker-php-ext-enable grpc protobuf

# 確認
RUN php -m | grep -E '(grpc|protobuf)'
```

### 手動インストール

1. [Releases](https://github.com/wizleap/grpc-php/releases)から対応するバイナリをダウンロード

```bash
# 例: PHP 8.3, Linux x86_64
wget https://github.com/wizleap/grpc-php/releases/latest/download/grpc-php8.3-linux-x86_64.so
wget https://github.com/wizleap/grpc-php/releases/latest/download/protobuf-php8.3-linux-x86_64.so
```

2. PHP拡張ディレクトリにコピー

```bash
# 拡張ディレクトリの確認
php-config --extension-dir

# コピー
sudo cp grpc-php8.3-linux-x86_64.so $(php-config --extension-dir)/grpc.so
sudo cp protobuf-php8.3-linux-x86_64.so $(php-config --extension-dir)/protobuf.so
```

3. php.iniで拡張を有効化

```ini
extension=protobuf.so
extension=grpc.so
```

4. 確認

```bash
php -m | grep -E '(grpc|protobuf)'
```

## ビルド時間の比較

### ビルド前（従来の方法）
```dockerfile
FROM php:8.3-cli
RUN pecl install grpc      # ⏱️ 5-10分かかる
RUN pecl install protobuf  # ⏱️ 3-5分かかる
# 合計: 8-15分
```

### ビルド後（このリポジトリを使用）
```dockerfile
FROM php:8.3-cli
ADD https://github.com/wizleap/grpc-php/releases/latest/download/grpc-php8.3-linux-x86_64.so \
    /usr/local/lib/php/extensions/no-debug-non-zts-20230831/grpc.so
ADD https://github.com/wizleap/grpc-php/releases/latest/download/protobuf-php8.3-linux-x86_64.so \
    /usr/local/lib/php/extensions/no-debug-non-zts-20230831/protobuf.so
RUN docker-php-ext-enable grpc protobuf  # ⚡ 数秒で完了
```

## ファイル名の形式

### gRPC拡張
```
grpc-php{PHPバージョン}-{OS}-{アーキテクチャ}.so
```

### Protobuf拡張
```
protobuf-php{PHPバージョン}-{OS}-{アーキテクチャ}.so
```

例:
- `grpc-php8.3-linux-x86_64.so` - PHP 8.3, Linux x86_64用 gRPC拡張
- `protobuf-php8.3-linux-x86_64.so` - PHP 8.3, Linux x86_64用 Protobuf拡張
- `grpc-php8.2-macos-arm64.so` - PHP 8.2, macOS ARM64(Apple Silicon)用 gRPC拡張
- `protobuf-php8.2-macos-arm64.so` - PHP 8.2, macOS ARM64(Apple Silicon)用 Protobuf拡張

## バージョン管理

- このリポジトリのタグ(例: `v1.68.0`)はビルドするgRPCのバージョンに対応
- Protobuf拡張は常に最新の安定版がビルドされます
- 各リリースには複数のPHP/プラットフォーム向けバイナリが含まれる

## 開発

### 手動ビルドの実行

```bash
# 特定のgRPCバージョンをビルド
# GitHub Actionsの "Run workflow" から手動実行可能
```

### 新しいリリースの作成

```bash
# タグをプッシュすると自動的にビルド・リリース
git tag v1.68.0
git push origin v1.68.0
```

## ライセンス

Apache License 2.0 (gRPCと同じ)

## トラブルシューティング

### "cannot open shared object file" エラー

PHPのextension directoryが正しくない可能性があります。

```bash
# 正しいパスを確認
php-config --extension-dir

# Dockerの場合、PHPバージョンに応じてパスが変わります
# PHP 8.3: /usr/local/lib/php/extensions/no-debug-non-zts-20230831/
# PHP 8.2: /usr/local/lib/php/extensions/no-debug-non-zts-20220829/
# PHP 8.1: /usr/local/lib/php/extensions/no-debug-non-zts-20210902/
# PHP 8.0: /usr/local/lib/php/extensions/no-debug-non-zts-20200930/
```

### アーキテクチャの確認

```bash
# Linux
uname -m

# macOS
uname -m

# x86_64 または aarch64/arm64 が表示される
```
