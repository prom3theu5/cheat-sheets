**Pulumi's Infrastructure as Code SDK** is the easiest way to create and deploy cloud software that use
containers, serverless functions, hosted services, and infrastructure, on any cloud.

Simply write code in your favorite language and Pulumi automatically provisions and manages your
[AWS](https://www.pulumi.com/docs/reference/clouds/aws/?utm_campaign=pulumi-pulumi-github-repo&utm_source=github.com&utm_medium=aws-reference-link),
[Azure](https://www.pulumi.com/docs/reference/clouds/azure/?utm_campaign=pulumi-pulumi-github-repo&utm_source=github.com&utm_medium=azure-reference-link),
[Google Cloud Platform](https://www.pulumi.com/docs/reference/clouds/gcp/?utm_campaign=pulumi-pulumi-github-repo&utm_source=github.com&utm_medium=gcp-reference-link), and/or
[Kubernetes](https://www.pulumi.com/docs/reference/clouds/kubernetes/?utm_campaign=pulumi-pulumi-github-repo&utm_source=github.com&utm_medium=kuberneters-reference-link) resources, using an
[infrastructure-as-code](https://en.wikipedia.org/wiki/Infrastructure_as_Code) approach.
Skip the YAML, and use standard language features like loops, functions, classes,
and package management that you already know and love.

For example, create three web servers:

```typescript
let aws = require("@pulumi/aws");
let sg = new aws.ec2.SecurityGroup("web-sg", {
    ingress: [{ protocol: "tcp", fromPort: 80, toPort: 80, cidrBlocks: ["0.0.0.0/0"]}],
});
for (let i = 0; i < 3; i++) {
    new aws.ec2.Instance(`web-${i}`, {
        ami: "ami-7172b611",
        instanceType: "t2.micro",
        securityGroups: [ sg.name ],
        userData: `#!/bin/bash
            echo "Hello, World!" > index.html
            nohup python -m SimpleHTTPServer 80 &`,
    });
}
```

Or a simple serverless timer that archives Hacker News every day at 8:30AM:

```typescript
const aws = require("@pulumi/aws");

const snapshots = new aws.dynamodb.Table("snapshots", {
    attributes: [{ name: "id", type: "S", }],
    hashKey: "id", billingMode: "PAY_PER_REQUEST",
});

aws.cloudwatch.onSchedule("daily-yc-snapshot", "cron(30 8 * * ? *)", () => {
    require("https").get("https://news.ycombinator.com", res => {
        let content = "";
        res.setEncoding("utf8");
        res.on("data", chunk => content += chunk);
        res.on("end", () => new aws.sdk.DynamoDB.DocumentClient().put({
            TableName: snapshots.name.get(),
            Item: { date: Date.now(), content },
        }).promise());
    }).end();
});
```

Many examples are available spanning containers, serverless, and infrastructure in
[pulumi/examples](https://github.com/pulumi/examples).

Pulumi is open source under the [Apache 2.0 license](https://github.com/pulumi/pulumi/blob/master/LICENSE), supports many languages and clouds, and is easy to extend.  This
repo contains the `pulumi` CLI, language SDKs, and core Pulumi engine, and individual libraries are in their own repos.

## Welcome

<img align="right" width="400" src="https://www.pulumi.com/images/docs/quickstart/console.png" />

* **[Get Started with Pulumi](https://www.pulumi.com/docs/get-started/)**: Deploy a simple application in AWS, Azure, Google Cloud, or Kubernetes using Pulumi.

* **[Learn](https://www.pulumi.com/learn/)**: Follow Pulumi learning pathways to learn best practices and architectural patterns through authentic examples.

* **[Examples](https://github.com/pulumi/examples)**: Browse several examples across many languages,
  clouds, and scenarios including containers, serverless, and infrastructure.

* **[Docs](https://www.pulumi.com/docs/)**: Learn about Pulumi concepts, follow user-guides, and consult the reference documentation.

* **[Registry](https://www.pulumi.com/registry/)**: Find the Pulumi Package with the resources you need. Install the package directly into your project, browse the API documentation, and start building.

* **[Pulumi Roadmap](https://github.com/orgs/pulumi/projects/44)**: Review the planned work for the upcoming quarter and a selected backlog of issues that are on our mind but not yet scheduled.

* **[Community Slack](https://slack.pulumi.com/?utm_campaign=pulumi-pulumi-github-repo&utm_source=github.com&utm_medium=welcome-slack)**: Join us in Pulumi Community Slack. All conversations and questions are welcome.

* **[GitHub Discussions](https://github.com/pulumi/pulumi/discussions)**: Ask questions or share what you're building with Pulumi.

## <a name="getting-started"></a>Getting Started

See the [Get Started](https://www.pulumi.com/docs/quickstart/?utm_campaign=pulumi-pulumi-github-repo&utm_source=github.com&utm_medium=getting-started-quickstart) guide to quickly get started with
Pulumi on your platform and cloud of choice.

Otherwise, the following steps demonstrate how to deploy your first Pulumi program, using AWS
Serverless Lambdas, in minutes:

1. **Install**:

    To install the latest Pulumi release, run the following (see full
    [installation instructions](https://www.pulumi.com/docs/reference/install/?utm_campaign=pulumi-pulumi-github-repo&utm_source=github.com&utm_medium=getting-started-install) for additional installation options):

    ```bash
    $ curl -fsSL https://get.pulumi.com/ | sh
    ```

2. **Create a Project**:

    After installing, you can get started with the `pulumi new` command:

    ```bash
    $ mkdir pulumi-demo && cd pulumi-demo
    $ pulumi new hello-aws-javascript
    ```

    The `new` command offers templates for all languages and clouds.  Run it without an argument and it'll prompt
    you with available projects.  This command created an AWS Serverless Lambda project written in JavaScript.

3. **Deploy to the Cloud**:

    Run `pulumi up` to get your code to the cloud:

    ```bash
    $ pulumi up
    ```

    This makes all cloud resources needed to run your code.  Simply make edits to your project, and subsequent
    `pulumi up`s will compute the minimal diff to deploy your changes.

4. **Use Your Program**:

    Now that your code is deployed, you can interact with it.  In the above example, we can curl the endpoint:

    ```bash
    $ curl $(pulumi stack output url)
    ```

5. **Access the Logs**:

    If you're using containers or functions, Pulumi's unified logging command will show all of your logs:

    ```bash
    $ pulumi logs -f
    ```

6. **Destroy your Resources**:

    After you're done, you can remove all resources created by your program:

    ```bash
    $ pulumi destroy -y
    ```

To learn more, head over to [pulumi.com](https://pulumi.com/?utm_campaign=pulumi-pulumi-github-repo&utm_source=github.com&utm_medium=getting-started-learn-more-home) for much more information, including
[tutorials](https://www.pulumi.com/docs/reference/tutorials/?utm_campaign=pulumi-pulumi-github-repo&utm_source=github.com&utm_medium=getting-started-learn-more-tutorials), [examples](https://github.com/pulumi/examples), and
details of the core Pulumi CLI and [programming model concepts](https://www.pulumi.com/docs/reference/concepts/?utm_campaign=pulumi-pulumi-github-repo&utm_source=github.com&utm_medium=getting-started-learn-more-concepts).