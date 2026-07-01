# mic-bridge

営業ロープレ研修（GAS Web App）用の**マイク・ブリッジ**ページ。

## なぜ必要か
GAS の Web App は画面をサンドボックス化された cross-origin iframe で表示し、その iframe には
microphone 権限が委譲されないため、iframe 内では音声認識（Web Speech API）が `not-allowed` で失敗する。
GAS 側のコードではこの iframe 権限を変更できない。

そこで、音声認識だけを**トップレベルのページ**（GitHub Pages）で実行し、認識テキストを
`postMessage` で元の GAS 画面へ返す。ユーザー操作で開いたポップアップは親 iframe の制約を受けず、
自身のオリジンの権限（＝マイク）をそのまま持てる（Google Meet と同じ扱い）。

## セキュリティ設計
- このページは**APIキー・業務ロジック・個人情報を一切持たない**。外部スクリプトも読み込まない自己完結HTML。
- 返信は「利用者自身の発話テキスト」のみ。受信側（GAS）が
  **①送信元オリジン一致 ②送信元ウィンドウ一致 ③ワンタイムnonce一致**の3条件を検証してから受け取る。
- 受信側はテキストを input.value へ代入するだけ（innerHTML/eval不使用）。

## 使い方
GAS 画面から `https://<user>.github.io/mic-bridge/?n=<nonce>` をポップアップで開く。
録音→「確定して戻す」で `{type:'STT_RESULT', nonce, text}` を opener へ postMessage して自動で閉じる。

Chrome 前提。音声は Google の音声認識でテキスト化される（通常の Web Speech API と同じ）。
