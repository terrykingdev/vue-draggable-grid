<template>
    <div
        :class="getCardClass"
        :style="getCardStyle"
        @mouseenter="addClass()"
        @mouseleave="removeClass()"
        >
        <v-menu
            :close-on-content-click=false
            left
            origin="20px 20px"
            offset-x
            v-model="popup"
        >
        <template v-slot:activator="{ on, attrs }">
            <v-icon v-bind="attrs" v-on="on" @mousedown.stop class="cog ml-auto">mdi-dots-vertical</v-icon>
        </template>

            <v-card dense>
                <v-list>
                <v-list-item>
                    <v-list-item-content>
                    <v-list-item-title></v-list-item-title>
                    </v-list-item-content>

                    <v-list-item-action class="d-flex flex-row">

                        <v-tooltip top>
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn icon v-bind="attrs" v-on="on" class="cw">
                                    {{currentBreakpoint}}
                                </v-btn>
                            </template>
                            <span>Current Breakpoint</span>
                        </v-tooltip>                        
                        
                        <v-tooltip top>
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn icon v-bind="attrs" v-on="on" width="24">
                                    <v-icon @click="reduceWidth">mdi-chevron-left-circle</v-icon>
                                </v-btn>
                            </template>
                            <span>Reduce Width</span>
                        </v-tooltip>

                        <v-tooltip top>
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn icon v-bind="attrs" v-on="on" class="cw">
                                    {{currentWidth}}
                                </v-btn>
                            </template>
                            <span>Current Width</span>
                        </v-tooltip>

                        <v-tooltip top>
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn icon v-bind="attrs" v-on="on" width="24">
                                    <v-icon @click="increaseWidth">mdi-chevron-right-circle</v-icon>
                                </v-btn>
                            </template>
                            <span>Increase Width</span>
                        </v-tooltip>

                        <v-tooltip top>
                            <template v-slot:activator="{ on, attrs }">

                                <v-btn icon v-bind="attrs" v-on="on">
                                    <v-icon @click="showColourPicker">mdi-circle-half-full</v-icon>
                                </v-btn>
                            </template>
                            <span>Background Colour</span>
                        </v-tooltip>

                        <v-tooltip top>
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn icon v-bind="attrs" v-on="on">
                                    <v-icon @click="duplicate">mdi-plus-circle-multiple-outline</v-icon>
                                </v-btn>
                            </template>
                            <span>Duplicate this card</span>
                        </v-tooltip>
                        <v-tooltip top>
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn icon v-bind="attrs" v-on="on">
                                    <v-icon @click="remove">mdi-delete</v-icon>
                                </v-btn>
                            </template>
                            <span>Remove this card</span>
                        </v-tooltip>
                        </v-list-item-action>
                </v-list-item>
                </v-list>

                <v-divider></v-divider>
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
		<colour-picker @change="colourPicked" :show="showPicker"></colour-picker>

		<div class="cardtitle" :style="contrastColour">{{getTitle}}</div>
		<div class="cardcontent" :style="contrastColour+';height:calc(100% - 26px)'">
            {{text}}
		</div>
    </div>
</template>

<script>
import ColourPicker from '../components/ColourPicker'

export default {
    name:'WidgetBored',
    props: ['initialize'],
    components: {
        ColourPicker
    },
    data: () => ({
        drawTimer: null,
        showPicker: false,
        popup:false,
        cogitem: null,
        cogClass: '',
        background: null,
        text: '',
        cols:{cols:12,xs:12,sm:12,md:6,lg:4,xl:1},
        colOrder: ['cols','xs','sm','md','lg','xl'],
        items: ['red', 'green', 'blue'],
    }),
    mounted(){
        console.log("mounted",JSON.stringify(this.initialize))
        if (this.initialize.cols){
            for(let [k,v] of Object.entries(this.initialize.cols)){
                this.cols[k]=v
            }
        }
        this.useImage = this.initialize.useImage?this.initialize.useImage:false
        this.background = this.initialize.background?  this.initialize.background:'#1976d2'
        this.text = this.initialize.options.text?  this.initialize.options.text:''
        // Need to send the setup straight back after it's mounted
        if (this.initialize.refresh){
            this.getBored()
            this.initialize.refresh=false
        }
        this.update(this.initialize.save!=undefined) // update parent and save if needed
    },
    destroyed(){
        clearInterval(this.drawTimer)
    },
    computed:{
        getTitle() {
            return 'Bored?'
        },
        getContrastColour(){
            let c=this.background
            if (this.background=='rgba(0,0,0,0)'){
                c=this.$vuetify.theme.currentTheme.background
            }				
            return this.rgbToLum(c) < 128 ? 'white' : 'black'
        },
        contrastColour() {
            return `color:${this.getContrastColour};`
        },
        currentBreakpoint(){
            return this.$vuetify.breakpoint.name
        },
        getCardClass(){
            return "card pa-2 "+this.cogClass
        },
        getCardStyle(){
            return `background-color:${this.background};`
        },
        currentWidth(){
            return this.cols[this.$vuetify.breakpoint.name]
        },
    },
    methods: {
        getBored(){
            fetch('https://www.boredapi.com/api/activity')
            .then(response => response.json())
            .then(data => {
                console.log(data)
                this.text=data.activity
                this.update(true)                    
            })
        },
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
            if (bp=='xs') this.cols.cols=currentCols
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
            if (bp=='xs') this.cols.cols=currentCols
            this.update(true)            
        },
        showColourPicker() {
            this.showPicker = true
        },        
        colourPicked(colour) {
            this.background = colour.value
            this.showPicker = false
            this.update(true)
        },
        rgbToLum(c) {
            if (c && c.match(/#....../)) {
                // Calculate the luminance of the background to decide whether cog should be black or white
                let r = parseInt(c.substring(1, 3), 16)
                let g = parseInt(c.substring(3, 5), 16)
                let b = parseInt(c.substring(5, 7), 16)
                let lum = (0.299 * r) + (0.587 * g) + (0.114 * b)
                return lum
            }
            return 0
        },        
        update(save){
            this.$emit("update", this.getSetup(),save)
        },
        getSetup(){
            let obj={
                name: this.$options.name,
                cols: this.cols,
                background: this.background,
                options: {
                    text: this.text,
                }
            }
            return JSON.stringify(obj)
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
.card{
    border-radius: 8px!important;
    box-shadow:
    0px 3px 1px -2px rgb(0 0 0 / 20%), 0px 2px 2px 0px rgb(0 0 0 / 14%), 0px 1px 5px 0px rgb(0 0 0 / 12%);
}
.cardtitle{
    padding: 8px;
    font-size: 1.2rem;
    font-weight: 500;
}
.cardcontent{
	text-align:center;
	font-size:1.5em;
    margin-top: -14px;	
	padding:6px;
}
.cardicon{
    display:inline-block;
    position: absolute;
    right: 34px;
}
</style>
