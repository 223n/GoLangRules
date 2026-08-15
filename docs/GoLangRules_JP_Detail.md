# Go開発のルール詳細解説(GoLangRules_JP_Detail)

## 0. 本ドキュメントで使用されている略称

| 略称  | 意味         |
| :---- | :----------- |
| Ex.   | 例です       |
| Ref.  | 参考資料です |
| Note. | 参考情報です |

## 1. 本ドキュメントについて

このドキュメントは、[Go開発のルール(GoLangRules_JP)](GoLangRules_JP.md)の
各ルールについて、背景や理由、より詳細な例をまとめたものです。

章番号と節番号は、本編と対応しています。

たとえば、本編の`3.1.`の詳細は、本ドキュメントの`3.1.`に記載しています。

ルールの一覧を確認したい場合は本編を、
ルールの理由や具体例を確認したい場合は本ドキュメントを参照してください。

## 2. Goコーディング関係のガイドラインについて

本編で参照している各ガイドラインの位置づけは、次のとおりです。

### 2.1. Effective Go

Go公式が公開している、Goらしいコードを書くための解説です。

命名、制御構造、データ構造、並行処理など、言語全体を幅広く扱っています。

- Ref. [Effective Go][link1]

### 2.2. Go Code Review Comments

Go公式のWikiで公開されている、コードレビューでよく指摘される項目集です。

頭字語の表記、レシーバー名、エラー文字列の書き方など、
本規約の多くのルールは、このドキュメントをベースにしています。

- Ref. [Go Code Review Comments][link2]

### 2.3. Google Go Style Guide

Google社が社内向けに定めたGoのスタイルガイドを、一般公開したものです。

判断に迷った際の指針（Style DecisionsやBest Practices）が充実しています。

- Ref. [Google Go Style Guide][link3]

### 2.4. Go Doc Comments

ドキュメントコメント（godocやpkg.go.devで表示されるコメント）の、
公式な書き方の解説です。

- Ref. [Go Doc Comments][link4]

## 3. 基本的なルール

### 3.1. 公開する識別子（エクスポート）

#### ルール

パッケージ外へ公開する型、関数、メソッド、定数、変数は、
Pascal形式で命名します。

#### 理由

Go言語には、`public`や`private`のようなアクセス修飾子がありません。

その代わり、識別子の先頭が大文字であれば公開、小文字であれば非公開と、
名前そのものが公開範囲を表す仕様になっています。

つまり、先頭を大文字にすることは、
「この識別子はパッケージの利用者向けのAPIである」という宣言を意味します。

#### 解説

公開した識別子は、パッケージの利用者から直接参照されます。

一度公開するとパッケージの互換性を保つために変更が難しくなるため、
公開する識別子は必要最小限にとどめます。

```go
package employee

// Employee は従業員の情報を表します。
type Employee struct {
    // Name は公開されるため、利用者が直接読み書きできます。
    Name string

    // salaryは非公開のため、パッケージ内からのみ参照できます。
    salary int
}

// Salary は給与を返します。
func (e *Employee) Salary() int {
    return e.salary
}
```

- Note.
  - 公開する識別子には、ドキュメントコメントを書きます（`5.12.`参照）。

### 3.2. 公開しない識別子（アンエクスポート）

#### ルール

パッケージ内でのみ使用する型、関数、変数、フィールドは、
Camel形式で命名します。

#### 理由

先頭を小文字にすることで、識別子はパッケージ内に限定されます。

非公開にしておけば、パッケージの内部実装を後から自由に変更できます。

#### 解説

`_`（アンダースコア）を含むスネーク形式は、
公開、非公開にかかわらず使用しません。

Go言語の標準ライブラリーやコミュニティー全体が、
MixedCaps（大文字と小文字による単語の区切り）で統一されているためです。

```go
// 良い例
var maxRetryCount = 3

// 悪い例（スネーク形式）
var max_retry_count = 3
```

- Note.
  - 関数のパラメーターや戻り値の名前も、Camel形式で命名します。

### 3.3. 頭字語（略語）の表記

#### ルール

`ID`や`URL`、`HTTP`などの頭字語は、
大文字または小文字で統一して記述します。

