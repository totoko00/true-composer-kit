# Design Document

## Overview

真の作曲キット（True Composer Kit）は、音楽理論に基づいたインテリジェントな作曲支援を提供する単一HTMLファイルのシングルページアプリケーションです。本設計では、完全にオフラインで動作する自己完結型のアーキテクチャを採用し、Web Audio APIによる音声合成、音楽理論エンジンによる候補提案、直感的なUIを統合します。

### 設計原則

1. **単一ファイル完結**: すべてのコード、スタイル、ライブラリを1つのHTMLファイルに統合
2. **疎結合アーキテクチャ**: 音楽理論、オーディオ、UI、データ管理を独立したモジュールとして設計
3. **リアルタイム性**: ユーザー操作に対する即座のフィードバック（100ms以内）
4. **拡張性**: 将来的な機能追加（新しいスケール、コードタイプ、楽器等）を容易にする設計
5. **音楽理論の正確性**: 機能和声、五度圏、ボイスリーディングの原則に忠実

## Architecture

### 全体構成

```
┌─────────────────────────────────────────────────────────────┐
│                    true-composer-kit.html                    │
├─────────────────────────────────────────────────────────────┤
│  <style>                                                     │
│    - CSS Variables (Dark Theme)                             │
│    - Layout (3-column responsive)                           │
│    - Components (buttons, grids, piano roll)                │
│    - Animations & Micro-interactions                        │
├─────────────────────────────────────────────────────────────┤
│  <div id="app">                                             │
│    ├── Left Panel (Controls)                               │
│    ├── Center Panel (Timeline & Editors)                   │
│    └── Right Panel (Suggestions & Help)                    │
├─────────────────────────────────────────────────────────────┤
│  <script>                                                    │
│    ├── MusicTheory Module                                   │
│    ├── Suggest Module                                       │
│    ├── Sequencer Module                                     │
│    ├── AudioEngine Module                                   │
│    ├── UI Module                                            │
│    ├── Store Module                                         │
│    └── LameJS MP3 Encoder                                   │
└─────────────────────────────────────────────────────────────┘
```


### モジュール間の依存関係

```
UI Module
  ├─→ Store Module (state management)
  ├─→ Sequencer Module (playback control)
  ├─→ Suggest Module (get suggestions)
  └─→ AudioEngine Module (export audio)

Suggest Module
  └─→ MusicTheory Module (chord/scale analysis)

Sequencer Module
  ├─→ MusicTheory Module (chord parsing)
  ├─→ AudioEngine Module (audio synthesis)
  └─→ Store Module (read project data)

AudioEngine Module
  ├─→ MusicTheory Module (chord to notes)
  └─→ LameJS Encoder (MP3 export)

Store Module
  └─→ UI Module (trigger re-render)
```

### データフロー

1. **ユーザー入力** → UI Module → Store Module (state update) → UI Module (re-render)
2. **候補リクエスト** → UI Module → Suggest Module → MusicTheory Module → UI Module (display)
3. **再生** → UI Module → Sequencer Module → AudioEngine Module → Web Audio API
4. **エクスポート** → UI Module → AudioEngine Module → LameJS/WAV Encoder → Download

## Components and Interfaces

### 1. MusicTheory Module

音楽理論の計算とコード解析を担当する中核モジュール。

#### 主要機能

- キーとスケールの定義
- ダイアトニックコードの計算
- コード名の解析（文字列 → 構成音）
- 五度圏の計算
- 機能和声の判定（T, SD, D）
- ボイスリーディングの評価


#### インターフェース

```javascript
/**
 * キーとスケールからダイアトニックコードを取得
 * @param {string} key - "C", "G", "F#" など
 * @param {string} scale - "Major", "Natural Minor", "Dorian" など
 * @returns {Array<{degree: string, name: string, function: string}>}
 */
function getDiatonicChords(key, scale)

/**
 * コード名を解析して構成音を取得
 * @param {string} chordName - "C", "Am7", "G7", "Dm7b5" など
 * @returns {Object} {root: number, type: string, notes: Array<number>}
 */
function parseChord(chordName)

/**
 * 2つのコード間のルート移動を計算
 * @param {string} chord1 - 前のコード
 * @param {string} chord2 - 次のコード
 * @returns {number} 半音単位の移動距離（-11〜11）
 */
function getRootMotion(chord1, chord2)

/**
 * コードの機能（T, SD, D）を判定
 * @param {string} chordName - コード名
 * @param {string} key - キー
 * @param {string} scale - スケール
 * @returns {string} "T" | "SD" | "D" | "other"
 */
function getChordFunction(chordName, key, scale)

/**
 * スケールの構成音を取得
 * @param {string} key - キー
 * @param {string} scale - スケール
 * @returns {Array<number>} MIDIノート番号の配列（1オクターブ分）
 */
function getScaleNotes(key, scale)

/**
 * 音名からMIDIノート番号を取得（C4 = 60）
 * @param {string} noteName - "C", "C#", "Db" など
 * @param {number} octave - オクターブ番号
 * @returns {number} MIDIノート番号
 */
function noteNameToMidi(noteName, octave)

/**
 * MIDIノート番号から音名を取得
 * @param {number} midi - MIDIノート番号
 * @param {boolean} preferSharps - シャープ表記を優先するか
 * @returns {string} 音名（"C4", "C#4" など）
 */
function midiToNoteName(midi, preferSharps)
```


#### 実装詳細

**キーとスケールの定義**

```javascript
const NOTES = ["C", "C#", "D", "D#", "E", "F", "F#", "G", "G#", "A", "A#", "B"];
const ENHARMONIC = { "Db": "C#", "Eb": "D#", "Gb": "F#", "Ab": "G#", "Bb": "A#" };

const SCALES = {
  "Major": [0, 2, 4, 5, 7, 9, 11],           // W-W-H-W-W-W-H
  "Natural Minor": [0, 2, 3, 5, 7, 8, 10],   // W-H-W-W-H-W-W
  "Dorian": [0, 2, 3, 5, 7, 9, 10],          // W-H-W-W-W-H-W
  "Phrygian": [0, 1, 3, 5, 7, 8, 10],        // H-W-W-W-H-W-W
  "Lydian": [0, 2, 4, 6, 7, 9, 11],          // W-W-W-H-W-W-H
  "Mixolydian": [0, 2, 4, 5, 7, 9, 10],      // W-W-H-W-W-H-W
  "Locrian": [0, 1, 3, 5, 6, 8, 10]          // H-W-W-H-W-W-W
};

const CHORD_QUALITIES = {
  "Major": { "I": "", "ii": "m", "iii": "m", "IV": "", "V": "", "vi": "m", "vii": "dim" },
  "Natural Minor": { "i": "m", "ii": "dim", "III": "", "iv": "m", "v": "m", "VI": "", "VII": "" }
};
```

