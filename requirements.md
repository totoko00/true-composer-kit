# Requirements Document

## Introduction

真の作曲キット（True Composer Kit）は、音楽理論に基づいたインテリジェントな作曲支援を提供する単一HTMLファイルのシングルページアプリケーションです。ユーザーがコード進行とメロディを入力する際に、音楽理論的に適切な次の候補を提案することで、音楽理論の知識が浅いユーザーでも理論的に破綻しにくい楽曲を作成できるようにします。外部CDNや外部APIへの依存を一切排除し、完全にオフラインで動作する自己完結型のアプリケーションです。

## Requirements

### Requirement 1: 単一HTMLファイルでの完全動作

**User Story:** アプリケーション利用者として、外部ネットワーク接続なしでローカル環境で完全に動作するアプリケーションを使いたい。そうすることで、インターネット環境に依存せず、どこでも作曲作業ができる。

#### Acceptance Criteria

1. WHEN ユーザーが `true-composer-kit.html` をブラウザで開く THEN アプリケーションは外部CDN、外部API、外部スクリプトへのネットワークリクエストなしで完全に動作する SHALL
2. WHEN アプリケーションが読み込まれる THEN すべてのCSS、JavaScript、音声エンコードライブラリ（LameJS相当）が単一HTMLファイル内に埋め込まれている SHALL
3. WHEN ブラウザの開発者ツールでネットワークタブを確認する THEN HTMLファイル自体の読み込み以外のネットワークリクエストが発生しない SHALL
4. WHEN アプリケーションを使用する THEN ESモジュールを使用せず、すべてのコードが単一ファイル内で完結している SHALL

### Requirement 2: コード進行エディタ

**User Story:** 作曲者として、タイムライン形式でコード進行を入力・編集したい。そうすることで、楽曲の和声構造を視覚的に構築できる。

#### Acceptance Criteria

1. WHEN ユーザーがコード進行トラックを表示する THEN 小節単位のグリッド（デフォルト16小節）が表示される SHALL
2. WHEN ユーザーがコードセルをクリックする THEN コード名（C, Am7, G7, Dm7b5など）を入力できるテキスト入力フィールドが表示される SHALL
3. WHEN ユーザーがコード名を入力する THEN 一般的なコード表記（メジャー、マイナー、7th、m7、dim、augなど）が解析され保存される SHALL
4. WHEN ユーザーがコードを入力する THEN そのコードが視覚的にグリッド上に表示される SHALL
5. WHEN ユーザーがコードセルを右クリックまたは削除キーを押す THEN そのコードが削除される SHALL
6. WHEN ユーザーがグローバル設定でキー（C〜B、♯/♭含む）を変更する THEN コード候補の提案がそのキーに基づいて更新される SHALL
7. WHEN ユーザーがスケール（Major / Natural Minor / Dorian等）を変更する THEN コード候補の提案がそのスケールに基づいて更新される SHALL

### Requirement 3: メロディエディタ

**User Story:** 作曲者として、ピアノロール風のインターフェースでメロディを入力・編集したい。そうすることで、直感的に旋律を作成できる。

#### Acceptance Criteria

1. WHEN ユーザーがメロディトラックを表示する THEN 縦軸が音高（MIDIノート番号48〜84程度）、横軸が時間のピアノロールが表示される SHALL
2. WHEN ユーザーがピアノロール上をクリックする THEN その位置にノートが配置される SHALL
3. WHEN ユーザーがノートの右端をドラッグする THEN ノートの長さが変更される SHALL
4. WHEN ユーザーがノートを選択してDeleteキーを押す THEN そのノートが削除される SHALL
5. WHEN ユーザーがノートをドラッグする THEN ノートの位置（音高と時間）が移動される SHALL
6. WHEN ユーザーがクオンタイズ設定（1/4, 1/8, 1/16）を変更する THEN 新規配置されるノートがその解像度にスナップされる SHALL
7. WHEN ユーザーが選択したノートに対して↑↓キーを押す THEN ノートが半音単位で上下に移動する SHALL
8. WHEN ユーザーが選択したノートに対して←→キーを押す THEN ノートの長さがクオンタイズ単位で増減する SHALL

### Requirement 4: インテリジェントなコード候補提案

**User Story:** 音楽理論の知識が浅い作曲者として、現在のコード進行に基づいて次に入れるべきコードの候補を提案してほしい。そうすることで、音楽理論的に自然なコード進行を作成できる。

#### Acceptance Criteria

