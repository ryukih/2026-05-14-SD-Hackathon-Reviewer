# 評価レポート: good_echo_iss_sim_cursor_50s_no_accident

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

ISS（国際宇宙ステーション）を模した閉鎖空間を舞台に、複数のクルーがどのようにストレスを受け、会話し、協力し、衝突し、修復するかを観測する多エージェント・シミュレーション。エージェントは ISS クルー（既定では 10 名）として実装され、各々がペルソナ、役割、疲労耐性、関係性を持ち、1 ステップ＝1 日として閉鎖環境内で生活・作業・対話を行う。主要メカニズムとして、混雑・睡眠・作業負荷・会話頻度・衝突・修復・孤立といった状態量をステップごとに更新し、`habitat_frames.jsonl` `messages.jsonl` `event_timeline.tsv` `nudge_effects.tsv` などのフレームデータとして出力する。Run A（ナッジなし対照群）と Run B（善性ナッジオブジェクトを配置した介入群）の A/B 比較を通じて、感謝の可視化・静穏時間・共同食・公平利用などの環境介入が関係修復や協力行動にどう影響するかを測定する設計である。本サブミッションは 50 ステップ・事故イベントなしの構成で、宇宙デブリ衝突などの外乱は含まずベースラインの閉鎖空間ストレスのみを扱う。テーマは「善性を育む環境設計」であり、性格や個人特性ではなく、空間配置・運用ルール・ナッジオブジェクトといった環境側の設計によって、高ストレス下での関係崩壊を防ぎ協力的振る舞いを引き出せるかという仮説検証を志向している。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  8   |  8   |  -   |    8.0    |
| B. 世界設定  |  9   |  9   |  -   |    9.0    |
| C. 発展性    |  9   |  9   |  -   |    9.0    |
| D. 技術実装  |  8   |  8   |  -   |    8.0    |
| **合計**    |  34  |  34  |  -   |  **34.0** |

---

## 評価詳細

### A. 創発設計 — 最終: 8.0/10

#### 評価
世界ルールの設計精度・行動自由度の両軸が高水準で釣り合っている。場所ごとのcapacity/occupancy、通信半径と「同じ場所同士のみ通話可能」の制約、4方向移動、火災は半径内のエージェントのみ知覚可能といった物理・知覚ルールが明示され、エージェントには定量値（占有率、距離、強度など）のみを渡している。プロンプトは「Decide what message」「Decide your action」と中立的で、「逃げろ」「警告せよ」といった行動指示を含まない。Run Bでは「善性オブジェクト」を場所の説明テキストとして世界側に埋め込み、行動指示ではなく環境アフォーダンスとして配置している点が秀逸。ただし `[Benevolence object: Talk-OK seat] welcomes conversation` などはやや誘導的な表現を含み、純粋なraw dataからわずかに踏み込んでいるため満点までは至らない。

#### 根拠
- `examples/spatial_demo/agent.py:282-301` — `_build_fire_section`はコメントで「Only quantitative data is provided ... No qualitative descriptions (e.g. "dangerous", "evacuate") are included.」と明示し、火災情報は数値のみ
- `examples/spatial_demo/agent.py:317-400` — メッセージプロンプトは「Decide what message you want to send」のみで、何をすべきかは指示しない
- `examples/spatial_demo/agent.py:116-139` — 通信ルール「Both outside OR both in same place」が明確に制約として実装
- `examples/spatial_demo/configs/config.iss.cursor.run_b.yaml:52,59,66` — 場所descriptionに「[Benevolence object: ...]」として配置物を埋め込み、世界の事実として提示
- `README.md` — 「単なる雰囲気ではなく、…衝突後に修復会話が起きたか、…どのナッジが、どのタイミングで効いたか」と創発を観測対象に明示

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
ISS閉鎖空間に「訓練されていない一般市民10名」（中国農村工場労働者、シリア難民、ケニアのパラアスリート、フランス元外交官など多文化背景）を50日間置く設定は極めて独創的。閉鎖空間ストレスと善性オブジェクト（持ち寄り棚・Talk-OK席・聖域マーク等）の効果検証はナッジ理論と組織心理学の交差点にあり、社会科学的意義が明確。さらにREADMEで災害避難所・病棟・介護施設・寮・船舶へのドメイン転用を明示しており、ビジネス／公共応用への射程が大きい。「善性は性格ではなく環境と運用で育つ」という仮説提示は本格的な社会実験設計。