#### 理由

Go Code Review Commentsで、頭字語は一貫した大文字小文字で書くよう
定められています。

標準ライブラリーも、`http.StatusOK`や`url.URL`など、
この表記で統一されています。

`Id`や`Url`のような表記は、標準ライブラリーと混在した際に
一貫性が失われるため使用しません。

#### 解説

主な頭字語の表記例は、下表のとおりです。

| 頭字語 | 公開する場合 | 公開しない場合 |
| :----- | :----------- | :------------- |
| ID     | UserID       | userID         |
| URL    | BaseURL      | baseURL        |
| HTTP   | HTTPClient   | httpClient     |
| API    | APIKey       | apiKey         |
| JSON   | JSONData     | jsonData       |
| DB     | DBConn       | dbConn         |

- Note.
  - 公開しない識別子の先頭に頭字語がくる場合は、
    `id`や`url`のようにすべて小文字にします。
  - `xmlHTTPRequest`のように、頭字語が連続しても表記は変えません。

## 4. 命名の規則

### 4.1. パッケージ

#### ルール

パッケージは、すべて小文字の短い1単語で命名します。

#### 理由

パッケージ名は、利用者が書くコードの一部（`employee.New`など）になります。

短く明確な名前であるほど、呼び出し側のコードが読みやすくなります。

#### 解説

`util`や`common`、`misc`のような名前は、
どんな機能でも入れられてしまい、パッケージの責務が曖昧になります。

機能を表す具体的な名前を付けられないときは、
パッケージの分割単位そのものを見直します。

```go
// 良い例（呼び出し側が読みやすい）
emp := employee.New("Yamada")

// 悪い例（何のNewなのか伝わらない）
emp := util.New("Yamada")
```

- Note.
  - 標準ライブラリーでは、複数単語をつなげる場合でも
    `strconv`（string conversion）のように短縮して1単語にしています。
  - ディレクトリー名は、パッケージ名と一致させます。

### 4.2. インターフェース

#### ルール

メソッドを1つだけ持つインターフェースは、
「メソッド名 + er」の形式で命名します。

#### 理由

`Read`を持つから`Reader`、`Write`を持つから`Writer`と、
名前を見ただけで何ができる型なのかが伝わるためです。

標準ライブラリーの`io.Reader`や`fmt.Stringer`など、
Go全体で定着している慣習です。

#### 解説

C#の`I`プレフィックスを付けないのは、
Goでは「インターフェースかどうか」よりも
「何ができるか」を名前で表すことを重視するためです。

また、インターフェースは小さく保ちます。

メソッドが少ないほど実装が容易になり、
モック作成などテストでも扱いやすくなります。

```go
// 良い例（メソッド1つの小さなインターフェース）
type Notifier interface {
    Notify(ctx context.Context, message string) error
}

// 悪い例（大きすぎるインターフェース）
type EmployeeManager interface {
    Create(name string) error
    Update(id string, name string) error
    Delete(id string) error
    Find(id string) (*Employee, error)
    Export() ([]byte, error)
}
```

- Note.
  - 「インターフェースは利用する側のパッケージで定義する」のも
    Goの重要な慣習です。
    実装を提供する側は具体型を返し、
    利用する側が必要なメソッドだけを持つインターフェースを定義します。

### 4.3. 関数、メソッド

#### ルール

関数やメソッドは、動詞もしくは動詞句で命名します。

ゲッターには`Get`のプレフィックスを付けません。

#### 理由

Effective Goで、ゲッターに`Get`を付けないことが明示されています。

フィールド`owner`に対して`Owner()`と命名すれば、
`GetOwner()`と書かなくても取得することは明らかであり、
呼び出し側のコードも短くなります。

#### 解説

ファクトリー関数は、C#の`Create...`ではなく`New...`で命名します。

パッケージの代表的な型を生成する関数は、
`NewEmployee`ではなく単に`New`と命名します。

パッケージ名と合わせて`employee.New`となり、意味が通じるためです。

```go
package employee

// New は新しいEmployeeを生成します。
// 呼び出し側では employee.New("Yamada") となります。
func New(name string) *Employee {
    return &Employee{name: name}
}
```

