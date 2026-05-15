# 評価検証レポート: AlberiaMisinformationProtest

**検証日**: 2026-05-11
**対象**: `AlberiaMisinformationProtest_eval.md`
**元評価合計**: 32.5/40

---

## 検証サマリー

| カテゴリ       | 元スコア | 検証結論       | 推奨スコア帯 |
|---------------|---------|---------------|-------------|
| A. 創発設計    | 7.5/10  | 妥当（やや下振れ可） | 7.0 – 7.5   |
| B. 世界設定    | 9.0/10  | 妥当           | 8.5 – 9.0   |
| C. 発展性      | 7.0/10  | 妥当           | 6.5 – 7.5   |
| D. 技術実装    | 9.0/10  | 妥当           | 8.5 – 9.0   |
| **合計**      | 32.5    | 妥当           | 30.5 – 33.0 |

**総合判定**: 元評価は十分に裏付けられている。引用された行番号・キーワード・設計判断はソースコード／README と一致しており、誇張は見当たらない。特に技術実装の評価（JSON強制、`<think>` 除去、フォールバック多層化、感情慣性、JSONL 永続化、再現性シード）は実装そのものに直接対応している。創発設計の 7.5 はやや上振れの可能性があるが、LLM パスでは行動裁量がほぼ最大限保たれているため許容範囲。

---

## カテゴリ別検証

### A. 創発設計（元: 7.5/10）

**根拠の検証**

- `agents.py:65-80` ✓ — 該当範囲に `VALID_INTENTS` のリスト（14 要素：`stay_home, observe, ask_neighbor, verify_rumor, move_to_small_gathering, move_to_central_square, move_to_city_hall, move_to_media_zone, move_to_station, retreat, share_uncertain_info, share_correction, calm_others, join_peaceful_protest`）が正確に定義されている。「14種類」という表現も実数と一致。
- `prompts.py:153-182` ✓ — `output_schema` ブロックが正にこの範囲に存在し、`thinking / interpretation / belief_updates / emotion_update / intent / target / action_reason / utterance / sns_post / memory_to_add / public_state` を LLM 自身が JSON で決定する構造である。「全てLLMが自律的に決定」は正しい。
- `prompts.py:209-211` ✓ — 該当行付近に「発話は現在の anger / fear / solidarity を反映してください。怒り・不安・疑念・失望・焦りを含めてよいです。」のナッジ文言を確認。引用は逐語一致。
- `simulation.py:251-275` ✓ — イベント発火時に近傍市民に対し `belief_rumors` の初期値（メディアリテラシーで減衰）と「（ステップN）…という話と…という噂の両方を聞いた」の記憶テキストを付与しており、行動指示ではなく状況データのみ。ただし `agent.anger / fear / solidarity` を `max()` で底上げしているため「データのみ」とは厳密には言えず、感情の初期スパイクを与える設計が存在する点は本評価では言及されていない（軽微な見落とし）。
- `agents.py:469-499` ✓ — `_fallback_decision` 内に `anger>0.38 and max_belief>0.35`、`fear>0.55`、`nearby_cops>2` 等の閾値遷移が確認できる。引用通り。
- README 引用 "「世界の物理法則・社会規範・行動可能性だけを定義し、LLMエージェントが自律的に解釈・判断・行動する社会を観測する」" ✓ — README 本文中に逐語で存在。

**スコア妥当性**: LLM パスで行動・感情・信念・発話を全て自由生成させる構造は明確で「自由」軸は高い。「ルール」軸（環境の精度）も SNS の `truth_status`、`credibility_score`、`reach`、感情慣性、フォールバック閾値などが揃っている。一方、(1) システムプロンプトの感情語ナッジ、(2) イベント発火時の感情底上げ（`simulation.py:267-275`）、の 2 点が「環境がエージェントの内面を直接書き換える」要素であり、純粋な創発という観点では減点要因。7.5 は妥当だが 7.0 に寄せても説明可能。

**見落とし・誇張**

