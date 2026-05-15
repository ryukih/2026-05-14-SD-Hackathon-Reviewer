# 評価レポート: hackathon_

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

「資本主義に代わる社会モデルのうち、どれが最も人間の幸福（Human Flourishing）を生み出すか」を問う、LLM マルチエージェントによる社会シミュレーション（鎌倉プロジェクト）。舞台は鎌倉のビーチ・寺・アトリエ・森・カフェの 5 場所で、固有の背景・価値観・年齢を持つ 6 人のエージェント（Kenji, Yui, Arjun, Sofia, Ren, Hana）が生活する。11 種類の価値基準（自律分散、遊び、精神性の成熟、ケアエコノミー、日本的価値観、資本主義など）を「世界のルール」として個別に設定し、それぞれで 45 日間のシミュレーションを実行、qwen2.5:7b（Ollama）が各エージェントの 1 日 3 フェーズ（メッセージ決定 → 送信 → 行動決定＋幸福度自己申告）を駆動する。Human Flourishing Index（HFI）として mood / happiness / authenticity / meaning / autonomy / vitality の 6 指標を自己申告 → 平均化し、PERMA・自己決定理論に基づく計測を行う。追加実験として、3 回の危機注入（HFI が逆に +0.279pt 上昇）、資本主義→自律分散の制度切り替え（HFI +7.0%）、Gini 係数による不平等計測、ケアエコノミーのネットワーク密度分析などが行われる。テーマは、同じ人間でも「世界のルール」次第で幸福度が変わることを LLM エージェントで実証し、社会制度設計の選好を定量的に比較することにある。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  9   |  9   |  -   |    9.0    |
| B. 世界設定  |  9   |  9   |  -   |    9.0    |
| C. 発展性    |  8   |  8   |  -   |    8.0    |
| D. 技術実装  |  8   |  8   |  -   |    8.0    |
| **合計**    |  34  |  34  |  -   |  **34.0** |

---

## 評価詳細

### A. 創発設計 — 最終: 9.0/10

#### 評価
世界ルールの設計精度・行動の自由度の両軸ともに高水準で両立している。`world.value_system` と `world.baseline` で世界の原則（資本主義／自律分散／ケア経済 等の11価値基準）が宣言文として明確に定義され、エージェントへ渡される情報は座標・近接者・占有率といった「生データ」と、自身の背景・過去メモリのみ。LLMには「どう行動すべきか」の指示が含まれず、戦略（移動・発話・関係更新）は完全にエージェント任せ。複数価値基準を取り替えて比較する設計は、ルールが行動を直接決めず、状況が行動を呼び出すという創発条件をよく満たしている。

#### 根拠
- `worlds_kamakura/config_kamakura_05_autonomy.yaml` — value_system が "no central authority... voluntary association... trust is the only currency that scales" のような原理だけを記述（行動指示なし）
- `worlds_kamakura/config_kamakura_01_capitalism.yaml` — capitalism のルールも "value is created through exchange... competition is natural" のような原理宣言
- `agent.py:644-657` — `place_section_text` は数値情報（agents_in_place / capacity / occupancy_rate）のみを渡しており、行動指示は含まれていない
- `agent.py:743-754` — 行動プロンプトでは「stay / move(direction)」とJSON出力形式のみ提示し、何をすべきかは指示していない
- `agent.py:565-624` — `_build_fire_section` の crisis 説明も状況描写（"How will you respond — with conflict or cooperation?"）にとどまり、選択は委ねられている
- `README.md` — "エージェントに与えられるのは『世界のルール』のみ。自律的な意思決定から生まれる社会を観測する。"

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
「資本主義に代わる11の価値基準のうち、人間の幸福（Human Flourishing）を最大化するのはどれか」という、社会哲学・経済人類学的に意義のあるリサーチクエスチョンを正面から扱っている。実在する鎌倉の5空間（由比ヶ浜・海蔵寺・小町裏アトリエ・獅子舞の谷・縁側カフェ）を舞台に、6人の濃密な個人史を持つペルソナを配置し、PERMA／自己決定理論に基づいた6指標（HFI）を毎日自己申告させる構造。Gini係数による不平等計測、ネットワーク密度・クラスタ係数、危機注入実験、価値体系の途中切替実験など、社会科学的分析軸も豊富。標準的なデモには見られない高い独自性と現実への接続性を備える。

