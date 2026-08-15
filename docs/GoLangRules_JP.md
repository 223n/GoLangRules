# Go開発のルール(GoLangRules_JP)

## 0. 本規約で使用されている略称

| 略称  | 意味         |
| :---- | :----------- |
| Ex.   | 例です       |
| Ref.  | 参考資料です |
| Note. | 参考情報です |

## 1. 本規約について

このルールは、基本的にGo言語での命名ルールなどをまとめたものです。

細かい点については、自身のルールに作りかえて利用していただければうれしく思います。

各ルールの背景や理由、より詳細な例は、
[Go開発のルール詳細解説(GoLangRules_JP_Detail)](GoLangRules_JP_Detail.md)に
まとめています。

## 2. Goコーディング関係のガイドラインについて

Go公式やGoogle社が、基本的なGoのコーディング規約などを公開しています。

本ルールは、これらをベースに記載していますので、ご一読ください。

- Ref. [Effective Go][link1]

- Ref. [Go Code Review Comments][link2]

- Ref. [Google Go Style Guide][link3]

- Ref. [Go Doc Comments][link4]

## 3. 基本的なルール

### 3.1. 公開する識別子（エクスポート）

パッケージ外へ公開する型、関数、メソッド、定数、変数は、Pascal形式で命名します。

Go言語では、名前の先頭を大文字にすると、その識別子はパッケージ外へ公開されます。

```go
// 公開する構造体
type LoginPage struct {

    // 公開するフィールド
    UserID string

}

// 公開するメソッド
func (p *LoginPage) Login() error {
    return nil
}
```

### 3.2. 公開しない識別子（アンエクスポート）

パッケージ内でのみ使用する型、関数、変数、フィールドは、Camel形式で命名します。

Go言語では、名前の先頭を小文字にすると、その識別子はパッケージ内に限定されます。

```go
// 公開しないフィールド
type loginPage struct {
    userID string
}

// 関数のパラメーター
func sampleFunc(userName string) {

    // 関数内の変数
    var message string

    _ = message
}
```

- Note.
  - Go言語では、C#のような`private`や`public`のキーワードはありません。
  - 名前の先頭が大文字か小文字かで、公開範囲が決まります。
  - `_`（アンダースコア）を含む名前（スネーク形式）は使用しません。

### 3.3. 頭字語（略語）の表記

`ID`や`URL`、`HTTP`などの頭字語は、大文字または小文字で統一して記述します。

`Id`や`Url`のように、先頭のみ大文字にする表記は使用しません。

```go
// 良い例
var userID string
var apiURL string

type HTTPClient struct{}

// 悪い例
var userId string
var apiUrl string

type HttpClient struct{}
```

- Note.
  - 公開しない識別子の先頭に頭字語がくる場合は、`id`や`url`のようにすべて小文字にします。

## 4. 命名の規則

### 4.1. パッケージ

パッケージは、すべて小文字の短い1単語で命名します。

複数形や`_`（アンダースコア）、Camel形式は使用しません。

また、`util`や`common`、`misc`のような、内容が伝わらない名前は避けます。

| 要素             | 規則                     | Ex.              |
| :--------------- | :----------------------- | :--------------- |
| パッケージ       | すべて小文字の1単語      | http, json, user |
| テストパッケージ | （パッケージ名） + _test | user_test        |

```go
// 良い例
package employee

// 悪い例
package employeeUtils
```

- Note.
  - テストパッケージ名の`_test`は、ブラックボックステストを行う場合の例外です。

### 4.2. インターフェース

インターフェースは、Pascal形式（公開する場合）で命名します。

メソッドを1つだけ持つインターフェースは、「メソッド名 + er」の形式で命名します。

C#のような`I`のプレフィックス（接頭辞）は付けません。

| 要素           | 規則             | Ex.            |
| :------------- | :--------------- | :------------- |
| メソッドが1つ  | （メソッド名）er | Reader, Writer |
| メソッドが複数 | 名詞、名詞句     | ReadWriter     |

```go
// 良い例
type Reader interface {
    Read(p []byte) (n int, err error)
}

// 悪い例
type IReader interface {
    Read(p []byte) (n int, err error)
}
```

### 4.3. 関数、メソッド

関数やメソッドは、動詞もしくは動詞句で命名します。

下表の関数、メソッドについては、適切なサフィックス（接尾辞）、プレフィックス（接頭辞）を付けます。

| 要素                 | 規則                    | Ex.           |
| :------------------- | :---------------------- | :------------ |
| 戻り値がbool型       | Is + 形容詞             | IsEdited      |
|                      | Can + 動詞              | CanEdit       |
|                      | Has + 名詞              | HasError      |
| ファクトリー関数     | New + オブジェクト名    | NewEmployee   |
| 初期化               | Init + 初期化名         | InitHandler   |
| テスト関数           | Test + テスト対象名     | TestLogin     |

