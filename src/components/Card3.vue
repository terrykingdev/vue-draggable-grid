<template>
    <v-card
        class="teal pa-2"
        dark
        >
        <v-list-item
            v-for="(item,index) in list"
            :key="index"
            >
            <v-list-item-content>
                {{ item.title }}
            </v-list-item-content>
        </v-list-item>
    </v-card>
</template>

<script>
export default {
    name:'Card3',
    props: ['list'],
}


</script>