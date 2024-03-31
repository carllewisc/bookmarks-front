<template>
	<div class="default">
		<slot />
	</div>
	<Toaster />
</template>

<script setup lang="ts">
import Toaster from '@/components/ui/toast/Toaster.vue'
</script>

<style scoped>
/*.default {
	border: 1px solid #b9e39e;
	padding: 1rem;
	position: relative;
	background-color: #b8e39e87;
}
.default::before {
	content: 'layouts/default.vue';
	position: absolute;
	top: 2px;
	left: 5px;
	color: #68b538;
	font-family: monospace;
}*/
</style>
