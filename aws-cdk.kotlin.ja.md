# AWS CDK Kotlin コーディング基準

本文書の目的は、justInCase のプロジェクトにおける AWS CDK を Kotlin で記述する際のコーディング基準である。justInCase の開発チームは少数であり、同時にジョブタイトルに囚われない動きを発揮したい場合が多くある。

したがって、インフラエンジニアのみならず、アプリケーションエンジニアでもある程度 CDK アプリ・スタックの関係性が把握できることを目指して基準を定めた。

## なぜ CDK by Kotlin か？

インフラの構成をコードで管理するのであれば、CloudFormation, Terraform 等のツールでも構わない。また、TypeScript や Python などの言語で CDK を用いる選択肢もある。にもかかわらず CDK by Kotlin を採用するのは、以下の理由による。

- [CDK] AWS SDK との連携
  - CloudFormation/CDK の世界で完結しないリソースを扱う際に、AWS SDK を組み合わせて用いる。
  - 例えば、API Gateway が自動で命名する CloudWatch LogGroup や、Amplify Console が自動で生成する Route53 の Record などを動的に参照する場合など。
- [CDK] deploy と dryrun の両立。
  - CloudFormation の欠点として、deploy コマンドに dryrun オプションが存在しないことが挙げられる。
  - すなわち、冪等かつ差分を表示可能な CI パイプラインの構築が困難である。
  - cdk synth, cdk diff cdk deploy コマンドはこれらの課題を解決する。
- [CDK by Kotlin] IntelliJ による型チェック
  - justInCase のバックエンドのアプリケーションでは Kotlin を採用している。
  - インフラのコードに Kotlin を採用することで、バックエンドのアプリケーションエンジニアが新たな環境構築をすることなくインフラのコードを読むことができる。

CDK は上記以外にも様々な CloudFormation の課題を解決しているので、インフラの管理には CDK by Kotlin を用いることが望ましい。

### CloudFormation/CDK(Kotlin 以外)を考慮すべきケース

- 外部の開発者と協力する場合
  - Kotlin を採用しているとは限らないため。
- ベンダー等から提供された CloudFormation テンプレートがある場合
  - 基本的にそのまま使う前提で作成されているので、保守性を考慮しなくて良い。

### なぜ AWS-CDK-Kotlin-DSL か？

Kotlin で CDK を利用するライブラリとしては、[AWS CDK Java](https://mvnrepository.com/artifact/software.amazon.awscdk)が公式に提供されている。

[AWS CDK Kotlin DSL](https://github.com/justincase-jp/AWS-CDK-Kotlin-DSL)を使用する理由は以下の通り。

- WIP

## 参考文献

- [Open CDK Guide](https://github.com/kevinslin/open-cdk)
-
