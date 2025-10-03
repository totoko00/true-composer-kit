# Implementation Plan

このドキュメントは、真の作曲キット（True Composer Kit）の実装タスクリストです。各タスクは段階的に実装され、テスト駆動で進めます。

## Tasks

- [ ] 1. HTMLファイルの基本構造とCSSスタイルの実装
  - 単一HTMLファイル `true-composer-kit.html` を作成
  - CSS変数でダークテーマのカラーパレットを定義
  - 3カラムレスポンシブレイアウトを実装
  - 基本的なコンポーネントスタイル（ボタン、入力、カード）を実装
  - _Requirements: 1.1, 1.2, 11.1, 11.2, 11.3, 11.5_

- [ ] 2. MusicTheory Moduleの実装
  - [ ] 2.1 基本的な音楽理論定数とユーティリティ関数
    - NOTES配列、ENHARMONIC変換、SCALES定義を実装
    - noteNameToMidi()関数を実装
    - midiToNoteName()関数を実装
    - getScaleNotes()関数を実装してスケールの構成音を取得
    - _Requirements: 13.1, 13.2_

  - [ ] 2.2 コード解析機能
    - parseChord()関数を実装してコード名を解析
    - 正規表現でルート音、コードタイプ、拡張を抽出
    - メジャー、マイナー、7th、dim、augなどの基本的なコードタイプをサポート
    - コードの構成音（MIDIノート番号）を計算
    - _Requirements: 2.3, 13.1, 13.2_

  - [ ] 2.3 ダイアトニックコードと機能和声
    - getDiatonicChords()関数を実装
    - Major、Natural Minorのダイアトニックコードを計算
    - getChordFunction()関数を実装してコードの機能（T, SD, D）を判定
    - getRootMotion()関数を実装してコード間のルート移動を計算
    - _Requirements: 4.3, 4.4, 13.3, 13.4_


- [ ] 3. Store Moduleの実装
  - [ ] 3.1 基本的な状態管理
    - 初期プロジェクトデータ構造を定義
    - getProject()関数を実装
    - updateProject()関数を実装
    - Pub/Subパターンでsubscribe()を実装
    - _Requirements: 8.2, 8.3_

  - [ ] 3.2 Undo/Redo機能
    - undoStack、redoStackを実装
    - saveState()関数で状態をスタックに保存
    - undo()関数を実装
    - redo()関数を実装
    - 最大50履歴の制限を実装
    - _Requirements: 9.1, 9.2, 9.3, 9.4, 9.5_

  - [ ] 3.3 コードとメロディの操作
    - setChord()関数を実装してコードを追加/更新
    - addNote()関数を実装してメロディノートを追加
    - deleteNote()関数を実装してノートを削除
    - moveNote()関数を実装してノートを移動
    - _Requirements: 2.2, 2.4, 3.2, 3.3, 3.4, 3.5_

  - [ ] 3.4 プロジェクトの保存/読込
    - exportJSON()関数を実装してプロジェクトをJSON文字列に変換
    - importJSON()関数を実装してJSON文字列からプロジェクトを復元
    - バリデーション機能を実装（スキーマチェック、バージョンチェック）
    - _Requirements: 8.1, 8.4, 8.5, 8.6_

- [ ] 4. UIの基本構造とコントロールパネルの実装
  - [ ] 4.1 左パネル（グローバル設定）
    - Key選択（C〜B、♯/♭含む）のセレクトボックスを実装
    - Scale選択（Major / Natural Minor）のセレクトボックスを実装
    - Tempo入力（BPM）を実装
    - Quantize選択（1/4, 1/8, 1/16）を実装
    - 設定変更時にStore.updateProject()を呼び出し
    - _Requirements: 2.6, 2.7_

  - [ ] 4.2 左パネル（トラックコントロール）
    - Piano、Bass、Drums、Padの各トラックコントロールを実装
    - 音量スライダーを実装
    - ミュート/ソロボタンを実装
    - 自動生成トグル（Bass、Drums）を実装
    - _Requirements: 6.10, 6.11, 6.12, 10.1, 10.2, 10.3_

  - [ ] 4.3 左パネル（エクスポートコントロール）
    - Export WAVボタンを実装（後でAudioEngineと接続）
    - Export MP3ボタンを実装（後でAudioEngineと接続）
    - Export Projectボタンを実装してJSON保存
    - Import Projectボタンを実装してJSON読込
    - ドラッグ&ドロップでJSON読込を実装
    - _Requirements: 7.1, 7.3, 7.7, 8.1, 8.4, 8.5_


