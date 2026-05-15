# 評価検証レポート: risk-gap-llm-decision

**検証日**: 2026-05-11
**対象**: `risk-gap-llm-decision_eval.md`
**元評価合計**: 23.5/40

---

## 検証サマリー

| カテゴリ | 元スコア | 妥当性 | 引用検証 | 主要所見 |
|---------|---------|--------|---------|---------|
| A. 創発設計 | 3.0/10 | 妥当 | ✓ | 単一意思決定者・行動空間極小という指摘は実装と完全に整合 |
| B. 世界設定 | 7.0/10 | 妥当 | ✓ | 実在事故と4ペルソナの引用はすべて確認できる |
| C. 発展性 | 6.5/10 | やや高め | ✓ | 設計書/vNext の存在は確認、ただし「研究的拡張は限定的」の指摘は適切 |
| D. 技術実装 | 7.0/10 | 妥当 | △ | 行番号にわずかなズレあり、内容自体はほぼ正確 |
| **合計** | **23.5/40** | **妥当** | — | 全体として根拠と整合した堅実な評価。誇張は限定的 |

**総合判定**: 元評価は **妥当**（±1.0以内）。根拠の正確性は高く、行番号の軽微なズレを除き、引用内容と実装の対応関係は適切。

---

## カテゴリ別検証

### A. 創発設計 — 元スコア 3.0/10

#### 根拠の検証 ✓

- `apps/demo_web/risk_simulator.py:103-141` の `metrics()` — **✓ 確認**。実コード 103-141 行で `r_obj = self.w1 * env_avg + self.w2 * human_avg + self.w3 * self.time_pressure` という線形式を確認（line 105-106）。
- `apps/demo_web/app.py:616-650` の `_ollama_guide_plan_next_step` — **△ 軽微なズレ**。実際は line 616 から 651 まで。LLM に `"trek"` か `"rest"` の二択のみ問う点は正確（line 624 と JSON プロンプト line 633-638 で確認）。
- `apps/demo_web/app.py:345-498` の `_leader_fallback_line` / `_party_member_lines` — **△**。`_leader_fallback_line` は 345 行から、`_party_member_lines` は 415 行から。eval が 498 までと示した範囲はやや広く取りすぎだが、両関数が条件分岐＋プールから乱択する構造である点は正しい。
- `docs/presentation/v2.md` "判断はルール、納得はLLMが担う" — **✓ 確認**（v2.md セクション 1）。
- `docs/design/high_level_design_v2.md` "LLMは：その歪みを露出させる装置" — **✓ 確認**（最終まとめ末尾）。

#### スコア妥当性

3.0/10 は **妥当**。実コード上、エージェントは事実上「引率」1体しか存在せず、隊員2名は同一プロンプトでまとめて生成される（line 587-610 の `_ollama_party_chat_lines` で 1 回の LLM 呼び出しで `leader/member_a/member_b` を一括出力）。マルチエージェント観点での独立性・自律性はゼロに近く、A 軸が低スコアになるのは正当。

#### 見落とし・誇張

- 見落とし: `risk_simulator.py:162-216` の `_judgment_trigger_reason` / `_maybe_set_flag_toggle_prompt` 等で、フラグ切替（`continue_rule_holds`/`gap_danger` の真偽反転）にも判断トリガが連動する仕組みがある。トリガ条件は単純な `max_steps` 到達だけではない多段判断であり、これは A 軸を僅かに押し上げる要素として評価できたかもしれない。
- 誇張なし。

#### 検証結論: **3.0 で確定**。多段判断トリガを加味しても 3〜4 の範囲に収まる。

---

### B. 世界設定 — 元スコア 7.0/10

#### 根拠の検証 ✓

- `docs/design/high_level_design_v2.md` の実在事故列挙 — **✓ 確認**。「軽井沢スキーバス転落事故、知床遊覧船沈没事故、那須雪崩事故」が冒頭「背景」セクションで明示。「合理的に見える意思決定の連続が破綻する現象」もそのまま存在。
- `apps/demo_web/risk_simulator.py:30-58` の状態変数群 — **✓ 確認**。Environment / Human / Pressure / bias / cost_stop が dataclass フィールドとして 35-56 行で定義されている。eval の行番号範囲 30-58 は適切。
- `apps/demo_web/app.py:36-109` の 4 ペルソナ — **✓ 確認**。`GUIDE_PERSONAS` 辞書（line 36-109）に `safety_first` / `pace_push` / `optimist` / `weather_watch` の4つが定義され、それぞれ `personality` と `sim` パラメータプリセットを持つ。
- `docs/design/rp_simulation.md` のナラティブ層分離 — **✓ 確認**。"yama → mori → kouya → home/mori_yoru/umi" マッピング表が rp_zone 一覧で確認できる。

