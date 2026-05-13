# 利用可能な機能一覧

本MCPサーバーが提供する機能（Tool）の一覧とその詳細です。<br>
新しい機能を今後も順次追加予定です。

## 目次

- [駅情報取得（ekispert_api_get_stations）](#駅情報取得)
- [経路探索（ekispert_api_search_routes）](#経路探索)
- [探索条件生成（ekispert_api_generate_condition）](#探索条件生成)

---

## 駅情報取得

**Tool名**: `ekispert_api_get_stations`

駅の詳細情報（駅コード、座標など）を取得します。

### 呼び出される「駅すぱあと API」

指定されたパラメータに応じて、下記のいずれかのAPIが呼び出されます。

- [駅情報 - `/station`](https://docs.ekispert.com/v1/api/station.html)
- [駅簡易情報 - `/station/light`](https://docs.ekispert.com/v1/api/station/light.html)

### パラメータ

| パラメータ | 型 | 必須 | 説明 | 例 | 実験的機能 |
|----------|-----|-----|------|-----|:------:|
| `name` | string | - | 検索する駅の名称 | `"東京"` | - |
| `code` | number | - | 検索する駅のコード | `22828` | - |
| `type` | string | - | 駅の交通種別 | `"train"` | - |
| `simplify` | string | - | `"true"` 指定時は簡略化されたレスポンスが返ります。トークン数を節約したい場合などに有効です。<br>挙動の詳細:<br>`"true"` 指定時は [`/station`](https://docs.ekispert.com/v1/api/station.html) ではなく [`/station/light`](https://docs.ekispert.com/v1/api/station/light.html) が内部的に呼び出され、対応するAPIのレスポンスが返ります。（すなわち、駅の座標情報がレスポンスで返らなくなります。）<br>指定可能な値: `"true"`, `"false"` | `"true"` (デフォルト) | ◯ |

「駅すぱあと API」で利用可能なその他のパラメータについても、今後のアップデートで追加される可能性があります。

> [!NOTE]
> 「**実験的機能**」に◯がついているパラメータは、今後のバージョンアップで仕様が大きく変わったり、パラメータ自体が廃止になる可能性があります。

**入力値の制約**
- `name`: 1〜100文字。空文字列や101文字以上はエラーになります。
- `code`: 正の整数のみ受け付けます（0や負数、小数はエラーになります）。

**注意**:
- `name`、`code`、`type` は全て任意（省略可能）です
- `name` または `code` のいずれか（または両方省略）で検索できますが、両方を同時に指定するとエラーになります
- 絞り込み条件を指定しない場合、大量の結果が返される可能性があります

#### `type` に指定可能な値

| 値 | 説明 |
|---|------|
| `train` | 鉄道 |
| `bus` | バス（すべて） |
| `bus.local` | 路線バス |
| `bus.connection` | 連絡バス |
| `bus.highway` | 高速バス |
| `bus.midnight` | 深夜急行バス |
| `plane` | 飛行機 |
| `ship` | 船 |

### レスポンス例（JSON/XML）

- **`simplify="true"` の場合**: [駅簡易情報 - `/station/light` のレスポンス例](https://docs.ekispert.com/v1/api/station/light.html#example) と同様
- **`simplify="false"` の場合**: [駅情報 - `/station` のレスポンス例](https://docs.ekispert.com/v1/api/station.html#example) と同様

### 使用例

- 自然言語での活用シナリオは [使用例](./examples.md#駅情報の取得) を参照してください。
- MCP ツールとして直接呼び出す場合の例を以下に示します。

**基本的な駅名検索（簡略化レスポンス・デフォルト）**:
```json
{
  "name": "ekispert_api_get_stations",
  "arguments": {
    "name": "東京"
  }
}
```

**駅名検索（座標情報を含む詳細レスポンス）**:
```json
{
  "name": "ekispert_api_get_stations",
  "arguments": {
    "name": "東京",
    "simplify": "false"
  }
}
```

**駅コードで検索**:
```json
{
  "name": "ekispert_api_get_stations",
  "arguments": {
    "code": 22828
  }
}
```

**交通種別を指定して検索**:
```json
{
  "name": "ekispert_api_get_stations",
  "arguments": {
    "name": "高円寺",
    "type": "bus.local"
  }
}
```

### エラー例

#### name と code を同時に指定した場合

```
Error: 'name' と 'code' は同時に指定できません。
```

#### 存在しない駅を指定した場合

「駅すぱあと API」がエラーステータスを返した場合、以下の形式でエラーが返されます:

```json
{
  "status": 400,
  "message": "駅が見つかりません。(xxxx)"
}
```

---

## 経路探索

**Tool名**: `ekispert_api_search_routes`

2地点間または複数地点を経由する経路を探索します。

### 呼び出される「駅すぱあと API」

- [経路探索 - `/search/course/extreme`](https://docs.ekispert.com/v1/api/search/course/extreme.html)

### 利用可能なパラメータ

| パラメータ | 型 | 必須 | 説明 | デフォルト値 |
|----------|-----|-----|------|-----------:|
| `viaList` | string | ○ | 出発駅・経由駅・目的駅のリスト | - |
| `date` | number | - | 探索日付（YYYYMMDD） | 現在日付 |
| `time` | string | - | 探索時刻（HHMM） | 現在時刻 |
| `searchType` | string | - | 探索種別 | `"plain"` |
| `sort` | string | - | ソート種別 | `"ekispert"` |
| `answerCount` | number | - | 最大回答数（1〜20） | `5` |
| `searchCount` | number | - | 最大探索数（1〜20） | `answerCount` と同値 |
| `resultDetail` | string | - | 結果の詳細情報 | - |
| `conditionDetail` | string | - | 詳細探索条件 | - |

「駅すぱあと API」で利用可能なその他のパラメータについても、今後のアップデートで追加される可能性があります。

**入力値の制約**
- `viaList`: 1文字以上の文字列が必須です。空文字列や未指定の場合はエラーになります。
- `date`: 19700101〜99991231 の整数のみ受け付けます。桁数が不足している場合もエラーになります。
- `time`: HHMM形式（00時00分〜26時59分）の文字列で指定してください。先頭2桁が24より大きい値は、翌日の午前0時から午前2時を指します（例: `2500` は翌日の1:00）。
  - **注意**: `searchType` が `"plain"`, `"lastTrain"`, `"firstTrain"` の場合は `time` パラメータを指定できません。エラーになります。
- `answerCount` / `searchCount`: 1〜20 の整数で指定してください。それ以外はエラーになります。
- `conditionDetail`: `ekispert_api_generate_condition` Toolで生成された文字列（`T...:F...:A...:` 形式）を指定してください。それ以外はエラーになります。
  - 固定の条件を使用する場合、`ekispert_api_generate_condition` Toolを呼び出さずに、直接生成済みの文字列を指定することも可能です。

> [!CAUTION]
> `time` パラメータは、既にお持ちのアクセスキーでもご利用可能です。
> ~~2026年7月~~ 2026年後半以降は専用のアクセスキーでのみご利用可能にする予定です。（専用アクセスキーの取得方法については、今後改めてご案内予定です。詳細は事前に告知いたします。）
> また、ダイヤ情報を利用する場合は、別途時刻情報ライセンスが必要となる場合があります。実際のサービスでご利用される場合は、当社まで [お問い合わせ](info@val.co.jp) ください。

#### `viaList` の指定方法

複数の地点をコロン（`:`）で区切って指定します。

**指定順序**:
- 先頭: 出発駅
- 末端: 目的駅
- 先頭と末端の間: 経由駅

**指定可能な形式**:
- 駅コード: `22828`
- 駅名称: `東京`
- 座標情報: `35.40.41.1,139.46.12.9,tokyo,20`
- 住所情報: `東京都杉並区高円寺北2-3-17`

**例**:
```
22828:22671:25853          # 東京→高円寺→大阪（駅コード）
東京:高円寺:大阪              # 東京→高円寺→大阪（駅名称）
35.40.41.1,139.46.12.9,tokyo,20:25853  # 座標→大阪
```

#### `searchType` に指定可能な値

| 値 | 説明 |
|---|------|
| `plain` | 平均待ち時間による探索（平均待ち時間探索）。<br>時刻表を加味しないで目的地への色々な経路を探索します。**（デフォルト）** |
| `departure` | ダイヤによる探索（発時刻探索）。早く到着し遅く出発する経路を優先して探索します。 |
| `arrival` | ダイヤによる探索（着時刻探索）。遅く出発し早く到着する経路を優先して探索します。 |
| `lastTrain` | ダイヤによる探索（終電探索）。指定された運行日の最終ダイヤの経路を優先して探索します。 |
| `firstTrain` | ダイヤによる探索（始発探索）。運行日の始発ダイヤの経路を優先して探索します。 |

**ダイヤ探索について**:
- `time` パラメータと組み合わせて使用することで、実際の出発時刻や到着時刻に基づいた正確な経路を取得できます
- `departure` または `arrival` を指定する場合、`time` パラメータで時刻を指定できます（省略時は現在時刻）
- `lastTrain` および `firstTrain` は `time` パラメータと同時指定できません
- `searchType` 省略時に `time` を指定した場合は、自動的に `departure` として探索されます

> [!CAUTION]
> ダイヤ探索の機能（`departure`, `arrival`, `lastTrain`, `firstTrain`）は、既にお持ちのアクセスキーでもご利用可能です。
> ~~2026年7月~~ 2026年後半以降は専用のアクセスキーでのみご利用可能にする予定です。（専用アクセスキーの取得方法については、今後改めてご案内予定です。詳細は事前に告知いたします。）
> また、ダイヤ情報を利用する場合は、別途時刻情報ライセンスが必要となる場合があります。実際のサービスでご利用される場合は、当社まで [お問い合わせ](info@val.co.jp) ください。

#### `sort` に指定可能な値

| 値 | 説明 |
|---|------|
| `ekispert` | 駅すぱあと探索順（デフォルト） |
| `price` | 料金順 |
| `time` | 時間順 |
| `teiki` | 定期券の料金順 |
| `transfer` | 乗換回数順 |
| `co2` | CO2 排出量順 |
| `teiki1` | 1 ヶ月定期券の料金順 |
| `teiki3` | 3 ヶ月定期券の料金順 |
| `teiki6` | 6 ヶ月定期券の料金順 |

#### `resultDetail` に指定可能な値

| 値 | 説明 |
|---|------|
| `addCorporation` | 路線に会社情報を付加 |

### レスポンス例（JSON/XML）

[経路探索 - `/search/course/extreme` のレスポンス例](https://docs.ekispert.com/v1/api/search/course/extreme.html#example) と同様

### 使用例

- 自然言語での利用例は [使用例](./examples.md#基本的な経路探索) を参照してください。
- 下記はMCPクライアントから直接ツールを呼び出す際の例です。

**基本的な経路探索（平均待ち時間探索）**:
```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "東京:新宿"
  }
}
```

**料金順でソート**:
```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "東京:新宿",
    "sort": "price"
  }
}
```

**経由駅を指定**:
```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "東京:高円寺:大阪"
  }
}
```

**日付を指定**:
```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "東京:新宿",
    "date": 20251215
  }
}
```

**回答数を増やす**:
```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "東京:新宿",
    "answerCount": 10,
    "searchCount": 15
  }
}
```

**詳細な探索条件を指定**:
```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "東京:大阪",
    "conditionDetail": "T32212332323191:F332112212000010:A23121141:"
  }
}
```

**ダイヤ探索（発時刻指定）**:
```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "東京:新宿",
    "searchType": "departure",
    "time": "0900"
  }
}
```

**ダイヤ探索（着時刻指定）**:
```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "東京:新宿",
    "searchType": "arrival",
    "time": "1800"
  }
}
```

**ダイヤ探索（終電）**:
```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "東京:新宿",
    "searchType": "lastTrain"
  }
}
```

**ダイヤ探索（始発）**:
```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "東京:新宿",
    "searchType": "firstTrain"
  }
}
```

### エラー例

#### viaList が未指定の場合

```
Error: 出発駅と到着駅を指定してください
```

#### 日付の形式が不正な場合

```
Error: 探索日付は1970年1月1日以降の日付を指定してください
```

> **補足**: 「駅すぱあと API」がエラーステータスを返した場合、`status` / `message` を含むJSON形式でエラーが返されます。

#### searchType と time の組み合わせが不正な場合

`searchType` が `plain` の場合に `time` を指定した場合:

```
Error: 'time' は 'searchType' が 'plain' の場合指定できません。
```

`searchType` が `lastTrain` または `firstTrain` の場合に `time` を指定した場合:

```
Error: 'time' は 'searchType' が 'lastTrain' の場合指定できません。
```

または

```
Error: 'time' は 'searchType' が 'firstTrain' の場合指定できません。
```

---

## 探索条件生成

**Tool名**: `ekispert_api_generate_condition`

経路探索（`ekispert_api_search_routes`）のToolで使用する、詳細な探索条件文字列を生成します。

### 呼び出される「駅すぱあと API」

- [探索条件生成 - `/toolbox/course/condition`](https://docs.ekispert.com/v1/api/toolbox/course/condition.html)

### 利用可能なパラメータ

全てのパラメータは省略可能です。<br>
「駅すぱあと API」で利用可能なその他のパラメータについても、今後のアップデートで追加される可能性があります。

#### 交通手段の利用設定

| パラメータ | 型 | 説明 | 指定可能な値 | デフォルト |
|----------|-----|------|-------------|-----------:|
| `plane` | string | 飛行機 | `light`, `normal`, `bit`, `never` | `normal` |
| `shinkansen` | string | 新幹線 | `normal`, `never` | `normal` |
| `shinkansenNozomi` | string | のぞみ・みずほ等 | `normal`, `never` | `normal` |
| `sleeperTrain` | string | 寝台列車 | `possible`, `normal`, `never` | `never` |
| `limitedExpress` | string | 有料特急 | `normal`, `never` | `normal` |
| `highwayBus` | string | 高速バス | `light`, `normal`, `bit`, `never` | `normal` |
| `connectionBus` | string | 連絡バス | `light`, `normal`, `bit`, `never` | `normal` |
| `localBus` | string | 路線バス | `normal`, `never` | `normal` |
| `communityBus` | string | コミュニティバス | `contain`, `except` | `contain` |
| `midnightBus` | string | 深夜急行バス | `normal`, `never` | `never` |
| `ship` | string | 船 | `light`, `normal`, `bit`, `never` | `normal` |
| `liner` | string | ライナー | `normal`, `never` | `normal` |

#### その他の設定

| パラメータ | 型 | 説明 | 指定可能な値 | デフォルト |
|----------|-----|------|-------------|-----------:|
| `walk` | string | 徒歩の許容度 | `normal`, `little`, `never` | `normal` |
| `waitAverageTime` | string | 平均待ち時間の考慮 | `true`, `false` | `true` |
| `transferTime` | string | 乗換時間の余裕 | `normal`, `moreMargin`, `mostMargin`, `lessMargin` | `normal` |
| `surchargeKind` | string | 料金種別 | `free`, `reserved`, `green` | `free` |
| `JRSeasonalRate` | string | ＪＲ季節料金の考慮 | `true`, `false` | `true` |
| `JRReservation` | string | EX予約/スマートEX | （下記参照） | `none` |
| `shinkansenETicket` | string | 新幹線eチケット | `none`, `eTicket` | `none` |
| `ticketSystemType` | string | 乗車券計算システム | `normal`, `ic` | `normal` |
| `preferredTicketOrder` | string | 優先する乗車券順序 | `none`, `normal`, `ic`, `cheap` | `none` |

#### `JRReservation` に指定可能な値

| 値 | 説明 |
|---|------|
| `none` | 計算しない |
| `exYoyaku` | ＥＸ予約 |
| `exETokkyu` | ＥＸ予約(ｅ特急券) |
| `exHayatoku` | ＥＸ予約(ＥＸ早特) |
| `exHayatoku1` | ＥＸ予約(ＥＸ早特１) |
| `exHayatoku21` | ＥＸ予約(ＥＸ早特２１) |
| `smartEx` | スマートＥＸ |
| `smartExHayatoku` | スマートＥＸ(ＥＸ早特) |
| `smartExHayatoku1` | スマートＥＸ(ＥＸ早特１) |
| `smartExHayatoku21` | スマートＥＸ(ＥＸ早特２１) |

### 探索種別による探索条件の扱いの違い

`ekispert_api_search_routes` で利用する探索条件は、探索種別（`searchType`）によって以下のような扱いの違いがあります。

**平均待ち時間探索でのみ有効なパラメータ**

以下のパラメータは、平均待ち時間探索の場合のみ設定が反映されます。

- `walk`
- `waitAverageTime`

**ダイヤ探索でのみ有効なパラメータ**

以下のパラメータは、ダイヤ探索の場合のみ設定が反映されます。

- `liner`
- `midnightBus`
- `transferTime`

**平均待ち時間探索でのみ詳細な指定が有効な値**

以下の交通手段設定における `light`（気軽に利用）や `bit`（極力利用しない）、`possible`（極力利用する）といった値は、平均待ち時間探索でのみ有効です。その他の探索種別では、それぞれ `normal`（利用する）や `never`（利用しない）と同様に扱われます。

- `plane` (`light`, `bit`)
- `ship` (`light`, `bit`)
- `highwayBus` (`light`, `bit`)
- `connectionBus` (`light`, `bit`)
- `sleeperTrain` (`possible`)

### レスポンス例（JSON/XML）

[探索条件生成 - `/toolbox/course/condition` のレスポンス例](https://docs.ekispert.com/v1/api/toolbox/course/condition.html#example) と同様

**返却される値の例**:

```json
"T32212332323191:F332112212000010:A23121141:"
```

### 使用例

**新幹線を利用しない条件を生成**:
```json
{
  "name": "ekispert_api_generate_condition",
  "arguments": {
    "shinkansen": "never"
  }
}
```

**ICカード優先で探索する条件を生成**:
```json
{
  "name": "ekispert_api_generate_condition",
  "arguments": {
    "ticketSystemType": "ic",
    "preferredTicketOrder": "ic"
  }
}
```

---

## 次のステップ

- [使用例](./examples.md) - 様々な経路探索パターンの実践的なサンプル
- [トラブルシューティング](./troubleshooting.md) - よくある問題と解決方法
