Japanese

Title: ジョブにアップロードされたアーティファクト URL へのアクセス

[store_artifacts step](https://circleci.com/docs/2.0/configuration-reference/#store_artifacts) の後、アップロードされたアーティファクトは、ジョブが完了する前であっても、API を介して直ちにアクセスできるようになります。

このため、API コールを行い、変数に値を格納することで、格納されたアーティファクトの URL にアクセスできます。これで、アーティファクトは、その後のステップで使用できるようになります。たとえば、次の目的でアーティファクトを使用できます。

- [Slack Orb](https://circleci.com/orbs/registry/orb/circleci/slack) を使用すると、アーティファクト URL をメッセージで送信できます。
- これを [when 属性](https://circleci.com/docs/2.0/configuration-reference/#the-when-attribute) と組み合わせ、API コール `on_fail` のみを実行することができます。

最終的には、ビルドが失敗した場合にのみ、アップロードされたアーティファクトを含む Slack メッセージを送信し、ビルドの失敗に関するスクリーンショットや詳細な情報を含むファイルを送信できます。

## ステップとプロセスの実行

[プロジェクトレベルの環境変数](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project)で(パーソナル API トークンを生成して格納)[https://circleci.com/docs/2.0/managing-api-tokens/#creating-a-personal-api-token]する必要があります。この例では、CIRCLE_API_TOKENとして格納する必要があります。

次のスニペットでは、(`vcs`)を `github` または `bitbucket` に、 `(org)` を自分の組織に、そして `(project)` を自分のプロジェクトの名前に更新する必要があります。これは、すべてのアーティファクトを利用可能にするために、すべての `store_artifacts` ステップの後に追加する必要があります。

```
- run:
    name: Get artifacts
    when: on_fail
    command: |
      artifacts=$(curl -X GET "https://circleci.com/api/v2/project/(vcs)/(org)/(project)/$CIRCLE_BUILD_NUM/artifacts" \
      -H "Accept: application/json" \
      -u "$CIRCLE_API_TOKEN:")
      echo "export ARTIFACT_RESPONSE=$artifacts" >> $BASH_ENV
```

上記のステップを実行したら、`$ARTIFACT_RESPONSE` を使用して、ビルドのアーティファクト情報にアクセスできます。これは、スラック メッセージを送信するステップなど、別のステップに渡すことができます。


