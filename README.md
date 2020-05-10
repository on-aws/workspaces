# ON-AWS/WORSPACES
My children decided to take some Python cources during [COVID-19](https://en.wikipedia.org/wiki/Coronavirus_disease_2019) outbreak. We decided to use [Amazon Workspaces](https://aws.amazon.com/workspaces/) for them. So they can use their **Chromebooks** to access remote Linux system provided by [Amazon Workspaces](https://aws.amazon.com/workspaces/). Hence this projected created.

:guardsman:**Security** without compromises is a must for us. That is why we are using Amazon Web Services ([AWS](https://aws.amazon.com/)) applying all the best practicies (including but not limited to the runnning all our workloads inside an [Amazon VPC](https://aws.amazon.com/vpc/) private subnets).

## How to install by using AWS CodePipeline
We are using **CodePipeline** as a primary method to a deploy.
This is very opinionated approach to deploy something outside of normal development process.
Usually you would prefer to use [CloudFormation Nested Stacks](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/using-cfn-nested-stacks.html) when you deploy several dependant CloudFormation stacks. And, probably, it is the most correct approach to be honest. So we should support it as well :)

1. Create  CloudFormation Stack by using [pipeline.yaml](pipeline.yaml) template. Name it **on-aws-workspaces-pipeline**. You can name it differntely. But it is what we are using ourselves. We recommend to use default provided value for the **Tag** CloudFormation parameter.
2. Open [AWS CodePipeline Console](https://console.aws.amazon.com/codesuite/codepipeline/pipelines). You should see 2 new pipelines (**on-aws-workspaces-create** and **on-aws-workspaces-delete**). You might see that both pipelines are *In progress* state. This happened because we just created them. But they will not be triggered automatically any time we change something in the repo. Only way to trigger them is to click **Release change** button.
3. Now pay attention to **on-aws-workspaces-delete** pipeline which is most likely *In progress* right now. You can safely go and **Stop execution** of it. Otherwise it will be *In progress* for a long time as you can find a *disabled transition* there. This is done for an extra protection, so you wouldn't delete your resources mistakenly. Next step will explain why do we need **on-aws-workspaces-delete**.
4. :moneybag:(OPTIONAL) You can deploy an optional CloudFormation Stack  **on-aws-workspaces-frugality** by using https://github.com/on-aws/frugality/template.yaml. This stack allows you to schedule execution of **on-aws-workspaces-pipeline** in the morning (hence to provision all the required resources) and **on-aws-workspaces-delete** at night (hence to delete al lthe resources).
**Why do I need it?** _Well this deployment produces several AWS resources which you will be charged for. Very first one is [Amazon VPC NAT gateway](https://aws.amazon.com/vpc/pricing/). Deleting it overnight will help to save some money in your pocket. In general we are trying to make sure that all data survived this harsh exercise and available for you next morning. **WARNING:** FRUGALITY DOES NOT GUARANTEE PERSISTENCE OF YOUR DATA YET. **But** we'll make sure to find a way to make it happen._ :point_right:So you should weight your usecase before applying this optional step.
5. :end:We are sorry to see you go. But our lives go on. And sometime we nned to clean a mess created previously. **on-aws-workspaces-delete** pipeline is exactly for this case (in addition to the optional :four:). It will delete all the AWS resources (including [Amazon CloudWatch Log Groups](https://docs.aws.amazon.com/AmazonCloudWatch/latest/logs/Working-with-log-groups-and-streams.html)), except data artifacts (if applicable for this deployment). Though few extra steps are required to completely wiped all memories:
   1. First make sure that you deleted all essential resources by **Release change** for **on-aws-workspaces-delete** pipeline
   2. Open [AWS Cloud Formation Console](https://console.aws.amazon.com/cloudformation)
   3. Find **on-aws-workspaces-create** and click **Delete**
   4. **IMPORTANT** You will see an error **The bucket you tried to delete is not empty. You must delete all versions in the bucket.**. This is not a bug. But a protective feature. This is your last chance to rethink tour decision.
      1. You can go and delete mentioned Amazon S3 bucket trough [Amazon S3 Console](https://s3.console.aws.amazon.com/s3).
      2. Or you can click **Delete** one more time and exclude S3 bucket resource from deletion.
   5. Find **on-aws-workspaces-delete** and click **Delete**.
   6. (OPTIONAL) Find and delete **on-aws-workspaces-frugality** if you were using it.

