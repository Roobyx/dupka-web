<script setup>
	import { ref } from 'vue'

	defineProps({
		msg: String
	})

	const count = ref(0)
	const url = window.location
	const imageAddres = url.toString().replace('/', '%2F').replace('http:%2F/localhost:3000/', 'https://firebasestorage.googleapis.com/')
</script>

<template>
	<h1>{{ msg }}</h1>
	<h1> {{ imageAddres }}</h1>
	<img :src="imageAddres" alt="">
	<!-- <img src="https://firebasestorage.googleapis.com/v0/b/dupka-d8175.appspot.com/o/reports%2FbkxtzYlf7UTsO6RqOz7ZSxRzVcw1%2F43aed090-bb5c-43a9-8ece-7b998a551559?alt=media&token=827725fb-5522-45a0-895a-76bff25ff46d" alt=""> -->

	<button type="button" @click="count++">count is: {{ count }}</button>
</template>

<style scoped>
	a {
		color: #42b983;
	}
</style>