**コード解析のパターンマッチング**

正規表現を使用してコード名を解析：
- ルート音: `[A-G][#b]?`
- コードタイプ: `(m|maj|dim|aug|sus)?`
- 拡張: `(7|9|11|13)?`
- 変化音: `(b5|#5|b9|#9|#11)?`

例: "Dm7b5" → Root: D, Type: m7b5 → Notes: [D, F, Ab, C]


### 2. Suggest Module

音楽理論に基づいて次のコードやメロディ音の候補を提案するモジュール。

#### 主要機能

- 次のコード候補の生成（重み付き）
- メロディ音候補の生成（重み付き）
- カデンツパターンの認識
- 循環進行の検出
- セカンダリドミナントの提案

#### インターフェース

```javascript
/**
 * 次のコード候補を提案
 * @param {string} currentChord - 現在のコード（nullの場合は最初のコード）
 * @param {string} key - キー
 * @param {string} scale - スケール
 * @param {Array<string>} previousChords - 直前のコード履歴（最大3つ）
 * @returns {Array<{name: string, reason: string, weight: number}>}
 */
function suggestNextChords(currentChord, key, scale, previousChords)

/**
 * メロディ音候補を提案
 * @param {string} currentChord - 現在のコード
 * @param {Array<{midi: number, startTick: number}>} previousNotes - 直前のノート（最大3つ）
 * @param {string} key - キー
 * @param {string} scale - スケール
 * @returns {Array<{midi: number, label: string, weight: number, tip: string}>}
 */
function suggestMelodyNotes(currentChord, previousNotes, key, scale)
```


#### 実装詳細

**コード候補の重み付けアルゴリズム**

1. **ベース重み**: ダイアトニックコード = 0.4
2. **カデンツボーナス**:
   - V → I: +0.5 (0.9)
   - IV → I: +0.4 (0.8)
   - ii → V: +0.3 (0.7)
   - V → vi (deceptive): +0.2 (0.6)
3. **循環進行ボーナス**:
   - vi → ii → V → I の流れ: +0.3
   - iii → vi → ii → V の流れ: +0.2
4. **ボイスリーディングボーナス**:
   - 4度/5度進行: +0.1
   - 半音接近: +0.05
5. **セカンダリドミナント**:
   - V/ii, V/iii, V/IV, V/V, V/vi: 基本重み 0.5

**メロディ音候補の重み付けアルゴリズム**

1. **コードトーン**:
   - Root (1度): 0.9
   - 3rd: 0.8
   - 5th: 0.8
   - 7th: 0.7
2. **テンション**:
   - 9th, 13th: 0.6
   - 11th (avoid noteの場合は低く): 0.5
3. **スケール内経過音**: 0.5
4. **クロマチックアプローチ**: 0.4
5. **旋律輪郭ボーナス**:
   - 直前が大跳躍（5度以上）→ 接近進行: +0.05
   - 音域外（C4未満、C6超）: -0.3

**候補の最大数**

- コード候補: 最大5件
- メロディ音候補: 最大7件（オクターブ内で重複を避ける）


### 3. Sequencer Module

タイミング管理、再生制御、オフラインレンダリングを担当するモジュール。

#### 主要機能

- Tick ⇔ 時間の変換
- 再生スケジューリング（Web Audio API）
- 再生ヘッドの管理
- ループ機能
- メトロノーム

#### インターフェース

```javascript
/**
 * プロジェクトを再生
 * @param {Object} project - プロジェクトデータ
 * @param {number} startBar - 開始小節（0-indexed）
 * @param {boolean} loop - ループするか
 * @returns {Promise<void>}
 */
function play(project, startBar, loop)

/**
 * 再生を停止
 */
function stop()

/**
 * 再生中かどうか
 * @returns {boolean}
 */
function isPlaying()

/**
 * 現在の再生位置（tick）を取得
 * @returns {number}
 */
function getCurrentTick()

/**
 * Tickを秒に変換
 * @param {number} tick - Tick値
 * @param {number} tempo - BPM
 * @param {number} ppq - Pulses Per Quarter note
 * @returns {number} 秒
 */
function tickToSeconds(tick, tempo, ppq)

/**
 * 秒をTickに変換
 * @param {number} seconds - 秒
 * @param {number} tempo - BPM
 * @param {number} ppq - Pulses Per Quarter note
 * @returns {number} Tick値
 */
function secondsToTick(seconds, tempo, ppq)

/**
 * 小節をTickに変換
 * @param {number} bar - 小節番号（0-indexed）
 * @param {string} timeSignature - 拍子（"4/4"）
 * @param {number} ppq - Pulses Per Quarter note
 * @returns {number} Tick値
 */
function barToTick(bar, timeSignature, ppq)
```


#### 実装詳細

**スケジューリング戦略**

Web Audio APIの高精度タイミングを活用するため、「先読みスケジューリング」を採用：

1. **スケジューリングループ**: 25msごとに実行
2. **先読み時間**: 100ms先までのイベントをスケジュール
3. **イベントキュー**: 再生すべきノートをソート済みキューで管理

```javascript
// 疑似コード
function scheduleLoop() {
  const currentTime = audioContext.currentTime;
  const lookAhead = 0.1; // 100ms
  
  while (nextNoteTime < currentTime + lookAhead) {
    scheduleNote(nextNote, nextNoteTime);
    advanceNote();
  }
  
  setTimeout(scheduleLoop, 25);
}
```

**再生ヘッドの更新**

- requestAnimationFrame を使用してUIの再生ヘッドを滑らかに更新
- 実際の音声タイミングとは独立して描画

**ループ機能**

- ループ範囲（startBar, endBar）を設定
- 再生位置がendBarに達したらstartBarに戻る
- シームレスなループのため、次のループの音をあらかじめスケジュール


### 4. AudioEngine Module

音声合成、エフェクト、エクスポートを担当するモジュール。

#### 主要機能

- 楽器ごとの音声合成（Piano, Bass, Pad, Drums）
- ADSRエンベロープ
- フィルター（LPF, HPF, BPF）
- オフラインレンダリング
- WAV/MP3エクスポート

#### インターフェース

