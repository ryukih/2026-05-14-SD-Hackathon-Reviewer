# 評価レポート: singulabo-hackathon

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

パチスロホールを舞台に、「最も脳汁（ドーパミン）が出る瞬間」を多面的に検証するマルチエージェント・シミュレーション。8 種類の人物属性（中毒者、初心者、ベテラン等）× 6 機種カテゴリの 48 セル格子に合計 1,008 人のエージェントを配置し、1 日 8,000G の実践を一斉に走らせる構成。各エージェントは脳汁・ストレス・幸福度・絶望度という心理状態パラメータを持ち、機種側は払出分布・連チャン構造・イベント確率で表現される。Ollama 上の qwen3.5:35b-a3b モデルがシミュレーション中の「心の声」やペルソナごとの状況判断を LLM 生成する。180 秒の可視化動画では、属性別脳汁ランキング、個人 Top 3 のストーリー、機種特徴の抽出、5 つの状況シーンにおける 8 代表の判断比較などが順に提示される。テーマは「ストレス極限下での爆発の兆し」「中毒者の脳汁」という 3 段論法的仮説を、人 × 機種の交差分析を通じて検証することにある。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  4   |  3   |  -   |    3.5    |
| B. 世界設定  |  9   |  9   |  -   |    9.0    |
| C. 発展性    |  6   |  6   |  -   |    6.0    |
| D. 技術実装  |  7   |  7   |  -   |    7.0    |
| **合計**    |  26  |  25  |  -   |  **25.5** |

---

## 評価詳細

### A. 創発設計 — 最終: 3.5/10

#### 評価
世界ルールの設計精度は極めて高い（6 機種完全校正、8 属性 × 1000 人のパラメータ化、stress / dopamine_burst / happiness の式が SSoT 化されている）。しかし行動の自由度は低く、エージェント側の意思決定は完全に rule-based の感情更新式に支配されている。各エージェントは「assigned_machine」を 50 step 打ち続ける固定行動で、`maybe_switch` ロジックも v4.5 で削除されている。LLM は心の声生成（Phase 3）、感情ラベル生成（Phase 9）、判断観察（Phase 10）の **観察用 sidecar** に徹し、シミュレーションのループには一切フィードバックしない。1000 人が並列に動くが相互作用も存在しないため、設計上の創発ポテンシャルはほぼゼロ。

#### 根拠
- `pachinko_hall_sim/spike/spike_v44.py:268-370` — `play()` 関数は確率抽選 + chain 進行のみで、エージェント自身の判断は介在しない
- `pachinko_hall_sim/spike/spike_v44_demo.py:209-230` — `run_full_simulation` のメインループは「全 persona について順次 play」のみ、エージェント間相互作用なし
- `CLAUDE.md` — 「**通常時**: `assigned_machine` を 50 step 打ち続ける（spike_v43 の `maybe_switch` は **削除**）」（v4.5 機種選択ロジック）
- `CLAUDE.md` — 「3. **LLM は行動決定 + 自己申告のみ**（5 step ごとに間引く）」と謳うが、実装上は Phase 10 の状況判断は観察データであり、シミュレーション状態には反映されない
- `submission/draft.md:32-34` — 「数値観察 = numpy + 確率モデル」「結果後の語り = LLM」と明示的に LLM をシミュレーションの外部に置く設計

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
パチスロホールという閉じた社会的・心理的場を題材に、依存症心理・ドーパミン報酬学習・ストレス × 大逆転認知 という極めて具体的な研究的問いを立てている。8 属性（中毒者 / 主婦 / シニア / パチプロなど）は 169 ペルソナカードを DeepResearch で収集して階層サンプリングしたもので、6 機種カテゴリは 4号機・5号機・初代GOD まで実機経験ベースで完全校正されている。シナリオの解像度が「世代差・依存症臨床・大逆転認知の神経科学」レベルで深く、社会科学的にも医療人類学的にも価値のある観察対象。標準的なデモシナリオとは一線を画す独自性。

#### 根拠
- `README.md:6-14` — 「8 種類の人物属性 × 6 機種カテゴリの 48 セルに 1,008 人を配置し、1 日 8,000G のパチスロ実践を一斉シミュレートして、**最も脳汁（ドーパミン）が出る瞬間**を多面的に検証する」
- `pachinko_hall_sim/spike/spike_v44.py:69-211` — 6 機種完全 SSoT（NORMAL_BONUS / LATE_4G_MASS / BURST_AT_4G / SELF_TRIGGER_5G / LOOP_CHAIN_5G / GOD_ORIGIN）、各機種に `chain_start_impact` / `event_probs` / `event_impacts` を持たせた抽象モデル
- `pachinko_hall_sim/spike/spike_v44_personas.py:48-137` — 8 属性の母集団パラメータ（sens_mean, base_stress_mean, personal_threshold, stress_floor, hammari_tau, extras）が DeepResearch 経由で根拠付け
- `persona-cards/all-cards-merged.yaml` — 226KB の DeepResearch 由来ペルソナデータ
- `docs/multi-path-addiction-model.md` — 14 経路中毒モデルの原典資料あり

