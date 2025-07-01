<script lang="ts">
    import { onMount, onDestroy, afterUpdate } from 'svelte';
    import { currentUser, pb } from './pocketbase';

    let newMessage: string;
    let messages: any[] = [];
    let unsubscribe: () => void;
    let messagesContainer: HTMLDivElement | null = null; // For scrolling

    $: userId = $currentUser?.id;

    onMount(async () => {
        const resultList = await pb.collection('messages').getList(1, 50, {
            sort: 'created',
            expand: 'user',
        });
        messages = resultList.items;

        unsubscribe = await pb
            .collection('messages')
            .subscribe('*', async (e) => {
                if (e.action === 'create') {
                    const user = await pb.collection('users').getOne(e.record.user);
                    messages = [...messages, { ...e.record, expand: { user } }];
                } else if (e.action === 'update') {
                    const user = await pb.collection('users').getOne(e.record.user);
                    messages = messages.map((msg) =>
                        msg.id === e.record.id ? { ...e.record, expand: { user } } : msg
                    );
                } else if (e.action === 'delete') {
                    messages = messages.filter((msg) => msg.id !== e.record.id);
                }
            });
    });

    onDestroy(() => {
        unsubscribe?.();
    });

    async function sendMessage() {
        if (!$currentUser) {
            alert('You need to be logged in to send a message.');
            return;
        }
        if (!newMessage.trim()) {
            alert('Cannot send an empty message.');
            return;
        }
        try {
            const data = {
                text: newMessage,
                user: $currentUser.id,
            };
            await pb.collection('messages').create(data);
            newMessage = '';
        } catch (err) {
            alert('Failed to send message.');
        }
    }

    // Auto-scroll to bottom when messages change
    afterUpdate(() => {
        if (messagesContainer) {
            messagesContainer.scrollTop = messagesContainer.scrollHeight;
        }
    });
</script>

<div class="chat-container">
    <p class="signed-as">Signed in as {$currentUser?.email}</p>
    <div class="messages" bind:this={messagesContainer}>
        {#each messages as message (message.id)}
            <div class="msg {message.expand.user.id === userId ? 'mine' : 'theirs'}">
                <img
                    class="avatar"
                    src={`https://api.dicebear.com/9.x/fun-emoji/svg?seed=${message.expand.user.username || message.expand.user.email}`}
                    alt="Avatar"
                    width="40"
                />
                <div class="content">
                    <div class="meta">
                        <span class="sender">
                            {#if message.expand?.user}
                                {#if message.expand.user.id === userId}
                                    <span class="you-label">(You)</span>
                                {:else}
                                    {message.expand.user.username || message.expand.user.email}
                                {/if}
                            {:else}
                                Unknown User
                            {/if}
                        </span>
                        <span class="timestamp">
                            {new Date(message.created).toLocaleTimeString([], { hour: '2-digit', minute: '2-digit' })}
                        </span>
                    </div>
                    <p>{message.text}</p>
                </div>
            </div>
        {/each}
    </div>
    <form class="msg-form" on:submit|preventDefault={sendMessage}>
        <input
            type="text"
            bind:value={newMessage}
            placeholder="Type your message…"
            required
        />
        <button type="submit">Send</button>
    </form>
</div>

<style>
    :global(body) {
        height: 100vh;
        overflow: hidden;
    }
    .chat-container {
        width: 100%;
        max-width: 1200px;
        height: 90%;
        background: #292929;
        border-radius: 18px;
        box-shadow: 0 2px 32px #0008;
        display: flex;
        flex-direction: column;
        justify-content: space-between;
        position: relative;
        margin: 0 auto;
    }
    .messages {
        flex: 1 1 0;
        overflow-y: auto;
        padding: 0.2rem 1.2rem 1.2rem 1.2rem;
        display: flex;
        flex-direction: column;
        gap: 1.2rem;
        min-height: 0;
    }
    .msg {
        display: flex;
        align-items: flex-end;
        gap: 0.8rem;
        max-width: 70%;
        background: #232323;
        border-radius: 14px;
        padding: 0.7rem 1rem;
        box-shadow: 0 1px 4px #0002;
        transition: background 0.2s;
        position: relative;
    }
    .msg.mine {
        margin-left: auto;
        flex-direction: row-reverse;
        background: #3535a3;
        color: #fff;
        box-shadow: 0 2px 8px #3535a355;
    }
    .msg.mine .avatar {
        border: 2px solid #646cff;
    }
    .msg.theirs {
        margin-right: auto;
        background: #232323;
        color: #fafafa;
    }
    .avatar {
        border-radius: 50%;
        background: #181818;
        border: 2px solid #333;
    }
    .content {
        flex: 1;
        display: flex;
        flex-direction: column;
        align-items: flex-start;
    }
    .msg.mine .content {
        align-items: flex-end;
    }
    .meta {
        display: flex;
        align-items: center;
        gap: 0.7rem;
        margin-bottom: 0.15rem;
    }
    .sender {
        font-weight: 600;
        font-size: 0.98rem;
        color: #8ab4f8;
        opacity: 0.95;
    }
    .msg.mine .sender {
        color: #ffe082;
    }
    .timestamp {
        color: #888;
        font-size: 0.8rem;
        margin-top: 0.1rem;
        display: block;
        opacity: 0.7;
    }
    .content p {
        margin: 0.2rem 0 0.1rem 0;
        color: inherit;
        font-size: 1.08rem;
        word-break: break-word;
        background: none;
    }
    .msg-form {
        display: flex;
        gap: 1rem;
        padding: 1.2rem;
        border-top: 1px solid #333;
        background: #292929;
        border-radius: 0 0 18px 18px;
        position: sticky;
        bottom: 0;
    }
    .msg-form input {
        flex: 1 1 0;
        min-width: 500px;
        max-width: 100%;
        background: #232323;
        border: 1px solid #333;
        color: #fafafa;
        padding: 0.8rem 1.1rem;
        border-radius: 10px;
        font-size: 1.08rem;
        outline: none;
        transition: border 0.2s;
    }
    .msg-form input:focus {
        border: 1.5px solid #646cff;
    }
    .msg-form button {
        flex: 0 0 auto;
        background: #646cff;
        color: #fff;
        border: none;
        border-radius: 10px;
        padding: 0.8em 1.4em;
        font-size: 1.08em;
        font-weight: 500;
        cursor: pointer;
        transition: background 0.2s;
    }
    .msg-form button:hover {
        background: #535bf2;
    }
    .you-label {
        color: #ffe082;
        font-size: 0.92em;
        margin-left: 0.3em;
        opacity: 0.8;
        font-weight: 400;
    }
</style>