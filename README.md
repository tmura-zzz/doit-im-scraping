# Doit.im のデータを export するツール

今更ですが[Doit.im](https://i.doit.im/)のデータを export するツールを作りました。  
かなり前からサポートされなくなってるようだけど Web 版を無料でずっと使い続けてて、
そろそろ乗り換えようと思ったけど公式の export 機能がない。  
[表示内容を印刷するブックマークレット](https://enblog.doit.im/post/16959256088/wanna-print-tasks-in-doitim-you-can-now)があるけど、なるべく情報維持して自作中の ToDo アプリに取り込みたいので目的に合わず..

というわけで、動的サイトの Web スクレイピングの演習がてら即席で自作しました。  
一時的な自分用なので .ipynb です。[Google Colab](https://colab.research.google.com/drive/1SH9A2eizIVgEHVls8vdJ2tSn5u9QWWpR)にも置いてあります。

自分の環境で上手く吐き出せるようにデバッグしただけなので、他の環境で正しく動くか分かりませんが..

# 使い方

## 実行環境のセットアップ

git clone して動かすなら、`pip install requirement.txt`で。  
エラーが出るなら、ipynb 冒頭の !pip install もコメントアウトします。  
arender() に cookie を渡すところのエラーなら、requests-html を github から取り直すのが重要です。  
足りないパッケージは適当に追加してください。

## 環境変数の設定

vscode 等で動かす場合、.env に環境変数を設定してください。

```shell:.env
DOIT_IM_USERNAME="xxxx@yyyyyy.zzz"           # Doit.imのユーザ名
DOIT_IM_PASSWORD="xxxxxxx"                   # Doit.imのパスワード
DOIT_IM_EXPORT_FILE="doit-im-export.json"    # exportするファイル（json形式です）
```

[Google Colab](https://colab.research.google.com/drive/1SH9A2eizIVgEHVls8vdJ2tSn5u9QWWpR)で動かす場合、同じ環境変数をノートブックのシークレットに設定します。

## 実行

あとは .ipynb を通しで実行するだけです。  
データ量に応じて時間かかります。自分の場合、アクティブなタスクが 150 件ぐらいで 約 6 分でした。  
開始時刻と通知設定を取るために個々のタスクのページを巡回しているためです。

## 抽出できる情報

タスク属性のうち、

- id
- title
- time
- repeater
- reminder
- context
- project
- priority
- notes

です。他の属性は個人的に使ってないので取ってません。

タスクの属性情報は以下のビューから取ります。

- #/inbox （収拾箱）
- #/today （今日）
- #/next （次の行動）
- #/tomorrow （明日）
- #/scheduled （スケジュール）
- #/someday （いつか/多分）
- #/waiting （連絡待ち）

以下のビューは名前だけ取ります。

- #/context/\*
- #/project/\*

以下のビューは無視します（時間かかるし量が多いとサーバに負荷もかかるので）。

- #/completed
- #/trash
- その他（上記に書いてないもの）

## 出力形式

適当に定義したデータ構造の json 出力で、以下の形式です（疑似コード）。

```python
{
  unixtime: int,       # exportした時刻 (unix time)
  localtime: string,   # exportした時刻 (local timeの可読文字列)
  views: {             # サイドバーのボタンで切り替えられるタスクリストのフィルタ条件
    view_id: {         # '#/today' とか '#/context/...' とか、フィルタ条件に対してブラウザのURLに表示されているパス
      {
        name: string,  # 「今日」「次の行動」や、プロジェクトやコンテキストの名称など
        contents: {
          [            # タスクをまとめるグループ単位の配列（グループ設定はステータスバーに表示されているデフォルト）
            {
              group-title:  # グループのタイトル（「優先度なし」とか「今週」とか、ビューによって異なる）
              group-size:   # グループに含まれるタスク数
              tasks: {
                id: string,        # タスクのID（タスクのリンクをつつくとURLに表示されるやつ）
                title: string,     # タスクのタイトル
                context: string,   # タスクのコンテキスト
                project: string,   # タスクのプロジェクト
                priority: int,     # タスクの優先度
                notes: string,     # タスクのメモ
                repeater: boolean, # タスクに繰り返し設定がされているか
                time: string,      # タスクの開始時間（分類によっては未定義）
                reminder: [ string, ... ]  # 通知の設定条件（複数あるので配列）
              }
            }, ...
          ]
        }
      }
    }

  }
}

```

出力された json ファイルをご覧ください。  
html から取れる属性値のフォーマットは人間が読むための表示用文字列なので、なかなか自由です。

## エラーで止まる? 情報を取りこぼす?

たぶんレンダリング時間の違いによるタイミング問題かと思います。  
ipynb の get_html() の sleep 値を増やしてみてください。

# コメント歓迎

誰かのお役に立つようであれば、もう少し説明追加します。