---

### C. 発展性 — 最終: 6.0/10

#### コード拡張性
モジュール分離は中程度。`spike_v44.py`（物理エンジン）/ `spike_v44_personas.py`（属性 + サンプリング）/ `spike_v44_demo.py`（動画生成）/ `spike_llm_voice.py`（LLM）が責務分割されており、新しい機種や属性は dataclass 追加で拡張可能。一方、`spike_v44_demo.py` は 2,273 行のモノリスで、Phase 1-10 が全て同一ファイル内にハードコードされている。本来想定された `pachinko_hall_sim/src/` 配下のプロダクション実装はほぼ空（多くが 100 バイト未満のスタブのみ）で、本体は spike フォルダに留まっている。`configs/machines.yaml` / `configs/run_mvp.yaml` も用意されているが、現行の動画生成パイプライン（spike_v44_demo.py）はこれらを参照していない。

#### 将来展望
README と CLAUDE.md（v4.6 進捗ログ）に、5 つの新規概念（streak_bonus / chain_anxiety / disappointment_stress / miren_uchi_policy / afterglow）の差し替え予定、ablation 実験（stress 係数 OFF モデル）、3 群比較実験（high-stress / low-stress / baseline）など具体的な将来計画が記述されている。submission/draft.md でも限界の明示と次の検証軸が言及される。研究的方向性は明確だが、コードベースをそのままスケールさせる構造的整備は不足。

#### 根拠
- `pachinko_hall_sim/src/` — 多くのファイルが 100 バイト未満のスタブ（`emotion.py` 90B, `llm_client.py` 111B, `machine.py` 81B）
- `pachinko_hall_sim/spike/spike_v44_demo.py` — 2,273 行の動画生成モノリス
- `CLAUDE.md` の v4.5 セクション — 「5 つの新規概念実装名: streak_bonus / chain_anxiety / disappointment_stress / miren_uchi_policy / afterglow」「ablation（stress 係数 OFF モデル 1 本を別途回す）」
- `CLAUDE.md` の Day10 マイルストーン — 「パッケージング: README、実行コマンド、出力例、PDF 化、GitHub 整備」
- `submission/draft.md` — 「再現は `pachinko_hall_sim/spike/spike_v44_demo.py` を実行する」（本番が spike フォルダにある）

---

### D. 技術実装 — 最終: 7.0/10

#### LLM利用
プロンプト設計は丁寧で、`spike_persona_probe.py` では `ollama format=json` を指定し JSON schema を強制、JSON decode 失敗時の retry (max_retries=2) を実装。temperature 0.85-0.95 / top_p 0.92 で多様性を確保しつつ、`voice_history` を直近 6 件渡して被り回避制約を入れている。`spike_llm_voice.py` では life_themes ヒント、禁止語（「また」「もう」「やっぱり」）、属性別の性別分布・年齢サンプリングなど多層の制御がある。一方、LLM は完全に observational sidecar であり、シミュレーション状態には反映されない（Phase 10 の状況判断結果は出力 JSON に保存されるだけ）。

#### メモリ設計
エージェント単位の持続的メモリは存在しない。`PersonaState`（cash / chain_active / miss_streak / win_streak）と `arousal_by_pid` / `stress_by_pid` の dict で動的状態を保持するが、これは個別の物理状態であり「記憶」ではない。LLM 呼び出しでは `voice_history` の直近 6 件を被り回避用に渡すのみで、エージェント個別の履歴・経験を文脈化する設計はない。CLAUDE.md には豊富な session_context / persona trait の設計があるが、それは特性であってメモリではない。

#### コード品質・シミュレーション正確性
コード品質は良好。dataclass による型付け、責務分割、日本語 docstring が充実。`spike_v44.py:32-66` の `MachineType.__post_init__` で lognormal の mu/sigma を mean/std から自動逆算しており、SSoT 違反を防止。`assign_machines` は人数不一致時に fail-fast で raise する。シミュレーション正確性は高く、`run_calibration` で 1000 人 × 50 step を回して target_return を実測し校正している（`spike_v44.py:373-465`）。RNG は `np.random.default_rng(seed)` で seed 固定。一方、`spike_v44_demo.py` は 2,273 行の単一ファイルで関心分離が甘く、Phase 1-10 のレンダリング関数が大量に詰まっている。

