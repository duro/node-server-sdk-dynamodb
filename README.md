# LaunchDarkly Server-Side SDK for Node.js - DynamoDB integration

[![CircleCI](https://circleci.com/gh/launchdarkly/node-server-sdk-dynamodb.svg?style=svg)](https://circleci.com/gh/launchdarkly/node-server-sdk-dynamodb)

This library provides a DynamoDB-backed persistence mechanism (feature store) for the [LaunchDarkly Node.js SDK](https://github.com/launchdarkly/node-server-sdk), replacing the default in-memory feature store. It uses the AWS SDK for Node.js.

The minimum version of the LaunchDarkly Node.js SDK for use with this library is 5.8.1.

For more information, see also: [Using a persistent feature store](https://docs.launchdarkly.com/v2.0/docs/using-a-persistent-feature-store).

## Quick setup

This assumes that you have already installed the LaunchDarkly Node.js SDK.

1. In DynamoDB, create a table which has the following schema: a partition key called "namespace" and a sort key called "key", both with a string type. The LaunchDarkly library does not create the table automatically, because it has no way of knowing what additional properties (such as permissions and throughput) you would want it to have.

2. Install this package with `npm`:

        npm install launchdarkly-node-server-sdk-dynamodb --save

3. Require the package:

        var DynamoDBFeatureStore = require('launchdarkly-node-server-sdk-dynamodb');

4. When configuring your SDK client, add the DynamoDB feature store:

        var store = DynamoDBFeatureStore('YOUR TABLE NAME');
        var config = { featureStore: store };
        var client = LaunchDarkly.init('YOUR SDK KEY', config);

    By default, the DynamoDB client will try to get your AWS credentials and region name from environment variables and/or local configuration files, as described in the AWS SDK documentation. You can also specify any valid [DynamoDB client options](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB.html#constructor-property) like this:

        var dynamoDBOptions = { accessKeyId: 'YOUR KEY', secretAccessKey: 'YOUR SECRET' };
        var store = DynamoDBFeatureStore('YOUR TABLE NAME', { clientOptions: dynamoDBOptions });

    Alternatively, if you already have a fully configured DynamoDB client object, you can tell LaunchDarkly to use that:

        var store = DynamoDBFeatureStore('YOUR TABLE NAME', { dynamoDBClient: myDynamoDBClientInstance });

5. If you are running a [LaunchDarkly Relay Proxy](https://github.com/launchdarkly/ld-relay) instance, or any other process that will prepopulate the DynamoDB table with feature flags from LaunchDarkly, you can use [daemon mode](https://github.com/launchdarkly/ld-relay#daemon-mode), so that the SDK retrieves flag data only from DynamoDB and does not communicate directly with LaunchDarkly. This is controlled by the SDK's `useLdd` option:

        var config = { featureStore: store, useLdd: true };
        var client = LaunchDarkly.init('YOUR SDK KEY', config);

6. If the same DynamoDB table is being shared by SDK clients for different LaunchDarkly environments, set the `prefix` option to a different short string for each one to keep the keys from colliding:

        var store = DynamoDBFeatureStore('YOUR TABLE NAME', { prefix: 'env1' });

## Caching behavior

To reduce traffic to DynamoDB, there is an optional in-memory cache that retains the last known data for a configurable amount of time. This is on by default; to turn it off (and guarantee that the latest feature flag data will always be retrieved from DynamoDB for every flag evaluation), configure the store as follows:

        var store = DynamoDBFeatureStore('YOUR TABLE NAME', { cacheTTL: 0 });

## About LaunchDarkly

* LaunchDarkly is a continuous delivery platform that provides feature flags as a service and allows developers to iterate quickly and safely. We allow you to easily flag your features and manage them from the LaunchDarkly dashboard.  With LaunchDarkly, you can:
    * Roll out a new feature to a subset of your users (like a group of users who opt-in to a beta tester group), gathering feedback and bug reports from real-world use cases.
    * Gradually roll out a feature to an increasing percentage of users, and track the effect that the feature has on key metrics (for instance, how likely is a user to complete a purchase if they have feature A versus feature B?).
    * Turn off a feature that you realize is causing performance problems in production, without needing to re-deploy, or even restart the application with a changed configuration file.
    * Grant access to certain features based on user attributes, like payment plan (eg: users on the ‘gold’ plan get access to more features than users in the ‘silver’ plan). Disable parts of your application to facilitate maintenance, without taking everything offline.
* LaunchDarkly provides feature flag SDKs for a wide variety of languages and technologies. Check out [our documentation](https://docs.launchdarkly.com/docs) for a complete list.
* Explore LaunchDarkly
    * [launchdarkly.com](https://www.launchdarkly.com/ "LaunchDarkly Main Website") for more information
    * [docs.launchdarkly.com](https://docs.launchdarkly.com/  "LaunchDarkly Documentation") for our documentation and SDK reference guides
    * [apidocs.launchdarkly.com](https://apidocs.launchdarkly.com/  "LaunchDarkly API Documentation") for our API documentation
    * [blog.launchdarkly.com](https://blog.launchdarkly.com/  "LaunchDarkly Blog Documentation") for the latest product updates
    * [Feature Flagging Guide](https://github.com/launchdarkly/featureflags/  "Feature Flagging Guide") for best practices and strategies
