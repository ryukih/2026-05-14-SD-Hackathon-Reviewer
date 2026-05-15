# 評価レポート: AlberiaMisinformationProtest

**評価日**: 2026-05-11
**実施回数**: 2回

---

## 概要

架空都市「アルベリア」を舞台にした、誤情報誘発型抗議運動のエージェントベース・シミュレーション。20×20 グリッド上に住宅エリア・中央広場・市庁舎・メディアゾーン・警察署など 9 種のランドマークが配置された 2D 空間で展開する。エージェントは、信念・感情・意図を LLM で自律決定する CitizenAgent と、ルールベースで動作する CopAgent（安全担当官）・InfluencerAgent（SNS 発信者）・OfficialAgent（架空行政）の 4 種類で構成され、市民は ORDINARY / OBSERVER / PARTICIPANT / MEDIATOR / SKEPTIC / WITHDRAWN / DETAINED の 7 状態を遷移する。主要メカニズムとして、SocialFeed と呼ばれる SNS 風情報空間に truth_status（TRUE / FALSE / UNVERIFIED / CORRECTION / MISLEADING）・感情トーン・到達範囲・信頼度を持つ投稿が流れ、市民は sns_activity に応じて閲覧・反応する。3 つの誤情報イベントが異なるステップで発火し、公式訂正は遅延後に OfficialAgent から発表される設計。Python 側は 2D 空間・視界・移動制約・行動可能リスト・禁止行動・可視化のみを定義し、情報の解釈・信念更新・感情変化・移動意図・発話・SNS 投稿は LLM が担う構造を採る。テーマは、局所的な目撃情報・SNS 情報伝播・誤情報と訂正・信頼／恐怖／怒り／連帯感が、抗議参加意図と群衆形成にどう影響するかの観察である。

---

## スコアサマリー

| カテゴリ      | Run1 | Run2 | Run3 | 最終(平均) |
|-------------|------|------|------|-----------|
| A. 創発設計  |  8   |  7   |  -   |    7.5    |
| B. 世界設定  |  9   |  9   |  -   |    9.0    |
| C. 発展性    |  7   |  7   |  -   |    7.0    |
| D. 技術実装  |  9   |  9   |  -   |    9.0    |
| **合計**    |  33  |  32  |  -   |  **32.5** |

---

## 評価詳細

### A. 創発設計 — 最終: 7.5/10

#### 評価
世界ルールは精度高く設計されており、20×20グリッド・9種類のランドマーク・視界半径・3イベントの噂/真実ペア・SNS投稿のtruth_status分類（TRUE/FALSE/UNVERIFIED/CORRECTION/MISLEADING）・信頼性スコアなど、市民の判断に意味ある差を生むに足る情報構造が定義されている。エージェントには14種類の`intent`選択肢と生データ（観測・SNS投稿・記憶・性格・感情）のみが渡され、「何をすべきか」の指示はない。一方、システムプロンプトには感情の発話への反映を促す表現的なナッジが含まれ、また`_fallback_decision`では`anger>0.38 and max_belief>0.35`等の閾値ベース遷移を持つルールロジックが存在する（フォールバックではあるが）。LLMパスでは行動の自由度はほぼ最大限保たれているため、世界設計の精度と行動自由度の両軸でバランスは良好だが、プロンプトの感情反映ガイダンスがやや行動誘導的に作用しうる点で満点には届かない。

#### 根拠
- `agents.py:65-80` — `VALID_INTENTS`に14種の意図が定義され、LLMはこの中から自由に選択する設計
- `prompts.py:153-182` — 出力スキーマは belief_updates / emotion_update / intent / utterance / sns_post を全てLLMが自律的に決定する構造
- `prompts.py:209-211` — `"発話は現在の anger / fear / solidarity を反映してください。怒り・不安・疑念・失望・焦りを含めてよいです。"` — 表現的ナッジ
- `simulation.py:251-275` — イベント発火時に近傍市民へ rumor/truth/severity を「データとして」付与（行動指示なし）
- `agents.py:469-499` — フォールバック側にはルールベースの状態遷移閾値が存在
- `README.md` — "「世界の物理法則・社会規範・行動可能性だけを定義し、LLMエージェントが自律的に解釈・判断・行動する社会を観測する」"

