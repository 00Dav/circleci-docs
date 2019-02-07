---
layout: classic-docs
title: "Examples"
short-title: "Examples"
description: "CircleCI 2.0 Examples Introduction"
categories:
  - migration
order: 1
---
It is possible to build, test, and deploy applications that run on Linux, Android, and iOS with CircleCI. See the following snippets for a peak into how you can customize the configuration of a job for any platform. You may also configure jobs to run on multiple platforms in a single [`.circleci/config.yml`]({{ site.baseurl }}/2.0/configuration-reference/) file.

## Linux with Docker

{% raw %}

```yaml
version: 2
jobs:
  build:
    working_directory: ~/mern-starter
    # 最初の1行目に指定されたイメージがプライマリコンテナのインスタンスとなります。 ジョブのコマンドはこのコンテナ内で実行されます。
    docker:
      - image: circleci/node:4.8.2-jessie
    # 2 番目に指定されたイメージがセカンダリコンテナのインスタンスとなります。このインスタンスは、ローカルホスト上のプライマリコンテナのポートを通じて共通ネットワークで動作します。
      - image: mongo:3.4.4-jessie
    steps:
      - checkout
      - run:
          name: Update npm
          command: 'sudo npm install -g npm@latest'
      - restore_cache:
          key: dependency-cache-{{ checksum "package.json" }}
      - run:
          name: Install npm wee
          command: npm install
      - save_cache:
          key: dependency-cache-{{ checksum "package.json" }}
          paths:
            - node_modules
```

{% endraw %}

## Linux with Machine

**Note**: Use of machine may require additional fees in a future pricing update.

To use the machine executor with the default machine image, set the machine key to true in `.circleci/config.yml`:

```yaml
version: 2
jobs:
  build:
    machine: true
```

## Android

{% raw %}

```yaml
version: 2
jobs:
  build:
    working_directory: ~/code
    docker:
      - image: circleci/android:api-25-alpha
    environment:
      JVM_OPTS: -Xmx3200m
    steps:
      - checkout
      - restore_cache:
          key: jars-{{ checksum "build.gradle" }}-{{ checksum  "app/build.gradle" }}
#      - run:
#         name: Chmod permissions #if permission for Gradlew Dependencies fail, use this.
#         command: sudo chmod +x ./gradlew
      - run:
          name: Download Dependencies
          command: ./gradlew androidDependencies
```

{% endraw %}          

## iOS

    jobs:
      build-and-test:
        macos:
          xcode: "9.3.0"
        steps:
          ...
          - run:
              name: Run tests
              command: fastlane scan
              environment:
                SCAN_DEVICE: iPhone 6
                SCAN_SCHEME: WebTests
    
    

## 関連情報

Learn more about the [executor types]({{ site.baseurl }}/2.0/executor-types/) used in the examples above.