# learnyourway-login (SvelteKit)

登录页的 SvelteKit 试点。与主应用 (learnyourway) 同域名、同端口运行。

## 开发

1. **同时启动两个服务**：
   ```bash
   # 终端 1：主应用（Vite，端口 3000）
   cd learnyourway && npm run dev

   # 终端 2：登录页（SvelteKit，端口 5174）
   cd learnyourway-login && npm install && npm run dev
   ```

2. 打开 http://localhost:3000/login 访问 SvelteKit 登录页  
3. 访问 http://localhost:3000/ 为 React 主应用  

Vite 会将 `/login` 和 `/_app` 代理到 SvelteKit，保证同域名、同端口，Supabase session 可共享。

## 生产部署

- 构建 SvelteKit：`npm run build`
- 用 Nginx/Cloudflare 等将 `/login`、`/_app` 指向 SvelteKit 构建产物
- 其余路径指向 React 主应用