---

### B. 世界設定 — 最終: 9.0/10

#### 評価
架空都市「アルベリア」を舞台に、誤情報・噂・公式訂正・SNS増幅・信頼・恐怖・連帯感が抗議参加意図に与える影響を観察する社会科学的シミュレーションで、独自性・社会的妥当性ともに高い。Vosoughi et al. (2018)「The spread of true and false news online」やEpstein (2002) のCivil Violenceモデル等の学術論文を参考にし、Civil Violenceとの差異も明示的に比較されている。媒体リテラシー・公式情報への信頼度・SNS活動度などの個人差、4種類のインフルエンサーalignment（sensational/skeptical/official_friendly/protest_sympathetic）、3段階の噂事象、遅延公式訂正など、現実の情報生態系の核心要素を架空空間に丁寧にマッピングしている。倫理的注意事項も明記されている。

#### 根拠
- `README.md` — "局所的な目撃情報・SNS風の情報伝播・誤情報・訂正情報・信頼・恐怖・怒り・連帯感が、市民の抗議参加意図や群衆形成にどう影響するかを観察する研究用シミュレーション"
- `README.md` — Civil Violence (Epstein 2002) との7項目比較表
- `README.md` — 参考文献にVosoughi/Centola/Lewandowsky/Bonabeauなどの査読論文を列挙
- `config:76-103` — 3つの誤情報イベント（配給排除疑惑/参加強制噂/メディア圧力疑惑）が階層的に設計されている
- `agents.py:687-692` — 4種類のinfluencer alignment（sensational/skeptical/official_friendly/protest_sympathetic）
- `README.md — 倫理的注意` — "現実の政治運動・暴動・抗議活動の扇動・最適化・支援" を目的としない明示

---

### C. 発展性 — 最終: 7.0/10

#### コード拡張性
ファイル単位での関心分離が良好（`agents.py` / `simulation.py` / `prompts.py` / `ollama_client.py` / `visualization.py` / `main.py`）。エージェント種別は4クラスに分かれており、新規エージェントクラスの追加は容易。YAMLでランドマーク座標・イベント・インフルエンサー設定・LLMモデル等が外部化されている。`VALID_INTENTS` と `INTENT_TO_LANDMARK` の対応も拡張しやすい構造。

#### 将来展望
READMEには「実験例」セクションがあり、情報リテラシー・公式対応速度・インフルエンサーalignmentを変えた具体的な実験提案が3つある。ただし、現行スコープを超えた研究方向（例：複数都市間の情報伝播、エージェントの長期記憶、政策介入実験など）の明示的なロードマップは無い。将来展望としてはやや控えめ。

#### 根拠
- `README.md — 実験例` — "実験1: 情報リテラシーの効果 / 実験2: 公式対応速度の効果 / 実験3: インフルエンサーの影響" の3パラメータスイープ提案
- `config:38-74` — landmark/events/influencers/officialが全てYAMLで外部化
- `agents.py:687-692` — `ALIGNMENT_TRUTH_MAP`の辞書追加で新規influencer alignment追加可能
- `simulation.py:41-101` — Simulation初期化はconfig駆動で、エージェント追加・地形変更がコード変更を最小化
- `README.md` — 概念的な将来研究方向（cross-city, longer memory, policy intervention 等）の記述は無い

---

### D. 技術実装 — 最終: 9.0/10

#### LLM利用
Ollamaクライアントは構造化JSON出力強制（`format: "json"`）、temperature/min_p/repeat_penalty等を設定可能、タイムアウト・接続チェック・モデル存在確認・例外ハンドリングを完備。qwen3の`<think>...</think>`ブロックの除去、JSON抽出のフォールバック、空応答時のフォールバック決定など、LLM不安定性への配慮が手厚い。プロンプトは7セクション構造（性格・状況フェーズ・内面状態・観察・SNS・記憶・行動選択肢）で、文脈管理が明確。

