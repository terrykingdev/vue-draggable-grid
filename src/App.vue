<template>
  <v-app>
    <v-container>

      <v-row @mousemove='mousemove' @mousedown.stop.prevent='mousedown' @mouseup='mouseup'>
      <v-card
        v-if="draggedItem"
        :height="draggedItemHeight"
        :width="draggedItemWidth"
        class="draggeditem"
        :class="cardColour(draggedItem)"
        :style="getDraggedStyle()"
        dark
        >
          {{ draggedItem.text }}
      </v-card>
        <v-col
            v-for="(item,index) in cards" :key="index"
            :cols="item.cols"
            :md="item.md"
            :lg="item.lg"
            >
            <v-card
              :ref="index"
              height=80
              :class="cardColour(item)"
              :style="getStyle(item)"
              dark
              >
                {{ item.text }}
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

export default {
  name: 'App',
  components: {
  },  
  data: () => ({
    cards: [
      { cols:12, md:6, lg:4, text: "Item 1 cols:12, md:6, lg:4"},
      { cols:12, md:6, lg:4, text: "Item 2 cols:12, md:6, lg:4"},
      { cols:12, md:6, lg:4, text: "Item 3 cols:12, md:6, lg:4"},
      { cols:12, md:12, lg:6, text: "Item 4 cols:12, md:12, lg:6"},
      { cols:12, md:12, lg:6, text: "Item 5 cols:12, md:12, lg:6"},
      { cols:12, md:12, lg:12, text: "Item 6 cols:12, md:12, lg:12"},
      // { id:1, cols:3, text: "1 cols3"},
      // { id:2, cols:3, text: "2 cols3"},
      // { id:3, cols:3, text: "3 cols3"},
      // { id:4, cols:3, text: "4 cols3"},
      // { id:5, cols:3, text: "5 cols3"},
      // { id:6, cols:3, text: "6 cols3"},
      // { id:7, cols:3, text: "7 cols3"},
      // { id:8, cols:3, text: "8 cols3"},
      // { id:9, cols:3, text: "9 cols3"},
      // { id:10, cols:3, text: "10 cols3"},
      // { id:11, cols:3, text: "11 cols3"},
      // { id:12, cols:3, text: "12 cols3"},
      // { id:13, cols:3, text: "13 cols3"},
      // { id:14, cols:3, text: "14 cols3"},
      // { id:15, cols:3, text: "15 cols3"},
      // { id:16, cols:3, text: "16 cols3"},
    ],
    collisionBoxes: [],
    startOrder: [],
    startIndex: null,
    currentIndex: null,
    dragging: false,
    newIndex: -1,
    draggedItem: null,
    draggedItemX: null,
    draggedItemY: null,
  }),
  computed: {
  },
  methods: {
    getDraggedStyle(){
      return `position:absolute;top:${this.draggedItemY}px;left:${this.draggedItemX}px;`
    },
    getStyle(item){
      if (item.absolute){
        return "position:absolute;top:0;left:0;"
      }
    },
    cardColour(item){
      if (item.blank){
        return "grey lighten-1"
      } else {
        return "primary pa-2"
      }
    },
    getBox(e){
        let found=this.collisionBoxes.findIndex(box => e.clientX >= box.left && e.clientX <= box.right && e.clientY >= box.top && e.clientY <= box.bottom)
        // Why to a string? Strange behaviour when it isn't, need to look into it.
        return found>=0?found.toString():null
    },
    mousemove(e){
      if (this.dragging){
        this.draggedItemX = e.clientX+this.dragOffsetX
        this.draggedItemY = e.clientY+this.dragOffsetY
        // Get the box your mouse is currently over
        let found=this.getBox(e)
        this.newIndex=found
        if (found && found!=this.currentIndex){
          // Position has changed
          let newcards=[]
          for(let i in this.startOrder){
            if (i!=this.startIndex){
              newcards.push(this.startOrder[i])
            }
          }
          console.log('inserting into pos '+found)
          newcards.splice(found,0,{ cols:this.draggedItem.cols, md:this.draggedItem.md, lg:this.draggedItem.lg, blank:true}) // insert blank

          // Update cards with the new order and vue updates visually
          this.cards = newcards

          this.currentIndex=found
        }
      }
    },
    mousedown(e){
      this.dragging = true
      this.calc()
      this.startIndex = this.getBox(e)
      this.cards[this.startIndex].absolute=true
      this.draggedItem = this.cards.splice(this.startIndex,1)[0] // remove item clicked on
      this.draggedItemWidth = this.collisionBoxes[this.startIndex].width
      this.draggedItemHeight = this.collisionBoxes[this.startIndex].height
      this.dragOffsetX = this.collisionBoxes[this.startIndex].left-e.clientX
      this.dragOffsetY = this.collisionBoxes[this.startIndex].top-e.clientY
      this.draggedItemX = e.clientX+this.dragOffsetX
      this.draggedItemY = e.clientY+this.dragOffsetY
      console.log('dragOffsets',this.dragOffsetX,this.dragOffsetY)
      this.cards.splice(this.startIndex,0,{ cols:this.draggedItem.cols, md:this.draggedItem.md, lg:this.draggedItem.lg, blank:true}) // insert blank
      this.currentIndex = this.startIndex
    },
    mouseup(){
      console.log('mouseup',this.newIndex)
      this.draggedItem.absolute = null
      this.cards.splice(this.newIndex,1) // delete blank
      this.cards.splice(this.newIndex,0,this.draggedItem) // insert original item
      this.draggedItem = null
      this.dragging = false
      this.collisionBoxes = []
    },
    calc(){
      // Cycle through the column grid using $refs and get the elements bounding box coordinates
      let bbs=[]
      let so=[]
      for(let i in this.$refs){
        let ref=this.$refs[i][0]
        let bb=ref.$el.getBoundingClientRect()
        bbs.push(bb)
      }
      for(let i in this.cards){
        so.push(this.cards[i])
      }
      this.startOrder = so // array of original stating order
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
.draggeditem{
  pointer-events: none;
  z-index:1;
}
</style>