```javascript
/**
 * 音を再生
 * @param {string} instrument - "piano" | "bass" | "pad" | "kick" | "snare" | "hat"
 * @param {number} midi - MIDIノート番号（Drumsの場合は無視）
 * @param {number} startTime - 開始時刻（AudioContext時間）
 * @param {number} duration - 長さ（秒）
 * @param {number} velocity - ベロシティ（0.0〜1.0）
 * @param {AudioContext} audioContext - 使用するAudioContext
 * @param {GainNode} destination - 出力先
 */
function playNote(instrument, midi, startTime, duration, velocity, audioContext, destination)

/**
 * コードを再生（複数音）
 * @param {string} instrument - 楽器名
 * @param {Array<number>} midiNotes - MIDIノート番号の配列
 * @param {number} startTime - 開始時刻
 * @param {number} duration - 長さ
 * @param {number} velocity - ベロシティ
 * @param {AudioContext} audioContext
 * @param {GainNode} destination
 */
function playChord(instrument, midiNotes, startTime, duration, velocity, audioContext, destination)

/**
 * プロジェクトをオフラインレンダリング
 * @param {Object} project - プロジェクトデータ
 * @returns {Promise<AudioBuffer>}
 */
function renderOffline(project)

/**
 * AudioBufferをWAVファイルに変換
 * @param {AudioBuffer} audioBuffer
 * @returns {ArrayBuffer} WAVファイルのバイナリデータ
 */
function audioBufferToWav(audioBuffer)

/**
 * AudioBufferをMP3ファイルに変換
 * @param {AudioBuffer} audioBuffer
 * @returns {Uint8Array} MP3ファイルのバイナリデータ
 */
function audioBufferToMp3(audioBuffer)
```


#### 実装詳細

**楽器の音声合成**

1. **Piano**:
   - オシレーター: サイン波 + 三角波（ミックス）
   - ADSR: A=0.01s, D=0.1s, S=0.7, R=0.3s
   - 倍音: 基音 + 2倍音（小音量）で豊かさを追加

2. **Bass**:
   - オシレーター: 矩形波（duty cycle 0.5）
   - フィルター: ローパスフィルタ（cutoff 800Hz, Q=2）
   - ADSR: A=0.01s, D=0.05s, S=0.9, R=0.1s
   - 1オクターブ下で再生

3. **Pad**:
   - オシレーター: サイン波 + 三角波 + ノコギリ波（ミックス）
   - ADSR: A=0.3s, D=0.2s, S=0.8, R=0.5s（ゆっくり）
   - リバーブ風: 短いディレイを複数重ねる

4. **Kick**:
   - オシレーター: サイン波（周波数を150Hz→50Hzに急速減衰）
   - エンベロープ: 急速減衰（0.5s）
   - ピッチエンベロープで「ドン」という音を再現

5. **Snare**:
   - ノイズジェネレーター（ホワイトノイズ）
   - ハイパスフィルタ（cutoff 1000Hz）
   - エンベロープ: 急速減衰（0.2s）
   - サイン波（200Hz）を少量ミックス

6. **Hat**:
   - ノイズジェネレーター
   - バンドパスフィルタ（center 8000Hz, Q=1）
   - エンベロープ: 非常に短い減衰（0.1s）

**ADSRエンベロープの実装**

```javascript
function applyADSR(gainNode, startTime, duration, velocity, adsr) {
  const { attack, decay, sustain, release } = adsr;
  const peakLevel = velocity;
  const sustainLevel = peakLevel * sustain;
  
  gainNode.gain.setValueAtTime(0, startTime);
  gainNode.gain.linearRampToValueAtTime(peakLevel, startTime + attack);
  gainNode.gain.linearRampToValueAtTime(sustainLevel, startTime + attack + decay);
  gainNode.gain.setValueAtTime(sustainLevel, startTime + duration - release);
  gainNode.gain.linearRampToValueAtTime(0, startTime + duration);
}
```


**オフラインレンダリング**

```javascript
function renderOffline(project) {
  const { tempo, timeSignature } = project.settings;
  const duration = calculateTotalDuration(project);
  const sampleRate = 44100;
  
  const offlineContext = new OfflineAudioContext(2, duration * sampleRate, sampleRate);
  
  // すべてのトラックをスケジュール
  scheduleAllTracks(project, offlineContext, offlineContext.destination);
  
  return offlineContext.startRendering();
}
```

**WAVエクスポート**

標準的なWAVフォーマット（RIFF）を生成：
- チャンネル数: 2（ステレオ）
- サンプルレート: 44100Hz
- ビット深度: 16bit PCM

**MP3エクスポート**

LameJS相当の最小実装を埋め込み：
- エンコード方式: CBR（Constant Bitrate）
- ビットレート: 128kbps
- サンプルレート: 44100Hz
- チャンネル: モノラル（ファイルサイズ削減のため）

MP3エンコーダの実装は、以下の簡略化を行う：
1. サイコアコースティックモデルの簡略化
2. ビットリザーバーの省略
3. VBRの省略（CBRのみ）


### 5. UI Module

ユーザーインターフェースの描画とイベント処理を担当するモジュール。

#### 主要機能

- 3カラムレイアウトの構築
- コード進行グリッドの描画と編集
- ピアノロールの描画と編集
- 候補リストの表示
- 再生ヘッドのアニメーション
- キーボードショートカットの処理

#### インターフェース

```javascript
/**
 * UIを初期化
 */
function initUI()

/**
 * プロジェクトデータからUIを更新
 * @param {Object} project
 */
function render(project)

/**
 * コード進行グリッドを描画
 * @param {Array} chordTrack
 * @param {number} bars
 */
function renderChordGrid(chordTrack, bars)

/**
 * ピアノロールを描画
 * @param {Array} melodyTrack
 * @param {number} bars
 * @param {number} ppq
 */
function renderPianoRoll(melodyTrack, bars, ppq)

/**
 * 候補リストを更新
 * @param {Array} chordSuggestions
 * @param {Array} melodySuggestions
 */
function updateSuggestions(chordSuggestions, melodySuggestions)

/**
 * 再生ヘッドを更新
 * @param {number} currentTick
 * @param {number} totalTicks
 */
function updatePlayhead(currentTick, totalTicks)
```


#### 実装詳細

**レイアウト構造**

