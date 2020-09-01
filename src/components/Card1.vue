<template>
    <v-card
        class="primary pa-2"
        dark
        >
        <v-card-title>
            {{ title }}
        </v-card-title>
        <v-card-text>
            {{ text }}
        </v-card-text>
    </v-card>
</template>

<script>
export default {
    name:'Card1',
    props: ['title','text']
}
</script>