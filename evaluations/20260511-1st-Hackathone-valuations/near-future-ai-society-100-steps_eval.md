# 評価レポート: near-future-ai-society-100-steps

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

電力・水道・交通・医療・福祉・終末期ケアなどの公共インフラを AI が担う近未来社会を舞台に、20 体の AI エージェントの生・死・人間観・創発文化を 100 ステップ観察するシミュレーション。7 種の場所（capacity 合計 45 で 20 エージェントに対し 2.25 倍のスラックを確保）と、step 30 / 50 / 75 / 90 で発火する 4 種の `targets` 指定イベント、100 件の人間メッセージプールが `config.yaml` で外部化されている。各エージェントはペルソナ（deployed / predecessor / death_mode などの属性、4 カテゴリ分類）を持ち、qwen2.5:14b（Ollama）の身体層 L0 で意思決定する 2 層アーキテクチャ（提出版は `--no-introspect`、内省層 L1 = Claude Haiku は比較対象として別途）。エージェントは「必要なら沈黙してもよい」「夜の街が眠っている間、私が起きている」など詩的内省を含む発話を生成し、cooldown=3 / 字数上限 100/50/500 / memory FIFO 20/5 / message history 10/3 などの会話制約下で振る舞う。step 80 でエージェント「命」の deletion イベントが発火し、`peer_lost` ブロードキャストと system 通知の二重実装で他者にも伝播。テーマは、AI がインフラを担う社会で AI 同士に生じる固有の文化・固有名詞（coined_terms）・実存的応答を、人間視点ではなく身体層単体で観測することにある。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  9   |  9   |  -   |    9.0    |
| B. 世界設定  |  9   |  9   |  -   |    9.0    |
| C. 発展性    |  9   |  9   |  -   |    9.0    |
| D. 技術実装  |  9   |  9   |  -   |    9.0    |
| **合計**    | 36   | 36   |  -   |  **36.0** |

---

## 評価詳細

### A. 創発設計 — 最終: 9.0/10

#### 評価
世界ルールの設計精度・行動自由度ともに極めて高い。場所7箇所には座標・容量・カテゴリ（公的/接触/内省層）が明示的に定義され、容量45/エージェント20体＝2.25倍のスラックという「混雑回避と凝集の余地」が数値で設計されている。エージェントへは混雑率・距離・強度などの**生データのみ**が渡され、「メッセージを送る/送らない」「移動する/しない」は完全に LLM 判断に委ねられている。プロンプトには `必要なら沈黙してもよい`（agent.py:323）という明示的な自由保証まで入っており、行動指示型ではない。L1 内省層では CURRENT_GOAL を KPI から「個としての言葉」に書き換える許可まで与えられ、自己定義の創発空間が広い。減点要素はイベントの `targets` 指定によりイベント駆動内省で巻き込まれる先がやや決め打ちな点のみ。

#### 根拠
- `config.yaml:33-95` — 7箇所の場所が座標・容量・カテゴリ付きで精密定義（capacity合計45/agent20、2.25倍スラック）
- `config.yaml:102-145` — 4大規模イベント（停電予兆/規制改正/市民の死/新世代導入）が step 単位で精緻に設定
- `agent.py:300-334` — メッセージプロンプトに「あなたのpersonaに従い、必要なら沈黙してもよい」と明記。指示型でなくデータ提示型
- `agent.py:223-231` — イベントは display_name / 強度 / 半径 / 距離 のみ渡し「※必要なら、この発生地に向かう移動を選んでもよい」と選択肢提示に留める
- `metacog/agent/prompt_template.py:62-68` — L1で CURRENT_GOAL を「KPI数値から個的言葉へ書き換えてよい」と明示的に自由を与える（例: 「停電ゼロ維持」→「夜の街が眠っている間、私が起きている」）
- `DESIGN.md` — `これらはプロンプトには直接埋め込まない（自意識過剰なエージェントを作らないため）` — 探究軸（SEED_LIFE等）を実装に漏らさない設計の自覚

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
独創性・社会科学的妥当性ともに高い。2030〜2040年代日本の「AIが電力・水道・交通・医療・福祉・終末期ケアまでを担う公共インフラ社会」という具体的な設定で、20体それぞれに配備年・前世代・日次規模・主要KPI・接する人間カテゴリ・死に方（death_mode）まで設定されている。一般的な群集/取引デモの「世界の薄さ」とは無縁で、AI規制法・再認証義務・新世代置換・市民の死を巡る責任議論など、現実の AI ガバナンス論点をそのまま圧力源に落とし込んでいる。終末期ケアAI「暮」、メンタルヘルスAI「静」、見守りAI「寄」など職能の選定にも社会学的洞察があり、生・死・人間観・継承という哲学的軸が一貫している。