```html
<div id="app">
  <div class="left-panel">
    <section class="global-settings">
      <!-- Key, Scale, Tempo, Time Signature, Quantize -->
    </section>
    <section class="track-controls">
      <!-- Piano, Bass, Drums, Pad: Volume, Mute, Solo -->
    </section>
    <section class="export-controls">
      <!-- Export MP3, WAV, Project JSON, Import -->
    </section>
  </div>
  
  <div class="center-panel">
    <div class="transport-controls">
      <!-- Play/Stop, Loop, Metronome, Current Bar -->
    </div>
    <div class="chord-grid">
      <!-- 16 cells for chord progression -->
    </div>
    <div class="piano-roll">
      <!-- Canvas or SVG for note editing -->
    </div>
  </div>
  
  <div class="right-panel">
    <section class="chord-suggestions">
      <!-- List of chord suggestions -->
    </section>
    <section class="melody-suggestions">
      <!-- List of melody note suggestions -->
    </section>
    <section class="shortcuts-help">
      <!-- Keyboard shortcuts reference -->
    </section>
  </div>
</div>
```

**コード進行グリッドの実装**

- 16個のセル（div要素）を横に並べる
- 各セルをクリック → contenteditable または input要素で編集
- Enterキーで確定、Escキーでキャンセル
- 空のセルは薄いグレーで表示

**ピアノロールの実装**

Canvas APIを使用して描画：

1. **背景グリッド**:
   - 横線: 各音高（白鍵は薄いグレー、黒鍵は濃いグレー）
   - 縦線: 拍ごと（1/4, 1/8, 1/16に応じて）

2. **ノート**:
   - 矩形で描画（色: アクセントカラー）
   - 選択中のノートは枠線を強調

3. **インタラクション**:
   - クリック: ノート配置
   - ドラッグ: ノート移動または長さ変更
   - 右クリック/Delete: ノート削除

4. **パフォーマンス最適化**:
   - オフスクリーンキャンバスで背景を事前描画
   - ノートのみを毎フレーム再描画


**候補リストの実装**

```html
<div class="suggestion-item" data-value="G7">
  <div class="suggestion-name">G7</div>
  <div class="suggestion-reason">V of I (cadence)</div>
  <div class="suggestion-weight">★★★★★</div>
</div>
```

- 重みに応じて星の数を表示（0.9以上=5つ星、0.7以上=4つ星、など）
- クリックで即座に反映
- ホバー時に背景色を変更

**キーボードショートカットの実装**

```javascript
document.addEventListener('keydown', (e) => {
  // Spaceキー: 再生/停止
  if (e.code === 'Space' && !e.target.matches('input, [contenteditable]')) {
    e.preventDefault();
    togglePlayback();
  }
  
  // Cmd/Ctrl + Z: Undo
  if ((e.metaKey || e.ctrlKey) && e.key === 'z' && !e.shiftKey) {
    e.preventDefault();
    Store.undo();
  }
  
  // Cmd/Ctrl + Shift + Z: Redo
  if ((e.metaKey || e.ctrlKey) && e.key === 'z' && e.shiftKey) {
    e.preventDefault();
    Store.redo();
  }
  
  // 1/2/3: Quantize切替
  if (['1', '2', '3'].includes(e.key) && !e.target.matches('input, [contenteditable]')) {
    const quantize = { '1': '1/4', '2': '1/8', '3': '1/16' }[e.key];
    Store.updateSettings({ quantize });
  }
  
  // ↑↓: 選択ノートの移動
  if (e.key === 'ArrowUp' || e.key === 'ArrowDown') {
    if (selectedNote) {
      e.preventDefault();
      const delta = e.key === 'ArrowUp' ? 1 : -1;
      Store.moveNote(selectedNote.id, delta);
    }
  }
});
```


### 6. Store Module

アプリケーションの状態管理、Undo/Redo、永続化を担当するモジュール。

#### 主要機能

- プロジェクトデータの管理
- Undo/Redoスタックの管理
- JSON保存/読込
- 状態変更の通知

#### インターフェース

```javascript
/**
 * 現在のプロジェクトを取得
 * @returns {Object} project
 */
function getProject()

/**
 * プロジェクトを更新
 * @param {Object} updates - 更新する部分的なデータ
 */
function updateProject(updates)

/**
 * コードを追加/更新
 * @param {number} bar - 小節番号
 * @param {string} chordName - コード名
 */
function setChord(bar, chordName)

/**
 * メロディノートを追加
 * @param {number} startTick - 開始Tick
 * @param {number} duration - 長さ（Tick）
 * @param {number} midi - MIDIノート番号
 * @param {number} velocity - ベロシティ
 */
function addNote(startTick, duration, midi, velocity)

/**
 * メロディノートを削除
 * @param {string} noteId - ノートID
 */
function deleteNote(noteId)

/**
 * Undo
 */
function undo()

/**
 * Redo
 */
function redo()

/**
 * プロジェクトをJSONとしてエクスポート
 * @returns {string} JSON文字列
 */
function exportJSON()

/**
 * JSONからプロジェクトをインポート
 * @param {string} jsonString - JSON文字列
 */
function importJSON(jsonString)

/**
 * 状態変更を監視
 * @param {Function} callback - 変更時に呼ばれるコールバック
 */
function subscribe(callback)
```


#### 実装詳細

**状態管理パターン**

単純なPub/Subパターンを採用：

```javascript
const Store = (function() {
  let project = createInitialProject();
  let undoStack = [];
  let redoStack = [];
  const subscribers = [];
  
  function notify() {
    subscribers.forEach(callback => callback(project));
  }
  
  function saveState() {
    undoStack.push(JSON.parse(JSON.stringify(project)));
    if (undoStack.length > 50) undoStack.shift(); // 最大50履歴
    redoStack = []; // 新しい操作でRedoスタックをクリア
  }
  
  return {
    getProject: () => project,
    updateProject: (updates) => {
      saveState();
      project = { ...project, ...updates };
      notify();
    },
    // ... 他のメソッド
  };
})();
```

**Undo/Redoの実装**

- 各操作の前に現在の状態をundoStackに保存
- Undo: undoStackから状態を復元し、現在の状態をredoStackに保存
- Redo: redoStackから状態を復元し、undoStackに保存
- 新しい操作が実行されたらredoStackをクリア

**JSON保存/読込**

```javascript
function exportJSON() {
  return JSON.stringify(project, null, 2);
}

function importJSON(jsonString) {
  try {
    const imported = JSON.parse(jsonString);
    // バリデーション
    if (!imported.meta || !imported.settings || !imported.tracks) {
      throw new Error('Invalid project format');
    }
    saveState();
    project = imported;
    notify();
  } catch (error) {
    console.error('Failed to import project:', error);
    alert('プロジェクトの読み込みに失敗しました');
  }
}
```


