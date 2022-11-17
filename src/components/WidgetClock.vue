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
                                <v-btn icon v-bind="attrs" v-on="on" width="24">
                                    <v-icon @click="decHeight">mdi-chevron-up-circle</v-icon>
                                </v-btn>
                            </template>
                            <span>Reduce Height</span>
                        </v-tooltip>

                        <v-tooltip top>
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn icon v-bind="attrs" v-on="on" class="cw">
                                    {{height}}
                                </v-btn>
                            </template>
                            <span>Current Height</span>
                        </v-tooltip>

                        <v-tooltip top>
                            <template v-slot:activator="{ on, attrs }">
                                <v-btn icon v-bind="attrs" v-on="on" width="24">
                                    <v-icon @click="incHeight">mdi-chevron-down-circle</v-icon>
                                </v-btn>
                            </template>
                            <span>Increase Height</span>
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
		<div ref="clockcontainer" class="cardcontent" style="height:calc(100% - 26px)">
            <canvas
                ref="clock"
            ></canvas>
		</div>
    </div>
</template>

<script>
import ColourPicker from '../components/ColourPicker'

export default {
    name:'WidgetClock',
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
        title: '',
        background: null,
        height: 268,
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
        this.height = this.initialize.height ? this.initialize.height : 136
        this.background = this.initialize.background?  this.initialize.background:'#1976d2'
        this.title = this.initialize.options.title?  this.initialize.options.title:'Clock'
        // Need to send the setup straight back after it's mounted
        if (this.initialize.refresh){
            this.initialize.refresh=false
        }
        this.update(this.initialize.save!=undefined) // update parent and save if needed
        this.drawTimer=setInterval(this.drawClock,1000)
    },
    destroyed(){
        clearInterval(this.drawTimer)
    },
    computed:{
        getTitle() {
            return 'Clock'
        },
        contrastColour() {
            let c=this.background
            if (this.background=='rgba(0,0,0,0)'){
                c=this.$vuetify.theme.currentTheme.background
            }				
            return this.rgbToLum(c) < 128 ? 'color:white;' : 'color:black;'
        },
        currentBreakpoint(){
            return this.$vuetify.breakpoint.name
        },
        getCardClass(){
            return "card pa-2 "+this.cogClass
        },
        getCardStyle(){
            return `background-color:${this.background};height:${this.height}px;`
        },
        currentWidth(){
            return this.cols[this.$vuetify.breakpoint.name]
        },
        getWidth(){
            let clockContainer=this.$refs.clockcontainer
            console.log("clockcontainer",clockContainer)
            return 500
        }
    },
    methods: {
        drawClock(){
            let clock=this.$refs.clock
            let ctx=clock.getContext("2d")
            ctx.beginPath()
            this.$nextTick(()=>{
                let w=clock.clientWidth
                let h=clock.clientHeight
                let min=Math.min(w,h)
                ctx.imageSmoothingEnabled = true
                ctx.canvas.width=w
                ctx.canvas.height=h
                let rad=min/2
                let x=w/2
                let y=h/2
                ctx.lineWidth = 2
                ctx.strokeStyle = "#00000000"
                ctx.arc(x,y,rad,0,2*Math.PI)
                ctx.fillStyle = "white"
                ctx.fill()
                ctx.strokeStyle = "#000000"
                ctx.translate(w/2, h/2)
                ctx.font = rad*0.15 + "px arial"
                ctx.fillStyle = "black"
                ctx.textBaseline="middle"
                ctx.textAlign="center"
                for(let num = 1; num < 13; num++){
                    let ang = num * Math.PI / 6
                    ctx.rotate(ang)
                    ctx.translate(0, -rad*0.85)
                    ctx.rotate(-ang)
                    ctx.fillText(num.toString(), 0, 0)
                    ctx.rotate(ang)
                    ctx.translate(0, rad*0.85)
                    ctx.rotate(-ang)
                }
                for(let num = 0;num<60;num++){
                    let s=Math.sin(2*Math.PI/60*num)
                    let c=Math.cos(2*Math.PI/60*num)
                    let x1=s*(rad*0.98)
                    let y1=c*(rad*0.98)
                    let x2=s*(rad*0.95)
                    let y2=c*(rad*0.95)
                    if (num%5==0){
                        ctx.lineWidth=4
                    } else {
                        ctx.lineWidth=1
                    }
                    ctx.beginPath()
                    ctx.moveTo(x1,y1)
                    ctx.lineTo(x2,y2)
                    ctx.stroke()
                }
                let now = new Date()
                let hour = now.getHours()
                let minute = now.getMinutes()
                let second = now.getSeconds()
                //hour
                hour=hour%12
                ctx.beginPath()
                ctx.arc(0,0,rad*0.05,0,Math.PI*2, false) // outer (filled)
                ctx.arc(0,0,rad*0.02,0,Math.PI*2, true) // inner (unfills it)
                ctx.fill()
                hour=(hour*Math.PI/6)+(minute*Math.PI/(6*60))+(second*Math.PI/(360*60))
                this.drawHand(ctx, hour, rad*0.05, rad*0.3, rad*0.04)
                this.drawHand(ctx, hour, rad*0.3, rad*0.7, rad*0.07)
                //minute
                minute=(minute*Math.PI/30)+(second*Math.PI/(30*60))
                this.drawHand(ctx, minute, rad*0.05, rad*0.3, rad*0.04)
                this.drawHand(ctx, minute, rad*0.3, rad*0.9, rad*0.07)
                // second
                ctx.strokeStyle = "red"
                second=(second*Math.PI/30)
                this.drawHand(ctx, second, -rad*0.1, rad*0.9, rad*0.02)                
            })
        },
        drawHand(ctx, pos, lengthStart,lengthEnd, width) {
            ctx.beginPath();
            ctx.lineWidth = width;
            ctx.lineCap = "round";
            ctx.rotate(pos);
            ctx.moveTo(0, -lengthStart);
            ctx.lineTo(0, -lengthEnd);
            ctx.stroke();
            ctx.rotate(-pos);
        },
        decHeight() {
            if (this.height > 136) this.height -= 32
            this.update(true)
        },
        incHeight() {
            this.height += 32
            this.update(true)
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
            this.drawClock()
            this.$emit("update", this.getSetup(),save)
        },
        getSetup(){
            return JSON.stringify({
                name: this.$options.name,
                cols: this.cols,
                background: this.background,
                height: this.height,
                options: {
                    title: this.title,
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
	font-size:5em;
    margin-top: -31px;	
	padding:6px;
}
.cardicon{
    display:inline-block;
    position: absolute;
    right: 34px;
}
canvas{
    width:100%;
    height:100%
}
</style>
