# 評価検証レポート: hackathon_

**検証日**: 2026-05-11
**対象**: `hackathon__eval.md`
**元評価合計**: 34.0/40

---

## 検証サマリー

| カテゴリ | 元スコア | 検証結果 | 妥当性 | コメント |
|---|---|---|---|---|
| A. 創発設計 | 9.0 | 引用ほぼ正確（行番号に軽微なずれ） | 妥当 | `value_system` / `baseline` / 生データのみ提供は実装で確認 |
| B. 世界設定 | 9.0 | 引用正確 | 妥当 | 11価値体系・6ペルソナ・PERMA/SDT指標を実装で確認 |
| C. 発展性 | 8.0 | 引用正確 | 妥当 | LLM抽象化、設定外部化、複数世界運用を確認 |
| D. 技術実装 | 8.0 | 引用は内容としては正しいが行番号が25〜750行ずれている | 概ね妥当 | 機能の事実関係は全て確認、軽微な引用ずれのみ |
| **合計** | **34.0** | — | **妥当** | スコア水準は概ね適正、引用精度に改善余地 |

**総合判定**: 元評価は **妥当** と判定する。スコア、論評の方向性、優れている点・改善点の指摘いずれも実装と整合しており、誇張は確認されない。減点要因として指摘した「乱数seed未固定」「リトライ未実装」も実装で確認できた。ただし D の根拠における行番号引用はおおむね 25〜750 行ずれており、検証者は本文内容で照合する必要があった。

---

## カテゴリ別検証

### A. 創発設計 — スコア妥当性: 妥当（9.0）

#### 根拠の検証

- ✓ `worlds_kamakura/config_kamakura_05_autonomy.yaml` の `value_system` に "no central authority... voluntary association... Trust is the only currency that scales" を直接確認（line 6-18）。行動指示なしの原理宣言文として実装されている。
- ✓ `worlds_kamakura/config_kamakura_01_capitalism.yaml` の `value_system` に "Value is created through exchange... Competition is natural and productive..." を確認（line 6-15）。
- ⚠ `agent.py:644-657` で `place_section_text`（数値情報のみ）の主張: 実際の該当コードは line 676-688（`create_message_prompt` 内）および line 768-779（`create_decision_prompt` 内）にあり、行番号は約30行のずれ。ただし内容は正確（`agents_in_place` / `capacity` / `occupancy_rate` のみを文字列化、行動指示なし）。
- ⚠ `agent.py:743-754` の "stay / move(direction) と JSON出力形式のみ提示" の主張: 実際の `create_decision_prompt` は line 737 開始、JSON出力スキーマは line 866-885 付近。約120行のずれだが、内容は正確（"stay / move with direction" のみ提示、戦略指示なし）。
- ⚠ `agent.py:565-624` の `_build_fire_section` の主張: 実際は line 568 開始、内容は正確（"How will you respond — with conflict or cooperation?" のような状況描写のみで強制なし、line 580 で確認）。
- ✓ README の "エージェントに与えられるのは『世界のルール』のみ" を確認（line 5）。

#### 見落とし・誇張

- 軽微な誇張なし。エージェントへ渡される情報が「原理宣言＋生データ＋自身の背景・記憶のみ」という記述は実装と完全に整合する。
- 見落とし: `_build_fire_section` には危機イベントだけでなく `creative_challenge` / `philosophical_prompt` / `governance_debate` のような肯定的イベントも実装されており、創発実験の幅は評価より広い（line 605-625 付近）。これは評価を更に支える要素。

#### 検証結論

スコア 9.0/10 は妥当。世界ルールと行動自由度の両立は実コードで確認でき、原理宣言＋自己決定の設計は創発条件をよく満たす。行番号の引用ずれは内容判断に影響しない範囲。

---

### B. 世界設定 — スコア妥当性: 妥当（9.0）

#### 根拠の検証

- ✓ README line 31 の "資本主義に代わる社会モデルのうち、どれが最も人間の幸福（Human Flourishing）を生み出すか？" を確認。
- ✓ 6ペルソナ全員（Kenji / Yui / Arjun / Sofia / Ren / Hana）を `worlds_kamakura/config_kamakura_05_autonomy.yaml` line 38-160 範囲で確認。背景は eval の要約（津波経験のアーティスト、元NGO代表の陶芸家、シングルマザー等）と一致。
- ⚠ Kenji の年齢: eval は「元エンジニア42歳」と記述。`config_kamakura.yaml` 系列では 42 歳、`worlds_kamakura/config_kamakura_05_autonomy.yaml` line 40 では 44 歳。Run 系列により年齢が異なる軽微な不一致だが、評価の本質には影響しない。
- ✓ `worlds_kamakura/` 配下に 11 価値体系の YAML を確認（capitalism / gift_economy / commons / autonomy / care_economy / mutual_aid / authenticity / inner_maturity / play / japanese_values / emergence）。`ls` で 11 ファイルを確認済み。
- ✓ `worlds_crisis/config_crisis_autonomy.yaml` line 202-215 で power_struggle / outside_threat / resource_crisis の3危機注入を確認。
- ✓ `worlds_crisis/config_transition_cap_to_auto.yaml` line 202-205 で `transitions: - step: 16` を確認。Day16 制度転換実験は実装されている。
- ⚠ `agent.py:707-720` で wellbeing 自己申告（mood/happiness/authenticity/meaning/autonomy/vitality + created/story_shared）の主張: 実装は line 988-998（`parse_action_response` の wellbeing 抽出部）にある。約280行のずれだが、内容は正確（6指標 + created / story_shared を確認）。
- ✓ Gini 係数・ネットワーク密度の分析スクリプトは `analyze_inequality.py`（10,078 bytes） / `analyze_network.py`（11,272 bytes）として実存。