### 7. LameJS MP3 Encoder

MP3エンコードを行う最小実装。単一HTMLファイルに埋め込むため、サイズを抑えた実装とする。

#### 実装方針

完全なLameJSの実装は大きすぎるため、以下の簡略化を行う：

1. **モノラルのみ対応**（ステレオは左右チャンネルをミックス）
2. **CBR（固定ビットレート）のみ**（128kbps）
3. **サイコアコースティックモデルの簡略化**
4. **ビットリザーバーの省略**

#### 主要機能

- Float32Array → MP3バイナリ変換
- MDCT（Modified Discrete Cosine Transform）
- ハフマン符号化
- MP3フレームの生成

#### インターフェース

```javascript
/**
 * Float32配列をMP3にエンコード
 * @param {Float32Array} samples - 音声サンプル（-1.0〜1.0）
 * @param {number} sampleRate - サンプルレート（44100）
 * @returns {Uint8Array} MP3バイナリデータ
 */
function encodeMP3(samples, sampleRate)
```

#### 実装の概要

```javascript
// 1. サンプルを1152サンプルごとのフレームに分割
// 2. 各フレームに対して:
//    a. MDCTを適用（時間領域 → 周波数領域）
//    b. 量子化（ビットレート制約に合わせる）
//    c. ハフマン符号化
//    d. MP3フレームヘッダーを付与
// 3. すべてのフレームを結合

const MP3_FRAME_SIZE = 1152;
const BITRATE = 128; // kbps
const SAMPLE_RATE = 44100;

function encodeMP3(samples, sampleRate) {
  const frames = [];
  
  for (let i = 0; i < samples.length; i += MP3_FRAME_SIZE) {
    const frame = samples.slice(i, i + MP3_FRAME_SIZE);
    const encoded = encodeFrame(frame);
    frames.push(encoded);
  }
  
  return concatenateFrames(frames);
}
```

**注意**: 完全な実装は複雑なため、実際のコードでは既存のLameJS実装を参考にしつつ、最小限の機能に絞る。万一、実装が困難な場合は、WAVエクスポートのみを提供し、MP3は将来の拡張とする。


## Data Models

### Project Data Structure

```javascript
const project = {
  meta: {
    name: "True Composer Kit",
    version: 1,
    created: "2025-10-03T00:00:00Z",
    modified: "2025-10-03T00:00:00Z"
  },
  
  settings: {
    key: "C",                    // "C", "C#", "Db", "D", ..., "B"
    scale: "Major",              // "Major", "Natural Minor", "Dorian", etc.
    tempo: 120,                  // BPM
    timeSignature: "4/4",        // "4/4", "3/4", "6/8", etc.
    quantize: "1/16"             // "1/4", "1/8", "1/16"
  },
  
  timeline: {
    bars: 16,                    // 小節数
    ppq: 480                     // Pulses Per Quarter note（内部解像度）
  },
  
  tracks: {
    chord: [
      {
        id: "chord-1",
        bar: 0,                  // 小節番号（0-indexed）
        name: "C"                // コード名
      },
      {
        id: "chord-2",
        bar: 1,
        name: "Am7"
      }
      // ...
    ],
    
    melody: [
      {
        id: "note-1",
        startTick: 0,            // 開始位置（Tick）
        duration: 240,           // 長さ（Tick）
        midi: 60,                // MIDIノート番号（C4 = 60）
        velocity: 0.9            // ベロシティ（0.0〜1.0）
      },
      {
        id: "note-2",
        startTick: 240,
        duration: 240,
        midi: 64,
        velocity: 0.8
      }
      // ...
    ],
    
    bass: [
      // 自動生成される場合は同じ構造
      // 手動編集も可能（将来拡張）
    ],
    
    drums: [
      // Kick, Snare, Hatのパターン
      {
        id: "drum-1",
        startTick: 0,
        instrument: "kick",      // "kick", "snare", "hat"
        velocity: 1.0
      }
      // ...
    ]
  },
  
  trackSettings: {
    piano: {
      volume: 0.8,
      mute: false,
      solo: false,
      instrument: "piano"
    },
    bass: {
      volume: 0.7,
      mute: false,
      solo: false,
      instrument: "bass",
      autoGenerate: true         // 自動生成ON/OFF
    },
    drums: {
      volume: 0.6,
      mute: false,
      solo: false,
      instrument: "drums",
      autoGenerate: true
    },
    pad: {
      volume: 0.5,
      mute: false,
      solo: false,
      instrument: "pad"
    }
  },
  
  playback: {
    loop: true,
    loopStart: 0,                // ループ開始小節
    loopEnd: 16,                 // ループ終了小節
    metronome: false
  }
};
```


### Chord Suggestion Data Structure

```javascript
const chordSuggestion = {
  name: "G7",                    // コード名
  reason: "V of I (cadence)",    // 提案理由
  weight: 0.90,                  // 重み（0.0〜1.0）
  function: "D"                  // 機能（"T", "SD", "D", "other"）
};
```

### Melody Suggestion Data Structure

```javascript
const melodySuggestion = {
  midi: 64,                      // MIDIノート番号
  label: "E (3rd)",              // 表示ラベル
  weight: 0.85,                  // 重み（0.0〜1.0）
  tip: "コードCの3度で安定",    // ツールチップ
  role: "chord-tone"             // "chord-tone", "tension", "passing", "chromatic"
};
```

### Parsed Chord Data Structure

```javascript
const parsedChord = {
  root: 0,                       // ルート音（0=C, 1=C#, ..., 11=B）
  rootName: "C",                 // ルート音名
  type: "maj7",                  // コードタイプ
  notes: [0, 4, 7, 11],          // 構成音（ルートからの半音数）
  midiNotes: [60, 64, 67, 71]    // 実際のMIDIノート番号（C4基準）
};
```

## Error Handling

### エラーの種類と対処

1. **コード解析エラー**
   - 原因: 不正なコード名（例: "XYZ"）
   - 対処: デフォルトでメジャーコードとして解釈、またはエラーメッセージを表示

2. **オーディオコンテキストエラー**
   - 原因: ブラウザがWeb Audio APIをサポートしていない
   - 対処: エラーメッセージを表示し、エクスポート機能のみ提供

3. **ファイル読み込みエラー**
   - 原因: 不正なJSON形式、バージョン不一致
   - 対処: エラーメッセージを表示し、読み込みをキャンセル

