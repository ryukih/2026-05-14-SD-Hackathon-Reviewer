# 評価検証レポート: shibuya-sim

**検証日**: 2026-05-11
**対象**: `shibuya-sim_eval.md`
**元評価合計**: 35.0/40

---

## 検証サマリー

| カテゴリ | 元スコア | 検証結論 | 根拠妥当性 | 主な所見 |
|---|---|---|---|---|
| A. 創発設計 (rules×freedom) | 9.0/10 | 妥当 | 概ね✓（行番号ズレあり） | 7ニーズ×POI副作用、5層会話確率、LLMへ自由判断委譲の事実関係はすべて実在。引用行番号に系統的誤差あり |
| B. 世界設定 (originality) | 9.0/10 | 妥当 | ✓ | 渋谷駅500m四方、citation付きdemographics、12アーキタイプ、OSM POI、A/B対照、3層創発指標すべて実在 |
| C. 発展性 (modularity+vision) | 8.0/10 | 妥当 | ✓ | モジュール分離は事実、READMEプレースホルダ残置も事実、`llm.fleet` `NotImplementedError` も実在 |
| D. 技術実装 (LLM+Mem+CodeQ+SimCorrect) | 9.0/10 | 概ね妥当（やや上振れ） | ⚠ | LLM頑健性、3層メモリ、2フェーズ同期、manifest出力すべて実在。ただし `simulation.py` 行数表現と引用行番号にズレ |
| **合計** | **35.0/40** | **概ね妥当（±0〜−1）** | | スコア水準は適切。誇張なし。引用行番号の精度に難 |

**総合判定**: 元評価のスコア（35/40）はソースコードの実在内容と整合しており、誇張・捏造は見られない。ただし `src/simulation.py` 内の引用行番号が系統的に若干ズレている（おおむね20〜40行下方）点は要修正。本質的なスコア改定は不要。

---

## カテゴリ別検証

### A. 創発設計 — 元スコア 9.0/10

**根拠の検証**: ✓（行番号にズレあり）

- `src/needs.py:36-65` ✓ — 実際の `_POI_NEED_DELTA` 定義は36-65行に存在。POI種別ごとの副作用を厳密にエンコード。
- `src/conversation_prob.py:79-125` ✓ — `conversation_probability` 関数は83〜131行で、L1関係性 / L2トリガー / L3性格 / L4文脈 / 補助のneeds_boost の5層構造が実装されている。
- `src/simulation.py:217-308` ⚠ — `build_prompt` は実際189〜360行。引用範囲は関数中盤を切り取った形で正確性に欠ける（関数本体は189行から始まる）。
- `src/simulation.py:283-291` ✓ — `schema_fields` の出力スキーマ宣言（action: "move"|"stay" 等）は実在し型のみを規定。
- `src/simulation.py:329-336` ✗ — 該当行に「Choose stay when … prefer to move」は存在せず、実際は **369行目**。約33行のズレ。文言自体は実在し、評価の「軽微な方向付けヒント」という指摘自体は妥当。
- 創発検出スクリプト `scripts/detect_emergence.py` の存在も確認（10049 bytes、L1 fiction / L1 attention / L3 norms の3指標）。

**スコア妥当性**: 9.0/10 は妥当。世界側（7ニーズ・5層会話確率・POI滞留・関係性ウォームアップ・SNS層）の規則設計と、エージェント側へ「生データのみを渡し行動はLLM任せ」という構造は事実であり、創発ポテンシャルは設計上極めて高い。

**見落とし・誇張**:
- 見落とし: `_PER_STEP_DRIFT`（needs.py 70-77行）の時刻ベースドリフト、`_needs_boost`（loneliness/boredom が会話確率を押し上げる正のフィードバックループ）の言及がない。これらは創発ループの重要要素。
- 誇張なし。むしろ「prefer to move」は単なるヒントではなく、`"most people don't stand still without reason"` という **明確な行動誘導文** で、創発設計の純度を若干損なう側面はある。減点幅は小（-0.5以下）。

**検証結論**: スコア維持（9.0/10）。引用行番号の修正を推奨。

---

### B. 世界設定 — 元スコア 9.0/10

**根拠の検証**: ✓

