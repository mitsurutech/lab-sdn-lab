# 授業で使える SDN ラボ: Open vSwitch、OpenFlow、学習スイッチを 1 台の VM で

[English](README.md) | [Français](README.fr.md) | [日本語](README.ja.md) | [Español](README.es.md) | [Deutsch](README.de.md)

[![check](https://github.com/mitsurutech/lab-sdn-lab/actions/workflows/check.yml/badge.svg)](https://github.com/mitsurutech/lab-sdn-lab/actions/workflows/check.yml)

スイッチがどのように通信を転送するかを教えるための、すぐ使える**ソフトウェア定義ネットワーク（SDN）ラボ**です。学校が生徒一人ひとりに専用のマシンとブラウザー上のターミナルを提供する [MitsuruTech ラーニングプラットフォーム](https://mitsurutech.co.uk/ja/learning)から提供しています。生徒ごとに **Open vSwitch**、**Mininet**、OpenFlow コントローラー用ライブラリ **os-ken** が入った Ubuntu **VM** と、3 台のホスト `h1`、`h2`、`h3` をつないだスイッチ `br0` が用意されます。スイッチはフローテーブルが空の状態で始まるので、何も転送しません。生徒は `ovs-ofctl` で **OpenFlow のフローを手で書き**、続いて、各 MAC アドレスがどのポートにあるかを学習してフローを入れる**学習スイッチのコントローラー**を仕上げます。生徒のコンピューターには何もインストールせず、ネットワーク機器も要りません。

## 生徒が行うこと

ラボはホームフォルダーに、何をするかを説明する `README.md`（答えは書かれていません）と、編集する 2 つのファイル `/etc/sdnlab/flows.txt` と `/etc/sdnlab/switch.py` を置きます。どちらもラボが起動するたびに読み込まれます。ホストはネットワーク名前空間で（`h1` は 10.0.0.1、`h2` は 10.0.0.2、`h3` は 10.0.0.3、MAC アドレスは末尾が 01、02、03）、スイッチは fail-mode secure なので、フローがなければすべてのパケットを破棄します。3 つのパートに分かれています。

1. **Mininet**（採点なし）: `sudo mn --mac --test pingall` を実行し、Mininet のプロンプトでノード、リンク、コントローラーが入れるフローを見ます。
2. **フローテーブルを手で書く**（`flows.txt`）: h1 と h2 が双方向に通信でき（エコーだけでなく ARP も）、h3 に一致するフローが h3 を破棄し、どのフローにもアクション `NORMAL` を使わないこと。
3. **コントローラーを仕上げる**（`switch.py`）: `packet_in` にある 3 つの TODO（送信元アドレスの学習、ポートの選択、フローの登録）を完成させ、会話の 2 つ目以降のパケットがコントローラーに届かないようにします。

ラボは `sdnlab` コマンド（`status`、`flows`、`controller`、`reset`、`shell h1`）を追加します。ラボの再構築、コントローラーの起動、2 つのファイルを配布時の状態に戻すこと、ホスト内のシェルを開くことができます。

## 採点のしくみ

ラボには**チェック**があります。提出された VM を、教員が中身を読まなくても採点するスクリプトです。保存されたマシンのコピーを新しく起動した状態で実行し、生徒の 2 つのファイルを自分で読み込むので、手で入力しただけで保存していないフローは数えられません。

- **パート 2（コントローラーなし）:** h1 と h2 が通信でき、h3 は誰にも届かず誰も h3 に届かないこと。h3 に一致するフローが h3 を破棄し、h3 のパケットが実際にそのフローに当たっていること。アクション `NORMAL` のフローは通りません。
- **パート 3（生徒のコントローラー）:** チェックがコントローラーを起動し、h2 に誰も見たことのない MAC アドレスを与えて（先に書いておいた表が役に立たないように）、すべての組み合わせで ping します。スイッチは未知のパケットをコントローラーに送り、コントローラーは各アドレスの組に対して、`priority=1,in_port=PORT,dl_src=SOURCE_MAC,dl_dst=DESTINATION_MAC` の形で、学習したポートへの出力アクションを持つフローを入れている必要があります。入れたフローのどれもフラッディングしてはいけません。

出力の 1 行目に、何が正しくないかが書かれます。

できないこと: ping で試してフローテーブルを読むだけで、ほかの種類の通信は試しません。また、自分の VM で root 権限を持つ生徒が、チェックが頼るツールを書き換えることは可能です。これは授業のためのチェックであって、証明ではありません。

## ファイルの中身

`sdn-lab.lab.yaml` は、プラットフォームがそのまま実行する **VM ラボ定義**の全体です。

| キー | 内容 |
| --- | --- |
| `cpus`, `memory`, `disk` | 生徒ごとの VM の大きさ: CPU 1 つ、1024 MB、10 GB |
| `open` | ブラウザーで開くポート: なし。ラボはターミナルです |
| `setup` | 各生徒の VM が初めて起動するときに root で 1 回実行されるスクリプト。Ubuntu のアーカイブから Open vSwitch、Mininet、os-ken、`iproute2`、`ping`、`tcpdump` をインストールし、`sdnlab` コマンド、コントローラーの小さな起動スクリプト、編集する 2 つのファイル、systemd ユニット、README を書き出します |
| `check` | 採点スクリプト。保存されたディスクのコピーを新しく起動した状態で root として実行し、終了ステータスが結果で、`0` なら合格です |

## プラットフォームでの使い方

**クラスの課題として出すには:**

1. 担当するモジュールでタスクを追加し、**Ubuntu VM ラボ**のカードを選びます。
2. Software-defined networking のサンプルを押すか、`sdn-lab.lab.yaml` の内容を貼り付けます。サンプルを使うとテキストが置き換わり、元に戻すで自分のテキストに戻せます。
3. 公開して、まず自分用のコピーを試します。生徒ごとに専用の VM があり、セッションをまたいで保持されます。初回の起動は、setup がパッケージをインストールするため数分かかります。
4. ほかのタスクと同じように割り当てます。提出されたら、クラス一覧の生徒の行でチェックを実行します。保存されたマシンのコピーが起動し（書き戻されません）、合格かどうかと出力内容が表示されます。

プラットフォームの Basics レッスン「SDN Basics」は、このラボで進めます。ヘルプマニュアルの[このラボのページ](https://mitsurutech.co.uk/ja/help/lab-sdn-lab)、[Ubuntu VM ラボのガイド](https://mitsurutech.co.uk/ja/help/labs)、[サンプルラボのページ](https://mitsurutech.co.uk/ja/help/sample-labs)に残りの説明があります。

まだ学校のアカウントがありませんか。[最初の 1 年は無料](https://mitsurutech.co.uk/ja/pricing)（生徒 50 人まで）で、[登録](https://mitsurutech.co.uk/ja/signup)にはメールアドレスだけが必要です。

## ファイルの検証

push のたびに `.github/workflows/check.yml` が、ファイルが上記のキーを持つ有効な YAML であること、`setup` と `check` がそれぞれプラットフォームの 16 KB の上限に収まっていることを確認します。VM は起動しません。それはプラットフォーム自身のマシンで行います。

## ラボで動かないこと

VM が到達できるのは Ubuntu のパッケージアーカイブ、PyPI、npm、GitHub だけです。ホスト間のネットワークは VM の中の仮想のもので、外には出ません。os-ken は非推奨の eventlet に依存していて、コントローラーのログに通知が出ます。そのためこのラボでは先に `ovs-ofctl` と Mininet を学びます。Mininet の `sudo mn -c` は `br0` を含むすべての Open vSwitch ブリッジを削除します。そのあとは `sudo sdnlab flows` でラボを作り直してください。setup とチェックは、VM の代わりのコンテナーで実行して確かめたものです。授業の前に、実際に起動した生徒用 VM で一度試すことをおすすめします。

## 提供元

[Mitsuru Technologies](https://mitsurutech.co.uk/ja) が [MitsuruTech ラーニングプラットフォーム](https://mitsurutech.co.uk/ja/learning)のために作りました。学校向けに、Python、Java、Go、Node、SQL、Linux をブラウザーで学べ、コンテナーラボ、VM ラボ、GitHub からデプロイする Web アプリ、自動採点のクイズがあります。アカウントなしで [プレイグラウンド](https://mitsurutech.co.uk/ja/playground)でコードを試せます。

同じ提供元のほかのラボ: [lab-netlab](https://github.com/mitsurutech/lab-netlab)、[lab-juice-shop](https://github.com/mitsurutech/lab-juice-shop)、[lab-dvwa](https://github.com/mitsurutech/lab-dvwa)、[lab-webgoat](https://github.com/mitsurutech/lab-webgoat)、[lab-mysql-phpmyadmin](https://github.com/mitsurutech/lab-mysql-phpmyadmin)、[lab-git-gitea](https://github.com/mitsurutech/lab-git-gitea)、[lab-sample](https://github.com/mitsurutech/lab-sample)、[lab-reverse-lab](https://github.com/mitsurutech/lab-reverse-lab)、[lab-formal-lab](https://github.com/mitsurutech/lab-formal-lab)、[lab-hdl-lab](https://github.com/mitsurutech/lab-hdl-lab)、[lab-k3s-lab](https://github.com/mitsurutech/lab-k3s-lab)、[lab-tofu-lab](https://github.com/mitsurutech/lab-tofu-lab)、[lab-ansible-roles](https://github.com/mitsurutech/lab-ansible-roles)、[lab-ansible-basics](https://github.com/mitsurutech/lab-ansible-basics)。

## ライセンス

この作品は [CC BY-NC-SA 4.0](https://creativecommons.org/licenses/by-nc-sa/4.0/) でライセンスされています。Mitsuru Technologies Ltd のクレジットを表示し、商用には使わず、派生物を同じライセンスで共有するなら、学習と教育のために共有・改変できます。全文は `LICENSE` ファイルにあります。

[Open vSwitch](https://www.openvswitch.org)、Mininet、os-ken は VM の起動時に Ubuntu のアーカイブからインストールされます。ここではそれらは配布していません。
