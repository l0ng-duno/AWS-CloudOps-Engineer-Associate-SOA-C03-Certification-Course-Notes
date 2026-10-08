# FCC 32 - SNS

> Video chapter [62:59:16](https://youtu.be/5JTBS09e7ew?t=226756) to 63:40:12 · 41 min total · about 15 min lecture, 25 min lab
> Fast path: read this note and skip the video. The slides are short and fully covered below. The lab connects a topic to a Lambda function and spends most of its time fixing the Python handler. Fan-out to SQS, subscription confirmation and the KMS trap with CloudWatch alarms are not taught; they are in "Not in the video".

## Summary

- SNS is managed publish/subscribe. A publisher sends a message to a topic and SNS pushes a copy to every subscription. Subscribers do not poll.
- A subscription joins one topic to one endpoint over one protocol: HTTP/S, email, email-JSON, SQS, Lambda, SMS or mobile push.
- Standard topics: high throughput, at-least-once delivery, no ordering. FIFO topics: ordered within a message group, no duplicates, name ends in `.fifo`.
- A filter policy on a subscription decides which messages that subscriber receives.
- Failed deliveries are retried on a fixed schedule per protocol. When retries run out the message is discarded, unless the subscription has an SQS dead-letter queue.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [63:07:33](https://youtu.be/5JTBS09e7ew?t=227253) | 25m | Creates a topic with CloudFormation, rebuilds it as a SAM template with a Lambda function subscribed to the topic, publishes with `aws sns publish`, and checks CloudWatch Logs | The Lambda subscription is confirmed with no action from him. `--message-structure json` fails with "no default entry", so he drops the flag. The function then fails with a `KeyError`: the payload is a string at `event["Records"][0]["Sns"]["Message"]` and needs `json.loads`. About 8 of the 25 minutes go on debugging the publish call and the handler. | Skip |

### Useful moments inside the Lambda lab

| Time | What you learn |
|------|----------------|
| [63:09:45](https://youtu.be/5JTBS09e7ew?t=227385) | The smallest topic template: `Type: AWS::SNS::Topic` with no properties |
| [63:15:52](https://youtu.be/5JTBS09e7ew?t=227752) | Subscription options in SAM: filter policy, filter policy scope, redrive policy (dead-letter queue) |
| [63:19:56](https://youtu.be/5JTBS09e7ew?t=227996) | The console shows the Lambda subscription as confirmed straight after the deploy |
| [63:24:03](https://youtu.be/5JTBS09e7ew?t=228243) | Two publish errors: "failed to parse" and "no default entry in the JSON message body" |
| [63:28:05](https://youtu.be/5JTBS09e7ew?t=228485) | The event that Lambda receives from SNS, printed and formatted |

## Lecture index

| Time | Topic |
|------|-------|
| [62:59:19](https://youtu.be/5JTBS09e7ew?t=226759) | The publish/subscribe pattern |
| [63:00:23](https://youtu.be/5JTBS09e7ew?t=226823) | What SNS is |
| [63:01:04](https://youtu.be/5JTBS09e7ew?t=226864) | Sources (AWS services that publish to SNS) |
| [63:01:33](https://youtu.be/5JTBS09e7ew?t=226893) | Destinations: application-to-application and application-to-person |
| [63:02:22](https://youtu.be/5JTBS09e7ew?t=226942) | Topics, standard and FIFO |
| [63:04:16](https://youtu.be/5JTBS09e7ew?t=227056) | Publishing: size, large messages, attributes, batches, message structure |
| [63:06:41](https://youtu.be/5JTBS09e7ew?t=227201) | Subscriptions and protocols |
| [63:32:58](https://youtu.be/5JTBS09e7ew?t=228778) | Filter policies |
| [63:34:17](https://youtu.be/5JTBS09e7ew?t=228857) | Message data protection |
| [63:35:55](https://youtu.be/5JTBS09e7ew?t=228955) | Raw message delivery |
| [63:36:39](https://youtu.be/5JTBS09e7ew?t=228999) | Delivery policy and retries |
| [63:38:43](https://youtu.be/5JTBS09e7ew?t=229123) | Dead-letter queues |

## Notes

### Topics and subscribers

- A topic groups subscriptions and can deliver over several protocols at once. The publisher does not know who the subscribers are.
- A subscription is one topic, one protocol, one endpoint. Make as many subscriptions as you need.
- The email protocol sends plain text only. Email-JSON sends the full JSON envelope.

| Subscriber group | Endpoints |
|------------------|-----------|
| Application-to-application (A2A) | SQS, Lambda, Firehose, HTTP/S endpoints, Event Fork Pipelines |
| Application-to-person (A2P) | SMS, email, mobile push, chat tools, PagerDuty |

### Standard versus FIFO topics

| | Standard | FIFO |
|---|----------|------|
| Throughput | Very high | Lower |
| Delivery | At least once; duplicates possible | No duplicates |
| Order | No guarantee | In order within a message group |
| Message groups | No | Yes; several ordered streams in one topic |
| Create | Default | `FifoTopic: true` and a name ending in `.fifo` |
| Use | Alerts, notifications | Banking transactions, order processing |

### Publishing

| Item | Detail |
|------|--------|
| Message size | 256 KB by default (a newer 1 MiB option is listed under the corrections) |
| Larger messages | SNS Extended Client Library for Java or Python: the payload goes to S3 and the topic carries a pointer; up to 2 GB |
| Message attributes | Metadata beside the body. Types: `String`, `String.Array`, `Number`, `Binary` |
| Batches | `publish-batch` sends up to 10 messages in one request, which lowers cost |
| Message structure | `--message-structure json` sends a different text per protocol. The JSON must contain a `default` key. |

```json
{ "default": "Order shipped", "email": "Your order has shipped.", "sms": "Shipped" }
```

### Subscription settings

| Setting | What it does |
|---------|--------------|
| `FilterPolicy` | JSON rules; only matching messages are delivered |
| `FilterPolicyScope` | `MessageAttributes` (the default) or `MessageBody` |
| `RawMessageDelivery` | Sends the bare message without the SNS JSON envelope. For SQS, Firehose and HTTP/S. HTTP/S requests carry the header `x-amz-sns-rawdelivery: true`. |
| `RedrivePolicy` | Names an SQS queue as the dead-letter queue for this subscription |
| `DeliveryPolicy` | Custom retry schedule; HTTP/S only |

Filter operators: exact match, anything-but, prefix, suffix, equals-ignore-case, numeric ranges, IP address, key exists, AND and OR logic.

```json
{ "event": [ { "anything-but": "order_cancelled" } ],
  "price_usd": [ { "numeric": [ ">=", 100 ] } ] }
```

### Delivery retries

SNS retries only on server-side errors, when the endpoint is unavailable.

| Phase | SQS, Lambda, Firehose | Email, SMS, mobile push |
|-------|-----------------------|-------------------------|
| Immediate | 3 times, no delay | 0 times |
| Pre-backoff | 2 times, 1 second apart | 2 times, 10 seconds apart |
| Backoff | 10 times, exponential, 1 to 20 seconds | 10 times, exponential, 10 to 600 seconds |
| Post-backoff | 100,000 times, 20 seconds apart | 38 times, 600 seconds apart |

- Only HTTP/S subscriptions can have their own delivery policy. The others are fixed.
- Backoff functions for HTTP/S: arithmetic, exponential, geometric, linear.

### Dead-letter queues

- Set on the subscription with `set-subscription-attributes` and a `RedrivePolicy` that holds the queue ARN.
- A standard topic needs a standard queue. A FIFO topic needs a FIFO queue.
- The queue's access policy must allow SNS to send to it.

### Message data protection

- A data protection policy on a topic scans messages for personal data (PII) and health data (PHI), using predefined data identifiers (name, address, credit card number) or your own.
- Actions: **audit** (sample up to 99% of messages; findings go to CloudWatch Logs, S3 or Firehose), **de-identify** (mask or redact), **deny** (block the message).
- Standard topics only.

### Not in the video

| Topic | Remember |
|-------|----------|
| Fan-out | One topic with several SQS queues subscribed. Each queue gets its own copy. Each queue's access policy must allow `sqs:SendMessage` from the topic's ARN. |
| Confirmation | Email and HTTP/S subscriptions stay "pending confirmation" until the owner confirms. SQS and Lambda in the same account are confirmed at once. |
| Topic access policy | A resource policy that lets other accounts or services (S3 events, EventBridge, CloudWatch alarms) publish to the topic. |
| Encryption | At rest with a KMS key (server-side encryption). In transit the API uses HTTPS. |
| KMS and AWS services | With the AWS managed key `alias/aws/sns`, CloudWatch alarms and other services cannot publish to the topic. Use a customer managed key whose key policy allows the service. |
| Troubleshooting | Metric `NumberOfNotificationsFailed`; delivery status logging writes each delivery result to CloudWatch Logs. |

## Memory hooks

- SNS pushes, SQS is pulled. SNS in front of several SQS queues is fan-out.
- Retries: AWS endpoints get 23 days of patience (100,015 attempts). People get about 6 hours (50 attempts).
- The dead-letter queue hangs on the subscription, not on the topic.

## Exam traps

- "Several applications must each process every order event": SNS topic with one SQS queue per application.
- "Each subscriber should receive only some of the messages": subscription filter policy, not more topics.
- "The alarm fires but no notification arrives from an encrypted topic": the KMS key policy does not allow CloudWatch. Use a customer managed key.
- "Keep notifications that could not be delivered": dead-letter queue on the subscription.
- "The consumer wants the original message without the SNS wrapper": raw message delivery.
- "Email subscriber receives nothing": the subscription is still pending confirmation.

## Outdated or wrong in the video

- **"Standard messages are 256 KB at most."** 256 KB is still the default, but a topic can now be raised to 1 MiB with the `MaximumMessageSize` attribute. Fairly sure.
- **"Configure an SNS topic to send to a dead-letter queue."** The dead-letter queue belongs to a subscription, not to the topic. His own command, `set-subscription-attributes`, shows this. Very sure.
- **Retry groups described as A2A and A2P.** AWS names them AWS-managed endpoints (SQS, Lambda, Firehose) and customer-managed endpoints (email, SMS, mobile push). HTTP/S is separate: by default 3 retries, 20 seconds apart, and it can be changed. Fairly sure.
- **"SAM stands for service application model."** It is the Serverless Application Model. Very sure.
- **"We talk about dead-letter queues more in SQS."** This video's SQS chapter does not cover them. See the table in FCC 31. Very sure.

Related: [[FCC 31 - SQS]] · [[FCC 23.3 - EventBridge and CloudWatch alarms]] · [[FCC 29 - AWS CloudFormation]] · [[FCC 21 - Identity and Access Management IAM]] · [[FCC 00 - Watch guide]]
