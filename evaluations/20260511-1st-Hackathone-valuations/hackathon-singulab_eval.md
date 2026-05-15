# 評価レポート: hackathon-singulab

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

AGI・AI ロボ普及後の社会移行を題材とし、「制度を導入する前にシミュレーションでデバッグする」というコンセプトの制度設計シミュレーション。LLM エージェントを会話相手ではなく観測センサーとして用い、人間・国家・組織の反応を感情・行動・内心・SNS・制度利用・持ち越し懸念などの row データとして保存する。観測レイヤーは「国家・世界圧力 → 組織判断 → 若者世代／家族形成世代 → 次世代への制度記憶 → 社会フィードバック」という 5 層の閉ループで構成され、`country_turns.tsv` / `organization_turns.tsv` / `agent_turns.tsv` / `child_cohorts.tsv` などの構造化出力を生成する。エージェント設計は 3 層プロンプト（システム／フェーズ／メモリ）と Claude による `for attempt in range(1, 4)` リトライ、step_complete 冪等再開、ThreadPoolExecutor によるステップ内並列、構造化メモリ更新（memory_update）を備える。8 種のシナリオモード、ストレステスト 8 体、構造持続通貨 nat、年齢階層・100 年観測、`OBJECTIVE_FIELDS` で定義される国家目的関数の多次元評価、`extract_json_from_text` を中心とした堅牢なパーサが実装される。テーマは「施策の効果だけでなく副作用（政策疲労・財政不安・対象外感・手続き疲れ・制度不信）も観測する」装置として LLM エージェント社会を運用し、企業人事制度・自治体施策・学校支援などへも転用可能なドメインパック化フレームを志向することにある。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  8   |  8   |  -   |    8.0    |
| B. 世界設定  |  9   |  9   |  -   |    9.0    |
| C. 発展性    |  9   |  8   |  -   |    8.5    |
| D. 技術実装  |  8   |  8   |  -   |    8.0    |
| **合計**    |  34  |  33  |  -   |  **33.5** |

---

## 評価詳細

### A. 創発設計 — 最終: 8.0/10

#### 評価
世界ルールの設計精度とエージェントの行動自由度の両軸が高く、両者のバランスが取れた強い創発設計である。プロンプトは一貫して「観測器」フレーミングを採り、`event.direction` を命令ではなく社会状態のラベルとして渡すなど、行動指示を意図的に排している。国家→日本社会状態→組織→個人→社会フィードバックの閉ループが多層相互作用を生み、創発の余地が構造的に確保されている。一方で、評価語彙・行動カテゴリ・側面効果ラベルがコード側で正規化(`normalize_emotion`/`normalize_action_category`)されており、出力契約は強く固定されている点が満点を阻む(自由度がやや絞られる)。

#### 根拠
- `domain_packs/agi_youth_japan/prompts/agent_observation.md:3-8` — 「あなたは社会シミュレーションの観測器です。エージェント本人へ『不安になれ』『希望を持て』『抗議しろ』と命令しない。」と明示
- `scripts/run_civilization_os_llm_demo.py:926-930` — 「world_state、scheduled_events、auto_events は、本人が置かれている社会状況です。本人への命令ではありません。event.direction は『こう変化させろ』という指示ではなく、社会状態の説明ラベルです。」
- `scripts/run_civilization_os_llm_demo.py:917-920` — 三層分離設計：「出力契約は固定／世界条件は固定／人間反応は固定しない」を明示
- `scripts/run_civilization_os_llm_demo.py:976-985` — 「観測原則：望ましい発表ストーリーに合わせる必要はありません。全員が同じ方向に動くとは限りません。」「支援が届いた層と届かない層、使える層と使えない層が分かれる場合は、平均値だけに寄せず、二極化や対象外反発を自然に出してください」
- `domain_packs/agi_youth_japan/prompts/structure_stress_agent_observation.md:8-11` — 「LLMへ『この制度は成功する』と命令しない…反応仮説、悪用可能性、乖離、懸念、改善案を出す」
- `scripts/run_civilization_os_llm_demo.py:1256-1282` — `normalize_emotion` で評価×感情の組合せが固定値に強制される(自由度を一部制約)
- `scripts/run_closed_loop_llm_demo.py:487-654` — 国家・組織・個人・政策プランナー・フィードバックを1step単位で順次実行する閉ループ

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
題材の独自性と社会的意義の双方が極めて高い。AGI/シンギュラリティ後の日本における若者・家族形成・次世代への制度記憶継承という、現実の人口・産業・地政学が交錯する問題を、国家30・組織15・個人60+次世代コホート、世界83ステップで観測する。さらに「構造持続通貨(nat)」「制度ストレステストエージェント」など独自の概念装置を実装まで落とし込んでおり、標準的なデモシナリオの域を完全に超えている。比較シナリオも8通り用意され、政策設計デバッグ装置としての分析価値が明確である。