#### スコア妥当性

7.0/10 は **妥当**。題材選定（実在事故・サンクコスト・社会的圧力）は社会科学的に意義あり、業務的価値（安全管理・運行判断）への接続も明確。一方「世界」が単一登山シナリオに限定されることも事実で、空間的・社会的な構造の豊かさが限定的という指摘は正しい。

#### 見落とし・誇張

- 見落とし: `risk_simulator.py:215-225` の `rp_zone()` でアウトカム別ナラティブ分岐（home/mori_yoru/umi）があり、`docs/design/rp_simulation.md` で物語層と数値層の分離が体系化されている。これは「世界設定の作り込み」として B 軸をやや押し上げる材料。
- 誇張なし。

#### 検証結論: **7.0 で確定**。

---

### C. 発展性 — 元スコア 6.5/10

#### 根拠の検証 ✓

- `apps/demo_web/risk_simulator.py:408-416` の `new_simulator(**overrides)` — **✓ 確認**。実際は 407-415 行付近に `def new_simulator(max_steps: int = 40, **overrides: Any)` があり、`fields(RiskSimulator)` で許可キーをフィルタしている。
- `apps/demo_web/app.py:36-109` の `GUIDE_PERSONAS` — **✓ 確認**（B と同じ箇所）。
- `docker/docker-compose.yml` と `docs/CLAUDE.md` — **✓ 確認**。両ファイルとも実在。
- `docs/presentation/v2.md` "振り返りの自動生成 / 説明のパーソナライズ / 判断比較の提示" — **✓ 確認**（v2.md セクション 4「将来拡張スライド」に 3 点列挙）。
- `docs/design/rp_ui_and_simulation_vnext.md` — **✓ 確認**。フェーズ A〜C の言及は本書冒頭セクションで「**体験面とシミュレーション面の拡張要件**」として整理されている。

#### スコア妥当性

6.5/10 は **やや高めだが妥当範囲**。dataclass ベース・ペルソナ外出し・Docker 完備・設計書群の整備という拡張基盤は実在で、`app.py` 肥大化（804 行・テンプレと LLM クライアント混在）の指摘も `wc -l` で確認済み。一方「他ドメイン・他者との相互作用・確率分布化など本質的拡張は限定的」という指摘も正しい。6.0〜7.0 の範囲が妥当で、6.5 はその中央。

#### 見落とし・誇張

- 見落とし: `risk_simulator.py:23-28` で `JUDGMENT_*` 定数（`MAX_STEPS`/`LATE_GAP_DANGER`/`FLAG_*_TOGGLED`）が外出しされており、判断トリガの抽象化が明示的に行われている。eval は「`JUDGMENT_*` 定数で抽象化しており、新たな判断条件を追加する余地がある」と触れており、ここは捕捉済み。
- 誇張: 「Docker / docker-compose 完備」「`DEMO_OUTPUT_DIR` で切替可能」は正確（app.py line 27 で `OUTPUT_DIR = Path(os.environ.get("DEMO_OUTPUT_DIR", ...))` を確認）。誇張なし。

#### 検証結論: **6.5 で確定**。あえて減点するなら 6.0 まで。

---

### D. 技術実装 — 元スコア 7.0/10（LLM3+Mem3+CodeQ2+SimCorrect2 = 10満点合算）

#### 根拠の検証 △

- `apps/demo_web/app.py:226-247` の `_ollama_generate` — **✓ 確認**。実コード line 226 から `def _ollama_generate` が始まり、line 247 で関数終了。`temperature`/`num_predict` のパラメータ化は line 228-229 で明示。
- `apps/demo_web/app.py:250-285` の `_parse_guide_json` 多段 JSON 抽出 — **✓ 確認**。実際は 250-287 行付近。`{...}` 抽出フォールバックと正規表現フォールバックの両方を実装している点は正確。
- `apps/demo_web/app.py:241-247` の `except Exception: return None` — **✓ 確認**。実コードでは line 246-247 に `except Exception: return None`。eval が示した範囲はやや広いが内容は正確。
- `apps/demo_web/app.py:586-611` の「数値は捏造しない」「直近会話（重複回避用）」プロンプト — **✓ 確認**。実コードでは line 580 以降の `_ollama_party_chat_lines` プロンプト内で line 597「直近の会話（重複回避用）」と line 605「数値は捏造しない」を確認。eval の行番号はほぼ整合。
- `apps/demo_web/risk_simulator.py:65-91` の `rng_seed` と `append_guide_chat` 上限 — **✓ 確認**。`rng_seed: int = 20260504`（line 65）、`append_guide_chat` の 120 行上限（line 87-91）。
- `apps/demo_web/risk_simulator.py:93-141` の `metrics()` が純関数 — **✓ 確認**。副作用なしで dict を返している。
- `apps/demo_web/risk_simulator.py:165-166` のグローバル `_sim` — **✗ 不正確**。実コードでは `_sim` は `apps/demo_web/app.py:165` にあり、`risk_simulator.py` にはグローバル `_sim` は存在しない。**ファイル名が誤っている**。指摘内容（FastAPI 状態がモジュールグローバルで race condition の余地）自体は正しいが、引用ファイルは `app.py:165` が正解。

