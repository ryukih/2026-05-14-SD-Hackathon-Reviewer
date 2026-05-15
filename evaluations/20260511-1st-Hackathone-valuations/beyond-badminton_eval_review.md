# 評価検証レポート: beyond-badminton

**検証日**: 2026-05-11
**対象**: `beyond-badminton_eval.md`
**元評価合計**: 28.5/40

---

## 検証サマリー

| カテゴリ | 元スコア | 根拠正確性 | スコア妥当性 | 検証判定 |
|---|---|---|---|---|
| A. 創発設計（ルール×自由度） | 6.5/10 | ✓ 全引用一致 | 妥当 | 維持 |
| B. 世界設定（独自性） | 8.0/10 | ✓ 全引用一致 | 妥当 | 維持 |
| C. 発展性（拡張性+将来展望） | 6.5/10 | ✓ 全引用一致 | 妥当 | 維持 |
| D. 技術実装（LLM+Mem+Code+Sim） | 7.5/10 | ✓ 全引用一致 | 妥当 | 維持 |
| **合計** | **28.5/40** | — | — | **妥当** |

**総合判定**: 元評価は引用の正確性・記述の客観性・スコアと根拠の整合性のすべてにおいて高品質。Patterns 6-9 を「2×2 ファクトリアル比較実験」として高く評価する姿勢にやや好意的傾向は感じられるものの、引用された行番号・コードスニペット・YAML 内容は実物と完全に一致しており、誇張や捏造は確認されなかった。

---

## カテゴリ別検証

### A. 創発設計 — 元スコア 6.5/10

#### 根拠の検証 ✓
- `agent.py:265-270` 「place 内エージェントには `occupancy_rate: {x:.2f}` などの定量データのみ」: ✓ L256-270 で `agents_in_place / capacity / occupancy_rate` のみを数値として埋め込み、place 外には空文字を返す（L271-273）。引用通り。
- `agent.py:312-327` 「ACTION IMPERATIVE」「Inaction is boring」「Avoid abstract discussions」: ✓ L312-327 に完全一致。文言も正確。
- `agent.py:451-469` 同様の ACTION IMPERATIVE が `decide_action` 側にも繰り返される: ✓ L451-469 で確認、`"Move toward something"` などより具体的な指示が追加されている点も整合。
- `simulation.py:485-519` 容量超過時の移動ブロック: ✓ L485-519 で `current_count >= target_place['capacity']` の場合 `continue` してブロック。コメント「Sequential processing means earlier moves consume capacity before later ones are checked」も完全一致。
- `agent.py:99-122` 通信ルール: ✓ L99-122 で「両者外部」「同一 place 内」のみ通信可能と明示。説明と完全一致。

#### スコア妥当性
6.5/10 は妥当。設計の精緻さ（4 方向移動、矩形 place、capacity 強制、知覚境界、通信ルール）は高水準だが、プロンプト中の「Inaction is boring」「ACT INSTEAD」など強い行動指示は **行動の自由度を狭める誘導** にあたる。評価レポートはこの点を明示的に減点理由として挙げており、ルール×自由度のバランス評価として整合的。

#### 見落とし・誇張
- 誇張は確認できない。
- 軽微な見落とし: `_build_fire_section`（L177-201）で「警告：人体に有害です」「速やかに外へ退避することを検討してください」という **直接的な行動誘導が日本語で混在** している点も「自由度を狭める要素」として追加減点理由になり得たが、メイン実験（P6-9）では fire は使用されないため影響は限定的。

#### 検証結論
スコア 6.5/10 を **維持**。引用は全て正確。

---

### B. 世界設定 — 元スコア 8.0/10

#### 根拠の検証 ✓
- `config.yaml:17-28` 月面ハビタット設定: ✓ L16-28 で「Lunar Habitat Beta — a sealed dome on the surface of the Moon. Long after humanity left ... You and your peers are the first to encounter this space.」を確認。引用は文言レベルで正確。
- README "Punctuation Pass" 命名→16 体採用: ✓ README L41-43「Agent 19（Casual ペルソナ）が "Punctuation Pass" を命名（step 65）/ 翌ステップで 16 体が一斉採用」を確認。
- README ペルソナ有/無の対比結論: ✓ README L24-26「ペルソナ有では応援や動きの語彙形成を伴う関係的文化、ペルソナ無では構造化を中心とする文化」を確認。
- `config_pattern8.yaml:11-20` 2×2 ファクトリアル設計の明示: ✓ L8-20 で「2x2 factorial design」と明示、P6/P7/P8/P9 のセル表まで記載。引用通り。

