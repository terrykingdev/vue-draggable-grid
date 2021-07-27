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
    name:'Card4',
    props: ['initialize'],
    data: () => ({
        cogitem: null,
        cogClass: '',
        title: '',
        text: '',
        background: null,
        options: [
            {title: "Configure", fnc: "config"},
            {title: "Duplicate", fnc: "duplicate"},
            {title: "Remove", fnc: "remove"},
        ]
    }),
    mounted(){
        console.log("mounted",JSON.stringify(this.initialize))
        this.title = this.initialize.title?  this.initialize.title:'Card4 Title'
        this.text = this.initialize.text?  this.initialize.text:'Card4 Text'
        this.background = this.initialize.background?  this.initialize.background:'indigo'
    },
    computed:{
        getCardClass(){
            return this.background+" pa-2 "+this.cogClass
        },
    },
    methods: {
        getSetup(){
            return JSON.stringify({
                title: this.title,
                text: this.text,
                background: this.background
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
