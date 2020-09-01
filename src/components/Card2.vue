<template>
    <div
        class="success"
        style="height:100px;"
        >
        {{ title }}
        {{ text }}
    </div>
</template>

<script>
export default {
    name:'Card2',
    props: ['title','text']
}
</script>