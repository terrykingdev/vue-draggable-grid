<template>
    <v-card
        :class="getCardClass"
        @mouseenter="addClass()"
        @mouseleave="removeClass()"
        dark
        flat
        tile
        >
        <v-menu :close-on-content-click=false left v-model="popup">
        <template v-slot:activator="{ on, attrs }">
            <v-icon v-bind="attrs" v-on="on" @mousedown.stop @mouseup.stop class="cog ml-auto">mdi-cog</v-icon>
        </template>
            <v-card dense>
                <v-list>
                <v-list-item>
                    <v-list-item-action class="d-flex flex-row">
                        <v-tooltip top>
                        <template v-slot:activator="{ on, attrs }">
                            <v-btn
                                icon
                                v-bind="attrs"
                                v-on="on"
                                >
                                <v-icon @click="remove">mdi-delete</v-icon>
                            </v-btn>                    
                        </template>
                        <span>Remove this card</span>
                        </v-tooltip>
                    </v-list-item-action>
                </v-list-item>
                </v-list>
                <v-card-actions class="my-0">
                <v-spacer></v-spacer>
                <v-btn
                    color="primary"
                    text
                    @click="popup=false"
                    >
                    Close
                </v-btn>
                </v-card-actions>
            </v-card>

        </v-menu>            

        <v-card-title class="d-flex">
        </v-card-title>
    </v-card>
</template>

<script>
export default {
    name:'CardSpacer',
    props: ['initialize'],
    data: () => ({
        popup:false,
        cogitem: null,
        cogClass: '',
        cols:{cols:12},
    }),
    mounted(){
        console.log("mounted",JSON.stringify(this.initialize))
        // Need to send the setup straight back after it's mounted
        this.$emit("update", this.getSetup())
    },
    computed:{
        getCardClass(){
            return "transparent pa-0 "+this.cogClass
        },
    },
    methods: {
        update(){
            this.$emit("update", this.getSetup())
        },
        getSetup(){
            return JSON.stringify({
                name: this.$options.name,
                cols: this.cols,
                options: {
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
        remove(){
            this.$emit("remove")
        },
    }
}
</script>
<style scoped>
.transparent{
    background: rgba(255,255,255,0)
}
.cog{
  position: absolute;
  right: 8px;
  top: 8px;
  cursor: -webkit-pointer;
  cursor: pointer;
  opacity: 0;
}
.showcog .cog{
  opacity: 1;
  color: white !important;
}
</style>
