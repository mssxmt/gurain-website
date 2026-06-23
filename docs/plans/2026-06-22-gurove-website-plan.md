# GuRove (GROVE) 販送サイト 実装計画書

- 作成日: 2026-06-22
- 対象プロジェクト: `GuRainHP/gurain-website/`（既存 Astro 6 + Tailwind v4）
- デプロイ先: `lab.mssxmt.com/gurove/`（GuRain と同じドメイン・同じビルド）
- 関連: drum-synth リポジトリ SPECIFICATION.md v0.5

---

## 0. ユーザー承認が必要な判断ポイント（先に決める）

| # | 判断ポイント | 推奨 | 理由 |
|---|---|---|---|
| **A** | **配色テーマ** | **ダーク（Asfalt 深夜 × Ember 琥珀 × Mycelium 緑）** | GuRain は明るい "Ethereal Mist"。ドラムマシンは深夜スタジオ・LED・テンポが本質。ブランド統一は **フォント体系・signature アニメーションの系譜・レイアウト骨格** で担保し、配色はプロダクト個性に譲る。双子プロダクトの正しい扱い。→ **要GO/NO判定** |
| **B** | **UI スクショ** | **画像なし → CSS/SVG 図解 + ライブグリッドで代替** | GuRove は未リリースで実機スクショが存在しない。画像を待たず、グリッド・波形・パラメータ数値を CSS/SVG で描く。frontend-design の「subject's own world」を活かす。→ **要GO/NO判定** |
| **C** | **ルートポータル（/）の "Coming Soon" カード** | **GuRove カードへ差し替え** | ページ完成後、`src/pages/index.astro` の Coming Soon を GuRove へ。→ ページ完成時に別途確認 |
| **D** | **計画書の場所** | `docs/plans/`（本ファイル） | drum-synth と同じ慣習。GuRain 既存 spec は `docs/superpowers/specs/`。→ 問題なければこのまま |

---

## 1. 目的・スコープ

### 目的
GuRove の紹介・販売ページを `/gurove/` に構築する。コア USP「7種ジェネラティブアルゴリズムが自律的にビートを生成し、seed で完全再現する」を1秒で体感させる。

### スコープ（In）
- `/gurove/index.astro` 単一ページ（ランディング）
- GuRove 専用ダークレイアウト・テーマ
- signature: ライブ・ジェネレーティブ・グリッド（純JS, 依存なし）
- 7アルゴリズム解説セクション
- シンセ/エフェクト/MIDI/プリセットのフィーチャーセクション

### スコープ（Out / 後続）
- マニュアル・リリースノート・免責事項ページ（後日）
- 実際の購入リンク（Polar 等）の差し込み（URL確定後）
- ルートポータルの GuRove カード差し替え（C, 完了後）
- OGP 画像・favicon（後日）

---

## 2. ディレクトリ・ファイル構成

```
gurain-website/
├── src/
│   ├── styles/
│   │   └── gurove-theme.css          【新規】ダークトークン+コンポーネント(body.grove scoped)
│   ├── layouts/
│   │   └── GuRoveLayout.astro        【新規】ダークガラス ヘッダー/フッター
│   ├── components/
│   │   └── gurove/
│   │       ├── GenerativeGrid.astro  【新規】signature: 7アルゴリズム自律発光グリッド
│   │       ├── Hero.astro            【新規】ヒーロー（キャッチコピー+グリッド）
│   │       ├── AlgorithmShowcase.astro【新規】7アルゴリズム解説+mini-grid
│   │       ├── FeatureRow.astro      【新規】汎用ジグザグ行（音作り/エフェクト/MIDI）
│   │       └── PresetGallery.astro   【新規】30プリセット チップ表示
│   └── pages/
│       └── gurove/
│           └── index.astro           【新規】ページ統合
└── docs/plans/
    └── 2026-06-22-gurove-website-plan.md  【本ファイル】
```

**GuRain 側への影響**: ゼロ。新規ファイルのみ。既存 `global.css` / `design-system.css` / `BaseLayout.astro` は編集しない。`body.grove` スコープで隔離。

---

## 3. デザイントークン（確定版）

