<template>
  <v-app>
    <v-container>
      <v-btn @click="calc()">Calc</v-btn>
      <v-row @mousemove='mousemove' @mousedown.stop.prevent='mousedown' @mouseup='mouseup'>
        <v-col
            v-for="(item,index) in cards" :key="index"
            :cols="item.cols"
            :md="item.md"
            :lg="item.lg"
            class=""
            >
            <v-card
              :ref="index"
              height=80
              class="primary pa-2"
              dark
              >
                {{ item.text }}
            </v-card>    
        </v-col>
      </v-row>
    </v-container>
  </v-app>
</template>

<script>

export default {
  name: 'App',
  components: {
  },  
  data: () => ({
    cards: [
      { id:1, cols:12, md:6, lg:4, text: "Item 1 cols:12, md:6, lg:4"},
      { id:2, cols:12, md:6, lg:4, text: "Item 2 cols:12, md:6, lg:4"},
      { id:3, cols:12, md:6, lg:4, text: "Item 3 cols:12, md:6, lg:4"},
      { id:4, cols:12, md:12, lg:6, text: "Item 4 cols:12, md:12, lg:6"},
      { id:5, cols:12, md:12, lg:6, text: "Item 5 cols:12, md:12, lg:6"},
      { id:6, cols:12, md:12, lg:12, text: "Item 6 cols:12, md:12, lg:12"},
    ],
    collisionBoxes: [],
    startOrder: [],
    startIndex: null,
    currentIndex: null,
    dragging: false,
  }),
  methods: {
    getBox(e){
        let found=this.collisionBoxes.findIndex(box => e.clientX >= box.left && e.clientX <= box.right && e.clientY >= box.top && e.clientY <= box.bottom)
        // Why to a string? Strange behaviour when it isn't, need to look into it.
        return found>=0?found.toString():null
    },
    mousemove(e){
      if (this.dragging){
        // Get the box your mouse is currently over
        let found=this.getBox(e)
        if (found && found!=this.currentIndex){
          // Position has changed
          let newcards=[]
          for(let i in this.startOrder){
            if (i==this.startIndex){
              console.log('do nothing')
            } else {
              newcards.push(this.startOrder[i])
            }
          }
          newcards.splice(found,0,this.startOrder[this.startIndex])
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
      this.currentIndex = this.startIndex
    },
    mouseup(){
      this.dragging = false
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
</style>