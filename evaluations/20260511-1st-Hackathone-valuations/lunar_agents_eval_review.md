# 評価検証レポート: lunar_agents
**検証日**: 2026-05-11
**対象**: `lunar_agents_eval.md`
**元評価合計**: 37.0/40

## 検証サマリー

| カテゴリ | 元スコア | 検証結論 | 推奨スコア帯 |
|---|---|---|---|
| A. 創発設計 | 8.5/10 | 妥当 | 8.0-9.0 |
| B. 世界設定 | 10.0/10 | 妥当（やや上振れの可能性） | 9.0-10.0 |
| C. 発展性   | 9.5/10 | 妥当 | 9.0-9.5 |
| D. 技術実装 | 9.0/10 | 妥当 | 9.0-9.5 |

**総合判定**: 37.0/40 は妥当。すべての主要引用が実コードと一致しており、各カテゴリのスコアは根拠とよく整合している。Bが満点であることはやや甘めだが、世界観の独自性・現実ミッションとの結節度を考えれば許容範囲。Dは1,997行の巨大agent.pyという減点要素を著者自身が明示しているため、9.0は妥当な水準。

---

## カテゴリ別検証

### A. 創発設計（元: 8.5/10）

**根拠の検証**:
- ✓ `environments/lunar_2d/map.py:143-217` — `_apply_layered_crater` 関数が rim/wall/floor 三層、`slope_deg`、`roughness`、`traverse_cost`、`illumination_fraction`、`temperature_k`、`water_potential` を独立に管理しており、評価記述と完全一致。
- ✓ `environments/lunar_2d/agent.py:49-58` — `DEFAULT_BATTERY_COSTS` 辞書に `move=3.0, survey=2.0, mark_resource=1.0, stay=0.5, propose_build=0.5, assist_build=0.5, message=0.5, step=0.5` が定義されている。引用一致。
- ✓ `environments/lunar_2d/agent.py:41-44` — `SAMPLE_DECAY_PER_STEP = 0.85`、`SAMPLE_QUALITY_FLOOR = 0.10` が定義され、コメントも「return now timing pressure」と明記されている。引用一致。
- ✓ `environments/lunar_2d/orbital_data.py:99-146` — `personal_view` メソッドが `visibility_fraction` と `per_agent_noise_std` で per-agent ビューを生成、決定論的 RNG seed まで実装。引用一致。
- ✓ `environments/lunar_2d/agent.py:461-523` — `build_message_prompt` で coords、illumination、water_potential、battery を数値のみ提示し、message_type も列挙のみ。引用一致。
- ✓ README引用「LLM プロンプトには、座標、距離、バッテリー、含水率、確度などの定量情報を渡します。」は実在（README「設計原則」セクション）。
- ✓ README引用「『危険』『豊富』『安全』などの評価語はできるだけプロンプトに含めず、判断は LLM 側に委ねます。」も実在。
- ✓ `environments/lunar_2d/personas.py:36-44` — `Persona("cautious", "prioritize battery margin and proximity to base")` 等が定義され、まさに「行動傾向を示唆」する短文。引用一致。
- ✓ `environments/lunar_2d/agent.py:1360-1372` — `"WARNING: at east boundary — right is blocked, move left/up/down"` 等のヒントが `_format_exploration_hints` 内に実在。引用一致。

**スコア妥当性**: ルール側の精緻さ（地形・電池・サンプル品質減衰・per-agent観測非対称）と、persona description やナビゲーションヒントによる軽い行動誘導の両方を正直に評価しており、8.5/10 は適切。

**見落とし・誇張**:
- 見落とし: `loop detection`（評価本文で言及はあるが根拠引用なし）はコード上に明確に存在することを agent.py の `_format_exploration_hints` から推定できるが、評価では具体行が示されていない。減点には繋がらないが補強可。
- 誇張: なし。

**検証結論**: ✓ 妥当。8.5/10 は推奨レンジ内（8.0-9.0）。

---

### B. 世界設定（元: 10.0/10）

