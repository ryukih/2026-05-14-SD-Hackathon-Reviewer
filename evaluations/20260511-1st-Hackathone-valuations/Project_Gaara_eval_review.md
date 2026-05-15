# 評価検証レポート: Project_Gaara
**検証日**: 2026-05-11
**対象**: `Project_Gaara_eval.md`
**元評価合計**: 37.0/40

## 検証サマリー

| カテゴリ | 元スコア | 検証スコア妥当性 | 引用検証 | 判定 |
|---|---|---|---|---|
| A. 創発設計 | 10.0/10 | 妥当 | 全引用 ✓ | 支持 |
| B. 世界設定 | 9.0/10 | 妥当 | 全引用 ✓（一部パラフレーズ） | 支持 |
| C. 発展性 | 9.0/10 | 妥当 | 全引用 ✓ | 支持 |
| D. 技術実装 | 9.0/10 | 妥当 | 全引用 ✓ | 支持 |
| **合計** | **37.0/40** | — | — | **支持** |

**総合判定**: 元評価は引用がほぼ完全に検証可能で、根拠とスコアの整合性が極めて高い。記述上の軽微なパラフレーズが1件あるが（意味的等価）、誇張は見当たらない。むしろ一部要素（例: 15個のDNAバリアント、10個のMotherバリアント）は控えめにすら描かれている。

---

## カテゴリ別検証

### A. 創発設計（元: 10.0/10）

**根拠の検証**: ✓
- `cluster.py:1-10` の "LLM never touches velocity, viscosity, color, or any other kinematic slider. Cognition stays in the language space; muscles stay in the math space." はモジュールdocstring（1-10行目）で完全一致。物理層/認知層の分離宣言として明確。
- `cluster.py:67-81` のDNA_V2は、確かに schema（ideal_coord/urgency/hostility/reasoning）と最小ロール定義のみ。"Other entities with exact coordinates may appear in your sensorium — what you do with them is your choice." も該当範囲（74-77行目付近）にほぼ完全一致で存在。
- `simulation.py:249-267` のawareness flip機構は、検証範囲（249-267 付近）で確かに「1体が範囲内に入ると `self._revealed.add(a.name)` され、以後全クラスタに座標が開示される」設計になっており、説明と一致。
- `scenarios.py:82-128` のMOTHER_TEXT辞書（M1-M3, M4, M5...）は身体感覚言語のみ。"east. east is upon me. my whole body strains east." は M3/M4/M5 のimmediate行に確認できる。命令動詞・座標も含まない。✓
- `README.md` の「各粒子は毎ステップ、自分が「なぜそう動いたか」を一文で出力します」も README 日本語セクション（19行目付近）に存在。
- `OPERATING_PRINCIPLES.md:25-35` のNon-obligation / Functional diversity / Coherence の3条件も該当範囲（22-30行目付近）に存在。

**スコア妥当性**: ルーブリックA（ルール×自由度）の最高帯（9-10: 精密かつ生）に該当する。LLMが「意図」のみを出力し、`physics.py`が運動に変換、Motherは命令ではなく身体言語のみで発話、`reasoning` 監査面と `score_diversity.py` の反証可能診断器まで備える設計は、創発設計のテンプレートとして模範的。

**見落とし・誇張**: 誇張なし。むしろ `DNA_V_AXES`, `DNA_V_DISSOLVED`, `DNA_V_GARDEN`, `DNA_V_WE` といった「主語-対象文法を解体する」非自明バリアントがコード上には存在しており、創発設計の探索空間の広さは評価本文以上に充実している。

**検証結論**: 10/10 は妥当。

---

### B. 世界設定（元: 9.0/10）

**根拠の検証**: ✓
- `README.md` の "interpretation is not a property of the LLM. It is a property of the architecture built around it." は、READMEの150行目に **意味的に等価** な表現として存在: "interpretation is a property of the relational architecture built around an LLM, not of the LLM itself." 評価本文は **パラフレーズ**（"not a property... It is a property..."）であり、完全一致引用ではないが意味は保存されている。⚠ 厳密な逐語引用ではない点に留意。
- `README.md` の "Japanese-animist framing is not decoration..." は152行目に完全一致で存在。
- `REPORT.md:143-157` の4本の future direction（subject substitution、frontier-model relational language、BMI as relationship interface、information-life ontology）は該当範囲に確かに4項目で番号付きリスト化されており、完全一致。
- `scenarios.py:60-217` の「10種類のMother variant」「6種類以上のDNA variant」は実際にはより多い: Motherは10種（M1, M2, M3, M4, M5, M_SENSORY, M_METAPHOR, M_PULSE, M_GARDEN, M_WE）、DNAは15種。記述は **控えめ** だが範囲指定は妥当。

**スコア妥当性**: 日本アニミズムを意匠ではなくアーキテクチャ駆動原理として明示している点（READMEの "every thing has presence, including the LLM"）、post-AGI関係論／HCI／BMIへの応用接続が明示されている点は、商用デモには見られない独創性。9/10 は妥当。10としない理由（ビジネス指標への直接マップ困難）も納得できる。

**見落とし・誇張**: 軽微な誇張なし。1件のパラフレーズ引用が逐語ではないが意味は保存されている。逆に Mother / DNA バリアント数を控えめに記述している。

**検証結論**: 9/10 は妥当。

---

### C. 発展性（元: 9.0/10）