### Color（6 named hex）
| トークン | Hex | 役割 |
|---|---|---|
| `--g-asfalt` | `#0E1014` | 背景（雨上がりのアスファルト） |
| `--g-panel` | `#16191F` | パネル面（ブラシド金属） |
| `--g-ember` | `#EFA13C` | 主アクセント（808 真空管LED・打撃） |
| `--g-mycelium` | `#3FD9A8` | 副アクセント（菌糸=ジェネラティブ有機） |
| `--g-bone` | `#ECE7DC` | テキスト（暖かいオフホワイト） |
| `--g-ash` | `#7E828C` | 補助テキスト・キャプション |

**リスクの正当化**: Ember（アナログ熱）×Mycelium（デジタル計算）の併用は、製品のコア「機械仕掛けの音作り × 自律アルゴリズム」を色で語る。acid-green/vermilion デフォルトではなく、808 インカネセントLED 由来なのでドラム機材に固有。

### Typography（3役）
| 役割 | 書体 | 意図 |
|---|---|---|
| Display | **Fraunces**（可変セリフ, SOFT/WONK 軸） | GuRain のセリフ DNA 継承。有機的なくせ字で自然。ドラムタイトルは WONK↑ |
| Body | **Space Grotesk** | ジオメトリック・テクノ体温 |
| Mono | **JetBrains Mono** | BPM・`seqSeed`・パラメータ値・コード断片 |

### Signature
**ライブ・ジェネレーティブ・グリッド**: 8レーン×16ステップのLEDグリッドが純JSで7種アルゴリズムをシミュレートし自律発光。`algo`/`seed`/`step` をモノスペースでリアルタイム表示。2ループ毎に進化。reduced-motion で静止。

---

## 4. ページ構成（セクション順）

```
NAV   GROVE ✦   Features  Presets  [Download]     ← sticky, hairline bottom, ダークガラス
─────────────────────────────────────────────────
HERO  §リズムを、植える。            ← Fraunces 巨大 + グリッド signature
      8つの声が歩き回り、7つのアルゴリズムがグルーヴを育てる。
      同じ種を蒔けば、同じ森が鳴る。
      [デモを聴く] [ダウンロード]
─────────────────────────────────────────────────
§01   生まれては、変わる。          ← イントロ + 統計(8ch/16voice/7algo/30preset/330param)
─────────────────────────────────────────────────
§02   CODE THAT GROOVES            ← 最大セクション。7アルゴリズム
      AlgorithmShowcase: 各アルゴリズム mini-grid で挙動再生
─────────────────────────────────────────────────
§03   音を作る                      ← FeatureRow ジグザグ(画像左)
      PolyBLEP/FM/Noise/Filter/Drive/PitchEnv
─────────────────────────────────────────────────
§04   空間と、刻み                  ← FeatureRow ジグザグ(画像右・reverse)
      Reverb/Delay/Stutter/Limiter
─────────────────────────────────────────────────
§05   息を合わせる                  ← FeatureRow ジグザグ(画像左)
      MIDI Learn/Clock同期/MIDI Out
─────────────────────────────────────────────────
§06   30の種                        ← PresetGallery: Standard16+Experimental12 チップ
─────────────────────────────────────────────────
FOOT  システム要件: macOS 14+ · Apple Silicon
      [マニュアル(後日)]  © 2026 GuRove · MSSXMTxLAB
```

**構造は情報**: `§01–§06` は「リズムが種から完成するまでの流れ」なので番号が意味を持つ（装飾ではない）。

---

## 5. 実装ステップ（チェックリスト）

- [ ] **S1**: `gurove-theme.css` — トークン + body.grove 基本スタイル + ボタン + パネル + グリッド + アルゴカード + チップ + reveal
- [ ] **S2**: `GuRoveLayout.astro` — `<body class="grove">`, ダークガラス固定ヘッダー, フッター, フォント import, reveal IntersectionObserver
- [ ] **S3**: `GenerativeGrid.astro`（signature）— 8×16 DOM grid + 7アルゴリズム純JS実装 + playhead + readout + reduced-motion
- [ ] **S4**: `Hero.astro` — タイトル(Fraunces, グリッド背景) + GenerativeGrid + CTA
- [ ] **S5**: `AlgorithmShowcase.astro` — 7アルゴカード各 mini-grid 静的/軽量アニメ
- [ ] **S6**: `FeatureRow.astro` — 汎用ジグザグ（画像枠は SVG 図解を内包）
- [ ] **S7**: `PresetGallery.astro` — 30 チップ
- [ ] **S8**: `pages/gurove/index.astro` — 統合
- [ ] **S9**: 検証 — `npm install`(要確認) → `npm run dev` → ブラウザ確認 → スクショ