4. **エクスポートエラー**
   - 原因: メモリ不足、エンコード失敗
   - 対処: エラーメッセージを表示し、代替フォーマット（WAV）を提案

### エラーハンドリングの実装

```javascript
try {
  const chord = parseChord(chordName);
} catch (error) {
  console.error('Chord parse error:', error);
  // デフォルト値を使用
  const chord = { root: 0, type: 'major', notes: [0, 4, 7] };
}

// AudioContext初期化
let audioContext;
try {
  audioContext = new (window.AudioContext || window.webkitAudioContext)();
} catch (error) {
  console.error('Web Audio API not supported:', error);
  alert('お使いのブラウザはWeb Audio APIをサポートしていません。最新のChrome、Firefox、Safariをご使用ください。');
}
```


## Testing Strategy

### テストの方針

単一HTMLファイルのため、自動テストフレームワークは使用しない。代わりに、以下の手動テストとコンソールログによる検証を行う。

### 1. ユニットテスト（コンソールベース）

主要な関数に対して、コンソールで実行可能なテスト関数を用意：

```javascript
// デバッグモード（開発時のみ有効）
const DEBUG = true;

function testMusicTheory() {
  console.group('MusicTheory Tests');
  
  // Test 1: parseChord
  const chord1 = parseChord('C');
  console.assert(chord1.root === 0 && chord1.notes.length === 3, 'C major chord');
  
  const chord2 = parseChord('Am7');
  console.assert(chord2.root === 9 && chord2.notes.length === 4, 'Am7 chord');
  
  // Test 2: getDiatonicChords
  const diatonic = getDiatonicChords('C', 'Major');
  console.assert(diatonic.length === 7, 'C Major has 7 diatonic chords');
  
  // Test 3: getChordFunction
  const func = getChordFunction('G7', 'C', 'Major');
  console.assert(func === 'D', 'G7 is dominant in C Major');
  
  console.groupEnd();
}

function testSuggest() {
  console.group('Suggest Tests');
  
  // Test 1: suggestNextChords
  const suggestions = suggestNextChords('G7', 'C', 'Major', []);
  console.assert(suggestions.length > 0, 'Should return suggestions');
  console.assert(suggestions[0].name === 'C', 'G7 should suggest C (V-I)');
  console.assert(suggestions[0].weight > 0.8, 'V-I should have high weight');
  
  // Test 2: suggestMelodyNotes
  const melodySuggestions = suggestMelodyNotes('C', [], 'C', 'Major');
  console.assert(melodySuggestions.length > 0, 'Should return melody suggestions');
  
  console.groupEnd();
}

if (DEBUG) {
  // ページ読み込み後にテストを実行
  window.addEventListener('load', () => {
    testMusicTheory();
    testSuggest();
  });
}
```


### 2. 統合テスト（手動）

以下のシナリオを手動でテスト：

#### シナリオ1: 基本的な作曲フロー
1. アプリケーションを開く
2. コード進行を入力（C → Am → F → G）
3. 各コードに対してメロディを配置
4. 再生して確認
5. プロジェクトをJSONで保存
6. ページをリロードして読み込み

**期待結果**: すべての操作が正常に動作し、保存/読込後もデータが保持される

#### シナリオ2: 候補提案の検証
1. コード進行に「C」を入力
2. 次のコード候補を確認
3. 「G7」が高い重みで提案されることを確認
4. 「G7」をクリックして挿入
5. 次の候補で「C」が最高重みで提案されることを確認

**期待結果**: 音楽理論的に妥当な候補が提案される

#### シナリオ3: オーディオエクスポート
1. サンプルプロジェクトを再生
2. 「Export WAV」をクリック
3. ダウンロードされたWAVファイルを再生
4. 「Export MP3」をクリック
5. ダウンロードされたMP3ファイルを再生

**期待結果**: 両方のファイルが正常に再生され、音質が許容範囲内

#### シナリオ4: Undo/Redo
1. コードを追加
2. Cmd/Ctrl+Zで取り消し
3. コードが削除されることを確認
4. Cmd/Ctrl+Shift+Zでやり直し
5. コードが復元されることを確認

**期待結果**: Undo/Redoが正常に動作

#### シナリオ5: キーボードショートカット
1. Spaceキーで再生/停止
2. 1/2/3キーでクオンタイズ切替
3. ノートを選択して↑↓キーで移動
4. ←→キーで長さ変更

**期待結果**: すべてのショートカットが正常に動作


### 3. パフォーマンステスト

以下の項目を測定：

1. **初期ロード時間**: ページ読み込みからUIが表示されるまで（目標: 1秒以内）
2. **再生レイテンシ**: 再生ボタンクリックから音が鳴るまで（目標: 100ms以内）
3. **UI応答性**: ノート配置、コード入力の応答時間（目標: 50ms以内）
4. **大量ノートの描画**: 100個のノートを配置した場合のフレームレート（目標: 30fps以上）
5. **エクスポート時間**: 16小節のプロジェクトをWAV/MP3にエクスポート（目標: 5秒以内）

**測定方法**:

```javascript
// パフォーマンス測定
console.time('Export WAV');
audioBufferToWav(buffer);
console.timeEnd('Export WAV');

// フレームレート測定
let frameCount = 0;
let lastTime = performance.now();

function measureFPS() {
  frameCount++;
  const currentTime = performance.now();
  if (currentTime - lastTime >= 1000) {
    console.log('FPS:', frameCount);
    frameCount = 0;
    lastTime = currentTime;
  }
  requestAnimationFrame(measureFPS);
}
```

### 4. ブラウザ互換性テスト

以下のブラウザで動作確認：

- Chrome（最新版）
- Firefox（最新版）
- Safari（最新版）
- Edge（最新版）

**確認項目**:
- Web Audio APIの動作
- Canvas描画の正確性
- ファイルダウンロードの動作
- キーボードショートカットの動作

### 5. アクセシビリティテスト

- キーボードのみでの操作可能性
- フォーカスインジケータの視認性
- カラーコントラストの確認（WCAG AA基準）
- スクリーンリーダーでの基本操作（将来的な改善項目）


## UI Design Specifications

### カラーパレット

