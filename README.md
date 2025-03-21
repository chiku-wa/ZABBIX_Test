# はじめに

Zabbixの構築や各種設定、技術メモをまとめるためのリポジトリ。

利用環境は以下を前提としている。

## 利用環境

| 項目名       | 値                                                                                                                          |
| ------------ | --------------------------------------------------------------------------------------------------------------------------- |
| ホスト端末   | M2 MacBookAir                                                                                                               |
| ホスト端末OS | Sonoma 14.6.1                                                                                                               |
| 仮想環境     | VM Ware Fusion Pro(プロフェッショナル バージョン) 13.5.2 (23775688)                                                         |
| ゲストOS     | Redhat9.5                                                                                                                   |
| MySQL        | 8.0.36                                                                                                                      |
| ZABBIX       | 7.2                                                                                                                         |
| その他       | Apache、PHPなどの各種ソフトウェアは、導入するZabbixのリポジトリ、バージョンに準じて自動的にインストールすることとする。<br> |

# Zabbix環境構築

## Ansibleを用いたSnipe-IT構築

本リポジトリでは、Ansibleで自動的にZabbixを構築できるようにプレイブックも同梱している。

ファイル構成は以下の通り。

| ファイル名           | 概要                                                             | 備考                                                                                                                                        |
| -------------------- | ---------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------- |
| zabbix-playbook.yml  | プレイブック本体。                                               | 本リポジトリ内の`ansible`フォルダに存在。プレイブックの内容は基本的に、Zabbix公式サイトのガイドライン（https://www.zabbix.com/jp/download?zabbix=7.2&os_distribution=red_hat_enterprise_linux&os_version=9&components=server_frontend_agent&db=mysql&ws=apache）を元にしている。                                                                                                   |
| ansible_settings.yml | 上記プレイブック実行時に必要となる各種設定情報をまとめたファイル | 本リポジトリ内の`ansible`フォルダ内に雛形ファイル`ansible_settings.yml.example`が存在する。<br>コピーして各人の環境に合わせて編集すること。 |
| hosts                | 構築対象のホスト情報が定義されているファイル。こ                 |                                                                                                                                             |

最終的に下記の構成になるようにファイルを配置すること。

```
ansible
└ansible_settings.yml
└zabbix-playbook.yml
└hosts
```

実行例：
```bash
ansible-playbook -i hosts zabbix-playbook.yml
```

上記実行後、`http://<対象のマシンのIPアドレス>/zabbix/setup.php`にアクセスすることで、Snipe-ITのプレフライト画面に遷移する。


