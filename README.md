# ai-design-system

プロジェクト内にデザインシステムのSSOT（Single Source of Truth）を作成・維持するためのClaude Codeスキル。

## コンセプト

デザインシステムは外部ライブラリの依存として管理されることが多い（Chakra UI、shadcn/uiなど）が、これは移行コストとロックインを生む。このスキルは異なるアプローチを取る:

- SSOTはプロジェクト内部に設定ファイルとして配置する
- スキル自体はプロジェクト非依存 — プロジェクト固有の値を含まない
- AIエージェントがSSOTを読み取り一貫性を担保する。コンポーネントライブラリの役割を置き換える

## 仕組み

```
SSOTが存在する？
  No → BOOTSTRAP.md: プロジェクトのコンテキストからSSOTを生成
  Yes → AUDIT.md: 現状とあるべき姿のgapを検出
```

あらゆる技術スタックで動作する。SSOTのフォーマットはプロジェクトに応じて決定される（Tailwind設定、ScriptableObject、デザイントークンJSONなど）。

## 使い方

gitサブモジュールとして追加:

```bash
git submodule add https://github.com/coil398/ai-design-system .claude/skills/design-system
```

`CLAUDE.md` で参照:

```md
## Design System
See .claude/skills/design-system/SKILL.md
```

## ファイル構成

| ファイル | 役割 |
|---------|------|
| `SKILL.md` | エージェントのエントリーポイント |
| `IDEAL.md` | スタック非依存のデザインシステムのあるべき姿の定義 |
| `BOOTSTRAP.md` | SSOTが存在しない場合の生成フロー |
| `AUDIT.md` | あるべき姿とのgapの検出・修正フロー |

## 設計思想

- IDEAL.mdは「あるべき姿」を定義する。特定の状態からの移行方法ではない。差分はエージェントが導出する
- SSOTはコードである。リンターとエージェントの両方が読める機械可読な形式でなければならない
- スキルの更新は意図的に非破壊。スキル更新時、エージェントは新しいidealに対して現状のコードベースを再評価する。移行スクリプトは不要
