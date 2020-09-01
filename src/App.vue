<template>
  <v-app>
    <v-container>

      <v-row @mousemove='mousemove' @mousedown='mousedown' @mouseup='mouseup' @touchstart='touchstart' @touchend='touchend' @touchmove='touchmove'>
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
        <v-card
          v-if="draggedItem"
          class="draggeditem"
          :style="`position:absolute;left:${draggedItemInfo.posX}px;top:${draggedItemInfo.posY}px;width:${draggedItemInfo.width}px;height:${draggedItemInfo.height}px;`"
          elevation="15"
          >
            <component :is="draggedItem.name" v-bind="draggedItem.props"/>
        </v-card>
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
    pickDelay: null,
    touchDrag: false,
    touchStartX: null,
    touchStartY: null,
  }),
  mounted(){
    // Get the custom order from localStorage
    let order=localStorage.getItem('cardOrder')
    if (order){
      this.cards = order.split(',').map(x => this.cards.filter(c => c.id == x)[0])
    }
  },
  methods: {
    touchstart(e){
      // Setup a slight delay to avoid accidental moving of items when trying to scroll. So touch devices need to hold the item for a fraction
      // of a second before it gets 'picked up'.
      this.pickDelay = setTimeout(()=>{
        this.touchDrag=true
        e.clientX = e.changedTouches[0].clientX
        e.clientY = e.changedTouches[0].clientY
        this.touchStartX = e.clientX
        this.touchStartY = e.clientY
        this.mousedown(e)
      },200)
    },
    touchend(e){
      clearTimeout(this.pickDelay)
      e.clientX = e.changedTouches[0].clientX
      e.clientY = e.changedTouches[0].clientY
      this.mouseup(e)
      this.touchDrag=false
    },
    touchmove(e){
      if (this.touchDrag){
        // Definitely dragging and not scrolling
        e.clientX = e.changedTouches[0].clientX
        e.clientY = e.changedTouches[0].clientY
        this.mousemove(e)
        // Stop scrolling by preventing default propagation
        e.preventDefault()
      } else {
        let dx = e.changedTouches[0].clientX-this.touchStartX
        let dy = e.changedTouches[0].clientY-this.touchStartY
        let dist = Math.sqrt(dx*dx+dy*dy)
        if (dist>20){
          // If we've moved more than the dist before the timer runs out we must be scrolling so cancel the timer and allow the event to propagate
          clearTimeout(this.pickDelay)
        }
      }
    },
    getBox(e){
      let x=e.clientX+window.scrollX
      let y=e.clientY+window.scrollY
        let found = this.collisionBoxes.findIndex(box => x >= box.left && x <= box.right && y >= box.top && y <= box.bottom)
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
      this.collisionBoxes = this.$refs.cards.map(x => {
        let bb=x.$el.getBoundingClientRect()
        return {left:bb.left, top:bb.top+window.scrollY, right:bb.right, bottom:bb.bottom+window.scrollY, width:bb.width, height:bb.height}
      }) // array of box collisions
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
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;  
  pointer-events: none;
  cursor: -webkit-grab;
  cursor: grab;
}
@keyframes chosen{
  0% {transform: rotateZ(0deg) scale(1);}
  100% {transform: rotateZ(1deg) scale(1.05);}
}
.draggeditem{
  -webkit-touch-callout: none;
  -webkit-user-select: none;
  -khtml-user-select: none;
  -moz-user-select: none;
  -ms-user-select: none;
  user-select: none;  
  pointer-events: none;
  z-index:1;
  cursor: -webkit-grabbing;
  cursor: grabbing;
  animation-name: chosen;
  animation-duration: 0.1s;
  animation-fill-mode: forwards;
  animation-timing-function: ease-out;
}
</style>