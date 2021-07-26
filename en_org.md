English

Title: Access uploaded artifact URL in job

After a [store_artifacts step](https://circleci.com/docs/2.0/configuration-reference/#store_artifacts), the uploaded artifacts become immediately accessible via the API, even before the job has completed.

With the above in mind, you can access the URL's for the stored artifacts by making an API call and storing the values in a variable. This can then be used on subsequent steps, such uses for this:

- Used with the [Slack Orb](https://circleci.com/orbs/registry/orb/circleci/slack) you can send the artifact URL's in your message
- This can be combined with the [when attribute](https://circleci.com/docs/2.0/configuration-reference/#the-when-attribute) to only run the API call `on_fail`

The end result could be that you only send a Slack message containing the artifacts uploaded from a failed build, such as screenshots or files containing more information about the failures.

## Run step and process

You will need to [generate and store a personal API token](https://circleci.com/docs/2.0/managing-api-tokens/#creating-a-personal-api-token) in your [project level environment variables](https://circleci.com/docs/2.0/env-vars/#setting-an-environment-variable-in-a-project), in this example, it should be stored as CIRCLE_API_TOKEN.

In the following snippet, you will need to update (`vcs`) to either github or bitbucket, (`org`) to your organization, and (`project`) to the name of your project. This needs to be added after all store_artifacts steps to ensure all artifacts are made available.

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

After the above step is run, then you can access the artifact information for the build with `$ARTIFACT_RESPONSE` -- which in turn can be passed into other steps, such as sending a Slack message.