**根拠の検証**:
- ✓ README引用「Lunar Agents は、LLM エージェント群が 2D 月面環境で水氷資源を探索し、観測・記憶・通信を通じて協調や競争を形成するかを調べるためのシミュレーション基盤です。」は実在（README冒頭）。
- ✓ `CLAUDE.md` 引用「シミュレーションは国際月探査ロードマップ（LUPEX・Artemis・Chang'e 7等）から逆算した5類型のミッションに対応する。」は CLAUDE.md L11 に実在。
- ✓ `scenarios/south_pole_survival_survey/scenario.yaml:1-139` — Haworth様コールドトラップ（floor_water_potential=0.88、floor_temperature_k=82等）と各種地形要素を139行で定義。引用一致。ファイル総行数も139で完全一致。
- ✓ `environments/lunar_2d/orbital_data.py:17-25` — `PSRCluster` クラスと「water probability (±0.15 std error)」「Unknown until in-situ survey」のコメントが該当行に存在。引用一致。
- ✓ `environments/lunar_2d/relay_stations.py` — 100行のリレーステーション管理ロジックが存在。`propose_build`/`assist_build` アクションは `environments/lunar_2d/agent.py` の `VALID_ACTIONS` に含まれており、評価の「共同建設アクション」は正しい。

**スコア妥当性**: 月南極PSRという現実ミッション（LUPEX/Artemis/Chang'e 7/ILRS）から逆算した独創性は確かに高く、地形・電池・サンプル品質・フレア・シェルター・リレー建設の統合が一貫している。10/10は強気だが妥当範囲。

**見落とし・誇張**:
- やや誇張気味: 「商業・宇宙開発・社会科学への発展余地」は CLAUDE.md の Phase 2.0/2.5 ロードマップに記載されているとはいえ、現状実装は Phase 1.5 までであり、満点根拠としては将来計画にやや依存している。9.0-9.5 でも整合する。
- 見落とし: Nobile rim、scarp などの具体地形名が scenario.yaml に存在することは確認できる（評価では言及あり、引用 ✓）。

**検証結論**: ✓ 妥当（やや上振れ）。世界観の独自性・整合性は突出しており10.0は容認可。控えめに評価するなら9.5。

---

### C. 発展性（元: 9.5/10）

**根拠の検証**:
- ✓ README引用「tools -> scenarios -> environments -> core」一方向依存はREADMEのアーキテクチャ節に実在。
- ✓ `core/llm/base.py:9-38` — `LLMClient` 抽象基底＋`generate_async` ＋asyncio.Semaphore による max_concurrency 制御が実装。引用一致。
- ✓ `core/llm/anthropic_client.py`, `core/llm/ollama.py` — 両ファイル存在確認済み。`anthropic_client.py` は環境変数チェックとimport遅延、`ollama.py` はモデル別concurrency。
- ✓ `environments/lunar_2d/memory.py:33-68` — `MemoryConfig` データクラスに `stream_enabled`, `timeline_enabled`, `reflection_interval`, `weight_recency/importance/relevance` のフラグが実装。引用一致。
- ✓ `scenarios/runner.py:546-552` — `MemoryConfig.from_dict(emergence_cfg_for_memory.get("memory"))` と `PsychStateConfig.from_dict(...)` を YAML から動的構築。引用一致。
- ✓ `experiments/configs/` — `desktop_*`, `e1_full`〜`e24_internal_state`, `south_pole_*`, `laptop_quick`, `micro_quick` 等で計33個のYAML configが存在。「30+」は誤りではない。
- ✓ README引用「その後は、LUPEX 型の資源品質判定、通信・電力インフラ共有、Artemis/ILRS 的な制度差、複数 LLM の協調・競争、3D Lunarcraft 環境へ拡張する計画です。」は実在。
- ✓ `CLAUDE.md` — Phase 1→1.5→1.6→1.7→2.0→2.5 のロードマップが L15-20 で表形式で定義。
- ✓ `tests/` — 3,672行（評価では「約3,700行」、誤差約0.8%）。引用一致。

**スコア妥当性**: 一方向依存、複数provider対応、opt-inメモリ/PsychState、30+config、~3,700行テスト、明確な多段階ロードマップは確かに高水準。9.5/10は適切。

**見落とし・誇張**:
- 軽微な誤差: 「~3,700行」は実際3,672行で誤差わずか。問題なし。
- 見落とし: README「主要ドキュメント」に列挙される docs/research-report-hackathon-v2.md, docs/emergence-implementation-roadmap.md 等の構造化文書群への言及があるが評価では概要的言及のみ。

**検証結論**: ✓ 妥当。9.5/10 は推奨レンジ上限。

---

### D. 技術実装（元: 9.0/10）

