---
title: Collecting feedback events from SES using Terraform
date: 2023-03-22 16:46:11+00:00
---
A client came to me in full panic mode. AWS had put their SES account under review. Their complaint rate had shot up to 0.6% overnight, and they woke up to a scary (automated) message from AWS.

A bug had caused an email to go out to around 20,000 people. Another bug sent the email up to nine times instead of once. Many of those email addresses had opted-out and they were __not__ happy.

A naive, slow-running SQL query was part of the problem. The job queue system considered the job dead, which it wasn't. It helpfully handed the job off to another worker, which got to work right away. Rinse, lather, and repeat.

There were a few other issues, which I won't go into here. We were most concerned, however, with the lack of visibility around which email addresses make complaints and which email addresses bounce.

We gained visibility by setting up a pipeline of feedback notifications from SES to their application.

After setting the pipeline up, they were able to:

1. Analyse which email addresses complained and what email they received.
2. Analyse which emails addresses bounced and why.
3. Improve the application to prevent complaints and bounces in the first place.

In the end, AWS was happy with the immediate remediation work done so far and were happy with our timeline of future works going forward. They took the account out of review. And, most importantly, the __client stopped panicking__.

- [The problem](#the-problem)
- [The solution](#the-solution)
  - [Configure an SQS queue](#configure-an-sqs-queue)
  - [Configure an IAM for consuming the queue](#configure-an-iam-for-consuming-the-queue)
  - [Configure an SNS topic for email feedback](#configure-an-sns-topic-for-email-feedback)
  - [Configure the SQS access policy](#configure-the-sqs-access-policy)
  - [Subscribe the queue to the SNS topic](#subscribe-the-queue-to-the-sns-topic)
  - [Configure an SES configuration set](#configure-an-ses-configuration-set)
  - [Configure an SNS access policy](#configure-an-sns-access-policy)
  - [Set the event destination for the configuration set](#set-the-event-destination-for-the-configuration-set)
  - [Pass necessary variables to your application](#pass-necessary-variables-to-your-application)
  - [Collect feedback](#collect-feedback)
    - [Storing email complaints](#storing-email-complaints)
    - [Storing bounced emails](#storing-bounced-emails)
- [Conclusion](#conclusion)

## The problem

AWS requires that you collect and action feedback from SES. It's possible to send email without doing that but this can quickly bite you. Your bounce and complaint rate will continue to rise and, as you send out more email, AWS will put your SES account under review. Or, worse, pause it outright.

SES can notify you about a wide range of feedback events. You **must**, however, action these two types of events:

1. **Complaints** - a recipient reporting an email you have sent as spam.
2. **Bounces** - a recipient's mail server has rejected an email you have sent.

For all complaints, it is crucial that you immediately stop sending email to that recipient. The same is also true for a subset of bounces (but not all of them. There are _transient_ or _soft_ bounces where it may be possible to send to that recipient in the future and don't count towards your bounce rate).

This can be achieved using suppression lists, which are enabled in SES by default. Unfortunately, sending suppressed emails still count towards your quota and provides zero additional insight.

As such, collecting feedback has a number of additional benefits over suppression lists:

1. You can analyse the types of complaints and bounces in a way that you cannot via suppression lists in the AWS console. This analysis can be used to identify and prevent sending emails that lead to complaints and bounces in the first place.
2. You can present the information to your users, allowing them to take manual action to rectify the problem. For example, an email address might not exist because it was misspelled.
3. The infrastructure to collect feedback on complaints and bounces can be used to track positive outcomes such as delivery, opens and clicks.

## The solution

We're going to use Terraform to configure a pipeline of feedback events from SES using SNS and SQS. From there, your application can consume feedback from the queue and do something with it:

![Architecture](/collecting-ses-feedback.svg)

We will work backwards, starting with the SQS queue. How your application uses the queue is, however, out of scope for this article - this depends entirely on your stack.

### Configure an SQS queue

This one is nice and simple. If you have a staging environment, you will need to modify the name accordingly.

```hcl
resource "aws_sqs_queue" "email_feedback_queue" {
  name = "EmailFeedbackQueue"
}
```

For this example, I have decided not to configure a dead-letter queue because there will only ever be two types of messages with a well-documented shape. If you plan to use a single general purpose queue for the whole application, using a dead-letter queue is recommended.

### Configure an IAM for consuming the queue

You could use an all-encompassing IAM. This _might_ work if you are small and/or just getting started. But it's probably best to create specific, least-privilege access keys sooner rather than later.

Again, nice and simple:

```hcl
resource "aws_iam_user" "email_feedback_consumer" {
  name = "EmailFeedbackConsumer"
}
```

After this, we want to configure a user policy, granting access to receive and delete messages from the queue we created:

```hcl
resource "aws_iam_user_policy" "email_feedback_consumer_policy" {
  name = "EmailFeedbackConsumerPolicy"
  user = aws_iam_user.email_feedback_consumer.name

  policy = jsonencode({
    Version   = "2012-10-17"
    Statement = [
      {
        Effect = "Allow"
        Action = [
          "sqs:ReceiveMessage",
          "sqs:DeleteMessage"
        ],
        Resource = aws_sqs_queue.email_feedback_queue.arn
      }
    ]
  })
}
```

We can use this IAM to generate exclusive AWS API keys for consuming the email feedback queue.

Finally, we create an access key:

```hcl
resource "aws_iam_access_key" "email_feedback_consumer_key" {
  user = aws_iam_user.email_feedback_consumer.name
}
```

You have a choice when using these keys:

1. Provide a PGP key with the `pgp_key` argument, which encrypts the key and makes it available as an attribute, e.g `aws_iam_access_key.email_feedback_consumer_key.encrypted_secret`. Unfortunately, you cannot automatically configure your services using this method, but your secret won't get stored in your terraform state file.
2. Store the `secret` attribute in AWS Secrets Manager and use it to configure your services automatically, at the expense of the secret being stored in the Terraform state file.

What you choose depends on how comfortable you are with the security around your state file and your particular circumstances. If you are managing your environment configuration files manually, there might not be much point in the added risk of the secret being stored in your Terraform state file.

### Configure an SNS topic for email feedback

Like SQS, creating the topic is simple and straightforward:

```hcl
resource "aws_sns_topic" "email_feedback_topic" {
  name = "EmailFeedbackTopic"
}
```

### Configure the SQS access policy

Now we need to allow SNS to publish notifications to the SQS queue. According to the AWS docs, this is an example of the access policy we need to set up:

```json
{
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "sns.amazonaws.com"
      },
      "Action": "sqs:SendMessage",
      "Resource": "arn:aws:sqs:us-east-2:123456789012:MyQueue",
      "Condition": {
        "ArnEquals": {
          "aws:SourceArn": "arn:aws:sns:us-east-2:123456789012:MyTopic"
        }
      }
    }
  ]
}
```

This leads to a straightforward translation:

```hcl
resource "aws_sqs_queue_policy" "email_feedback_queue_policy" {
  queue_url = aws_sqs_queue.email_feedback_queue.id

  policy = jsonencode({
    Version   = "2012-10-17"
    Id        = "EmailFeedbackQueuePolicy"
    Statement = [
      {
        Sid       = "EmailFeedbackQueuePolicy"
        Effect    = "Allow"
        Principal = "*"
        Action    = "SQS:SendMessage"
        Resource  = aws_sqs_queue.email_feedback_queue.arn
        Condition = {
          ArnEquals = {
            "aws:SourceArn" = aws_sns_topic.email_feedback_topic.arn
          }
        }
      }
    ]
  })
}
```

### Subscribe the queue to the SNS topic

This one is straightforward again:

```hcl
resource "aws_sns_topic_subscription" "email_feedback_subscription" {
  topic_arn = aws_sns_topic.email_feedback_topic.arn
  protocol  = "sqs"
  endpoint  = aws_sqs_queue.email_feedback_queue.arn
}
```

### Configure an SES configuration set

Next, we need to configure an SES configuration set.

We can decide, at runtime, which configuration set to use for each email we send. This is extremely useful, allowing us to have configuration sets for different types of email.

Alternatively, you can set the default configuration set for verified identities. If you're starting from scratch, this is useful.

In our case, we already had a significant number of verified identities, so specifying the config set at the time of sending an email was the simplest approach.

```hcl
resource "aws_ses_configuration_set" "email_feedback_config_set" {
  name = "EmailFeedbackConfigSet"
}
```

### Configure an SNS access policy

This step was _not_ as straightforward as I expected. It won't, thankfully, be as bad for you.

Similar to the SQS access policy, AWS recommends using this example:

```json
{
  "Version": "2012-10-17",
  "Id": "notification-policy",
  "Statement": [
    {
      "Effect": "Allow",
      "Principal": {
        "Service": "ses.amazonaws.com"
      },
      "Action": "sns:Publish",
      "Resource": "arn:aws:sns:topic_region:111122223333:topic_name",
      "Condition": {
        "StringEquals": {
          "AWS:SourceAccount": "111122223333",
          "AWS:SourceArn": "arn:aws:ses:topic_region:111122223333:configuration-set/configuration-set-name"
        }
      }
    }
  ]
}
```

This translates into:

```hcl
resource "aws_sns_topic_policy" "email_feedback_topic_policy" {
  arn = aws_sns_topic.email_feedback_topic.arn

  policy = jsonencode({
    Version   = "2012-10-17"
    Id        = "EmailFeedbackTopicPolicy"
    Statement = [
      {
        Effect    = "Allow"
        Principal = {
          Service = "ses.amazonaws.com"
        }
        Action    = "sns:Publish"
        Resource  = aws_sns_topic.email_feedback_topic.arn
        Condition = {
          StringEquals = {
            "AWS:SourceArn" = aws_ses_configuration_set.email_feedback_config_set.arn
          }
        }
      }
    ]
  })
}
```

Unfortunately, AWS did not like this. No matter what I tried, SES constantly complained about being unable to publish a message to SNS with this error message:

```
InvalidSNSDestination: Could not publish message to SNS topic config set
```

Loosening the condition fixed this problem at the cost of making things a little less isolated:

```hcl
resource "aws_sns_topic_policy" "email_feedback_topic_policy" {
  arn = aws_sns_topic.email_feedback_topic.arn

  policy = jsonencode({
    Version   = "2012-10-17"
    Id        = "EmailFeedbackTopicPolicy"
    Statement = [
      {
        Effect    = "Allow"
        Principal = {
          Service = "ses.amazonaws.com"
        }
        Action    = "sns:Publish"
        Resource  = aws_sns_topic.email_feedback_topic.arn
        Condition = {
		      StringLike = {
		        "AWS:SourceArn" = "arn:aws:ses:*"
		      }
		    }
      }
    ]
  })
}
```

### Set the event destination for the configuration set

Things are, fortunately, back to being straight forward:

```hcl
resource "aws_ses_event_destination" "email_feedback_event_destination" {
  configuration_set_name = aws_ses_configuration_set.email_feedback_config_set.name
  name                   = "EmailFeedback"
  enabled                = true
  matching_types         = ["bounce", "complaint"]

  sns_destination {
    topic_arn = aws_sns_topic.email_feedback_topic.arn
  }
}
```

This should immediately start publishing bounce and complaint feedback events to the SNS topic we configured, which in turn will send these messages to the SQS queue.

### Pass necessary variables to your application

This is, unfortunately, entirely depend on your stack. Typically, you would pass these attributes to your application in some capacity or other (e.g environment variables or configuration management):

1. `aws_sqs_queue.email_feedback_queue.id` - the URL of the feedback queue in SQS.
2. `aws_ses_configuration.email_feedback_config_set.name` - the name of the configuration set.
3. `aws_iam_access_key.email_feedback_consumer_key.id` - the access ID for the queue consumer IAM we configured.

You should also pass `aws_iam_access_key.email_feedback_consumer_key.secret` but be warned: __this is sensitive information__. This should __not__ be passed directly to your services.

As mentioned, either:

1. Encrypt it and pass it in manually
2. Or use AWS Secrets Manager and lock your Terraform state file down as tightly as possible

### Collect feedback

Again, this step entirely depends on your stack. But one simple option is to store feedback events in database tables.

In our case, we set up two simple tables for each event type:

1. `email_complaints`
2. `email_bounces`

#### Storing email complaints

This is how we set up the `email_complaints` table in Postgres:

| Field         | Type      | Constraint |
| ------------- | --------- | ---------- |
| id            | varchar   | PK         |
| email         | varchar   |            |
| complained_at | timestamp |            |
| type          | varchar   |            |
| mail          | jsonb     |            |

The `mail` field represents what SES tried to send. This is useful to help identify the email which caused the complaint.

The indexing strategy is up to you but, for immediate purposes, an index on the `email` field was most useful for us, so we could actively prevent sending emails to known complainers.

#### Storing bounced emails

In a similar vein, this is how we set up the `email_bounces` table:

| Field      | Type      | Constraint |
| ---------- | --------- | ---------- |
| id         | varchar   | PK         |
| email      | varchar   |            |
| bounced_at | timestamp |            |
| type       | varchar   |            |
| subtype    | varchar   |            |
| mail       | jsonb     |            |

Again, the `mail` field contains what SES tried to send.

## Conclusion

Collecting feedback is the first step. Now you need to do something with it.

The first, and simplest, thing you can do is:

1. Prevent sending to __all__ email addresses that have complained
2. Prevent sending to email addresses that result in a hard bounce

As a rough guide, hard bounces are bounce events that have a type of `Permanent`.

After some time, you should be in a position to analyse the collected feedback. You might ask:

- Why might a recipient complain?
- And what about the sent email causes them to complain?
- Are we attempting to send to email addresses that have spelling mistakes in them?
- Can we inform the user about full mailboxes, so they can attempt an alternative form of contact?
- Is there more we can do to verify email addresses before we send to them?

