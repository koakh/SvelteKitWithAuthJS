# NOTES

## Links

- [https://javascript.plainenglish.io/how-to-use-authjs-in-sveltekit-eab6aa18a2ca](https://javascript.plainenglish.io/how-to-use-authjs-in-sveltekit-eab6aa18a2ca)

## Setup

Install the SvelteKit adapter & core for AuthJS:

```shell
$ pnpm add @auth/core @auth/sveltekit
```

## Authentication Providers

Next, we need to configure the authentication providers we want to use. For this demo, we’ll use **GitHub**. Create a new file in the src directory called `hooks.server.ts` and add the following code:

```ts
import { SvelteKitAuth } from "@auth/sveltekit"
import GitHub from '@auth/core/providers/github';
import { GITHUB_ID, GITHUB_SECRET } from "$env/static/private"

export const handle = SvelteKitAuth({
  providers: [
    GitHub({
      clientId: GITHUB_ID,
      clientSecret: GITHUB_SECRET
    })
  ]
});
```

The exported handle function is able to **intercept requests to the server** and handle authentication. The `clientId` and `clientSecret` are passed in as environment variables. We’ll set these up next.

## GitHub OAuth

Go to GitHub’s developer settings and create a new OAuth app. Set the callback URL to `http://localhost:5173/auth/callback/github`. The `http://localhost:5173` part is the default URL for SvelteKit. 
If you’re using a different port, or are deploying your app, you’ll need to change this. You’ll also need to set the “Homepage URL” to `http://localhost:5173`. Save the client ID and client secret for the next step.

## Configure Environment Variables

Next, we need to configure the environment variables. Create a new file in the root of the project called `.env` and add the following, replacing the placeholders with your own values. `AUTH_SECRET` must be a random string of 32 characters. The documentation suggests on Linux, you generate one using `openssl rand -hex 32`, or you can navigate to `generate-secret.vercel.app` to generate one in the browser.

```shell
$ openssl rand -hex 32
1f564669508448584b7a63f13661f0e70e60ad54911c63c112f5ecbcfb0a135c
```

```config
GITHUB_ID=<your-github-id>
GITHUB_SECRET=<your-github-secret>
AUTH_SECRET=<auth-secret>
```

## Fetch the Session

Now we need to fetch the session data. Create a new file in the src/routes directory called `+page.server.ts` and export a `load` function that returns the session data:

```ts
import type { PageServerLoad } from "./$types"

export const load: PageServerLoad = async (event) => {
  return {
    session: await event.locals.getSession(),
  }
}
```

### Add Login Actions

Now we need to add some actions to the app. Navigate to `src/routes/+page.svelte` and add the following code:

```svelte
<script>
 import { signIn, signOut } from '@auth/sveltekit/client';
 import { page } from '$app/stores';
</script>

<p>
 {#if Object.keys($page.data.session || {}).length}

    {#if $page.data.session?.user?.image}
      <img
        src={$page.data.session?.user?.image}
        alt={$page.data.session?.user?.name}
        style="width: 64px; height: 64px;" />
      <br/>
  {/if}
  <span>
   <strong>{$page.data.session?.user?.email || $page.data.session?.user?.name}</strong>
  </span>
    <br/>
    <span>Session expires: {new Date($page.data.session?.expires ?? 0).toUTCString()}</span>
    <br/>
  <button on:click={() => signOut()} class="button">Sign out</button>

  {:else}
  <span>You are not signed in</span><br/>
  <button on:click={() => signIn('github')}>Sign In with GitHub</button>
 {/if}
</p>
```

Run the app using npm run dev and navigate to `http://localhost:5173` and try it out!

## Conclusion

**AuthJS is an amazing library** that simplifies authentication for so many different account providers. From GitHub, to Twitter, even just plain old email & password, or magic links, AuthJS has you covered. 