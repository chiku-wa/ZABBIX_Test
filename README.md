# はじめに

Zabbixの構築や各種設定、技術メモをまとめるためのリポジトリ。

利用環境は以下を前提としている。

## 利用環境

| 項目名     | 値                                                                       |
| ------- | ----------------------------------------------------------------------- |
| ホスト端末   | M2 MacBookAir                                                           |
| ホスト端末OS | Sonoma 14.6.1                                                           |
| 仮想環境    | VM Ware Fusion Pro(プロフェッショナル バージョン) 13.5.2 (23775688)                   |
| ゲストOS   | Redhat9.5                                                               |
| MySQL   | 8.0.36                                                                  |
| ZABBIX  | 7.2                                                                     |
| その他     | Apache、PHPなどの各種ソフトウェアは、導入するZabbixのリポジトリ、バージョンに準じて自動的にインストールすることとする。<br> |
|   Ansible      | ansible [core 2.18.1]                                                                        |

# Zabbix環境構築

本手順では、Ansibleで実行する前提としている。

## Ansible構成

本リポジトリでは、Ansibleで自動的にZabbixを構築できるようにプレイブックも同梱している。

ファイル構成は以下の通り。

| ファイル名                              | 概要                               | 備考                                                                                                                                                                                                                          |
| ---------------------------------- | -------------------------------- | --------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `construction-zabbix-playbook.yml` | プレイブック本体。                        | 本リポジトリ内の`ansible`フォルダに存在。プレイブックの内容は基本的に、Zabbix公式サイトのガイドライン（https://www.zabbix.com/jp/download?zabbix=7.2&os_distribution=red_hat_enterprise_linux&os_version=9&components=server_frontend_agent&db=mysql&ws=apache）を元にしている。 |
| `ansible_settings.yml`             | 上記プレイブック実行時に必要となる各種設定情報をまとめたファイル | 本リポジトリ内の`ansible`フォルダ内に雛形ファイル`ansible_settings.yml.example`が存在する。<br>コピーして各人の環境に合わせて編集すること。                                                                                                                                 |
| `hosts`                            | 構築対象のホスト情報が定義されているファイル。          |                                                                                                                                                                                                                             |

最終的に下記の構成になるようにファイルを配置すること。

```
ansible
└construction-ansible_settings.yml
└zabbix-playbook.yml
└hosts
```

## 構築方法

実行例：
```bash
ansible-playbook -i hosts construction-zabbix-playbook.yml
```

上記実行後、以下のURLにアクセスし、Zabbixの初期設定を行うこと。

http://<対象のマシンのIPアドレス>/zabbix/setup.php

# Zabbixの監視設定

## Ansible構成

サンプルとして、Zabbixに対してホストやアイテムの作成処理をAnsibleでプレイブック化している。

なお、OSS製品のWebシステムであるSnipe-ITを監視する想定で作成している。

ファイル構成は以下の通り。

| ファイル名           | 概要                                                             | 備考                                                                                                                                        |
| -------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| `setup-inspection-zabbix-playbook.yml`  | プレイブック本体。                                               | 本リポジトリ内の`ansible`フォルダに存在。                                                                                                   |
| `ansible_settings.yml` | 上記プレイブック実行時に必要となる各種設定情報をまとめたファイル<br>※Zabbix構築で使用した設定ファイルと同じファイル。 | 本リポジトリ内の`ansible`フォルダ内に雛形ファイル`ansible_settings.yml.example`が存在する。<br>コピーして各人の環境に合わせて編集すること。 |
| `hosts`                | Zabbixがインストールされているホスト情報が定義されているファイル。                 |                                                                                                                                             |
最終的に下記の構成になるようにファイルを配置すること。

```
ansible
└construction-ansible_settings.yml
└zabbix-playbook.yml
└hosts
```

## 構築方法

### ①APIトークンの発行

Zabbixのサイトにアクセスし、`ユーザー`→`APIトークン`→`APIトークンの作成`から発行すること。

![alt text](image.png)

### ②Ansibleの設定ファイルにAPIトークンを設定する

下記の通り発行したZabbixのAPIトークンを設定ファイルに定義すること。

zabbix-playbook.yml

```yml
・・・
zabbix:
  api_token: XXXXX
```

### ③プレイブックを実行

下記コマンドでプレイブックを実行すること。

実行例：
```bash
ansible-playbook -i hosts setup-inspection-zabbix-playbook.yml
```


# 参考

ZabbixへAPIリクエストするにあたって、VisualStudioCodeの`REST Client`(https://marketplace.visualstudio.com/items?itemName=humao.rest-client)の利用を前提として、リクエスト用のテキストのサンプルファイルを以下に記述する。


ホストの一覧を取得：

```json
GET http://XXX.XXX.XXX.XXX/zabbix/api_jsonrpc.php
Content-Type: application/json
Authorization: Bearer <APIトークン>

{
    "jsonrpc": "2.0",
    "method": "host.get",
    "params": {
        "output": [
            "hostid",
            "host"
        ],
        "selectInterfaces": [
            "interfaceid",
            "ip"
        ]
    },
    "id": 2
}
```

正規表現を作成する：

```json
GET http://XXX.XXX.XXX.XXX/zabbix/api_jsonrpc.php
Content-Type: application/json
Authorization: Bearer <発行したAPIトークン>

{
    "jsonrpc": "2.0",
    "method": "regexp.create",
    "params": {
        "name": "Snipe-ITのダッシュボード画面表示判定",
        "expressions": [
            {
                "expression_type": 0,
                "expression": "ダッシュボード",
                "case_sensitive": 0
            }
        ]    },
    "id": 2
}
```
