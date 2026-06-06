# Decision Lab

**意思決定の前に、どの価値観を会議に参加させるべきかを設計するツール**

🔗 https://yutate.github.io/decision-lab/

---

## コンセプト

Decision Lab は「何を優先するか」を決めるツールではない。

> *«意思決定の前に、どの価値観を会議に参加させるべきかを生成し、その価値観同士の衝突を可視化するツールである。»*

AIは答えを決めない。AIは会議を設計する。

### ツール系譜

| ツール | 扱うもの |
|--------|----------|
| Media Planner | 何をするか |
| Ad Strategy Studio | なぜやるか |
| AdCP Execution Lab | どう調整するか（媒体交渉） |
| **Decision Lab** | **誰を会議に参加させ、何が争点になるか** |

Execution Lab を作る中での気づきが起点：媒体同士が交渉するより、**価値観同士が交渉している**のが本質。

---

## アーキテクチャ

```
Input（テーマ + コンテキスト）
  ↓
① Agent Generation    — AIが会議参加者を設計
  ↓
② Conflict Detection  — Agent間の対立を分析
  ↓
③ Options Generation  — 主要Conflictへの選択肢を生成（on demand）
  ↓
Human Decision Layer  — 人間が選択・確定
```

### 3ステップAPI設計

| Step | 内容 | モデル | 出力量 |
|------|------|--------|--------|
| generateAgents | テーマ → 4〜6 Agent設計 | Haiku | 軽量 |
| generateConflicts | Agent間のConflict分析（最大3件） | Haiku | 中量 |
| generateOptions | 選択したConflictの選択肢3案 | Haiku | 軽量 |

2回呼び出しにするとトークン超過が発生したため3分割に設計変更（v0.4）。

---

## Agent設計

テーマ入力からAIが動的に生成。構造：

```json
{
  "name": "Ecosystem Agent",
  "mission": "経済圏全体の成長を最大化する",
  "priority": ["cross use", "ecosystem expansion", "network effect"],
  "risk": ["service fragmentation", "best service preference"],
  "why_invited": "なぜこのテーマでこのAgentが必要か"
}
```

**Missing Voice**：この会議に不足している視点もAIが自動検出。

---

## Conflict Detection

Agent間の対立ペアを分析し、各Conflictに以下を生成：

```json
{
  "label": "財務効率 vs エコシステム統合",
  "tension_score": 82,
  "level": "High",
  "why": "対立の理由",
  "debate": "「実際の会議でぶつかる発言」",
  "human_question": "人間が判断すべき問い",
  "axis_a": "価値軸A",
  "axis_b": "価値軸B"
}
```

**Conflict Level**：High（70+）/ Medium（40〜69）/ Low（〜39）

Alignment Map も同時生成（方向性が一致するAgentペア）。

---

## Human Decision Layer

主要Conflictに対して3択を生成：

- **Option A** — 価値軸Aを優先する案
- **Option B** — 価値軸Bを優先する案
- **Balanced** — 両者を折衷する案

各案に `desc` / `best_for` / `risk` 付き。最終決定は人間が行う。

---

## 出力

- **画面表示**：Meeting Design → Conflict Map → Alignment Map → Trade-off Explanation → Human Decision → Decision Record
- **JSON Export**：全セクションを構造化データで出力
- **PDF Export**：A4レポート（jsPDF + html2canvas）

---

## 技術構成

| 項目 | 内容 |
|------|------|
| 実装 | 単一HTML（Vanilla JS） |
| AIモデル | claude-haiku-4-5-20251001 |
| API | Anthropic Messages API（ブラウザ直接呼び出し） |
| PDF | jsPDF 2.5.1 |
| デプロイ | GitHub Pages |
| API Key保存 | localStorage |

---

## バージョン履歴

| Version | 内容 |
|---------|------|
| v0.1 | 固定6 Agent（Growth / Profit / Brand / Youth / LTV / Trust）、ルールベーススコアリング |
| v0.2 | 設計整理（入力設計・スコアリングマトリクス・均衡判定・名前付き戦略案） |
| v0.3 | コンセプト転換：「優先順位診断」→「争点可視化」。Conflict Detection導入、Conflict起点のHDL |
| v0.4 | Agent Generation Layer追加。固定Agent廃止→AIによる動的生成。3ステップAPI分割。PDF Export追加 |

---

## 設計思想

- **決めるAIではなく、決めさせるAI**：AIは選択肢と争点を提示し、決定は人間が行う
- **最適化ではなく合意形成**：単一の正解ではなく、組織内の価値観の衝突を可視化する
- **会議の前に会議を設計する**：誰を呼ぶかを決めることが、意思決定の質を左右する

---

## 関連ツール

- [Ad Strategy Studio](https://yutate.github.io/ad-strategy-studio/) — 広告戦略設計
- [AdCP Execution Lab](https://yutate.github.io/lab/adcp-exe/) — 媒体交渉シミュレーター
- [Media Planner](https://yutate.github.io/media-planner/) — 媒体配分設計
- [Tools Portal](https://yutate.github.io/tools/) — ツール一覧