#### 見落とし・誇張

- 見落とし: HFI の clamp 処理（1-10）が `parse_action_response` line 994 にあり、評価でも触れているが「6指標を毎日自己申告」の社会科学的厳密さを補強する重要実装。
- 軽微な誇張なし。Gini / ネットワーク密度 / 危機注入 / 制度転換が全て実装で裏付けられている。

#### 検証結論

スコア 9.0/10 は妥当。社会哲学的な問いの深さ、ペルソナの個人史、PERMA/SDT に基づく定量指標、複数の実験設計が全て実装で確認でき、独自性・現実接続性の主張は誇張なしに成立する。

---

### C. 発展性 — スコア妥当性: 妥当（8.0）

#### 根拠の検証

- ✓ `simulation.py` line 11 で `from agent import Agent, SQ_LEVEL_BACKGROUNDS...`、line 12 で `from llm_factory import create_llm_client`、line 13 で `from utils import ...` を確認。責務分離の主張は実装と整合。
- ✓ `llm_factory.py` line 12-51 で `create_llm_client` を確認。ollama / command（CLI）の2プロバイダ抽象化を実装。新規プロバイダ追加は1分岐追加で済む設計。
- ✓ `generate_kamakura_top_configs.py`（13,719 bytes）、`generate_world_configs.py`（23,135 bytes）、`generate_kamakura_configs.py`（17,120 bytes）の3つの config 自動生成スクリプトが実存。
- ✓ `worlds_kamakura/`（11ファイル）、`worlds_kamakura_top/`（6ファイル）、`worlds_crisis/`（2ファイル）、`worlds/`（複数）の合計16+世界設定が実存。eval の「16+世界を運用」は妥当。
- ✓ README line 187-192 の "次フェーズ：『どんな環境設計が自律性を育てるか』を検証する" を確認。
- ✓ `cli_client.py`（5,875 bytes）と `codex_wrapper.py`（1,988 bytes）も実存し、Claude/Codex 切替の実装が裏付けられている。

#### 見落とし・誇張

- 見落とし: `output_world_*`、`output_kamakura_*`、`output_transition_*` 等のディレクトリが30+存在し、運用実績は評価以上に充実している。
- 見落とし: `paper/` ディレクトリ、`reports/` ディレクトリ、`analysis_kamakura/` ディレクトリの存在は研究プロジェクト性をさらに補強する。
- 「将来展望は概念レベルにとどまる」の指摘は妥当。README の「教育環境」「組織設計」「地域コミュニティ」記述は1-2行ずつで、各論の仮説・指標・介入条件には踏み込まれていない。

#### 検証結論

スコア 8.0/10 は妥当。コード拡張性は高水準だが、将来展望の具体性が概念レベルにとどまる点が 9/10 に届かない理由として整合する。

---

### D. 技術実装 — スコア妥当性: 妥当（8.0）

#### 根拠の検証

- ⚠ `agent.py:914-947` の `_extract_json_from_text`（ブレース深度トラッキング）の主張: 実装は line 891-927（in_string / escape_next フラグ + depth カウンタ）。約25行のずれだが、ブレース深度＋エスケープ追跡の堅牢な実装は確認できた。
- ⚠ `agent.py:1010-1042` の `decide_action` 主張: 実装は line 1051-1095。約40行のずれ。LLM呼び出し＋try/except＋メモリ更新＋perspective_shift 累積のフローは正確（line 1064-1095）。
- ✓ `agent.py:1149-1175` の `_reflect_on_growth` 主張: 実装は line 1150-1178（ほぼ一致、+3行のずれのみ）。10日ごとに過去ジャーナル要約を LLM に依頼し `evolved_perspective` を更新する実装を確認。
- ⚠ `simulation.py:474-595` の「4フェーズ同期実行」主張: 実装は line 480-595 にコメント `# Phase 1:`, `# Phase 2:`, `# Phase 3:`, `# Phase 3.5:`, `# Phase 4:` を確認。実際は 4 フェーズ + 中間 3.5 で計 5 段階だが、評価の「message decision → send → action decision → move」という同期構造の主張は正確。race condition 回避の主張も整合。
- ⚠ `simulation.py:595-615` の Turning Point 検出主張: 実装は line 549-581（`HFI_THRESHOLD = 1.5` で line 551、検出ロジックは line 565-580）。約45行のずれだが、HFI±1.5 検出は確認。
- ✓ `ollama_client.py:46-90` の temperature/min_p/repeat_penalty config 制御主張: 実装は line 44-92（`generate` メソッド、line 71-77 で options 構築）。ほぼ一致。
- ✓ `llm_factory.py` の Ollama/CLI 切替: 確認済み（line 12-51）。
- ⚠ `agent.py:298-303` の wellbeing 1-10 クランプ主張: 実装は line 988-996（`parse_action_response` の wellbeing 抽出、`max(1, min(10, int(...)))` を line 994 で確認）。約690行のずれだが、内容は正確。
- ⚠ `agent.py:316-330` の `memory_limit` rolling buffer 主張: 実装は line 459（属性定義）と line 1078-1079（pop 制御）。約750行のずれだが、内容は正確。

