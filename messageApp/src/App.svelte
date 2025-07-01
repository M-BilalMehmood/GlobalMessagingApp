<script lang="ts">
    import Login from "./lib/Login.svelte";
    import Messages from "./lib/Messages.svelte";
    import { currentUser } from './lib/pocketbase';

    let page: 'login' | 'signup' | 'chat' = 'login';

    $: if ($currentUser) page = 'chat';
</script>

<div class="app-container">
    <header>
        <h1>Chat in thy Pocket</h1>
    </header>
    {#if page === 'chat' && $currentUser}
        <div class="logout-btn">
            <Login />
        </div>
        <main>
            <Messages />
        </main>
    {:else if page === 'login'}
        <main>
            <Login on:signup={() => page = 'signup'} />
        </main>
    {:else if page === 'signup'}
        <main>
            <Login signupMode={true} on:login={() => page = 'login'} />
        </main>
    {/if}
</div>

<style>
    .app-container {
        min-height: 100vh;
        height: 100vh;
        display: flex;
        flex-direction: column;
        background: #242424;
        color: #e0e0e0;
    }
    .logout-btn {
        position: fixed;
        top: 2rem;
        left: 2rem;
        z-index: 10;
    }
    header {
        width: 100%;
        padding: 2.5rem 0 1.5rem 0;
        background: #242424;
        text-align: center;
        border-bottom: 1px solid #333;
    }
    h1 {
        margin: 0;
        font-size: 2.4rem;
        font-weight: 700;
        letter-spacing: 1px;
        color: #fafafa;
    }
    main {
        flex: 1;
        display: flex;
        justify-content: center;
        align-items: center;
        min-height: 0;
    }
</style>