#### 根拠
- `README.md` — 「避難所、病棟、介護施設、寮、企業の高圧プロジェクト環境では、…人の判断や関係性が壊れていきます。…このプロジェクトでは、その崩壊過程をLLMエージェントで再現し、環境側の介入…をA/B比較します」
- `examples/spatial_demo/configs/config.iss.cursor.run_a.yaml:30-40` — 多文化・多年齢層の具体的なペルソナ10名（出稼ぎ労働者、難民、母親、退職者等）
- `domain_packs/iss_benevolence/data/agents.tsv` — ベースラインストレス・自己効力感・希望度・脆弱性ノートまで定量化されたペルソナ表
- `domain_packs/iss_benevolence/data/objects_menu.tsv` — ハンドレール・メロディ、廃棄物サウンドボックス、リソース・スコアボード等、創造的なナッジオブジェクト群
- `README.md` — 「この観測によって、「善性」は性格だけで決まるものではなく、環境と運用によって育つのではないか、という仮説を検証します」

---

### C. 発展性 — 最終: 9.0/10

#### コード拡張性
`sim_core/`（domain pack ローダ・バリデータ・hook 規格）、`domain_packs/iss_benevolence/`（YAML/TSVベースの世界・ペルソナ・イベント定義）、`examples/spatial_demo/`（シミュレーションエンジン）、`scripts/`（状態構築・フィードバック・可視化出力）と層が分離。`hooks.py` で `pre_run/pre_step/pre_observation/post_observation/aggregate/feedback/post_step/export_viewer/audit` の9ステージが規格化されており、新ルールを差し込む拡張ポイントが明示。`runtime.profiles` で claude/codex/cursor の3エージェント × smoke/full × run_a/run_b の組合せがYAMLで切替可能。LLMバックエンドはOllamaと任意CLIをサポート。設定はYAML/TSVで完全外部化されており、コア書換なしでドメイン差替が可能。

#### 将来展望
READMEに「差し替え可能な設計」「終盤イベントだけを差し替える」「新しいドメインへの差し替え」と独立した節が設けられ、TSV列単位での具体的な差替手順（避難所なら `gym_floor / medical_corner / food_line / quiet_area`）まで例示。さらに「ストレスイベントを弱・中・大のように強度別に挿入」「チームは強くなったのか、それとも崩壊したのか」など、研究的に意味のある拡張方向を提示。提示された5つの代替シナリオ（睡眠不足・設備不具合・物資不足・通信遅延・緊急作業割り込み）はいずれも社会科学的に検証価値がある。

#### 根拠
- `sim_core/hooks.py:7-17` — 9ステージのフック仕様が定義され、`normalize_hooks` で外部からの差込が可能
- `sim_core/domain_pack.py:158-208` — `validate_domain_pack` がスキーマ／必須ファイル／カラム別名／集団重みまでバリデート
- `domain_packs/iss_benevolence/domain.yaml:67-81` — Claude/Codex/Cursorの3バックエンド×smoke/full×A/Bの12プロファイルを登録
- `examples/spatial_demo/llm_backends.py:290-325` — `create_llm_client` で provider文字列から ollama/command/cli を切替
- `README.md` — 「人物: ペルソナ、役割、…/ 空間: 部屋、動線、…/ イベント: 睡眠不足、作業遅延、…/ ナッジ: 感謝、静穏、…」と差替軸を明示
- `README.md` — 災害避難所向けの具体的TSV差替例（`gym_floor`, `medical_corner` 等）

---

### D. 技術実装 — 最終: 8.0/10

#### LLM利用
2段階プロンプト設計（Phase1: メッセージ決定（位置情報なし） → Phase2: 行動決定（位置情報あり＋自分の発話））により、発話と移動の因果順序を分離している。プロンプトは構造化されたJSON応答を要求し、`_extract_json_from_text` で文字列リテラル・エスケープを考慮したブレース対応パーシングを実装。`CommandLLMClient` には `max_retries`、`retry_backoff_seconds`、`timeout_seconds`、ANSIエスケープ除去、stdout正規表現フィルタなど運用的に堅牢な機構を実装。並列度 (`llm_parallelism`) で ThreadPoolExecutor の並列実行も対応。JSON失敗時のフォールバック解析もあり。

#### メモリ設計
`Agent.memory` は LLM自身が生成した `memory` フィールド（次ステップに何を覚えておきたいか）を `Step N: …` として蓄積。`memory_limit` で上限、`memory_size` で直近何件をプロンプトに含めるかを分離管理。`received_messages` は別バッファで `message_history_limit`/`message_context_size` 制御。単なるログダンプではなく、LLMが自己選択した思考継続線を残す設計は良質。一方、長期的な「関係」「人物別の蓄積」など階層化はなく、フラットなリスト止まりという点で改善余地あり。

