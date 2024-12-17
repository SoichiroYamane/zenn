---
title: "Dockerで始めるkeyball用QMK"
emoji: "⌨️"
type: "tech"
topics: ["自作キーボード", "QMK Firmware", "Docker", "QMK", "keyball"]
published: true
---

# はじめに

初めてzennに投稿しますので、何かしら誤植があるかもしれませんが、ご容赦ください。🙌

keyballを使っている方は、remapによってキーマップを変更することができます。しかし、remap上でのキーマップ変更は少しばかり制約が多く、keyballの能力を最大限引き出すためには、QMK Firmwareを利用して詳細な設定を行うことが求められます。QMK Firmwareでは、キーマップの変更だけでなく、トラックボール、LEDの制御やマクロの設定などがC言語を用いて可能になります。しかし、これらの環境を構築するためには少しばかりめんどくさい面や、ローカル環境が汚くなる側面があります。そこで、本記事ではDockerを使用することで、爆速でkeyball用のQMK Firmwareを構築し、ビルドする方法を紹介します。✨

この記事はkeyball39🎱で記事を書いています。
記事の最後に私の使用しているkeyball39🎱のキーマップを公開しています。

## 対象読者

- keyballを使っていて、QMK Firmwareを使いたい人
- QMK Firmwareを使いたいが、環境構築が面倒くさい人
- ローカル環境を汚したくない人
- Dockerを使用してQMK Firmwareを使いたい人

## 本記事の目的

- Dockerを使用して、keyball用のQMK Firmwareを構築し、ビルドする方法を紹介する。外部ツールを利用し、ビルドしたファームウェアをkeyballに書き込む。

## 結論

- Dockerを使用することで、ローカル環境を汚すことなく、keyball用のQMK Firmwareをビルドすることができる。

# 0. 事前準備

Dockerのインストールは完了しているものとします。Windowsではパワーシェル(もしくはコマンドプロンプト)、MacではターミナルでDockerが使えることを確認してください。現在の私のDockerのバージョンは以下の通りです。

```bash
docker --version
```

```bash
Docker version 27.2.0, build 3ab4256
```

また、`docker compose`のコマンドも実行できることを確認してください。

# 1. Dockerファイルのダウンロード

