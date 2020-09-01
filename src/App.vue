<template>
  <v-app>
    <v-container>

      <v-row @mousemove='mousemove' @mousedown.stop.prevent='mousedown' @mouseup='mouseup'>
      <v-card
        v-if="draggedItem"
        class="draggeditem"
        :style="`position:absolute;left:${draggedItemInfo.posX}px;top:${draggedItemInfo.posY}px;width:${draggedItemInfo.width}px;height:${draggedItemInfo.height}px;`"
        elevation="15"
        >
          <component :is="draggedItem.name" v-bind="draggedItem.props"/>
      </v-card>
        <v-col
            v-for="(item,index) in cards" :key="index"
            v-bind="item.cols"
            >
            <v-card
              ref="cards"
              :class="item.blank?'grey lighten-2':''"
              :style="item.blank?`width:${draggedItemInfo.width}px;height:${draggedItemInfo.height}px;`:''"
              class="draggable"
              >
                <component v-if="!item.blank" :is="item.name" v-bind="item.props"/>
            </v-card>    
        </v-col>
      </v-row>

    </v-container>
    <!--div
      class="overlay"
      v-for="(box,index) in collisionBoxes" :key="index"
      :style="`left:${box.left}px;top:${box.top}px;width:${box.width}px;height:${box.height}px;`"
      ></div-->
  </v-app>
</template>

<script>
import Card1 from './components/Card1'
import Card2 from './components/Card2'
import Card3 from './components/Card3'

export default {
  name: 'App',
  components: {
  },  
  data: () => ({
    cards: [
      {
        id:0,
        cols:{
          cols:12,
          md:6,
          lg:4
        },
        name: Card1,
        props: {
          title:'Title 1',
          text:'Item 1 cols:12, md:6, lg:4'
        }
      },
      {
        id:1,
        cols:{
          cols:12,
          md:6,
          lg:4
        },
        name: Card1,
        props: {
          title:'Title 2',
          text:'Item 1 cols:12, md:6, lg:4'
        }
      },
      {
        id:2,
        cols:{
          cols:12,
          md:6,
          lg:4
        },
        name: Card1,
        props: {
          title:'Title 3',
          text:'Item 1 cols:12, md:6, lg:4'
        }
      },
      { 
        id:3,
        cols:{
          cols:12,
          md:6,
        },
        name: Card2,
        props: {
          content:'Content 1'
        }
      },
      { 
        id:4,
        cols:{
          cols:12,
          md:6,
        },
        name: Card2,
        props: {
          content:'Content 2'
        }
      },
      { 
        id:'large',
        cols:{
          cols:12,
        },
        name: Card3,
        props: {
          list:[
            {title: 'Item 1'},
            {title: 'Item 2'},
            {title: 'Item 3'},
          ]
        }
      },      
    ],
    collisionBoxes: [],
    startOrder: [],
    startIndex: null,
    currentIndex: null,
    newIndex: -1,
    draggedItem: null,
    draggedItemInfo: null,
  }),
  mounted(){
    // Get the custom order from localStorage
    let order=localStorage.getItem('cardOrder')
    if (order){
      this.cards = order.split(',').map(x => this.cards.filter(c => c.id == x)[0])
    }
  },
  methods: {
    getBox(e){
        let found = this.collisionBoxes.findIndex(box => e.clientX >= box.left && e.clientX <= box.right && e.clientY >= box.top && e.clientY <= box.bottom)
        // Why to a string? Strange behaviour when it isn't, need to look into it.
        return found>=0?found.toString():null
    },
    mousemove(e){
      if (this.draggedItem){
        this.draggedItemInfo.posX = e.clientX-this.draggedItemInfo.offsetX
        this.draggedItemInfo.posY = e.clientY-this.draggedItemInfo.offsetY
        // Get the box your mouse is currently over
        let found = this.getBox(e)
        if (found && found != this.currentIndex){
          this.newIndex = found
          // Position has changed
          let newcards = this.startOrder.filter((x,i) => i != this.startIndex)
          newcards.splice(found,0,{ blank:true }) // insert blank
          // Update cards with the new order and vue updates visually
          this.cards = newcards
          this.currentIndex = found
        }
      }
    },
    mousedown(e){
      // Cycle through the column grid using $refs and get the elements bounding box coordinates
      this.collisionBoxes = this.$refs.cards.map(x => x.$el.getBoundingClientRect()) // array of box collisions
      this.startOrder = Object.assign([],this.cards) // array of original starting order
      this.startIndex = this.getBox(e)
      if (this.startIndex){
        // Clicked in a valid box
        this.currentIndex = this.startIndex
        this.newIndex = this.startIndex
        this.draggedItem = this.cards.splice(this.startIndex,1)[0] // remove the item clicked on and assign it to the dragged item
        let box = this.collisionBoxes[this.startIndex]
        this.draggedItemInfo = {
          width: box.width,
          height: box.height,
          offsetX: e.clientX-box.left,
          offsetY: e.clientY-box.top,
          posX: e.clientX-(e.clientX-box.left),
          posY: e.clientY-(e.clientY-box.top)
        }
        this.cards.splice(this.startIndex,0,{ blank:true }) // insert blank
      }
    },
    mouseup(){
      if (this.draggedItem){
        if (!this.newIndex){
          this.newIndex = this.startIndex
        }
        this.cards.splice(this.newIndex,1,this.draggedItem) // delete blank and insert original item
        localStorage.setItem('cardOrder', this.cards.map(x => x.id)) // save the new order in local storage
        this.draggedItem = null
        this.collisionBoxes = []
      }
    },
  }
};
</script>

<style scoped>
.overlay{
  position:absolute;
  left:0;
  top:0;
  width:0;
  height:0;
  border: 2px solid red;  
  pointer-events: none;
}
.draggable{
  cursor: -webkit-grab;
  cursor: grab;
}
@keyframes chosen{
  0% {transform: rotateZ(0deg) scale(1);}
  100% {transform: rotateZ(1deg) scale(1.05);}
}
.draggeditem{
  z-index:1;
  cursor: -webkit-grabbing;
  cursor: grabbing;
  animation-name: chosen;
  animation-duration: 0.1s;
  animation-fill-mode: forwards;
  animation-timing-function: ease-out;
}
</style>