# RHEMS Billsan Focus エクスポート設定

## S3バケットセットアップ

### 1. 公開バケットの作成
CloudFormation クイックリンク用テンプレートを設置するための公開バケットを作成します。

```bash
aws s3api create-bucket --bucket billsan-cf-template --region us-west-1 
aws s3api delete-public-access-block --bucket billsan-cf-template
```

### 2. バケットポリシーの設定

`bucket-policy.json` ファイルを作成し、以下の内容を設定：

```json
{
    "Version": "2012-10-17",
    "Statement": [
        {
            "Sid": "Statement1",
            "Effect": "Allow",
            "Principal": "*",
            "Action": "s3:GetObject",
            "Resource": "arn:aws:s3:::billsan-cf-template/*"
        }
    ]
}
```

```bash
aws s3api put-bucket-policy --bucket billsan-cf-template-507011107728 --policy file://bucket-policy.json
```

## CloudFormation デプロイ

### 初回作成（クイックリンク）

以下のリンクから CloudFormation スタックを作成できます：

https://us-east-1.console.aws.amazon.com/cloudformation/home?region=us-east-1#/stacks/quickcreate?&templateURL=https://billsan-cf-template.s3.us-east-1.amazonaws.com/billsan-client-optimization.yaml&stackName=rhems-billsan&param_SlackWebhookURL=[YOUR_SLACK_WEBHOOK_URL]

**パラメータ設定：**
- **Stack名**: `rhems-billsan`
- **Slack通知URL**: `[YOUR_SLACK_WEBHOOK_URL]`

### スタック更新

既存のスタックを更新する場合：

```bash
aws cloudformation update-stack --region us-east-1 --stack-name rhems-billsan \
  --capabilities CAPABILITY_NAMED_IAM \
  --template-url https://billsan-cf-template.s3.us-east-1.amazonaws.com/billsan-client-optimization.yaml \
  --parameters ParameterKey=SlackWebhookURL,UsePreviousValue=true
```

**パラメータ説明：**
- `--stack-name rhems-billsan`: 置換するスタック名
- `--parameters ParameterKey=SlackWebhookURL,UsePreviousValue=true`: 前回の値を利用
- `--capabilities CAPABILITY_NAMED_IAM`: CloudFormationにIAM操作許可（定義が存在する場合必須）
