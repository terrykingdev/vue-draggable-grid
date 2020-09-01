<template>
    <div
        class="primary"
        style="height:100px;"
        >
        {{ title }}
        {{ text }}
    </div>
</template>

<script>
export default {
    name:'Card1',
    props: ['title','text']
}
</script>