- [ ] 5. コード進行グリッドの実装
  - [ ] 5.1 グリッドの描画
    - 16個のコードセルをHTMLで生成
    - 各セルにbar番号を割り当て
    - 空のセルと入力済みセルのスタイルを実装
    - _Requirements: 2.1, 2.4_

  - [ ] 5.2 コード入力機能
    - セルクリックでcontenteditableまたはinput要素を表示
    - Enterキーで入力確定、Escキーでキャンセル
    - 入力されたコード名をStore.setChord()で保存
    - MusicTheory.parseChord()でコードを検証
    - _Requirements: 2.2, 2.3_

  - [ ] 5.3 コード削除機能
    - 右クリックまたはDeleteキーでコードを削除
    - Store経由で状態を更新
    - _Requirements: 2.5_

- [ ] 6. Suggest Moduleの実装（コード候補）
  - [ ] 6.1 基本的なコード候補生成
    - suggestNextChords()関数を実装
    - ダイアトニックコードを候補として生成（基本重み0.4）
    - MusicTheory.getDiatonicChords()を使用
    - _Requirements: 4.1, 4.3, 4.4_

  - [ ] 6.2 カデンツパターンの重み付け
    - V→I（Authentic cadence）の検出と高重み付け（+0.5）
    - IV→I（Plagal cadence）の検出と重み付け（+0.4）
    - ii→Vの検出と重み付け（+0.3）
    - V→vi（Deceptive cadence）の検出と重み付け（+0.2）
    - _Requirements: 4.5_

  - [ ] 6.3 循環進行とボイスリーディングの重み付け
    - vi→ii→V→Iなどの循環進行を検出（+0.3）
    - 4度/5度進行のボーナス（+0.1）
    - 半音接近のボーナス（+0.05）
    - 候補を重み順にソートして最大5件を返す
    - _Requirements: 4.6, 4.7, 4.8_

- [ ] 7. 右パネル（候補表示）の実装
  - [ ] 7.1 コード候補リストの表示
    - Suggest.suggestNextChords()を呼び出し
    - 候補をリスト形式で表示（コード名、理由、重み）
    - 重みを星の数で視覚化
    - _Requirements: 4.1, 4.2_

  - [ ] 7.2 候補クリックでコード挿入
    - 候補アイテムにクリックイベントを設定
    - クリック時に次の小節にコードを挿入
    - Store.setChord()を呼び出し
    - _Requirements: 4.9_

  - [ ] 7.3 ショートカット表示
    - キーボードショートカット一覧を右パネルに表示
    - Space、Cmd/Ctrl+Z、1/2/3などの説明
    - _Requirements: 12.9_


- [ ] 8. ピアノロールの実装
  - [ ] 8.1 Canvas要素とグリッド描画
    - Canvas要素を中央パネルに配置
    - 縦軸に音高（MIDIノート48〜84）を配置
    - 横軸に時間（小節、拍）を配置
    - 背景グリッドを描画（白鍵/黒鍵、拍線）
    - オフスクリーンキャンバスで背景を事前描画
    - _Requirements: 3.1_

  - [ ] 8.2 ノート配置機能
    - Canvasクリックイベントを実装
    - クリック位置から音高とTickを計算
    - Quantize設定に応じてTickをスナップ
    - Store.addNote()でノートを追加
    - _Requirements: 3.2_

  - [ ] 8.3 ノート描画
    - プロジェクトのmelodyトラックからノートを取得
    - 各ノートを矩形で描画
    - 選択中のノートは枠線を強調
    - _Requirements: 3.1_

  - [ ] 8.4 ノート選択と削除
    - ノートクリックで選択状態を切り替え
    - Deleteキーで選択ノートを削除
    - Store.deleteNote()を呼び出し
    - _Requirements: 3.4_

  - [ ] 8.5 ノートのドラッグ編集
    - ノートの右端をドラッグして長さを変更
    - ノート全体をドラッグして位置を移動
    - Quantizeに応じてスナップ
    - Store経由で状態を更新
    - _Requirements: 3.3, 3.5_

