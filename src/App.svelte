<script lang="ts">
  import { onMount } from 'svelte';
  type Todo = { id: string; title: string; completed: boolean };
  let todos = $state<Todo[]>([]);
  let title = $state('');
  let loading = $state(true);
  let busy = $state(false);
  let error = $state('');
  const finished = $derived(todos.filter(t => t.completed).length);

  // Path /api diteruskan oleh proxy Vite ke server Bun port 3000.
  async function api<T>(path = '', options: RequestInit = {}): Promise<T> {
    const res = await fetch(`/api/todos${path}`, {
      ...options, headers: { 'Content-Type': 'application/json' }
    });
    if (!res.ok) {
      const data = await res.json().catch(() => ({}));
      throw new Error(data.message || `Permintaan gagal (${res.status})`);
    }
    return res.status === 204 ? undefined as T : await res.json();
  }
  async function load() {
    loading = true; error = '';
    try { todos = await api<Todo[]>(); }
    catch (e) { error = e instanceof Error ? e.message : 'Gagal memuat tugas'; }
    finally { loading = false; }
  }
  async function mutate(action: () => Promise<void>) {
    if (busy) return;
    busy = true; error = '';
    try { await action(); }
    catch (e) { error = e instanceof Error ? e.message : 'Permintaan gagal'; }
    finally { busy = false; }
  }
  function add(event: SubmitEvent) {
    event.preventDefault();
    if (!title.trim()) return;
    void mutate(async () => {
      const todo = await api<Todo>('', { method: 'POST', body: JSON.stringify({ title }) });
      todos = [...todos, todo]; title = '';
    });
  }
  function toggle(todo: Todo) {
    void mutate(async () => {
      const updated = await api<Todo>(`/${todo.id}`, { method: 'PATCH', body: JSON.stringify({ completed: !todo.completed }) });
      todos = todos.map(t => t.id === updated.id ? updated : t);
    });
  }
  function remove(id: string) {
    void mutate(async () => {
      await api<void>(`/${id}`, { method: 'DELETE' });
      todos = todos.filter(t => t.id !== id);
    });
  }
  onMount(() => { void load(); });
</script>

<main>
  <p class="eyebrow">To Do Kesyfi Anand</p>
  <h1>Daftar tugas</h1>
  <p class="intro">Mulai dari satu hal kecil hari ini.</p>
  <form onsubmit={add}>
    <label for="title">Tugas baru</label>
    <div class="input-row">
      <input id="title" bind:value={title} placeholder="Contoh: Belajar REST API" maxlength="120" required disabled={busy || loading} />
      <button disabled={busy || loading || !title.trim()}>Tambah</button>
    </div>
  </form>
  {#if error}<div class="error" role="alert">{error} <button type="button" onclick={load} disabled={busy || loading}>Muat ulang</button></div>{/if}
  <div class="summary"><h2>Tugas saya</h2><span>{finished}/{todos.length} selesai</span></div>
  {#if loading}
    <p role="status">Memuat tugas…</p>
  {:else if todos.length === 0}
    <p class="empty">Belum ada tugas. Tambahkan tugas pertamamu.</p>
  {:else}
    <ul>
      {#each todos as todo (todo.id)}
        <li>
          <button class="task" class:done={todo.completed} aria-pressed={todo.completed} aria-label={`${todo.completed ? 'Batalkan selesai' : 'Selesaikan'}: ${todo.title}`} onclick={() => toggle(todo)} disabled={busy}>
            <span class="check" aria-hidden="true">{todo.completed ? '✓' : '○'}</span><span>{todo.title}</span>
          </button>
          <button class="delete" aria-label={`Hapus ${todo.title}`} onclick={() => remove(todo.id)} disabled={busy}>Hapus</button>
        </li>
      {/each}
    </ul>
  {/if}
  <footer>Selesaikannn</footer>
</main>

<style>
  :global(*){box-sizing:border-box} :global(body){margin:0;background:#f2f7f4;color:#186ccc;font-family:poppins,sans-serif}
  main{max-width:650px;margin:70px auto;padding:32px;background:white;border:1px solid #dfebe3;border-radius:18px}
  .eyebrow{font-size:11px;font-weight:700;letter-spacing:2px;color:#425882}h1{font-size:36px;margin:14px 0 8px}.intro{color:#186ccc;margin-bottom:32px}
  label{display:block;font-size:24px;font-weight:600;margin-bottom:8px}.input-row{display:flex;gap:10px}input{min-width:0;flex:1;padding:13px;border:1px solid #bfcec5;border-radius:8px;font:inherit}
  button{font:inherit;cursor:pointer;border:0;border-radius:8px;padding:12px 17px;background:#191f73;color:white}button:disabled{opacity:.5;cursor:not-allowed}button:focus-visible,input:focus-visible{outline:3px solid #186ccc;outline-offset:3px}
  .summary{display:flex;justify-content:space-between;align-items:center;margin-top:28px}h2{font-size:16px}.summary span{font-size:12px;color:#69707e}
  ul{list-style:none;padding:0}li{display:flex;align-items:center;gap:8px;border-bottom:1px solid #edf1ed;padding:10px 0}.task{flex:1;display:flex;text-align:left;gap:12px;align-items:center;background:transparent;color:inherit;padding:8px 0;overflow-wrap:anywhere;min-width:0}.check{color:#193d73}.done span:last-child{text-decoration:line-through;color:#7a8c81}.delete{background:#fff0ed;color:#a14132;font-size:12px;padding:8px 10px}
  .empty{padding:28px 12px;background:#f6faf7;border-radius:8px;text-align:center;font-size:14px;color:#697e72}.error{background:#fff0ed;padding:12px;margin-top:16px;color:#a14132;font-size:14px}.error button{font-size:12px;margin-top:8px}footer{font-size:11px;color:#7a8c81;margin-top:30px}
  @media(max-width:600px){main{margin:20px 12px;padding:22px}h1{font-size:30px}.input-row{flex-wrap:wrap}.input-row button{flex-grow:1}}
</style>
