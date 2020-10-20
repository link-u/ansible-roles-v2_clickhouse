# ClickHouse

![ansible ci](https://github.com/link-u/ansible-roles-v2_clickhouse/workflows/ansible%20ci/badge.svg)

## 概要

ClickHouse をインストールする role

## 動作確認バージョン

* Ubuntu: 18.04, 20.04
* ansible: 2.8, 2.9

<br>

## 使い方 (ansible)

### Role variables

```yaml
---
### インストール設定 ###############################################################################
## 基本設定
clickhouse_install_flag: True          # インストールフラグ
clickhouse_install_client_only: False  # clickhouse-client のみインストールするかどうかの設定

### clickhouse の設定 #############################################################################
## 基本設定
clickhouse_http_port: 8123              # web からの clickhouse アクセス用ポート
clickhouse_tcp_port: 9000               # clickhouse-client からのアクセス用ポート
clickhouse_interserver_http_port: 9009  # clickhouse クラスタのノード間でデータを交換するためのポート
clickhouse_logger_log: /var/log/clickhouse-server/clickhouse-server.log
clickhouse_logger_errorlog: /var/log/clickhouse-server/clickhouse-server.err.log
clickhouse_data_dir: /var/lib/clickhouse/      # ※ パス最後のスラッシュは残すこと
clickhouse_tmp_path: /var/lib/clickhouse/tmp/  # ※ パス最後のスラッシュは残すこと
clickhouse_timezone: Asia/Tokyo
clickhouse_max_memory_usage: "{{ 10 * 1000 * 1000 * 1000 }}"                 # 10GB # シングルクエリのメモリ最大使用量 (0は未制限)
clickhouse_max_memory_usage_for_user: 0                                      # 1ユーザーあたりのメモリ最大使用量 (0は未制限)
clickhouse_max_memory_usage_for_all_queries: "{{ 20 * 1000 * 1000 * 1000 }}" # 20GB # 1サーバーあたりのメモリ最大使用量 (0は未制限)

## 外部からの全接続を許可するかどうかの設定
# ※ clickhouse レベルでの設定であり, ufw とは別項目
# ※ yes で全接続を許可, no で全接続不可
clickhouse_allow_from_external_access: no

## clickhouse 冗長化設定
clickhouse_interserver_http_host: "{{ local_ipv4 }}"  # 他ノードがこのノードにアクセスするために使用する IP アドレス or ホスト名
clickhouse_use_zookeeper: no                          # zookeeper を使用するフラグ
clickhouse_zookeeper_server_ips: >-                   # zookeeper クラスタに参加しているの IP アドレスリスト
  {{ (group_names | map('extract', groups) | list)[0] |
      map('extract', hostvars, 'local_ipv4') | list }}


### clickhouse のユーザ設定 ########################################################################
## ユーザー設定 (インストール時のusers.xmlと同じ内容にしてある)
# ※ パスワードを設定したいときは以下のワンライナーでランダム文字列を取得する.
#    1. 12文字のランダム文字列を生成する方法
#      NUM=12; PASSWORD=$(base64 < /dev/urandom | head -c"${NUM}"); echo "${PASSWORD}"
#
#    2. パスワードの sha256 ハッシュ値の確認方法
#      echo -n "${PASSWORD}" | sha256sum

# default ユーザの設定
clickhouse_user_default_password_hash: no   # もしくは "{{ 'hogehoge' | hash('sha256') }}" とする.  yes は無効. ※ パスワードは適切に！
clickhouse_user_default_ip_list:
  - "::/0"

# readonly ユーザの設定
clickhouse_user_readonly_password_hash: no  # もしくは "{{ 'fugafuga' | hash('sha256') }}" とする. yes は無効. ※ パスワードは適切に！
clickhouse_user_readonly_ip_list:
  - "::1"
  - "127.0.0.1"

# 上記以外の追加ユーザの設定
clickhouse_extra_users: []
# 設定例
# clickhouse_extra_users:
#   - name: example
#     profile: "readonly"
#     password_hash: no # もしくは "{{ 'piyopiyo' | hash('sha256') }}"
#     allow_ip_list:
#       - "::1"
#       - "127.0.0.1"
```

### Example playbook

```yaml
- hosts:
    - servers
  roles:
    - { role: clickhouse, tags: ["clickhouse"] }
```

## 後方互換性について

### 削除された変数の一覧

以下の変数は `group_vars` から削除して頂いて大丈夫です.

* `clickhouse_hosts`, `clickhouse_host_ips`, `clickhouse_change_ufw`
  * これらの変数は ufw の設定に関する項目であり, それらは ufw role で実行する方針に切り替えたためこの role からは削除しました.

## License

Unless otherwise stated this repository licensed under MIT license. Please read [LICENSE](LICENSE) for the details.

### templates/*.xml.j2

The original version of `templates/*.xml.j2` is obtained from https://clickhouse.tech/ is licensed under the Apache License 2.0. Please read [LICENSE.clickhouse](LICENSE.clickhouse) for the details.