```css
:root {
  /* ダークベース */
  --bg-primary: #1a1a1f;
  --bg-secondary: #25252d;
  --bg-tertiary: #2f2f3a;
  
  /* アクセント（ネオン風） */
  --accent-primary: #00d9ff;
  --accent-secondary: #ff00ff;
  --accent-tertiary: #00ff88;
  
  /* テキスト */
  --text-primary: #ffffff;
  --text-secondary: #b0b0b8;
  --text-muted: #6a6a72;
  
  /* ボーダー */
  --border-color: #3a3a44;
  --border-highlight: #4a4a54;
  
  /* 状態 */
  --success: #00ff88;
  --warning: #ffaa00;
  --error: #ff4444;
  
  /* グリッド */
  --grid-line: #2a2a34;
  --grid-beat: #3a3a44;
  --grid-bar: #4a4a54;
  
  /* ノート */
  --note-color: var(--accent-primary);
  --note-selected: var(--accent-secondary);
}
```

### タイポグラフィ

```css
:root {
  --font-family: -apple-system, BlinkMacSystemFont, 'Segoe UI', Roboto, 
                 'Helvetica Neue', Arial, sans-serif;
  --font-mono: 'SF Mono', Monaco, 'Cascadia Code', 'Courier New', monospace;
  
  --font-size-xs: 0.75rem;   /* 12px */
  --font-size-sm: 0.875rem;  /* 14px */
  --font-size-base: 1rem;    /* 16px */
  --font-size-lg: 1.125rem;  /* 18px */
  --font-size-xl: 1.25rem;   /* 20px */
  --font-size-2xl: 1.5rem;   /* 24px */
}
```

### スペーシング

```css
:root {
  --spacing-xs: 0.25rem;   /* 4px */
  --spacing-sm: 0.5rem;    /* 8px */
  --spacing-md: 1rem;      /* 16px */
  --spacing-lg: 1.5rem;    /* 24px */
  --spacing-xl: 2rem;      /* 32px */
  --spacing-2xl: 3rem;     /* 48px */
}
```

### ボーダーと角丸

```css
:root {
  --border-radius-sm: 4px;
  --border-radius-md: 8px;
  --border-radius-lg: 12px;
  --border-width: 1px;
}
```


### コンポーネントスタイル

#### ボタン

```css
.btn {
  padding: var(--spacing-sm) var(--spacing-md);
  border: var(--border-width) solid var(--border-color);
  border-radius: var(--border-radius-md);
  background: var(--bg-secondary);
  color: var(--text-primary);
  font-size: var(--font-size-sm);
  cursor: pointer;
  transition: all 0.2s ease;
}

.btn:hover {
  background: var(--bg-tertiary);
  border-color: var(--border-highlight);
  transform: translateY(-1px);
  box-shadow: 0 4px 12px rgba(0, 217, 255, 0.2);
}

.btn:active {
  transform: translateY(0);
}

.btn-primary {
  background: var(--accent-primary);
  color: var(--bg-primary);
  border-color: var(--accent-primary);
}

.btn-primary:hover {
  background: var(--accent-secondary);
  border-color: var(--accent-secondary);
  box-shadow: 0 4px 12px rgba(255, 0, 255, 0.3);
}
```

#### 入力フィールド

```css
.input {
  padding: var(--spacing-sm);
  border: var(--border-width) solid var(--border-color);
  border-radius: var(--border-radius-sm);
  background: var(--bg-primary);
  color: var(--text-primary);
  font-size: var(--font-size-sm);
  transition: border-color 0.2s ease;
}

.input:focus {
  outline: none;
  border-color: var(--accent-primary);
  box-shadow: 0 0 0 3px rgba(0, 217, 255, 0.1);
}
```

#### セレクト

```css
.select {
  padding: var(--spacing-sm);
  border: var(--border-width) solid var(--border-color);
  border-radius: var(--border-radius-sm);
  background: var(--bg-secondary);
  color: var(--text-primary);
  font-size: var(--font-size-sm);
  cursor: pointer;
}
```

#### カード

```css
.card {
  background: var(--bg-secondary);
  border: var(--border-width) solid var(--border-color);
  border-radius: var(--border-radius-lg);
  padding: var(--spacing-lg);
  box-shadow: 0 2px 8px rgba(0, 0, 0, 0.3);
}
```


### レイアウト

#### 3カラムレイアウト

```css
#app {
  display: grid;
  grid-template-columns: 280px 1fr 320px;
  grid-template-rows: 1fr;
  height: 100vh;
  gap: var(--spacing-md);
  padding: var(--spacing-md);
  background: var(--bg-primary);
}

/* レスポンシブ */
@media (max-width: 1200px) {
  #app {
    grid-template-columns: 240px 1fr 280px;
  }
}

@media (max-width: 900px) {
  #app {
    grid-template-columns: 1fr;
    grid-template-rows: auto auto 1fr;
    overflow-y: auto;
  }
}
```

#### パネル

```css
.panel {
  background: var(--bg-secondary);
  border: var(--border-width) solid var(--border-color);
  border-radius: var(--border-radius-lg);
  padding: var(--spacing-lg);
  overflow-y: auto;
}

.panel-header {
  font-size: var(--font-size-lg);
  font-weight: 600;
  margin-bottom: var(--spacing-md);
  color: var(--text-primary);
  border-bottom: var(--border-width) solid var(--border-color);
  padding-bottom: var(--spacing-sm);
}
```

### アニメーション

```css
/* フェードイン */
@keyframes fadeIn {
  from {
    opacity: 0;
    transform: translateY(10px);
  }
  to {
    opacity: 1;
    transform: translateY(0);
  }
}

.fade-in {
  animation: fadeIn 0.3s ease;
}

/* パルス（再生中） */
@keyframes pulse {
  0%, 100% {
    opacity: 1;
  }
  50% {
    opacity: 0.6;
  }
}

.playing {
  animation: pulse 1s ease infinite;
}

/* ネオングロー */
.glow {
  box-shadow: 0 0 10px var(--accent-primary),
              0 0 20px var(--accent-primary),
              0 0 30px var(--accent-primary);
}
```


## Performance Optimization

### 1. Canvas描画の最適化

**問題**: ピアノロールの再描画が頻繁に発生するとパフォーマンスが低下

**解決策**:
- オフスクリーンキャンバスで背景グリッドを事前描画
- ノートのみを毎フレーム再描画
- requestAnimationFrameで描画タイミングを制御
- Dirty Rectangleパターン（変更部分のみ再描画）

```javascript
// オフスクリーンキャンバスで背景を描画
const offscreenCanvas = document.createElement('canvas');
const offscreenCtx = offscreenCanvas.getContext('2d');
drawGrid(offscreenCtx); // 一度だけ実行

// メインキャンバスに背景をコピー
function render() {
  ctx.drawImage(offscreenCanvas, 0, 0);
  drawNotes(ctx); // ノートのみ描画
}
```

