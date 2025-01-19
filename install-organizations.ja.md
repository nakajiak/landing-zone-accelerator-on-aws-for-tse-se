# 2. AWS Organizations を使用してランディングゾーンアクセラレータをデプロイする

## 2.1 必須アカウントの作成

[リファレンスアーキテクチャアカウントプロビジョニング AWS CloudFormation スクリプト](./reference-artifacts/organizations-setup/setup-prerequisites.yaml) を実行して、必要な AWS Organizations 組織単位 (OU) とアカウントを設定します。これにより、以下の手順が実行されます。

1. [ホームリージョンで AWS Organizations を有効にする](https://docs.aws.amazon.com/ja_jp/organizations/latest/userguide/orgs_manage_org_create.html)
2. 以下の組織単位を作成します
   - Security
   - Infrastructure
3. 以下のアカウントを作成し、正しい組織単位に配置します。メールアドレスは前提条件のセットアップで割り当てられています。

| アカウント | アカウント名 | 組織単位 |
|---|---|---|
| セキュリティアカウント | Audit | Security |
| ログアーカイブアカウント | LogArchive | Security |

## 2.2 インストーラー CloudFormation スタックのデプロイ
[ステップ 1. スタックの起動](https://docs.aws.amazon.com/ja_jp/solutions/latest/landing-zone-accelerator-on-aws/step-1.-launch-the-stack.html) ページの **Launch Solution** ボタンをクリックします。**リージョンは、通常は US East (N. Virginia) にデフォルト設定されていますが、希望するホームリージョンに設定されていることを確認してください**

スタックに `AWSAccelerator-InstallerStack` という名前を付け、テンプレートのパラメータを確認し、必要に応じてデフォルト値を入力または調整します。例えば：

| パラメータ | 値 |
|---|---|
| **Manual Approval Stage notification email list** | 前提条件で定義されている「LZ operators」のメールを使用するか、カスタマイズすることができます。 |
| **Management Account Email** | 前提条件で定義されている「Management Account」のメールを使用します。 |
| **Log Archive Account Email** | 前提条件で定義されている「Log Archive Account」のメールを使用します。 |
| **Audit Account Email** | 前提条件で定義されている「Security Account」のメールを使用します。 |
| **Control Tower Environment** | これを「No」に設定します |
| **Configuration Repository Location** | 新規デプロイの場合は、これを S3 に設定する必要があります |

特にカスタマイズする理由がない限り、他のすべての値はデフォルトのままにします。

## 2.3 パイプラインが完了するまで待機する
[ステップ 2. 初期環境のデプロイメントを待機する](https://docs.aws.amazon.com/ja_jp/solutions/latest/landing-zone-accelerator-on-aws/step-2.-await-initial-environment-deployment.html)

- `AWSAccelerator-Pipeline` パイプラインが正常に完了するまで待機します。

## 2.4 管理アカウントで IAM Identity Center を有効にする

1. 管理アカウントにログインします。
2. コンソールのリージョンがホーム AWS リージョンに設定されていることを確認します。
3. [AWS IAM Identity Center の有効化](https://docs.aws.amazon.com/ja_jp/singlesignon/latest/userguide/get-set-up-for-idc.html)に関するガイダンスに従います。

> **注：**
> 委任された管理を設定しないでください。これは次のステップで LZA パイプラインによって行われます。

# 3. リファレンスアーキテクチャをデプロイする

Landing Zone Accelerator on AWS ソリューションは、`aws-accelerator-config-<account>-<region>` という名前の S3 バケットと、6 つのカスタマイズ可能な YAML 設定ファイルをデプロイします。YAML ファイルには、ソリューションの最小限の設定が事前に入力されています。このリポジトリの '[config](./config/)' にある設定ファイルは、環境固有の設定を調整した後、デフォルトの設定 S3 バケット内のファイルを置き換える必要があります。

リファレンスアーキテクチャのデプロイを続行する前に、[設定ファイルの使用](https://docs.aws.amazon.com/ja_jp/solutions/latest/landing-zone-accelerator-on-aws/using-configuration-files.html)に関する LZA ガイダンスを読むことをお勧めします。

各設定ファイルを確認し、デフォルト値がニーズに対応していることを確認することをお勧めします。設定ファイルに記載されているコメントに注意を払ってください。リファレンス設定の将来の更新を容易にするために、同じファイル構造を維持し、不要な部分をコメントアウトすることをお勧めします。
</region></account>

## 3.1 リファレンスアーキテクチャの設定ファイルの準備

1. `aws-accelerator-config` という名前のローカルディレクトリを作成します
  a) `mkdir aws-accelerator-config`
2. 設定 S3 バケット (`aws-accelerator-config-<account>-<region>`) から開始設定をダウンロードします。バケット内のオブジェクトキーは `zipped/aws-accelerator-config.zip` です
    - AWS CLI を使用して現在のローカルディレクトリにファイルをダウンロードするには: `aws s3 cp s3://aws-accelerator-config-<account>-<region>/zipped/aws-accelerator-config.zip .`
3. S3 からコピーした設定を解凍します
    - Bash: `unzip aws-accelerator-config.zip -d aws-accelerator-config/`  
    - Powershell: `Expand-Archive -Path aws-accelerator-config.zip -DestinationPath aws-accelerator-config\`
4. このリポジトリ (`landing-zone-accelerator-on-aws-for-tse-se`) をクローンします
5. リポジトリ `landing-zone-accelerator-on-aws-for-tse-se` の `config` フォルダの内容をローカルの `aws-accelerator-config` フォルダにコピーします。accounts-config.yaml などの重複する設定を上書きするよう求められる場合があります。
</region></account></region></account>

## 3.2 追加の組織単位を準備する

[`setup-organizational-units`](./reference-artifacts/organizations-setup/setup-organizational-units.yaml) CloudFormation スクリプトを実行して、リファレンスアーキテクチャで必要とされる以下の組織単位を作成できます。
- Central
- Dev
- Test
- Prod
- UnClass
- Sandbox

## 3.3 必須のカスタマイズ

お好みの IDE を使用して、ローカルの `aws-accelerator-config` フォルダで、以下の値を更新します。
- replacements-config.yaml - このファイルには、他のすべての設定ファイルから参照できるグローバル変数が含まれています。各変数の値を確認し、デプロイメントに適していることを確認してください。**注：** アクティブディレクトリアカウントのパスワードは、[AWS Secrets Manager](https://aws.amazon.com/jp/secrets-manager/) で利用可能です。
- accounts-config.yaml - 事前準備のセクションで割り当てたメールアドレスと一致するように、設定のメールアドレスを更新します。

### 3.3.1 ホームリージョンの変更

ホームリージョンを *ca-central-1* から別のリージョンに変更する場合、以下の設定ファイルに変更を加える必要があります。

- global-config.yaml - **homeRegion: &HOME_REGION ca-central-1** を *ca-central-1* からホームリージョンとして使用しているリージョンに更新する必要があります。例：*homeRegion: &HOME_REGION eu-west-2*
- global-config.yaml - **excludeRegions** ブロック内のホームリージョンへの参照をすべて削除し、*ca-central-1* を追加する必要があります。
- security-config.yaml - **excludeRegions** ブロック内のホームリージョンへの参照をすべて削除し、*ca-central-1* を追加する必要があります。
- customizations-config.yaml - *ca-central-1* への参照をホームリージョンとして使用しているリージョンに更新します

### 3.3.2 アクセラレータプレフィックスの変更

LZA デプロイメント中に **AWSAccelerator** から [アクセラレータプレフィックス](https://docs.aws.amazon.com/ja_jp/solutions/latest/landing-zone-accelerator-on-aws/step-1.-launch-the-stack.html) を変更した場合、以下の設定ファイルに変更を加える必要があります。

- global-config.yaml - **cdkOptions/customDeploymentRole** を *&lt;カスタムプレフィックス>-PipelineRole* に更新します。例えば、*ExamplePrefix-PipelineRole* のようにします。
- iam-config.yaml - **managedActiveDirectories/logs/groupName** を *&lt;カスタムプレフィックス>-/MAD/{{MadDnsName}}* に更新します。例えば、*/ExamplePrefix/MAD/{{MadDnsName}}* のようにします。
- **dynamic-partitioning/log-filters.json** - acceleratorPrefix を *&lt;カスタムプレフィックス>* に更新します。例えば、プレフィックスが *TSEProd* の場合、設定ファイルは以下のようになります。
```
[
  { "logGroupPattern": "/TSEProd/MAD", "s3Prefix": "managed-ad" },
  { "logGroupPattern": "/TSEProd/rql", "s3Prefix": "rql" },
  { "logGroupPattern": "/TSEProd-SecurityHub", "s3Prefix": "security-hub" },
  { "logGroupPattern": "TSEProdFirewallFlowLogGroup", "s3Prefix": "nfw" },
  { "logGroupPattern": "/TSEProd/rsyslog", "s3Prefix": "rsyslog" },
  { "logGroupPattern": "TSEProd-sessionmanager-logs", "s3Prefix": "ssm" }
]
```

実験目的でデモ環境をデプロイし、オンプレミスのネットワークと重複しない特定の CIDR 範囲を定義するなどの特定のカスタマイズを行う必要がない場合は、パイプラインの実行に関するセクションにスキップしてもよいでしょう。

## 3.4 ネットワークのカスタマイズ

顧客が既存のオンプレミス要件 (CIDR 範囲や特定のワークロード要件など) に基づいて、VPN を使用してオンプレミスサービスと統合するなど、ネットワークを制御したいと考える場合がよくあります。

デフォルトでは、リファレンスアーキテクチャは、開発、テスト、本番環境間で分離された、完全に機能する共有ネットワークをデプロイします。次のセクションでは、必要に応じて共有ネットワーキングの CIDR 範囲を変更する方法について説明します。

### 3.4.1 共有ネットワークのカスタマイズ

お客様は新しい IPAM スキーマから始めることをお勧めします。IPAM 設計の詳細については、[アーキテクチャ設計ドキュメント](../doc-tse/architecture-doc/readme.md)をご覧ください。この新しいパターンを採用するには、[network-config.yaml.ipam](../config/network-config.yaml.ipam) を `network-config.yaml` に、[replacements-config.yaml.ipam](../config/replacements-config.yaml.ipam) を `replacements-config.yaml` に名前を変更してください。

IPAM は、ソリューション全体に対して連続した CIDR を使用します。これは現在 `10.0.0.0/8` として指定されており、アーキテクチャ設計ドキュメントで定義されているスキーマに従ってプールに細分化されます。

`replacements-config.yaml` でこれらの範囲をカスタマイズすることができます。

将来的にオンプレミス環境を TGW に接続する予定がある場合は、ASN が一意であることを確認する必要があります。デフォルトの ASN は `65521` です。これを更新する必要がある場合は、`network-config.yaml` の `transitGateways/asn` 値を編集してください。

## 3.5 資産をアセットバケットにコピーする

サンプル設定ファイルは、自己署名証明書を使用してアプリケーションロードバランサーに接続します。有効な証明書は、管理アカウントの S3 アセットバケットにコピーする必要があります。(例: `aws-accelerator-assets-<account-id>-<home-region>`)。

`network-config.yaml` は、アプリケーションロードバランサー (ALB) で使用される証明書を参照しますが、サンプル証明書はローカルで生成する必要があります。初期デプロイメントとデモンストレーションの目的でサンプル証明書を生成するには、次の手順に従ってください。理想的には、既存の認証局を使用して実際の証明書を生成します。設定では、サンプル証明書を `certs` フォルダで参照しているため、サンプル証明書は S3 バケットの `certs` フォルダにアップロードする必要があります。

```
Example1:
openssl req -newkey rsa:2048 -nodes -keyout example1-cert.key -out example1-cert.csr -subj "/C=CA/ST=Ontario/L=Ottawa/O=AnyCompany/CN=*.example.ca"
openssl x509 -signkey example1-cert.key -in example1-cert.csr -req -days 1095 -out example1-cert.crt
```

S3 にコピーするコマンドの例
```
aws s3 cp example1-cert.crt  s3://aws-accelerator-assets-<account-id>-<home-region>/certs/example1-cert.crt
aws s3 cp example1-cert.key  s3://aws-accelerator-assets-<account-id>-<home-region>/certs/example1-cert.key
```

また、設定を更新して、Amazon Certificate Manager (ACM) から自動的に証明書を要求することもできます。LZA の [CertificateConfig](https://awslabs.github.io/landing-zone-accelerator-on-aws/latest/typedocs/v1.6.2/classes/_aws_accelerator_config.CertificateConfig.html) ドキュメントを参照してください。
</home-region></account-id></home-region></account-id></home-region></account-id>

## 3.6 (オプション) 設定ファイルの検証

設定ファイルをローカルで検証するために、LZA 開発者ツールに慣れることをお勧めします。

設定ファイルを検証するには、[Landing Zone Accelerator コード](https://github.com/awslabs/landing-zone-accelerator-on-aws) をダウンロードしてビルドする必要があります。

設定検証を実行するための手順は、[LZA 開発者ガイド](https://awslabs.github.io/landing-zone-accelerator-on-aws/latest/developer-guide/scripts/#configuration-validator) に記載されています。

## 3.7 パイプラインの実行

1. ローカルの設定ファイルを zip 形式に圧縮し、設定 S3 バケットにコピーします。zip アーカイブには、`aws-accelerator-config` のトップフォルダではなく、アーカイブのルートに直接すべてのファイルが含まれていることを確認してください。

    Bash (Linux/MacOS)
    ```bash
    cd aws-accelerator-config/
    rm ../aws-accelerator-config.zip
    zip -r ../aws-accelerator-config.zip . *
    aws s3 cp ../aws-accelerator-config.zip s3://aws-accelerator-config-<account>-<region>/zipped/aws-accelerator-config.zip
    ```

    Powershell (Windows)
    ```powershell
    cd aws-accelerator-config\
    rm ..\aws-accelerator-config.zip
    Compress-Archive -Path .\ -DestinationPath ..\aws-accelerator-config.zip
    aws s3 cp ../aws-accelerator-config.zip s3://aws-accelerator-config-<account>-<region>/zipped/aws-accelerator-config.zip
    ```

2. `AWSAccelerator-Pipeline` パイプラインに手動で変更をリリースします。
3. `AWSAccelerator-Pipeline` パイプラインが正常に完了するまで待ちます。
</region></account></region></account>

# 4. デプロイメント後の手順

パイプラインの正常な実行後、[デプロイメント後の手順](post-deployment.md)に進みます。