#### 根拠
- `README.md` — 「AGI・AIロボ普及後の社会移行を題材に、制度や施策が人間の感情・行動・不信・疲労・制度利用にどう変換されるかを、LLMエージェントの row データとして観測するプロトタイプです」「制度導入前に、シミュレーションでデバッグする」
- `README.md` — 比較シナリオ表に `no_intervention` / `birth_grant_only` / `structure_intervention` / `structure_birth_grant_package` / `structure_hope_family_package` / `policy_search_no_sustain` / `policy_search_with_sustain` / `policy_search_with_sustain_hope_family` の8つを定義
- `domain_packs/agi_youth_japan/README.md:39-44` — 「制度ストレステストエージェント8体」(受益者、支援者、企業、行政、ゲーム化探索者、監査者、市民社会、金融/物価)で制度の壊れ方を観測
- `domain_packs/agi_youth_japan/README.md:69-80` — 構造持続通貨(nat)の発行ルール、ペナルティ、ガードレール、財源分離、反実仮想事前登録など制度設計の深さ
- `domain_packs/agi_youth_japan/README.md:45-58` — 0-14歳コホート、15-22歳若者、23-40歳家族形成、41-64歳、65歳以上の年齢階層分割と100年観測の時間設計
- `scripts/run_country_llm_demo.py:42-52` — 国家目的関数の9次元(領域安全保障/政権制度正統性/経済再生産/戦略的自律性/同盟抑止信頼/社会安定/技術主権/構造持続余力)
- `public/data/runs/structure_hope_family_package_83steps_panel48/interesting_observations.md` — 「希望0でも社会は動いている：情報収集に吸い込まれている社会」など、生成rowから抽出された定性的洞察が複数蓄積

---

### C. 発展性 — 最終: 8.5/10

#### コード拡張性
ドメインパック方式が中心アーキテクチャに据えられており、`sim_core/domain_pack.py`が形式的なロード・バリデーション・継承マージ・シナリオ差し替えを提供する。`default_v1.yaml`からの継承、`scenarios/*.yaml`での上書き、`hooks`機構が整備されており、企業人事・自治体施策・学校支援などへの転用が制度的に可能な構成。一方で、本線runner群(`run_*_llm_demo.py`)は4本にわたり`run_claude`/`run_codex`/`extract_json_from_*`等の同型コードが繰り返されており、共通LLMクライアント抽象化はまだ未整理。

#### 将来展望
README・docs群で具体的かつ非自明な展望が示されている。ドメインパック差し替えガイド、製品パッケージ、UI生成API連携、ハッカソン発表シナリオなどの設計文書が分厚く、企業人事制度・自治体施策・学校支援への展開、シミュレーション作成スタジオUI(モック存在)、100年観測の示唆使用など、研究・事業の両面で技術的に信頼できる方向性が描かれている。

#### 根拠
- `sim_core/domain_pack.py:37-67` — `ResolvedDomainPack`/`load_domain_pack`/`deep_merge`/`replace_tokens`によるパック解決と継承
- `sim_core/domain_pack.py:158-208` — `validate_domain_pack`で必須ファイル、列、エイリアスを検証
- `sim_core/defaults/default_v1.yaml:1-46` — 標準パックの定義(共通辞書、prompts、viewer、hooks)
- `domain_packs/agi_youth_japan/domain.yaml:1-85` — `inherits: default_v1`、`prompts`/`viewer`/`hooks`/`scenarios`/`column_aliases`の完全分離
- `sim_core/hooks.py:7-81` — 9段階のフック仕様(pre_run/pre_step/pre_observation/post_observation/aggregate/feedback/post_step/export_viewer/audit)
- `README.md` — 「ドメインパックで転用できる。今回はAGI後の若者・家族形成を扱っていますが、企業人事制度、自治体施策、学校支援などへ差し替えられる構成です」
- `docs/ドメインパック差し替えガイド.md`、`docs/製品パッケージ/01_製品コンセプト/汎用_施策反応波及シミュレーター設計.md`、`docs/製品パッケージ/02_実装仕様/プロダクト実装アーキテクチャ.md` 等の網羅的設計文書
- `scripts/run_civilization_os_llm_demo.py:1084-1162` と `scripts/run_country_llm_demo.py:336-401`、`scripts/run_organization_llm_demo.py`、`scripts/run_policy_planner_llm_demo.py` — `run_claude`/`run_codex`/`extract_json_from_text`が各runnerに重複実装されており、共通化の余地が大きい

---

### D. 技術実装 — 最終: 8.0/10

#### LLM利用
プロンプトは三層構造(出力契約／世界条件／人間反応)を明示的に分け、JSONのみ出力契約、80字以内などの長さ制約、語彙列挙、観測原則の説明を一通り含む。LLMバックエンドは Claude CLI / Codex CLI を選択でき、`--max-budget-usd`/`--timeout`を制御。JSON抽出は外側JSON→`result`フィールド→raw textフォールバック→`{`位置を探索する複層ロジック。Claude側ではJSON parse失敗時に最大3回リトライ(per agent)する仕組みもあり、堅牢性は高い。