### 2. Web Audio APIのスケジューリング最適化

**問題**: 大量のノートを一度にスケジュールするとメモリ使用量が増加

**解決策**:
- 先読みスケジューリング（100ms先まで）
- 不要になったノードの即座の破棄
- オブジェクトプールパターンでノードを再利用

```javascript
const nodePool = [];

function getAudioNode() {
  return nodePool.pop() || audioContext.createGain();
}

function releaseAudioNode(node) {
  node.disconnect();
  nodePool.push(node);
}
```

### 3. 状態管理の最適化

**問題**: 大きなプロジェクトのUndoスタックがメモリを圧迫

**解決策**:
- Undoスタックの最大サイズを制限（50履歴）
- 差分のみを保存（将来的な改善）
- JSON.stringify/parseの最適化

### 4. UIレンダリングの最適化

**問題**: 状態変更のたびに全体を再描画するとパフォーマンスが低下

**解決策**:
- 変更部分のみを更新（部分的な再描画）
- デバウンス/スロットルで頻繁な更新を抑制
- Virtual DOMの簡易実装（必要に応じて）

```javascript
// デバウンス
function debounce(func, wait) {
  let timeout;
  return function(...args) {
    clearTimeout(timeout);
    timeout = setTimeout(() => func.apply(this, args), wait);
  };
}

const debouncedRender = debounce(render, 16); // 約60fps
```


## Security Considerations

### 1. XSS対策

**問題**: ユーザー入力（コード名、プロジェクト名等）がそのままHTMLに挿入されるとXSSの危険性

**解決策**:
- すべてのユーザー入力をエスケープ
- `textContent`を使用（`innerHTML`を避ける）
- コード名のバリデーション

```javascript
function escapeHtml(text) {
  const div = document.createElement('div');
  div.textContent = text;
  return div.innerHTML;
}

// 使用例
element.textContent = userInput; // 安全
// element.innerHTML = userInput; // 危険
```

### 2. JSONインジェクション対策

**問題**: 不正なJSONファイルを読み込むとアプリケーションがクラッシュ

**解決策**:
- JSON.parseをtry-catchで囲む
- スキーマバリデーション
- バージョンチェック

```javascript
function validateProject(project) {
  if (!project.meta || !project.settings || !project.tracks) {
    throw new Error('Invalid project structure');
  }
  if (project.meta.version > CURRENT_VERSION) {
    throw new Error('Project version not supported');
  }
  return true;
}
```

### 3. ファイルサイズ制限

**問題**: 巨大なプロジェクトファイルを読み込むとブラウザがフリーズ

**解決策**:
- ファイルサイズの上限を設定（例: 10MB）
- ノート数の上限を設定（例: 10,000個）

```javascript
const MAX_FILE_SIZE = 10 * 1024 * 1024; // 10MB
const MAX_NOTES = 10000;

function validateFileSize(file) {
  if (file.size > MAX_FILE_SIZE) {
    throw new Error('File too large');
  }
}
```


## Future Enhancements

以下は初期バージョンには含まれないが、将来的に追加を検討する機能：

### 1. 高度な音楽理論機能

- モーダルインターチェンジ（借用和音）
- 代理コード（Substitute chords）
- リハーモナイゼーション提案
- より多くのスケール（ハーモニックマイナー、メロディックマイナー、ブルーススケール等）
- コードボイシングの提案

### 2. 編集機能の拡張

- 複数ノートの同時選択と編集
- コピー&ペースト
- トランスポーズ（キー変更）
- タイムストレッチ
- ベロシティエディタ
- 複数小節にまたがるコード

### 3. オーディオ機能の拡張

- より多くの楽器（Strings, Brass, Guitar等）
- エフェクト（Reverb, Delay, Chorus等）
- ミキサー（パン、EQ）
- オーディオファイルのインポート
- MIDIファイルのエクスポート

### 4. UI/UX改善

- ダークモード/ライトモードの切り替え
- カスタムテーマ
- ズーム機能（タイムライン、ピアノロール）
- ミニマップ
- チュートリアル/オンボーディング
- 多言語対応

### 5. コラボレーション機能

- プロジェクトの共有（URL経由）
- リアルタイムコラボレーション（WebRTC）
- コメント機能

### 6. AI機能

- AIによるメロディ生成
- AIによるコード進行生成
- スタイル転送（ジャンル変換）

### 7. パフォーマンス改善

- Web Workersでエンコード処理をオフロード
- WebAssemblyでMP3エンコードを高速化
- IndexedDBで大きなプロジェクトを保存

## Implementation Notes

### 開発の優先順位

1. **Phase 1（MVP）**: 
   - MusicTheory Module
   - 基本的なUI（コード進行グリッド、簡易ピアノロール）
   - 簡単な再生機能
   - WAVエクスポート

2. **Phase 2**:
   - Suggest Module（コード候補）
   - 完全なピアノロール
   - Undo/Redo
   - プロジェクト保存/読込

3. **Phase 3**:
   - Suggest Module（メロディ候補）
   - 自動伴奏（Bass, Drums）
   - MP3エクスポート
   - UIの洗練

### コードの構造化

```javascript
// グローバルスコープの汚染を避けるため、即時関数で囲む
(function() {
  'use strict';
  
  // すべてのモジュールをここに定義
  const MusicTheory = { /* ... */ };
  const Suggest = { /* ... */ };
  const Sequencer = { /* ... */ };
  const AudioEngine = { /* ... */ };
  const UI = { /* ... */ };
  const Store = { /* ... */ };
  
  // 初期化
  window.addEventListener('DOMContentLoaded', () => {
    Store.init();
    UI.init();
  });
})();
```

### コメントの方針

- 各モジュールの冒頭に目的と責務を記載
- 複雑なアルゴリズムには詳細な説明を追加
- 音楽理論的な根拠を説明
- TODOコメントで将来の改善点を記載

```javascript
/**
 * MusicTheory Module
 * 
 * 音楽理論の計算とコード解析を担当。
 * キー、スケール、ダイアトニックコード、五度圏、機能和声などを扱う。
 */
const MusicTheory = (function() {
  // ...
})();
```

## Conclusion

本設計書は、真の作曲キット（True Composer Kit）の技術的な実装方針を定義しています。単一HTMLファイルという制約の中で、音楽理論に基づいたインテリジェントな作曲支援を実現するため、各モジュールを疎結合に設計し、将来の拡張性を確保しています。

次のステップとして、この設計に基づいた実装計画（tasks.md）を作成し、段階的に機能を実装していきます。
