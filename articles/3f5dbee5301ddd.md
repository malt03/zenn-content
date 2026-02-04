---
title: "現在のパスキーは単一障害点である"
emoji: "🔐"
type: "tech"
topics: ["セキュリティ", "passkey", "macOS", "WebAuthn"]
published: true
---

## パスキーは二要素認証をスキップ

GoogleやGitHubといった多くのサービスで、パスキーでの認証時に TOTP などの二要素認証をスキップします。パスキーは単一で安全な認証として扱われているからです。
これは一見合理的に見えますが、現在のパスキー実装と組み合わさって、深刻なセキュリティホールを生んでいます。

## クラウド同期

iCloud キーチェーン、Google パスワードマネージャー、1Password、Bitwarden——現在の主要なパスキー実装は、すべてクラウド同期を前提としています。
そして、**ローカルにのみ保存するオプションは存在しません**。

## 攻撃シナリオ

1. GitHub や Google などの重要なサービスプロバイダーで 2FA を有効化
2. これらのサービスにパスキーを登録し、iCloud キーチェーンに保存
3. サービスプロバイダーはパスキー使用時に 2FA をスキップする
4. 攻撃者が Apple アカウントを乗っ取る
5. **すべてのパスキーの秘密鍵が攻撃者のデバイスに同期される**
6. 攻撃者は、あなたが設定した 2FA をスキップして全サービスにアクセス可能になる

せっかく設定した二要素認証が、Appleアカウントという単一障害点によって無効化されます。

## 根本原因

WebAuthn の仕様で、サービスプロバイダーはクラウド同期されたパスキーと、端末に閉じたパスキーを区別できます。
これにより「クラウド同期パスキーには追加認証を要求し、端末に閉じたパスキーは単一で信頼する」というリスクベース認証が可能です。
しかし、2 つの問題があります。

1. **サービスプロバイダーがフラグを無視する**：ほとんどのサービスはパスキーの種類に関係なく 2FA をスキップする
2. **ローカル専用オプションが存在しない**：Apple も主要なパスワードマネージャーも、端末に閉じたパスキーを作成する方法を提供していない

サービスプロバイダーの対応を待つしかない 1 番目の問題とは異なり、2 番目の問題はクライアント側で解決できます。

## LocalPasskey を作った

ここで急に宣伝ですが、解決する macOS アプリを作成しました。
https://github.com/malt03/local-passkey-manager
秘密鍵は [Secure Enclave](https://ja.wikipedia.org/wiki/Secure_Enclave) に保存されます。

- root 権限でも抽出できない
- 生体認証後にのみ署名に使用可能
- iCloud に同期されない

Apple アカウントが侵害されても、パスキーは安全です。

## 実装のポイント

重要なのは以下の設定です。

- `kSecAttrTokenIDSecureEnclave`：鍵を Secure Enclave 内に生成
- `kSecAttrAccessibleWhenUnlockedThisDeviceOnly`：このデバイスでのみアクセス可能（iCloud 同期を防止）
- `.privateKeyUsage, .biometryAny`：秘密鍵の使用には生体認証が必要

仮にアプリに脆弱性があったとしても、コンピューターのroot権限を取られたとしても、秘密鍵自体が漏洩することはありません。

```swift
let accessControl = SecAccessControlCreateWithFlags(
    nil,
    kSecAttrAccessibleWhenUnlockedThisDeviceOnly,
    [.privateKeyUsage, .biometryAny],
    nil
)!

let attributes: [String: Any] = [
    kSecAttrKeyType as String: kSecAttrKeyTypeECSECPrimeRandom,
    kSecAttrKeySizeInBits as String: 256,
    kSecAttrTokenID as String: kSecAttrTokenIDSecureEnclave,
    kSecPrivateKeyAttrs as String: [
        kSecAttrIsPermanent as String: true,
        kSecAttrAccessControl as String: accessControl,
        kSecAttrAccessGroup as String: accessGroup,
    ],
]

var error: Unmanaged<CFError>?
guard let privateKey = SecKeyCreateRandomKey(attributes as CFDictionary, &error) else {
    throw error!.takeRetainedValue()
}
```

## 使い方

### インストール

セキュリティを向上させるために個人開発者を信頼するのもどうかと思うので、コードを読んで自分でビルドすることをオススメします。確認すべきは[鍵生成のコード](https://github.com/malt03/local-passkey-manager/blob/v1.0.0/CredentialProvider/Sources/Registration.swift#L70-L97)だけです。鍵が抽出不可能な方法で保存されていることを確認してください。
もしも僕のことを信頼できる場合は[リリース](https://github.com/malt03/local-passkey-manager/releases)から dmg ファイルをダウンロードしてインストールも可能です。

### セットアップ

1. **システム設定** > **一般** > **自動入力とパスワード** を開く
2. 「自動入力の取得元」で **LocalPasskey** を有効化

## 既知のバグ

Apple の Credential Provider Extension 実装にはバグがあり、正確な情報をサービスプロバイダーに伝えることができません。セキュリティには影響しませんが、メタデータが不正確になっています。

- **BE/BS フラグが強制的に 1 になる**：サービスプロバイダーはデバイスに閉じているかクラウド同期かを区別できない [Forum Thread](https://developer.apple.com/forums/thread/813844)
- **AAGUID がゼロに上書きされる**：どのパスキーマネージャーが作成したかを識別できない [Forum Thread](https://developer.apple.com/forums/thread/814547)

両方とも Apple からの回答がありません。応援してくれる人は、ぜひこれらのスレッドをブーストしてください。

## ちなみに

本来、このアプリは存在する必要はありません。Apple が Passkey をローカルに保存するオプションを作ってくれればそれで終わりです。
もしこの記事とかがバズって問題視され、Apple がネイティブで提供してくれたら最高ですね。
