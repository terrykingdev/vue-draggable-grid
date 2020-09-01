<template>
    <v-card
        class="success pa-2"
        dark
        >
        {{ content }}
    </v-card>
</template>

<script>
export default {
    name:'Card2',
    props: ['content']
}
</script>