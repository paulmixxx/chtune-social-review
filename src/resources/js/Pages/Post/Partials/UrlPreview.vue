<script setup>
defineProps({
    preview: Object,
    url: String
})

</script>

<template>
    <a :href="url"
       target="_blank"
       v-if="preview && preview.title"
       class="block mt-4 border border-sky-200 dark:border-neutral-700 text-gray-800 bg-sky-50 dark:bg-neutral-50">
        <img class="max-w-full"
             :src="preview.image"
             :alt="preview.title">
        <div class="p-2">
            <h3 class="font-semibold">{{ preview.title }}</h3>
            <p class="text-sm m-0">{{ preview.description }}</p>
        </div>
    </a>
    <a :href="url"
       target="_blank"
       v-else-if="url"
       class="block mt-4 border border-indigo-200 bg-indigo-50">
        <div class="p-2">{{ url }}</div>
    </a>
</template>

<style scoped>

</style>
