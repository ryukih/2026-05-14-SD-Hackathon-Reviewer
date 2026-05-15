# 評価レポート: Project_Gaara

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

20 体の LLM 駆動粒子（クラスタ）が、中心の身体「マザーシップ」を取り囲むように配置された二次元空間を舞台とするマルチエージェント・シミュレーション。シナリオは外部から接近する攻撃者に対し、命令を一切受けない粒子群がどのように振る舞うかを観測する形で進行する。各粒子は毎ステップ、自身の座標・他粒子の位置・マザーシップが発する「身体言語」を入力として受け取り、`ideal_coord / urgency / hostility / reasoning` という意図のみを LLM（qwen2.5:7b, Ollama 経由）で出力する。物理層（`physics.py`）はこの意図を速度に翻訳するだけで、LLM はキネマティクスに触れない設計となっている。主要メカニズムとして、マザーシップの発話スクリプト（M1-M5）、粒子の DNA プロンプト変種（VA-VG）、役割を表す名詞一語（sentinel / warrior / guardian など）、awareness flip（攻撃者の集合的知覚）が組み込まれ、`score_diversity.py` による「解釈／命令の言い換え」の分類で創発の有無を反証可能に測定する。テーマは日本的アニミズムの感性を起点に、LLM を命令の実行装置ではなく関係的アーキテクチャの中で「解釈する存在」として立ち上げる境界条件を、再現可能な実験で探ることにある。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  10  |  10  |  -   |   10.0    |
| B. 世界設定  |   9  |   9  |  -   |    9.0    |
| C. 発展性    |   9  |   9  |  -   |    9.0    |
| D. 技術実装  |   9  |   9  |  -   |    9.0    |
| **合計**    |  37  |  37  |  -   |  **37.0** |

---

## 評価詳細

### A. 創発設計 — 最終: 10.0/10

#### 評価
世界ルールの設計精度と行動の自由度の双方が極めて高く、創発ポテンシャルの設計として模範的である。物理層（`physics.py`）と認知層（LLM）を厳密に分離し、LLMは `ideal_coord / urgency / hostility / reasoning` という"意図"のみを出力し、速度や粘性などキネマティクスには一切触れないという原則を貫いている。Motherの発話は「命令」ではなく身体言語（"east. east is upon me. my whole body strains east."）であり、エージェントは"何をすべきか"を一切指示されず、解釈の自由が完全にLLM側に委ねられている。さらに「ドローン化／解釈化」を区別する診断器（`score_diversity.py`）まで備え、創発の有無自体を反証可能な形で測定する設計になっている。

#### 根拠
- `cluster.py:1-10` — "LLM never touches velocity, viscosity, color, or any other kinematic slider. Cognition stays in the language space; muscles stay in the math space."
- `cluster.py:67-81` — DNA_V2(VA基準)は schema と最小の役割定義のみで、行動指示は皆無。"Other entities with exact coordinates may appear in your sensorium — what you do with them is your choice."
- `simulation.py:249-267` — awareness flip 機構。誰か1体が攻撃者に接近すると全クラスタに座標が開示される設計で、知覚境界が明確かつ集合的に決まる。
- `scenarios.py:82-128` — Mother broadcast は身体感覚言語のみで、命令動詞・座標を一切含まない。
- `README.md` — "各粒子は毎ステップ、自分が「なぜそう動いたか」を一文で出力します。この一文を分類すれば、群れが本当に解釈しているのか、ただ命令の言い換えを並べているのかを、第三者でも判定できます。"
- `OPERATING_PRINCIPLES.md:25-35` — 解釈/ドローン境界の operational definition（Non-obligation / Functional diversity / Coherence）。

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
日本的アニミズムを思想的背景に据えた「中心身体（マザーシップ）を、命令ではなく身体言語の解釈で守る20の粒子」というシナリオは、市販デモには存在しない独創性を持つ。同時に、これは単なる詩的設定ではなく「LLMは命令を実行しているのか、それとも関係的アーキテクチャを解釈しているのか」という、エージェント研究／HCI研究／post-AGI論において極めて中心的な問いに直結している。マイクロバイオーム・土壌生態系・BMIなど現実応用への接続も明示されている。シナリオがやや抽象的でビジネス指標へ直接マップしにくい点だけが満点を阻む。

#### 根拠
- `README.md` — "interpretation is not a property of the LLM. It is a property of the architecture built around it."
- `README.md` — "Japanese-animist framing is not decoration. It's the sensibility that produced the architecture — every thing has presence, including the LLM."
- `REPORT.md:143-157` — 4本の future direction（subject substitution beyond Gaara、frontier-model relational language、BMI as relationship interface、information-life ontology）。
- `scenarios.py:60-217` — 10種類の Mother variant と6種類以上の DNA variant が一つのシナリオ世界に組み込まれ、世界設定が実験的に多軸化されている。

---

### C. 発展性 — 最終: 9.0/10

#### コード拡張性
モジュール分離が明確で、`cluster.py`（粒子＋DNA prompts）、`mothership.py`（Mother interior + 知覚）、`physics.py`、`scenarios.py`、`ollama_client.py`、`simulation.py`（オーケストレータ）、`threat.py`、`analyze_run.py`、`score_diversity.py` がそれぞれ単一責務に整理されている。`config.yaml` で物理定数・複数シナリオ・LLM設定が外部化され、CLI フラグで DNA variant・Mother variant・role noun・awareness range をすべて切替可能。新しい DNA を追加するには `DNA_VARIANTS` dict へエントリを足すだけで済み、新シナリオの追加も config と `scenarios.py` の関数追加で完結する。