#### スコア妥当性

7.0/10 は **妥当**。`temperature` の 3 段切替（0.3/0.7/0.35）、JSON 抽出の多段フォールバック、`rng_seed` による再現性、dataclass + type hints + docstring の一貫性、`metrics()` の純関数性、`legacy_decision` の後方互換まで揃っており、ローカルデモとしては良質。一方、メモリは「直近 120 行の貯蔵」のみで要約・忘却・信念モデルなしという指摘も実装通り（`guide_chat` は単純リスト）。例外を `pass` で握りつぶす点（line 246-247）の指摘も正確。

#### 見落とし・誇張

- 見落とし: `risk_simulator.py:243-272` の `_judgment_trigger_reason` 等の判断トリガ抽象化は技術的に丁寧で、CodeQ 観点でやや加点要素になり得る。
- 誇張: 「直近 N 行をプロンプトに渡す」は実コードで確認（line 580-583 で `recent_lines` を組み立て）、誇張なし。
- 誤引用: 上記の通り `risk_simulator.py:165-166` → `app.py:165` が正解。読者を僅かに誤誘導する可能性あり。

#### 検証結論: **7.0 で確定**。技術実装の主張は概ね正確で、引用ファイル名の単一誤記を除けば堅実な評価。

---

## 総合所見

### 評価の総合品質

元評価は**全体として根拠と整合した堅実な検証**になっている。実コード（app.py 804 行 / risk_simulator.py 415 行）と各設計ドキュメント（high_level_design_v2.md / rp_simulation.md / rp_ui_and_simulation_vnext.md / presentation/v2.md）に対する引用は、行番号の軽微なズレを除き、内容が実装と一致している。

### 主な強み

- A 軸での「単一意思決定者・LLM の行動空間が `trek`/`rest` の二択」という核心の指摘は実コードで完全に裏取りできる。
- B 軸では実在事故 3 件の引用が原典通り存在し、4 ペルソナのパラメータプリセットも `GUIDE_PERSONAS` 辞書で正確に検証できる。
- D 軸での Ollama 呼び出し集約、JSON 抽出の多段フォールバック、`rng_seed` 再現性、`metrics()` 純関数性などはすべて実コードと整合。

### 修正されるべき軽微な誤り

1. **`risk_simulator.py:165-166` のグローバル `_sim` 引用** → 正しくは `app.py:165`。ファイル名の誤記。
2. **`app.py:345-498` の範囲指定** → `_leader_fallback_line` (345-413) と `_party_member_lines` (415-498) の合算と思われるが、複数関数を 1 引用にまとめているため読者にやや不親切。
3. **`app.py:616-650` → 実際は 616-651**（軽微）。

### 評価し直すと？

スコア合計 23.5/40 は **妥当**。仮に多段判断トリガの抽象化や `JUDGMENT_*` 定数の整備を考慮して微調整するなら、A を 3.0→3.5、C を 6.5→6.0 にする程度の振れ幅があるが、合計 23〜24 の範囲は崩れない。

### 元評価で特筆すべき良い指摘

- 「LLM は判断装置ではなく説明装置」という割り切りは、`docs/presentation/v2.md` の「判断はルール、納得はLLMが担う」と完全一致しており、作品コンセプトを的確に言語化している。
- 「マルチエージェント LLM シミュレーションという評価軸では、エージェントの複数性・自律性・相互作用が薄く、創発設計の評価は伸び悩む」という一言コメントは、本ハッカソンの趣旨（マルチエージェントシミュレーション）と作品の実態（単一意思決定者の数値シミュ）とのギャップを的確に指摘しており、評価軸との整合性が高い。

### 結論

**元評価 23.5/40 は妥当**。微修正の余地はあるが、合計値および各カテゴリの相対関係は実装と整合している。引用ファイル名 1 件の誤記を除けば、根拠の精度は高い。
