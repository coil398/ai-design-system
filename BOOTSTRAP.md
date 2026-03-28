# Bootstrap

SSOTが存在しない場合にデザインシステムのSSOTを生成するフロー。

---

## Step 1: プロジェクトのコンテキストを把握する

以下を調査せよ：

**技術スタック**
- フロントエンドフレームワーク（React / Vue / Unity / その他）
- スタイリング手法（Tailwind / CSS Modules / CSS-in-JS / UXML / その他）
- 既存のUIライブラリ（shadcn / Chakra / MUI / なし）

**現状のデザイン資産**
- 既存コンポーネントで使われている色・スペーシング・フォントを収集する
- 繰り返し現れる値をトークン候補とする
- ハードコードされた値の一覧を作る

**プロジェクトの性質**
- Web / ゲーム / モバイル / デスクトップ → SSOTのフォーマット選択とトークンの粒度に影響する
- レスポンシブ対応の有無 → ブレークポイントトークンを定義するか判断する
- ダークモード/テーマ切り替えの有無 → セマンティックトークンの階層設計に影響する

---

## Step 2: SSOTのフォーマットを決定する

スタックに応じて以下のフォーマットを選択する：

| スタック | 推奨フォーマット | ファイル名 |
|---------|----------------|----------|
| Tailwind | TypeScript / JS | `design-system.config.ts` |
| CSS Modules | CSS Custom Properties | `design-system.config.css` |
| Unity | C# static class | `DesignSystem.cs` |
| CSS-in-JS (styled-components, Emotion) | TypeScript | `design-system.config.ts` |
| Vue (SFC + scoped CSS) | CSS Custom Properties | `design-system.config.css` |
| スタック非依存 | JSON | `design-system.config.json` |
| 複合 | TypeScript（他形式を生成するスクリプト込み） | `design-system.config.ts` |

---

## Step 3: SSOTを生成する

### Tailwind プロジェクトの場合

`design-system.config.ts` を生成し、`tailwind.config.ts` からimportする構成にする：

```ts
// design-system.config.ts
export const designSystem = {
  tokens: {
    colors: {
      // 収集した色値をセマンティックな名前でまとめる
      // 例: primary: { DEFAULT: "#2563eb", hover: "#1d4ed8" }
    },
    spacing: {
      // 繰り返し現れたスペーシング値
    },
    typography: {
      // フォントファミリー・サイズ・ウェイト
    },
  },
  conventions: {
    componentDir: "src/components/ui",
    namingPattern: "PascalCase",
  },
  antipatterns: [
    "インラインスタイルの使用",
    "tailwind.configで定義されていない任意値 (text-[#xxx])",
    "コンポーネント外でのスタイル定義",
  ],
} as const;
```

`tailwind.config.ts` での参照：

```ts
import { designSystem } from "./design-system.config";

export default {
  theme: {
    extend: {
      colors: designSystem.tokens.colors,
      spacing: designSystem.tokens.spacing,
    },
  },
};
```

### Unity プロジェクトの場合

`Assets/Scripts/UI/DesignSystem.cs` を生成する：

```csharp
public static class DesignSystem
{
    public static class Colors
    {
        public static readonly Color Primary = new Color32(37, 99, 235, 255);
        public static readonly Color PrimaryHover = new Color32(29, 78, 216, 255);
    }

    public static class Spacing
    {
        public const float SM = 8f;
        public const float MD = 16f;
        public const float LG = 24f;
    }
}
```

### 既存UIライブラリがある場合

既存ライブラリのトークンをSSOTに取り込む形にする。
ライブラリの値をSSOTが「再エクスポート」することで、将来的にライブラリを剥がす際の移行コストを下げる。

---

## Step 4: 既存コードを検証する

生成したSSOTに対して `AUDIT.md` を実行し、既存コードとのgapを確認する。
bootstrap直後は多くの⚠️・❌が出ることが想定される。全て即修正する必要はない。

---

## Step 5: CLAUDE.mdに記録する

プロジェクトの `CLAUDE.md` に以下を追記せよ：

```md
## Design System

SSOTは `design-system.config.ts`（またはスタックに応じたファイル）。
新しいスタイル値を追加する場合は必ずSSOTを先に更新すること。
詳細はデザインシステムスキルの `SKILL.md` を参照。
```