#### 根拠
- `pachinko_hall_sim/spike/spike_persona_probe.py:183-212` — `call_llm` の retry 機構 + `format: "json"` 指定
- `pachinko_hall_sim/spike/spike_llm_voice.py:38-73` — few-shot プロンプト + voice_history + temperature 0.95 / top_p 0.92
- `pachinko_hall_sim/spike/spike_v44.py:56-66` — `__post_init__` で lognormal パラメータ自動同期、`event_impacts["chain_start"]` の二重 SSoT 解消
- `pachinko_hall_sim/spike/spike_v44.py:375-415` — `run_calibration` で 1000 × 50 step の実測値検証
- `pachinko_hall_sim/spike/spike_v44.py:237-240` — `assign_machines` で 125 人/属性の前提チェック（fail-fast）
- `pachinko_hall_sim/spike/spike_v44_demo.py` — 2,273 行モノリス、Phase ごとの描画関数が混在
- メモリの不在: `spike_v44_demo.py:203-208` — エージェント状態は dict と PersonaState のみ、履歴の保持なし

---

## 総評

### 優れている点
- **世界設定の解像度と独自性**: 6 機種完全校正 + 8 属性 × 1000 人 + 169 ペルソナカード由来の設定は、ハッカソン提出物としては突出して詳細。依存症心理・ドーパミン報酬学習という社会的に意味のある題材を、抽象モデルとして真摯に扱っている。
- **校正の厳密さ**: `run_calibration` で target_return を実測検証し、payout_cap や lognormal mu/sigma の自動同期など、物理エンジンとしての堅牢性が高い。`MachineType.__post_init__` の SSoT 設計は秀逸。
- **仮説検証の科学的態度**: 仮説提示（中毒者 × 爆発の兆し）→ 自己成就回避のための反論軸（非中毒者ピーク、低ストレス時ピーク、機種独立効果）→ 結論（「人と機種の合作」）という構造が、submission/draft.md と動画 Phase 6-8 で一貫している。
- **LLM プロンプト品質**: format=json + retry + voice_history + life_themes + 禁止語など、LLM 出力の安定化と多様性確保に多層の工夫がある。
- **提出物の完成度**: 180 秒の動画 + 解説 PDF + ソースコードが揃い、Phase 9（Persona Reaction Probe）/ Phase 10（状況判断観察）まで含めた構成は提出物として極めて完成度が高い。

### 改善点・提言
- **エージェントの自律性が薄い**: 1000 人いるが全員 rule-based で、LLM はシミュレーションのループに介入しない。「LLM ベースのマルチエージェントシミュレーション」の文脈では、LLM 駆動の意思決定が状態変化に寄与する設計（例: Phase 10 の状況判断結果を次 step の `assigned_machine` 再選択にフィードバック）があれば創発設計の評価は大きく上がる。
- **エージェント間相互作用の欠如**: 1000 人が並列実行されるだけで、隣接台のヒットを観察して感情が変化する、ホール全体のムードが個人の判断に影響する、などのソーシャル動態が一切ない。CLAUDE.md には `social_commitment_density` などの session context があるが、エージェント間の伝播は実装されていない。
- **メモリ設計の不在**: エージェントが過去の体験を「記憶」として保持し、それが行動に効くという構造がない。例えば「中毒者は前回の GOD 当選の余韻が持続している」のような afterglow（CLAUDE.md には記載あり）が実装されれば、メモリ評価が上がる。
- **src/ 配下のプロダクション化**: `src/` のスタブを実装に置き換え、`spike_v44_demo.py` 2,273 行を Phase 別モジュールに分解すれば、コード拡張性は大幅に向上する。
- **configs の活用不徹底**: `configs/machines.yaml` / `configs/run_mvp.yaml` があるのに、本番の `spike_v44_demo.py` はベタ書き値を使っている。外部化を進めれば実験設計のスイープが楽になる。

### 一言コメント
題材の深さと物理エンジン・LLM 連携の完成度は屈指だが、LLM がシミュレーションの外部観察者に留まり、エージェントの自律的意思決定・相互作用・記憶という「マルチエージェント創発」の本質的要素が薄い点が惜しい。仮説検証実験としての設計は卓越している一方で、「LLM ベースの自律マルチエージェント」というハッカソンテーマの軸からはやや距離がある。