bool型を返す関数の命名例は、次のとおりです。

```go
// Is + 形容詞
func (e *Employee) IsRetired() bool

// Can + 動詞
func (e *Employee) CanEdit() bool

// Has + 名詞
func (e *Employee) HasError() bool
```

- Note.
  - テスト関数は`Test`で始めないと、`go test`が認識しません。
  - ベンチマーク関数は`Benchmark`、Example関数は`Example`で始めます。

### 4.4. 変数

#### ルール

スコープが狭い変数には短い名前を、
スコープが広い変数には説明的な名前を使用します。

#### 理由

数行で使い終わるループ変数に長い名前を付けても、情報量は増えません。

逆に、パッケージ全体で使う変数の名前が短いと、
参照箇所で意味を思い出せなくなります。

「宣言から使用箇所までの距離」に応じて名前の長さを決めるのが、
Goの慣習です。

#### 解説

Goには、慣習として定着している短縮名があります。

| 変数            | 慣習的な名前 |
| :-------------- | :----------- |
| ループ変数      | i, j, k      |
| エラー          | err          |
| context.Context | ctx          |
| 存在チェック    | ok           |
| バッファー      | buf          |
| リクエスト      | req, r       |
| レスポンス      | resp, w      |

```go
// 良い例（狭いスコープでは短く）
for i, e := range employees {
    fmt.Println(i, e.Name)
}

// 良い例（広いスコープでは説明的に）
var defaultRetryInterval = 10 * time.Second
```

- Note.
  - `ok`は、マップの存在チェックや型アサーションの
    2番目の戻り値として使用します（`v, ok := m[key]`）。

### 4.5. 定数

#### ルール

定数は、Camel形式（公開する場合はPascal形式）で命名します。

すべて大文字のスネーク形式（`MAX_LENGTH`など）は使用しません。

#### 理由

Goでは、定数も変数と同じ命名規則に従います。

先頭の大文字小文字が公開範囲を表す仕様のため、
「大文字＝定数」という他言語の慣習は成り立ちません。

#### 解説

連続する定数を定義する場合は、`iota`を使用します。

```go
// 良い例（iotaによる連番定数）
type Status int

const (
    StatusActive Status = iota
    StatusInactive
    StatusRetired
)

// 悪い例（スネーク形式）
const STATUS_ACTIVE = 0
```

- Note.
  - Goには列挙型（enum）がないため、
    独自の型と`iota`の組み合わせで代用するのが慣習です。

### 4.6. 構造体

#### ルール

構造体名にはパッケージ名の情報を繰り返さないようにします。

#### 理由

構造体は、常に`employee.Person`のようにパッケージ名を付けて参照されます。

構造体名にもパッケージ名を含めると、
`employee.EmployeePerson`のように同じ情報が重複（スタッター）します。

#### 解説

パッケージ名と型名は、組み合わせて読んだときに
自然な意味になるよう命名します。

```go
// 良い例（employee.Personと自然に読める）
package employee

type Person struct {
    Name string
}

// 悪い例（employee.EmployeePersonと冗長になる）
type EmployeePerson struct {
    Name string
}
```

- Note.
  - 同じ理由で、関数名にもパッケージ名を繰り返しません。
    `yaml.ParseYAML`ではなく`yaml.Parse`と命名します。

### 4.7. レシーバー

#### ルール

レシーバーは、型名の頭文字などを使用した
1文字から2文字の短い名前で命名します。

`this`や`self`は使用しません。

#### 理由

レシーバー名は、メソッド内で何度も登場します。

短い名前で統一しておくことで、コードの密度が上がり読みやすくなります。

`this`や`self`を使用しないのは、
Goではレシーバーが「特別なキーワード」ではなく
「単なる第1パラメーター」であるためです。

#### 解説

同じ型のメソッドでは、レシーバー名を必ず統一します。

メソッドによって`e`だったり`emp`だったりすると、
コードを読む際の負荷が上がります。

```go
// 良い例（すべてのメソッドでeに統一）
func (e *Employee) Name() string  { return e.name }
func (e *Employee) IsRetired() bool { return e.retired }

// 悪い例（メソッドごとに名前が異なる）
func (e *Employee) Name() string    { return e.name }
func (emp *Employee) IsRetired() bool { return emp.retired }
```

