```python
import boto3
from pipe import *

# Pretty print.
import pprint
pp = pprint.PrettyPrinter(indent=4)

# Get the service resource.
dynamodb = boto3.resource('dynamodb')

# Check if table exists.
def table_exists(table_name):
    return \
        list(dynamodb.tables.all()) | \
        select(lambda x: x.name) | \
        where(lambda name: name == "users") | \
        count > 0 

# Create table.
def table_create(table_name, partition_key, hash_key):
    table = dynamodb.create_table(TableName=table_name,
        KeySchema=[
            { 'AttributeName': partition_key, 'KeyType': 'HASH'  },
            { 'AttributeName': hash_key,      'KeyType': 'RANGE' } ],
        AttributeDefinitions=[
            { 'AttributeName': partition_key, 'AttributeType': 'S' },
            { 'AttributeName': hash_key,      'AttributeType': 'S' } ],
        ProvisionedThroughput={ 'ReadCapacityUnits': 5, 'WriteCapacityUnits': 5 })

    # Wait until the table exists.
    table.meta.client.get_waiter('table_exists').wait(TableName=table_name)
    return table

# Print out some data about the table.
if not table_exists('users'): table = table_create('users', 'username', 'last_name')
pp.pprint(table.item_count)
pp.pprint(table.creation_date_time)

# Put item.
table.put_item(Item={ 'username': 'janedoe', 'last_name': 'Doe',
        'first_name': 'Jane', 'age': 25, 'account_type': 'standard_user', })
response = table.get_item(Key={ 'username': 'janedoe', 'last_name': 'Doe' })
item = response['Item']
pp.pprint(item)

# Update item.
table.update_item(Key={ 'username': 'janedoe', 'last_name': 'Doe' },
    UpdateExpression='SET age = :val1', ExpressionAttributeValues={ ':val1': 26 })
response = table.get_item(Key={ 'username': 'janedoe', 'last_name': 'Doe' })
item = response['Item']
pp.pprint(item)

# Delete item.
table.delete_item(Key={ 'username': 'janedoe', 'last_name': 'Doe' })

# Batch write.
with table.batch_writer() as batch:
    batch.put_item(Item={ 'username': 'johndoe', 'last_name': 'Doe',
        'account_type': 'standard_user', 'first_name': 'John', 'age': 25,
        'address': { 'road': '1 Jefferson Street', 'city': 'Los Angeles', 
            'state': 'CA', 'zipcode': 90001 } })
    batch.put_item(Item={ 'username': 'janedoering', 'last_name': 'Doering',
        'account_type': 'super_user', 'first_name': 'Jane', 'age': 40,
        'address': { 'road': '2 Washington Avenue', 'city': 'Seattle',
            'state': 'WA', 'zipcode': 98109 } })
    batch.put_item(Item={ 'username': 'bobsmith', 'last_name':  'Smith',
        'account_type': 'standard_user', 'first_name': 'Bob', 'age': 18,
        'address': { 'road': '3 Madison Lane', 'city': 'Louisville',
            'state': 'KY', 'zipcode': 40213 } })
    batch.put_item(Item={ 'username': 'alicedoe', 'last_name': 'Doe',
        'account_type': 'super_user', 'first_name': 'Alice', 'age': 27,
        'address': { 'road': '1 Jefferson Street', 'city': 'Los Angeles',
            'state': 'CA', 'zipcode': 90001 } })

# Massive batch.
with table.batch_writer() as batch:
    for i in range(50):
        batch.put_item(Item={  'username': 'user' + str(i), 'last_name': 'unknown',
            'account_type': 'anonymous', 'first_name': 'unknown', })

# Querying and scanning.
from boto3.dynamodb.conditions import Key, Attr

# Query.
response = table.query(KeyConditionExpression=Key('username').eq('johndoe'))
items = response['Items']
pp.pprint(items)

# Scan.
response = table.scan(FilterExpression=Attr('age').lt(27))
items = response['Items']
pp.pprint(items)

# Scan.
response = table.scan(FilterExpression=Attr('first_name').begins_with('J') & \
    Attr('account_type').eq('super_user'))
items = response['Items']
pp.pprint(items)

# Scan with nested attributes.
response = table.scan(FilterExpression=Attr('address.state').eq('CA'))
items = response['Items']
pp.pprint(items)

# Table delete.
table.delete()
```