1. WHEN ユーザーがコード進行を入力している THEN 次に入れるべきコード候補が重み順にリスト表示される SHALL
2. WHEN コード候補が表示される THEN 各候補にその理由（機能和声、カデンツ、循環、セカンダリドミナント等）が注釈として表示される SHALL
3. WHEN 現在のキーがMajorである THEN ダイアトニックコード（I, ii, iii, IV, V, vi, vii°）が候補に含まれる SHALL
4. WHEN 現在のキーがNatural Minorである THEN ダイアトニックコード（i, ii°, III, iv, v, VI, VII）が候補に含まれる SHALL
5. WHEN 強カデンツ（V→I）の文脈である THEN その候補の重みが0.9程度の高い値になる SHALL
6. WHEN 循環進行（vi→ii→V→I等）の文脈である THEN その候補の重みが0.7程度になる SHALL
7. WHEN 前のコードとのルート移動が4度/5度である THEN その候補の重みに+0.1のボーナスが付与される SHALL
8. WHEN コード候補が表示される THEN 最大5件の候補が表示される SHALL
9. WHEN ユーザーがコード候補をクリックする THEN そのコードが次の小節に自動的に挿入される SHALL

### Requirement 5: インテリジェントなメロディ音候補提案

**User Story:** 作曲者として、現在のコード上で使えるメロディ音の候補を提案してほしい。そうすることで、コードに対して不協和な音を避け、音楽的に適切なメロディを作成できる。

#### Acceptance Criteria

1. WHEN ユーザーがメロディを入力している THEN 現在のコード上で使えるメロディ音候補がリスト表示される SHALL
2. WHEN メロディ音候補が表示される THEN コードトーン（1, 3, 5, 7度）が最も高い重み（0.9, 0.8, 0.8, 0.7）で表示される SHALL
3. WHEN メロディ音候補が表示される THEN テンション（9, 11, 13度等）が中程度の重み（0.6）で表示される SHALL
4. WHEN メロディ音候補が表示される THEN ダイアトニックスケール内の経過音が低めの重み（0.5）で表示される SHALL
5. WHEN 直前2音の跳躍が大きい THEN 次音として接近進行する音の重みに+0.05のボーナスが付与される SHALL
6. WHEN メロディ音候補が表示される THEN 各候補に音名とコード内での役割（例：「E (3rd)」）が表示される SHALL
7. WHEN メロディ音候補が表示される THEN 最大7件の候補が表示される SHALL
8. WHEN ユーザーがメロディ音候補をクリックする THEN その音が現在の拍に配置される SHALL

### Requirement 6: 音声再生機能

**User Story:** 作曲者として、入力したコード進行とメロディを再生して確認したい。そうすることで、作成中の楽曲を聴覚的に評価できる。

#### Acceptance Criteria

1. WHEN ユーザーが再生ボタンをクリックする THEN Web Audio APIを使用して楽曲が再生される SHALL
2. WHEN 楽曲が再生される THEN Piano、Bass、Drums、Padの最低4つの楽器トラックが再生される SHALL
3. WHEN Pianoトラックが再生される THEN 減衰付きのサイン/三角/矩形波の合成音とADSRエンベロープが使用される SHALL
4. WHEN Bassトラックが再生される THEN 矩形波とローパスフィルタで厚みのある音が生成される SHALL
5. WHEN Drumsトラックが再生される THEN Kick（減衰サイン波）、Snare（ノイズ+HPF）、Hat（ノイズ+バンドパス）の簡易合成音が使用される SHALL
6. WHEN ユーザーがSpaceキーを押す THEN 再生が開始または停止される SHALL
7. WHEN 楽曲が再生される THEN 再生ヘッドがタイムライン上を移動し、現在位置が視覚的に表示される SHALL
8. WHEN ユーザーがループ機能を有効にする THEN 指定範囲（デフォルトは全16小節）が繰り返し再生される SHALL
9. WHEN ユーザーがメトロノームをONにする THEN クリック音が拍に合わせて再生される SHALL
10. WHEN ユーザーが各トラックの音量を調整する THEN その音量が再生に反映される SHALL
11. WHEN ユーザーがトラックをミュートする THEN そのトラックが再生されない SHALL
12. WHEN ユーザーがトラックをソロにする THEN そのトラックのみが再生される SHALL

### Requirement 7: オーディオエクスポート機能

**User Story:** 作曲者として、作成した楽曲をWAVまたはMP3ファイルとしてエクスポートしたい。そうすることで、他のアプリケーションで使用したり、共有したりできる。

#### Acceptance Criteria