#### メモリ設計
`deque(maxlen=memory_limit)` でテキスト記憶が保持され、イベント発火時には自動的に「ステップNでXとYを聞いた」とエピソード的に記憶に追加される。プロンプトでは番号付きリストで提示される。`belief_rumors: dict[str, float]` で噂ごとの信念強度を別管理しており、感情とは別チャネル。記憶はやや浅め（フラットなテキストログ）だが、噂・信念の構造化と分離されているため意味的整合性は保たれている。

#### コード品質・シミュレーション正確性
モジュール分離が明確、型ヒントが概ね付与され、docstringがクラス・関数に整備されている。`random.seed(seed)` でシード再現性確保、`random.shuffle(citizens)` で順序公平性、`grid: dict[pos→list]` で複数エージェント同一セル対応。検証パイプライン（intent検証/暴力ワードフィルタ/不自然な疑念表現ブロック/日本語クリーニング）も堅実。感情の慣性（emotional inertia, max(new, old*0.70)）で1ステップの揺らぎ吸収。JSONLログ・summary.json・統計グラフ・PNGフレーム・動画生成・ブラウザビューアまで出力系も充実。

#### 根拠
- `ollama_client.py:71-104` — payload構築・タイムアウト・ConnectionError/Timeout/RequestException 別の例外処理
- `agents.py:316-326` — `<think>` ブロック除去 + JSON正規表現抽出 + フォールバック
- `agents.py:303-415` — `_parse_and_validate` による intent / public_state / emotion / belief_updates / utterance / sns_post の全フィールド検証
- `agents.py:835-869` — 暴力キーワード検出と日本語テキストクリーニング
- `agents.py:523-531` — `self.anger = _clamp(max(eu.get("anger", self.anger), self.anger * 0.70))` 感情の慣性
- `simulation.py:50` — `random.seed(seed)` 再現性
- `simulation.py:432-492` — JSONL形式での decisions / messages / social_feed の永続化
- `agents.py:221` — `self.memory: deque[str] = deque(maxlen=self.memory_limit)` 記憶のリングバッファ
- `simulation.py:259-262` — イベント発火時のエピソード記憶自動追加

---

## 総評

### 優れている点
- 世界ルール（グリッド・ランドマーク・SNS truth_status分類・信頼性スコア・イベント発火）が精密に定義され、LLMには生データのみを渡す設計思想が明文化されている
- Civil Violence (Epstein 2002) 等の先行研究との比較を表で明示しており、学術的位置づけが明確
- LLM統合の品質が高い（JSON強制・think除去・例外処理・フォールバック・暴力フィルタ・日本語クリーニング）
- 出力系が極めて充実（PNGフレーム/JSONL/統計グラフ/動画/HTMLビューア）
- 倫理的配慮が明示されており、研究倫理面の意識が高い

### 改善点・提言
- システムプロンプト内の感情表現ナッジ（「怒り・不安・疑念・失望・焦りを含めてよい」等）が、エージェントの自由判断にやや方向性を与えうる。より中立的な「感情状態を持っている事実」だけを伝える表現に絞れば創発設計スコアが上がる
- 将来展望セクションが「パラメータスイープ実験例」に留まっている。複数都市・長期記憶・政策介入シナリオ・他LLMモデル比較などの研究的ロードマップを明示すると発展性が増す
- Cop/Influencer/Officialの3種がルールベースに固定されている。例えばCopAgentのLLM化（過剰反応 vs. 抑制の判断）も創発の幅を広げる候補

### 一言コメント
誤情報の社会的拡散と抗議参加意図の創発を、架空都市というセーフガード付きでLLMエージェントベースに実装した、設計思想・倫理意識・実装品質のいずれも高水準な作品。世界ルールの精度とLLM統合の堅牢性が特に光る。