- Note.
  - ゲッターには`Get`のプレフィックスを付けません。
    フィールド`owner`のゲッターは、`GetOwner`ではなく`Owner`と命名します。
  - セッターには`Set`のプレフィックスを付けます（`SetOwner`など）。

```go
// 良い例（ゲッターとセッター）
func (e *Employee) Owner() string {
    return e.owner
}

func (e *Employee) SetOwner(owner string) {
    e.owner = owner
}
```

### 4.4. 変数

変数は、Camel形式（公開する場合はPascal形式）で命名します。

スコープが狭い変数には短い名前を、スコープが広い変数には説明的な名前を使用します。

| 要素               | 規則         | Ex.       |
| :----------------- | :----------- | :-------- |
| ループ変数         | 1文字        | i, j, k   |
| bool型             | is + 形容詞  | isEdited  |
|                    | can + 動詞   | canEdit   |
|                    | has + 名詞   | hasError  |
| スコープが広い変数 | 名詞、名詞句 | userCount |

```go
// 良い例（スコープに応じた名前の長さ）
for i, u := range users {
    fmt.Println(i, u.Name)
}

var defaultTimeoutSeconds = 30
```

### 4.5. 定数

定数は、Camel形式（公開する場合はPascal形式）で命名します。

C#や他言語で見られる、すべて大文字のスネーク形式（`MAX_LENGTH`など）は使用しません。

```go
// 良い例
const maxLength = 100
const DefaultPort = 8080

// 悪い例
const MAX_LENGTH = 100
```

### 4.6. 構造体

構造体は、名詞、名詞句で命名します。

構造体名にはパッケージ名の情報を繰り返さないようにします。

```go
// 良い例（employee.Personで参照される）
package employee

type Person struct{}

// 悪い例（employee.EmployeePersonと冗長になる）
type EmployeePerson struct{}
```

### 4.7. レシーバー

メソッドのレシーバーは、型名の頭文字などを使用した1文字から2文字の短い名前で命名します。

`this`や`self`は使用しません。

また、同じ型のメソッドでは、レシーバー名を統一します。

```go
// 良い例
func (e *Employee) Name() string {
    return e.name
}

// 悪い例
func (this *Employee) Name() string {
    return this.name
}
```

### 4.8. エラー

エラー変数には`Err`のプレフィックス（接頭辞）を付けます。

独自のエラー型には`Error`のサフィックス（接尾辞）を付けます。

| 要素       | 規則         | Ex.             |
| :--------- | :----------- | :-------------- |
| エラー変数 | Err + 名詞   | ErrNotFound     |
| エラー型   | 名詞 + Error | ValidationError |

```go
// エラー変数
var ErrNotFound = errors.New("employee: not found")

// エラー型
type ValidationError struct {
    Field string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("validation failed: %s", e.Field)
}
```

- Note.
  - エラーメッセージは小文字で始め、末尾に句読点を付けません。
  - エラーメッセージの先頭には、パッケージ名を付けることを推奨します。

### 4.9. ファイル名

ファイル名は、すべて小文字で命名します。

複数の単語をつなげる場合は、`_`（アンダースコア）で区切ります。

| 要素             | 規則                   | Ex.              |
| :--------------- | :--------------------- | :--------------- |
| 通常のファイル   | 小文字 + .go           | employee.go      |
| テストファイル   | （対象名） + _test.go  | employee_test.go |
| OS固有のファイル | （対象名） + _（OS名） | server_linux.go  |

## 5. コーディング時に意識すること

### 5.1. gofmtで整形する

ソースコードは、必ず`gofmt`（または`goimports`）で整形します。

Go言語では、公式ツールによって整形ルールが統一されているため、
インデントや括弧の位置などをチームで議論する必要はありません。

コミット前に必ず実行するか、エディターの保存時整形を有効にしておきます。

### 5.2. 1関数あたり30行以内におさめる

1関数あたりの行数は、30行以内におさまるように目指します。

30行を超える場合は、`5.3.`以降を参考に、おさまるようリファクタリングを行います。

ただし、`switch`を使用している場合は例外です。

### 5.3. 1関数は1機能のみ

1つの関数に複数の機能を持たせると、再利用性が低くなる原因となります。

たとえば、行追加の関数（`AddRow`）内で、行追加と保存の処理を行うなどです。

この場合、行追加と保存の処理は異なる関数に分けておき、それぞれを呼び出します。

### 5.4. 不要なコードは削除する

ソースコードをGitなどで適切に管理している場合、
不要なソースコードやコメントは削除すべきです。

不要なソースコードやコメントは、可読性を低くする原因となります。