- Note.
  - レシーバーを値にするかポインターにするかも重要な設計判断です。
    フィールドを変更する場合や構造体が大きい場合はポインターを、
    それ以外は値を使用します。
    ただし、同じ型の中で値とポインターを混在させないようにします。

### 4.8. エラー

#### ルール

エラー変数には`Err`のプレフィックスを付けます。

独自のエラー型には`Error`のサフィックスを付けます。

#### 理由

`Err`で始まる変数は「`errors.Is`で比較できる番兵エラー」、
`Error`で終わる型は「`errors.As`で取り出せるエラー型」と、
名前を見ただけで使い方が判別できるためです。

標準ライブラリーの`io.EOF`を除き、
`os.ErrNotExist`や`sql.ErrNoRows`などで定着している慣習です。

#### 解説

番兵エラーは、呼び出し側が「特定の失敗」を判定したい場合に定義します。

```go
package employee

// ErrNotFound は従業員が見つからない場合のエラーです。
var ErrNotFound = errors.New("employee: not found")

// 呼び出し側
emp, err := employee.Find(id)
if errors.Is(err, employee.ErrNotFound) {
    // 見つからない場合の処理
}
```

エラーに付加情報を持たせたい場合は、エラー型を定義します。

```go
// ValidationError は入力チェックのエラーです。
type ValidationError struct {
    Field string
}

func (e *ValidationError) Error() string {
    return fmt.Sprintf("employee: validation failed: %s", e.Field)
}

// 呼び出し側
var vErr *ValidationError
if errors.As(err, &vErr) {
    fmt.Println("エラーの項目:", vErr.Field)
}
```

エラーメッセージは、次のルールで記述します。

- 小文字で始めます（他のメッセージと連結されるため）。
- 末尾に句読点を付けません。
- 先頭にパッケージ名を付けることを推奨します。

- Note.
  - 呼び出し側で判定する必要がないエラーは、
    変数や型を定義せず`fmt.Errorf`で生成して返します。

### 4.9. ファイル名

#### ルール

ファイル名は、すべて小文字で命名します。

複数の単語をつなげる場合は、`_`（アンダースコア）で区切ります。

#### 理由

大文字を含むファイル名は、
OSによって大文字小文字の扱いが異なるため、トラブルの原因となります。

また、Goのビルドツールは、
`_test.go`や`_linux.go`などのサフィックスを特別に解釈します。

#### 解説

OS名やアーキテクチャー名のサフィックスを持つファイルは、
その環境でのみビルドされます（ビルド制約）。

| ファイル名        | 扱い                      |
| :---------------- | :------------------------ |
| employee.go       | 通常のファイル            |
| employee_test.go  | go test時のみビルドされる |
| server_linux.go   | Linuxでのみビルドされる   |
| server_windows.go | Windowsでのみビルドされる |

- Note.
  - 意図せず`_linux`などで終わる名前を付けると、
    特定のOSでしかビルドされなくなるため注意が必要です。

## 5. コーディング時に意識すること

### 5.1. gofmtで整形する

#### 理由

Go言語は「フォーマットの議論に時間を使わない」という設計思想を持ち、
公式ツールの`gofmt`が唯一の整形ルールです。

チームや個人の好みに合わせた整形ルールを作らないことで、
どのGoプロジェクトを見ても同じスタイルで読めるようになっています。

#### 解説

`goimports`は、`gofmt`の整形に加えて、
インポート文の追加と削除も自動で行います。

エディターの保存時整形を有効にしておくのが最も確実です。

CIでは、次のようなコマンドで整形漏れを検出できます。

```bash
# 整形が必要なファイルがあれば一覧を表示する
gofmt -l .

# goimportsの場合
goimports -l .
```

- Note.
  - インデントにタブを使用するかスペースを使用するかも、
    `gofmt`が決めます（タブに統一されます）。

### 5.2. 1関数あたり30行以内におさめる

#### 理由

行数が多い関数は、複数の機能を持っている可能性が高く、
テストや再利用が難しくなります。