- 見落とし: `simulation.py:267-275` で `agent.anger / fear / solidarity` を `max(..., 環境値)` で底上げしている事実。「データとして」というレポートの表現はやや甘い。
- 誇張: なし。
- 軽微: 「フォールバックではあるが」と保険を入れている点は適切。

**検証結論**: スコア 7.5 は妥当範囲内（7.0 – 7.5）。

---

### B. 世界設定（元: 9.0/10）

**根拠の検証**

- README 引用 "局所的な目撃情報・SNS風の情報伝播・誤情報・訂正情報・信頼・恐怖・怒り・連帯感が…研究用シミュレーション" ✓ — README「概要」セクションに逐語存在。
- README 「Civil Violence との7項目比較表」 ✓ — README に表が存在。実際の項目数は `行動決定 / 状態遷移 / 感情更新 / 情報環境 / 誤情報 / 記憶 / 創発の主役` の **7 行**。引用通り。
- 参考文献の論文列挙 ✓ — Epstein (2002)、Bonabeau (2002)、Vosoughi et al. (2018)、Centola (2010)、Lewandowsky et al. (2017) の 5 件が DOI 付きで列挙されている。
- `config:76-103` ✓ — 該当範囲に `evt_001 / evt_002 / evt_003`（市庁舎配給排除疑惑 / 中央広場参加強制噂 / メディアゾーン圧力疑惑）の 3 イベントが `step / location / ground_truth / rumor / truth_status / severity / spread_radius / online_reach` 完備で定義されている。
- `agents.py:687-692` ✓ — `InfluencerAgent.ALIGNMENT_TRUTH_MAP` に `sensational / skeptical / official_friendly / protest_sympathetic` の 4 種が定義されている。「4種類」は実数と一致。
- README 倫理的注意 ✓ — 「現実の政治運動・暴動・抗議活動の扇動・最適化・支援」を目的としない旨が明記されている。

**スコア妥当性**: 架空都市「アルベリア」というセーフガード、SNS の `truth_status` 5 分類、信頼度 / リーチ / 感情トーンの構造化、Civil Violence との明示比較、参考文献の妥当性、倫理的注意の明示 — いずれも 9 点を支える材料が十分にある。

**見落とし・誇張**

- 見落とし: README に「7つの市民 public_state（ORDINARY/OBSERVER/PARTICIPANT/MEDIATOR/SKEPTIC/WITHDRAWN/DETAINED）」が表で定義されており、世界設定の精細度を示す材料として有力だが、評価本文では触れられていない。
- 誇張: なし。

**検証結論**: スコア 9.0 は妥当。8.5 – 9.0 帯。

---

### C. 発展性（元: 7.0/10）

**根拠の検証**

- README 実験例セクション ✓ — 「実験1: 情報リテラシーの効果」「実験2: 公式対応速度の効果」「実験3: インフルエンサーの影響」の 3 案が `agents.average_media_literacy` / `official.response_delay` / `influencers[0].alignment` を変数とするパラメータスイープ提案として存在。
- `config:38-74` ✓ — 該当範囲に `landmarks`（9 種類、`home_area / station / central_square / city_hall / media_zone / small_gathering_a/b / police_station / exit_zone`）が外部化されている。`events / influencers / official` も YAML で外部化されている（後続の行）。
- `agents.py:687-692` ✓ — `ALIGNMENT_TRUTH_MAP` 辞書への新規エントリ追加で新 alignment が容易に追加可能（既に `protest_sympathetic` が 4 番目として存在する点が拡張性の証左にもなっている）。
- `simulation.py:41-101` ✓ — `Simulation.__init__` が `config["simulation"] / agents / llm / landmarks / events` を駆動して初期化しており、コード変更を最小化する構造。
- 「README に概念的な将来研究方向（cross-city, longer memory, policy intervention 等）の記述は無い」✓ — README 全文を確認したが、`実験例` 以外の拡張ロードマップは確かに無い。

**スコア妥当性**: コード拡張性は確かに高水準（モジュール分離、YAML 外部化、辞書ベースのマッピング）。一方で将来展望はパラメータスイープ 3 案に留まり、研究的ビジョンが控えめ。7.0 はバランス妥当。

