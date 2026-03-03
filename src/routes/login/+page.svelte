<script lang="ts">
  import { goto } from '$app/navigation';
  import { createClient } from '@supabase/supabase-js';
  import { SUPABASE_URL, SUPABASE_ANON_KEY, API_URL } from '$lib/config';

  let email = '';
  let password = '';
  let error = '';
  let loading = false;

  async function resolveLogin(identifier: string): Promise<{ email: string }> {
    const res = await fetch(`${API_URL.replace(/\/$/, '')}/api/auth/resolve-login`, {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({ identifier: identifier.trim() }),
    });
    const json = await res.json().catch(() => ({}));
    if (!res.ok) throw new Error(json.error || '无法解析登录账号');
    if (!json.email || typeof json.email !== 'string') throw new Error('无效的账号');
    return { email: json.email };
  }

  async function handleLogin(e: Event) {
    e.preventDefault();
    loading = true;
    error = '';
    try {
      const loginEmail = email.includes('@') ? email : (await resolveLogin(email)).email;
      const client = createClient(SUPABASE_URL, SUPABASE_ANON_KEY);
      const { error: authError } = await client.auth.signInWithPassword({ email: loginEmail, password });
      if (authError) {
        const msg = authError.message || 'Login failed';
        const isEmailNotConfirmed = /email not confirmed/i.test(msg);
        throw new Error(isEmailNotConfirmed
          ? '邮箱尚未验证，请先点击注册邮件中的确认链接，或联系管理员确认账号。'
          : msg);
      }
      const redirect = typeof window !== 'undefined' ? new URLSearchParams(window.location.search).get('redirect') : null;
      const target = redirect?.startsWith('/') ? redirect : '/';
      if (typeof window !== 'undefined') {
        window.location.href = target;
      } else {
        await goto(target);
      }
    } catch (err: unknown) {
      error = err instanceof Error ? err.message : '登录失败';
    } finally {
      loading = false;
    }
  }
</script>

<svelte:head>
  <title>登录 - AI随心学</title>
</svelte:head>

<div class="page-wrap">
  <div class="hero">
    <h1>为每位学习者<br /><span class="accent">重塑</span><br />自学体验</h1>
    <p>AI随心学将教材和学习资料转化为引人入胜的多媒体学习体验，为你量身定制。</p>
  </div>
  <div class="form-panel">
    <div class="card">
      <h2>欢迎使用</h2>
      <p class="subtitle">登录开始您的学习之旅</p>
      {#if error}
        <div class="error">{error}</div>
      {/if}
      <form onsubmit={handleLogin}>
        <label for="email">邮箱 / 手机号 / 账号</label>
        <input
          id="email"
          type="text"
          placeholder="输入邮箱、手机号或账号名"
          bind:value={email}
          required
        />
        <label for="password">密码</label>
        <input
          id="password"
          type="password"
          placeholder="输入您的密码"
          bind:value={password}
          required
        />
        <button type="submit" disabled={loading}>
          {loading ? '登录中...' : '登录'}
        </button>
      </form>
      <p class="hint">使用 React 主应用账号登录，登录后将跳转回主应用</p>
      <p class="svelte-badge">SvelteKit 页面</p>
    </div>
  </div>
</div>

<style>
  .page-wrap {
    min-height: 100vh;
    display: flex;
    background-color: #f8f9fa;
  }
  .hero {
    flex: 1;
    display: none;
    align-items: center;
    justify-content: center;
    padding: 2rem;
    background: linear-gradient(to bottom right, #fed7aa, #fdba74, #fb923c);
    opacity: 0.9;
  }
  @media (min-width: 1024px) {
    .hero {
      display: flex;
      flex-direction: column;
    }
  }
  .hero h1 {
    font-size: 2.5rem;
    font-weight: bold;
    color: #111;
    line-height: 1.3;
    margin: 0 0 1rem 0;
  }
  .hero .accent {
    color: #4f46e5;
  }
  .hero p {
    font-size: 1.125rem;
    color: #374151;
    line-height: 1.6;
    margin: 0;
  }
  .form-panel {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 2rem;
    background: white;
  }
  .card {
    width: 100%;
    max-width: 28rem;
    padding: 2rem;
    border-radius: 0.75rem;
    box-shadow: 0 10px 25px -5px rgb(0 0 0 / 0.1);
    border: 1px solid #e5e7eb;
  }
  .card h2 {
    font-size: 1.5rem;
    color: #1a73e8;
    text-align: center;
    margin: 0 0 0.5rem 0;
  }
  .subtitle {
    text-align: center;
    color: #6b7280;
    margin: 0 0 1.5rem 0;
    font-size: 0.875rem;
  }
  .error {
    padding: 0.75rem;
    background: #fef2f2;
    border: 1px solid #fecaca;
    border-radius: 0.5rem;
    color: #dc2626;
    font-size: 0.875rem;
    margin-bottom: 1rem;
  }
  form {
    display: flex;
    flex-direction: column;
    gap: 1rem;
  }
  label {
    font-size: 0.875rem;
    font-weight: 500;
    color: #374151;
  }
  input {
    padding: 0.5rem 0.75rem;
    border: 1px solid #d1d5db;
    border-radius: 0.375rem;
    font-size: 1rem;
  }
  input:focus {
    outline: none;
    border-color: #1a73e8;
    box-shadow: 0 0 0 2px rgba(26, 115, 232, 0.2);
  }
  button {
    padding: 0.75rem 1rem;
    background: #1a73e8;
    color: white;
    border: none;
    border-radius: 0.375rem;
    font-size: 1rem;
    font-weight: 500;
    cursor: pointer;
  }
  button:hover:not(:disabled) {
    background: #1557b0;
  }
  button:disabled {
    opacity: 0.6;
    cursor: not-allowed;
  }
  .hint {
    margin-top: 1rem;
    font-size: 0.75rem;
    color: #9ca3af;
    text-align: center;
  }
  .svelte-badge {
    margin-top: 0.75rem;
    font-size: 0.7rem;
    color: #6b7280;
    text-align: center;
    padding: 0.25rem 0.5rem;
    background: #e5e7eb;
    border-radius: 0.375rem;
    display: inline-block;
    width: 100%;
    box-sizing: border-box;
  }
</style>
