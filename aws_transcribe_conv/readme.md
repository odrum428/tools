## 目的
AWS Transcribeで文字起こしした結果をいい感じに解析して、不要部分を削除するプログラム

## 実施すること
推定結果からメタデータを排除する。
推論された文章が話者ごとに解析されていたので、話者ごとの発言一覧に戻す。
また、実際に話していた時間も残っているので、MTGのタイムラインを作成する。

テキストが入っている箇所 : {"results":{"transcripts":[{"transcript".
一番濃い情報が入っている箇所
{"start_time":"0.57","end_time":"1.44","alternatives":[{"confidence":"0.737","content":"村上"}],"type":"pronunciation"}
話者情報が入っている部分
{"start_time":"0.57","speaker_label":"spk_0","end_time":"4.49","items":[個別の話者データ]}

含まれる要素
- 解析対象が発生していた時間
- 解析結果の信頼性
- 解析内容
- 音声のタイプ

この情報を元に信頼性が一定の確率以下(0.6)のテキストを削除。
start_timeとend_timeから発言をタイムライン形式に並べなおす。
さらに話者が誰かを特定することで、発言者の名前も入れる。
話者の情報が入っている部分は別の要素として用意されている。
さらに、話者が連続する部分はあらかじめ一つの配列要素にまとまっている。
ので、テキストデータを先にまとめ、連続する部分を結合させておく必要性がある。

また、話者データに文章の被りが発生してはいけない。
ので、個別の話者データの最初と最後の要素を取得し、それのstart_time&end_timeで
話者区間の選択を行う。
まず、話者情報のstartとendのみを抜き出して、逆流しているところがないか調べる。
逆流はない。なので、一定区間の話者をそれぞれ分類している。

やりたいこと
- 話者の数が決まっていないときでも柔軟に対応してほしい。
 - 話者が連続するところは一つのラインにまとめる
 - start_timeの順番に会話を並べる

処理
1. 濃い情報が入っているところをlistに区切って、listに格納
2. start_timeの順番に要素をソートする
3. 話者が連続している部分があれば、一つの文にまとめる
2. 話者情報が入っている部分を名寄せする.同じ順番になっていることを前提
2. 全体から信頼率が低い要素を削除する N
4. 不要な部分を削除して、〇〇:はなしたこと の形にする
6. ファイルに書き出す(txtファイル)

## 参考文献
https://qiita.com/shonansurvivors/items/2efcb99dff07b91310c6