Dockerファイルなどはこちらの[リポジトリ](https://github.com/SoichiroYamane/qmk-docker-keyball)に置いております。ダウンロードをするには、`git clone`でもいいですし、手動で`< > Code` -> `Download ZIP`してから展開でも構いません。

# 2. Dockerイメージのビルドおよびコンテナの起動

ダウンロードしたファイルのディレクトリ(写真ではqmk-testとなっていますが、デフォルトではqmk-docker-keyballです。)に移動し、以下のコマンドを実行してください。実行をすると、写真のようなログが表示されると思います。また、ビルドには時間がかかるので、コーヒーでも飲みながら気長に待ちましょう。

:::message
個人の趣向としてコンテナ内のエディターとしてnvimなどをインストールしていますが、必要ない場合はDockerfileを編集してください。
:::

```bash
docker compose up -d --build
```

![](/images/qmk-docker-keyball-init/qmk-docker-keyball-build.jpg)

ビルドに成功したら、以下のコマンドでコンテナに入ります。

```bash
docker compose exec qmk-docker-keyball fish
```

# 3. keyballのファームウェアのビルド

コンテナに入ると、ホスト側(ローカルのパソコンの環境)のディレクトリにkeyballというディレクトリが作成されていると思います。エクスプローラーもしくはファインダーで確認してください。

```bash
 .
├──  keyball
├──  docker-compose.yml
├──  Dockerfile
├──  LICENSE
├──  Makefile
├──  ocs52_setting.lua
├──  README.md
└──  sync-to-host.fish
```

例として、この中の`keyball/keyball39/keymaps/via`にあるキーマップをビルドをしてみます。keyball39以外の方は、適宜変更してください。コンテナ内で以下のコマンドを実行すると、keyball39のviaキーマップがビルドされます。

```bash
make SKIP_GIT=yes keyball/keyball39:via
```

そうするとホスト側のディレクトリに`build`ディレクトリが作成され、その中にビルドされたファームウェア(`keyball_keyball39_via.hex`)が格納されます。

```bash
 .
├──  build
│   └── 󱊧 keyball_keyball39_via.hex
```

Dockerコンテナ内からは`exit`で抜けることができます。

# 4. ファームウェアの書き込み

ビルドされたファームウェアをkeyballに書き込むために、以下のツールを使用します。

- [QMK Toolbox](https://qmk.fm/toolbox)
- [Pro Micro Web Updater](https://sekigon-gonnoc.github.io/promicro-web-updater/index.html)

詳しくは以下の説明が参考になると思います。

- <https://kbd.dailycraft.jp/claw44/buildguide/10_firmware/toolbox/>
- <https://yushakobo.zendesk.com/hc/ja/articles/1500011696701--QMK%E3%83%95%E3%82%A1%E3%83%BC%E3%83%A0%E3%82%A6%E3%82%A7%E3%82%A2%E3%81%AE%E6%9B%B8%E3%81%8D%E8%BE%BC%E3%81%BF-Pro-Micro-Web-Updater%E5%88%A9%E7%94%A8>

個人的な注意点です。

- リセットボタンは素早く2回押すことでPro Microがブートローダーモードになります。
- Pro Micro Web Updaterで認識されない場合は、リセットボタンを2回を押す。

正しく書き込みができたら、keyballを試してみましょう。🎉
viaは`keyball/keyball39/keymaps/via/rules.mk`に`VIA_ENABLE = yes`が記述されているため、Remapなどのサービスで設定を変更することができます。私のキーマップでは、メモリ不足のためこのVIAを無効にしています。

# 5. キーマップを変更してみよう

`keyball/keyball39/keymaps/via`をコピーして、`keyball/keyball39/keymaps/via_test`という名前で保存しましょう。その後、`keyball/keyball39/keymaps/via_test/keymap.c`を編集して、以下のようにキーマップを変更してみましょう。`KC_W`と`KC_Q`を入れ替えてみました。

```diff c
  [0] = LAYOUT_universal(
-   KC_Q     , KC_W     , KC_E     , KC_R     , KC_T     ,                            KC_Y     , KC_U     , KC_I     , KC_O     , KC_P     ,
+   KC_W     , KC_Q     , KC_E     , KC_R     , KC_T     ,                            KC_Y     , KC_U     , KC_I     , KC_O     , KC_P     ,
    KC_A     , KC_S     , KC_D     , KC_F     , KC_G     ,                            KC_H     , KC_J     , KC_K     , KC_L     , KC_MINS  ,
    KC_Z     , KC_X     , KC_C     , KC_V     , KC_B     ,                            KC_N     , KC_M     , KC_COMM  , KC_DOT   , KC_SLSH  ,
    KC_LCTL  , KC_LGUI  , KC_LALT  ,LSFT_T(KC_LNG2),LT(1,KC_SPC),LT(3,KC_LNG1),KC_BSPC,LT(2,KC_ENT),LSFT_T(KC_LNG2),KC_RALT,KC_RGUI, KC_RSFT
  ),
```

保存をし、もう一度Dockerコンテナ内でビルドを行いましょう。

```bash
docker compose exec qmk-docker-keyball fish
make SKIP_GIT=yes keyball/keyball39:via_test
```

ビルドが成功したら、同様の手順でファームウェアを書き込み試してみましょう。QとWが入れ替わっていることが確認できると思います。さらなるカスタマイズの方法は、QMK Firmwareの公式ドキュメントを参照してください。

- [QMK Firmware](https://docs.qmk.fm/)
- [キーコード](https://docs.qmk.fm/keycodes_basic)
- [マクロ](https://docs.qmk.fm/feature_macros)

# おわりに

Dockerを使用することで、ローカル環境を汚すことなく、keyball用のQMK Firmwareをビルドすることができました。もし問題などがあれば、お気軽にコメント、もしくはgithubでissueを立てていただければと思います。また、記事にいいねや、リポジトリにスターをいただけると励みになります。最後まで読んでいただき、ありがとうございました。🙇

## 私のキーマップ

私のキーマップはこちらの[リポジトリ](https://github.com/SoichiroYamane/keyball-keymap)に置いています。キーマップは`keyball/keyball39/yama`に置いてありますが、keyball用の設定ファイルも変更していますので、一番上の`keyball`ごとコピーして使用していただければと思います。また`dist`以下に`hex`ファイルも置いていますので気軽に試せると思います。参考になれば幸いです。

以下特徴です。

- LEDはLED MATRIXを使用し、ヒートマップの様に色が変化します。
- neovimなどのエディターを使用することを前提にキーマップを設定しています。
- トラックボール操作時を右手のみで操作できるように設定しています。
- ソフトウェアと組み合わせたキーマップを設定しています。

定義したキーコード

- `JP_TOGGLE`: 長押ししている間のみ、日本語入力になる。(タイピング時に変に力が入るので、現在は使用していません。)
- `TMUX_OPWIN`, `TMUX_SPH`, `TMUX_SPV`, `TMUX_CPMOD`, `TMUX_SHELL`: tmuxの操作を行うためのキーコード
- `ENT_IMEVIM`: neovim内でIMEのENTERを押すためのキーコード(修正が必要となっています。)
- `ALT_TAB`: ALTTABをトリガーするためのキーコード
- `HM_TOG`: LED MATRIXのON/OFFを切り替えるためのキーコード
- `KC_S_0`: (と)を続けて高速で入力するためのキーコード
- `LY_TGML`: マウスレイヤーをトグルするためのキーコード
- `ACMD_SP`: yabai用のキーコード

定義したコンボ

- `jk` -> `ESC`
