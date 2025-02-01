<script lang="ts">
	import { onDestroy, onMount, tick } from 'svelte';
	import Post from './post.svelte';
	import { browser } from '$app/environment';
	import { images, texts } from './content';
	import { unicode } from '$lib/unicode';
	import Head from '$lib/head.svelte';

	let posts: {
		image: { alt: string; src: string; href: string };
		text: string;
		tellRectExport: { tellRect?: () => DOMRect | null };
		isLiked: boolean;
		isDisliked: boolean;
		comments: string[];
	}[] = $state([]);

	const likePost = (index: number) => {
		posts[index].isDisliked = false;
		posts[index].isLiked = !posts[index].isLiked;
		posts = posts;
	};

	const dislikePost = (index: number) => {
		posts[index].isLiked = false;
		posts[index].isDisliked = !posts[index].isDisliked;
		posts = posts;
	};

	const addComment = (index: number, content: string) => {
		if (content.length === 0) return;
		posts[index].comments.push(content);
		posts = posts;
	};

	const selectPostText = () => {
		if (posts.length === 0) return texts[0];
		// eslint-disable-next-line no-constant-condition
		while (true) {
			const postText = texts[Math.floor(Math.random() * texts.length)];
			let acceptText = true;
			for (let i = posts.length - 1; i >= 0 && i >= posts.length - 5; i--) {
				if (posts[i].text === postText) {
					acceptText = false;
					break;
				}
			}
			if (acceptText) return postText;
		}
	};

	const selectImage = () => {
		if (posts.length === 0) return images[0];
		// eslint-disable-next-line no-constant-condition
		while (true) {
			const image = images[Math.floor(Math.random() * images.length)];
			let acceptImage = true;
			for (let i = posts.length - 1; i >= 0 && i >= posts.length - 5; i--) {
				if (posts[i].image === image) {
					acceptImage = false;
					break;
				}
			}
			if (acceptImage) return image;
		}
	};

	const addPost = async () => {
		posts.push({
			image: selectImage(),
			text: selectPostText(),
			tellRectExport: {},
			isLiked: false,
			isDisliked: false,
			comments: []
		});
		posts = posts;
		await tick();
	};

	const conditionallyAddPosts = async () => {
		// eslint-disable-next-line no-constant-condition
		while (true) {
			if (posts.length < 1) {
				await addPost();
				continue;
			}
			const lastPost = posts[posts.length - 1];
			const lastRect = lastPost.tellRectExport.tellRect?.();
			if (!lastRect) return;
			if (lastRect.bottom < Math.max(200, window.innerHeight) * 2) {
				await addPost();
				continue;
			}
			return;
		}
	};

	const removePosts = async () => {
		if (posts.length < 1) return;
		const firstRect = posts[0].tellRectExport.tellRect?.();
		if (!firstRect) return;
		if (firstRect.bottom > -200) return;
		let firstIndex = 0;
		let lastIndex = posts.length - 1;
		while (lastIndex - firstIndex > 1) {
			const middleIndex = Math.floor((firstIndex + lastIndex) / 2);
			const middleRect = posts[middleIndex].tellRectExport.tellRect?.();
			if (!middleRect) return;
			if (middleRect.bottom > -200) {
				lastIndex = middleIndex;
			} else {
				firstIndex = middleIndex;
			}
		}
		const lastRect = posts[lastIndex].tellRectExport.tellRect?.();
		if (!lastRect) return;
		const delta = lastRect.top - firstRect.top;
		const newScrollPosition = window.scrollY - delta;
		posts = posts.slice(lastIndex);
		await tick();
		window.scroll({
			top: newScrollPosition,
			behavior: 'instant'
		});
	};

	let isManagePostsRunning = false;

	const managePosts = async () => {
		if (isManagePostsRunning) return;
		isManagePostsRunning = true;
		await conditionallyAddPosts();
		await removePosts();
		isManagePostsRunning = false;
	};

	onMount(() => {
		if (!browser) return;
		addEventListener('scroll', managePosts);
		managePosts();
	});

	onDestroy(() => {
		if (!browser) return;
		removeEventListener('scroll', managePosts);
	});
</script>

<Head
	canonicalUrl="https://aaronstanek.com/projects/doom-scroll-simulator"
	title="Doom Scroll Simulator"
	description="The ultimate social media experience you never asked for! Repetitive AI-generated nonsense, liking posts that no one cares about, and leaving comments destined to be ignored."
	image="https://aaronstanek.com/static-web-content/doom-scroll-simulator/eiffel.jpg"
/>

<h1 class="text-center mt-2 mb-2">Doom Scroll Simulator</h1>

<div class="intro">
	Welcome to Doom Scroll Simulator{unicode.em}the ultimate social media experience you never asked
	for! Here, you'll enjoy all the thrilling features of your favorite platforms, like endlessly
	scrolling through repetitive AI-generated nonsense, liking posts that no one cares about, and
	leaving comments destined to be ignored.
</div>

<div class="intro">
	But that's not all! Just like the competition, Doom Scroll Simulator delivers the full package:
	existential dread, social isolation, and a sprinkle of depression. It's like hanging out on every
	other social app{unicode.ellipsis} but with more honesty.
</div>

<div class="intro">
	Now, we do have one small drawback: we're still working on key features like spying on your every
	move and selling your data for targeted ads. But hey, we'll get there{unicode.em}eventually.
</div>

{#each posts as post, index}
	<Post
		{index}
		alt={post.image.alt}
		src={post.image.src}
		href={post.image.href}
		text={post.text}
		tellRectExport={post.tellRectExport}
		isLiked={post.isLiked}
		{likePost}
		isDisliked={post.isDisliked}
		{dislikePost}
		comments={post.comments}
		{addComment}
	/>
{/each}

<style>
	.intro {
		background-color: white;
		color: black;
		padding: 20px;
	}
</style>
