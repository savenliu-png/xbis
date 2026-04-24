请为本项目设计一套基础 Design Tokens，布局以当前项目的界面布局为主，包括：
颜色（主色、副色、成功、警告、错误）
字体（标题、正文）
间距（spacing scale）
圆角
阴影
要求适用于 SaaS + AI 产品风格，偏简洁、可信、现代。
其它要求：
- 当前项目可参考UI/styles.png样式
- 有品牌主色
- 有暗色模式需求
- 用户端和管理端共用 Token
- 输出需包括变量命名建议
- 输出需区分用户端 Token 与管理端 Token 的复用关系
- 输出需给出 CSS / Tailwind / SCSS 映射建议

执行结果必须沉淀成项目内的真实文件，例如：
- `tokens/colors.ts`
- `tokens/spacing.ts`
- `tokens/typography.ts`
- `docs/design-tokens.md`