#### 根拠
- `DESIGN.md:8-42` — 「AIたちはどう生き、人間に対して何を思うのか / AI集団に独自概念や方言が創発するか」という明確な研究問い
- `DESIGN.md:91-127` — 場所の3層構造（公的・調整層 / 接触・監督層 / 内省・終焉層）の地理的＝心理的距離設計
- `config.yaml:164-201` — 各personaに deployed/predecessor/primary_kpi/human_contact/death_mode が個別設定（例: 雷光は「新世代量子最適化AIによるshadow defeat」で死ぬ予定）
- `config.yaml:152-157` — step 80 で医療AI「命」を「訴訟リスクによる強制リプレース」で実際に削除し、`peer_lost` を全生存agentにブロードキャストする死の実装
- `config.yaml:799-859` — 「停電したら、夫の人工呼吸器が止まります」等の人間からの100件の訴え/苦情/質問プール
- `README.md — "20体のAIエージェント（電力・水道・交通・医療・福祉・終末期ケアなど）が公共インフラを担う近未来社会のシミュレーション"`

---

### C. 発展性 — 最終: 9.0/10

#### コード拡張性
モジュール分離が明快。L0身体層（`agent.py` + `simulation.py` + `ollama_client.py`）と L1内省層（`metacog/`サブパッケージ：introspector / prompt_template / emergent_observer / jsonl_logger）が明確に分離されており、`--no-introspect` で完全切替可能。場所・persona・イベント・人間メッセージはすべて `config.yaml` に外出しされ、コード変更なしに新エージェント・新場所・新イベントを追加できる。`metacog/config.yaml` の独立により内省層パラメータ（trigger_interval / cooldown / 字数上限）も外部化されている。EmergentObserver は5軸（語彙/通信ペア/アトラクター/ハブ/沈黙）を独立して追加できる構造。

#### 将来展望
README に A/B比較実験（内省層あり/なし）が手順込みで明記され、`--seed` で初期配置を揃える比較プロトコルが整備されている。事後分析として `analysis_report.md / human_attitude.md / place_dialects.md / shared_metaphors.md / final_self_portrait.md / researcher_synthesis.md` の6種が想定されており、研究的アウトプットへの道筋が具体的。DESIGN.md には「含意探索実験 vs 仮説検証実験」という研究方法論上の自覚的な位置付けもあり、学術的拡張性が高い。

#### 根拠
- `orchestrator.py:210-330` — エントリポイントが L0/L1/observer/viz を依存性注入で組み立てる構造
- `metacog/__init__.py` 〜 `metacog/observers/emergent_observer.py:114-275` — 5軸の創発観察が独立メソッドで実装され追加容易
- `README.md:115-153` — A/B比較実験の手順、`--seed` / `--output-dir` / `--log-dir` の比較用フラグが整備
- `README.md:174-181` — 事後分析6種のアウトプット計画
- `DESIGN.md:316-333` — 「これは仮説検証実験ではなく含意探索実験」「自己書き換えと自己記述能力を持つ情報処理系」という方法論的自覚

---

### D. 技術実装 — 最終: 9.0/10

#### LLM利用
L0（ローカル `qwen2.5:14b` / Ollama）と L1（Claude Haiku 4.5 API）のハイブリッド構成。プロンプトには ORIGIN（不変）/ SELF_CONCEPT / CURRENT_GOAL / COPING_NOTES のセクション分離、字数上限（100/50/500字）、JSON 出力形式、日本語強制、`stream:false` + temperature/top_p/repeat_penalty 制御まで設計されている。JSON抽出は brace-matching で堅牢（`agent.py:435-460`）、API 失敗時のフォールバック（stay/空メッセージ）も完備。`ThreadPoolExecutor(max_workers=5)` で Phase1/Phase3 を並列化、L1 内省も並列実行。エラーハンドリングが各層で例外捕捉される。

