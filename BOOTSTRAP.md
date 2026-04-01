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

**トークン構造の判断**
- 以下のいずれかに該当する場合、2層構造（値の定義 + セマンティックな役割割り当て）を推奨する：
  - ダークモード/テーマ切り替えがある
  - 複数プラットフォーム（Web + モバイル等）で同じデザイン言語を使う
  - チームが複数人で、デザイナーとの協業がある
- 上記に該当しない小規模プロジェクトでは、フラットなトークン定義で十分な場合がある
- コンポーネント固有のトークン層（3層目）は、共有UIライブラリを外部に公開する場合やホワイトラベル対応が必要な場合にのみ検討する

---

## Step 2: SSOTのフォーマットを決定する

スタックに応じて以下のフォーマットを選択する：

| スタック | 推奨フォーマット | ファイル名 |
|---------|----------------|----------|
| Tailwind v4 | CSS Custom Properties | `design-system.config.css` + `design-system.config.json` |
| Tailwind v3 | TypeScript / JS | `design-system.config.ts` |
| CSS Modules | CSS Custom Properties | `design-system.config.css` |
| Unity | C# static class | `DesignSystem.cs` |
| CSS-in-JS (styled-components, Emotion) | TypeScript | `design-system.config.ts` |
| Vue (SFC + scoped CSS) | CSS Custom Properties | `design-system.config.css` |
| スタック非依存 | JSON | `design-system.config.json` |
| 複合 | TypeScript（他形式を生成するスクリプト込み） | `design-system.config.ts` |

---

## Step 3: SSOTを生成する

### Tailwind v4 プロジェクトの場合

v4はCSS-firstのアプローチを取る。`design-system.config.css` をSSOTとし、`@theme` で値を定義、`:root` でセマンティックな役割を割り当てる2層構造にする：

```css
/* design-system.config.css */

/* --- Reference tokens（値の定義） --- */
@theme {
  --color-blue-500: oklch(0.55 0.20 260);
  --color-blue-700: oklch(0.40 0.20 260);
  --color-gray-50: oklch(0.98 0 0);
  --color-gray-900: oklch(0.15 0 0);
  --spacing-1: 0.25rem;
  --spacing-2: 0.5rem;
  --spacing-4: 1rem;
  --font-size-sm: 0.875rem;
  --font-size-base: 1rem;
}

/* --- System tokens（役割の割り当て） --- */
:root {
  --color-primary: var(--color-blue-500);
  --color-primary-hover: var(--color-blue-700);
  --color-surface: var(--color-gray-50);
  --color-on-surface: var(--color-gray-900);
}
.dark {
  --color-surface: var(--color-gray-900);
  --color-on-surface: var(--color-gray-50);
}
```

`app.css` での読み込み：

```css
@import "./design-system.config.css";
@import "tailwindcss";
```

conventions と antipatterns は `design-system.config.json` に分離する：

```json
{
  "conventions": {
    "componentDir": "src/components/ui",
    "namingPattern": "PascalCase"
  },
  "antipatterns": [
    "インラインスタイルの使用",
    "tailwind.configで定義されていない任意値 (text-[#xxx])",
    "コンポーネント外でのスタイル定義"
  ]
}
```

### Tailwind v3 プロジェクトの場合

`design-system.config.ts` をSSOTとし、`tailwind.config.ts` からimportする構成にする。2層構造はCSS Custom Propertiesとの併用で実現する：

```ts
// design-system.config.ts
export const designSystem = {
  // Reference tokens（値の定義）
  reference: {
    colors: {
      blue: { 500: "#2563eb", 700: "#1d4ed8" },
      gray: { 50: "#f9fafb", 900: "#111827" },
    },
    spacing: { 1: "0.25rem", 2: "0.5rem", 4: "1rem" },
    typography: {
      fontSize: { sm: "0.875rem", base: "1rem" },
    },
  },
  // System tokens（役割の割り当て）はCSS Custom Propertiesで定義する
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

```ts
// tailwind.config.ts
import { designSystem } from "./design-system.config";

export default {
  theme: {
    extend: {
      colors: designSystem.reference.colors,
      spacing: designSystem.reference.spacing,
    },
  },
};
```

```css
/* globals.css — System tokens */
:root {
  --color-primary: theme('colors.blue.500');
  --color-primary-hover: theme('colors.blue.700');
  --color-surface: theme('colors.gray.50');
  --color-on-surface: theme('colors.gray.900');
}
.dark {
  --color-surface: theme('colors.gray.900');
  --color-on-surface: theme('colors.gray.50');
}
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
