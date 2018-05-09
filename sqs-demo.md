```python
# Test SQS.

# Create queue.
sqs = boto3.resource('sqs')
queue = sqs.create_queue(QueueName='test', Attributes={'DelaySeconds': '5'})
print(queue.url)
print(queue.attributes.get('DelaySeconds'))

# Get existing queue.
queue = sqs.get_queue_by_name(QueueName='test')
print(queue.url)
print(queue.attributes.get('DelaySeconds'))

# Get all queues.
for queue in sqs.queues.all(): print queue

# Send message.
response = queue.send_message(MessageBody='world')
print(response.get('MessageId'))
print(response.get('MD5OfMessageBody'))

# Send message with custom attributes.
queue.send_message(
    MessageBody='boto3', 
    MessageAttributes={ 'Author': { 'StringValue': 'Asim', 'DataType': 'String' } })

# Send batch.
response = queue.send_messages(Entries=[
    { 'Id': '1', 'MessageBody': 'world' },
    { 'Id': '2', 'MessageBody': 'hello' } ])
pp.pprint(response)

# Processing messages.
for message in queue.receive_messages(MessageAttributeNames=['Author']):
    author_text = ''
    if message.message_attributes is not None:
        author_name = message.message_attributes.get('Author').get('StringValue')
        if author_name:
            author_text = ' ({0})'.format(author_name)
    print('Hello, {0}!{1}'.format(message.body, author_text))
    message.delete()
```
