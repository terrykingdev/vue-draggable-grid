<template>
    <v-card
        :class="getCardClass"
        @mouseenter="addClass()"
        @mouseleave="removeClass()"
        >
        <v-card-title class="d-flex">
            <div>{{ title }}</div>
            <v-menu offset-y>
            <template v-slot:activator="{ on, attrs }">
                <v-icon v-bind="attrs" v-on="on" @mousedown.stop @mouseup.stop class="cog ml-auto">mdi-cog</v-icon>
            </template>
            <v-list dense
              >
              <v-list-item-group
                dark
                >
                <v-list-item
                v-for="(item, index) in options"
                :key="index"
                @click="callfnc(item.fnc)"
                >
                <v-list-item-title>{{ item.title }}</v-list-item-title>
                </v-list-item>
              </v-list-item-group>
            </v-list>
            </v-menu>            
        </v-card-title>
        <v-card-text>
            {{ text }}
        </v-card-text>
    </v-card>
</template>

<script>
export default {
    name:'Card2',
    props: ['initialize'],
    data: () => ({
        cogitem: null,
        cogClass: '',
        title: '',
        text: '',
        background: null,
        cols:{cols:12,md:3},
        options: [
            {title: "Configure", fnc: "config"},
            {title: "Duplicate", fnc: "duplicate"},
            {title: "Remove", fnc: "remove"},
        ]
    }),
    mounted(){
        console.log("mounted",JSON.stringify(this.initialize))
        if (this.initialize.cols) this.cols = this.initialize.cols
        this.title = this.initialize.options.title?  this.initialize.options.title:'Card 2'
        this.text = this.initialize.options.text?  this.initialize.options.text:'Card 2 text'
        this.background = this.initialize.options.background?  this.initialize.options.background:'teal'
        // Need to send the setup straight back after it's mounted
        this.$emit("update", this.getSetup())
    },
    computed:{
        getCardClass(){
            return this.background+" pa-2 "+this.cogClass
        },
    },
    methods: {
        getSetup(){
            return JSON.stringify({
                name: this.$options.name,
                cols: this.cols,
                options: {
                    title: this.title,
                    text: this.text,
                    background: this.background
                }
            })
        },
        addClass(){
            this.cogClass="showcog"
        },
        removeClass(){
            this.cogClass=""
        },
        callfnc(f){
            this[f]()
        },
        config(){
            this.background = 'secondary'
            this.cols = {cols:6,md:2}
            this.$emit("update", this.getSetup())
        },
        duplicate(){
            this.$emit("duplicate", this.getSetup())
        },
        remove(){
            this.$emit("remove")
        },
    }
}
</script>
<style scoped>
.cog{
  cursor: -webkit-pointer;
  cursor: pointer;
  opacity: 0;
}
.showcog .cog{
  opacity: 1;
  color: white !important;
}
</style>