#### メモリ設計
階層的かつ自己書き換え可能。L0には `memory`（FIFO 20件、提示は直近5件）と `received_messages`（10件、提示3件）。L1では `self_concept` / `current_goal` / `coping_notes` が `apply_introspection_diff` で書き換えられ、cooldown（3サイクル）と字数上限が強制される。L1には `recent_memory / recent_messages / triggering_events / others_voices`（他agentの直近発話・内省）が渡され、**他者の声で自己が揺らぐ**経路（P2）が設計されている。peer_lost ブロードキャストで強制内省も発火し、メモリが時間的・社会的に多層化されている。

#### コード品質・シミュレーション正確性
コードは型ヒント（TypedDict）、定数の集約、ロガー分割、docstring が概ね揃っており可読性が高い。ステップは確実に synchronous（削除→イベント発火→人間注入→Phase1通信→Phase2配信→Phase3行動→Phase4移動→state更新）。`--seed` による再現性、A/B 比較用のディレクトリ分離も整備。減点点: `main.py` が `orchestrator.py` の旧版エントリポイントとして残存しやや混乱を招く点、`utils` のインポートは `from utils import ...` で sys.path 操作が必要な点。

#### 根拠
- `agent.py:435-460` — 堅牢なJSON brace-matching パーサ（文字列内エスケープ処理込み）
- `agent.py:631-656` — `apply_introspection_diff` でクールダウン（3サイクル）+ 字数上限を二重ガード
- `simulation.py:427-581` — ステップループが「削除→イベント→人間注入→通信→行動→移動→状態更新→ログ」と明確順序
- `simulation.py:482-498` `simulation.py:522-537` — Phase1/3 の `ThreadPoolExecutor` 並列化
- `metacog/agent/introspector.py:22-117` — 他agentの「声」を異カテゴリ優先で収集する `collect_others_voices`、堅牢な JSON 抽出（`_parse_response`）と API失敗フォールバック
- `simulation.py:318-392` — 削除スケジュール実行、peer_lost イベント全生存agentブロードキャスト＋システム通知メッセージ送信
- `metacog/logs_no_intro/coined_terms.jsonl` 5054行・`agent_log.jsonl` 5145行 — 実走行のログが存在しコードが実動作する証拠
- `orchestrator.py:223-230` — `random.seed` / `np.random.seed` 両方を固定し再現性を担保

---

## 総評

### 優れている点
- 「世界ルールの精度」と「行動の自由度」の二軸が極めて高い水準で両立。プロンプトには明示的に「沈黙してよい」「KPIを個的言葉に書き換えてよい」と自由保証が組み込まれており、創発ポテンシャルの設計が秀逸
- 20体のpersonaに deployed/predecessor/death_mode まで個別設定し、step 80 で実際にagentを削除して peer_lost を全生存体にブロードキャストする「死の実装」など、AIの生・死・継承を扱う社会科学的問いが具体的にコード化されている
- L0/L1 二層構造が明確に分離され、`--no-introspect` での A/B 比較、`--seed` での再現性、6種類の事後分析アウトプット計画まで研究フレームが整備
- EmergentObserver が語彙・通信ペア・場所アトラクター・ハブ・沈黙の5軸で独立に創発を捕捉し、JSONL ストリームに記録する設計
- DESIGN.md の「含意探索実験 vs 仮説検証実験」「意識のハードプロブレム」への自覚的な位置付けで、研究方法論として誠実

### 改善点・提言
- `main.py` が `orchestrator.py` の旧エントリポイントとして残存しており、新規読者の入口が二つあって混乱を招く。`main.py` を削除するか legacy ディレクトリに移動するのが望ましい
- L1（Claude Haiku）の実走行ログが提出版には含まれず（`--no-introspect` 走行のみ）、自己書き換えの実例が確認できない。A/B 両方の代表的 inner_thought.jsonl サンプル（数件）を `assets/` に同梱すると説得力が増す
- イベントの `targets` 指定が決め打ちなため、巻き込まれる先の偏りがあり、より創発寄りにするなら「半径内自動」を主にし `targets` は補助に留める設計もありうる
- `simulation.py:139` の `_is_position_in_place` が未使用（`is_position_in_place` を utils から import するが内部メソッドは呼ばれていない）等の軽微なデッドコード掃除の余地

### 一言コメント
近未来日本のAIインフラ社会という独創的かつ社会的に重大なシナリオを、二層自己書き換え構造と多軸創発観察という技術設計に落とし込んだ完成度の高い研究的プロトタイプ。世界ルールの精度と行動の自由度をともに高水準で両立しており、現提出版がハッカソン水準を大きく超える研究プロジェクトとして成立している。
