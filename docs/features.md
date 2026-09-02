# 利用可能な機能一覧

本MCPサーバーが提供する機能（Tool）の一覧とその詳細です。<br>
新しい機能を今後も順次追加予定です。

## 目次

- [駅情報取得（ekispert_api_get_stations）](#駅情報取得)
- [住所からの周辺駅検索（ekispert_api_get_stations_from_address）](#住所からの周辺駅検索)
- [経路探索（ekispert_api_search_routes）](#経路探索)
- [探索条件生成（ekispert_api_generate_condition）](#探索条件生成)
- [範囲探索（ekispert_api_search_ranges）](#範囲探索)

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
| `gcs` | string | - | レスポンスに含まれる座標の測地系。<br>指定可能な値: `"tokyo"`（日本測地系）, `"wgs84"`（世界測地系） | `"wgs84"` (デフォルト) | - |

「駅すぱあと API」で利用可能なその他のパラメータについても、今後のアップデートで追加される可能性があります。

> [!NOTE]
> 「**実験的機能**」に◯がついているパラメータは、今後のバージョンアップで仕様が大きく変わったり、パラメータ自体が廃止になる可能性があります。

**入力値の制約**
- `name`: 1〜100文字。空文字列や101文字以上はエラーになります。
- `code`: 正の整数のみ受け付けます（0や負数、小数はエラーになります）。
- `gcs`: `simplify` が `"true"`（デフォルト）の場合は指定できません。エラーになります。座標情報を取得する場合は `simplify="false"` を合わせて指定してください。

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

**日本測地系の座標を取得**:
```json
{
  "name": "ekispert_api_get_stations",
  "arguments": {
    "name": "東京",
    "simplify": "false",
    "gcs": "tokyo"
  }
}
```

### エラー例

#### name と code を同時に指定した場合

```
Error: 'name' と 'code' は同時に指定できません。
```

#### gcs と simplify="true" を同時に指定した場合

`simplify` を省略した場合もデフォルト値の `"true"` が適用されるため、同様にエラーになります。

```
Error: 'gcs' は 'simplify' が 'true'（デフォルト）の場合指定できません。座標情報を取得する場合は simplify='false' を指定してください。
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

## 住所からの周辺駅検索

**Tool名**: `ekispert_api_get_stations_from_address`

指定した住所の周辺にある駅の情報（駅コード、座標、住所からの距離など）を取得します。

### 呼び出される「駅すぱあと API」

- [住所情報からの周辺駅検索 - `/address/station`](https://docs.ekispert.com/v1/api/address/station.html)

### 利用可能なパラメータ

| パラメータ | 型 | 必須 | 説明 | デフォルト値 |
|----------|-----|-----|------|-----------:|
| `address` | string | ○ | 検索する住所。都道府県から開始し、少なくとも町・字まで含める必要があります。 | - |
| `radius` | number | - | 検索半径（単位: m）。指定できる値: 1〜10000の整数。省略時は最寄り駅1件のみを返します。 | - |
| `types` | string[] | - | 検索する駅の交通種別。複数指定可能です。 | 全種別 |
| `stationCount` | number | - | 検索結果の最大回答数。0以上の整数。 | `10` |
| `gcs` | string | - | レスポンスに含まれる座標の測地系。`"tokyo"`（日本測地系）または `"wgs84"`（世界測地系）。 | `"wgs84"` |
| `communityBus` | string | - | 取得結果にコミュニティバスを含めるかどうか。`"contain"`（含める）または `"except"`（含めない）。`types` を省略した場合、または `types` に `bus` もしくは `bus.local` を含む場合のみ有効です。 | `"contain"` |

**入力値の制約**
- `address`: 1文字以上の文字列が必須です。空文字列や未指定の場合はエラーになります。
- `radius`: 1〜10000 の整数のみ受け付けます。0や小数、10001以上はエラーになります。
- `types`: 指定する場合は1件以上の配列で指定してください。空配列や、指定できない種別を含む場合はエラーになります。
- `stationCount`: 0以上の整数のみ受け付けます。負数や小数はエラーになります。
  - `radius` を省略した場合は最寄り駅1件のみが返却され、`stationCount` の指定は無視されます。
  - `0` を指定した場合は検索結果全件が返却されます。
- `gcs`: `"tokyo"`, `"wgs84"` のいずれかを指定してください。それ以外はエラーになります。
- `communityBus`: `"contain"`, `"except"` のいずれかを指定してください。それ以外はエラーになります。
  - `types` を省略した場合、または `types` に `bus` もしくは `bus.local` を含む場合のみ効果があります。それ以外の `types` を指定した場合は無視されます。

#### `types` に指定可能な値

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

> **補足**: 経路探索時に長い待ち時間が発生するなど、最寄り駅を使った探索がうまくいかない場合は、`communityBus: "except"` を指定してコミュニティバスを除外し、最寄り駅を再検索することも有効です。

### レスポンス例（JSON/XML）

[住所情報からの周辺駅検索 - `/address/station` のレスポンス例](https://docs.ekispert.com/v1/api/address/station.html#example)

### 使用例

- 自然言語での活用シナリオは [使用例](./examples.md#住所からの周辺駅検索) を参照してください。
- 下記はMCPクライアントから直接ツールを呼び出す際の例です。

**住所から最寄り駅を検索**:
```json
{
  "name": "ekispert_api_get_stations_from_address",
  "arguments": {
    "address": "東京都杉並区高円寺北"
  }
}
```

**検索半径と交通種別を指定**:
```json
{
  "name": "ekispert_api_get_stations_from_address",
  "arguments": {
    "address": "大阪府大阪市北区梅田",
    "radius": 1000,
    "types": ["train", "bus.local"]
  }
}
```

**検索結果の件数と測地系を指定**:
```json
{
  "name": "ekispert_api_get_stations_from_address",
  "arguments": {
    "address": "東京都杉並区高円寺北",
    "radius": 2000,
    "stationCount": 5,
    "gcs": "tokyo"
  }
}
```

**コミュニティバスを除外して検索**:
```json
{
  "name": "ekispert_api_get_stations_from_address",
  "arguments": {
    "address": "東京都杉並区高円寺北",
    "radius": 1000,
    "communityBus": "except"
  }
}
```

### エラー例

#### address が未指定の場合

```
Error: 住所の入力は必須です
```

#### 存在しない住所や解釈できない住所の場合

「駅すぱあと API」がエラーステータスを返した場合、以下の形式でエラーが返されます:

```json
{
  "status": 400,
  "message": "住所が存在しないか、解釈できない住所です。(xxxx)"
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
| `gcs` | string | - | 座標の測地系（`"tokyo"`: 日本測地系, `"wgs84"`: 世界測地系） | `"wgs84"` |

「駅すぱあと API」で利用可能なその他のパラメータについても、今後のアップデートで追加される可能性があります。

**入力値の制約**
- `viaList`: 1文字以上の文字列が必須です。空文字列や未指定の場合はエラーになります。
- `date`: 19700101〜99991231 の整数のみ受け付けます。桁数が不足している場合もエラーになります。
- `time`: HHMM形式（00時00分〜26時59分）の文字列で指定してください。先頭2桁が24より大きい値は、翌日の午前0時から午前2時を指します（例: `2500` は翌日の1:00）。
  - **注意**: `searchType` が `"plain"`, `"lastTrain"`, `"firstTrain"` の場合は `time` パラメータを指定できません。エラーになります。
- `answerCount` / `searchCount`: 1〜20 の整数で指定してください。それ以外はエラーになります。
- `conditionDetail`: `ekispert_api_generate_condition` Toolで生成された文字列（`T...:F...:A...:` 形式）を指定してください。それ以外はエラーになります。
  - 固定の条件を使用する場合、`ekispert_api_generate_condition` Toolを呼び出さずに、直接生成済みの文字列を指定することも可能です。
- `gcs`: `"tokyo"`, `"wgs84"` のいずれかを指定してください。それ以外はエラーになります。

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
- 座標情報: `35.40.41.1,139.46.12.9,tokyo,20`（度分秒・十進法度のどちらでも指定可）
- 住所情報: `東京都杉並区高円寺北2-3-17`

**例**:
```
22828:22671:25853          # 東京→高円寺→大阪（駅コード）
東京:高円寺:大阪              # 東京→高円寺→大阪（駅名称）
35.40.41.1,139.46.12.9,tokyo,20:25853  # 座標→大阪（度分秒）
35.681236,139.767125,wgs84:25853       # 座標→大阪（十進法度）
```

**座標情報の測地系について**:

座標情報では、`35.40.41.1,139.46.12.9,tokyo,20` のように地点ごとに測地系を個別指定できます。<br>
測地系を省略した場合（例: `35.40.41.1,139.46.12.9`）は、`gcs` パラメータで指定した値（デフォルト: `"wgs84"`）が採用されます。

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

**世界測地系の座標を出発地に指定**:

`viaList` の座標で測地系を省略しているため、`gcs` の値（`"wgs84"`）が採用されます。

```json
{
  "name": "ekispert_api_search_routes",
  "arguments": {
    "viaList": "35.681236,139.767125:大阪",
    "gcs": "wgs84"
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

## 範囲探索

**Tool名**: `ekispert_api_search_ranges`

起点駅から所要時間・乗換回数などの条件内で到達できる駅を探索します。起点駅を複数指定した場合は、すべての起点駅から到達できる共通の駅が返ります。

### 呼び出される「駅すぱあと API」

- [範囲探索 - `/search/multipleRange`](https://docs.ekispert.com/v1/api/search/multipleRange.html)

### 利用可能なパラメータ

| パラメータ | 型 | 必須 | 説明 | デフォルト値 |
|----------|-----|-----|------|-----------:|
| `baseList` | string[] | ○ | 起点駅のリスト（1〜5件） | - |
| `upperMinutes` | number[] | ○ | 起点駅ごとの所要時間の上限（分、10〜200） | - |
| `upperTransferCounts` | number[] | - | 起点駅ごとの乗換回数の上限（-1以上） | 考慮しない |
| `plane` | string | - | 飛行機の利用設定 | `"true"` |
| `shinkansen` | string | - | 新幹線（のぞみを含む）の利用設定 | `"true"` |
| `limitedExpress` | string | - | 有料特急の利用設定 | `"true"` |
| `waitAverageTime` | string | - | 出発駅での平均的な乗車待ち時間の設定 | `"true"` |
| `includeBaseStation` | string | - | 探索結果に起点駅を含めるかの設定 | `"true"`（起点駅2件以上の場合） |
| `limit` | number | - | 探索結果の最大件数 | 全件 |
| `date` | number | - | 探索日付（YYYYMMDD） | 現在日付 |

**入力値の制約**
- `baseList`: 1〜5件の配列で指定します。交通種別が鉄道（`train`）の駅のみ指定できます。
  - 各要素には駅コード（`22671`）または駅の名称（`新宿`）を指定します。
  - **1つの要素に複数の起点駅をコロン（`:`）区切りでまとめることはできません。** 必ず配列の別々の要素として指定してください。
- `upperMinutes`: 10〜200の整数で指定します。`baseList` と同じ要素数が必要です。
- `upperTransferCounts`: 省略可能です。指定する場合は `baseList` と同じ要素数が必要です。
- `includeBaseStation`: `baseList` が2件以上の場合のみ指定できます。1件の場合に指定するとエラーになります。
- `limit`: 1以上の整数で指定します。
- `date`: 19700101〜99991231 の整数のみ受け付けます。桁数が不足している場合もエラーになります。

**配列の対応関係**

`baseList` / `upperMinutes` / `upperTransferCounts` は、配列の同じ位置の要素同士が対応します。例えば `baseList: ["新宿", "横浜"]`、`upperMinutes: [20, 40]` の場合、新宿からは20分以内、横浜からは40分以内が条件になります。

#### `upperTransferCounts` の指定方法

乗換回数の上限は起点駅ごとに個別に指定できます。上限を設けたくない起点駅には `-1` を指定します。

| 指定値 | 意味 |
|---|------|
| `0` 以上の整数 | その回数以内に制限する（`0` は「乗り換えなし」） |
| `-1` | 乗換回数を考慮しない（無制限） |

**例**:
```
[1, 2]      # 1件目は乗換1回以内、2件目は乗換2回以内
[1, -1]     # 1件目は乗換1回以内、2件目は無制限
[-1, -1]    # 両方とも無制限（パラメータ自体を省略した場合と同じ）
```

**注意**: 「乗り換え回数は気にしない」という条件を表す場合は、`0` ではなく `-1` を指定してください。`0` は「乗り換えなし」という別の条件になります。

#### `includeBaseStation` の挙動

探索結果に起点駅自体を含めるかを指定します。複数の起点駅を指定した待ち合わせ場所の検討で、起点駅そのものが候補になり得るかを制御できます。

| 値 | 説明 |
|---|------|
| `"true"` | 他のすべての起点駅からの探索条件を満たす起点駅を結果に含めます。**（デフォルト）** |
| `"false"` | 起点駅を結果に含めません。 |

起点駅が1件の場合は指定できず、起点駅は結果に含まれません。

#### `limit` の挙動

探索結果の最大件数を指定します。省略した場合は全件が対象です。

指定した場合、起点駅からの所要時間が短い駅から順に採用されます。複数の起点駅を指定した場合は、各到達駅について、起点駅ごとの所要時間のうち最も短いもの（以下「代表所要時間」）を比較に使用します。

**例**: 2つの起点駅からA駅までが2分・10分、B駅までが5分・5分、C駅までが8分・3分の場合、代表所要時間はA駅=2分、B駅=5分、C駅=3分です。`limit` が2の場合、代表所要時間が短いA駅とC駅が採用されます。

> [!NOTE]
> 代表所要時間が同じ駅が複数ある場合、どの駅が `limit` の範囲内に採用されるかは「駅すぱあと API」内部の順序規則によって決まります。同じ条件であれば結果は常に同じですが、この順序は公開していません。すべての駅が必要な場合は、`limit` を指定せずに全件を取得してください。

所要時間の上限を大きくすると、探索結果が数百〜数千件になることがあります。一覧の網羅性が不要な場合は `limit` を指定することで、応答サイズを抑えられます。

### レスポンス例（JSON/XML）

- [範囲探索 - `/search/multipleRange` のレスポンス例](https://docs.ekispert.com/v1/api/search/multipleRange.html#example)

探索結果の各駅には、起点駅ごとの所要時間・乗換回数が `Cost` として含まれます。`baseIndex` は `baseList` の何番目の起点駅に対応するかを表します（1始まり）。

### 使用例

- 自然言語での活用シナリオは [使用例](./examples.md#範囲探索) を参照してください。
- 下記はMCPクライアントから直接ツールを呼び出す際の例です。

**単一の起点駅から到達できる駅を探索**:
```json
{
  "name": "ekispert_api_search_ranges",
  "arguments": {
    "baseList": ["新宿"],
    "upperMinutes": [30]
  }
}
```

**複数の起点駅から到達できる共通の駅を探索**:
```json
{
  "name": "ekispert_api_search_ranges",
  "arguments": {
    "baseList": ["新宿", "横浜"],
    "upperMinutes": [30, 30]
  }
}
```

**起点駅ごとに異なる条件を指定**:
```json
{
  "name": "ekispert_api_search_ranges",
  "arguments": {
    "baseList": ["新宿", "渋谷"],
    "upperMinutes": [30, 45],
    "upperTransferCounts": [0, -1]
  }
}
```

**新幹線・有料特急を使わずに探索し、結果を100件に絞る**:
```json
{
  "name": "ekispert_api_search_ranges",
  "arguments": {
    "baseList": ["東京"],
    "upperMinutes": [60],
    "shinkansen": "false",
    "limitedExpress": "false",
    "limit": 100
  }
}
```

### エラー例

#### baseList と upperMinutes の要素数が一致しない場合

```
Error: 'baseList' と 'upperMinutes' の要素数を一致させてください。
```

#### upperTransferCounts と baseList の要素数が一致しない場合

```
Error: 'upperTransferCounts' を指定する場合、'baseList' と要素数を一致させてください。
```

#### 単一の起点駅で includeBaseStation を指定した場合

```
Error: 'includeBaseStation' は 'baseList' が2件以上の場合のみ指定できます。
```

#### baseList の要素にコロン（`:`）を含めた場合

```
Error: 起点駅に ':' は使用できません。複数の起点駅を指定する場合は、配列の別々の要素として指定してください（例: ['新宿', '横浜']）
```

> **補足**: 「駅すぱあと API」がエラーステータスを返した場合、`status` / `message` を含むJSON形式でエラーが返されます。

---

## 次のステップ

- [使用例](./examples.md) - 様々な経路探索パターンの実践的なサンプル
- [トラブルシューティング](./troubleshooting.md) - よくある問題と解決方法