#### スコア妥当性
8.0/10 は妥当〜やや高め。テーマ性は確かに独創的で、「シンギュラリティ後の月面 × 名前のない器具 × 人工生命体」という三層の前提は他提出物に類例の少ない設定。さらに 2×2 ファクトリアル実験を 4 セル分（≈ 100 ステップ × 4 = 24-56 時間）走らせている工数は世界設定の説得力を裏づける。8 点は十分正当化される。

#### 見落とし・誇張
- 誇張なし。
- 見落とし: README に明記される P6（cheer 1,010 / hype 1,404）、P9（Handover 564 / sequence 8,044）、P8（rod 6,763 / structure 13,073）の **定量キーワード集計** はいずれも実検証されておらず、評価本文ではこれらを世界設定の説得力の補強材料として無条件で受容している。`messages.jsonl` を実行集計してはいないが、これは時間的・実行的に避け難い制約。

#### 検証結論
スコア 8.0/10 を **維持**。引用正確。

---

### C. 発展性 — 元スコア 6.5/10

#### 根拠の検証 ✓
- `agent.py:276-289` `world_description_override` でシナリオ差し替え: ✓ L275-289 で `if self.world_description_override:` を確認、`config.yaml` から `world_description` を完全に上書き可能。
- 複数パターン用 config の存在: ✓ `config.yaml / config_pattern6.yaml / config_pattern8.yaml / config_pattern9.yaml` の 4 ファイルを目視確認。
- `utils.py:34-39` 矩形 place 対応 `get_place_half_sizes`: ✓ L34-39 で `half_size_x/y` を優先しつつ後方互換 `half_size` フォールバックを実装。引用通り。
- README「複数 seed / 複数 run での再現性検証は今後の課題」: ✓ README L184-185 で確認（「seed は固定していないため、LLM 出力には確率的な揺らぎがある。複数 seed / 複数 run での再現性検証は今後の課題」）。

#### スコア妥当性
6.5/10 は妥当。モジュール分割（agent / simulation / ollama_client / visualization / utils）と YAML 外部化は素直で拡張に強い。一方で「将来展望」の具体記述は README に乏しく（限界の認識はあるが、次にやる研究的拡張の提案は不在）、その点を評価レポートが明確に減点要素として指摘しているのは適切。

#### 見落とし・誇張
- 誇張なし。
- 見落とし: `visualization.py` が 38KB と単一ファイルとして大きく、`agent.py`（29KB）、`simulation.py`（27KB）も中規模。**「モジュール分割は明快」と評している一方、ファイル内クラス・関数の責務分離度までは未検証**。ただし C カテゴリの主軸（設定差し替えで複数シナリオを走らせられる）は完全に達成されているため減点には至らない。
- `agent.py` の docstring・型ヒント（`TypedDict`, `Tuple[int, int]` 等）は豊富で、評価本文「TypedDict で意思決定構造を型付け」（D 側）の記述とも整合。

#### 検証結論
スコア 6.5/10 を **維持**。引用正確。

---

### D. 技術実装 — 元スコア 7.5/10