#### コード品質・シミュレーション正確性
`simulation.py` は920行と肥大化しており、空間配置・火災・経済層（雇用・通貨）が同一クラスに混在している。一方で TypedDict による型注釈、dataclass、logger、設定スキーマ検証など基礎品質は高い。シミュレーションは「全エージェント決定 → 一括メッセージ送信 → 全エージェント行動決定 → 一括移動」と同期4フェーズで race condition を回避。`_run_in_parallel` は順序を保持。乱数シード（reproducibility）の明示は確認できず減点要因。

#### 根拠
- `examples/spatial_demo/simulation.py:590-769` — 4フェーズ（message decide → send → action decide → move）の同期実行ループ
- `examples/spatial_demo/agent.py:515-550` — ブレース深度＋文字列リテラル＋エスケープを考慮したJSON抽出
- `examples/spatial_demo/agent.py:678-687` — LLM生成 `memory` を `Step N:` プレフィックス付きで蓄積、`memory_limit` で打切り
- `examples/spatial_demo/llm_backends.py:152-274` — CLI実行のリトライ・タイムアウト・ANSI除去・stdout正規表現フィルタ
- `examples/spatial_demo/simulation.py:178-185` — `ThreadPoolExecutor` で順序保存並列実行
- `sim_core/domain_pack.py:158-208` — スキーマ／必須ファイル／カラム別名のバリデーション
- 減点要因: `examples/spatial_demo/simulation.py:24-176` 経済層・火災・空間が同一クラスに混在、ランダムシード初期化は未確認

---

## 総評

### 優れている点
- 「世界に物を置く」というナッジ理論を、エージェントへの行動指示ではなく場所descriptionへの埋め込みで実装している点が、創発設計として極めて思想的に一貫している。
- 10名の多文化・多階層ペルソナ（難民、出稼ぎ労働者、退職者、若年学生、元外交官など）が、ISSという非日常的閉鎖空間に置かれる設定は、ハッカソン提出物として高い独創性と社会的射程を持つ。
- `sim_core` / `domain_packs` / `examples` / `scripts` の4層分離、9ステージのhook規格、TSV/YAMLによる完全な設定外部化、複数LLMバックエンド対応により、ISS以外のドメイン（避難所・病棟・介護施設等）への展開が現実的に実装可能。
- 2段階プロンプト（発話 → 行動）と4フェーズ同期実行で因果順序とrace condition回避を両立し、CLI実行にもリトライ・タイムアウトを完備。
- A/B比較設計（同一外乱／善性オブジェクトのみ差分）が実験計画として明快で、`reciprocity_rate`、`repair_after_conflict_rate`、`bridge_agent_count`、`isolated_agents` 等のKPIまで事前に定義されている。

### 改善点・提言
- `simulation.py` 920行に空間・経済（雇用・通貨）・火災が混在している。経済層と火災層を別モジュールに分離すれば、ISS以外のドメイン適用時の認知負荷が下がる。
- 乱数シードの初期化が確認できず、A/B比較や複数実行の再現性が担保されない。`config.simulation.seed` を導入して `random` / `numpy` を一括seed設定する仕組みが望ましい。
- メモリ設計がフラットなリストに留まる。「ISS03に対する関係性」「持ち寄り棚との接触回数」のような構造化メモリ（人物別／オブジェクト別）が入れば、長期的な関係創発の解析が深まる。
- 場所descriptionの善性オブジェクト記述は「welcomes conversation」「encourages moving」などやや誘導的。完全な raw data 化（例: `objects: [{id: talk_ok_seat, type: invitation_marker, location: cupola}]`）とすれば、LLMが自分でアフォーダンスを解釈する余地がさらに広がる。
- 結果分析パイプライン (`analyze_iss_pair.py`) と可視化UI (`iss_habitat_demo.html`) はあるものの、本READMEでは50ステップ・no_accidentシナリオの定量結果（A/Bの差）が示されていない。サンプル結果の数値・グラフがあると、デモとしての説得力が増す。

### 一言コメント
「世界に物を置く」だけで人の振る舞いがどう変わるかというナッジ理論を、行動指示なしのLLMエージェントで観測しようという思想が一貫しており、ハッカソン提出物として完成度・拡張性ともに非常に高い。コード分量と層の混在は今後の整理余地として残るが、設計上の創発ポテンシャルと社会科学的意義はトップクラス。
