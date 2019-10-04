---
layout: classic-docs
title: "テストの並列実行"
short-title: "テストの並列実行"
description: "テストを並列に実行する方法"
categories:
  - optimization
order: 60
---

If your project has a large number of tests, it will need more time to run them on one machine. To reduce this time, you can run tests in parallel by spreading them across multiple machines. それには、並列処理レベルを指定する必要があります。 You can use either the CircleCI CLI to split test files, or use environment variables to configure each parallel machine individually.

- 目次
{:toc}

## ジョブの並列処理レベルの指定

テストスイートは通常、`.circleci/config.yml` ファイルの[ジョブ]({{ site.baseurl }}/ja/2.0/jobs-steps/#並列ジョブの設定例)レベルで定義されます。 The `parallelism` key specifies how many independent executors will be set up to run the steps of a job.

To run a job's steps in parallel, set the `parallelism` key to a value greater than 1.

```yaml
# ~/.circleci/config.yml
version: 2
jobs:
  test:
    docker:
      - image: circleci/<language>:<version TAG>
    parallelism: 4
```

![Parallelism]({{ site.baseurl }}/assets/img/docs/executor_types_plus_parallelism.png)

For more information, see the [Configuring CircleCI]({{ site.baseurl }}/2.0/configuration-reference/#parallelism) document.

## CircleCI CLI を使用したテストの分割

CircleCI supports automatic test allocation across your containers. The allocation is file-based. It requires the CircleCI CLI, which is automatically injected into your build at run-time.

To install the CLI locally, see the [Using the CircleCI Local CLI]({{ site.baseurl }}/2.0/local-cli/) document.

### テストファイルのグロブ
{:.no_toc}

The CLI supports globbing test files using the following patterns:

- `*` は、任意の文字シーケンスに一致します (パス区切り文字を除く)。
- `**` は、任意の文字シーケンスに一致します (パス区切り文字を含む)。
- `?` は、任意の 1文字に一致します (パス区切り文字を除く)。
- `[abc]` は、角かっこ内の任意の文字に一致します (パス区切り文字を除く)。
- `{foo,bar,...}` は、中かっこ内のいずれかの文字シーケンスに一致します。

To glob test files, pass one or more patterns to the `circleci tests glob` command.

    circleci tests glob "tests/unit/*.java" "tests/functional/*.java"
    

To check the results of pattern-matching, use the `echo` command.

```yaml
# ~/.circleci/config.yml
version: 2
jobs:
  test:
    docker:
      - image: circleci/<language>:<version TAG>
    parallelism: 4
    steps:
      - run:
          command: |
            echo $(circleci tests glob "foo/**/*" "bar/**/*")
            circleci tests glob "foo/**/*" "bar/**/*" | xargs -n 1 echo
```

### テストファイルの分割
{:.no_toc}

The CLI supports splitting tests across machines when running parallel builds. To do this, pass a list of filenames to the `circleci tests split` command.

    circleci tests split test_filenames.txt
    

The CLI looks up the number of containers, along with the current container index. Then, it uses deterministic splitting algorithms to split the test files across all available containers.

By default, the number of containers is specified by the `parallelism` key. You can manually set this by using the `--total` flag.

    circleci tests split --total=4 test_filenames.txt
    

Similarly, the current container index is automatically picked up from environment variables, but can be manually set by using the `--index` flag.

    circleci tests split --index=0 test_filenames.txt
    

#### ファイル名に基づいた分割
{:.no_toc}

By default, `circleci tests split` expects a list of filenames and splits tests alphabetically by test name. There are a few ways to provide this list:

Create a text file with test filenames.

    circleci tests split test_filenames.txt
    

Provide a path to the test files.

    circleci tests split < /path/to/items/to/split
    

Or pipe a glob of test files.

    circleci tests glob "test/**/*.java" | circleci tests split
    

#### ファイルサイズに基づいた分割
{:.no_toc}

When provided with filepaths, the CLI can also split by filesize. To do this, use the `--split-by` flag with the `filesize` split type.

    circleci tests glob "**/*.go" | circleci tests split --split-by=filesize
    

#### タイミングデータに基づいた分割
{:.no_toc}

On each successful run of a test suite, CircleCI saves timings data to a directory specified by the path in the [`store_test_results`]({{ site.baseurl }}/2.0/configuration-reference/#store_test_results) step. If you do not use `store_test_results`, there will be no timing data available for splitting your tests.

To split by test timings, use the `--split-by` flag with the `timings` split type.

    circleci tests glob "**/*.go" | circleci tests split --split-by=timings
    

The CLI expects both filenames and classnames to be present in the timing data produced by the testing suite. By default, splitting defaults to filename, but you can specify classnames by using the `--timings-type` flag.

    cat my_java_test_classnames | circleci tests split --split-by=timings --timings-type=classname
    

If you need to manually store and retrieve timing data, use the [`store_artifacts`]({{ site.baseurl }}/2.0/configuration-reference/#store_artifacts) step.

## 環境変数を使用したテストの分割

For full control over parallelism, CircleCI provides two environment variables that you can use in lieu of the CLI to configure each container individually. `CIRCLE_NODE_TOTAL` is the total number of parallel containers being used to run your job, and `CIRCLE_NODE_INDEX` is the index of the specific container that is currently running. See the [built-in environment variable documentation]({{ site.baseurl }}/2.0/env-vars/#built-in-environment-variables) for more details.

## 分割テストの実行

Globbing and splitting tests does not actually run your tests. To combine test grouping with test execution, consider saving the grouped tests to an environment variable, then passing this variable to your test runner.

```bash
TESTFILES=$(circleci tests glob "spec/**/*.rb" | circleci tests split --split-by=timings)
bundle exec rspec -- ${TESTFILES}
```

The TESTFILES var will have a different value in each container, based on $CIRCLE_NODE_INDEX and $CIRCLE_NODE_TOTAL.

### ビデオ：グロブのトラブルシューティング
{:.no_toc}

<iframe width="854" height="480" src="https://www.youtube.com/embed/fq-on5AUinE" frameborder="0" allow="autoplay; encrypted-media" allowfullscreen mark="crwd-mark"></iframe> 

## 関連項目

[Using Containers]({{ site.baseurl }}/2.0/containers/)

## その他のテスト分割方法

Some third party applications and libraries might help you to split your test suite. These applications are not developed or supported by CircleCI. Please check with the owner if you have issues using it with CircleCI. If you're unable to resolve the issue you can search and ask on our forum, [Discuss](https://discuss.circleci.com/).

- **[Knapsack Pro](https://knapsackpro.com)**：並列 CI ノード間でテストを動的に割り当て、テストスイートの実行を高速化します。 CI のビルド時間への効果は[こちらのグラフ](https://docs.knapsackpro.com/2018/improve-circleci-parallelisation-for-rspec-minitest-cypress)でご確認ください。

- **[PHPUnit Finder](https://github.com/previousnext/phpunit-finder)**：`phpunit.xml` ファイルに対してクエリを行い、テストファイル名の一覧を取得して印刷するヘルパー CLI ツールです。 テストを分割して CI ツールのタイミングに基づいて並列に実行する場合に、このツールを使用すると便利です。

- **[go list](https://golang.org/cmd/go/#hdr-List_packages_or_modules)** - Use the built-in Go command `go list ./...` to glob Golang packages. This allows splitting package tests across multiple containers. ```go test -v $(go list ./... | circleci tests split)```