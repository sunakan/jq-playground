
## :

```
$ echo '["a","b","c","d","e"]' | jq -r '.[2:4]'
[
  "c",
  "d"
]
```

## ,

```
$ echo '{"a":1,"b":2,"c":3}' | jq -r '.a,.c'
1
3
```

## to_entries

```
$ echo '{"a":{"a2":{"a3":3}}}' | jq -r '. | to_entries | .[] | .key,.value.a2.a3'
a
3
```

## from_entries

```
$ echo '[{"key":"a","value":1},{"key":"b","value":2}]' | jq -r 'from_entries'
{
  "a": 1,
  "b": 2
}
```

## with_entries

```
$ echo '{"a":1,"b":2}' | jq 'to_entries | map(.key |= "key_"+.) | from_entries'
{
  "key_a": 1,
  "key_b": 2
}
$ echo '{"a":1,"b":2}' | jq 'with_entries(.key |= "key_"+.)'
{
  "key_a": 1,
  "key_b": 2
}
```

## select

```
$ echo '[{"name":"a","type":"apple"},{"name":"b","type":"banana"},{"name":"c","type":"apple"}]' | jq '.[] | select(.type == "apple")'
{
  "name": "a",
  "type": "apple"
}
{
  "name": "c",
  "type": "apple"
}
```

## map

```
$ echo '[{"name":"a","type":"apple"},{"name":"b","type":"banana"},{"name":"c","type":"apple"}]' | jq '. | map(select(.type == "apple"))'
[
  {
    "name": "a",
    "type": "apple"
  },
  {
    "name": "c",
    "type": "apple"
  }
]
```

## --slurpでmapっぽいこと

```
$ '[{"name":"a","type":"apple"},{"name":"b","type":"banana"},{"name":"c","type":"apple"}]' | jq '.[] | select(.type == "apple")' | jq -rs '.'
[
  {
    "name": "a",
    "type": "apple"
  },
  {
    "name": "c",
    "type": "apple"
  }
]
```

## defで関数定義

```
$ echo '[1,2,3]' | jq -r 'def add_one(arg): . + 1; map(add_one(.))'
[
  2,
  3,
  4
]
```

## 応用1:タスク定義

```
$ cat ecs-task-definition.json | jq -r '.taskDefinition | {family: .family, containers: (.containerDefinitions | map({image: .image, log: .logConfiguration}))}'
{
  "family": "example-task-definition-20191001",
  "containers": [
    {
      "image": "httpd:2.4",
      "log": {
        "logDriver": "awsfirelens",
        "options": {
          "Name": "firehose",
          "region": "ap-northeast-1",
          "delivery_stream": "firelens-example-stream"
        }
      }
    },
    {
      "image": "123456789012.dkr.ecr.ap-northeast-1.amazonaws.com/aws-for-fluent-bit:latest",
      "log": {
        "logDriver": "awslogs",
        "options": {
          "awslogs-group": "/ecs/firelens-example",
          "awslogs-region": "ap-northeast-1",
          "awslogs-stream-prefix": "firelens"
        }
      }
    }
  ]
}
```