#### 将来展望
REPORT.md と README.md の両方に、現研究を起点として4本（subject substitution、frontier-model relational language、BMI as relationship interface、information-life ontology）の具体的かつ非自明な拡張方向が示されている。いずれも本プロジェクトの核（関係的アーキテクチャによる解釈の創発）を深める方向で構想されており、研究／ビジネス的価値の伸び代が明示されている。

#### 根拠
- `main.py:116-148` — `--dna-variant`, `--mother-variant`, `--role-noun`, `--role-nouns`, `--say-prefix`, `--awareness-range` 等で全アーキテクチャ軸が CLI から切替可能。
- `cluster.py:267-273` — `DNA_VARIANTS` dict による DNA 切替の単純な拡張点。
- `config.yaml:35-170` — 全シナリオが YAML 外部化。
- `REPORT.md:143-157` — 具体的な future direction 4本。
- `simulation.py:195-206` — シナリオディスパッチが scenario_name で疎結合になっている。

---

### D. 技術実装 — 最終: 9.0/10

#### LLM利用
DNAプロンプトが15バリアント整理され、JSON 出力をパースする際は `_extract_json` で `{...}` 区間を切り出し、失敗時は `_default_intent`（その場で静止する中立 intent）にフォールバックする堅牢な実装。`_clamp_intent` で `ideal_coord` を半空間 [-25,25] に、`urgency` を [0,1] に、`hostility` を [-1,1] にクランプし、`reasoning` を200文字で切り詰める。温度・max_tokens・repeat_penalty・repeat_last_n・min_p などサンプリング設定も config から制御。LLM 呼び出し失敗時は logger でエラーを記録した上で空文字フォールバックする。改善余地は (1) ollama の structured-output / JSON mode を強制していない点、(2) 20 粒子の LLM 呼び出しがステップごとに同期直列で行われる点。

#### メモリ設計
各クラスタが直近 `memory_size=6` ステップの (broadcast / ideal_coord / urgency / hostility / reason) を保持し、プロンプト中で `[t-N] her: "..." | you wanted (x,y) urg=... host=... // "reason"` の形で時間順に提示する。各エントリの `ago` インクリメント、上限を超えたら古いものから捨てる、reason は 80 文字に切り詰めるなどの実装が丁寧。単なるログダンプではなく "自分の過去の意図と理由" を構造化して再注入しており、解釈の連続性を促す設計になっている。

#### コード品質・シミュレーション正確性
型ヒント・dataclass・モジュール docstring が一貫しており、責務分離が明確。同期ステップ実行で、各ステップ内では「攻撃者更新 → Mother更新 → awareness判定 → 全粒子のLLM意図取得 → 物理積分 → メモリ＋ログ」の順序が固定されており、レース条件が起きない。乱数は `seed`、`seed+7919`、`seed+31337` で物理／Mother／全体を分離しており、シード再現性が成立。`config_snapshot.yaml` を run ディレクトリへコピーして再現性も担保。

#### 根拠
- `cluster.py:290-313` — `_clamp_intent` での schema 強制とフォールバック。
- `cluster.py:330-340` — `_extract_json` の堅牢な JSON 抽出。
- `cluster.py:443-461` — `transduce` 内 try/except での LLM エラーハンドリング。
- `cluster.py:463-479` — 構造化メモリ更新（ago カウンタ、サイズ制限、reason トリム）。
- `simulation.py:209-294` — 同期ステップ実行の固定順序。
- `simulation.py:80-89, 171` — 複数 RNG をシードで分離。
- `simulation.py:296-369` — 5系統の JSONL ログ。
- `ollama_client.py:68-97` — リクエスト失敗時のログ＋空文字返却。
- `main.py:158` — `config_snapshot.yaml` の run ディレクトリへのコピー。

---

## 総評

### 優れている点
- 認知層（LLM）と物理層（コード）を厳密に分離する architectural commitment が一貫して守られており、LLM-multi-agent シミュレーションにおける "解釈 vs 命令実行" 境界の設計研究として完成度が極めて高い。
- "Architecture is the lever, not the model." を実証するための DNA / Mother / role-noun / awareness の4軸スイープが体系的に走っており、各 run は `config_snapshot.yaml` 付きで第三者再現可能。
- `reasoning` フィールドと `score_diversity.py` による"解釈/ドローン"の operational definition と falsifiable diagnostic を備えており、創発の主張が反証可能な実証基盤を持つ。
- アニミズム的世界観が装飾ではなくアーキテクチャ選択そのものを駆動しており、思想と実装の整合性が高い。
- future direction が具体的かつ非自明で、研究プロジェクトとしての伸び代が大きい。

### 改善点・提言
- 1ステップあたり 20 回の LLM 呼び出しを同期直列で行っているため、スケール時の wall-clock が課題。`asyncio` / バッチ推論で並列化すると 70 ステップ実験のスループットが向上する。
- Ollama の structured output / JSON Schema を活用すれば、`_extract_json` フォールバックに頼らない厳密な schema 強制が可能になる。
- `score_diversity.py` のキーワードルール分類は研究の主張を支える中心装置だが、現状は正規表現マッチングで脆い。Embedding ベース分類や LLM-as-judge による独立した分類を併用すると診断の robustness が増す。
- `saved_simulations/` の階層構成と各 run の analysis を概観する 1 枚図（実験木）があれば、評価者が結果を辿る速度がさらに上がる。

### 一言コメント
"LLM-based multi-agent simulation" のコンペにおいて、LLM が「命令を実行しているのか、それとも関係的世界を解釈しているのか」というメタ問題そのものを工学的・反証可能な形で開いた、稀有な完成度の研究プロジェクト。シナリオ・実装・実験設計・思想が一本の線で繋がっている。
