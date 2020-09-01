<template>
  <v-app>
    <v-container>

      <v-row @mousemove='mousemove' @mousedown.stop.prevent='mousedown' @mouseup='mouseup'>
      <v-card
        v-if="draggedItem"
        class="draggeditem"
        :class="cardColour(draggedItem)"
        :style="getDraggedStyle()"
        elevation="15"
        >
          <component :is="draggedItem.name" v-bind="draggedItem.props"/>
      </v-card>
        <v-col
            v-for="(item,index) in cards" :key="index"
            v-bind="item.cols"
            >
            <v-card
              :ref="index"
              :class="cardColour(item)"
              :style="cardStyle(item)"
              class="draggable"
              >
                <!-- blank item doesn't have a name -->
                <component v-if="item.name" :is="item.name" v-bind="item.props"/>
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
  methods: {
    cardStyle(item){
      if (item.blank){
        // Make the blanked item the same width and height as the dragged
        return `width:${this.draggedItemInfo.width}px;height:${this.draggedItemInfo.height}px;`
      } else {
        return ''
      }
    },
    getDraggedStyle(){
      return `position:absolute;left:${this.draggedItemInfo.posX}px;top:${this.draggedItemInfo.posY}px;width:${this.draggedItemInfo.width}px;height:${this.draggedItemInfo.height}px;`
    },
    cardColour(item){
      if (item.blank){
        return "grey lighten-2"
      } else {
        return ""
      }
    },
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
        if (found && found!=this.currentIndex){
          this.newIndex = found
          // Position has changed
          let newcards=[]
          for(let i in this.startOrder){
            if (i != this.startIndex){
              newcards.push(this.startOrder[i])
            }
          }
          newcards.splice(found,0,{ cols:this.draggedItem.cols, md:this.draggedItem.md, lg:this.draggedItem.lg, blank:true}) // insert blank
          // Update cards with the new order and vue updates visually
          this.cards = newcards
          this.currentIndex=found
        }
      }
    },
    mousedown(e){
      this.calc()
      this.startIndex = this.getBox(e)
      if (this.startIndex){
        this.currentIndex = this.startIndex
        this.newIndex = this.startIndex
        this.cards[this.startIndex].absolute = true // flag the selected card for absolute positioning
        this.draggedItem = this.cards.splice(this.startIndex,1)[0] // remove item clicked on
        let box = this.collisionBoxes[this.startIndex]
        this.draggedItemInfo = {
          width: box.width,
          height: box.height,
          offsetX: e.clientX-box.left,
          offsetY: e.clientY-box.top,
          posX: e.clientX-(e.clientX-box.left),
          posY: e.clientY-(e.clientY-box.top)
        }
        this.cards.splice(this.startIndex,0,{ cols:this.draggedItem.cols, md:this.draggedItem.md, lg:this.draggedItem.lg, blank:true}) // insert blank
      }
    },
    mouseup(){
      if (this.draggedItem){
        if (!this.newIndex){
          this.newIndex = this.startIndex
        }
        this.draggedItem.absolute = null
        this.cards.splice(this.newIndex,1) // delete blank
        this.cards.splice(this.newIndex,0,this.draggedItem) // insert original item
        this.draggedItem = null
        this.collisionBoxes = []
      }
    },
    calc(){
      // Cycle through the column grid using $refs and get the elements bounding box coordinates
      let bbs=[]
      let so=[]
      for(let i in this.$refs){
        let ref=this.$refs[i][0]
        bbs.push(ref.$el.getBoundingClientRect())
      }
      for(let card of this.cards){
        so.push(card)
      }
      this.startOrder = so // array of original starting order
      this.collisionBoxes = bbs // array of box collisions
    }
  }
};
</script>

<style scoped>
.anim{
  transition: all 0.5s ease-in;
}
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
.draggeditem{
  z-index:1;
  cursor: -webkit-grabbing;
  cursor: grabbing;
	transform: rotateZ(1.5deg) scale(1.05);
}
</style>