- Note.
  - Go言語では、未使用の変数やインポートはコンパイルエラーになります。

### 5.5. 早期リターンでネスト構造を浅くする

ネスト構造は浅くするべきです。

Go言語では、エラーを先に処理して早期リターンする「ガード節」のスタイルが推奨されています。

正常系の処理がネストの浅い位置に並ぶため、可読性が高くなります。

```go
// 良い例
func Process(val string) error {
    if val == "" {
        return ErrEmptyValue
    }

    if !isValid(val) {
        return ErrInvalidValue
    }

    return save(val)
}

// 悪い例
func Process(val string) error {
    if val != "" {
        if isValid(val) {
            return save(val)
        }
        return ErrInvalidValue
    }
    return ErrEmptyValue
}
```

### 5.6. エラーは握りつぶさない

関数が返すエラーは、必ず処理します。

`_`（ブランク識別子）でエラーを破棄すると、障害時の調査が困難になります。

エラーを上位へ返す場合は、`fmt.Errorf`と`%w`で文脈を付けてラップします。

```go
// 良い例
data, err := os.ReadFile(path)
if err != nil {
    return fmt.Errorf("設定ファイルの読み込みに失敗: %w", err)
}

// 悪い例
data, _ := os.ReadFile(path)
```

- Note.
  - ラップしたエラーは、`errors.Is`や`errors.As`で判定できます。

### 5.7. panicは使用しない

通常のエラー処理には、`panic`ではなく`error`の戻り値を使用します。

`panic`は、プログラムの継続が不可能な初期化エラーなど、
回復できない状況に限定して使用します。

### 5.8. アクセス範囲は厳格に

識別子の公開範囲は、できる限り厳格にします。

パッケージ外から使用しない型や関数は、小文字始まりにして公開しません。

これにより、意図しない外部からの呼び出しや値の変更などを防げます。

また、公開する識別子が少ないほど、パッケージの利用者が
目的の型や関数を早く見つけられるようになります。

### 5.9. contextは第一引数で受け取る

キャンセル処理やタイムアウトを扱う`context.Context`は、
関数の第一引数`ctx`として受け取ります。

構造体のフィールドには保持しません。

```go
// 良い例
func FetchEmployee(ctx context.Context, id string) (*Employee, error) {
    return nil, nil
}
```

### 5.10. goroutineの終了条件を明確にする

goroutineを起動する場合は、終了する条件と手段を必ず用意します。

終了しないgoroutineは、メモリリーク（goroutineリーク）の原因となります。

`context.Context`によるキャンセルや、チャネルのクローズなどで終了を通知します。

### 5.11. ログなどで文字列を結合する場合には、strings.Builderを使う

ログなど、既存の文字列の後ろに文字列を追加していく場合、`string`同士を`+`では結合せず、
`strings.Builder`の`WriteString`メソッドなどを用いて結合します。

なお、短い文字列を組み立てる場合は、`fmt.Sprintf`の使用を推奨します。

```go
var b strings.Builder
for _, line := range lines {
    b.WriteString(line)
}
log.Print(b.String())
```

### 5.12. 公開する識別子にはドキュメントコメントを書く

公開する型、関数、メソッド、定数には、ドキュメントコメントを書きます。

コメントは、対象の名前で始まる完全な文で記述します。

```go
// Employee は従業員の情報を表します。
type Employee struct{}

// NewEmployee は新しいEmployeeを生成します。
func NewEmployee(name string) *Employee {
    return &Employee{}
}
```

## 6. 静的解析とテストを行う

### 6.1. go vet

`go vet`は、コンパイルは通るものの、バグの可能性が高いコードを検出します。

チェックイン（コミット）前に、必ず実行して警告の確認を行います。

```bash
go vet ./...
```

### 6.2. staticcheck、golangci-lint

`go vet`よりも詳細な解析を行う場合は、
`staticcheck`や`golangci-lint`などの静的解析ツールを使用します。

警告が無くなるように実装することで、一定のルールに従った実装ができます。

```bash
golangci-lint run
```

### 6.3. テスト

テストは、標準の`testing`パッケージを使用して記述します。

テストファイルは、テスト対象と同じディレクトリーに`_test.go`の名前で配置します。

チェックイン（コミット）前に、必ず実行してすべてのテストが通ることを確認します。

```bash
go test ./...
```

- Note.
  - 入力と期待値の組み合わせが多い場合は、テーブル駆動テストを推奨します。

[link1]: https://go.dev/doc/effective_go "Effective Go"
[link2]: https://go.dev/wiki/CodeReviewComments "Go Code Review Comments"
[link3]: https://google.github.io/styleguide/go/ "Google Go Style Guide"
[link4]: https://go.dev/doc/comment "Go Doc Comments"
