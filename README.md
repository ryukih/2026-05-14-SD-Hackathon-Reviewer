# SD Hackathon Reviewer — 第1回ハッカソン評価レポート

LLM ベース・マルチエージェント・シミュレーションのハッカソン（2026-05-14 開催）の提出物について、共通基準で自動評価した結果を公開するリポジトリです。

> **本評価は Claude（Anthropic）による LLM 自動採点であり、人間審査ではありません。** 評価軸・方法論・既知の限界を本書で明示したうえで、参考情報として共有します。

---

## 🎯 このリポジトリについて

- **目的**: 全提出物を同一ルーブリックで採点し、結果を作者・関心者に透明に共有する
- **対象読者**: ハッカソン提出者、および本評価結果を参考にしたい関係者
- **採点者**: Claude Code 上の `evaluate-submission` スキル（LLM による自動評価）

---

## 📊 自分の評価結果を見る

各提出物に対して、以下の 2 種類のレポートを公開しています。

| ファイル | 内容 |
|----------|------|
| `evaluations/20260511-1st-Hackathone-valuations/<repo_name>_eval.md` | 4 軸スコア・根拠引用・総評を含む評価本体 |
| `evaluations/20260511-1st-Hackathone-valuations/<repo_name>_eval_review.md` | 上記評価の独立再検証（引用根拠・スコア妥当性の精査） |

- **総合ランキング（最終結果一覧）**: [`evaluations/README.md`](evaluations/README.md)
- **検証総括**: [`evaluations/20260511-1st-Hackathone-valuations/_VERIFICATION_SUMMARY.md`](evaluations/20260511-1st-Hackathone-valuations/_VERIFICATION_SUMMARY.md)
- **発表スライド**: [`slides/`](slides/) 配下
  - 命名規則: `順位-研究員番号-リポジトリ名.pdf`
  - `s` プレフィックス（例 `s14-...`）はコード未添付などの理由で**未評価**扱いとなった提出物

---

## 📏 評価軸（40 点満点）

| カテゴリ | 評価軸 | 配点 |
|---------|--------|------|
| **A. 創発設計** | 世界ルールの設計精度 × エージェント行動の自由度のバランス | /10 |
| **B. 世界設定** | シナリオの独創性・ビジネス／社会科学的意義の深さ | /10 |
| **C. 発展性** | コードの拡張性・将来展望の具体性 | /10 |
| **D. 技術実装** | LLM 利用・メモリ設計・コード品質・シミュレーション正確性 | /10 |

各軸の詳細な採点ガイドラインと根拠の書式は [`.claude/skills/evaluate-submission/SKILL.md`](.claude/skills/evaluate-submission/SKILL.md) を参照してください。

---

## ⚙️ 評価方法論

### 採点プロトコル

1. 各カテゴリを**独立に 2 回**スコアリング（Run 1 / Run 2）
2. いずれかのカテゴリで両者の差が **3 点以上**ある場合、Run 3 を実施
3. 最終スコア = 実施回数分の平均（小数第 1 位で丸め）

このプロトコルは、LLM 採点の分散を平均化で抑制するためのものです。詳細は [`SKILL.md` の Phase 1–5](.claude/skills/evaluate-submission/SKILL.md) を参照。

### 検証フロー

各 `_eval.md` に対し、独立に `_eval_review.md` を生成し、以下を再点検しています。

- 引用根拠（ファイル名・行番号・README 引用）の正確性
- スコア妥当性（ルーブリックとの整合）
- 見落とし・誇張・事実誤認の有無

総括は [`_VERIFICATION_SUMMARY.md`](evaluations/20260511-1st-Hackathone-valuations/_VERIFICATION_SUMMARY.md) にまとめています。

---

## ⚠️ 評価の限界・免責事項

本評価には以下の既知の限界があります。**スコアを絶対的な評価として受け取らず、参考情報として解釈してください。**

- **LLM 採点には分散がある**: 同一コードを複数回採点しても結果はブレます。本評価は平均化で抑制していますが、±1 点程度の揺らぎは残ります。
- **相対評価であり絶対評価ではない**: スコアは「全提出物を一貫した尺度で並べる」ためのものであり、個別作品の絶対的価値を断定するものではありません。
- **コード未添付の提出物は減点または評価対象外**: ソースコードが添付されていない提出物は引用根拠を検証できないため、低めのスコアまたはランキング対象外となります。（該当: `echo-mortality-lab`, `fukushima-fire-sim`）
- **引用ズレ・パラフレーズ・稀な事実誤認**: 再検証で行番号の軽微なズレや意味等価な言い換え、稀に事実誤認が確認されています。詳細は `_VERIFICATION_SUMMARY.md` をご覧ください。
- **採点プロトコルの逸脱事例**: 一部の評価で 3 回目スコアリングが規定どおり実施されていないケースが検出されています。
- **本評価は「初回評価として提示」する性格のものです**。異議申し立て・再評価リクエストの公式窓口は設置しておりません。

---

## 🏆 総合ランキング

総合点（4 軸合計・40 点満点）の降順ランキングは [`evaluations/README.md`](evaluations/README.md) にまとめています。同点処理・未評価提出物の扱い・リファレンス実装スコアも同ページに記載しています。

---

## 📁 リポジトリ構成

```
SD-Hackathon-Reviewer/
├── README.md                                         # 本ファイル
├── submissions/                                      # 各提出物のソースコード
│   ├── README.md                                     # 提出物一覧
│   └── <repo_name>/                                  # 各チームのリポジトリ
├── evaluations/                                      # 評価結果
│   ├── README.md                                     # ランキング表と評価軸の説明
│   └── 20260511-1st-Hackathone-valuations/
│       ├── _VERIFICATION_SUMMARY.md                  # 再検証の総括
│       ├── <repo_name>_eval.md                       # 各提出物の評価レポート
│       └── <repo_name>_eval_review.md                # 上記の独立再検証
├── slides/                                           # 発表スライド PDF
└── .claude/skills/
    └── evaluate-submission/SKILL.md                  # 評価スキル定義
```

---

## 🔧 評価ツールについて

評価は Claude Code 上で動く `evaluate-submission` スキルで実施しています。仕組みを確認・再現したい場合は以下を参照してください。

- スキル定義: [`.claude/skills/evaluate-submission/SKILL.md`](.claude/skills/evaluate-submission/SKILL.md)
- 起動方法: Claude Code から `/evaluate-submission <repo_name>` を実行すると `submissions/<repo_name>/` を読み込み、`evaluations/<repo_name>_eval.md` を出力

---

## 📜 注意事項とクレジット

- **個人情報**: 本リポジトリでは作者の本名を公開していません。研究員名（ハンドル）と研究員番号、公開 GitHub リポジトリ URL のみ記載しています。
- **著作権**: 各提出物の著作権は各作者に帰属します。本リポジトリは公開済みの GitHub リポジトリをサブモジュール／クローンとして集約しているものです。
- **評価レポートの著作権**: `evaluations/` 以下の評価レポートは Claude（Anthropic）が生成したものを編集・公開しています。