- `config/demographics_shibuya.yaml:1-30` ✓ — `area_target: "Shibuya station 500m square (35.6595, 139.7005)"`、`time_target: "weekday 15:00-22:00 JST"`、`citations: [R, C, T, S, P]`（国勢調査・通勤通学流動・観光・区基本計画）すべて確認。手書きエンコード明示。
- `config/archetypes.yaml:9-180` ✓ — 12アーキタイプ（OL/IT/高校生/大学生/飲食店経営/販売/クリエイティブ/高齢者/海外観光客/年配ビジネス/公務員/文化系）weight合計1.00 を確認。渋谷駅前の流動人口比率と整合的に重み付け。
- `config/config.yaml:25-37` ✓ — `conditions: A_physical_only / B_physical_internet` の対照群設計確認（実際は29-33行付近）。
- `scripts/detect_emergence.py:11-15` ✓ — `L1 fiction (proper nouns not matching POIs)` / `L1 attention (converging vocabulary)` / `L3 norms (〜すべき, 〜は普通...)` の3指標を確認。
- `data/pois.geojson` ✓ — 40028 bytes、Featureが102個（OSM由来の実POI、`fetch_pois.py` でキャッシュ生成）。

**スコア妥当性**: 9.0/10 は妥当。「SNSの有無が物理空間での集合的注目や架空地名生成、規範発話の創発に影響を与えるか」という研究的問いは、メディア社会学・情報拡散研究の核心トピックを直接モデル化しており、抽象都市シミュレーションを超えた独自性が際立つ。

**見落とし・誇張**:
- 見落とし: `config/events.yaml`（時刻指定イベント：雨/演説/セール等）と `config/environment.yaml`（2時間バケットの天気・気温・ニュース話題）の存在、`smartphone_use` 計算式（年齢×職業×性格×興味の加重和）にも独自性がある。
- 誇張なし。「OSMから実POI」「citation付き人口分布」「12アーキタイプ」「対照群設計」はすべて事実。

**検証結論**: スコア維持（9.0/10）。

---

### C. 発展性 — 元スコア 8.0/10

**根拠の検証**: ✓

- `src/llm_client.py:111-145` ⚠ — `get_client` ファクトリは実際 132-170行。範囲がやや早い側にズレ。実装内容（provider/model で動的分岐）は正確。
- `config/config.yaml:53-66` ✓ — `llm.fleet` セクション（low: gpt-4o-mini 60%, medium: gemini-2.0-flash 30%, high: claude-haiku-4-5 10%）を確認。実際は55-66行。tier名はlow/medium/highで、評価の「cognitive_tier ごとに別LLM」という記述と整合。
- `src/llm_client.py:117-119` ⚠ — `if mode \!= "single": raise NotImplementedError` は実際 137-140行。行番号ズレあり、内容は正確。
- README冒頭プレースホルダ（`<\!-- # プロジェクト概要（ここはご自分で記述してください...）-->`）✓ — 確認済み。
- `scripts/` 配下 ✓ — `generate_pool.py / warmup_relationships.py / aggregate_runs.py / detect_emergence.py / estimate_cost.py / build_personas_index.py / fetch_demographics.py / fetch_pois.py / summarize_run.py / run_metrics.py / debug_one_call.py / test_llm.py` の12スクリプトを確認。パイプライン各段階のツールが揃う。

**スコア妥当性**: 8.0/10 は妥当。

- 加点要因: モジュール分離（src/ 7ファイル、単一責任）、設定の8YAML外部化、スクリプト群の充実、`llm.fleet` スキャフォールドの先取り、データクラス `Agent` のフィールド設計の余裕度（4thレイヤペルソナ属性まで含む拡張準備）。
- 減点要因: READMEの「プロジェクト概要」プレースホルダ未記入、`llm.fleet` 未実装、`scripts/aggregate_runs.py` などのスクリプトが揃っているが、ロードマップ文書がない。

**見落とし・誇張**: 誇張なし。むしろ `src/agent.py:48-60` で4thレイヤペルソナ属性（attitude/speech_register/money_sense/social_density/place_criteria/time_pace/contradiction/past_setback）が宣言済みで拡張余地が大きいことに触れられていない。

**検証結論**: スコア維持（8.0/10）。READMEプレースホルダ未記入は確かに減点要因として妥当。

---

### D. 技術実装 — 元スコア 9.0/10

**根拠の検証**: ⚠（誤差あり、概ね正確）