30行は、エディターの1画面内で処理の全体を把握できる目安です。

#### 解説

30行を超えそうな場合は、次の順で見直します。

1. 早期リターンでネストを解消できないか確認します（`5.5.`参照）。
2. 複数の機能が混ざっていないか確認します（`5.3.`参照）。
3. 機能ごとに関数を分割します。

ただし、`switch`で分岐を列挙している場合や、
テーブル駆動テストのテストケース定義などは例外です。

### 5.3. 1関数は1機能のみ

#### 理由

1つの関数に複数の機能を持たせると、
一部の機能だけを再利用できず、テストの組み合わせも増えます。

#### 解説

「関数名を素直に付けられるか」が良い分割の目安です。

行追加と保存を行う関数に`AddRowAndSave`のような名前を
付けたくなったら、分割のサインです。

```go
// 良い例（機能ごとに分割し、呼び出し側で組み合わせる）
func (t *Table) AddRow(row Row) error { /* 行追加のみ */ }
func (t *Table) Save() error          { /* 保存のみ */ }

// 悪い例（行追加と保存が密結合になっている）
func (t *Table) AddRowAndSave(row Row) error {
    // 行追加と保存を同時に行う
    return nil
}
```

### 5.4. 不要なコードは削除する

#### 理由

コメントアウトされた古いコードや使われない関数は、
「まだ必要かもしれない」という判断コストを読み手に強いるためです。

Gitで管理していれば、削除したコードはいつでも履歴から復元できます。

#### 解説

Go言語は、言語仕様とツールの両面で不要コードの検出を支援しています。

- 未使用のローカル変数とインポートは、コンパイルエラーになります。
- 未使用のパッケージレベルの変数や関数は、
  `staticcheck`などの静的解析で検出できます（`6.2.`参照）。

- Note.
  - 一時的に変数を未使用のままにしたい場合は、
    `_ = v`のようにブランク識別子へ代入します。

### 5.5. 早期リターンでネスト構造を浅くする

#### 理由

Goのコードは「正常系を左端に揃える」スタイルが推奨されています。

エラーや例外的な条件を先に処理して抜けることで、
関数の主目的である正常系の流れが、上から下へ一直線に読めます。

#### 解説

`if err != nil { return err }`のパターンを積み重ねるのが基本形です。

`else`は、早期リターンで置き換えられる場合が多く、
使用する前に一度見直します。

```go
// 良い例（異常系を先に抜け、正常系が左端に揃う）
func Register(e *Employee) error {
    if e == nil {
        return ErrNilEmployee
    }

    if err := e.Validate(); err != nil {
        return fmt.Errorf("register: %w", err)
    }

    return save(e)
}

// 悪い例（正常系がネストの奥に沈む）
func Register(e *Employee) error {
    if e != nil {
        if err := e.Validate(); err == nil {
            return save(e)
        } else {
            return err
        }
    }
    return ErrNilEmployee
}
```

### 5.6. エラーは握りつぶさない

#### 理由

Goのエラーは例外と異なり、無視しても自動的には伝わりません。

`_`で破棄されたエラーは完全に消滅し、
障害が起きた際に原因へたどり着けなくなります。

#### 解説

エラーを上位へ返す場合は、`fmt.Errorf`と`%w`で文脈を追加します。

「どの処理の途中で失敗したか」を各層で積み重ねることで、
最終的なログから失敗の経路を追跡できます。

```go
data, err := os.ReadFile(path)
if err != nil {
    return fmt.Errorf("load config %s: %w", path, err)
}
```

`%w`でラップしたエラーは、元のエラーの情報を保持しているため、
上位層で`errors.Is`や`errors.As`による判定ができます。

- Note.
  - `%v`でラップすると元のエラーの情報が失われ、
    `errors.Is`での判定ができなくなります。
    呼び出し側に内部実装のエラーを見せたくない場合を除き、
    `%w`を使用します。
  - ログ出力とエラーの返却を同時に行うと、
    同じエラーが多重に記録されます。どちらか一方にします。

### 5.7. panicは使用しない

#### 理由