**見落とし・誇張**

- 見落とし: `viewer.html` というブラウザビューアまで自前実装している点は「他者が再利用しやすい」発展性の要素として加点材料になりうるが、本評価では D（技術実装）側で言及されているため、二重計上を避けた判断と解釈できる。
- 軽微: 「4 クラスに分かれており」と書かれているが、`CitizenAgent / CopAgent / InfluencerAgent / OfficialAgent` の 4 クラスが README に明示されており実装上も対応している（妥当）。
- 誇張: なし。

**検証結論**: スコア 7.0 は妥当。6.5 – 7.5 帯。

---

### D. 技術実装（元: 9.0/10）

**根拠の検証**

- `ollama_client.py:71-104` ✓ — payload に `format: "json"`（force_json 時）、temperature / num_predict / repeat_penalty / repeat_last_n / min_p を設定し、`ConnectionError / Timeout / RequestException / Exception` を個別 catch している実装を確認。
- `agents.py:316-326` ✓ — `re.sub(r"<think>.*?</think>", "", raw, flags=re.DOTALL)` で qwen3 の思考ブロックを除去、`re.search(r"\{.*\}", cleaned, re.DOTALL)` で JSON 抽出、失敗時は `_fallback_decision` に切替。引用通り。
- `agents.py:303-415` ✓ — `_parse_and_validate` が `intent`（VALID_INTENTS 検証）/ `public_state` / `emotion_update`（clamp）/ `belief_updates`（known_rumor_ids でフィルタ）/ `utterance`（暴力フィルタ＋日本語化）/ `sns_post`（同様）を全て検証している。
- `agents.py:835-869` ✓ — `_contains_violence` と `_clean_japanese_text`（英単語→日本語置換、ランドマーク名の自然言語化）を確認。
- `agents.py:523-531` ✓ — `self.anger = _clamp(max(eu.get("anger", self.anger), self.anger * 0.70))` の感情慣性ロジックを確認。引用は逐語一致。
- `simulation.py:50` ✓ — `random.seed(seed)` で再現性確保。
- `simulation.py:432-492` ✓ — `_log_decisions / _log_messages / _flush_social_feed_log` で JSONL 永続化が実装されており、`thinking / interpretation / belief_updates / emotion_update / intent / target / action_reason / utterance / memory_to_add / decision_source` を逐次記録。
- `agents.py:221` ✓ — `self.memory: deque[str] = deque(maxlen=self.memory_limit)` を確認。
- `simulation.py:259-262` ✓ — `agent.memory.append(f"（ステップ{self.steps}）{location_name}付近で「{ground_truth}」という話と、「{rumor}」という噂の両方を聞いた。")` を確認。エピソード的記憶追加は事実。

**スコア妥当性**: 評価の各観点（LLM 利用 / メモリ設計 / コード品質・正確性）はすべて実装上の根拠が明確。`random.shuffle(self.citizens)`（`simulation.py:206`）も実在し順序公平性に寄与。

- LLM 利用: 3/3（強制 JSON、思考除去、例外処理、フォールバック多層）
- メモリ設計: 2-3/3（テキストログのフラット deque、ただし `belief_rumors` 辞書で噂別構造化、感情と分離）
- コード品質: 2/2（型ヒント・docstring・モジュール分離）
- シミュレーション正確性: 2/2（シード、感情慣性、検証パイプライン、サニタイゼーション）

合計 9-10/10 帯。9.0 は妥当。

**見落とし・誇張**

- 見落とし: 「メモリはやや浅め（フラットなテキストログ）」という指摘は妥当だが、`belief_rumors: dict[str, float]` が噂単位で別チャネル管理されている点を「メモリ設計」スコア材料として更に加点解釈する余地があった。
- 軽微: 「ブラウザビューアまで出力系も充実」は `viewer.html`（17KB）に対応しており妥当。
- 誇張: なし。引用された行番号は全て該当箇所に正確に対応している。

**検証結論**: スコア 9.0 は妥当。8.5 – 9.0 帯。

