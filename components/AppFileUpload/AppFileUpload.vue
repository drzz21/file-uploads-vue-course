<script setup>
function handleFileSelect(event) {
	console.log(event);
}
</script>

<template>
	<div>
		<!-- follow along here -->
		<label class="px-4 py-2 bg-blue-500 text-white rounded cursor-pointer">
			<input type="file" @change="handleFileSelect" hidden />
      Upload
		</label>
	</div>
</template>

