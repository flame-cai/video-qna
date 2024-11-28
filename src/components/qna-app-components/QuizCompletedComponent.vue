<script setup>

const props = defineProps(['score'])

</script>

<template>
    <p>
        Your score is {{ props.score }}
    </p>
</template>