1. WHEN ユーザーが「Export WAV」ボタンをクリックする THEN OfflineAudioContextを使用して楽曲がレンダリングされ、WAVファイルとしてダウンロードされる SHALL
2. WHEN WAVファイルがエクスポートされる THEN 44.1kHzのサンプルレートで出力される SHALL
3. WHEN ユーザーが「Export MP3」ボタンをクリックする THEN LameJS相当のMP3エンコーダを使用して楽曲がMP3ファイルとしてダウンロードされる SHALL
4. WHEN MP3ファイルがエクスポートされる THEN 44.1kHz、128kbps程度のビットレートで出力される SHALL
5. WHEN MP3エンコーダが単一HTMLファイルに埋め込まれる THEN LameJS相当の最小実装（最低限モノラルCBR）が `<script>` タグ内に含まれる SHALL
6. WHEN エクスポート処理が実行される THEN ユーザーに進行状況が視覚的に表示される SHALL
7. WHEN エクスポートが完了する THEN ファイルが自動的にダウンロードされる SHALL

### Requirement 8: プロジェクト保存・読込機能

**User Story:** 作曲者として、作成中のプロジェクトをJSON形式で保存し、後で読み込みたい。そうすることで、作業を中断して後で再開できる。

#### Acceptance Criteria

1. WHEN ユーザーが「Export Project」ボタンをクリックする THEN 現在のプロジェクト状態（コード進行、メロディ、設定等）がJSON形式でダウンロードされる SHALL
2. WHEN プロジェクトJSONがエクスポートされる THEN キー、スケール、テンポ、拍子、クオンタイズ設定が含まれる SHALL
3. WHEN プロジェクトJSONがエクスポートされる THEN すべてのトラックデータ（chord, melody, bass, drums）が含まれる SHALL
4. WHEN ユーザーが「Import Project」ボタンをクリックしてJSONファイルを選択する THEN そのプロジェクトが読み込まれ、UIに反映される SHALL
5. WHEN ユーザーがJSONファイルをアプリケーション上にドラッグ&ドロップする THEN そのプロジェクトが読み込まれる SHALL
6. WHEN プロジェクトが読み込まれる THEN すべての設定とトラックデータが正確に復元される SHALL

### Requirement 9: Undo/Redo機能

**User Story:** 作曲者として、操作を取り消したりやり直したりしたい。そうすることで、試行錯誤しながら安心して作曲できる。

#### Acceptance Criteria

1. WHEN ユーザーがCmd/Ctrl+Zを押す THEN 直前の操作が取り消される SHALL
2. WHEN ユーザーがCmd/Ctrl+Shift+Zを押す THEN 取り消した操作がやり直される SHALL
3. WHEN Undo/Redoが実行される THEN コード進行、メロディ、設定の変更が対象となる SHALL
4. WHEN Undo/Redoが実行される THEN UIが即座に更新される SHALL
5. WHEN 新しい操作が実行される THEN Redoスタックがクリアされる SHALL

### Requirement 10: 自動伴奏機能

**User Story:** 作曲者として、コード進行に基づいてBassとDrumsを自動生成したい。そうすることで、手早く楽曲の骨格を作成できる。

#### Acceptance Criteria

1. WHEN ユーザーがBass自動生成トグルをONにする THEN 各小節のコードのルート音と5度の音でBassラインが自動生成される SHALL
2. WHEN ユーザーがDrums自動生成トグルをONにする THEN 4つ打ちKick、2-4拍Snare、8分音符Hatの固定パターンが生成される SHALL
3. WHEN 自動生成トグルがOFFになる THEN 自動生成されたトラックがクリアされる SHALL
4. WHEN コード進行が変更される AND 自動生成がONである THEN Bassラインが自動的に更新される SHALL

### Requirement 11: モダンで美しいUI

**User Story:** アプリケーション利用者として、モダンで美しい「音楽スタジオ」風のUIを使いたい。そうすることで、快適で楽しい作曲体験ができる。

#### Acceptance Criteria

1. WHEN アプリケーションが表示される THEN ダーク基調のカラースキームが使用される SHALL
2. WHEN アプリケーションが表示される THEN アクセントカラーとして微光（ネオン風）の色が使用される SHALL
3. WHEN アプリケーションが表示される THEN 3カラムレイアウト（左：コントロール、中央：エディタ、右：候補）が表示される SHALL
4. WHEN ユーザーがボタンやコントロールにホバーする THEN box-shadow、transform: translateY等のマイクロインタラクションが発生する SHALL
5. WHEN アプリケーションが表示される THEN Tailwind風の余白・角丸の雰囲気がCSSで再現される SHALL
6. WHEN アプリケーションが異なる画面サイズで表示される THEN レスポンシブに適切にレイアウトが調整される SHALL
7. WHEN ユーザーがフォーカス可能な要素にフォーカスする THEN 視覚的なフォーカスインジケータが表示される SHALL

