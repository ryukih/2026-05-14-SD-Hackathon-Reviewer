# 評価検証レポート: singulabo-hackathon

**検証日**: 2026-05-11
**対象**: `singulabo-hackathon_eval.md`
**元評価合計**: 25.5/40

---

## 検証サマリー

| カテゴリ | 元スコア | 検証結論 | 推奨帯 |
|---|---|---|---|
| A. 創発設計 | 3.5/10 | 妥当 | 3-4 |
| B. 世界設定 | 9.0/10 | 妥当（やや高めだが許容範囲） | 8-9 |
| C. 発展性 | 6.0/10 | 妥当 | 5-6 |
| D. 技術実装 | 7.0/10 | 妥当 | 7-8 |

**総合判定**: 元評価 25.5/40 はバランスがよく、引用された file:line もほぼ全てが正確に検証できた。スコア帯はそれぞれ推奨帯の範囲内であり、結論をそのまま支持する。

---

## カテゴリ別検証

### A. 創発設計（元: 3.5/10）

**根拠の検証**:

- ✓ `spike_v44.py:268-370` の `play()` — line 273 から `play()` が始まり、確率抽選 + chain 進行のみで、エージェントの判断は介在しない。引用通り。
- ✓ `spike_v44_demo.py:209-230` の `run_full_simulation` メインループ — line 209 から始まる `run_full_simulation` は「全 persona について順次 play」のみで、エージェント間相互作用は確認できなかった。引用通り。
- ✓ CLAUDE.md `maybe_switch` 削除 — line 1350 に「`assigned_machine` を 50 step 打ち続ける（spike_v43 の `maybe_switch` は **削除**）」と明記、引用通り。
- ✓ CLAUDE.md「LLM は行動決定 + 自己申告のみ（5 step ごとに間引く）」 — line 86 に該当記述あり。ただし「実装上は Phase 10 の状況判断は観察データであり、シミュレーション状態には反映されない」という指摘は draft.md と動画構成から裏付けられる。
- ✓ `submission/draft.md:32-34` — 「数値観察 = numpy + 確率モデル」「結果後の語り = LLM」「結果前の判断 = LLM」のテーブルが存在し、LLM をシミュレーション外部に置く設計が明示されている。

**スコア妥当性**:
ルール精度は極めて高い（6 機種 SSoT・8 属性パラメータ化・lognormal 自動逆算・払出 cap・event_probs/event_impacts による中間イベント実装）一方、エージェント自由度は事実上ゼロ（rule-based 固定 + LLM は sidecar）という構造は引用通り。1000 体並列でも相互作用がないことも `run_full_simulation` を実際に読んで確認した。ルーブリック「LLM as formatter（1-2）」よりはルールが精密なため 3-4 帯が妥当。3.5 は適切。

**見落とし・誇張**:
- 「LLM がシミュレーションのループには一切フィードバックしない」はやや断定的だが、コード上 LLM 出力はログ・JSON への保存のみで、`PersonaState` や物理乱数系列を変えない事実は確認できた。誇張ではない。
- Phase 10 の状況判断結果は `spike_llm_situations.py` で生成されるが、確かに main simulation loop には戻らない（独立スクリプト）。引用は正確。
- 評価は明示していないが、`session_context`（social_commitment_density など、CLAUDE.md line 331、676）も設計はあるが simulation loop には適用されていない点を補足できた可能性あり。

**検証結論**: 3.5/10 を支持。

---

### B. 世界設定（元: 9.0/10）

**根拠の検証**:

- ✓ `README.md:6-14` — 「8 種類の人物属性 × 6 機種カテゴリの 48 セルに 1,008 人を配置し、1 日 8,000G のパチスロ実践を一斉シミュレートして、最も脳汁（ドーパミン）が出る瞬間を多面的に検証する」は README に存在。引用通り。
- ✓ `spike_v44.py:69-211` — 6 機種 SSoT 定義（NORMAL_BONUS / LATE_4G_MASS / BURST_AT_4G / SELF_TRIGGER_5G / LOOP_CHAIN_5G / GOD_ORIGIN）が line 78 から 211 にかけて完全に存在。`chain_start_impact` / `event_probs` / `event_impacts` も実装あり。引用通り。
- ✓ `spike_v44_personas.py:48-137` — 8 属性母集団パラメータ（sens_mean, base_stress_mean, threshold_median, cash_median, stress_floor_mean, hammari_tau, extras）が line 49 から実装。引用通り。
- ✓ `persona-cards/all-cards-merged.yaml` — 226200 バイト（評価は「226KB」と表記、正確）。
- ✓ `docs/multi-path-addiction-model.md` — 18,480 バイトのドキュメントが存在。