`panic`は呼び出し側に回復手段を強制する仕組みであり、
Goでは「エラーは値として返し、呼び出し側が判断する」のが原則です。

ライブラリーが`panic`すると、
利用者のプログラム全体が停止する恐れがあります。

#### 解説

`panic`が許容されるのは、次のような場合に限られます。

- プログラムの継続が不可能な初期化エラー
  （必須の設定ファイルが読めないなど）
- 「呼び出し側のバグ」を早期に検出したい場合
  （標準ライブラリーの`regexp.MustCompile`など）

```go
// 許容される例（パッケージ初期化時の正規表現コンパイル）
var validID = regexp.MustCompile(`^[a-z]+[0-9]+$`)
```

- Note.
  - `Must`プレフィックスは、
    「失敗時にpanicする関数」を表すGoの慣習です。

### 5.8. アクセス範囲は厳格に

#### 理由

公開した識別子は、パッケージの互換性を保つ義務が生じます。

非公開であれば、内部実装をいつでも自由に変更できます。

#### 解説

迷ったら非公開で始めます。

非公開を公開に変えることは容易ですが、
公開してしまった識別子を非公開に戻すことは、破壊的変更になります。

- Note.
  - `internal`ディレクトリーに置いたパッケージは、
    その親ディレクトリー配下からしかインポートできません。
    リポジトリー外へ公開したくないパッケージは、
    `internal`配下に置きます。

### 5.9. contextは第一引数で受け取る

#### 理由

`context.Context`は、キャンセルやタイムアウトの伝播を担うため、
呼び出しの連鎖に沿ってリクエストごとに引き渡す必要があります。

第一引数`ctx`に統一する慣習によって、
「この関数はキャンセル可能である」ことが一目で分かります。

#### 解説

構造体のフィールドに保持しないのは、
`context`が「1回の呼び出し」に紐づくものであり、
「オブジェクトの寿命」に紐づくものではないためです。

```go
// 良い例
func (r *Repository) FindEmployee(
    ctx context.Context, id string) (*Employee, error) {
    return r.queryOne(ctx, id)
}

// 悪い例（構造体にcontextを保持している）
type Repository struct {
    ctx context.Context
}
```

- Note.
  - `nil`のcontextは渡しません。
    適切なcontextが決まらない場合は`context.TODO()`を渡します。

### 5.10. goroutineの終了条件を明確にする

#### 理由

終了しないgoroutineは回収されず、
メモリとリソースをリークし続けます（goroutineリーク）。

リークは即座に障害とならないため、
長期間稼働するサーバーで徐々に蓄積し、発見が遅れがちです。

#### 解説

goroutineを起動する際は、
「いつ、どうやって終了するか」を必ず設計します。

代表的な終了手段は、次のとおりです。

- `context.Context`のキャンセルを`ctx.Done()`で受け取る
- 入力チャネルのクローズをrangeループの終了で受け取る

```go
// ctx.Done()で終了できるワーカーの例
func worker(ctx context.Context, jobs <-chan Job) {
    for {
        select {
        case <-ctx.Done():
            return
        case job, ok := <-jobs:
            if !ok {
                return
            }
            process(job)
        }
    }
}
```

- Note.
  - 起動したgoroutineの完了を待つ場合は、
    `sync.WaitGroup`や`errgroup.Group`を使用します。
  - Go 1.25以降であれば、`testing/synctest`パッケージで
    goroutineリークを含む並行処理のテストがしやすくなっています。

### 5.11. ログなどで文字列を結合する場合には、strings.Builderを使う

#### 理由

Goの`string`は不変であるため、`+`で結合するたびに
新しい文字列のためのメモリ確保とコピーが発生します。

ループ内での`+`結合は、結合回数が増えるほど性能が劣化します。

#### 解説

`strings.Builder`は内部バッファーへ追記していくため、
メモリ確保の回数を大幅に減らせます。

最終的なサイズの見当がつく場合は、
`Grow`であらかじめバッファーを確保しておくと、さらに効率的です。

```go
var b strings.Builder
b.Grow(1024)
for _, line := range lines {
    b.WriteString(line)
    b.WriteString("\n")
}
log.Print(b.String())
```