**根拠の検証**:
- ✓ `core/simulation.py:69-110` — `_run_step_two_phase` で Phase 1=メッセージ決定、Phase 2=配信、Phase 3=行動決定、Phase 4=行動実行が明示的に実装。引用一致。
- ✓ `core/llm/parsing.py:30-36` — `ok`/`partial`/`fallback`/`empty_response` の4値仕様がdocstringに明記。引用一致。
- ✓ `core/llm/parsing.py:57-71` — `strip_thinking_tags` 関数が `<think>/<thinking>/<thought>` を `_THINKING_TAG_RE` と `_UNCLOSED_THINKING_RE` で除去。Qwen3のmax_tokens切れ対応が明記。引用一致。
- ✓ `core/llm/ollama.py:14-27` — `_MODEL_CONCURRENCY_DEFAULTS` で 70b/72b→2、14b/13b→4、7b/8b→6、fallback=8 のヒューリスティック。引用一致。
- ✓ `environments/lunar_2d/memory.py:105-156` — `MemoryStream.retrieve` で recency × importance × relevance の重み付きスコア＋Jaccardが実装。引用一致。
- ✓ `environments/lunar_2d/memory.py:165-212` — `build_reflection_prompt` で Q1（count）/Q2（outcome）/Q3（decision: continue/change）/Q4（next）の構造化質問が実装。引用一致。
- ✓ `environments/lunar_2d/psych_state.py:1-100` — AgentSociety (Piao et al. 2025) を冒頭で引用、Maslow型 needs・4軸emotion・per-step update_from_step が記述。引用一致。
- ✓ `scenarios/runner.py:615-792` — `on_step_end` 内で messages/reasoning/events/state/metrics ログをjsonlで出力、parse_status/parse_error/raw_response_excerpt まで保存。引用一致。
- ✓ `tools/emergence_metrics.py:64-217` — `individuality_index`、`reasoning_diversity`、`spatial_entropy`、`info_propagation_rate`、`role_specialization` が定義済み。引用一致。
- ✓ `tools/emergence_metrics.py:224-247` — `parse_health` 関数で `parse_fallback_rate > 0.05` のラン除外方針がdocstringに明示。引用一致。
- ✓ `environments/lunar_2d/agent.py` — 1,997行（wc -l 出力で一致）。引用完全一致。
- ✓ `anthropic_client.py:16-22` — APIキーが環境変数に存在するかboolean確認のみ、値は読まない実装。引用一致。

**スコア妥当性**: LLM出力parsing（4値status、thinking-tag防御、fallback rate監視）、3層メモリ（legacy/timeline/Smallville stream）、PsychState（AgentSociety引用）、2-phaseシミュレーション、seed管理、複数provider対応はいずれも丁寧な実装。1,997行agent.pyの責務分離余地という減点を明示しており9.0/10は適切。

**見落とし・誇張**:
- 見落とし: テスト総行数（3,672）に対しテストカバレッジ範囲（contracts/core/environments/tools）の指摘が薄い。
- 軽微な過大評価可能性なし: parsing.py の Cycle 11 ストーリー（C10で90分run失敗の経験を基にした実装）が「物語性」として評価で言及されるが、実装に裏付けがあるため誇張ではない。

**検証結論**: ✓ 妥当。9.0/10 は推奨レンジ内。控えめに評価するなら 9.0、強気なら 9.5。

---

## 総合所見

**引用検証結果**: 評価レポートに記載された **すべての主要citation（行番号付きを含む20件以上）が実コードと一致**。誤引用・捏造は発見されなかった。サンプルチェックした行番号 (`map.py:143-217`, `agent.py:41-58`, `orbital_data.py:99-146`, `memory.py:33-68`, `simulation.py:69-110`, `parsing.py:30-36`, `ollama.py:14-27`, `runner.py:546-552`, `emergence_metrics.py:64-217 / 224-247`) は全て正しい位置に該当コードが存在する。`agent.py` 行数1,997も完全一致。

**評価のバランス**: 創発設計（A）と技術実装（D）では減点要素（persona description のテキスト性、agent.pyの巨大化）を著者の主張に流されず独立に指摘しており、検証視点として健全。世界設定（B）の満点はやや甘いがレンジ内。

**推奨総合スコア**: 37.0/40 を維持して問題ない。控えめに評価する場合の下限は 36.0/40（B=9.5, D=9.0）。

**特筆事項**:
1. memo/ と docs/ に Cycle 10/11 等の失敗・改善履歴が記録されており、再現可能な研究プラットフォームとしての成熟度が高い。
2. parse_status の4値分類と parse_fallback_rate>0.05 のラン除外ルールは、LLM-based simulationの信頼性確保における優れた工夫。
3. PsychState（AgentSociety 2025引用）の opt-in 設計は最先端文献を反映している。

**改善提案（評価者へ）**: 元評価の改善点・提言セクションが既に十分網羅的（agent.py分割、persona description除去、再現用seed/run ID列挙、アトラクタ対策の定量効果）であり、追加すべき重大な指摘なし。
