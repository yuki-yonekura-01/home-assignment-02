Japanese

Title: ジョブ内で、アップロードされたアーティファクト URL にアクセスする

[store_artifacts](https://circleci.com/docs/ja/2.0/configuration-reference/#storeartifacts) ステップ実行後は、ジョブ完了前であっても、API を介してすぐにアップロードされたアーティファクトへアクセスできるようになります。

このため、API を実行して取得した値を環境変数に設定することで、保存されているアーティファクトの URL にアクセスできます。以下のように、アーティファクト URL は、後続のステップで使用できます。

- [Slack Orb](https://circleci.com/orbs/registry/orb/circleci/slack) を使用して、アーティファクト URL を Slack メッセージで送信できます。
- ステップが失敗（`on_fail`）した場合に限りAPI を実行するように、 [`when` 属性](https://circleci.com/docs/ja/2.0/configuration-reference/#the-when-attribute)と組み合わせることができます。


結果的に、ビルドが失敗した場合に限り、アップロードされたアーティファクト（スクリーンショットや失敗の詳細情報を含むファイルなど）を含む Slack メッセージを送信できます。

## ステップとプロセスの実行

[プロジェクトでの環境変数の設定](https://circleci.com/docs/ja/2.0/env-vars/#setting-an-environment-variable-in-a-project)で、[パーソナル API トークンを作成](https://circleci.com/docs/ja/2.0/managing-api-tokens/#creating-a-personal-api-token)し、環境変数に設定します。なお、以下のスニペット例は、環境変数名を `CIRCLE_API_TOKEN` としてパーソナル API トークンの値を設定していることを前提としています。

以下のスニペットは、`(vcs)` をご自身のVCS（`github` または `bitbucket`）に、`(org)` を組織名に、そして `(project)` をプロジェクト名に、それぞれ変更してください。以下のスニペットは、すべてのアーティファクトを利用可能にするために、すべての `store_artifacts` ステップの後に追加してください。

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

上記ステップ実行後、環境変数 `$ARTIFACT_RESPONSE` を使って、ビルド アーティファクトの情報にアクセスできるようになります。`$ARTIFACT_RESPONSE` は、Slack メッセージの送信ステップなど、別のステップに渡すことができます。