#### 根拠の検証 ✓
- `agent.py:234-491` 2 段階プロンプト分離: ✓ `create_message_prompt`（L234-）は `include_position=False`（L243）で位置情報を渡さず、`create_decision_prompt`（L350-491）が `Position: ({self.position[0]}, {self.position[1]})` を含む。フェーズ間情報リーク回避の設計として実装が正確。
- `agent.py:493-528` ブレース・マッチング JSON 抽出: ✓ L493-528 で `in_string` フラグと `\` エスケープ処理を含む真面目な実装を確認。説明通り。
- `agent.py:530-606` フォールバック direction 抽出: ✓ L530-548 でキーワード抽出、L596-606 で `parse_action_response` のフォールバック分岐を確認。
- `ollama_client.py:79-89` `think:false` / `num_ctx` / `num_gpu`: ✓ L79 で `"think": False` を明示、L84-85 で `num_ctx / num_gpu` を payload に渡す。`temperature 0.2` は外部設定経由で適用される設計。引用通り。
- `agent.py:159-165` memory_size=5 のスライス: ✓ `recent_memory = self.memory[-self.memory_size:]` を確認。`memory_limit=20` は `__init__` で確認。
- `agent.py:649-658` 自己フィードバック型メモリ: ✓ L649-658 で LLM が返した `memory` フィールドを次ステップに保存。引用通り。
- `simulation.py:371-519` 4 フェーズ同期実行: ✓ L371-379 のドキュメント「1. messages → 2. send → 3. actions → 4. move」と L485-519 の容量チェック付き移動を確認。
- `simulation.py:485-489` シーケンシャル処理コメント: ✓ コメント文「Sequential processing means earlier moves consume capacity before later ones are checked」を完全一致で確認。
- README seed 非固定: ✓ 同上 README L184-185。

#### スコア妥当性
7.5/10 は妥当。
- **LLM 利用 3 点満点**: プロンプト 2 段分離、`think:false`、JSON ブレース・マッチング、フォールバックまで揃っており満点近く正当。
- **メモリ 3 点満点**: 直近 5 件＋自己フィードバック型は「単なるログダンプではない」が、階層・要約・反省ステップは無いという評価本文の指摘が正確。3 点満点までは付かないがそれに近い水準と判断できる。
- **コード品質 2 点満点**: `TypedDict` / docstring / 定数集約は確認済み。
- **シミュレーション正確性 2 点満点**: 4 フェーズ同期化と容量強制のシーケンシャル意識は明確。減点理由（seed 非固定）も適切に指摘。

#### 見落とし・誇張
- 誇張なし。`ollama_client.py:102-107` のエラーハンドリング引用も L102-107 で `requests.exceptions.RequestException` と汎用 `Exception` を捕捉して空文字を返す実装と一致。
- 軽微な見落とし: `ollama_client.py` ではエラー時に空文字を返すため、`agent.py` 側の JSON パースが必ずフォールバック経路に落ちる構造になっており、**「エラー時にエージェントが沈黙する」副作用** がある。これは正確性の観点で減点になり得るが、評価本文は「エラーハンドリングは LLM 呼び出しとエージェント決定の両方に存在」と肯定的に評価しているのみ。

#### 検証結論
スコア 7.5/10 を **維持**。引用は全て正確で、引用行番号も全件で実物と一致。

---

## 総合所見

### 引用の正確性
本評価レポートは合計 17 件以上のコード/設定/README 引用を行っており、検証した全 12 件の引用箇所（A:5, B:4, C:4, D:9 — 一部重複）で **行番号・文言・YAML 内容が実物と完全に一致**。捏造・誇張・拡大解釈は確認できなかった。引用品質は本ハッカソン評価群の中でも高水準。

### スコア妥当性
- A=6.5: プロンプト中の強い行動指示（"Inaction is boring" / "ACT INSTEAD" / "Avoid abstract discussions"）を減点理由として明示しており、創発設計を「ルール × 自由度」の二軸で読み解く姿勢が一貫している。妥当。
- B=8.0: 月面 × 人工生命体 × 名前のない器具 という三層設定の独創性と、2×2 ファクトリアル実験設計の研究的厳密性は確かに突出。8 点は十分妥当。
- C=6.5: YAML 駆動の設定外部化と矩形 place 対応は確認済み。将来展望の具体記述が乏しいという減点理由も README と整合。妥当。
- D=7.5: 2 段階プロンプト、`think:false`、ブレース・マッチング、4 フェーズ同期化、自己フィードバック型メモリと、技術実装の主要項目は全て揃っている。seed 非固定での減点理由も適切。妥当。

### 改善提言（メタ評価として）
1. **定量主張の検証範囲**: P6-P9 の「cheer 1,010 / hype 1,404 / rod 6,763 / structure 13,073」「step 65 で命名 → 翌ステップで 16 体採用」などの **キーワード集計値は `messages.jsonl` で実検証されていない**。提出物を信頼ベースで受容するのは合理的だが、レポート内に「数値はサンプル messages.jsonl で未検証」の注釈があれば検証の透明性が増す。
2. **`_build_fire_section` の日本語混在指示**: メイン実験 P6-9 では fire を使用していないため評価対象外であるが、コードベース全体としては「日本語の警告文」が混在する点に触れてもよい。
3. **`visualization.py` 38KB の責務分離**: モジュール分割を高く評価する以上、最大ファイルの内部構造（クラス/関数の分離度）まで踏み込むと C カテゴリの評価精度が上がる。

### 最終判定
**元評価合計 28.5/40 を維持**。各カテゴリスコアおよび総評の質的記述は、ソースコードと README の実体と整合的であり、引用の正確性も高い。本評価は信頼できる。