- [ ] 9. Suggest Moduleの実装（メロディ候補）
  - [ ] 9.1 基本的なメロディ音候補生成
    - suggestMelodyNotes()関数を実装
    - 現在のコードをMusicTheory.parseChord()で解析
    - コードトーン（1, 3, 5, 7度）を高重み（0.9, 0.8, 0.8, 0.7）で生成
    - _Requirements: 5.1, 5.2_

  - [ ] 9.2 テンションと経過音の候補
    - テンション（9, 11, 13度）を中重み（0.6）で生成
    - スケール内の経過音を低重み（0.5）で生成
    - クロマチックアプローチを最低重み（0.4）で生成
    - _Requirements: 5.2_

  - [ ] 9.3 旋律輪郭の考慮
    - 直前2音の跳躍を計算
    - 大跳躍後は接近進行を優遇（+0.05）
    - 音域外（C4未満、C6超）は重みを減少（-0.3）
    - 候補を重み順にソートして最大7件を返す
    - _Requirements: 5.3, 5.7_

  - [ ] 9.4 メロディ候補リストの表示
    - 右パネルにメロディ音候補を表示
    - 音名、コード内での役割、重みを表示
    - 候補クリックで現在の拍にノートを配置
    - _Requirements: 5.4, 5.5, 5.6, 5.8_


- [ ] 10. Sequencer Moduleの実装
  - [ ] 10.1 Tick/時間変換ユーティリティ
    - tickToSeconds()関数を実装
    - secondsToTick()関数を実装
    - barToTick()関数を実装
    - _Requirements: 6.1_

  - [ ] 10.2 再生スケジューリング
    - play()関数を実装
    - 先読みスケジューリング（100ms先まで）を実装
    - 25msごとのスケジューリングループを実装
    - AudioEngineと連携してノートをスケジュール
    - _Requirements: 6.1, 6.7, 14.1, 14.2_

  - [ ] 10.3 再生制御
    - stop()関数を実装
    - isPlaying()関数を実装
    - getCurrentTick()関数を実装
    - _Requirements: 6.1_

  - [ ] 10.4 再生ヘッドの更新
    - requestAnimationFrameで再生ヘッドを更新
    - UIに現在の再生位置を通知
    - _Requirements: 6.7_

  - [ ] 10.5 ループ機能
    - ループ範囲（startBar, endBar）を設定
    - 再生位置がendBarに達したらstartBarに戻る
    - シームレスなループを実装
    - _Requirements: 6.8_

- [ ] 11. AudioEngine Moduleの実装
  - [ ] 11.1 AudioContextの初期化
    - AudioContextを作成
    - マスターゲインノードを作成
    - ブラウザ互換性チェック（webkitAudioContext）
    - _Requirements: 6.1_

  - [ ] 11.2 Piano音声合成
    - playNote()関数を実装（Piano用）
    - サイン波+三角波のミックス
    - ADSRエンベロープ（A=0.01s, D=0.1s, S=0.7, R=0.3s）
    - 倍音（2倍音）を追加
    - _Requirements: 6.3_

  - [ ] 11.3 Bass音声合成
    - playNote()関数を実装（Bass用）
    - 矩形波オシレーター
    - ローパスフィルタ（cutoff 800Hz）
    - 1オクターブ下で再生
    - _Requirements: 6.4_

  - [ ] 11.4 Drums音声合成
    - Kick: サイン波（周波数減衰150Hz→50Hz）
    - Snare: ホワイトノイズ+ハイパスフィルタ
    - Hat: ホワイトノイズ+バンドパスフィルタ
    - _Requirements: 6.5_

  - [ ] 11.5 Pad音声合成
    - playNote()関数を実装（Pad用）
    - サイン波+三角波+ノコギリ波のミックス
    - ゆっくりしたADSR（A=0.3s, D=0.2s, S=0.8, R=0.5s）
    - _Requirements: 6.3_

  - [ ] 11.6 コード再生機能
    - playChord()関数を実装
    - 複数のノートを同時に再生
    - _Requirements: 6.1_


