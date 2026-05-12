# 「駅すぱあと API MCPサーバー」公式ドキュメント

このリポジトリは、「**駅すぱあと API MCPサーバー**」（以降、「本MCPサーバー」と表記）に関する公式ドキュメントです。
（本MCPサーバーのプログラム自体は非公開です。）

## 「駅すぱあと API MCPサーバー」とは

- 「[駅すぱあと API](https://docs.ekispert.com/v1/)」をAIエージェント経由で利用可能にするMCPサーバーです
- MCPサーバーに接続可能な任意の生成AIツール（Visual Studio Code、Claude Desktop、など）で利用可能です

## 特徴

- AIエージェント経由で（すなわち、自然言語で）「駅すぱあと API」を呼び出せるようになります
- [MCP (Model Context Protocol)](https://modelcontextprotocol.io/) という標準的な規格に対応しているため、AIエージェントへの組み込みも容易です

## 本MCPサーバーの仕様

- **MCP仕様準拠**: 
  - MCPの公式の仕様に準拠、および最新バージョンへの追従
- **通信方式**: 
  - Streamable HTTP
- **認証・認可方式**: 
  - 「駅すぱあと API」のアクセスキーを用いた認証・認可
  - ※ ~~2026/07/01（仮）以降~~ **2026年後半以降** に変更する可能性があります。変更する場合には事前に告知させていただきます。

## ご利用にあたって

本MCPサーバーの利用には、「駅すぱあと API」のアクセスキーが必要です。<br>
現在は、以下のいずれかの方法でご利用いただけます。

- 「駅すぱあと API」を利用中の方: 既にお持ちのアクセスキーで利用可能
- 「駅すぱあと API」を初めて使う方: [90日無料評価版](https://api-info.ekispert.com/form/trial/) のアクセスキーでお試し可能

> [!WARNING]
> MCPサーバー経由で「駅すぱあと API」へのアクセスが発生した場合には、アクセスキー毎にアクセス数および使用料が通常通り加算されます。

~~2026/07/01（仮）以降~~ **2026年後半以降** に、本MCPサーバーを正式にご利用いただくためのアクセスキー、および、その発行手続きについては、現在準備中です。

## :rocket: クイックスタート

<details><summary>VS Code (Visual Studio Code) の場合</summary>

`mcp.json` の設定例:

```json
{
  "servers": {
    "ekispert-api-mcp-server": {
      "type": "http",
      "url": "https://api-mcp.ekispert.jp/mcp",
      "headers": {
        "ekispert-api-access-key": "${env:EKISPERT_API_ACCESS_KEY}"
      }
    }
  }
}
```

</details>

<details><summary>Claude Code の場合</summary>

`.claude.json` の設定例:

```json
{
  "mcpServers": {
    "ekispert-api-mcp-server": {
      "type": "http",
      "url": "https://api-mcp.ekispert.jp/mcp",
      "headers": {
        "ekispert-api-access-key": "${EKISPERT_API_ACCESS_KEY}"
      }
    }
  }
}
```

</details>

<details><summary>Claude Desktop の場合</summary>

`claude_desktop_config.json` の設定例:

```json
{
  "mcpServers": {
    "ekispert-api-mcp-server": {
      "command": "npx",
      "args": [
        "-y",
        "mcp-remote@latest",
        "https://api-mcp.ekispert.jp/mcp",
        "--header",
        "ekispert-api-access-key:${EKISPERT_API_ACCESS_KEY}"
      ],
      "env": {
        "EKISPERT_API_ACCESS_KEY": "YOUR_ACCESS_KEY_HERE"
      }
    }
  }
}
```

</details>

<br>

> [!TIP]
> `ekispert-api-access-key` には「駅すぱあと API」のアクセスキーを設定してください。<br>
> セキュリティの観点から、アクセスキーの値は直接設定するのではなく、環境変数（例: `EKISPERT_API_ACCESS_KEY` ）から取得する方法を推奨します。

> [!TIP]
> 詳細な設定方法については、各生成AIツールの公式ドキュメントをご確認ください。<br>
> また、上記以外の生成AIツールを使う場合には、上記設定例を適宜読み替えてください。

本MCPサーバーに接続後、AIエージェントと以下のようなやりとりができるようになります。

:speech_balloon: プロンプトの例:

```
新宿駅から品川駅への最も安いIC運賃の経路を教えて
```

:robot: AIエージェントからの回答例:

```
新宿駅から品川駅への最も安いIC運賃の経路は以下の通りです:

新宿 → ＪＲ山手線内回り → 品川
IC運賃: 208円 (普通乗車券は210円)
所要時間: 約23分(乗車19分+その他4分)
乗換回数: 0回
```

## 次のステップ

- [詳細なセットアップガイド](./docs/getting-started.md)
- [利用可能な機能一覧](./docs/features.md)
- [使用例（様々なプロンプトの例）](./docs/examples.md)
- [トラブルシューティング](./docs/troubleshooting.md)

## 試してみる・学ぶ

- [「駅すぱあと API MCPサーバー」ハンズオン](https://github.com/ValLaboratory/ekispert-api-mcp-server-handson)
- [「駅すぱあと API MCPサーバー」利用サンプル集](https://github.com/ValLaboratory/ekispert-api-mcp-server-samples)

## 利用規約

本MCPサーバーは、「駅すぱあと API」に付随するサービスであり、「駅すぱあと API」の利用規約に準じます。

- [利用規約（日本語版）](https://docs.ekispert.com/v1/WebService_TOS.pdf)
- [利用規約（英語版）](https://docs.ekispert.com/v1/WebService_TOS_en.pdf)