### Requirement 12: キーボードショートカット

**User Story:** 作曲者として、キーボードショートカットで効率的に操作したい。そうすることで、マウス操作を減らして作曲に集中できる。

#### Acceptance Criteria

1. WHEN ユーザーがSpaceキーを押す THEN 再生が開始または停止される SHALL
2. WHEN ユーザーがCmd/Ctrl+Zを押す THEN Undoが実行される SHALL
3. WHEN ユーザーがCmd/Ctrl+Shift+ZまたはCmd/Ctrl+Yを押す THEN Redoが実行される SHALL
4. WHEN ユーザーが1キーを押す THEN クオンタイズが1/4に設定される SHALL
5. WHEN ユーザーが2キーを押す THEN クオンタイズが1/8に設定される SHALL
6. WHEN ユーザーが3キーを押す THEN クオンタイズが1/16に設定される SHALL
7. WHEN ユーザーが↑↓キーを押す AND ノートが選択されている THEN 選択ノートが半音単位で移動する SHALL
8. WHEN ユーザーが←→キーを押す AND ノートが選択されている THEN 選択ノートの長さがクオンタイズ単位で増減する SHALL
9. WHEN アプリケーションが表示される THEN ショートカット一覧が右パネルに表示される SHALL

### Requirement 13: コード解析と音楽理論エンジン

**User Story:** システムとして、入力されたコード名を正確に解析し、音楽理論に基づいた適切な候補を生成したい。そうすることで、ユーザーに有用な提案ができる。

#### Acceptance Criteria

1. WHEN コード名が入力される THEN メジャー、マイナー、7th、m7、maj7、dim、dim7、aug、sus4、sus2等の一般的な表記が解析される SHALL
2. WHEN コード名が解析される THEN ルート音、コードタイプ、構成音（MIDIノート番号）が抽出される SHALL
3. WHEN キーとスケールが設定される THEN そのキーのダイアトニックコードが計算される SHALL
4. WHEN 次のコード候補が生成される THEN 機能和声理論（T, SD, D）に基づいた重み付けが行われる SHALL
5. WHEN 次のコード候補が生成される THEN 五度圏に基づいた循環進行が考慮される SHALL
6. WHEN 次のコード候補が生成される THEN セカンダリドミナント（V/ii, V/iii等）が適切な文脈で提案される SHALL
7. WHEN メロディ音候補が生成される THEN 現在のコードの構成音が分析され、コードトーンとテンションが識別される SHALL
8. WHEN メロディ音候補が生成される THEN 旋律輪郭（跳躍後の接近進行等）が考慮される SHALL

### Requirement 14: パフォーマンスと安定性

**User Story:** アプリケーション利用者として、カクつきやフリーズのない安定した動作を期待する。そうすることで、ストレスなく作曲作業ができる。

#### Acceptance Criteria

1. WHEN 楽曲が再生される THEN 音声がカクつかず滑らかに再生される SHALL
2. WHEN 楽曲が再生される THEN Web Audio APIのスケジューラが適切に使用され、タイミングが正確である SHALL
3. WHEN ユーザーがUIを操作する THEN レスポンスが即座に返される（100ms以内）SHALL
4. WHEN 大量のノート（100個以上）が配置される THEN UIの描画パフォーマンスが維持される SHALL
5. WHEN エクスポート処理が実行される THEN メインスレッドがブロックされず、UIが応答性を保つ SHALL

### Requirement 15: 初期データとサンプル

**User Story:** 初めてアプリケーションを使うユーザーとして、すぐに試せるサンプルデータが用意されていてほしい。そうすることで、アプリケーションの機能を素早く理解できる。

#### Acceptance Criteria

1. WHEN アプリケーションが初めて読み込まれる THEN キーがC Major、テンポが120BPM、16小節に設定される SHALL
2. WHEN アプリケーションが初めて読み込まれる THEN サンプルのコード進行（C | Am | Dm | G7 | C | F | Dm | G7 | C | C | F | G | Em | Am | Dm | G7）が配置される SHALL
3. WHEN アプリケーションが初めて読み込まれる THEN メロディトラックは空の状態である SHALL
4. WHEN ユーザーがサンプルを再生する THEN 正常に音が鳴る SHALL
