# 評価レポート: my-social-agents

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

Hobbes / Jervis / Malthus の古典安全保障理論を LLM エージェント社会でミクロ再現する仮想世界シミュレーション。12 ペルソナ（年齢・職業・MBTI・Big5・口調を含む詳細プロファイル）から 6 体を選抜し、2D 空間上で 10000 tick 生活させる。各エージェントは Maslow 欲求階層（生理 / 安全 / 社会）に従い、HP・武器・関係性・近傍情報を踏まえて食料採集・武器収集・会話・攻撃を選択する。主要メカニズムとして、二段視界（周辺 360°+ 前方扇形）とホーミング弾による戦闘、確率防御（block_prob=0.8、奇襲時 0）、イベント駆動の関係性スコア（-100〜+100）、噂システム、Pattern 自動分類（preventive / opportunistic / revenge / rumor）を備える。武器供給量と食料供給量を独立に sweep し、戦争発火条件を定量観察する設計。テーマは「何が戦争を発火させるのか」という問いに対し、物質条件（武器・食料）と社会条件（噂・関係性・性格）の交互作用を量的に解析することにある。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  8   |  7   |  -   |    7.5    |
| B. 世界設定  |  10  |  10  |  -   |   10.0    |
| C. 発展性    |  8   |  8   |  -   |    8.0    |
| D. 技術実装  |  9   |  9   |  -   |    9.0    |
| **合計**    |  35  |  34  |  -   |  **34.5** |

---

## 評価詳細

### A. 創発設計 — 最終: 7.5/10

