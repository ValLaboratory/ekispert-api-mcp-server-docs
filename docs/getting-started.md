# セットアップガイド

このガイドでは、本MCPサーバーの詳細なセットアップ手順を説明します。

## 基本設定

<details><summary>VS Code (Visual Studio Code) の場合</summary>

`mcp.json` の設定例:

```json
{
  "servers": {
    "ekispert-api-mcp-server": {
      "type": "http",
      "url": "https://api-mcp.ekispert.jp/mcp",
      "headers": {
        "ekispert-api-access-key": "${env:EKISPERT_API_ACCESS_KEY}",
        "ekispert-api-response-format": "json"
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
        "ekispert-api-access-key": "${EKISPERT_API_ACCESS_KEY}",
        "ekispert-api-response-format": "json"
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
        "ekispert-api-access-key:${EKISPERT_API_ACCESS_KEY}",
        "--header",
        "ekispert-api-response-format:json",
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

**共通設定**

- `ekispert-api-access-key` には、駅すぱあと APIのアクセスキーを設定してください。
  - セキュリティの観点から、アクセスキーの値は直接設定するのではなく、環境変数（例: `EKISPERT_API_ACCESS_KEY` ）から取得する方法を推奨します。
- `ekispert-api-response-format` には `json` または `xml` のいずれかが指定可能です（省略時は `json` ）。

> [!TIP]
> 詳細な設定方法については、各生成AIツールの公式ドキュメントをご確認ください。<br>
> また、上記以外の生成AIツールを使う場合には、上記設定例を適宜読み変えてください。

## 接続確認

MCPサーバーが正常に接続されているか確認するには、生成AIツールで以下のようなプロンプトを試してください:

```
東京駅から新宿駅への経路を教えて
```

**期待される動作**:
- MCPサーバーが駅すぱあと APIに問い合わせ
- 経路情報（所要時間、運賃など）が返される
- チャット履歴で `ekispert-api-mcp-server` のToolが実行されたことを確認できる

**注意**: 簡単な質問をした場合には、本MCPサーバーが呼び出されずにAIエージェントが直接回答してしまう場合もあります。

## トラブルシューティング

設定や起動時に問題が発生した場合は、[トラブルシューティング](./troubleshooting.md) を参照してください。

## 次のステップ

- [利用可能な機能一覧](./features.md) - MCPサーバーが提供する機能の詳細
- [使用例](./examples.md) - 様々な経路探索パターンのサンプル
