# AWS CDK Kotlin コーディング基準

本文書は、justInCase のプロジェクトにおける AWS CDK を Kotlin で記述する際のコーディング基準である。

justInCase の開発チームは少数であり、同時にジョブタイトルに囚われない動きを発揮したい場合が多くある。したがって、インフラエンジニアのみならず、アプリケーションエンジニアでもある程度 CDK App, Stack, Resource の関係性が把握できることを目指して基準を定めた。

## はじめに

### なぜ CDK by Kotlin か？

単にインフラの構成をコードで管理するのが目的なら、CloudFormation, Terraform 等のツールでも構わない。また、TypeScript や Python などの言語で CDK を用いる選択肢もある。にもかかわらず CDK by Kotlin を採用するのは、以下の理由による。

- [CDK] AWS SDK との連携
  - CloudFormation/CDK の世界で完結しないリソースを扱う際に、AWS SDK を組み合わせて用いる。
  - 例えば、API Gateway が自動で命名する CloudWatch LogGroup や、Amplify Console が自動で生成する Route53 の Record などを動的に参照する場合など。
- [CDK] deploy と dryrun の両立。
  - CloudFormation の欠点として、deploy コマンドに dryrun オプションが存在しないことが挙げられる。
  - すなわち、冪等かつ差分を表示可能な CI パイプラインの構築が困難である。
  - cdk synth, cdk diff cdk deploy などのコマンドはこの課題を解決する。
- [CDK by Kotlin] IntelliJ による型チェック
  - justInCase のバックエンドのアプリケーションでは Kotlin を採用している。
  - インフラのコードに Kotlin を採用することで、バックエンドのアプリケーションエンジニアが新たな環境構築をすることなくインフラのコードを読むことができる。

CDK は上記以外にも様々な CloudFormation の課題を解決しているので、インフラの管理には CDK by Kotlin を用いることが望ましい。

#### CloudFormation/CDK(Kotlin 以外)を考慮すべきケース

- 外部の開発者と協力する場合
  - Kotlin を採用しているとは限らないため。
- ベンダー等から提供された CloudFormation テンプレートがある場合
  - 基本的にそのまま使う前提で作成されているので、保守性を考慮しなくて良い。

## コーディング基準

### ディレクトリ構成

シングルプロジェクトかマルチプロジェクトのいずれかの構成になる。

#### シングルプロジェクト（リポジトリ内に App が一つだけ）の場合

[AWS CDK Kotlin DSL Example](https://github.com/justincase-jp/AWS-CDK-Kotlin-DSL/tree/master/example)に準じる。

##### `/src/main/kotlin/**`

`Main.kt` や `App`クラスのためのディレクトリ。

##### `/src/main/kotlin/**/stacks`

`Stack` クラスの拡張クラスのためのディレクトリ。

##### `/src/main/kotlin/**/resources`

各種`Resource`クラスの拡張クラスのためのディレクトリ。

##### `/functions`

Lambda Function のソースコードのためのディレクトリ。

#### マルチプロジェクト(リポジトリ内に複数の App)の場合

- [App](https://docs.aws.amazon.com/ja_jp/cdk/latest/guide/apps.html)ごとにモジュールを作成する（例: `/ecs`, `/lambda`, `/dynamodb`）
- `/buildSrc` モジュールを作成し、`/buildSrc/src/main/kotlin/versions.kt` でライブラリのバージョンを管理する。

### クラス

#### Stack

[CloudFormation のクラス](https://docs.aws.amazon.com/ja_jp/cdk/latest/guide/stacks.html)を表す。

- `software.amazon.awscdk.core.Stack` を継承すること。
- id は `"${env}-${app}-{stack}"` を基本とする。
  - ハイフン・アンダーバー・スペースは、CloudFormation のテンプレートに変換される際に消えてしまう。
- Tag を用いる。`props = StackProps { tags = mapOf("app" to app) }` のように引数から渡すことができる。
  - CDK ではタグが子の Construct に伝播するため、Stack に設定するだけで個別の Resource に設定せず済み便利。

#### Resource

[CloudFormation のリソース](https://docs.aws.amazon.com/ja_jp/cdk/latest/guide/resources.html)を表す。例として、`lambda.Function` や `s3.Bucket` など。

- 必ずしも CloudFormation テンプレートのリソースと 1:1 である必要はない。
- id はキャメルケースで簡潔に命名する。
  - CDK により suffix が付与されるため、CloudFormation では長い ID になる。
- 物理 ID は CloudFormation/CDK により生成されるため、コーディングしないこと。

### 運用

#### デバッグ

- [aws-vault](https://github.com/99designs/aws-vault) を利用する。
  - CDK CLI が MFA に対応していないため。

## 参考ドキュメント

- [Open CDK Guide](https://github.com/kevinslin/open-cdk)
- [CloudFormation Style Guide by alexgibs](https://github.com/alexgibs/cfnstyle)
- [Kotlin Coding Conventions](https://kotlinlang.org/docs/reference/coding-conventions.html)
- [Standard Go Project Layout](https://github.com/golang-standards/project-layout)