**スコア妥当性**:
題材の独自性（パチスロ × 依存症心理 × ドーパミン報酬学習）、ペルソナの解像度（DeepResearch 由来、R1-women / R2-elderly / R3-extremes 由来の 169 ペルソナ）、機種の校正精度（4号機・5号機・初代GOD のスペック）は突出して高い。原典資料（multi-path-addiction-model.md）も整備されており、9 は妥当。10 にしないのは LLM 駆動マルチエージェントというハッカソン主題からはやや外れた「物理校正実験」寄りの色合いに留まる点で、評価のバランスは適切。

**見落とし・誇張**:
- 「169 ペルソナカードを DeepResearch で収集」は persona-cards/ ディレクトリ内の R1-R3 yaml 群（R1-women、R2-elderly、R3-extremes）から実在を確認。誇張ではない。
- 「6 機種完全校正」は `run_calibration` と `summarize_calibration` で実測 vs ターゲット出力する仕組みが実装されていることから妥当。
- 「医療人類学的にも価値のある」という形容は若干修辞的だが、`docs/multi-path-addiction-model.md` が 14 経路中毒モデルとして実在することから過剰評価ではない。

**検証結論**: 9.0/10 を支持。

---

### C. 発展性（元: 6.0/10）

**根拠の検証**:

- ✓ `pachinko_hall_sim/src/` 配下のスタブ — `emotion.py` 90B、`llm_client.py` 111B、`machine.py` 81B、その他多くが 100 バイト程度。引用通り。
- ✓ `spike_v44_demo.py` 2,273 行 — `wc -l` で確認、正確に 2273 行。
- ✓ CLAUDE.md 5 つの新規概念 — line 1285 に「5 つの新規概念実装名: `streak_bonus / chain_anxiety / disappointment_stress / miren_uchi_policy / afterglow`」明記。引用通り。
- ✓ ablation 言及 — line 1234-1243 に「ablation（自己成就回避） / stress 係数 OFF モデル 1 本を別途回す」あり、引用通り。
- ✓ 3 群比較実験 — line 976 に「比較実験（high-stress / low-stress / baseline）」あり、引用通り。
- ✓ Day10 マイルストーン — line 978 に「パッケージング: README、実行コマンド、出力例、動画ファイル名規則、PDF 化、GitHub 整備」あり、引用通り。
- ✓ `submission/draft.md` line 116 — 「再現は `pachinko_hall_sim/spike/spike_v44_demo.py` を実行する」、引用通り。
- ✓ `configs/machines.yaml` / `configs/run_mvp.yaml` の存在は確認、サイズも妥当（861B / 728B）。本番が spike フォルダにベタ書きという指摘もコード上整合。

**スコア妥当性**:
モジュール分離は中程度（spike_v44.py / personas / demo / llm_voice）で責務分割は確認できる一方、デモ本体が 2273 行モノリスで src/ が空という構造的弱さも事実。一方、CLAUDE.md と draft.md には豊富な将来計画あり。README 自体は簡潔だが、CLAUDE.md / docs / submission/draft.md にビジョンの蓄積は十分。6 は妥当な中庸スコア。

**見落とし・誇張**:
- 「README と CLAUDE.md（v4.6 進捗ログ）」と書かれているが、CLAUDE.md は v4.5 中心で v4.6 表記は厳密には主流ではない。軽微なラベル違い。
- README には将来展望そのものは少なく、CLAUDE.md / draft.md が将来計画の主担当である点はやや言及不足だが、引用箇所自体は正確。
- 「configs/ の活用不徹底」は評価の指摘通り。`spike_v44.py` の MACHINES 定義はベタ書きで、`configs/machines.yaml` を読まないという観察が正しい。

**検証結論**: 6.0/10 を支持。

---

### D. 技術実装（元: 7.0/10）

**根拠の検証**:

- ✓ `spike_persona_probe.py:183-212` — line 183 から `call_llm` 関数があり、`max_retries=2`、`format: "json"`、retry ループが実装されている。引用通り。
- ✓ `spike_llm_voice.py:38-73` — few-shot プロンプト + `voice_history`（line 17, 21-22 で「直近 6 件」も確認） + `temperature 0.95 / top_p 0.92` を line 71-72 で確認。引用通り。
- ✓ `spike_v44.py:56-66` — `__post_init__` で lognormal mu/sigma 自動逆算と `event_impacts["chain_start"]` 同期を実装。引用通り（厳密には line 55-66 だが許容範囲）。
- ✓ `spike_v44.py:373-415`（評価は 375-415 と表記）— `run_calibration` で 1000 人 × 50 step の実測値検証。引用通り。
- ✓ `spike_v44.py:237-240` — `assign_machines` で 125 人/属性の前提チェック（`raise ValueError`）。引用通り（実際は line 233-238 付近の `if len(members) \!= expected_per_attr: raise ValueError`）。
- ✓ 2,273 行モノリス — 確認済み。
- ✓ 禁止語「また」「もう」「やっぱり」 — `spike_llm_voice.py:27` に「特に『また』『もう』『やっぱり』など同じ書き出しを避ける」あり、引用通り。
- ✓ temperature 0.85 / top_p 0.92（persona_probe）と 0.95 / 0.92（voice）— コードで確認、評価の表記「temperature 0.85-0.95 / top_p 0.92」は正確。
- ✓ RNG seed 固定 — `np.random.default_rng(seed)` の利用は `run_calibration` 内（line 384）と `run_full_simulation` 内（demo.py 210）で実装、引用通り。

**スコア妥当性**:
LLM 3 点（プロンプト品質高い: format=json + retry + voice_history + life_themes + 禁止語の多層構造） + Memory 1-2 点（履歴は voice_history 6 件のみで個別エージェント記憶なし） + CodeQ 1.5-2 点（dataclass、SSoT 設計、ただしモノリス課題） + SimCorrect 2 点（校正実測あり、seed 固定）の合計でおよそ 7-8 帯が妥当。7.0 は適切で、メモリ設計の薄さを差し引いた評価として妥当。

**見落とし・誇張**:
- 「メモリの不在」の根拠として「`spike_v44_demo.py:203-208`」を引いているが、実際の `run_full_simulation` 内の状態保持 dict（arousal_by_pid / stress_by_pid）は line 211-212 付近。行番号がわずかにずれているが趣旨は正確（PersonaState と dict のみで履歴なし）。
- 「日本語 docstring が充実」は事実だが、全関数に網羅されているわけではなく、主要部のみ。やや好意的な記述。
- `spike_v44_demo.py` のサイズと内容を「Phase 1-10 が全て同一ファイル内にハードコードされている」と表現しているが、Phase ごとの描画関数が混在するモノリスである点は概ね正確。

**検証結論**: 7.0/10 を支持。

---

## 総合所見

元評価 25.5/40 は引用検証の精度が高く、各カテゴリのスコアもルーブリックに照らして妥当である。特に A（創発設計 3.5）は「LLM が観察 sidecar に徹し、ループにフィードバックしない」という核心的構造を正確に捉えており、B（世界設定 9.0）は題材の独自性と物理校正の厳密さを適切に評価している。C（発展性 6.0）も src/ スタブと spike/ モノリスという構造的弱さを正確に指摘した上で、CLAUDE.md / draft.md の豊富な将来計画を加点している。D（技術実装 7.0）も LLM プロンプト品質・SSoT 設計・校正実測を肯定しつつ、メモリ設計の不在を妥当に減点している。

軽微な瑕疵としては、(1) 一部の行番号引用が 2-5 行ずれる箇所（spike_v44.py:237-240、:56-66、:375-415、demo.py:203-208）があるが、引用された関数や記述の存在自体は確実に確認できる範囲。(2) 「v4.6 進捗ログ」というラベルは CLAUDE.md 内に明示的にはなく、v4.5 が主であるという軽微なずれがあるが、評価結論への影響はない。

総評の「優れている点」「改善点・提言」「一言コメント」の各論点は、いずれも実装と整合的である。特に「LLM がシミュレーションの外部観察者に留まり、エージェントの自律的意思決定・相互作用・記憶という『マルチエージェント創発』の本質的要素が薄い」という結語は、本作品の本質を端的に表現している。

総じて、元評価は引用検証に堪える誠実な分析であり、スコア合計 25.5/40 およびカテゴリ別配分はそのまま採用してよい。Run1 / Run2 のスコア揺れも 1 点以内で収まっており、評価の安定性も高い。

---

**100 語サマリー**:
The evaluation of singulabo-hackathon at 25.5/40 is well-grounded. Every cited file:line was verified: the 6-machine SSoT in spike_v44.py:69-211, 8-attribute personas in spike_v44_personas.py:48-137, LLM retry logic in spike_persona_probe.py:183-212, calibration loop, and CLAUDE.md references all check out. The core thesis is accurate: the physics engine is precise but the LLM is an observational sidecar with no feedback into simulation state, and no agent-to-agent interaction exists. Minor line-number drifts (2-5 lines) and a slightly hyperbolic "v4.6" label do not affect the conclusion. Category scores 3.5/9.0/6.0/7.0 are all within recommended bands.