- `src/llm_client.py:55-79` ⚠ — Ollama リトライループは実際 67-82行。内容（`for attempt in range(self.max_retries + 1)` + `time.sleep(1.0 * (attempt + 1))`）は正確。
- `src/llm_client.py:101-108` ⚠ — OpenAI `response_format={"type": "json_object"}` は実際 120-123行。行ズレあり。
- `src/simulation.py:507-518` ⚠ — `parse_decision` は実際 496-510行。`first = s.find("{")`, `last = s.rfind("}")` による頑健JSONパースを確認。
- `src/simulation.py:798-806` ⚠ — LLM失敗時のフォールバック決定（`decision = {"thinking": f"error: {e}", "action": "stay", ...}`）は実際 808-816行付近。内容は正確。
- `src/simulation.py:243-251` ⚠ — `[past acquaintance: warmth=...]` 関係性メモリ再注入は実際 226-235行付近。内容は正確（warmth/common_topics/first_impression を整形してプロンプトへ）。
- `src/simulation.py:610-682` ⚠ — `manifest.json` 出力は実際 700-720行付近。行ズレあり。`run_id` は `{cond_id}_seed{seed}_{timestamp}` で生成、`config_snapshot` も含む。
- `src/simulation.py:822-823` ⚠ — 2フェーズ同期（全エージェントの `decisions` 収集後に apply phase）は実際 820行付近。内容は正確。
- `src/agent.py:48-60` ⚠ — relationships/needs/memory/current_goal の明示的フィールドは実際 33-65行（agent.py全体は67行）。内容は正確。

**スコア妥当性**: 9.0/10 はやや上振れだが妥当範囲。

- 加点要因（事実確認済）: ① LLMクライアントの Protocol 抽象、`format_json` で `response_format=json_object` 強制、コードフェンス除去＋`{...}` 抽出による頑健パース、try/except でのフォールバック決定。② 3層メモリ（rolling memory ≤5行、`current_goal`、`relationships`）が意味的に分離されてプロンプト再注入。③ 同期2フェーズ（収集→適用）で競合回避、シード分離（`phone_rng`, `sample_rng`）、manifest+config_snapshot出力、JSONL構造化ログ。
- 減点要因: `src/simulation.py` は **958行**（「約1000行」とほぼ整合、わずかに過大）でモノリシック。プロンプト構築・IO・apply phase・persona loading が同居。

**見落とし・誇張**:
- 誇張: 「約1000行」は958行で概ね正確、許容範囲。
- 見落とし: ① `phone_rng = random.Random(f"{seed}-phone-check")` のような **seed派生RNGによる完全決定性** はかなり丁寧な実装で言及に値する。② `needs.py` の `init_for_persona` が persona の extraversion に応じて初期 loneliness を変えるなど、シミュレーション正確性に寄与する細部もある。③ Ollama/OpenAI 両対応で **ローカル開発→クラウド本実験** のワークフローが現実的。
- LLM呼び出しが各エージェント逐次（並列化なし）でスケールしないことには触れられていない（200エージェント×96ステップ＝19,200呼び出しが直列）。

**検証結論**: スコア維持可（9.0/10）。並列化欠如への言及があれば 8.5 寄りの評価にもなり得るが、ハッカソンMVPとしては許容範囲。

---

## 総合所見

### 元評価の信頼性
スコア（A=9 / B=9 / C=8 / D=9 / 合計35）はすべてソースコードの実在内容と整合しており、捏造・虚偽記載はない。文章記述（「7ニーズ×2層」「5層会話確率」「12アーキタイプ」「3層メモリ」「2フェーズ同期」「manifest出力」「detect_emergence の3指標」など）は実装と一致している。

### 主な問題点
1. **引用行番号の系統的ズレ**: `src/simulation.py` の引用行が概ね **20〜40行下方** にズレている（最大はA項目の「329-336 → 実際369」で約33行）。原因はおそらく評価時に異なるリビジョンを参照したか、行カウントの誤り。`src/llm_client.py` でも10〜30行のズレ。**スコアへの影響は無いが、検証者の手間が増えるため修正推奨**。
2. **モノリシック性の表現**: 「`src/simulation.py` が約1000行」は実際958行で誇張気味ではないが「約950行」がより正確。
3. **見落とし**: 並列化欠如、`smartphone_use` 計算の独自性、`_needs_boost` の正フィードバックループ、phone_rng/sample_rng の決定性設計 — これらに触れるとさらに説得力が増す。

### 推奨スコア調整
- A: 9.0 → **9.0** 維持（「prefer to move」が予想以上に強い誘導文だがMVPとして許容）
- B: 9.0 → **9.0** 維持
- C: 8.0 → **8.0** 維持
- D: 9.0 → **8.5〜9.0** 並列化への言及があれば 8.5、なくとも MVP として 9.0 維持可

**最終総合**: **34.5〜35.0/40 が妥当**。元評価の 35.0 はわずかに上限寄りだが許容範囲内。

### 結論
元評価は**実装内容を正確に把握しており、誇張・捏造はない**。引用行番号の精度のみ要修正。スコア改定は不要。shibuya-sim は本ハッカソン提出物として、研究的問い・世界設計・コード構成・LLM統合の各面で突出した完成度を示しており、評価レポートの「READMEの研究目的明文化」「`simulation.py` 分割」「`llm.fleet` 実装または将来計画明示」という改善提言も的を射ている。
