# Quantum-Classical Hybrid Machine Learning Framework
# 量子古典ハイブリッド機械学習フレームワーク

A quantum-classical hybrid classifier that achieves classical-competitive accuracy without any preprocessing, using fixed quantum feature extraction, sector-split encoding, and STE noise reduction.

前処理なしで古典手法と同水準の精度を達成する量子古典ハイブリッド分類器。固定量子特徴抽出・セクター分割エンコーディング・STEノイズ除去を組み合わせた独自設計。

---

## Research Overview / 研究概要

### Architecture / アーキテクチャ

```
Raw Input (no preprocessing)
生データ（前処理なし）
        ↓
Quantum Circuit (Fixed Weights)
量子回路（固定重み）
  - Sector-Split Angle Encoding
    セクター分割角度エンコーディング
  - CNOT Entanglement Layer
    CNOTエンタングルメント層
  - Pauli Z/X/Y Measurement
    パウリ Z/X/Y 測定
        ↓
STE Layer (Noise Reduction)
STE層（ノイズ除去）
        ↓
Classical Network (GPU)
古典ネットワーク（GPU）
  - Linear → GELU → Dropout → Linear
        ↓
Classification Output
分類出力
```

### Key Techniques / 主要技術

**① Fixed Quantum Feature Extractor / 固定量子特徴抽出器**
Quantum weights are fixed (not learned). The quantum circuit acts as a universal preprocessor.
量子重みは学習しない。量子回路が汎用前処理器として機能する。

**② Sector-Split Encoding / セクター分割エンコーディング**
A single physical axis (360°) is divided into multiple sectors, enabling high-dimensional data to be embedded without collision. This is equivalent to a software implementation of qudit.
1本の物理軸（360度）を複数のセクターに分割し、高次元データを衝突なく埋め込む。これはquditのソフトウェア的実装に相当する。

**③ STE (Straight-Through Estimator) Layer / STE層**
Quantizes quantum circuit output with `step_size=0.01` to suppress noise, while preserving gradient flow to the classical layer.
量子回路出力を `step_size=0.01` で量子化してノイズを抑制しつつ、古典層への勾配を通す。

**④ Seed Search / Seed探索**
Quantum space structure is determined by random seed. An optimal seed is found by parallel search, exploiting the "spatial limit accuracy" property.
量子空間の構造はランダムSeedで決まる。「空間限界精度」の性質を利用して並列探索で最適Seedを発見する。

---

## Experimental Results / 実験結果

All results are achieved without any dataset-specific preprocessing (no normalization, no feature selection, no SMOTE). The quantum circuit itself provides an internal normalization effect through angle embedding, eliminating the need for manual preprocessing steps.

すべての結果はデータセットに特化した前処理なし（正規化・特徴選択・SMOTE等を一切使用しない）で達成。量子回路が角度埋め込みを通じて内部的な正規化効果を持つため、手動の前処理が不要になっている。

| Dataset | Dimensions | Classes | Samples | Best Accuracy |
|---------|-----------|---------|---------|---------------|
| BreastCancer | 30D | 2 | 569 | **99.12%** |
| Ionosphere | 34D | 2 | 351 | **95.77%** |
| Wine | 13D | 3 | 178 | **100.00%** |
| Parkinsons | 22D | 2 | 195 | **94.87%** |
| Iris | 4D | 3 | 150 | **100.00%** |
| Sonar | 60D | 2 | 208 | **78.57%** |
| Diabetes | 8D | 2 | 768 | **76.62%** |
| Glass | 9D | 6 | 214 | **74.42%** |
| Optdigits | 64D | 10 | 1797 | *TBD* |

> \* Optdigits is included in the preceding paper but has been temporarily excluded from this table as re-experimentation under the current configuration (seed search + cache generation + full training) requires significant computation time and is not yet complete. Results will be added upon completion.
>
> \* OptdigitsはJXiv先行論文に掲載済みだが、現在の構成（Seed探索＋キャッシュ生成＋本学習）での再実験に相応の計算時間が必要なため、完了次第追記予定。

Detailed seed search logs are available in `logs/`.
詳細なSeed探索ログは `logs/` に収録されています。

---

## Supported Datasets / 対応データセット

Loaded automatically from UCI ML Repository or scikit-learn.
UCI MLリポジトリまたはscikit-learnから自動ロード。

| Dataset | Source |
|---------|--------|
| BreastCancer | scikit-learn |
| Ionosphere | UCI ML Repository |
| Sonar | UCI ML Repository |
| Wine | UCI ML Repository |
| Parkinsons | UCI ML Repository |
| Heart_Disease | UCI ML Repository |
| Iris | jbrownlee/Datasets |
| Glass | jbrownlee/Datasets |
| Diabetes | jbrownlee/Datasets |

---

## Installation & Usage / インストールと使い方

### Requirements / 動作環境

- Windows 10/11 (64-bit)
- No Python installation required / Python不要

### Download / ダウンロード

Download `QML_Inference.zip` from the [Releases](../../releases) page and extract it.
[Releases](../../releases) ページから `QML_Inference.zip` をダウンロードして解凍してください。

```
QML_Inference/
  QML_Inference.exe   ← Run this / これを実行
  _internal/          ← Required (do not delete) / 必須（削除しないこと）
  models/             ← Pre-trained models / 学習済みモデル
```

### Launch / 起動

Double-click `QML_Inference.exe`.
`QML_Inference.exe` をダブルクリックして起動。

### How to Use / 使い方

1. **Select a model / モデルを選択**
   Choose a `.pt` file from the left panel.
   左パネルから `.pt` ファイルを選択する。

2. **Run Inference / 推論を実行**
   Click `>> Run Inference`. The dataset is downloaded automatically.
   `>> Run Inference` をクリック。データセットは自動でダウンロードされる。

3. **View Results / 結果を確認**
   Train / Test / All accuracy and quantum circuit raw output are displayed.
   Train / Test / All の精度と量子回路の生出力が表示される。

> **Note:** Internet connection required for first run (dataset download).
> **注意:** 初回実行時はデータセットのダウンロードのためインターネット接続が必要です。

---

## Model Files / モデルファイル

Pre-trained models are included in `models/`. Filename format:
学習済みモデルは `models/` に含まれています。ファイル名の形式：

```
{Dataset}_{N}QB_{L}L_{H}HL_S{Seed}_best.pt

Example: BreastCancer_7QB_2L_32768HL_S63_best.pt
  Dataset : BreastCancer
  7QB     : 7 Qubits
  2L      : 2 Layers
  32768HL : 32768 Hidden Nodes
  S63     : Seed 63
```

---

## Repository Structure / リポジトリ構成

```
quantum-plateau-bypass/
  models/       ← Pre-trained model files (.pt) / 学習済みモデル
  logs/         ← Seed search logs (.csv) / Seed探索ログ
  README.md
```

Inference software is distributed via Releases (binary only).
推論ソフトはReleases経由で配布（バイナリのみ）。

---

## Citation / 引用

> Preprint available on JXiv / JXivにてプレプリントで公開予定

---

## License / ライセンス

The pre-trained models and inference software are provided for research reproducibility only.
学習済みモデルおよび推論ソフトウェアは論文の再現性確認を目的として提供しています。
