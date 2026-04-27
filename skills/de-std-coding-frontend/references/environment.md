# 环境配置规范

> 本规范定义前端环境配置的使用规范，包括环境变量、构建配置、代理配置和部署配置。

---

## 📋 目录

- [环境变量](#环境变量)
- [环境文件](#环境文件)
- [Vite配置](#vite配置)
- [代理配置](#代理配置)
- [构建配置](#构建配置)
- [部署配置](#部署配置)
- [最佳实践](#最佳实践)

---

## 环境变量

### 环境变量命名

| 前缀 | 说明 | 示例 |
|------|------|------|
| `VITE_` | Vite环境变量（前端可访问） | `VITE_API_URL` |
| `NODE_ENV` | Node环境（Vite自动设置） | `development`, `production` |
| 无前缀 | 仅构建时可用 | `API_KEY` |

### 常用环境变量

```bash
# API相关
VITE_API_URL              # API基础URL
VITE_API_TIMEOUT          # API请求超时时间
VITE_WS_URL               # WebSocket URL

# 应用配置
VITE_APP_TITLE            # 应用标题
VITE_APP_VERSION          # 应用版本
VITE_APP_ENVIRONMENT      # 应用环境

# 功能开关
VITE_ENABLE_MOCK          # 是否启用Mock
VITE_ENABLE_DEBUG         # 是否启用调试模式
VITE_ENABLE_ANALYTICS     # 是否启用分析

# 第三方服务
VITE_SENTRY_DSN           # Sentry DSN
VITE_GA_TRACKING_ID       # Google Analytics ID
```

---

## 环境文件

### 开发环境（.env.development）

```bash
# 开发环境配置
NODE_ENV=development

# API配置
VITE_API_URL=http://localhost:8080
VITE_API_TIMEOUT=30000
VITE_WS_URL=ws://localhost:8080/ws

# 应用配置
VITE_APP_TITLE=数擎数据中台（开发）
VITE_APP_VERSION=1.0.0
VITE_APP_ENVIRONMENT=development

# 功能开关
VITE_ENABLE_MOCK=false
VITE_ENABLE_DEBUG=true
VITE_ENABLE_ANALYTICS=false

# 第三方服务
VITE_SENTRY_DSN=
VITE_GA_TRACKING_ID=
```

### 生产环境（.env.production）

```bash
# 生产环境配置
NODE_ENV=production

# API配置
VITE_API_URL=https://api.example.com
VITE_API_TIMEOUT=30000
VITE_WS_URL=wss://api.example.com/ws

# 应用配置
VITE_APP_TITLE=数擎数据中台
VITE_APP_VERSION=1.0.0
VITE_APP_ENVIRONMENT=production

# 功能开关
VITE_ENABLE_MOCK=false
VITE_ENABLE_DEBUG=false
VITE_ENABLE_ANALYTICS=true

# 第三方服务
VITE_SENTRY_DSN=https://xxx@sentry.io/xxx
VITE_GA_TRACKING_ID=G-XXXXXXXXXX
```

### 测试环境（.env.staging）

```bash
# 测试环境配置
NODE_ENV=production

# API配置
VITE_API_URL=https://staging-api.example.com
VITE_API_TIMEOUT=30000
VITE_WS_URL=wss://staging-api.example.com/ws

# 应用配置
VITE_APP_TITLE=数擎数据中台（测试）
VITE_APP_VERSION=1.0.0-staging
VITE_APP_ENVIRONMENT=staging

# 功能开关
VITE_ENABLE_MOCK=false
VITE_ENABLE_DEBUG=false
VITE_ENABLE_ANALYTICS=true

# 第三方服务
VITE_SENTRY_DSN=https://xxx@sentry.io/xxx
VITE_GA_TRACKING_ID=
```

### 本地环境（.env.local）

```bash
# 本地环境配置（不提交到git）
NODE_ENV=development

# 本地API配置
VITE_API_URL=http://localhost:8080
VITE_API_TIMEOUT=30000

# 本地功能开关
VITE_ENABLE_MOCK=true
VITE_ENABLE_DEBUG=true
```

---

## Vite配置

### 基础配置

```typescript
// vite.config.ts
import { defineConfig, loadEnv } from 'vite'
import vue from '@vitejs/plugin-vue'
import { resolve } from 'path'

export default defineConfig(({ mode }) => {
  // 加载环境变量
  const env = loadEnv(mode, process.cwd())

  return {
    plugins: [vue()],

    // 路径别名
    resolve: {
      alias: {
        '@': resolve(__dirname, 'src')
      }
    },

    // 开发服务器
    server: {
      port: 5173,
      host: true,
      open: true,
      proxy: {
        '/api': {
          target: env.VITE_API_URL || 'http://localhost:8080',
          changeOrigin: true,
          secure: false
        }
      }
    },

    // 构建配置
    build: {
      target: 'es2015',
      outDir: 'dist',
      assetsDir: 'assets',
      sourcemap: mode === 'development',
      minify: 'terser',
      terserOptions: {
        compress: {
          drop_console: mode === 'production',
          drop_debugger: mode === 'production'
        }
      },
      rollupOptions: {
        output: {
          // 分包策略
          manualChunks: {
            'vue-vendor': ['vue', 'vue-router', 'pinia'],
            'element-plus': ['element-plus'],
            'echarts': ['echarts']
          }
        }
      }
    },

    // CSS配置
    css: {
      preprocessorOptions: {
        scss: {
          additionalData: `@import "@/styles/variables.scss";`
        }
      }
    },

    // 优化配置
    optimizeDeps: {
      include: [
        'vue',
        'vue-router',
        'pinia',
        'element-plus',
        'axios'
      ]
    }
  }
})
```

### TypeScript配置

```typescript
// tsconfig.json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "moduleResolution": "bundler",
    "jsx": "preserve",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "allowSyntheticDefaultImports": true,
    "forceConsistentCasingInFileNames": true,
    "resolveJsonModule": true,
    "isolatedModules": true,
    "noEmit": true,
    "baseUrl": ".",
    "paths": {
      "@/*": ["src/*"]
    },
    "types": ["vite/client"]
  },
  "include": [
    "src/**/*.ts",
    "src/**/*.d.ts",
    "src/**/*.tsx",
    "src/**/*.vue"
  ],
  "exclude": ["node_modules"]
}
```

---

## 代理配置

### 基础代理

```typescript
// vite.config.ts
server: {
  proxy: {
    // 代理所有 /api 请求
    '/api': {
      target: 'http://localhost:8080',
      changeOrigin: true,
      secure: false
    }
  }
}
```

### 高级代理配置

```typescript
// vite.config.ts
server: {
  proxy: {
    // API代理
    '/api': {
      target: 'http://localhost:8080',
      changeOrigin: true,
      secure: false,
      rewrite: (path) => path.replace(/^\/api/, '')
    },

    // WebSocket代理
    '/ws': {
      target: 'ws://localhost:8080',
      changeOrigin: true,
      ws: true
    },

    // 静态资源代理
    '/static': {
      target: 'http://localhost:8080',
      changeOrigin: true,
      secure: false
    }
  }
}
```

### 环境区分代理

```typescript
// vite.config.ts
import { defineConfig, loadEnv } from 'vite'

export default defineConfig(({ mode }) => {
  const env = loadEnv(mode, process.cwd())

  return {
    server: {
      proxy: {
        '/api': {
          target: env.VITE_API_URL,
          changeOrigin: true,
          secure: false
        }
      }
    }
  }
})
```

---

## 构建配置

### 开发构建

```bash
# 开发构建
npm run dev

# 指定端口
npm run dev -- --port 3000

# 指定host
npm run dev -- --host 0.0.0.0
```

### 生产构建

```bash
# 生产构建
npm run build

# 预览构建结果
npm run preview

# 指定配置文件
vite build --config vite.config.prod.ts
```

### 分析构建结果

```typescript
// vite.config.ts
import { defineConfig } from 'vite'
import { visualizer } from 'rollup-plugin-visualizer'

export default defineConfig({
  plugins: [
    vue(),
    visualizer({
      open: true,
      gzipSize: true,
      brotliSize: true
    })
  ]
})
```

### 多环境构建

```json
// package.json
{
  "scripts": {
    "dev": "vite",
    "build": "vite build",
    "build:staging": "vite build --mode staging",
    "build:prod": "vite build --mode production",
    "preview": "vite preview"
  }
}
```

---

## 部署配置

### Nginx配置

```nginx
# /etc/nginx/conf.d/dataengine-frontend.conf
server {
    listen 80;
    server_name example.com;

    root /var/www/dataengine-frontend/dist;
    index index.html;

    # Gzip压缩
    gzip on;
    gzip_types text/plain text/css application/json application/javascript text/xml application/xml application/xml+rss text/javascript;

    # SPA路由支持
    location / {
        try_files $uri $uri/ /index.html;
    }

    # API代理
    location /api {
        proxy_pass http://localhost:8080;
        proxy_set_header Host $host;
        proxy_set_header X-Real-IP $remote_addr;
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header X-Forwarded-Proto $scheme;
    }

    # 静态资源缓存
    location ~* \.(js|css|png|jpg|jpeg|gif|ico|svg|woff|woff2|ttf|eot)$ {
        expires 1y;
        add_header Cache-Control "public, immutable";
    }

    # 安全头
    add_header X-Frame-Options "SAMEORIGIN" always;
    add_header X-Content-Type-Options "nosniff" always;
    add_header X-XSS-Protection "1; mode=block" always;
}
```

### Docker配置

```dockerfile
# Dockerfile
# 构建阶段
FROM node:18-alpine as builder

WORKDIR /app

# 复制package文件
COPY package*.json ./
RUN npm ci

# 复制源代码
COPY . .

# 构建
RUN npm run build

# 生产阶段
FROM nginx:alpine

# 复制nginx配置
COPY nginx.conf /etc/nginx/conf.d/default.conf

# 复制构建结果
COPY --from=builder /app/dist /usr/share/nginx/html

# 暴露端口
EXPOSE 80

CMD ["nginx", "-g", "daemon off;"]
```

### Docker Compose

```yaml
# docker-compose.yml
version: '3.8'

services:
  frontend:
    build:
      context: .
      dockerfile: Dockerfile
    ports:
      - "80:80"
    environment:
      - VITE_API_URL=http://backend:8080
    depends_on:
      - backend
    restart: unless-stopped

  backend:
    image: dataengine-api:latest
    ports:
      - "8080:8080"
    environment:
      - SPRING_PROFILES_ACTIVE=production
    restart: unless-stopped
```

---

## 最佳实践

### 环境变量使用

```typescript
// 正确：使用环境变量
const apiUrl = import.meta.env.VITE_API_URL

// 错误：不要使用 process.env（Vite中不可用）
const apiUrl = process.env.VITE_API_URL // ❌
```

### 环境变量类型定义

```typescript
// src/types/env.d.ts
/// <reference types="vite/client" />

interface ImportMetaEnv {
  readonly VITE_API_URL: string
  readonly VITE_API_TIMEOUT: string
  readonly VITE_APP_TITLE: string
  readonly VITE_ENABLE_DEBUG: string
  readonly VITE_ENABLE_MOCK: string
}

interface ImportMeta {
  readonly env: ImportMetaEnv
}
```

### 条件编译

```typescript
// 环境判断
if (import.meta.env.DEV) {
  console.log('开发环境')
}

if (import.meta.env.PROD) {
  console.log('生产环境')
}

// 功能开关
if (import.meta.env.VITE_ENABLE_DEBUG === 'true') {
  // 调试代码
}
```

### 敏感信息处理

```bash
# ❌ 不要在环境文件中存储敏感信息
VITE_API_KEY=sk-xxxxxxxxxxxxxxxxxxxxx

# ✅ 使用构建时注入或后端代理获取敏感信息
```

### 环境文件管理

```gitignore
# .gitignore
# 本地环境变量（不提交）
.env.local
.env.*.local

# 提交模板环境变量
.env.example
.env.development.example
.env.production.example
```

### 版本号管理

```typescript
// package.json
{
  "version": "1.0.0",
  "scripts": {
    "version:patch": "npm version patch",
    "version:minor": "npm version minor",
    "version:major": "npm version major"
  }
}

// 自动注入版本号
// vite.config.ts
export default defineConfig({
  define: {
    __APP_VERSION__: JSON.stringify(process.env.npm_package_version)
  }
})
```

---

## 🔗 相关文档

- [API调用规范](./api-calling.md) - API调用详细规范
- [Vue 3编码规范](./vue-coding.md) - Vue 3详细规范
- [性能优化规范](./performance.md) - 性能优化详细规范

---

**最后更新**：2026-03-12