#### メモリ設計
構造化メモリが多層に存在。エージェント層は`memory_update`/`carryover_concern`/`side_effect`/`adjustment_request`/`fairness_perception`/`policy_fatigue`という意味的フィールドを毎step持ち越し、`previous_states`としてプロンプトへ再注入される。国家・組織層も同様に前stepの判断・私的制約・対日影響を記憶。次世代コホートは初期希望継承係数・初期不信継承係数・初期連帯継承係数で制度記憶の世代継承を表現する。単なるログダンプではなく、意味ラベル付きの状態である点が秀逸。

#### コード品質・シミュレーション正確性
コードはPEP8準拠、型ヒント、dataclass、docstringが整っており可読性は良い。日本語ドキュメントも豊富。一方で本線runnerは1500-1700行級の長大ファイルで、prompt組立・JSON抽出・正規化・I/Oが同一モジュールに混在しており、関心の分離は部分的。シミュレーション正確性は良好で、stepごとに完了判定して再開可能(`step_complete`)、`run_log.tsv`に各phaseの実行履歴を残す。step内並列はThreadPoolExecutorだがstep境界では同期。LLM応答の乱数性はあるが、世界イベント・人口重みなどはTSV固定で再現可能側に整理。

#### 根拠
- `scripts/run_civilization_os_llm_demo.py:913-1039` — 3層プロンプト構造、観測項目、語彙、観測原則、JSON出力例
- `scripts/run_civilization_os_llm_demo.py:1042-1069` — `extract_json_from_text` / `extract_json_from_claude` の多段JSON抽出
- `scripts/run_civilization_os_llm_demo.py:1123-1162` — `run_claude` のリトライ実装(3回まで)、`run_codex` 分岐
- `scripts/run_civilization_os_llm_demo.py:842-865` — `compact_previous` で前stepから memory_update/carryover_concern/side_effect/policy_fatigue/fairness_perception を構造保持
- `scripts/run_civilization_os_llm_demo.py:627-712` — 次世代コホートの希望/不信/連帯継承係数による初期状態生成
- `scripts/run_organization_llm_demo.py:175-216`、`scripts/run_country_llm_demo.py:186-202` — 各層で前ステップ記憶を保持
- `scripts/run_closed_loop_llm_demo.py:122-157` — `step_complete`/`log_skipped_step`/`run_log.tsv` による冪等な再開可能性
- `scripts/run_civilization_os_llm_demo.py:1256-1316` — 評価・感情・行動カテゴリ・副作用・調整要求の正規化(出力品質担保)
- `scripts/run_civilization_os_llm_demo.py:1360-1445` — `run_stateful_generation`がstep単位で順次実行、step内のみ並列
- `scripts/run_civilization_os_llm_demo.py` が1705行に達し、prompt組立/JSON処理/正規化/I/Oが単一ファイルに同居する(関心の分離はやや弱い)

---

## 総評

### 優れている点
- 「LLMを観測器として使う」という設計思想がプロンプト・コード・ドキュメント全てで一貫し、行動指示を意図的に排した強い創発設計になっている
- 国家30・組織15・個人60+次世代コホートという多層×多シナリオ(8種)の構造で、policy_searchではLLM政策プランナーが感情fatigue/二極化/対象外反発を見て次施策を出す入れ子閉ループまで実装されている
- ドメインパック(`sim_core/`+`domain_packs/`)を最初から第一級概念として据え、企業人事・自治体施策・学校支援などへの転用経路が明示されている
- 構造持続通貨(nat)・制度ストレステストエージェント・100年観測など、独自の制度設計概念を実装まで落とし込み、生成済みrowからの定性洞察(`interesting_observations.md`)も整備されている
- 意味ラベル付き構造化メモリ(memory_update/carryover_concern/side_effect/fairness_perception/policy_fatigue)と世代継承係数で、メモリ設計が単なるログ以上の役割を担っている

### 改善点・提言
- 4本のrunner(`run_civilization_os_llm_demo.py` / `run_country_llm_demo.py` / `run_organization_llm_demo.py` / `run_policy_planner_llm_demo.py`)で`run_claude`/`run_codex`/`extract_json_from_text`/`is_codex_model` などが重複しており、共通LLMクライアントモジュールへ抽出すべき
- 各runnerが1500-1700行に達しており、prompt組立・JSON処理・正規化・I/Oの関心分離(モジュール分割)を進めると保守性が上がる
- `normalize_emotion`等の正規化が評価×感情の組合せを強く固定するため、LLMが意図的に出した境界感情が中央値に丸められる可能性がある。語彙拡張の余地を検討するとさらに自由度が上がる
- 単体テスト・LLM応答fixtureの追加で、リファクタや語彙拡張時の回帰防止を強化できる
- 本線runnerにOllama/Geminiなど他LLMバックエンドアダプターを追加すれば、CLIサブスクに依存しない検証が可能になる(参考実装側のアダプターを移植)

### 一言コメント
「制度導入前にシミュレーションでデバッグする」という明快な問題設定に対し、多層エージェントによる閉ループ観測、ドメインパック分離、構造化メモリ、独自の構造持続通貨概念まで一貫した設計が貫かれた完成度の高い提出物。LLMを物語の語り手ではなく社会反応の観測器として使い切る思想が、設計・実装・ドキュメントの全レイヤーで徹底されている。