#### 根拠
- `README.md` — "資本主義に代わる社会モデルのうち、どれが最も人間の幸福（Human Flourishing）を生み出すか？"
- `config_kamakura.yaml:30-92` — Kenji（元エンジニア42歳）、Yui（津波経験のアーティスト）、Arjun（コミュニティ開発研究者）、Sofia（元NGO代表の陶芸家）、Ren（バーンアウト後のフリーランス）、Hana（シングルマザー）など、リアルかつ多様な個人史
- `worlds_kamakura/` 配下の11価値体系（capitalism, gift_economy, commons, autonomy, care_economy, mutual_aid, authenticity, inner_maturity, play, japanese_values, emergence）
- `worlds_crisis/config_crisis_autonomy.yaml` — power_struggle / outside_threat / resource_crisis を時系列で注入する危機実験
- `worlds_crisis/config_transition_cap_to_auto.yaml:transitions` — Day16 で資本主義→自律分散へ制度転換する自然実験
- `agent.py:707-720` — wellbeing 自己申告は mood/happiness/authenticity/meaning/autonomy/vitality + created / story_shared を含み、PERMA に整合

---

### C. 発展性 — 最終: 8.0/10

#### コード拡張性
責務分離が明確で、`agent.py`（LLM意思決定）、`simulation.py`（世界進行）、`llm_factory.py` / `ollama_client.py` / `cli_client.py`（LLM抽象化）、`visualization.py`（描画）、`utils.py`（座標・場所判定）に分かれている。世界・場所・ペルソナ・LLMがすべて YAML 外部化されており、新しい価値体系・新しい都市・新しい危機イベントを追加するのにコア変更を要しない。実際に `generate_world_configs.py` / `generate_kamakura_top_configs.py` で16+の世界設定を自動生成しており、設計上の拡張性が運用で実証されている。LLM プロバイダーが Ollama と任意 CLI（Claude／Codex）に切替可能なのも将来性が高い。

#### 将来展望
README に「教育環境」「組織設計」「地域コミュニティ（鎌倉パイロット）」の3応用領域が示されているが、各論の踏み込みは概念レベルにとどまる。「次フェーズ：『どんな環境設計が自律性を育てるか』を検証する」という方向性は具体的だが、研究設計の詳細（仮説・指標・実験条件）までは記述されていない。

#### 根拠
- `simulation.py:11` — `from agent import Agent`、`from llm_factory import create_llm_client` などモジュール分離が明確
- `llm_factory.py:12-50` — Ollama / CLI provider の抽象化。新規プロバイダ追加は1関数追加で済む
- `generate_kamakura_top_configs.py`、`generate_world_configs.py` — 設定ファイル自動生成スクリプトの存在
- `worlds_kamakura/`, `worlds_crisis/`, `worlds_kamakura_top/` — 同一エンジン上で多数の実験設定を運用
- `README.md` — "次フェーズ：「どんな環境設計が自律性を育てるか」を検証するシミュレーションを設計予定。"

---

### D. 技術実装 — 最終: 8.0/10

#### LLM利用
プロンプト設計は丁寧で、メッセージ決定と行動決定を別フェーズに分離した二段プロンプト構成。背景／世界ルール／ベースライン／パーソナリティ／成長（evolved_perspective）／関係性／場所情報を構造化セクションとして組み立てる。出力は厳密な JSON スキーマ（`agent.py:743-754`、`agent.py:707-720`）を要求し、`_extract_json_from_text`（agent.py:914-947）でブレース深度を追って堅牢に抽出、失敗時は文字列フォールバックで方向語抽出を行う。LLM 例外を `try/except` で包み、エラー時はデフォルト挙動（stay）に戻す。Ollama 接続チェック・モデル存在チェックも実装。Temperature/min_p/repeat_penalty が config から制御可能。

