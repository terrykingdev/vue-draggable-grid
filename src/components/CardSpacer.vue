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
                    <v-list-item-content>
                    <v-list-item-title>Card Spacer Options</v-list-item-title>
                    </v-list-item-content>

                    <v-list-item-action class="d-flex flex-row">
                        <v-tooltip top>
                        <template v-slot:activator="{ on, attrs }">
                            <v-btn
                                icon
                                v-bind="attrs"
                                v-on="on"
                                >
                                <v-icon @click="reduceWidth">mdi-minus-circle</v-icon>
                            </v-btn>                    
                        </template>
                        <span>Reduce Width</span>
                        </v-tooltip>
                        <v-tooltip top>
                        <template v-slot:activator="{ on, attrs }">
                            <v-btn
                                icon
                                v-bind="attrs"
                                v-on="on"
                                >
                                <v-icon @click="increaseWidth">mdi-plus-circle</v-icon>
                            </v-btn>                    
                        </template>
                        <span>Increase Width</span>
                        </v-tooltip>
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
        cols:{cols:12,md:12,lg:12,xl:12},
        colOrder: ['cols','xs','sm','md','lg','xl'],   
    }),
    mounted(){
        console.log("mounted",JSON.stringify(this.initialize))
        if (this.initialize.cols) this.cols = this.initialize.cols
            if (this.initialize.refresh){
                this.initialize.refresh=false
            }
            this.update(this.initialize.save!=undefined) // update parent and save if needed
    },
    computed:{
        getCardClass(){
            return "transparent pa-0 "+this.cogClass
        },
    },
    methods: {
        reduceWidth(){
            let bp=this.$vuetify.breakpoint.name
            let currentCols=this.cols[bp]
            if (!this.cols[bp]){
                let i=this.colOrder.indexOf(bp)
                do{
                    i--
                } while (!this.cols[this.colOrder[i]] && i>0)
                currentCols = this.cols[this.colOrder[i]]
            }
            console.log(currentCols)
            if (currentCols>1){
                currentCols--
            }
            this.cols[bp]=currentCols
            this.update(true)
        },
        increaseWidth(){
            console.log(this.$vuetify.breakpoint.name)
            let bp=this.$vuetify.breakpoint.name
            let currentCols=this.cols[bp]
            if (!this.cols[bp]){
                let i=this.colOrder.indexOf(bp)
                do{
                    i--
                } while (!this.cols[this.colOrder[i]] && i>0)
                currentCols = this.cols[this.colOrder[i]]
            }
            console.log(currentCols)
            if (currentCols<12){
                currentCols++
            }
            this.cols[bp]=currentCols
            this.update(true)            
        },        
        update(save){
            this.$emit("update", this.getSetup(), save)
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
  color: black !important;
}
</style>