#### メモリ設計の3層構造主張

- ✓ Layer 1: ローリング journal（`self.memory`, `memory_limit=60`）— `worlds_kamakura_top/config_kamakura_top_*.yaml` で `memory_limit: 60` / `memory_size: 10` を確認。`worlds_kamakura/` 系では 30/7。eval の「60 / 7-10」表記は両系列を含む正確な記述。
- ✓ Layer 2: 受信メッセージ履歴（`message_history_limit=15`, `message_context_size=7`）— 同 YAML で確認。
- ✓ Layer 3: 10日ごとの成長振り返り（`_reflect_on_growth`）— `agent.py` line 1067-1068 で `if step > 0 and step % 10 == 0: self._reflect_on_growth(step)` を確認。

#### 見落とし・誇張

- 誇張なし。LLM 例外ハンドリング（try/except + デフォルト stay 復帰）は `decide_action` line 1093-1095 で確認。
- 改善点として挙げた「乱数 seed 未固定」は `grep` で `random.seed` / `np.random.seed` がコード全体に存在しないことを確認。指摘は妥当。
- 改善点「LLM リトライ／タイムアウト戦略がやや素朴」も妥当。`ollama_client.py` には `API_TIMEOUT = 180` のみで、失敗時は空文字列を返すだけのフォールバック（line 90-92）。指数バックオフ等は未実装。
- 見落とし: `_extract_direction_from_text`（line 928-942）は単純な "up/down/left/right" の含有チェックで、"downtown" 等の語に対する誤検出リスクがある点に言及があれば更に良い。ただし英語プロンプト設計と JSON 優先処理によりリスクは限定的。

#### 検証結論

スコア 8.0/10 は妥当。LLM 利用・メモリ設計・コード品質・同期処理いずれも事実関係は正確で、減点要素（seed未固定、リトライ未実装）も実装と整合する。引用行番号は最大 750 行近くずれているが内容判断は揺るがない。

---

## 総合所見

### 元評価の強み

- 11価値体系の比較という研究的野心、PERMA/SDT による定量指標、Gini/ネットワーク/危機/転換の多面分析を正しく捉え、ハッカソン作品の枠を超えるという論評は実装証拠と整合している。
- メモリ設計を「3層 + Turning Point + perspective_shift + 関係性スコア」と整理した記述は実装と完全に一致し、単なる機能列挙ではなく設計意図に踏み込んだ良質な評価。
- 改善提言（seed固定、リトライ、LLMバイアス感度分析、将来展望の具体化）はいずれも実装と一致した的確な指摘。

### 元評価の弱点

- **行番号引用の精度に問題あり**。特に D カテゴリでは引用された行番号と実装の実際の位置に 25〜750 行のずれがある（例: `agent.py:298-303` は実際 988-996、`agent.py:316-330` は実際 1078-1079）。内容自体は正確だが、検証者・第三者が該当箇所を直接参照する際の妨げになる。
- Kenji の年齢（42 vs 44）の Run 系列間差異への言及がない。複数 config 系列の存在は評価本文に書かれているが、ペルソナ詳細記述は1系列のみに依拠している。
- `_extract_direction_from_text` の単純文字列マッチによる潜在的誤検出（"downtown" を含む語の誤判定）への言及がない。コード品質の細部としては触れる余地あり。

### スコア妥当性

合計 34.0/40 は **妥当**。各カテゴリ単体でも誇張は見当たらず、改善点指摘も実装根拠に基づく。本作はハッカソン応募作の中で技術実装・社会科学的設計・運用実績のいずれにおいても高水準であり、9-9-8-8 という配点は適正と判断する。

### 推奨アクション

- 行番号引用の精度向上（特に D セクション）。再評価時は `grep -n` 等で位置確認を推奨。
- 11価値体系のうち実際に Run3（45日 + HFI計測）まで完走したのは上位5世界に絞られている（`worlds_kamakura_top/` は6設定）ため、評価本文で「11世界比較」と「上位5世界深掘り」の運用フェーズの違いを明示するとさらに正確になる。
