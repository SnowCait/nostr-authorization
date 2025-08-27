<script lang="ts" module>
	import type { Event, Nip07 } from 'nostr-typedef';

	declare const window: {
		nostr: Nip07.Nostr | undefined;
	};

	declare global {
		interface DocumentEventMap {
			'clipboard-copy': CustomEvent & { target: HTMLElement };
		}
	}
</script>

<script lang="ts">
	import { onMount } from 'svelte';

	let method = 'GET';
	let url = '';
	let event: Event | undefined;

	const methods = ['GET', 'HEAD', 'POST', 'PUT', 'DELETE', 'CONNECT', 'OPTIONS', 'TRACE', 'PATCH'];
	const kind = 27235;

	async function generate(e: SubmitEvent) {
		e.preventDefault();

		if (!window.nostr) {
			alert('Please install NIP-07 browser extension.');
		}

		try {
			event = await window.nostr?.signEvent({
				kind,
				content: '',
				tags: [
					['u', url],
					['method', method]
				],
				created_at: Math.floor(Date.now() / 1000)
			});
		} catch (error) {
			alert(error);
		}
	}

	onMount(() => {
		import('@github/clipboard-copy-element');
		document.addEventListener('clipboard-copy', (event: CustomEvent & { target: HTMLElement }) => {
			const notice: HTMLElement = event.target.querySelector('.notice')!;
			notice.hidden = false;
			setTimeout(function () {
				notice.hidden = true;
			}, 1000);
		});
	});
</script>

<h1>
	<span>
		Nostr Authorization (<a
			href="https://github.com/nostr-protocol/nips/blob/master/98.md"
			target="_blank"
			rel="noopener noreferrer"
		>
			NIP-98
		</a>)
	</span>
	<span>
		<a
			href="https://github.com/SnowCait/nostr-authorization"
			target="_blank"
			rel="noopener noreferrer"
		>
			<picture>
				<source srcset="github-mark-white.svg" media="(prefers-color-scheme: dark)" />
				<img src="github-mark.svg" alt="GitHub" />
			</picture>
		</a>
	</span>
</h1>

<form onsubmit={generate}>
	<input type="text" bind:value={method} list="methods" required />
	<datalist id="methods">
		{#each methods as method}
			<option value={method}></option>
		{/each}
	</datalist>
	<input type="url" bind:value={url} placeholder="https://example.com/" required />
	<input type="submit" value="Sign with NIP-07" />
</form>

{#if event}
	<h2>
		<span>JSON</span>
		<clipboard-copy for="json">
			<span>Copy</span>
			<span class="notice" hidden>Copied!</span>
		</clipboard-copy>
	</h2>

	<pre><code id="json">{JSON.stringify(event, null, 2)}</code></pre>

	<h2>
		<span>Base64 Encode</span>
		<clipboard-copy for="base64">
			<span>Copy</span>
			<span class="notice" hidden>Copied!</span>
		</clipboard-copy>
	</h2>
	{@const auth = btoa(JSON.stringify(event))}
	<pre><code id="base64">{auth}</code></pre>

	<h2>
		<span>HTTP Request</span>
		<clipboard-copy for="http">
			<span>Copy</span>
			<span class="notice" hidden>Copied!</span>
		</clipboard-copy>
	</h2>
	<pre><code id="http">{`${method} ${url}\nAuthorization: Nostr ${auth}`}</code></pre>
{/if}

<style>
	img {
		width: 2rem;
		height: 2rem;
	}

	clipboard-copy {
		font-size: 1rem;
		font-weight: normal;
		border-radius: 5px;
		padding: 0.5rem;
		background-color: lightgray;
		cursor: pointer;
	}
</style>