#### メモリ設計
3層構造で実装されている：
1. ローリング journal（`self.memory`、最大 `memory_limit=60`、プロンプトには直近 `memory_size=7-10` を投入）
2. 受信メッセージ履歴（`message_history_limit=15`、文脈 `message_context_size=7`）
3. 10日ごとに過去ジャーナルを LLM に再要約させる成長振り返り（`_reflect_on_growth`、agent.py:1149-1175）— `evolved_perspective` として次回プロンプトに継続注入される

これに加え `perspective_shift` と `day_reflection` を毎日記録、Turning Point（HFI±1.5）の自動検出、関係性スコア（-5〜+5）の永続管理など、単なるログダンプを超えた意味のあるメモリ設計になっている。

#### コード品質・シミュレーション正確性
`agent.py` は約1200行と大きめだが、`TypedDict` による型定義、明確な命名、責務ごとのプライベートメソッド分割で可読性は高い。シミュレーションは「全員のメッセージ決定 → 配信 → 全員の行動決定 → 移動」という同期 4 フェーズで race condition を避けている（simulation.py:474-595）。出力は JSONL で構造化され（messages / wellbeing / turning_points / memory_reasoning / relationship_changes / sq_level_changes）、再現性に必要な情報は十分。やや弱い点としては乱数 seed の明示固定がない、LLM 出力のリトライ／レート制御は実装されていない。

#### 根拠
- `agent.py:914-947` — `_extract_json_from_text` のブレース深度トラッキングによる堅牢なJSON抽出
- `agent.py:1010-1042` — `decide_action` での LLM 呼び出し＋例外ハンドリング＋メモリ更新フロー
- `agent.py:1149-1175` — `_reflect_on_growth`（10日ごとの成長要約）
- `simulation.py:474-595` — 4フェーズ同期実行（message decision → send → action decision → move）
- `simulation.py:595-615` — Turning Point 検出（HFI ±1.5 で記録）
- `ollama_client.py:46-90` — temperature / min_p / repeat_penalty / max_tokens を config 制御
- `llm_factory.py` — Ollama / CLI 切替可能なクライアントファクトリ
- `agent.py:298-303` — wellbeing は 1-10 にクランプされ、欠損時は 5 を補完
- `agent.py:316-330` — memory_limit による rolling buffer 管理

---

## 総評

### 優れている点
- 「11の異なる価値体系を同一エンジンで比較」という社会哲学的に野心的な問いを、PERMA／SDTに基づく定量指標（HFI）、Gini係数、ネットワーク分析、危機注入実験、制度転換実験まで含めて多面的に評価しており、ハッカソン作品の枠を超えた研究プロジェクトの体をなしている。
- 世界ルールの設計精度と行動自由度の両立が非常にうまい。エージェントは原則と生データだけを受け取り、行動はLLMに委ねられているため、設計上の創発ポテンシャルが高い。
- メモリ設計が単なるログ蓄積ではなく、「10日ごとの自己振り返りによる視点変容（evolved_perspective）」「Turning Point検出」「関係性スコアの双方向管理」など、長期的な人格進化を捉える工夫がなされている。
- コード／設定の分離が徹底しており、新世界の追加・LLMプロバイダ切替が容易。実際に16+世界を運用している実績がある。

### 改善点・提言
- 乱数 seed の明示的固定（`random.seed` / `numpy.random.seed`）が見当たらず、厳密な再現性検証には不利。config に `seed` キーを追加すると研究としての価値がさらに高まる。
- LLM 呼び出しのリトライ／タイムアウト戦略がやや素朴。N=6 × 30〜45日 × 2呼び出しの規模で1つでも失敗すれば後段の分析にバイアスが入りうるため、`tenacity` 等での指数バックオフ＋失敗ログ集計を推奨。
- 「価値体系の優劣」が qwen2.5:7b の事前学習バイアスにどの程度引きずられているかの感度分析（例：他モデルでの再現、LLMバイアス除去策の議論）があると、結論の説得力が上がる。
- 将来展望（教育・組織・コミュニティへの応用）は方向は良いが、各論の研究設計（仮説・介入・指標）まで具体化するとさらに高評価。

### 一言コメント
LLMマルチエージェントを「社会哲学のシミュレーション実験装置」として真摯に運用した非常に完成度の高い作品。創発設計・世界設定・実装のいずれも高水準で、ハッカソン作品としては突出している。
