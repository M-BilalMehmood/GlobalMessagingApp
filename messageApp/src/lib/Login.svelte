<script lang="ts">
    import { currentUser, pb } from './pocketbase'

    let email: string = '';
    let password: string = '';
    let username: string = '';
    let errorMessage: string = '';
    let isSignup = false;
    // Basic email validation
    function isValidEmail(email: string): boolean {
        const emailRegex = /^[^\s@]+@[^\s@]+\.[^\s@]+$/;
        return emailRegex.test(email);
    }

    async function login() {
        errorMessage = '';
        try {
            console.log('Attempting login with:', email);
            await pb.collection('users').authWithPassword(email, password);
            console.log('Login successful');
        } catch (error) {
            console.error('Login failed:', error);
            errorMessage = 'Login failed. Please check your credentials.';
        }
    }

    async function signUp() {
        errorMessage = '';
        
        // Client-side validation
        if (!isValidEmail(email)) {
            errorMessage = 'Please enter a valid email address.';
            return;
        }
        
        if (password.length < 8) {
            errorMessage = 'Password must be at least 8 characters long.';
            return;
        }
        
        if (!username) {
            errorMessage = 'Please enter a username.';
            return;
        }
        
        try {
            console.log('Attempting signup with email:', email);
            
            const data = {
                email: email,
                password: password,
                passwordConfirm: password,
                username
            };
            
            const record = await pb.collection('users').create(data);
            console.log('User created successfully:', record);
            
            // Auto-login after successful signup
            await pb.collection('users').authWithPassword(email, password);
            console.log('Auto-login successful');
            
        } catch (error: unknown) {
            console.error('Sign up failed:', error);
            
            // Extract more specific error information if available
            if (typeof error === 'object' && error !== null) {
                const err = error as any;
                if (err.data?.data) {
                    const fieldErrors = Object.entries(err.data.data)
                        .map(([field, errors]) => `${field}: ${errors}`)
                        .join(', ');
                    errorMessage = `Validation failed: ${fieldErrors}`;
                } else {
                    errorMessage = err.message || 'Failed to create account. Please try again.';
                }
            } else {
                errorMessage = 'Failed to create account. Please try again.';
            }
        }
    }

    function signOut() {
        pb.authStore.clear();
    }
</script>

{#if $currentUser}
    <button class="logout" on:click={signOut}>⟵ Logout</button>
{:else}
    {#if isSignup}
        <form class="login-form" on:submit|preventDefault={signUp}>
            <h2>Sign up</h2>
            {#if errorMessage}
                <div class="error-message">{errorMessage}</div>
            {/if}
            <input type="text" bind:value={username} placeholder="Username" required />
            <input type="email" bind:value={email} placeholder="Email" required autocomplete="username" />
            <input type="password" bind:value={password} placeholder="Password" required minlength="8" autocomplete="new-password" />
            <div class="actions">
                <button type="submit" disabled={!email || !password || !username}>Sign Up</button>
                <button type="button" on:click={() => isSignup = false}>Go to Login</button>
            </div>
        </form>
    {:else}
        <form class="login-form" on:submit|preventDefault={login}>
            <h2>Login</h2>
            {#if errorMessage}
                <div class="error-message">{errorMessage}</div>
            {/if}
            <input type="email" bind:value={email} placeholder="Email" required autocomplete="username" />
            <input type="password" bind:value={password} placeholder="Password" required minlength="8" autocomplete="current-password" />
            <div class="actions">
                <button type="submit" disabled={!email || !password}>Login</button>
                <button type="button" on:click={() => isSignup = true}>Go to Signup</button>
            </div>
        </form>
    {/if}
{/if}

<style>
    .logout {
        background: none;
        border: none;
        color: #bdbdbd;
        font-size: 1rem;
        cursor: pointer;
        padding: 0.5rem 1rem;
        border-radius: 6px;
        transition: background 0.2s;
    }
    .logout:hover {
        background: #333;
        color: #fff;
    }
    .login-form {
        display: flex;
        flex-direction: column;
        gap: 1.2rem;
        background: none;
    }
    .login-form h2 {
        color: #fafafa;
        font-size: 1.3rem;
        margin-bottom: 0.5rem;
        font-weight: 600;
        text-align: center;
    }
    .login-form input {
        background: #232323;
        border: 1px solid #333;
        color: #fafafa;
        padding: 0.8rem 1rem;
        border-radius: 8px;
        font-size: 1rem;
        outline: none;
        transition: border 0.2s;
    }
    .login-form input:focus {
        border: 1.5px solid #646cff;
    }
    .actions {
        display: flex;
        gap: 1rem;
        justify-content: center;
    }
    .login-form button {
        background: #646cff;
        color: #fff;
        border: none;
        border-radius: 8px;
        padding: 0.7em 1.5em;
        font-size: 1em;
        font-weight: 500;
        cursor: pointer;
        transition: background 0.2s;
    }
    .login-form button:disabled {
        background: #444;
        color: #888;
        cursor: not-allowed;
    }
    .login-form button:hover:not(:disabled) {
        background: #535bf2;
    }
    .error-message {
        color: #ff5252;
        background: #2a1a1a;
        border: 1px solid #ff5252;
        border-radius: 6px;
        padding: 0.5rem 1rem;
        text-align: center;
        font-size: 0.95rem;
    }
</style>