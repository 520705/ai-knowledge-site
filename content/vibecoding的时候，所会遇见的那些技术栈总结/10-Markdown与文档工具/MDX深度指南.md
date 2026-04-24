# MDX 深度指南

> MDX 是 Markdown + JSX 的融合产物，它允许在 Markdown 文档中直接使用 React 组件，实现内容与交互的完美统一。本文全面解析 MDX 的核心概念、在主流框架中的使用、组件嵌入技术以及工具链生态。

## 目录

- [[#MDX 核心概念]]
- [[#在 Next.js 中的使用]]
- [[#在 Astro 中的使用]]
- [[#组件嵌入与交互]]
- [[#MDX 工具链]]
- [[#最佳实践]]
- [[#相关资源]]

---

## MDX 核心概念

### 什么是 MDX

MDX（Markdown + JSX）是一种可移植的文档格式，它将 Markdown 的简洁语法与 JSX 的强大组件能力结合在一起。MDX 文件本质上是一个 JavaScript 模块，导出一个 React 组件，可以在其中使用 Markdown 语法和任意 React 组件。

```mdx
import { Chart } from './components/Chart';

# 销售报告

这是一个交互式图表：

<Chart data={salesData} type="line" />

上面的图表展示了 Q1 的销售趋势。
```

### MDX 工作原理

MDX 的编译过程包含以下步骤：

```
MDX Source → MDX Compiler → JSX → React.createElement → DOM
```

1. **解析阶段**：MDX 编译器解析 Markdown 和 JSX 语法
2. **转换阶段**：将 Markdown 转换为 JSX 元素
3. **编译阶段**：生成可执行的 JavaScript 代码
4. **渲染阶段**：React 环境渲染最终组件

```javascript
// MDX 编译器核心 API
import { compile } from '@mdx-js/mdx';

const code = await compile(`# Hello World

<Button>Click me</Button>
`, {
  outputFormat: 'function-body',
  development: false
});

// 输出 JSX 代码字符串
console.log(code);
```

### MDX vs 标准 Markdown

| 特性 | Markdown | MDX |
|------|----------|-----|
| 组件嵌入 | 不支持 | 完全支持 |
| JavaScript 表达式 | 不支持 | 支持 |
| 导入导出 | 不支持 | 支持 |
| 样式系统 | CSS 类 | CSS-in-JS、组件 |
| 交互能力 | 静态 | 动态交互 |

```markdown
<!-- 标准 Markdown - 静态内容 -->
# 标题
这是一段文字

<!-- MDX - 可嵌入组件 -->
import { Chart } from './Chart';

# 标题
<Chart data={data} />
```

### MDX 2.0 新特性

MDX 2.0 带来了重大升级：

- **性能优化**：更快的编译速度和更小的产物
- **ESM 原生支持**：更好的模块化
- **增强的插件 API**：更灵活的扩展能力
- **改进的 MDX pragma**：更直观的配置方式

---

## 在 Next.js 中的使用

### Next.js App Router 中的 MDX

在 App Router 架构下，MDX 文件可以作为页面组件直接使用：

```typescript
// app/docs/page.mdx
import { Callout } from '@/components/ui/callout';
import { TOC } from '@/components/toc';

# 欢迎使用文档中心

<Callout type="info">
  这是一个重要的提示信息
</Callout>

## 目录

<TOC items={[
  { title: '安装', href: '#安装' },
  { title: '配置', href: '#配置' }
]} />

## 安装

首先安装必要依赖...
```

```typescript
// next.config.mjs
/** @type {import('next').NextConfig} */
const nextConfig = {
  pageExtensions: ['js', 'jsx', 'mdx', 'ts', 'tsx'],
  experimental: {
    mdxRs: true,  // 使用 Rust 编写的 MDX 编译器
  },
};

export default nextConfig;
```

### 使用 next-mdx-remote

`next-mdx-remote` 提供了更灵活的使用方式：

```typescript
// pages/blog/[slug].tsx
import { serialize } from 'next-mdx-remote/serialize';
import { MDXRemote } from 'next-mdx-remote/rsc';
import remarkGfm from 'remark-gfm';
import rehypePrism from 'rehype-prism';
import { components } from '@/components/mdx-components';

export async function function generateStaticParams() {
  const posts = await getAllPosts();
  return posts.map((post) => ({
    slug: post.slug,
  }));
}

export default async function Page({ params }: { params: { slug: string } }) {
  const post = await getPostBySlug(params.slug);
  const mdxSource = await serialize(post.content, {
    mdxOptions: {
      remarkPlugins: [remarkGfm],
      rehypePlugins: [rehypePrism],
    },
  });

  return (
    <article>
      <MDXRemote {...mdxSource} components={components} />
    </article>
  );
}
```

### MDX 内容集合

使用 Content Collections 管理 MDX：

```typescript
// lib/mdx.ts
import { compileMDX } from 'next-mdx-remote/rsc';
import { mdxComponents } from '@/components/mdx-components';

export async function getPost(slug: string) {
  const source = await fs.readFile(`content/posts/${slug}.mdx`);
  return compileMDX({
    source,
    components: mdxComponents,
    options: { parseFrontmatter: true },
  });
}

// types/blog.ts
import type { MDXRemoteProps } from 'next-mdx-remote/rsc';

export interface Post {
  title: string;
  date: string;
  author: string;
  tags: string[];
  excerpt: string;
}

export type PostFrontmatter = Post;

export type MdxContentProps = MDXRemoteProps<PostFrontmatter>;
```

---

## 在 Astro 中的使用

### Astro MDX 集成

Astro 提供了原生的 MDX 支持：

```bash
npx astro add mdx
```

```astro
---
// src/layouts/BaseLayout.astro
interface Props {
  title: string;
  description?: string;
}

const { title, description } = Astro.props;
---

<!DOCTYPE html>
<html>
  <head>
    <title>{title}</title>
    <meta name="description" content={description} />
  </head>
  <body>
    <main>
      <slot />
    </main>
  </body>
</html>
```

```mdx
---
// src/pages/docs/getting-started.mdx
layout: '../layouts/BaseLayout.astro'
title: 快速开始
description: 学习如何快速上手
---

# 快速开始

import { Steps } from '@/components/steps.astro';
import InfoCard from '@/components/InfoCard.astro';

<Steps>
  1. 安装依赖
  2. 配置项目
  3. 创建第一个页面
</Steps>

<InfoCard client:visible>
  这是一个可交互的卡片组件
</InfoCard>
```

### Astro 中的组件孤岛

Astro 的孤岛架构允许选择性激活交互组件：

```mdx
---
// 使用 client 指令控制水合策略
---

<!-- 默认不激活，仅静态渲染 -->
<StaticComponent />

<!-- 页面加载时激活 -->
<HydratedComponent client:load />

<!-- 可见时激活 -->
<InteractiveChart client:visible />

<!-- 仅在空闲时激活 -->
<HeavyWidget client:idle />

<!-- 仅在特定媒体查询时激活 -->
<MobileOnly client:media="(max-width: 768px)" />
```

```typescript
// components/InteractiveChart.tsx
import { useState, useEffect } from 'react';

interface ChartProps {
  data: number[];
  labels: string[];
}

export function InteractiveChart({ data, labels }: ChartProps) {
  const [mounted, setMounted] = useState(false);

  useEffect(() => {
    setMounted(true);
  }, []);

  if (!mounted) {
    return <div class="chart-placeholder">Loading chart...</div>;
  }

  return (
    <div class="chart-container">
      <canvas id="myChart" />
    </div>
  );
}
```

---

## 组件嵌入与交互

### MDX 组件设计模式

```typescript
// components/mdx-components.tsx
import type { MDXComponents } from 'mdx/types';
import { CodeBlock } from './code-block';
import { InfoBox } from './info-box';
import { ImageGrid } from './image-grid';
import { VideoPlayer } from './video-player';

export const mdxComponents: MDXComponents = {
  // 覆盖默认元素
  h1: (props) => <h1 class="mdx-h1" {...props} />,
  h2: (props) => <h2 class="mdx-h2" {...props} />,
  pre: (props) => <CodeBlock {...props} />,
  img: (props) => <ImageGrid images={[props]} />,

  // 自定义组件
  Callout: InfoBox,
  Video: VideoPlayer,
  Gallery: ImageGrid,

  // 代码高亮主题
  code: ({ children, className, ...props }) => {
    const language = className?.replace('language-', '');
    return (
      <CodeBlock language={language} {...props}>
        {children}
      </CodeBlock>
    );
  },
};
```

### 可交互组件示例

```typescript
// components/ColorPicker.tsx
import { useState } from 'react';

interface ColorPickerProps {
  defaultColor?: string;
  onChange?: (color: string) => void;
}

export function ColorPicker({ 
  defaultColor = '#3b82f6',
  onChange 
}: ColorPickerProps) {
  const [color, setColor] = useState(defaultColor);
  const [history, setHistory] = useState<string[]>([defaultColor]);

  const handleChange = (newColor: string) => {
    setColor(newColor);
    setHistory(prev => [...prev.slice(-9), newColor]);
    onChange?.(newColor);
  };

  return (
    <div class="color-picker">
      <div 
        class="preview" 
        style={{ backgroundColor: color }}
      >
        {color}
      </div>
      
      <input
        type="color"
        value={color}
        onChange={(e) => handleChange(e.target.value)}
      />
      
      <div class="history">
        {history.map((c, i) => (
          <button
            key={i}
            onClick={() => setColor(c)}
            style={{ backgroundColor: c }}
            aria-label={`选择颜色 ${c}`}
          />
        ))}
      </div>
    </div>
  );
}
```

```mdx
import { ColorPicker } from '@/components/color-picker';

# 颜色选择器演示

选择一个你喜欢的颜色：

<ColorPicker 
  defaultColor="#6366f1"
  onChange={(color) => console.log('选中的颜色:', color)}
/>

你也可以点击历史记录快速选择之前使用过的颜色。
```

### 动态内容加载

```typescript
// components/AsyncContent.tsx
import { useState, useEffect } from 'react';

interface AsyncContentProps {
  fetchUrl: string;
  fallback?: React.ReactNode;
  render: (data: any) => React.ReactNode;
}

export function AsyncContent({ 
  fetchUrl, 
  fallback = <p>加载中...</p>,
  render 
}: AsyncContentProps) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState<Error | null>(null);

  useEffect(() => {
    setLoading(true);
    fetch(fetchUrl)
      .then(res => res.json())
      .then(setData)
      .catch(setError)
      .finally(() => setLoading(false));
  }, [fetchUrl]);

  if (loading) return fallback;
  if (error) return <p>加载失败: {error.message}</p>;
  return render(data);
}
```

---

## MDX 工具链

### 核心编译工具

```bash
# 核心包
npm install @mdx-js/mdx @mdx-js/rollup remark-mdx

# GFM 支持
npm install remark-gfm

# 代码高亮
npm install rehype-prism @shikijs/rehype
```

```javascript
// mdx.config.js
import { mdxAnnotations } from 'mdx-annotations';
import remarkGfm from 'remark-gfm';
import rehypeShiki from '@shikijs/rehype';

export default {
  plugins: [
    // 代码块注释语法支持
    mdxAnnotations.remark,
    // GitHub Flavored Markdown
    remarkGfm,
    // Shiki 代码高亮
    [rehypeShiki, { theme: 'github-dark' }],
  ],
};
```

### Velite：下一代内容框架

Velite 是一个声明式的内容处理工具：

```typescript
// velite.config.ts
import { defineConfig, defineCollection, s } from 'velite';

const posts = defineCollection({
  name: 'Post',
  pattern: 'posts/**/*.mdx',
  schema: s.object({
    title: s.string(),
    date: s.date(),
    tags: s.array(s.string()),
    draft: s.boolean().default(false),
  }),
  transform: (post) => ({
    ...post,
    slug: post._meta.filename.replace('.mdx', ''),
  }),
});

export default defineConfig({
  collections: { posts },
  output: {
    path: '.velite',
    name: 'index',
  },
});
```

### MDX 与样式系统

```typescript
// Tailwind CSS 集成
/** @type {import('tailwindcss').Config} */
export default {
  content: [
    './**/*.mdx',
    './src/**/*.{js,ts,jsx,tsx}',
  ],
  theme: {
    extend: {
      typography: {
        DEFAULT: {
          css: {
            '--tw-prose-body': '#334155',
            '--tw-prose-headings': '#0f172a',
          },
        },
      },
    },
  },
  plugins: [
    require('@tailwindcss/typography'),
  ],
};
```

```mdx
<!-- 使用 prose 样式 -->
<div class="prose prose-slate max-w-none">
  <h1>标题</h1>
  <p>正文内容...</p>
</div>
```

---

## 最佳实践

### 项目结构组织

```
project/
├── content/
│   ├── docs/
│   │   ├── getting-started.mdx
│   │   ├── configuration.mdx
│   │   └── api-reference.mdx
│   └── blog/
│       └── *.mdx
├── components/
│   ├── mdx/
│   │   ├── Callout.tsx
│   │   ├── CodeBlock.tsx
│   │   └── Table.tsx
│   └── ui/
├── lib/
│   └── mdx.ts
└── mdx.config.js
```

### 性能优化

> [!TIP] MDX 性能优化建议
> - 对静态内容使用 SSG，避免不必要的客户端水合
> - 使用 `next-mdx-remote` 的 RSC 版本减少包体积
> - 大型组件使用 `React.lazy` 延迟加载
> - 利用 `client:idle` 和 `client:visible` 指令控制激活时机

```typescript
// 延迟加载大型组件
const HeavyChart = lazy(() => import('./HeavyChart'));

export function MDXContent({ content }) {
  return (
    <div>
      {content}
      <Suspense fallback={<ChartSkeleton />}>
        <HeavyChart />
      </Suspense>
    </div>
  );
}
```

---

## 相关资源

- [[Next.js 文档工具链]]
- [[Astro 内容管理]]
- [[React 组件设计]]
- [[MDX 官方文档]]
- [[Shiki 代码高亮]]

---

*本文档由 [[归愚知识系统]] 自动生成*

---

## MDX 高级应用

### MDX 编译器深度解析

MDX 的编译过程是一个复杂的多阶段转换，理解其内部机制对于优化性能和解决问题至关重要。

```typescript
// MDX 编译过程详解
import { compile, run } from '@mdx-js/mdx';
import * as runtime from 'react/jsx-runtime';
import remarkParse from 'remark-parse';
import remarkRehype from 'remark-rehype';
import rehypeStringify from 'rehype-stringify';
import remarkGfm from 'remark-gfm';
import rehypePrism from 'rehype-prism';

// 原始 MDX 内容
const mdxSource = `
import { Chart } from './Chart';

# 销售报告

这是一个交互式图表：

<Chart data={salesData} type="line" />
`;

// 完整编译流程
async function compileMDX(source: string) {
  // 阶段1: 解析 MDX
  // 输入: MDX 字符串
  // 输出: MDX AST (mdast)
  const mdast = await remarkParse()(source);
  
  // 阶段2: 转换 Remark 插件
  // 处理 GFM 语法、表格、脚注等
  const processedMdst = await remarkGfm()(mdast);
  
  // 阶段3: Remark to Rehype
  // 将 Markdown AST 转换为 HTML AST
  const hast = await remarkRehype()(processedMdst);
  
  // 阶段4: 转换 Rehype 插件
  // 处理代码高亮、ID 生成等
  const processedHast = await rehypePrism()(hast);
  
  // 阶段5: 生成 HTML
  const html = await rehypeStringify()(processedHast);
  
  return html;
}

// MDX 专用编译
async function compileToJSX(source: string) {
  // 编译为可执行的 JSX 代码
  const code = await compile(source, {
    outputFormat: 'function-body',
    development: false,
  });
  
  // code 现在是 JSX 字符串，可以：
  // 1. 使用 @mdx-js/react 的 MDXProvider 渲染
  // 2. 使用 babel 编译运行
  // 3. 使用 esbuild 编译运行
  
  return code;
}

// 运行时执行 MDX
async function executeMDX(source: string, scope = {}) {
  const code = await compile(source, {
    outputFormat: 'function-body',
    development: false,
  });
  
  const React = await import('react');
  const runtimeCode = `
    const { jsx: _jsx, jsxs: _jsxs, Fragment: _Fragment } = arguments[0];
  `;
  
  const fn = new Function(
    'React', 
    '_jsx', '_jsxs', '_Fragment',
    ...Object.keys(scope),
    runtimeCode + code
  );
  
  return fn(
    React.default,
    React.jsx,
    React.jsxs,
    React.Fragment,
    ...Object.values(scope)
  );
}
```

### MDX 插件系统

```typescript
// 自定义 Remark 插件
function remarkCustomDirective() {
  return (tree: mdast.Root) => {
    visit(tree, 'paragraph', (node, index, parent) => {
      const children = node.children;
      if (children.length === 1 && children[0].type === 'text') {
        const text = children[0].value;
        
        // 处理 :::warning 语法
        if (text.startsWith(':::warning')) {
          const content = text.replace(':::warning', '').trim();
          
          const container: mdast.ContainerDirective = {
            type: 'containerDirective',
            name: 'warning',
            children: [
              {
                type: 'paragraph',
                children: [{ type: 'text', value: content }],
              },
            ],
          };
          
          parent!.children.splice(index!, 1, container);
        }
      }
    });
  };
}

// 自定义 Rehype 插件
function rehypeAddCopyButton() {
  return (tree: hast.Root) => {
    visit(tree, 'element', (node) => {
      if (node.tagName === 'pre') {
        // 找到代码块
        const codeNode = node.children.find(
          (n): n is hast.Element => 
            (n as hast.Element).tagName === 'code'
        );
        
        if (codeNode) {
          // 添加复制按钮容器
          const wrapper: hast.Element = {
            type: 'element',
            tagName: 'div',
            properties: { className: ['code-block-wrapper'] },
            children: [
              node,
              {
                type: 'element',
                tagName: 'button',
                properties: {
                  className: ['copy-button'],
                  'data-copy': 'true',
                },
                children: [{ type: 'text', value: '复制' }],
              },
            ],
          };
          
          // 替换原节点
          Object.assign(node, {
            tagName: 'div',
            properties: { className: ['code-block-inner'] },
          });
        }
      }
    });
  };
}

// 组合插件
const combinedPlugins = [
  remarkCustomDirective,
  [remarkGfm, { singleLineCodeFrames: true }],
  remarkRehype,
  rehypePrism,
  [rehypeAddCopyButton],
];
```

### MDX 作用域与上下文

```typescript
// MDX 作用域管理
interface MDXScope {
  // 组件
  components: Record<string, React.ComponentType>;
  // 工具函数
  utils: Record<string, Function>;
  // 数据
  data: Record<string, unknown>;
  // 主题
  theme: ThemeConfig;
}

// 提供作用域给 MDX
function createMDXProvider(scope: MDXScope) {
  return function MDXContent(props: MDXContentProps) {
    return (
      <MDXProvider components={scope.components}>
        <ScopeProvider value={scope}>
          <MDXRemote {...props} />
        </ScopeProvider>
      </MDXProvider>
    );
  };
}

// 使用 useMDXScope 访问作用域
function useMDXScope() {
  const context = useContext(ScopeContext);
  if (!context) {
    throw new Error('useMDXScope must be used within MDXScopeProvider');
  }
  return context;
}

// 在 MDX 组件中使用
const CustomChart = () => {
  const { data, theme } = useMDXScope();
  
  return (
    <Chart 
      data={data.chartData} 
      theme={theme.colors}
    />
  );
};
```

### MDX 内容管理策略

```typescript
// 内容集合类型定义
interface ContentCollection<T extends Frontmatter = Frontmatter> {
  // 所有文档
  all: () => Promise<CollectionEntry<T>[]>;
  // 按 slug 查询
  getBySlug: (slug: string) => Promise<CollectionEntry<T> | null>;
  // 按标签筛选
  getByTag: (tag: string) => Promise<CollectionEntry<T>[]>;
  // 按日期范围筛选
  getByDateRange: (
    start: Date,
    end: Date
  ) => Promise<CollectionEntry<T>[]>;
  // 搜索
  search: (query: string) => Promise<SearchResult<T>[]>;
}

// 内容集合实现
class MDXContentCollection<T extends Frontmatter> 
  implements ContentCollection<T> {
  
  private directory: string;
  private glob: string;
  private parseFrontmatter: boolean;
  
  constructor(options: {
    directory: string;
    pattern?: string;
    parseFrontmatter?: boolean;
  }) {
    this.directory = options.directory;
    this.glob = options.pattern || '**/*.mdx';
    this.parseFrontmatter = options.parseFrontmatter ?? true;
  }
  
  async all(): Promise<CollectionEntry<T>[]> {
    const files = await glob(`${this.directory}/${this.glob}`);
    
    return Promise.all(
      files.map(async (file) => {
        const source = await fs.readFile(file, 'utf-8');
        const { content, frontmatter } = this.parseFrontmatter
          ? this.parse(source)
          : { content: source, frontmatter: {} };
        
        const slug = this.getSlugFromPath(file);
        
        return {
          slug,
          source,
          content,
          frontmatter: frontmatter as T,
          meta: {
            path: file,
            createdAt: await this.getCreatedAt(file),
            modifiedAt: await this.getModifiedAt(file),
          },
        };
      })
    );
  }
  
  async getBySlug(slug: string): Promise<CollectionEntry<T> | null> {
    const all = await this.all();
    return all.find((entry) => entry.slug === slug) || null;
  }
  
  async getByTag(tag: string): Promise<CollectionEntry<T>[]> {
    const all = await this.all();
    return all.filter((entry) =>
      entry.frontmatter.tags?.includes(tag)
    );
  }
  
  async search(query: string): Promise<SearchResult<T>[]> {
    const all = await this.all();
    const lowerQuery = query.toLowerCase();
    
    return all
      .map((entry) => {
        const titleMatch = entry.frontmatter.title
          ?.toLowerCase()
          .includes(lowerQuery);
        const contentMatch = entry.content
          .toLowerCase()
          .includes(lowerQuery);
        
        return {
          ...entry,
          score: (titleMatch ? 2 : 0) + (contentMatch ? 1 : 0),
        };
      })
      .filter((result) => result.score > 0)
      .sort((a, b) => b.score - a.score);
  }
  
  private getSlugFromPath(file: string): string {
    return file
      .replace(this.directory, '')
      .replace(/\.mdx$/, '')
      .replace(/^\//, '');
  }
  
  private parse(source: string): { 
    content: string; 
    frontmatter: Record<string, unknown>;
  } {
    const match = source.match(/^---\n([\s\S]*?)\n---\n([\s\S]*)$/);
    
    if (!match) {
      return { content: source, frontmatter: {} };
    }
    
    const [, frontmatterYaml, content] = match;
    const frontmatter = yaml.parse(frontmatterYaml);
    
    return { content, frontmatter };
  }
}

// 使用示例
const posts = new MDXContentCollection<PostFrontmatter>({
  directory: './content/posts',
  pattern: '**/*.mdx',
});

// 获取所有文章
const allPosts = await posts.all();

// 获取单个文章
const post = await posts.getBySlug('getting-started-with-mdx');

// 按标签查询
const reactPosts = await posts.getByTag('react');

// 搜索
const results = await posts.search('MDX');
```

---

## MDX 性能优化

### 编译时优化

```typescript
// Vite 配置优化
// vite.config.ts
export default defineConfig({
  optimizeDeps: {
    include: ['react', 'react-dom', '@mdx-js/react'],
  },
  
  build: {
    // 分块策略
    rollupOptions: {
      output: {
        manualChunks: {
          // 将 MDX 运行时单独打包
          'mdx-runtime': ['@mdx-js/react', '@mdx-js/mdx'],
          // 按内容类型分块
          'mdx-components': glob.sync('./components/**/*.tsx'),
        },
      },
    },
    
    // 压缩选项
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },
  
  // MDX 编译选项
  mdx: {
    compileOptions: {
      providerImportSource: '@mdx-js/react',
    },
    remarkPlugins: [],
    rehypePlugins: [],
  },
});

// Next.js 配置优化
// next.config.js
module.exports = {
  experimental: {
    // 使用 Rust 编写的 MDX 编译器
    mdxRs: true,
  },
  
  images: {
    formats: ['image/avif', 'image/webp'],
  },
};
```

### 运行时优化

```typescript
// 懒加载大型组件
const HeavyDataTable = lazy(() => import('./HeavyDataTable'));
const InteractiveMap = lazy(() => import('./InteractiveMap'));

function MDXContent({ source }) {
  return (
    <MDXRemote
      source={source}
      components={{
        DataTable: (props) => (
          <Suspense fallback={<TableSkeleton />}>
            <HeavyDataTable {...props} />
          </Suspense>
        ),
        Map: (props) => (
          <Suspense fallback={<MapSkeleton />}>
            <InteractiveMap {...props} />
          </Suspense>
        ),
      }}
    />
  );
}

// 虚拟化长列表
import { FixedSizeList } from 'react-window';

function VirtualizedList({ items }) {
  const Row = ({ index, style }) => (
    <div style={style}>
      <ListItem item={items[index]} />
    </div>
  );
  
  return (
    <FixedSizeList
      height={400}
      itemCount={items.length}
      itemSize={50}
    >
      {Row}
    </FixedSizeList>
  );
}

// 代码分割策略
// 方案1: 动态导入
const DynamicChart = dynamic(
  () => import('./components/Chart'),
  { 
    ssr: false,
    loading: () => <ChartSkeleton />
  }
);

// 方案2: 按需加载组件
function useAsyncComponent(loader: () => Promise<any>) {
  const [Component, setComponent] = useState(null);
  
  useEffect(() => {
    loader().then((mod) => {
      setComponent(() => mod.default);
    });
  }, [loader]);
  
  return Component || <Skeleton />;
}

// 方案3: 预加载关键组件
function preloadComponent(path: string) {
  return import(/* webpackPrefetch: true */ path);
}
```

### 缓存策略

```typescript
// 静态生成缓存
export async function generateStaticParams() {
  const posts = await getAllPosts();
  
  return posts.map((post) => ({
    slug: post.slug,
  }));
}

// ISR (增量静态再生成)
export const revalidate = 3600; // 每小时重新验证

// 客户端缓存
function useMDXContent(slug: string) {
  return useSWR(
    `/api/mdx/${slug}`,
    fetcher,
    {
      revalidateOnFocus: false,
      revalidateOnReconnect: true,
      dedupingInterval: 60000,
      fallbackData: null,
    }
  );
}

// SWR 配置
const swrConfig = {
  refreshInterval: 0,           // 不自动刷新
  revalidateOnFocus: false,     // 失焦时不重新验证
  revalidateOnReconnect: true,   // 断网重连时验证
  shouldRetryOnError: false,     // 错误时不重试
  dedupingInterval: 2000,        // 2秒内去重
  loadingTimeout: 3000,          // 3秒后显示 loading
};
```

---

## MDX 与内容平台集成

### Notion 集成

```typescript
// Notion API 集成
import { Client } from '@notionhq/client';
import { notRichTextToString } from '@notionhq/client/build/src/helpers';

class NotionToMDXConverter {
  private notion: Client;
  
  constructor(accessToken: string) {
    this.notion = new Client({ auth: accessToken });
  }
  
  async convertPage(pageId: string): Promise<string> {
    const page = await this.notion.pages.retrieve({ page_id: pageId });
    const blocks = await this.notion.blocks.children.list({
      block_id: pageId,
    });
    
    let mdxContent = `---\n`;
    mdxContent += `title: "${notRichTextToString(page.properties.Name?.title || [])}"\n`;
    mdxContent += `createdAt: "${page.created_time}"\n`;
    mdxContent += `lastEdited: "${page.last_edited_time}"\n`;
    mdxContent += `---\n\n`;
    
    for (const block of blocks.results) {
      mdxContent += this.convertBlock(block);
    }
    
    return mdxContent;
  }
  
  private convertBlock(block: BlockObjectResponse): string {
    switch (block.type) {
      case 'paragraph':
        return `${notRichTextToString(block.paragraph.rich_text)}\n\n`;
      
      case 'heading_1':
        return `## ${notRichTextToString(block.heading_1.rich_text)}\n\n`;
      
      case 'heading_2':
        return `### ${notRichTextToString(block.heading_2.rich_text)}\n\n`;
      
      case 'bulleted_list_item':
        return `- ${notRichTextToString(block.bulleted_list_item.rich_text)}\n`;
      
      case 'numbered_list_item':
        return `1. ${notRichTextToString(block.numbered_list_item.rich_text)}\n`;
      
      case 'code':
        return `\`\`\`${block.code.language}\n${notRichTextToString(block.code.rich_text)}\n\`\`\`\n\n`;
      
      case 'image':
        const imageUrl = block.image.type === 'external' 
          ? block.image.external.url 
          : block.image.file.url;
        return `![${notRichTextToString(block.image.caption)}](${imageUrl})\n\n`;
      
      case 'callout':
        return `> [!NOTE]\n> ${notRichTextToString(block.callout.rich_text)}\n\n`;
      
      default:
        return '';
    }
  }
  
  async syncDatabase(databaseId: string, outputDir: string) {
    const pages = await this.notion.databases.query({
      database_id: databaseId,
      filter: {
        property: 'Status',
        select: { equals: 'Published' },
      },
      sorts: [{ property: 'Date', direction: 'descending' }],
    });
    
    for (const page of pages.results) {
      if ('id' in page) {
        const mdxContent = await this.convertPage(page.id);
        const slug = this.generateSlug(page);
        const filename = `${outputDir}/${slug}.mdx`;
        
        await fs.writeFile(filename, mdxContent);
      }
    }
  }
}
```

### Contentful 集成

```typescript
// Contentful 集成
import { createClient } from 'contentful';
import { documentToReactComponents } from '@contentful/rich-text-react-renderer';

class ContentfulToMDXConverter {
  private client: ContentfulClientApi;
  
  constructor(config: {
    spaceId: string;
    accessToken: string;
  }) {
    this.client = createClient(config);
  }
  
  async fetchEntries(
    contentType: string,
    options: {
      limit?: number;
      skip?: number;
      order?: string[];
    } = {}
  ) {
    const entries = await this.client.getEntries({
      content_type: contentType,
      ...options,
    });
    
    return entries.items.map((item) => this.convertEntry(item));
  }
  
  private convertEntry(item: Entry): MDXEntry {
    const fields = item.fields as Record<string, any>;
    
    return {
      slug: fields.slug,
      title: fields.title,
      content: this.richTextToMDX(fields.content),
      publishedAt: fields.publishDate || item.sys.createdAt,
      tags: fields.tags || [],
      author: fields.author?.fields?.name,
      heroImage: fields.heroImage?.fields?.file?.url,
    };
  }
  
  private richTextToMDX(node: Document): string {
    return documentToReactComponents(
      node,
      {
        // 自定义渲染器
        BLOCKS: {
          [BLOCKS.PARAGRAPH]: (node, children) => `${children}\n\n`,
          [BLOCKS.HEADING_1]: (node, children) => `# ${children}\n\n`,
          [BLOCKS.HEADING_2]: (node, children) => `## ${children}\n\n`,
          [BLOCKS.UL_LIST]: (node, children) => `${children}\n`,
          [BLOCKS.OL_LIST]: (node, children) => `${children}\n`,
          [BLOCKS.LIST_ITEM]: (node, children) => `- ${children}\n`,
          [BLOCKS.QUOTE]: (node, children) => `> ${children}\n\n`,
          [BLOCKS.CODE]: (node) => {
            const code = node.content[0];
            return `\`\`\`${code.language}\n${code.content[0].value}\n\`\`\`\n\n`;
          },
          [BLOCKS.EMBEDDED_ASSET]: (node) => {
            const asset = node.data.target;
            return `![](${asset.fields.file.url})\n\n`;
          },
        },
        MARKS: {
          [MARKS.BOLD]: (text) => `**${text}**`,
          [MARKS.ITALIC]: (text) => `*${text}*`,
          [MARKS.CODE]: (text) => `\`${text}\``,
        },
      }
    );
  }
}
```

### Sanity CMS 集成

```typescript
// Sanity 集成
import { createClient } from '@sanity/client';
import { portableTextToMarkdown } from '@sanity/portable-text-to-markdown';

class SanityToMDXConverter {
  private client: SanityClient;
  
  constructor(config: {
    projectId: string;
    dataset: string;
    token?: string;
  }) {
    this.client = createClient({
      ...config,
      useCdn: false, // 开发环境不使用 CDN
      apiVersion: '2024-01-01',
    });
  }
  
  async fetchPost(slug: string): Promise<MDXPost> {
    const query = `*[_type == "post" && slug.current == $slug][0] {
      title,
      slug,
      publishedAt,
      "author": author->name,
      "categories": categories[]->title,
      body
    }`;
    
    const post = await this.client.fetch(query, { slug });
    return this.convertPost(post);
  }
  
  private async convertPost(post: SanityPost): Promise<MDXPost> {
    const body = await portableTextToMarkdown(post.body);
    
    return {
      frontmatter: {
        title: post.title,
        date: post.publishedAt,
        author: post.author,
        categories: post.categories,
      },
      content: body,
    };
  }
  
  // GROQ 查询构建器
  buildQuery(options: {
    type: string;
    filters?: string[];
    projection?: string;
  }): string {
    const { type, filters = [], projection } = options;
    
    let query = `*[_type == "${type}"`;
    
    if (filters.length > 0) {
      query += ` && [${filters.join(' && ')}]`;
    }
    
    query += `]`;
    
    if (projection) {
      query += ` { ${projection} }`;
    }
    
    return query;
  }
}
```

---

## MDX 生态系统工具链

### 开发工具配置

```json
// .vscode/settings.json
{
  "files.associations": {
    "*.mdx": "markdown"
  },
  "[markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.formatOnSave": true,
    "editor.wordWrap": "on"
  },
  "markdown.validate.enabled": true,
  "typescript.tsdk": "node_modules/typescript/lib"
}
```

```json
// .vscode/extensions.json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "christian-kohler.path-intellisense",
    "vunguyentuan.vscode-mdx",
    "mutantdino.resourcemonitor"
  ]
}
```

### ESLint 配置

```javascript
// .eslintrc.js
module.exports = {
  extends: [
    'next/core-web-vitals',
    'plugin:mdx/recommended',
  ],
  
  plugins: ['mdx'],
  
  rules: {
    'mdx/no-unescaped-entities': 'off',
    'mdx/remark': [
      'error',
      {
        pluginPaths: [
          'remark-gfm',
          'remark-frontmatter',
          'remark-mdx-frontmatter',
        ],
      },
    ],
  },
  
  overrides: [
    {
      files: ['*.mdx'],
      parser: 'eslint-plugin-mdx/dist/parser',
      rules: {
        'no-unused-vars': 'off',
      },
    },
  ],
};
```

### Prettier 配置

```javascript
// .prettierrc.js
module.exports = {
  // MDX 文件使用 MDX 插件
  plugins: [
    require('prettier-plugin-mdx'),
    require('@prettier/plugin-sort-imports'),
  ],
  
  // MDX 特定选项
  semi: false,
  singleQuote: true,
  trailingComma: 'all',
  printWidth: 80,
  
  // MDX 格式选项
  mdxLanguageEngine: 'remark',
  mdxFileExtensions: ['mdx'],
};
```

```javascript
// prettier-plugin-mdx 配置
const mdxConfig = {
  // 导入排序
  importOrder: [
    '^@/(.*)$',
    '^@/components/(.*)$',
    '^[./]',
  ],
  
  // MDX 代码块格式
  mdxLanguageEngine: {
    // 使用 remark 或 mdast
    parser: 'remark',
    // 保留空白
    preserveWhitespace: false,
    // 表格对齐
    tableAlignment: true,
  },
};
```

### 测试工具

```typescript
// Vitest 配置
// vitest.config.ts
import { defineConfig } from 'vitest/config';
import mdx from 'vite-plugin-mdx';

export default defineConfig({
  plugins: [
    mdx({
      MDXProvider: require.resolve('@mdx-js/react'),
    }),
  ],
  
  test: {
    environment: 'jsdom',
    globals: true,
    setupFiles: ['./test/setup.ts'],
    
    // MDX 特定测试配置
    coverage: {
      provider: 'v8',
      reporter: ['text', 'json', 'html'],
      include: ['**/*.mdx'],
    },
  },
});
```

```typescript
// 测试 MDX 组件
// __tests__/MyComponent.test.tsx
import { describe, it, expect, vi } from 'vitest';
import { render, screen, fireEvent } from '@testing-library/react';
import { MDXProvider } from '@mdx-js/react';

// 测试 MDX 内容
describe('MDX Content Tests', () => {
  it('should render heading', () => {
    render(<TestComponent>{"# Hello World"}</TestComponent>);
    expect(screen.getByRole('heading', { level: 1 })).toHaveTextContent('Hello World');
  });
  
  it('should render interactive component', async () => {
    const handleClick = vi.fn();
    
    render(
      <MDXProvider>
        <TestComponent onClick={handleClick}>
          {"<Counter />"}
        </TestComponent>
      </MDXProvider>
    );
    
    const button = screen.getByRole('button');
    fireEvent.click(button);
    
    expect(handleClick).toHaveBeenCalledTimes(1);
  });
});

// 快照测试
describe('MDX Snapshot Tests', () => {
  it('should match expected output', () => {
    const { container } = render(<Article source={mdxContent} />);
    expect(container).toMatchSnapshot();
  });
  
  it('should match serialized MDX', () => {
    const result = serialize(mdxContent);
    expect(result).toMatchSnapshot();
  });
});
```

---

## MDX 高级模式

### MDX 与微前端

```typescript
// 微前端 MDX 加载器
class MFEMDXLoader {
  private registry: Map<string, () => Promise<any>> = new Map();
  
  register(name: string, loader: () => Promise<any>) {
    this.registry.set(name, loader);
  }
  
  async loadComponent(name: string): Promise<React.ComponentType> {
    const loader = this.registry.get(name);
    if (!loader) {
      throw new Error(`Component ${name} not found in registry`);
    }
    
    const module = await loader();
    return module.default;
  }
  
  async renderMDXWithRemote(
    source: string,
    remoteComponents: Record<string, string>
  ): Promise<React.ReactElement> {
    const components: Record<string, React.ComponentType> = {};
    
    // 加载远程组件
    await Promise.all(
      Object.entries(remoteComponents).map(async ([name, url]) => {
        components[name] = await this.loadRemoteComponent(url);
      })
    );
    
    // 使用 MDX 渲染
    return <MDXRemote source={source} components={components} />;
  }
}

// 注册远程组件
const mdxLoader = new MFEMDXLoader();

mdxLoader.register('DataGrid', () => 
  import('dataGridMFE/DataGrid')
);

mdxLoader.register('Chart', () => 
  import('chartsMFE/Charts')
);
```

### MDX 与 GraphQL

```typescript
// GraphQL 查询集成
const POST_QUERY = gql`
  query GetPost($slug: String!) {
    post(slug: $slug) {
      title
      content
      author {
        name
        avatar
      }
      tags
      relatedPosts {
        slug
        title
      }
    }
  }
`;

// 带 GraphQL 的 MDX 渲染
function GraphQLMDXPage({ slug }) {
  const { data, loading, error } = useQuery(POST_QUERY, { variables: { slug } });
  
  if (loading) return <Loading />;
  if (error) return <Error error={error} />;
  
  return (
    <MDXProvider components={customComponents}>
      <article>
        <h1>{data.post.title}</h1>
        <MDXRemote source={data.post.content} />
      </article>
      
      <aside>
        <RelatedPosts posts={data.post.relatedPosts} />
      </aside>
    </MDXProvider>
  );
}

// MDX 中使用 GraphQL
const MDX_WITH_GRAPHQL = `
import { useQuery, gql } from '@apollo/client';

export const AuthorQuery = gql\`
  query GetAuthor($id: ID!) {
    author(id: $id) {
      name
      bio
    }
  }
\`;

# 作者介绍

<AuthorCard authorId="author-123" />

function AuthorCard({ authorId }) {
  const { data } = useQuery(AuthorQuery, { 
    variables: { id: authorId } 
  });
  
  if (!data) return <Loading />;
  
  return (
    <div className="author-card">
      <h3>{data.author.name}</h3>
      <p>{data.author.bio}</p>
    </div>
  );
}
`;
```

### MDX 与状态管理

```typescript
// MDX 上下文状态管理
const MDXContext = createContext<{
  state: Record<string, any>;
  setState: (key: string, value: any) => void;
}>({
  state: {},
  setState: () => {},
});

function MDXStateProvider({ 
  children, 
  initialState = {} 
}: MDXStateProviderProps) {
  const [state, setState] = useState(initialState);
  
  const contextValue = useMemo(() => ({
    state,
    setState: (key: string, value: any) => 
      setState(prev => ({ ...prev, [key]: value })),
  }), [state]);
  
  return (
    <MDXContext.Provider value={contextValue}>
      {children}
    </MDXContext.Provider>
  );
}

// MDX 内容中使用状态
const MDX_WITH_STATE = `
import { useContext } from 'react';
import { MDXContext } from './context';

# 计数器示例

<Counter />

function Counter() {
  const { state, setState } = useContext(MDXContext);
  const count = state.count || 0;
  
  return (
    <div>
      <p>计数: {count}</p>
      <button onClick={() => setState('count', count + 1)}>
        增加
      </button>
      <button onClick={() => setState('count', 0)}>
        重置
      </button>
    </div>
  );
}
`;
```

---

## MDX 常见问题与解决方案

### 问题1：MDX 在 SSR 中 hydration 不匹配

```typescript
// 问题原因：服务端和客户端渲染结果不一致
// 解决方案：使用 client-only 标记

// ClientOnly 组件
function ClientOnly({ 
  children, 
  fallback = null 
}: ClientOnlyProps) {
  const [mounted, setMounted] = useState(false);
  
  useEffect(() => {
    setMounted(true);
  }, []);
  
  if (!mounted) {
    return fallback;
  }
  
  return children;
}

// 使用
<ClientOnly fallback={<Skeleton />}>
  <InteractiveComponent />
</ClientOnly>
```

### 问题2：大文件 MDX 编译缓慢

```typescript
// 问题：大型 MDX 文件编译时间过长
// 解决方案：拆分为多个小文件 + 使用 content collections

// 原来的大文件
// posts/large-post.mdx (10000行)

// 拆分后
// posts/large-post/index.mdx (主文件)
// posts/large-post/part1.mdx
// posts/large-post/part2.mdx
// posts/large-post/part3.mdx

// 主文件使用导入
// posts/large-post/index.mdx
import Part1 from './part1.mdx';
import Part2 from './part2.mdx';
import Part3 from './part3.mdx';

# 大型文章

<Part1 />

<Part2 />

<Part3 />
```

### 问题3：MDX 中使用三方库导致 bundle 过大

```typescript
// 问题：MDX 中直接 import 图表库等重型依赖
// 解决方案：动态导入 + 代码分割

// 方案1：动态导入
// 不要这样写：
// import { Chart } from 'heavy-chart';

// 而是这样：
const Chart = dynamic(() => import('./Chart'));

// 方案2：使用 client-only 组件
// Chart.client.tsx
export function Chart(props) {
  const [ChartComponent, setChartComponent] = useState(null);
  
  useEffect(() => {
    import('recharts').then((mod) => {
      setChartComponent(() => mod.LineChart);
    });
  }, []);
  
  if (!ChartComponent) return <Skeleton />;
  
  return <ChartComponent {...props} />;
}

// 方案3：使用 CDN 外链
// 在 MDX 中使用 HTML
<div id="chart-container" data-type="line" />
<script src="https://cdn.example.com/chart.js" />
```

### 问题4：MDX 中使用 useState 导致警告

```typescript
// 问题：组件在服务端渲染时报错
// 原因：useState 在 SSR 时行为不一致

// 解决方案：始终在 useEffect 中初始化状态

// 错误写法
function Counter() {
  const [count, setCount] = useState(0); // SSR 时可能有警告
  return <div>{count}</div>;
}

// 正确写法
function Counter() {
  const [count, setCount] = useState(null);
  
  // 确保只在客户端执行
  useEffect(() => {
    setCount(0);
  }, []);
  
  if (count === null) return <Skeleton />;
  
  return <div>{count}</div>;
}

// 或者使用 useHydrated
function useHydrated() {
  const [hydrated, setHydrated] = useState(false);
  
  useEffect(() => {
    setHydrated(true);
  }, []);
  
  return hydrated;
}
```

---

## MDX 未来趋势

### MDX 3.0 预览

```typescript
// MDX 3.0 的新特性（草案）

// 1. 更好的 TypeScript 支持
type MDXProps<T extends Frontmatter = Frontmatter> = {
  source: string;
  scope?: Record<string, unknown>;
  components?: MDXComponents;
  frontmatter?: T;
};

// 2. 异步插件支持
const asyncPlugin = async (tree: mdast.Root) => {
  const data = await fetchExternalData();
  // 处理数据...
  return tree;
};

// 3. 更好的性能
// - 增量编译
// - WebWorker 编译
// - WASM 加速

// 4. ESM 原生支持
// import.meta.resolve 支持
```

### 与 AI 集成的趋势

```typescript
// AI 辅助 MDX 创作

// 1. 智能代码块生成
const AICodeBlock = ({ 
  language, 
  description 
}: AICodeBlockProps) => {
  const [code, setCode] = useState<string | null>(null);
  const [loading, setLoading] = useState(false);
  
  const generate = async () => {
    setLoading(true);
    const response = await ai.generateCode({
      language,
      description,
      context: getMDXContext(),
    });
    setCode(response.code);
    setLoading(false);
  };
  
  return (
    <div>
      {loading ? <Skeleton /> : code ? (
        <CodeBlock language={language}>{code}</CodeBlock>
      ) : (
        <Button onClick={generate}>AI 生成代码</Button>
      )}
    </div>
  );
};

// 2. 智能内容补全
const MDXEditor = () => {
  const [suggestions, setSuggestions] = useState<string[]>([]);
  
  const handleInput = async (value: string) => {
    const suggestions = await ai.suggest({
      type: 'mdx-completion',
      content: value,
    });
    setSuggestions(suggestions);
  };
  
  return (
    <div>
      <Textarea onChange={handleInput} />
      <SuggestionList suggestions={suggestions} />
    </div>
  );
};

// 3. 语义搜索
const SemanticSearch = async (query: string) => {
  return ai.search({
    collection: 'posts',
    query,
    type: 'semantic',
    embeddings: true,
  });
};
```

---

## MDX 高级内容管理系统

### 内容架构设计

大型 MDX 项目需要系统化的内容管理策略：

```typescript
// 内容目录结构
// content/
// ├── docs/
// │   ├── getting-started/
// │   │   ├── index.mdx
// │   │   ├── installation.mdx
// │   │   └── configuration.mdx
// │   ├── guides/
// │   │   ├── index.mdx
// │   │   ├── beginner/
// │   │   │   ├── index.mdx
// │   │   │   └── *.mdx
// │   │   └── advanced/
// │   │       ├── index.mdx
// │   │       └── *.mdx
// │   └── api/
// │       ├── index.mdx
// │       └── *.mdx
// ├── blog/
// │   ├── index.mdx
// │   └── posts/
// │       └── *.mdx
// └── changelog/
//     └── *.mdx

// 内容元数据定义
interface ContentMeta {
  slug: string;
  title: string;
  description: string;
  author?: string;
  date?: string;
  tags?: string[];
  category?: string;
  draft?: boolean;
  deprecated?: boolean;
  version?: string;
}

// 导航配置
interface NavigationConfig {
  sidebar: {
    [path: string]: SidebarItem[];
  };
  prevNext?: boolean;
  breadcrumb?: boolean;
}

interface SidebarItem {
  title: string;
  path?: string;
  items?: SidebarItem[];
  collapsed?: boolean;
}

// 示例导航配置
const navigation: NavigationConfig = {
  sidebar: {
    '/docs/getting-started': [
      { title: '简介', path: '/docs/getting-started' },
      { title: '安装', path: '/docs/getting-started/installation' },
      { title: '快速开始', path: '/docs/getting-started/quickstart' },
    ],
    '/docs/guides': [
      {
        title: '入门指南',
        collapsed: false,
        items: [
          { title: '基础语法', path: '/docs/guides/beginner/syntax' },
          { title: '组件使用', path: '/docs/guides/beginner/components' },
        ],
      },
      {
        title: '进阶指南',
        collapsed: true,
        items: [
          { title: '插件开发', path: '/docs/guides/advanced/plugins' },
          { title: '自定义主题', path: '/docs/guides/advanced/theming' },
        ],
      },
    ],
  },
  prevNext: true,
  breadcrumb: true,
};
```

### 内容版本管理

```typescript
// 版本管理配置
interface VersionConfig {
  current: string;
  versions: Version[];
}

interface Version {
  version: string;
  label: string;
  path: string;
  banner?: 'none' | 'unmaintained' | 'beta';
  default?: boolean;
}

// 版本配置文件
const versions: VersionConfig = {
  current: 'v2',
  versions: [
    { version: 'v2', label: 'v2.x (最新)', path: '/docs', default: true },
    { version: 'v1', label: 'v1.x', path: '/docs/v1', banner: 'unmaintained' },
  ],
};

// MDX 文件中的版本标记
// v2-content.mdx
/*
---
title: 新功能
version: v2
deprecatedIn: v3
replaces: /docs/v1/old-feature
seeAlso:
  - /docs/v2/new-feature
  - /docs/v2/migration
---
*/

// 版本降级提示组件
interface VersionBannerProps {
  version: string;
  message?: string;
  action?: {
    label: string;
    href: string;
  };
}

function VersionBanner({ version, message, action }: VersionBannerProps) {
  return (
    <div className="version-banner">
      <span>本文档适用于 {version} 版本</span>
      {message && <p>{message}</p>}
      {action && (
        <a href={action.href} className="version-action">
          {action.label}
        </a>
      )}
    </div>
  );
}
```

### 内容国际化

```typescript
// i18n 配置
interface I18nConfig {
  defaultLocale: string;
  locales: LocaleConfig[];
}

interface LocaleConfig {
  code: string;
  name: string;
  path: string;
  rtl?: boolean;
}

const i18n: I18nConfig = {
  defaultLocale: 'zh-CN',
  locales: [
    { code: 'zh-CN', name: '简体中文', path: '/' },
    { code: 'en', name: 'English', path: '/en/' },
    { code: 'ja', name: '日本語', path: '/ja/' },
  ],
};

// MDX 文件命名规范
// content/
// ├── docs/
// │   ├── getting-started.mdx          # 默认语言
// │   ├── getting-started.zh-CN.mdx   # 中文版本
// │   ├── getting-started.en.mdx       # 英文版本
// │   ├── getting-started.ja.mdx      # 日文版本
// │   └── index.en.mdx                # 仅英文版本

// 本地化工具函数
function useTranslations(locale: string) {
  const translations = {
    'zh-CN': {
      'nav.docs': '文档',
      'nav.blog': '博客',
      'nav.github': 'GitHub',
      'search.placeholder': '搜索文档...',
      'search.noResults': '未找到结果',
    },
    'en': {
      'nav.docs': 'Docs',
      'nav.blog': 'Blog',
      'nav.github': 'GitHub',
      'search.placeholder': 'Search docs...',
      'search.noResults': 'No results found',
    },
  };

  return function t(key: string): string {
    return translations[locale]?.[key] || key;
  };
}

// MDX 中的国际化
/*
import { useTranslations } from '@/i18n';

# 欢迎使用文档

<Callout type="info">
  <Trans i18nKey="welcome.message" />
</Callout>

<Link href="/zh-CN/guide">中文指南</Link>
<Link href="/en/guide">English Guide</Link>
*/
```

---

## MDX 部署与运维

### 构建优化

```typescript
// 完整的构建配置
// next.config.js 或 vite.config.ts

// Next.js 配置
/** @type {import('next').NextConfig} */
const nextConfig = {
  // MDX 配置
  pageExtensions: ['ts', 'tsx', 'mdx'],
  
  experimental: {
    // 使用 Rust 编写的 MDX 编译器（更快）
    mdxRs: true,
    
    // 优化包导入
    optimizePackageImports: ['lucide-react', '@radix-ui/react-icons'],
  },
  
  // 图片优化
  images: {
    formats: ['image/avif', 'image/webp'],
    remotePatterns: [
      { protocol: 'https', hostname: 'images.unsplash.com' },
      { protocol: 'https', hostname: 'picsum.photos' },
    ],
  },
  
  // 压缩
  compress: true,
  
  // 生成 Etags
  generateEtags: true,
  
  // 静态导出（可选）
  output: 'export',
};

// Vite 配置
import { defineConfig } from 'vite';
import mdx from '@mdx-js/vite';
import remarkGfm from 'remark-gfm';
import rehypeShiki from '@shikijs/rehype';
import rehypeSlug from 'rehype-slug';
import rehypeAutolinkHeadings from 'rehype-autolink-headings';

export default defineConfig({
  plugins: [
    mdx({
      remarkPlugins: [remarkGfm],
      rehypePlugins: [
        rehypeSlug,
        [rehypeAutolinkHeadings, { behavior: 'wrap' }],
        [rehypeShiki, { theme: 'github-dark' }],
      ],
      providerImportSource: '@mdx-js/react',
    }),
  ],
  
  build: {
    // 分块策略
    rollupOptions: {
      output: {
        manualChunks: {
          'mdx-runtime': ['@mdx-js/mdx', '@mdx-js/react'],
          'shiki': ['shiki'],
        },
      },
    },
    
    // 压缩配置
    minify: 'terser',
    terserOptions: {
      compress: {
        drop_console: true,
        drop_debugger: true,
      },
    },
  },
  
  // 优化依赖预构建
  optimizeDeps: {
    include: ['react', 'react-dom', '@mdx-js/react', '@mdx-js/mdx'],
  },
});
```

### CI/CD 配置

```yaml
# .github/workflows/deploy.yml
name: Deploy Documentation

on:
  push:
    branches: [main]
  pull_request:
    branches: [main]

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    
    steps:
      - uses: actions/checkout@v4
        with:
          fetch-depth: 0
      
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
          cache: 'npm'
      
      - name: Install dependencies
        run: npm ci
      
      - name: Run linter
        run: npm run lint
      
      - name: Run tests
        run: npm test
      
      - name: Build
        run: npm run build
        env:
          NODE_ENV: production
      
      - name: Lighthouse CI
        uses: treosh/lighthouse-ci-action@v11
        with:
          configPath: './lighthouserc.json'
          uploadArtifacts: true
      
      - name: Deploy to GitHub Pages
        if: github.ref == 'refs/heads/main'
        uses: peaceiris/actions-gh-pages@v4
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: ./out
          cname: docs.example.com

  # Netlify 部署
  deploy-netlify:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v4
      - uses: actions/setup-node@v4
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run build
      - uses: nwtgck/actions-netlify@v3
        with:
          publish-dir: ./out
          production-deploy: true
          github-token: ${{ secrets.GITHUB_TOKEN }}
          deploy-message: "Deploy from GitHub Actions"
        env:
          NETLIFY_AUTH_TOKEN: ${{ secrets.NETLIFY_AUTH_TOKEN }}
          NETLIFY_SITE_ID: ${{ secrets.NETLIFY_SITE_ID }}
```

### 监控与分析

```typescript
// 性能监控
interface PerformanceMetrics {
  // Core Web Vitals
  LCP?: number;  // Largest Contentful Paint
  FID?: number;  // First Input Delay
  CLS?: number;  // Cumulative Layout Shift
  
  // 自定义指标
  TTFB?: number; // Time to First Byte
  FCP?: number;  // First Contentful Paint
  TTMC?: number; // Time to Main Content
}

// 性能监控实现
class PerformanceMonitor {
  private metrics: PerformanceMetrics = {};
  
  constructor() {
    if (typeof window !== 'undefined') {
      this.initWebVitals();
    }
  }
  
  private initWebVitals() {
    // LCP
    new PerformanceObserver((entryList) => {
      const entries = entryList.getEntries();
      const lastEntry = entries[entries.length - 1] as LargestContentfulPaint;
      this.metrics.LCP = lastEntry.renderTime || lastEntry.loadTime;
    }).observe({ entryTypes: ['largest-contentful-paint'] });
    
    // FID
    new PerformanceObserver((entryList) => {
      const entries = entryList.getEntries();
      const firstEntry = entries[0] as EventTiming;
      this.metrics.FID = firstEntry.processingStart - firstEntry.startTime;
    }).observe({ entryTypes: ['first-input'] });
    
    // CLS
    new PerformanceObserver((entryList) => {
      for (const entry of entryList.getEntries()) {
        if (!(entry as any).hadRecentInput) {
          this.metrics.CLS = (this.metrics.CLS || 0) + (entry as any).value;
        }
      }
    }).observe({ entryTypes: ['layout-shift'] });
  }
  
  report() {
    // 发送到分析服务
    if (typeof navigator !== 'undefined' && navigator.sendBeacon) {
      navigator.sendBeacon('/api/analytics', JSON.stringify({
        url: window.location.href,
        metrics: this.metrics,
        timestamp: Date.now(),
      }));
    }
  }
}

// 内容使用分析
interface ContentAnalytics {
  pageViews: number;
  uniqueVisitors: number;
  avgTimeOnPage: number;
  scrollDepth: number;
  completionRate: number;
}

// 使用分析 Hook
function useContentAnalytics(pageId: string) {
  const [analytics, setAnalytics] = useState<ContentAnalytics | null>(null);
  
  useEffect(() => {
    // 滚动深度跟踪
    let maxScroll = 0;
    const handleScroll = () => {
      const scrollPercent = Math.round(
        (window.scrollY / (document.body.scrollHeight - window.innerHeight)) * 100
      );
      maxScroll = Math.max(maxScroll, scrollPercent);
    };
    
    window.addEventListener('scroll', handleScroll, { passive: true });
    
    // 页面离开时报告
    const handleUnload = () => {
      fetch('/api/analytics', {
        method: 'POST',
        body: JSON.stringify({
          pageId,
          scrollDepth: maxScroll,
          timeSpent: Date.now() - startTime,
        }),
      });
    };
    
    window.addEventListener('beforeunload', handleUnload);
    
    return () => {
      window.removeEventListener('scroll', handleScroll);
      window.removeEventListener('beforeunload', handleUnload);
    };
  }, [pageId]);
  
  return analytics;
}
```

---

## MDX 主题开发完整指南

### 主题架构设计

```typescript
// 主题目录结构
// theme/
// ├── index.ts                 # 主题入口
// ├── Layout.tsx               # 根布局组件
// ├── components/
// │   ├── Header.tsx
// │   ├── Sidebar.tsx
// │   ├── Footer.tsx
// │   ├── TableOfContents.tsx
// │   ├── Search.tsx
// │   └── CopyButton.tsx
// ├── mdx/
// │   ├── Callout.tsx
// │   ├── CodeBlock.tsx
// │   ├── Tabs.tsx
// │   └── Steps.tsx
// ├── styles/
// │   ├── globals.css
// │   ├── typography.css
// │   └── syntax-theme.css
// └── utils/
//     ├── toc.ts
//     └── slug.ts

// 主题入口文件
// theme/index.ts
import { h } from 'vue';
import type { Theme } from 'vitepress';
import DefaultTheme from 'vitepress/theme';
import './styles/main.css';

import Layout from './Layout.vue';
import Home from './components/Home.vue';
import NotFound from './components/NotFound.vue';

export default {
  extends: DefaultTheme,
  
  Layout,
  
  enhanceApp({ app, router, siteData }) {
    // 注册全局组件
    app.component('Home', Home);
    app.component('NotFound', NotFound);
    
    // 注册 MDX 组件
    const mdxComponents = import.meta.glob('./mdx/*.tsx');
    for (const [path, component] of Object.entries(mdxComponents)) {
      const name = path.split('/').pop()?.replace('.tsx', '');
      app.component(name!, component as any);
    }
  },
} satisfies Theme;

// 布局组件
// theme/Layout.vue
<template>
  <DefaultTheme.Layout>
    <template #home-hero>
      <slot name="hero" />
    </template>
    
    <template #doc-before>
      <div class="doc-header">
        <Breadcrumb />
        <ArticleMeta />
      </div>
    </template>
    
    <template #doc-after>
      <ArticleFooter />
    </template>
  </DefaultTheme.Layout>
</template>

<script setup lang="ts">
import DefaultTheme from 'vitepress/theme';
import Breadcrumb from './components/Breadcrumb.vue';
import ArticleMeta from './components/ArticleMeta.vue';
import ArticleFooter from './components/ArticleFooter.vue';
</script>
```

### 主题组件开发

```vue
<!-- theme/components/Callout.vue -->
<template>
  <div :class="['callout', `callout-${type}`]">
    <div class="callout-icon">
      <component :is="iconComponent" />
    </div>
    <div class="callout-content">
      <div v-if="title" class="callout-title">{{ title }}</div>
      <div class="callout-body">
        <slot />
      </div>
    </div>
  </div>
</template>

<script setup lang="ts">
import { computed } from 'vue';
import { InfoIcon, AlertIcon, WarningIcon, AlertCircleIcon } from 'lucide-vue-next';

const props = defineProps<{
  type?: 'note' | 'tip' | 'warning' | 'danger' | 'info';
  title?: string;
}>();

const iconComponent = computed(() => {
  const icons = {
    note: InfoIcon,
    tip: AlertIcon,
    warning: WarningIcon,
    danger: AlertCircleIcon,
    info: InfoIcon,
  };
  return icons[props.type || 'note'];
});
</script>

<style scoped>
.callout {
  display: flex;
  gap: 12px;
  padding: 16px;
  border-radius: 8px;
  margin: 16px 0;
}

.callout-note {
  background: #e0f2fe;
  border-left: 4px solid #0ea5e9;
}

.callout-tip {
  background: #dcfce7;
  border-left: 4px solid #22c55e;
}

.callout-warning {
  background: #fef9c3;
  border-left: 4px solid #eab308;
}

.callout-danger {
  background: #fee2e2;
  border-left: 4px solid #ef4444;
}

.callout-icon {
  flex-shrink: 0;
}

.callout-title {
  font-weight: 600;
  margin-bottom: 4px;
}

.callout-content {
  flex: 1;
  min-width: 0;
}
</style>
```

```vue
<!-- theme/components/CodeBlock.vue -->
<template>
  <div class="code-block">
    <div v-if="filename || language" class="code-header">
      <span v-if="filename" class="code-filename">{{ filename }}</span>
      <span v-if="language" class="code-language">{{ language }}</span>
      <button class="copy-button" @click="copyCode">
        <CopyIcon v-if="!copied" />
        <CheckIcon v-else />
      </button>
    </div>
    <pre class="code-pre"><code ref="codeRef" :class="`language-${language}`"><slot /></code></pre>
  </div>
</template>

<script setup lang="ts">
import { ref } from 'vue';
import { Copy as CopyIcon, Check as CheckIcon } from 'lucide-vue-next';

const props = defineProps<{
  language?: string;
  filename?: string;
  highlights?: number[];
}>();

const codeRef = ref<HTMLElement>();
const copied = ref(false);

async function copyCode() {
  const code = codeRef.value?.textContent || '';
  await navigator.clipboard.writeText(code);
  copied.value = true;
  setTimeout(() => {
    copied.value = false;
  }, 2000);
}
</script>

<style scoped>
.code-block {
  position: relative;
  margin: 16px 0;
  border-radius: 8px;
  overflow: hidden;
  background: #1e293b;
}

.code-header {
  display: flex;
  align-items: center;
  gap: 8px;
  padding: 8px 16px;
  background: #0f172a;
  border-bottom: 1px solid #334155;
}

.code-filename {
  color: #e2e8f0;
  font-size: 13px;
}

.code-language {
  color: #94a3b8;
  font-size: 12px;
  margin-left: auto;
}

.copy-button {
  background: transparent;
  border: none;
  color: #94a3b8;
  cursor: pointer;
  padding: 4px;
  border-radius: 4px;
}

.copy-button:hover {
  color: #e2e8f0;
  background: #334155;
}

.code-pre {
  margin: 0;
  padding: 16px;
  overflow-x: auto;
}

.code-pre code {
  font-family: 'JetBrains Mono', monospace;
  font-size: 14px;
  line-height: 1.6;
}
</style>
```

### 暗色模式实现

```typescript
// 主题切换逻辑
// theme/composables/useTheme.ts
import { ref, watch, computed } from 'vue';

type Theme = 'light' | 'dark' | 'auto';

const theme = ref<Theme>('auto');
const prefersDark = ref(false);

export function useTheme() {
  // 监听系统主题变化
  if (typeof window !== 'undefined') {
    const mediaQuery = window.matchMedia('(prefers-color-scheme: dark)');
    prefersDark.value = mediaQuery.matches;
    
    mediaQuery.addEventListener('change', (e) => {
      prefersDark.value = e.matches;
    });
  }
  
  // 计算实际生效的主题
  const isDark = computed(() => {
    if (theme.value === 'auto') {
      return prefersDark.value;
    }
    return theme.value === 'dark';
  });
  
  // 应用主题到文档
  function applyTheme() {
    if (typeof document !== 'undefined') {
      document.documentElement.classList.toggle('dark', isDark.value);
    }
  }
  
  // 监听变化
  watch([theme, prefersDark], applyTheme, { immediate: true });
  
  // 切换主题
  function setTheme(newTheme: Theme) {
    theme.value = newTheme;
    localStorage.setItem('theme', newTheme);
  }
  
  // 初始化
  function initTheme() {
    const stored = localStorage.getItem('theme') as Theme | null;
    if (stored) {
      theme.value = stored;
    }
  }
  
  return {
    theme,
    isDark,
    setTheme,
    initTheme,
  };
}

// 使用示例
/*
<script setup>
import { onMounted } from 'vue';
import { useTheme } from '../composables/useTheme';

const { initTheme, isDark, setTheme } = useTheme();

onMounted(() => {
  initTheme();
});
</script>

<template>
  <button @click="setTheme(isDark ? 'light' : 'dark')">
    {{ isDark ? '切换到浅色模式' : '切换到深色模式' }}
  </button>
</template>
*/
```

---

## MDX 编辑器与创作工具

### VS Code 配置

```json
// .vscode/settings.json
{
  "editor.formatOnSave": true,
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  "[markdown]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode",
    "editor.wordWrap": "on",
    "editor.quickSuggestions": {
      "other": true,
      "comments": false,
      "strings": true
    }
  },
  "[mdx]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "files.associations": {
    "*.mdx": "mdx"
  },
  "markdown.styles": [],
  "markdown.validate.enabled": true,
  "[typescript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  },
  "emmet.includeLanguages": {
    "mdx": "html"
  },
  "cSpell.language": "en,zh-CN",
  "cSpell.words": [
    "MDX",
    "VitePress",
    "frontmatter",
    "codeblock"
  ]
}
```

```json
// .vscode/extensions.json
{
  "recommendations": [
    "dbaeumer.vscode-eslint",
    "esbenp.prettier-vscode",
    "christian-kohler.path-intellisense",
    "formulahendry.auto-rename-tag",
    "ms-vscode.vscode-typescript-next",
    "styled-components.vscode-styled-components",
    "bradlc.vscode-tailwindcss",
    "davidanson.vscode-markdownlint",
    "bierner.emojisense",
    "oderwat.indent-rainbow",
    "usernamehw.errorlens",
    "contextusdc.theme-dracula-at-night"
  ]
}
```

### MDX 预览工具

```typescript
// 本地预览服务器
// scripts/preview.ts
import { createServer } from 'vite';
import mdx from '@mdx-js/vite';
import react from '@vitejs/plugin-react';
import { resolve } from 'path';

async function startPreview() {
  const server = await createServer({
    plugins: [
      react(),
      mdx(),
    ],
    resolve: {
      alias: {
        '@': resolve(__dirname, '../src'),
        '@components': resolve(__dirname, '../src/components'),
        '@content': resolve(__dirname, '../content'),
      },
    },
    server: {
      port: 3000,
      open: true,
    },
  });
  
  await server.listen();
  server.printUrls();
}

startPreview();
```

```bash
# 预览脚本
# package.json
{
  "scripts": {
    "preview": "tsx scripts/preview.ts",
    "preview:build": "npm run build && npm run preview:static",
    "preview:static": "npx serve dist"
  }
}
```

### 内容质量检查

```typescript
// scripts/lint-content.ts
import { glob } from 'glob';
import matter from 'gray-matter';
import { z } from 'zod';

const frontmatterSchema = z.object({
  title: z.string().min(1),
  description: z.string().optional(),
  date: z.string().optional(),
  author: z.string().optional(),
  tags: z.array(z.string()).optional(),
  draft: z.boolean().optional(),
});

async function lintContent() {
  const files = await glob('content/**/*.mdx');
  const errors: string[] = [];
  
  for (const file of files) {
    const content = await Bun.file(file).text();
    const { data, content: body } = matter(content);
    
    // 验证 frontmatter
    const result = frontmatterSchema.safeParse(data);
    if (!result.success) {
      errors.push(`${file}: Invalid frontmatter - ${result.error.message}`);
    }
    
    // 检查标题存在性
    if (!body.includes('# ')) {
      errors.push(`${file}: Missing main heading`);
    }
    
    // 检查代码块语言标注
    const codeBlocks = body.match(/```(\w+)/g);
    if (codeBlocks) {
      const withoutLang = codeBlocks.filter(block => {
        const lang = block.replace('```', '');
        return !lang || lang === 'text';
      });
      if (withoutLang.length > 0) {
        errors.push(`${file}: Found ${withoutLang.length} code blocks without language`);
      }
    }
    
    // 检查链接有效性（可选）
    const links = body.match(/\[([^\]]+)\]\(([^)]+)\)/g);
    // ... 验证链接
  }
  
  if (errors.length > 0) {
    console.error('Content lint errors:');
    errors.forEach(e => console.error(`  - ${e}`));
    process.exit(1);
  }
  
  console.log(`✓ Linted ${files.length} files`);
}

lintContent();
```

---

## MDX 生态系统资源

### 官方资源

| 资源 | 链接 |
|------|------|
| MDX 官方网站 | https://mdxjs.com/ |
| MDX GitHub | https://github.com/mdx-js/mdx |
| VitePress | https://vitepress.dev/ |
| Docusaurus | https://docusaurus.io/ |
| Next.js | https://nextjs.org/ |
| Astro | https://astro.build/ |

### 插件生态

| 插件 | 功能 |
|------|------|
| remark-gfm | GitHub Flavored Markdown 支持 |
| remark-frontmatter | Frontmatter 解析 |
| remark-mdx-frontmatter | Frontmatter 导出为变量 |
| rehype-prism | Prism 代码高亮 |
| @shikijs/rehype | Shiki 代码高亮（支持更多主题） |
| rehype-slug | 自动添加标题 ID |
| rehype-autolink-headings | 标题自动链接 |
| remark-toc | 目录生成 |
| rehype-katex | LaTeX 数学公式 |
| rehype-mathjax | MathJax 数学渲染 |

### 学习资源

- [[MDX 官方文档]]
- [[Next.js 文档工具链]]
- [[Astro 内容管理]]
- [[React 组件设计]]
- [[Shiki 代码高亮]]
- [[VitePress 配置指南]]
- [[Docusaurus 教程]]

---

## 相关资源

- [[Next.js 文档工具链]]
- [[Astro 内容管理]]
- [[React 组件设计]]
- [[MDX 官方文档]]
- [[Shiki 代码高亮]]
- [[Contentful CMS 集成]]
- [[Notion API 开发]]
- [[静态站点生成器对比]]

---

*本文档由 [[归愚知识系统]] 自动生成*
