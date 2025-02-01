<script lang="ts">
	import Link from '$lib/links/link.svelte';
	import { onDestroy, onMount } from 'svelte';

	interface Props {
		index: number;
		alt: string;
		src: string;
		href: string;
		text: string;
		tellRectExport: { tellRect?: () => DOMRect | null };
		isLiked: boolean;
		likePost: (index: number) => void;
		isDisliked: boolean;
		dislikePost: (index: number) => void;
		comments: string[];
		addComment: (index: number, content: string) => void;
	}

	let {
		index,
		alt,
		src,
		href,
		text,
		tellRectExport,
		isLiked,
		likePost,
		isDisliked,
		dislikePost,
		comments,
		addComment
	}: Props = $props();

	let div: HTMLDivElement | undefined = $state();

	const tellRect = () => {
		if (div === undefined) return null;
		const rect = div.getBoundingClientRect();
		if (rect.width === 0 || rect.height === 0) return null;
		return rect;
	};

	const registerTellRect = (exporter: { tellRect?: () => DOMRect | null }) => {
		exporter.tellRect = tellRect;
	};

	$effect(() => {
		registerTellRect(tellRectExport);
	});

	let newCommentContent = $state('');

	const addCommentInit = () => {
		addComment(index, newCommentContent);
		newCommentContent = '';
	};

	let newCommentBlock: HTMLTextAreaElement | undefined = $state();

	const addCommentByEnter = (event: KeyboardEvent) => {
		if (event.key === 'Enter') {
			event.preventDefault();
			addCommentInit();
		}
	};

	onMount(() => {
		if (newCommentBlock === undefined) return;
		newCommentBlock.addEventListener('keydown', addCommentByEnter);
	});

	onDestroy(() => {
		if (newCommentBlock === undefined) return;
		newCommentBlock.removeEventListener('keydown', addCommentByEnter);
	});
</script>

<div bind:this={div} class="flex flex-col items-center post">
	<div class="bar"></div>
	<Link {href}><img {alt} {src} /></Link>
	<div class="body-text">{text}</div>
	<div class="flex flex-row justify-start thumb-holder gap-2 mb-3">
		<button class={isLiked ? ' liked' : ''} onclick={() => likePost(index)}
			><img
				class="thumb-up"
				src="https://aaronstanek.com/static-web-content/thumbs-up.svg"
				alt="Like this post"
			/></button
		>
		<button class={isDisliked ? ' disliked' : ''} onclick={() => dislikePost(index)}
			><img
				class="thumb-down"
				src="https://aaronstanek.com/static-web-content/thumbs-up.svg"
				alt="Dislike this post"
			/></button
		>
	</div>
	<div class="flex flex-col gap-3 items-start button-holder">
		{#each comments as comment}
			<div class="flex flex-row gap-1 comment-blob">
				<div class="comment-user">User</div>
				<div class="comment-content">{comment}</div>
			</div>
		{/each}
	</div>
	<textarea
		bind:this={newCommentBlock}
		bind:value={newCommentContent}
		class="mt-3 new-comment"
		placeholder="Add your comment!"
	></textarea>
	<div class="mt-3 flex flex-row justify-end button-holder">
		<button onclick={addCommentInit}>
			<img
				class="submit-comment"
				src="https://aaronstanek.com/static-web-content/left-pointing-arrowhead.svg"
				alt="Post Comment"
			/>
		</button>
	</div>
</div>

<style>
	.post {
		background-color: white;
		padding: 20px;
	}
	.bar {
		background-color: var(--somewhat-white);
		height: 1px;
		width: 90%;
	}
	img {
		margin-top: 20px;
		max-height: 400px;
		max-width: 90%;
		height: auto;
		width: auto;
	}
	.body-text {
		margin-top: 10px;
		max-width: 90%;
		color: black;
	}
	.thumb-holder {
		width: 90%;
	}
	.thumb-up {
		width: 30px;
		transform: scale(-1, 1);
	}
	.thumb-down {
		width: 30px;
		transform: scale(1, -1);
	}
	.liked {
		filter: drop-shadow(0 0 3px yellow);
	}
	.disliked {
		filter: drop-shadow(0 0 3px red);
	}
	.new-comment {
		width: 90%;
		min-height: 20px;
		padding: 20px;
		border-radius: 18px;
		background-color: rgb(240, 242, 245);
		color: black;
		font-size: var(--text-size-minus-2);
	}
	.button-holder {
		width: 90%;
		overflow: hidden;
	}
	.submit-comment {
		width: 25px;
		transform: scale(-1, 1);
	}
	.comment-blob {
		color: black;
		font-size: var(--text-size-minus-2);
	}
	.comment-user {
		padding: 10px;
		border-radius: 18px;
	}
	.comment-content {
		padding: 10px;
		border-radius: 18px;
		background-color: rgb(240, 242, 245);
	}
</style>