---

## 6. GenerativeGrid のアルゴリズム実装仕様

本物のロジックの簡易実装（製品 SPECIFICATION §7 準拠）:

| アルゴリズム | 実装 | seed依存 |
|---|---|---|
| Euclidean | Bjorklund(steps, hits) + offset 回転 | 否(決定的) |
| ProbDensity | 各step独立確率(density) | 是(LCG) |
| LFSR | タップ位置 XOR, length ビットレジスタ | 是(初期値) |
| Markov | order次状態→確率テーブル, bias オフセット | 是(テーブル生成) |
| CellularAutomaton | rule(Wolfram) で次世代, rows 世代進行 | 否(決定的) |
| RandomWalk | stepSize 移動, bounds 折返 | 是(初期位置) |
| 固定 | Kick/Snare/Hihat 定義済みパターン | 否 |

- 8レーンに7アルゴ+固定1を割当（ch1-7=各アルゴ, ch8=固定ハihat的）
- テンポ: 120bpm → 125ms/16th（`--g-step-ms`）。requestAnimationFrame + 時間ベース
- 2ループ毎: 乱数系は seed 再抽選、CA は次世代へ進行
- 色: hit=Ember, アクセントレーン(RandomWalk/CA)=Mycelium
- playhead 列ハイライト

---

## 7. コピーライティング（GuRain 詩的レジスター継承）

| 場所 | コピー |
|---|---|
| Hero H1 | リズムを、植える。 |
| Hero sub | 8つの声が自律的に歩き回り、7つのアルゴリズムがグルーヴを育てる。macOSネイティブのジェネラティブ・ドラムシンセ。 |
| Hero 3行目 | 同じ種（seed）を蒔けば、同じ森が鳴る。 |
| §01 | 生まれては、変わる。 |
| §02 eyebrow | CODE THAT GROOVES |
| CTA | ダウンロード / デモを聴く |

語彙: grove(森)・seed(種)・mycelium(菌糸)・rove(歩行)。ジェネラティブ=有機成長メタファーを全篇で保持。GuRain「雨・粒」と同系譜の自然語彙。

---

## 8. GuRain との統一 / 差別化

| 軸 | GuRain | GuRove |
|---|---|---|
| キャッチコピー | 音の粒で紡ぐ、無限の創造性（雨・静） | リズムを、植える。（森・動） |
| レイアウト骨格 | ジグザグ | **継承** + グリッド signature |
| フォント DNA | セリフ display | **継承**(Fraunces) + mono 機械感 |
| signature | 雨アニメ(3層) | ライブグリッド(7アルゴ) |
| 配色 | Mist(明) | **Asfalt(暗)** ← 対比 |

---

## 9. リスク・制約

- **R1**: 画像非存在 → CSS/SVG 図解で代替（B 判断）。完成度は図解の質に依存。
- **R2**: ダークテーマ vs GuRain 明テーマの視覚的ギャップ → ヘッダーリンクで行き来可能なら違和感なし。判断 A。
- **R3**: グリッド JS のパフォーマンス → 128セル更新は軽量。requestAnimationFrame + reduced-motion 安全装置。
- **R4**: npm install 未実施 → S9 で要実行（パッケージ追加・確認取得）。

---

## 10. 検証基準（完了条件）

- [ ] `npm run dev` で `/gurove/` が表示される
- [ ] グリッドが7アルゴリズムを循環発光（目視）
- [ ] reduced-motion でグリッド静止
- [ ] モバイル幅(375px)でレイアウト崩れなし
- [ ] GuRain 側（/gurain/）に影響なし（diff 確認）
- [ ] キーボードフォーカス可視
