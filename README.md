# Travis S3 Deploy Script

This script is used by [Travis CI](https://travis-ci.com/) to continuously deploy artifacts to an S3 bucket.

When Travis builds a push to your project (not a pull request), any files matching `target/*.{war,jar,zip}` will be uploaded to your S3 bucket with the prefix `builds/$DEPLOY_BUCKET_PREFIX/deploy/$BRANCH/$BUILD_NUMBER/`. Pull requests will upload the same files with a prefix of `builds/$DEPLOY_BUCKET_PREFIX/pull-request/$PULL_REQUEST_NUMBER/`.

For example, the 36th push to the `master` branch will result in the following files being created in your `exampleco-ops` bucket:

```
builds/exampleco/deploy/master/36/exampleco-1.0-SNAPSHOT.war
builds/exampleco/deploy/master/36/exampleco-1.0-SNAPSHOT.zip
```

When the 15th pull request is created, the following files will be uploaded into your bucket:
```
builds/exampleco/pull-request/15/exampleco-1.0-SNAPSHOT.war
builds/exampleco/pull-request/15/exampleco-1.0-SNAPSHOT.zip
```

## build.rb

The included build script (inspired by
[travis-maven-deploy](https://github.com/perfectsense/travis-maven-deploy)'s
brightspot-deploy.rb) takes advantage of Travis's .m2 cache to skip building
artifacts that have not changed. It only works if your project follows the
multi-module conventions as established in Brightspot's express-archetype. Minor
deviations are possible via command line parameters.

To speed up pull request builds even further, use the parameter `--skip-tests-if-pr`.

## Usage

Your .travis.yml should look something like this:

```yaml
language: java

jdk:
  - oraclejdk8

install: true

branches:
  only:
    - develop
    - master
    - /^release-.*$/

cache:
  directories:
    - $HOME/.m2
    - node
    - node_modules

env:
  global:
    - DEPLOY_BUCKET=exampleco-builds
    - DEPLOY_BUCKET_PREFIX=exampleco # optional if using a shared deployment bucket
    - DEPLOY_BRANCHES=develop\|release # optional - all branches defined in "branches" above is the default
    - DEPLOY_SOURCE_DIR=$TRAVIS_BUILD_DIR/site/target # optional - if your war file is somewhere other than ./target

before_script:
  - git clone https://github.com/perfectsense/travis-s3-deploy.git
  - cp travis-s3-deploy/settings.xml ~/.m2/settings.xml

script:
  - travis-s3-deploy/build.rb --skip-tests-if-pr && travis-s3-deploy/deploy.sh
```

Note that any of the above environment variables can be set in Travis, and do not need to be included in your .travis.yml. `AWS_ACCESS_KEY_ID` and `AWS_SECRET_ACCESS_KEY` should always be set to your S3 bucket credentials as environment variables in Travis, not this file.