- [ ] 12. 再生コントロールUIの実装
  - [ ] 12.1 トランスポートコントロール
    - 再生/停止ボタンを実装
    - ループON/OFFトグルを実装
    - メトロノームON/OFFトグルを実装
    - 現在小節の表示を実装
    - _Requirements: 6.7, 6.8, 6.9_

  - [ ] 12.2 再生ヘッドの描画
    - Canvasまたはオーバーレイで再生ヘッドを描画
    - Sequencerから現在位置を取得して更新
    - _Requirements: 6.7_

  - [ ] 12.3 再生機能の統合
    - 再生ボタンクリックでSequencer.play()を呼び出し
    - 停止ボタンクリックでSequencer.stop()を呼び出し
    - 再生中はボタンの状態を変更（アニメーション）
    - _Requirements: 6.1_

- [ ] 13. 自動伴奏機能の実装
  - [ ] 13.1 Bass自動生成
    - 各小節のコードからルート音と5度を抽出
    - 4分音符でルート→5度のパターンを生成
    - 自動生成トグルON時にbassトラックを更新
    - _Requirements: 10.1, 10.4_

  - [ ] 13.2 Drums自動生成
    - 4つ打ちKickパターンを生成（各拍）
    - 2-4拍Snareパターンを生成
    - 8分音符Hatパターンを生成
    - 自動生成トグルON時にdrumsトラックを更新
    - _Requirements: 10.2, 10.4_

  - [ ] 13.3 自動生成トグルの実装
    - トグルOFF時に自動生成トラックをクリア
    - コード進行変更時に自動生成を更新
    - _Requirements: 10.3_

- [ ] 14. キーボードショートカットの実装
  - [ ] 14.1 基本ショートカット
    - Spaceキーで再生/停止を実装
    - Cmd/Ctrl+ZでUndoを実装
    - Cmd/Ctrl+Shift+ZまたはCmd/Ctrl+YでRedoを実装
    - _Requirements: 12.1, 12.2, 12.3_

  - [ ] 14.2 Quantize切替ショートカット
    - 1キーで1/4に切替
    - 2キーで1/8に切替
    - 3キーで1/16に切替
    - _Requirements: 12.4, 12.5, 12.6_

  - [ ] 14.3 ノート編集ショートカット
    - ↑↓キーで選択ノートを半音移動
    - ←→キーで選択ノートの長さを変更
    - 入力フィールドにフォーカスがある場合は無効化
    - _Requirements: 12.7, 12.8, 3.7, 3.8_


- [ ] 15. オフラインレンダリングとWAVエクスポート
  - [ ] 15.1 オフラインレンダリング
    - renderOffline()関数を実装
    - OfflineAudioContextを作成（44.1kHz、ステレオ）
    - プロジェクトの全トラックをスケジュール
    - startRendering()でAudioBufferを取得
    - _Requirements: 7.1, 7.2_

  - [ ] 15.2 WAVエンコーダ
    - audioBufferToWav()関数を実装
    - RIFFヘッダーを生成
    - 16bit PCMデータに変換
    - WAVファイルのArrayBufferを返す
    - _Requirements: 7.1, 7.2_

  - [ ] 15.3 WAVダウンロード機能
    - Export WAVボタンにイベントを設定
    - renderOffline()とaudioBufferToWav()を呼び出し
    - Blobを作成してダウンロード
    - 進行状況を表示
    - _Requirements: 7.1, 7.6, 7.7_

