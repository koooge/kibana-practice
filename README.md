### kibana-practice

仮想マシンを立ち上げる
```
$ vagrant up
```

elasticsearchとkibanaコンテナを立ち上げる
```
docker-composeをinstallし、
docker-compose.yml置いて
$ docker-compose up -d
```

td-agentのinstall
```
$ yum install -y dstat
$ curl -L https://toolbelt.treasuredata.com/sh/install-redhat-td-agent2.sh | sh
$ sudo /etc/init.d/td-agent start
$ sudo /etc/init.d/td-agent status

$ sudo td-agent-gem install fluent-plugin-dstat fluent-plugin-elasticsearch --no-document
```

dstatの結果をファイルに出力、加えてelasticsearchに送信。
```
$ cat << EOF | sudo tee /etc/td-agent/td-agent.conf > /dev/null
<source>
  type dstat
  tag dstat
  option -tlcmsgdpyn
</source>

<match dstat>
  type copy
  <store>
    type elasticsearch
    host 192.168.33.10
    port 9200
    logstash_format true
    logstash_prefix logstash
    type_name dstat
    flush_interval 5s
  </store>
  <store>
    type file
    path /var/log/td-agent/dstat.log
  </store>
</match>
EOF

$ sudo systemctl restart td-agent
```

elasticsearch全部消す
```
curl -XDELETE 'http://192.168.33.10:9200/*'
```

参考
$ dstat -tlcmsgdpyn
の出力結果

```
{
  "hostname":"localhost.localdomain",
  "dstat":{
    "system":{ // -tと-y
      "time":"04-04 14:15:28", // -t: システム時間
      "int":"34.0", // -y // 割り込み回数
      "csw":"64.0" // -y // コンテキストスイッチ回数
    },
    "load_avg":{ // -l
      "1m":"0.110", // 1分間のロードアベレージ
      "5m":"0.050", // 5分間のロードアベレージ
      "15m":"0.050" // 15分間のロードアベレージ
    },
    "total_cpu_usage":{ // -c
      "usr":"0.990", // ユーザ空間で使われたCPU時間の割合
      "sys":"0.0", // カーネル空間で使われたCPU時間の割合
      "idl":"99.010", //idle時間の割合
      "wai":"0.0", // 応答待ち状態にあったCPU時間の割合
      "hiq":"0.0", // ハードウェア割り込み処理時間の割合
      "siq":"0.0" // ソフトウェア割り込み処理時間の割合
    },
    "memory_usage":{ // -m
      "used":"197414912.0", //総メモリ使用量
      "buff":"73728.0", // バッファキャッシュのメモリ量（ブロックデバイス用キャッシュ）
      "cach":"211402752.0", // ページキャッシュからバッファキャッシュを除いたメモリ量
      "free":"71782400.0" // 未使用メモリ量
    },
    "swap":{ // -s
      "used":"0.0", // 利用されているスワップのバイト数
      "free":"1107292160.0" // 利用されていないスワップのバイト数
    },
    "paging":{ // -g
      "in":"0.0", // ディスクからメモリに読み込んだバイト数。swap-in
      "out":"0.0" // メモリ不足時にディスクに書き出したバイト数。swap-out
    },
    "dsk/total":{ // -d
      "read":"0.0", // diskの読み込みバイト数
      "writ":"0.0" // diskの書き込みバイト数
    },
    "procs":{ // -p
      "run":"0.0", // 実行中プロセス数
      "blk":"0.0", // カーネル要求後のブロック中プロセス数
      "new":"0.0" // 新しく実行されたプロセス数
    },
    "net/total":{ // -n
      "recv":"60.0", // ネットワーク全体の受信データ量
      "send":"138.0" // ネットワーク全体の送信データ量
    }
  }
}
```