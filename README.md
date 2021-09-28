# Pulsar Meetup Demo
## 準備
1. [apache-pulsar-2.8.0-bin.tar.gz](https://archive.apache.org/dist/pulsar/pulsar-2.8.0/apache-pulsar-2.8.0-bin.tar.gz) のダウンロード
```
$ wget https://archive.apache.org/dist/pulsar/pulsar-2.8.0/apache-pulsar-2.8.0-bin.tar.gz
$ tar xvfz apache-pulsar-2.8.0-bin.tar.gz
$ cd apache-pulsar-2.8.0
```

## standaloneモードで起動
1. PulsarをStandaloneモードで起動する
```
$ bin/pulsar standalone
```

以降こちらは起動したままにしておく

## テナント / ネームスペースの作成
1. テナントを作成
```
$ bin/pulsar-admin tenants create test-tenant

# 作成されたことを確認
$ bin/pulsar-admin tenants list

"test-tenant"
```

2. ネームスペースを作成
```
$ bin/pulsar-admin namespaces create test-tenant/test-namespace

# 作成されたことを確認
$ bin/pulsar-admin namespaces list test-tenant

"test-tenant/test-namespace"
```

3. メッセージの送受信
producer/consumerごとに別のターミナルを使用します  
接続するconsumerは増やす場合は別ターミナルを追加します

**Consumer**
```
# Consumerを起動
# -s でサブスクリプション名を指定
# -n で受信するメッセージの数を指定が可能、0にするとずっと受信し続ける
$ bin/pulsar-client consume -s sub -n 0 persistent://test-tenant/test-namespace/topic1
```

**Producer**
```
# Producerからメッセージを送信
# -m で送信するメッセージを指定が可能、。カンマ区切りで複数送信できる
$ bin/pulsar-client produce -m 'one,two,three' persistent://test-tenant/test-namespace/topic1
```

## サブスクリプションを試す

サブスクリプションは4タイプ (Exclusive, Shared, Failover, Key_Shared) 存在します  
今回は、Exclusive と Shared の挙動を確認します

### Exclusive
```
# Consumerを起動
$ bin/pulsar-client consume -s sub1 -n 0 persistent://test-tenant/test-namespace/topic2

# 同一のサブスクリプション名でConsumerの起動しようとすると失敗する（別ターミナルで）
$ bin/pulsar-client consume -s sub1 persistent://test-tenant/test-namespace/topic2

ERROR Error while consuming messages
ERROR Exclusive consumer is already connected
org.apache.pulsar.client.api.PulsarClientException$ConsumerBusyException: Exclusive consumer is already connected
...

# 違うサブスクリプション名であれば接続できる（別ターミナルで）
$ bin/pulsar-client consume -s sub2 -n 0 persistent://test-tenant/test-namespace/topic2

# Producerからメッセージを5個送信するとsub,sub2の両方に5個のメッセージが送られる（別ターミナルで）
$ bin/pulsar-client produce -m 1,2,3,4,5 persistent://test-tenant/test-namespace/topic2
```

### Shared
```
# ConsumerをSharedで起動
$ bin/pulsar-client consume -s sub -t Shared -n 0 persistent://test-tenant/test-namespace/topic3

# 同じサブスクリプション名で別のConsumerを起動（別ターミナルで）
$ bin/pulsar-client consume -s sub -t Shared -n 0 persistent://test-tenant/test-namespace/topic3

# Producerからメッセージを5個送信（別ターミナルで）
$ bin/pulsar-client produce -m 1,2,3,4,5 persistent://test-tenant/test-namespace/topic3

# 各Consumerにメッセージがラウンドロビン（ただし厳密なラウンドロビンではなく偏る）で配られる
```
