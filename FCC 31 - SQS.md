# FCC 31 - SQS

> Video chapter [61:58:19](https://youtu.be/5JTBS09e7ew?t=223099) to 62:59:16 · 61 min total · about 13 min lecture, 48 min labs
> Fast path: read this note. The lectures are a run of 30 to 60 second slides, all covered in the tables below. The 41-minute standard queue lab is mostly script and typo trouble; skip it. If you watch anything, watch the FIFO lab from [62:48:05](https://youtu.be/5JTBS09e7ew?t=226085) (4 min), where three error messages show how FIFO differs. Dead-letter queues are not taught in this chapter; see "Not in the video".

## Summary

- SQS is a managed queue that decouples a producer from a consumer. Consumers poll for messages; nothing is pushed to them.
- Standard queue: nearly unlimited throughput, at-least-once delivery, best-effort order. FIFO queue: strict order within a message group, no duplicates, 300 calls per second (3,000 messages with batches of 10).
- Receiving does not delete. The message is hidden for the visibility timeout (default 30 seconds). The consumer must then delete it with the receipt handle, or it becomes visible again.
- Numbers: retention 4 days by default (60 seconds to 14 days), delay up to 15 minutes, long polling up to 20 seconds, visibility timeout up to 12 hours, 10 messages per batch.
- Access is granted by IAM policies and by the queue's access policy. The access policy is what lets an SNS topic or another account send to the queue.

## Labs and demos

| Time | Len | What it shows | Takeaway | Verdict |
|------|-----|---------------|----------|---------|
| [62:04:49](https://youtu.be/5JTBS09e7ew?t=223489) | 41m | Standard queue: CloudFormation template, console send and poll, CLI `send-message` and `receive-message`, then send, receive and delete in Ruby | Works in the end, after about 20 minutes of trouble: `FifoQueue: false` is rejected by CloudFormation, the attribute type must be `String` with a capital S, and his send and receive scripts were swapped. Final test: five messages come back out of order and in partial sets (2, 3, 5, then 1, 4). | Skip |
| [62:46:21](https://youtu.be/5JTBS09e7ew?t=225981) | 7m | FIFO queue: same template with `FifoQueue: true`, same five messages | Three errors, each one an exam fact: the send needs a message group ID, it needs a deduplication ID, and a per-message `DelaySeconds` is not allowed. The messages then come back as 1, 2, 3, 4, 5. Watch [62:48:05](https://youtu.be/5JTBS09e7ew?t=226085) to 62:52:30. | Skim |

### Useful moments inside the standard queue lab

| Time | What you learn |
|------|----------------|
| [62:12:04](https://youtu.be/5JTBS09e7ew?t=223924) | "Unknown attribute FifoQueue": for a standard queue, leave the property out instead of setting it to false |
| [62:16:07](https://youtu.be/5JTBS09e7ew?t=224167) | Console: Send and receive messages, Poll for messages, and the built-in integrations (Lambda trigger, EventBridge Pipes, dead-letter queue, SNS subscription) |
| [62:20:14](https://youtu.be/5JTBS09e7ew?t=224414) | Message attributes: up to 10 per message, each with a name, a data type and a value |
| [62:25:29](https://youtu.be/5JTBS09e7ew?t=224729) | `receive-message` with a maximum of 10 returns only one. He never explains it: short polling asks only some of the servers. Long polling (`--wait-time-seconds`) fixes it. |
| [62:33:37](https://youtu.be/5JTBS09e7ew?t=225217) | The received message is still in the queue. Deleting is a separate call that needs the receipt handle. |
| [62:41:57](https://youtu.be/5JTBS09e7ew?t=225717) | Ruby: `delete_message` with the receipt handle; the next receive returns nothing |
| [62:45:02](https://youtu.be/5JTBS09e7ew?t=225902) | Standard queue returns messages out of order |

## Lecture index

| Time | Topic |
|------|-------|
| [61:58:19](https://youtu.be/5JTBS09e7ew?t=223099) | Messaging and queueing, what SQS is, queue types, size, retention, encryption |
| [62:00:54](https://youtu.be/5JTBS09e7ew?t=223254) | Extended Client Library for large messages |
| [62:01:57](https://youtu.be/5JTBS09e7ew?t=223317) | Standard queues |
| [62:03:17](https://youtu.be/5JTBS09e7ew?t=223397) | FIFO queues, high throughput mode |
| [62:53:00](https://youtu.be/5JTBS09e7ew?t=226380) | Attribute-based access control (tags) |
| [62:53:47](https://youtu.be/5JTBS09e7ew?t=226427) | Access policies |
| [62:54:35](https://youtu.be/5JTBS09e7ew?t=226475) | Message attributes (metadata) |
| [62:55:14](https://youtu.be/5JTBS09e7ew?t=226514) | Visibility timeout |
| [62:55:48](https://youtu.be/5JTBS09e7ew?t=226548) | Delay queues |
| [62:56:33](https://youtu.be/5JTBS09e7ew?t=226593) | Message timers |
| [62:57:25](https://youtu.be/5JTBS09e7ew?t=226645) | Temporary queues |
| [62:58:28](https://youtu.be/5JTBS09e7ew?t=226708) | Short and long polling |

## Notes

### Standard versus FIFO

| | Standard | FIFO |
|---|----------|------|
| Throughput | Nearly unlimited | 300 calls per second for each of send, receive and delete; 3,000 messages per second with batches of 10; more with high throughput mode |
| Delivery | At least once; duplicates are possible | Exactly-once processing; duplicates are removed |
| Order | Best effort | Strict within a message group |
| Needed on send | Message body | Body, message group ID, and a deduplication ID (or content-based deduplication on the queue) |
| Per-message delay | Allowed | Not allowed; queue-level delay only |
| CloudFormation | Omit `FifoQueue` | `FifoQueue: true` |
| Change type later | Cannot be converted to FIFO | Must be created as FIFO |

- **Message group ID**: messages with the same ID are delivered in order. Give each producer or each ordered stream its own ID.
- **Deduplication ID**: a token that makes a repeat send of the same message be ignored.
- The consumer of a standard queue must cope with duplicates and with messages out of order.

### Numbers to memorise

| Setting | Default | Range |
|---------|---------|-------|
| Message retention | 4 days | 60 seconds to 14 days |
| Visibility timeout | 30 seconds | 0 seconds to 12 hours |
| Delay (`DelaySeconds`) | 0 seconds | 0 to 15 minutes |
| Long polling wait (`WaitTimeSeconds`) | 0 (short polling) | Up to 20 seconds |
| Messages per receive or per batch | 1 per receive | 1 to 10 |
| Message attributes | None | Up to 10 per message |
| Message size | | 256 KB in the video; now 1 MiB |
| Large payloads | | Up to 2 GB with the Extended Client Library (Java, Python); the body goes to S3 and the queue carries a pointer |

### Life of a message

1. The producer sends it. With a delay it stays hidden at first.
2. A consumer receives it. SQS hides it from other consumers for the visibility timeout.
3. The consumer finishes the work and deletes the message with its receipt handle.
4. If it is not deleted in time, it becomes visible again and is processed a second time.
5. If nobody deletes it, SQS drops it when the retention period ends.

### Delays and polling

| Feature | Set on | What it does |
|---------|--------|--------------|
| Delay queue | Queue | Hides every new message for `DelaySeconds` after it arrives. On a standard queue a change affects new messages only; on FIFO it also affects messages already in the queue. |
| Message timer | One message | `DelaySeconds` on the send call. Not supported on FIFO. |
| Visibility timeout | Queue, or one receive call | Hides a message after it has been received |
| Short polling | `WaitTimeSeconds` = 0 | Returns at once, even when empty. The default. |
| Long polling | `WaitTimeSeconds` 1 to 20 | Waits for a message to arrive. Fewer empty responses, so lower cost. |

### Access, encryption, attributes

- **Access policy**: a resource policy on the queue, like a bucket policy. His example allows an SNS topic to call `sqs:SendMessage`.
- **ABAC**: allow or deny by tags on the queue or the caller. Condition keys: `aws:ResourceTag`, `aws:RequestTag`, `aws:TagKeys`.
- **Encryption at rest**: SSE-SQS (key managed by SQS) or SSE-KMS (your own KMS key).
- **Message attributes**: metadata beside the body. Data types `String`, `Number`, `Binary`, with optional custom labels such as `Binary.gif`. Type names are case-sensitive.
- **Temporary queues**: a Java client that makes short-lived queues for request-response patterns and removes them when unused.

```bash
aws sqs send-message --queue-url "$QUEUE_URL" --message-body "Hello Mars" \
  --message-attributes '{"fruit":{"DataType":"String","StringValue":"apple"}}'
aws sqs receive-message --queue-url "$QUEUE_URL" --max-number-of-messages 10 \
  --wait-time-seconds 20 --message-attribute-names All
aws sqs delete-message --queue-url "$QUEUE_URL" --receipt-handle "<handle>"
```

For FIFO add `--message-group-id` and `--message-deduplication-id` to the send.

### Not in the video

| Topic | Remember |
|-------|----------|
| Dead-letter queue (DLQ) | A second queue for messages that keep failing. Set a redrive policy on the source queue with `maxReceiveCount`; after that many receives the message moves to the DLQ. |
| DLQ rules | Same type as the source (FIFO needs a FIFO DLQ). Give the DLQ a long retention period. |
| DLQ redrive | After fixing the bug, move messages from the DLQ back to the source queue with "Start DLQ redrive". |
| FIFO naming | A FIFO queue name must end in `.fifo`. |
| Deduplication window | 5 minutes. Content-based deduplication uses a SHA-256 hash of the body. |
| CloudWatch metrics | `ApproximateNumberOfMessagesVisible` (backlog, used to scale consumers) and `ApproximateAgeOfOldestMessage` (consumers are falling behind). |
| Lambda as consumer | Set the visibility timeout to at least six times the function timeout. |

## Memory hooks

- SQS: you pull. SNS: it pushes.
- Three hiding places in time order: delay hides a message when it arrives, visibility timeout hides it after it is read, retention removes it at the end.
- Rising numbers: 4 and 14 (days kept), 15 (minutes of delay), 20 (seconds of long poll), 30 (seconds invisible).
- FIFO needs two IDs: the group ID is the line you stand in, the deduplication ID is your ticket number.

## Exam traps

- "The same message is processed twice": the visibility timeout is shorter than the processing time. Raise it.
- "Many empty receives" or "reduce the cost of polling an idle queue": long polling, `WaitTimeSeconds` up to 20.
- "Order must be kept and no duplicates": FIFO queue. An existing standard queue cannot be converted.
- "Postpone every message by five minutes": delay queue. "Postpone one message": message timer, standard queues only.
- "A bad message is retried for ever": dead-letter queue with `maxReceiveCount`.
- "SNS cannot deliver to the queue": the queue's access policy must allow `sqs:SendMessage` from the topic.

## Outdated or wrong in the video

- **"Message size is up to 256 KB."** The limit has been 1 MiB since August 2025. Older practice questions still say 256 KB. The Extended Client Library is still the answer for anything larger, up to 2 GB. Very sure.
- **"Your consumer has to specify a message group ID to receive."** The producer sets the group ID when sending. A receive call cannot ask for a group. Very sure.
- **"High throughput mode allows 3,000 messages per second with batching, and probably costs extra."** 3,000 with batching is the normal FIFO limit. High throughput mode reaches up to 70,000 calls per second without batching in the largest Regions. It has no separate charge. Fairly sure.
- **"Temporary queues are like a mock SQS inside your app."** They are virtual queues that the client multiplexes onto one real SQS queue. Fairly sure.
- **Extended Client Library slide.** He reads "published to the SNS topic", copied from the SNS version. For SQS the pointer goes into the queue. Very sure.

Related: [[FCC 32 - SNS]] · [[FCC 23.3 - EventBridge and CloudWatch alarms]] · [[FCC 20 - ASG]] · [[FCC 30 - Elastic Beanstalk Follow Along]] · [[FCC 00 - Watch guide]]
