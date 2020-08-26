<template>
    <v-col
        :cols="item.cols"
        :md="item.md"
        :lg="item.lg"
        >
        <v-card :key="item.id">
            {{ item.text }}
        </v-card>    
    </v-col>
</template>

<script>
export default {
    name:'Card',
    props:['item']
}
</script>