- Note.
  - 数個の値を1回で組み立てる場合は、
    `fmt.Sprintf`の方が読みやすく、性能差も問題になりません。
  - スライスの要素を区切り文字で結合する場合は、
    `strings.Join`が最も簡潔です。

### 5.12. 公開する識別子にはドキュメントコメントを書く

#### 理由

公開する識別子のコメントは、
godocやpkg.go.devでAPIドキュメントとして表示されます。

コメントを書くだけで、ドキュメントの生成と公開が完結します。

#### 解説

ドキュメントコメントは、次のルールで記述します。

- 対象の直前に、空行を挟まず記述します。
- 対象の名前で始まる完全な文で記述します。
- パッケージにも`package`宣言の直前にコメントを書きます。

```go
// Package employee は従業員情報の管理機能を提供します。
package employee

// MaxNameLength は名前の最大文字数です。
const MaxNameLength = 50

// Employee は従業員の情報を表します。
type Employee struct{}

// New は名前を指定して新しいEmployeeを生成します。
// 名前が空の場合はエラーを返します。
func New(name string) (*Employee, error) {
    return &Employee{}, nil
}
```

- Note.
  - 非推奨にしたい識別子には、
    `// Deprecated:`で始まる行をコメントに追加します。
    静的解析ツールやエディターが警告を表示するようになります。

## 6. 静的解析とテストを行う

### 6.1. go vet

#### 解説

`go vet`は、コンパイルは通るものの、
バグの可能性が高いコードを検出する公式ツールです。

検出できる問題の例は、次のとおりです。

- `Printf`系関数のフォーマット指定子と引数の不一致
- 到達しないコード
- ロックのコピー（`sync.Mutex`を含む構造体の値渡しなど）

```bash
go vet ./...
```

- Note.
  - `go test`の実行時にも、`go vet`の主要な検査が自動で行われます。

### 6.2. staticcheck、golangci-lint

#### 解説

`staticcheck`は、`go vet`より広範囲の問題を検出する静的解析ツールです。

未使用コードの検出、非推奨APIの使用、簡略化できるコードの指摘など、
数百のルールを持っています。

`golangci-lint`は、`staticcheck`を含む多数の静的解析ツールを
まとめて実行するランナーです。

設定ファイル（`.golangci.yml`）で、
プロジェクトごとに有効なルールを管理できます。

```yaml
# .golangci.ymlの例
linters:
  enable:
    - staticcheck
    - govet
    - errcheck
    - unused
```

```bash
golangci-lint run
```

- Note.
  - `errcheck`は、戻り値のエラーを無視している箇所を検出します。
    `5.6.`のルールをツールで機械的に確認できます。

### 6.3. テスト

#### 解説

Goでは、テストフレームワークを追加せず、
標準の`testing`パッケージでテストを記述するのが基本です。

入力と期待値の組み合わせが多い場合は、
テーブル駆動テストで記述します。

テストケースの追加が1行で済み、
テストの構造が統一されるため読みやすくなります。

```go
func TestIsValidName(t *testing.T) {
    tests := []struct {
        name  string
        input string
        want  bool
    }{
        {"通常の名前", "yamada", true},
        {"空文字", "", false},
        {"最大長超過", strings.Repeat("a", 51), false},
    }

    for _, tt := range tests {
        t.Run(tt.name, func(t *testing.T) {
            if got := IsValidName(tt.input); got != tt.want {
                t.Errorf("IsValidName(%q) = %v, want %v",
                    tt.input, got, tt.want)
            }
        })
    }
}
```

カバレッジは、次のコマンドで確認できます。

```bash
go test -cover ./...
```

- Note.
  - `t.Run`でサブテストにしておくと、
    失敗したケースの名前が出力され、原因の特定が容易になります。
  - `-race`フラグを付けると、データ競合を検出できます
    （`go test -race ./...`）。

[link1]: https://go.dev/doc/effective_go "Effective Go"
[link2]: https://go.dev/wiki/CodeReviewComments "Go Code Review Comments"
[link3]: https://google.github.io/styleguide/go/ "Google Go Style Guide"
[link4]: https://go.dev/doc/comment "Go Doc Comments"
