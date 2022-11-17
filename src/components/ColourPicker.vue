<template>
    <v-dialog
        v-model="show"
        max-width="400"
    >
        <v-card>
            <v-card-title>Colour picker</v-card-title>
            <div class="grid-container">
                <div @click.stop="picked(c)" class="grid-item" v-for="(c,index) in colourValues" :key="index" :style="getBackgroundColour(c.value)">
                    <svg v-if="index==0">
                        <line x1='22px' y1='0px' x2='0' y2='22px' />
                    </svg>
                </div>
            </div>
        </v-card>
    </v-dialog>
</template>

<script>
import colors from 'vuetify/lib/util/colors'

export default {
    props: ['show'],
    data: () => ({
        colourValues:[],
    }),    
    mounted(){
      for(let [colourName,v] of Object.entries(colors)){
        let colourSet=[]
        for(let [base,rgb] of Object.entries(v)){
            if (rgb.match(/#....../)){
                colourSet.push({text: `${colourName}-${base}`, value: rgb})
            }
        }
        colourSet.sort((a,b)=>this.rgbToLum(a.value)<this.rgbToLum(b.value)?1:-1)
        this.colourValues=[...this.colourValues,...colourSet]
      }       
      this.colourValues[0]={text: `transparent`, value: 'rgba(0,0,0,0)'}
    },
    methods:{
        picked(colour){
            this.$emit('change',colour)
        },
        getBackgroundColour(colour){
            if (colour=='rgba(0,0,0,0)'){
                return 'background-color:white;'
            }
            return `background-color:${colour};`
        },
        rgbToLum(c){
            if (c && c.match(/#....../)){
                // Calculate the luminance of the background to decide whether cog should be black or white
                let r=parseInt(c.substring(1,3),16)
                let g=parseInt(c.substring(3,5),16)
                let b=parseInt(c.substring(5,7),16)
                let lum=(0.299*r)+(0.587*g)+(0.114*b)
                return lum
            }
            return 0
        },        
    }
}
</script>

<style scoped>
.grid-container {
  display: grid;
  grid-template-columns: auto auto auto auto auto auto auto auto auto auto auto auto auto auto auto auto ;
  /* background-color: #2196F3; */
  padding: 10px;
}
.grid-item {
  background-color: rgba(255, 255, 255, 0.8);
  border: 1px solid rgba(0, 0, 0, 0.8);
  width:24px;
  height:24px;
}
</style>
<style lang="scss" scoped>
 svg {
    width: 24px;
    height: 24px;
    position: absolute;
    line {
        stroke: red;
        stroke-width: 2;
    }
}
</style>