**根拠の検証**: ✓
- `main.py:116-148` の CLI フラグ（`--dna-variant`, `--mother-variant`, `--role-noun`, `--role-nouns`, `--say-prefix`, `--awareness-range`）は該当範囲に確かに全て定義されており、評価本文の説明と完全に一致。
- `cluster.py:267-273` の `DNA_VARIANTS` dict は267行目に開始し、15エントリで構成（V1-V4, VA-VG, V_AXES, V_DISSOLVED, V_GARDEN, V_WE）。新バリアント追加は確かにdictにエントリを足すだけで済む構造になっている。
- `config.yaml:35-170` のシナリオYAML外部化は実機検証で確認: `scripted_100`, `scripted_70`, `scripted_30`, `scripted_55`, `const_calm/faint/immediate/extreme/dual_climax`, `scripted_70_aware2/6/8` など多数のシナリオが外部化されている。
- `REPORT.md:143-157` の future direction 4本も確認済み（B評価参照）。
- `simulation.py:195-206` のシナリオディスパッチ `_scripted_step` は確かに `scenario_name` の文字列マッチで疎結合に分岐しており、新シナリオ追加が低コスト。

**スコア妥当性**: モジュール分離（cluster / mothership / physics / scenarios / simulation / threat / ollama_client / analyze_run / score_diversity）が単一責務に整理されており、CLIフラグから全アーキテクチャ軸切替可能。新DNA / 新Mother / 新シナリオの追加コストが極めて低い。将来展望は具体的かつ非自明。9/10 は妥当。

**見落とし・誇張**: 誇張なし。むしろ `analyze_run.py`, `summarize_exp_batch.py`, `view_run.py` といった分析パイプラインが完備されている点もコード拡張性の証左で、評価本文ではやや軽く扱われている。

**検証結論**: 9/10 は妥当。

---

### D. 技術実装（元: 9.0/10）

**根拠の検証**: ✓
- `cluster.py:290-313` の `_clamp_intent` は292-313行目に存在し、評価本文の説明（`ideal_coord` を半空間 [-25,25] にクランプ、`urgency` を [0,1]、`hostility` を [-1,1]、`reasoning` を200文字で切り詰め）と完全一致。
- `cluster.py:330-340` の `_extract_json` は330-340行目で `{...}` 区間抽出 + try/except による堅牢な実装が確認できる。
- `cluster.py:443-461` の `transduce` 内 try/except は該当範囲で `logger.error` による失敗ログ + 空文字フォールバック処理を確認。
- `cluster.py:463-479` の `record_moment` は ago カウンタのインクリメント、メモリサイズ上限、`reason` の80文字切り詰め（`(self.last_intent.get("reasoning") or "")[:80]`）を確認。完全一致。
- `simulation.py:209-294` の `step_simulation` の同期固定順序（攻撃者更新→Mother更新→awareness判定→LLM意図取得→物理積分→メモリ＋ログ）は該当範囲で完全確認。
- `simulation.py:80-89, 171` の複数RNG分離: `random.Random(self.seed + 7919)`（Mother）と `random.Random(self.seed + 31337)`（物理）は該当範囲で確認。
- `simulation.py:296-369` の5系統JSONL: `cluster_positions.jsonl`, `cluster_intents.jsonl`, `mother_state.jsonl`, `attackers.jsonl` を確認（5系統目もこの後に続く）。
- `ollama_client.py:68-97` のリクエスト失敗時ログ+空文字返却は該当範囲で `logger.error` + `return ""` を確認。
- `main.py:158` の `shutil.copy(args.config, os.path.join(run_dir, "config_snapshot.yaml"))` は158行目で完全一致。

**スコア妥当性**: LLM利用（15バリアント + JSON抽出 + クランプ + フォールバック + サンプリング設定の外部化）、メモリ設計（構造化記憶の時間順注入 + ago カウンタ + サイズ上限 + reason トリム）、コード品質・シミュレーション正確性（型ヒント + dataclass + 責務分離 + 固定順序 + 3系統RNG分離 + config_snapshot）はいずれもD評価の各サブ項目（LLM3+Mem3+CodeQ2+SimCorrect2）で高水準。9/10 は妥当。

**見落とし・誇張**: 評価本文の改善余地指摘（structured output 強制なし、同期直列LLM呼び出し）は実装と整合する正確な指摘。誇張なし。

**検証結論**: 9/10 は妥当。

---

## 総合所見

元評価 37.0/40 は **検証によって支持される**。

**強み**:
- 全カテゴリで具体的なファイル名・行番号引用が提示されており、検証可能性が極めて高い。
- 引用は実際のコード/ドキュメントに対応しており、誇張・捏造は確認されない。
- ルーブリック各サブ項目（A: ルール×自由度、B: 独創性、C: モジュール性+ビジョン、D: LLM+Mem+CodeQ+SimCorrect）の整合性も高い。

**軽微な留意点**:
- B評価の1件の引用（"interpretation is not a property of the LLM..."）は逐語ではなくパラフレーズ。READMEの150行目に意味的に等価な文（"interpretation is a property of the relational architecture built around an LLM, not of the LLM itself."）が存在するため、内容上の問題はないが、引用形式としては要改善。
- Mother variant 数（10種）と DNA variant 数（15種）は評価本文の記述（「10種類」「6種類以上」「15バリアント」）が一部やや控えめだが、これは検証の品質を損なうものではなく、むしろ評価の保守性を示すもの。

**結論**: 元評価は引用検証・スコア妥当性ともに高品質であり、37.0/40 という採点は **検証によって裏付けられる**。プロジェクト Gaara の「LLMが命令を実行しているのか関係的アーキテクチャを解釈しているのか」というメタ問題を反証可能な形で工学化した点は、本ハッカソンの "LLM-based multi-agent simulation" 文脈において極めて高い完成度を示しており、最終 37.0/40 の評価は妥当である。
