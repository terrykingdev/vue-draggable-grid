<template>
    <v-card
        :class="getCardClass"
        @mouseenter="addClass()"
        @mouseleave="removeClass()"
        dark
        >
        <v-menu :close-on-content-click=false left v-model="popup">
        <template v-slot:activator="{ on, attrs }">
            <v-icon v-bind="attrs" v-on="on" @mousedown.stop @mouseup.stop class="cog ml-auto">mdi-cog</v-icon>
        </template>

            <v-card dense>
                <v-list>
                <v-list-item>
                    <v-list-item-content>
                    <v-list-item-title>Card 3 Options</v-list-item-title>
                    </v-list-item-content>

                    <v-list-item-action class="d-flex flex-row">
                        <v-tooltip top>
                        <template v-slot:activator="{ on, attrs }">
                            <v-btn
                                icon
                                v-bind="attrs"
                                v-on="on"
                                >
                                <v-icon @click="duplicate">mdi-plus-circle-multiple-outline</v-icon>
                            </v-btn>                    
                        </template>
                        <span>Duplicate this card</span>
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

                <v-divider></v-divider>
                <v-select
                :items="items"
                filled
                dense
                label="Filled style"
                v-model="selectedColour"
                @change="newBackground"
                ></v-select>
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
            <div>{{ title }}</div>
        </v-card-title>
        <v-card-text>
            {{ text }}
        </v-card-text>
    </v-card>
</template>

<script>
export default {
    name:'Card3',
    props: ['initialize'],
    data: () => ({
        popup:false,
        selectedColour: null,
        cogitem: null,
        cogClass: '',
        title: '',
        text: '',
        background: null,
        cols:{cols:12},
        items: ['teal', 'purple', 'lime lighten-4'],
    }),
    mounted(){
        console.log("mounted",JSON.stringify(this.initialize))
        if (this.initialize.cols) this.cols = this.initialize.cols
        this.title = this.initialize.options.title?  this.initialize.options.title:'Card 3 Title'
        this.text = this.initialize.options.text?  this.initialize.options.text:'Card 3 Text'
        this.background = this.initialize.options.background?  this.initialize.options.background:'teal'
        // Need to send the setup straight back after it's mounted
        if (this.initialize.refresh){
            this.text = "Card text 3: "+Date.now()
            this.initialize.refresh=false
        }
        this.$emit("update", this.getSetup())
    },
    computed:{
        getCardClass(){
            return this.background+" pa-2 "+this.cogClass
        },
    },
    methods: {
        newBackground(){
            this.background = this.selectedColour
            this.update()
        },
        update(){
            this.$emit("update", this.getSetup())
        },
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