- [ ] 16. MP3エンコーダの実装
  - [ ] 16.1 LameJS最小実装の統合
    - LameJS相当のMP3エンコーダコードを `<script>` に埋め込み
    - モノラルCBR（128kbps）に簡略化
    - MDCTの実装
    - ハフマン符号化の実装
    - _Requirements: 1.2, 7.3, 7.4, 7.5_

  - [ ] 16.2 MP3エンコード関数
    - audioBufferToMp3()関数を実装
    - AudioBufferをFloat32Arrayに変換
    - ステレオをモノラルにミックス
    - LameJSエンコーダを呼び出し
    - MP3バイナリ（Uint8Array）を返す
    - _Requirements: 7.3, 7.4_

  - [ ] 16.3 MP3ダウンロード機能
    - Export MP3ボタンにイベントを設定
    - renderOffline()とaudioBufferToMp3()を呼び出し
    - Blobを作成してダウンロード
    - 進行状況を表示
    - エラー時はWAVエクスポートを提案
    - _Requirements: 7.3, 7.6, 7.7_

- [ ] 17. 初期データとサンプルプロジェクトの実装
  - サンプルコード進行を初期データとして設定
  - C | Am | Dm | G7 | C | F | Dm | G7 | C | C | F | G | Em | Am | Dm | G7
  - キー: C Major、テンポ: 120BPM、16小節
  - メロディトラックは空
  - _Requirements: 15.1, 15.2, 15.3, 15.4_


- [ ] 18. UIの洗練とマイクロインタラクション
  - ボタンホバー時のtransform: translateY(-1px)を実装
  - ボタンホバー時のbox-shadowグロー効果を実装
  - 候補リストのホバー効果を実装
  - 再生中のパルスアニメーションを実装
  - フォーカスインジケータを実装
  - _Requirements: 11.4, 11.7_

- [ ] 19. レスポンシブデザインの調整
  - 画面幅1200px以下でカラム幅を調整
  - 画面幅900px以下で1カラムレイアウトに切り替え
  - モバイルでのタッチ操作を考慮
  - _Requirements: 11.6_

- [ ] 20. エラーハンドリングとバリデーション
  - コード解析エラーのハンドリング
  - AudioContext初期化エラーのハンドリング
  - JSON読み込みエラーのハンドリング
  - ファイルサイズ制限のチェック
  - ユーザーフレンドリーなエラーメッセージ表示
  - _Requirements: 1.1_

- [ ] 21. パフォーマンス最適化
  - Canvas描画のデバウンス/スロットル
  - オフスクリーンキャンバスの活用
  - オーディオノードのプーリング
  - Undoスタックのサイズ制限
  - _Requirements: 14.1, 14.2, 14.3, 14.4, 14.5_

- [ ] 22. 最終テストとデバッグ
  - すべての機能が正常に動作することを確認
  - コード候補が音楽理論的に妥当であることを確認
  - メロディ候補がコードトーン優先で提示されることを確認
  - 再生がカクつかないことを確認
  - WAV/MP3エクスポートが成功することを確認
  - プロジェクトJSON保存/読込が正常に動作することを確認
  - Undo/Redoが正常に動作することを確認
  - キーボードショートカットが正常に動作することを確認
  - 外部ネットワークリクエストが発生しないことを確認（開発者ツールで検証）
  - 複数ブラウザ（Chrome、Firefox、Safari、Edge）で動作確認
  - _Requirements: すべて_

- [ ] 23. コメントとドキュメントの追加
  - 各モジュールの冒頭にJSDocコメントを追加
  - 複雑なアルゴリズムに詳細な説明を追加
  - 音楽理論的な根拠を説明するコメントを追加
  - コードの可読性を最終チェック
  - _Requirements: 1.1_

## 実装の進め方

1. タスクは上から順に実装することを推奨します
2. 各タスクは独立して実装・テストできるように設計されています
3. サブタスクがある場合は、サブタスクを先に完了させてください
4. 各タスク完了後、ブラウザで動作確認を行ってください
5. 問題が発生した場合は、コンソールログでデバッグしてください

## 注意事項

- すべてのコードは単一HTMLファイル内に記述します
- 外部CDN、外部API、外部スクリプトへの依存は禁止です
- ESモジュールは使用せず、即時関数でスコープを管理します
- MP3エンコーダの実装が困難な場合は、WAVエクスポートのみを提供し、MP3は将来の拡張とすることも可能です