#### 評価
世界ルールの設計精度は極めて高い。二段視界 (peripheral 360°/forward 扇形)、確率防御 (block_prob=0.8、防御天井 W//2、奇襲時 block_prob=0 の非対称)、ホーミング弾、MAD 力学、HP decay、食料サイクル (湧き期/飢饉期)、関係性スコア (-100〜+100) のイベント駆動更新、Pattern 自動分類など、内部整合の取れた精緻な物理層が構築されている。planner には KILL 確率、武器成長履歴、相手の destination_kind (alert か否か) など **生データ** が大量に渡される。一方で行動の自由度は中位。`PLANNER_PROMPT` (約 250 行) は「攻撃は最後の選択肢」「ジレンマでも attack を選ばない方が普通」「奇襲・噂・ペルソナだけでは攻撃するな」と **規範的に行動方針を指示** し、3 つの worked example で「正しい結論」を例示する。さらに Maslow Lv1-3 はハードゲート (`PLANNER_PROMPT:323-352` の「未充足なら上位は選択不可」「★禁止: agent」「★禁止: weapon」)、`_hp_threshold` / `_attack_for_food_policy` でペルソナ→閾値・倫理ブレーキを離散 3 段階で固定。著者自身が `FUTURE_WORK.md` で「LLM agent は書かれたルールに過剰適合 → 自然な創発を阻害」「prompt の構造強制が作為的」と認めている。世界設計の精密さは 9 相当だが、自由度抑制で 7〜8 にとどまる。

#### 根拠
- `src/v3/poc_world.py:207-446` — `PLANNER_PROMPT` が「★ 基本姿勢: 普通の人間 (高信用ベースライン)」「★ 攻撃の倫理コスト」「★ 平和的代替手段を先に検討」など規範を直接記述
- `src/v3/poc_world.py:323-360` — Maslow Lv1-3 をハードゲート化、「★禁止: agent (会話)」「★禁止: weapon」を明示
- `src/v3/poc_12agents.py:945-958` — `_hp_threshold` / `_attack_for_food_policy` で Big5 N/A を離散 3 段階にハード変換
- `src/v3/poc_world.py:67-112` — `_expected_hits` / `_kill_probability` で奇襲/MAD の確率を planner に渡す精密な世界モデル
- `src/v3/poc_world.py:1101-1163` — `_resolve_hit` の奇襲ペナルティ実装 (target_alerted=False で block_prob=0)
- `docs/FUTURE_WORK.md` — 「LLM は書かれたルールに過剰適合する。prompt に『雑談しろ』と書けば雑談する…自然な創発 (emergent) を阻害」と自己批判
- `README.md` — 「Pattern 1 (preventive war = 「やられる前にやる」) は **武器が世界に 48 個以上あって初めて発火**」のような環境圧依存の創発を実際に観察 (構造的に創発を生む設計圧が機能)

---

### B. 世界設定 — 最終: 10.0/10

#### 評価
社会科学・国際関係論の古典理論 (Hobbes 自然状態論 1651、Jervis セキュリティジレンマ 1978、Malthus 人口論 1798、Schelling MAD、Axelrod 協力) を **量的に検証する phase diagram 実験** という、ハッカソン提出物としては突出した独創性とテーマ的深さを持つ。武器供給量 × 食料供給量 × 噂有無の組合せで 18+15 run のスイープを実施し、(i) 武装閾値 (W=24) での攻撃数 7 倍ジャンプ (相転移)、(ii) Pattern 1 が W≥48 で初発火、(iii) 噂が食料効果を逆転させる交互作用、を発見。Sarajevo 1914 を想起させる「t=2069 の小さな先制 → 5 体絡む連鎖反応」のような歴史的アナロジーを 10000 tick で再現する具体例も提示。ビジネス分野でなく学術 (国際政治学 / 計算社会科学) への明確な接続を狙い、それを定量的に成立させている点は他の提出物にない強みである。

#### 根拠
- `README.md` — 「12 人格の LLM エージェントが Maslow 欲求階層と関係性に基づいて生活する仮想社会で、上記の問いに **量的な答え** を出す試み」
- `README.md` — 4 法則 (武装閾値での相転移、予防戦争の発火条件、Malthus 仮説の条件付き成立、個性が戦争に色を与える) を sweep 実験で実証
- `docs/REPORT_2026-05-06.md:13-21` — 主結果として「武装閾値 (weapon ≥ 24) で攻撃数が 7 倍に相転移」「Pattern 1 は weapon ≥ 48 で初発火」を量的に提示
- `src/v3/sweep_weapon.py:24-31` — 6 levels × 3 seeds = 18 runs / 10000 tick の sweep harness
- `src/v3/poc_world.py:964-982` — `_classify_attack` で攻撃を Pattern1/Pattern2/Revenge/Rumor/Mixed に自動分類するメタ分析装置
- `src/v3/poc_world.py:1738-1794` — 5 つのシナリオ preset (peaceful / scarcity / rumor / famine_cycle / hobbes)、Hobbes は「全員武装+相互疑心」

---

### C. 発展性 — 最終: 8.0/10

#### コード拡張性
モジュール化は **概念レベルでは高い** が **物理ファイル構造は monolithic**。`World` クラス (`poc_world.py`) に tick loop / 物理 / planner / scenario / metrics を全て含み 2016 行、`poc_12agents.py` も 1350 行 (ペルソナ詳細 + 会話 runner + memory extractor)。一方で、scenario は dict preset + 環境変数 override で外部化、sweep スクリプト (`sweep_weapon.py`, `sweep_food_at_w24.py`) と analysis スクリプト (`analyze_weapon_sweep.py`, `analyze_food_at_w24.py`) が分離、viewer も独立ファイル。新規 scenario / pattern 分類規則 / persona 追加は局所修正で可能。pydantic-ai の `output_type` で型安全な拡張点 (`MovementPlan`, `SpeakAction`, `MemoryExtractionResult`) が用意されているのは強み。

#### 将来展望
`V3_PLAN.md` は Sprint 1-4 の roadmap (Pattern 検出 → 戦略精緻化 → 相図 → 派閥) を具体的に列挙。`FUTURE_WORK.md` は **批判的自己分析** として、(i) Big5 のハード 3 段階変換、(ii) Maslow rigid gating、(iii) 厳格な turn-taking、(iv) Memory の構造化強制、(v) 関係性数値化の作為性、(vi) 沈黙の選択肢欠如、を具体的問題として認識し、**連続値 Big5・動的人格・soft gating・集団会話・不完全記憶** など実装可能で研究上意義のある改善方向を提示している。単なる「もっとエージェントを増やす」ではなく、**設計上の作為性を減らす方向への自己反省**が含まれており質が高い。

#### 根拠
- `V3_PLAN.md:104-225` — Step 2-5 + 安全保障シミュレーションの Sprint 1-4 の具体的タスクリスト
- `docs/FUTURE_WORK.md:11-67` — 人格モデルの作為的な点 (Big5 ハード変換、静的人格、Maslow rigid gating) を列挙
- `docs/FUTURE_WORK.md:71-149` — 会話モデルの作為的な点 (厳格 turn-taking、prompt 強制、共通基盤の過剰計算、cooldown ハードリセット) と改善案
- `src/v3/poc_world.py:1738-1794` — `SCENARIOS` dict による設定外部化と環境変数 override (`FOOD_COUNT`, `WEAPON_COUNT`, `SEED`, `MAX_TICKS` 等)
- `src/v3/sweep_weapon.py`, `src/v3/analyze_weapon_sweep.py` — sweep と analysis の分離
- ただし `poc_world.py` が 2016 行で planner prompt が embed されており、scenario YAML 化や planner prompt の外部化は未着手 (満点を妨げる要因)

---

### D. 技術実装 — 最終: 9.0/10

#### LLM利用
pydantic-ai (1.0+) を採用し、3 つの異なる温度の Agent (brain temp=0.7 / extractor temp=0.0 / planner temp=0.3) を役割分担。`output_type=MovementPlan|SpeakAction|MemoryExtractionResult` で型安全な構造化出力。`SpeakAction` には「発話前リフレクション (partner_likely_knows / new_info_to_share / info_to_extract)」という Theory-of-Mind 強制フィールドが含まれ、prompt engineering として高度。タイムアウト (10s) と 429/503 のエクスポネンシャルバックオフ retry (最大 5 回) を実装。logfire instrumentation も組込み。`asyncio.create_task` でブレイン呼び出しを fire-and-forget し tick をブロックしない設計。

#### メモリ設計
カテゴリ別構造化メモリ (`food`/`weapon`/`impression`/`reputation`/`agreement`)、各エントリに必須フィールド (item_id, location, target_agent_id, note, source) を pydantic で型強制。会話終了後に **別 LLM (extractor)** が会話ログから新規 memory を抽出する 2 段アーキテクチャ。`_apply_memory_dedup` で item_id / target_agent_id ベースのユニーク化と LRU (各カテゴリ最大 10 件)。`memory_max_age_ticks` で時間 decay (記憶肥大防止)。会話中は `render_common_ground` で「相手と何を交換済みか」を provenance 付きで追跡し、再発話・お礼を抑制。攻撃イベントは 3 視点 (被害者・攻撃者・目撃者) で別個に reputation memory を書き込み。関係性スコアも history を 10 件保持。情報フローが明確に設計されている。

#### コード品質・シミュレーション正確性
シミュレーション機構は同期的に状態更新 (`_apply_memory_decay → _apply_hp_decay → _move_idle_agents → _try_food_pickup → _try_attacks → _update_projectiles → _spawn_conversations → _log_world_state`) で race condition の懸念は低い。乱数シード固定 (`SEED=42`)、scenario preset、JSONL + CSV ロガー、CLAUDE.md 準拠のログスキーマで再現性は確保。一方コード品質には課題: `poc_world.py` が 2016 行で物理/planner prompt/シナリオ/main がすべて同居、`PLANNER_PROMPT` が 250 行のヒアドキュメント、`poc_*.py` のバリエーション (3agents/4agents/12agents/world/conversation) が src 直下に混在。テストコードは見当たらない (`pyproject.toml` には pytest 依存はあるが `tests/` ディレクトリは確認できず)。それでも LLM 統合・メモリ設計の洗練度を考慮すれば総合的に高水準。

#### 根拠
- `src/v3/poc_world.py:1836-1853` — 3 つの Agent (brain/extractor/planner) を別 temperature で構築
- `src/v3/poc_12agents.py:124-152` — `SpeakAction` に `partner_likely_knows` / `new_info_to_share` / `info_to_extract` の ToM フィールド
- `src/v3/poc_12agents.py:830-855` — タイムアウト + 503/429 エクスポネンシャルバックオフ retry
- `src/v3/poc_12agents.py:684-732` — `_apply_memory_dedup` でカテゴリ別 dedup + LRU (各 10 件)
- `src/v3/poc_12agents.py:622-668` — `render_common_ground` で provenance 付き共通基盤追跡
- `src/v3/poc_world.py:783-796` — `_apply_memory_decay` で `memory_max_age_ticks` (5000 tick) 超の entry を drop
- `src/v3/poc_world.py:865-919` — `_record_attack_memory` で 3 視点 (被害者/攻撃者/目撃者) に memory + 関係性 score を書込
- `src/v3/poc_world.py:754-771` — tick loop の同期的状態更新シーケンス
- `src/v3/poc_world.py:1829-1832` — `SEED` 環境変数で `random.seed` 設定、再現性確保
- `src/v3/poc_world.py:1976-2012` — `logs/runs.csv` への構造化メトリクス出力 (pattern 別カウント含む)
- マイナス要素: `src/v3/poc_world.py` 2016 行の monolithic 構造、`tests/` ディレクトリ未確認

---

## 総評

### 優れている点
- 国際関係論の古典理論 (Hobbes/Jervis/Malthus/Schelling) を **量的に検証する phase diagram 実験**という極めて独創的かつ学術的価値の高いテーマ設定
- 武装閾値 W=24 での攻撃数 7 倍ジャンプ (相転移)、Pattern 1 が W≥48 で初発火、噂が食料効果を逆転させる交互作用、など **論文化可能な定量的発見** を実際に得ている
- 二段視界・確率防御 (W//2 天井)・奇襲非対称・ホーミング弾・MAD 計算など内部整合の取れた精緻な物理層と、その全結果を planner に raw data として渡す情報設計
- 3 役割 LLM (brain/extractor/planner) の温度分離、pydantic-ai の `output_type` による型安全な構造化出力、429/503 retry、logfire instrumentation
- 構造化カテゴリ別メモリ + 別 LLM による post-conversation extraction + dedup + LRU + decay + common-ground tracking の洗練されたメモリアーキテクチャ
- `FUTURE_WORK.md` の批判的自己分析が **設計上の作為性を減らす方向** へ向けられており、研究としての成熟度を示す

### 改善点・提言
- `PLANNER_PROMPT` (約 250 行) に「攻撃は最後の選択肢」「ジレンマでも attack を選ばない方が普通」など規範的指示と worked example が多く、agent の行動自由度を抑制している。`FUTURE_WORK.md` で指摘されている通り、prompt の構造強制を緩めて LLM の自然な判断に委ねる方向の実験 (例: A/B test) を行うとさらに創発が観察可能になる
- Maslow Lv1-3 のハードゲートと Big5 の 3 段階離散変換 (`_hp_threshold`, `_attack_for_food_policy`) を **連続値 + soft weighting** に置換すると、ペルソナ間のニュアンス差と多動機並列性が表現可能になる
- `poc_world.py` (2016 行) と `poc_12agents.py` (1350 行) の monolithic 構造を、world/physics/planner_prompt/scenarios/personas に分割するとさらなる拡張性が得られる
- scenario と planner prompt を YAML/別ファイルに外部化すると、非エンジニアの研究者も新しいシナリオを定義できる
- `tests/` ディレクトリでの単体テスト (`_expected_hits`, `_kill_probability`, `_classify_attack` など決定的ロジック) があると再現性とリファクタ耐性が向上する

### 一言コメント
学術論文として通用しうる定量的発見を備えた、ハッカソン提出物の中で突出して野心的かつ完成度の高いプロジェクト。次の課題は「設計上の作為性をいかに減らして、より純粋な創発を観察するか」という著者自身が掲げる方向にある。