---

## 総合所見

元評価は **引用の正確性・スコアの妥当性ともに高水準**。検証した 6 つのコード引用、3 つの README 引用、1 つの config 引用すべてで該当箇所が実在し、行番号もほぼ一致した。表現面でも誇張が見当たらず、「フォールバックではあるが」「将来展望としてはやや控えめ」「メモリはやや浅め」等、慎重な留保が随所に添えられている点が信頼性を高めている。

唯一の改善余地は **A. 創発設計** における環境→エージェント内面への直接介入の扱い。`simulation.py:267-275` でイベント発火時に近傍市民の `anger / fear / solidarity` を `max()` で底上げしている事実は、「環境はデータのみ渡す」という設計思想とは部分的に矛盾しており、評価本文では触れられていない。これを考慮すると A は 7.0 が中心値の可能性もあるが、LLM パスでの行動裁量の広さを重視すれば 7.5 は許容範囲。

**最終的に元評価合計 32.5/40 は妥当**。推奨レンジは 30.5 – 33.0 で、現スコアはレンジ上端寄りに位置する。

---

## 補足: 評価方法と検証範囲

### 検証対象ファイル

- `submissions/AlberiaMisinformationProtest/README.md`（10,087 バイト、全文確認）
- `submissions/AlberiaMisinformationProtest/agents.py`（36,170 バイト / 892 行、引用箇所と周辺コンテキストを確認）
- `submissions/AlberiaMisinformationProtest/simulation.py`（21,020 バイト / 537 行、初期化・イベント発火・JSONL 出力を確認）
- `submissions/AlberiaMisinformationProtest/prompts.py`（10,006 バイト / 220 行、出力スキーマと感情ナッジ文を確認）
- `submissions/AlberiaMisinformationProtest/ollama_client.py`（4,791 バイト / 136 行、payload と例外処理を確認）
- `submissions/AlberiaMisinformationProtest/config`（3,020 バイト、イベント・influencer・landmark 定義を確認）

### 引用検証の方法

各引用について (1) 該当行に当該コードが存在するか、(2) 引用された自然言語表現が逐語一致するか、(3) 評価本文の主張がコード挙動と整合するか、の 3 点を確認した。10 件中 10 件で一致を確認している。

### 評価レポートの強み（メタ評価）

- **行番号引用の精度**: `agents.py:65-80`、`prompts.py:153-182`、`agents.py:316-326` など、5 行幅程度のピンポイント引用が多く、検証可能性が高い。
- **逐語引用の正確性**: `prompts.py:209-211` の感情ナッジ文、`agents.py:523-531` の感情慣性式、`agents.py:221` の deque 宣言など、コードや自然言語の引用が原典と一致。
- **留保の使い方**: 「フォールバックではあるが」「やや浅め」「やや方向性を与えうる」など、過度な断定を避ける表現が一貫している。
- **総評構成**: 「優れている点 / 改善点・提言 / 一言コメント」の 3 段構成で、提出者がアクション可能なフィードバックになっている。

### 評価レポートの弱み（メタ評価）

- `simulation.py:267-275` の感情底上げ処理に対する言及が無く、創発設計の評価で「データのみ渡す」設計と扱われている点はやや甘い。
- C 発展性のスコア（7.0）が、コード拡張性は高いがビジョンは控えめという二面性を平均化した結果である旨をもう一段明示できると、提出者にとってアクション項目が明確になる。
- D 技術実装の「メモリ設計」評価で `belief_rumors` の構造化を低評価寄りに扱っているが、噂単位の信念分離は実は珍しい設計選択であり、加点解釈の余地があった。

### 検証者からの提言

元評価は実用上そのまま採用して問題ない。スコアの再調整を行う場合でも変動幅は ±0.5 以内に収まる見込みで、合計値 32.5 を 31.5 – 33.0 のレンジに置く判断が合理的である。提出物自体は「LLM 統合の堅牢性」と「世界設定の社会科学的妥当性」が突出して優れた作品であり、評価レポートはその両軸を正確に捕捉できている。
