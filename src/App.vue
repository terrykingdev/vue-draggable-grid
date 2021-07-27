<template>
  <v-app>
    <v-container>
      <v-btn @click="dump">Dump</v-btn>
      <v-menu offset-y>
        <template v-slot:activator="{ on, attrs }">
          <v-btn
            color="primary"
            dark
            v-bind="attrs"
            v-on="on"
          >
            Add
          </v-btn>
        </template>
        <v-list dense
          >
          <v-list-item-group
            dark
            >
            <v-list-item
            v-for="(item, index) in addlist"
            :key="index"
            @click="addCard(item)"
            >
            <v-list-item-title>{{ item.title }}</v-list-item-title>
            </v-list-item>
          </v-list-item-group>
        </v-list>
      </v-menu>
      <v-row @mousemove='mousemove' @mousedown='mousedown' @mouseup='mouseup' @touchstart='touchstart' @touchend='touchend' @touchmove='touchmove'>
        <v-col
            v-for="(item,index) in cards" :key="index"
            v-bind="item.props.initialize.cols"
            >
            <v-card
              ref="cards"
              :style="item.blank?`width:${draggedItemInfo.width}px;height:${draggedItemInfo.height}px;`:''"
              :class="draggableitem(item)"
              >
                <component ref="card" :data-index="index" v-if="!item.blank" :is="getComponent(item.name)" v-bind="item.props" v-on:remove="removeCard(item)" v-on:duplicate="duplicateCard(item,...arguments)" v-on:update="updateCard(item,...arguments)"/>
            </v-card>    
        </v-col>
        <v-card
          v-if="draggedItem"
          class="draggeditem"
          :style="`position:absolute;left:${draggedItemInfo.posX}px;top:${draggedItemInfo.posY}px;width:${draggedItemInfo.width}px;height:${draggedItemInfo.height}px;`"
          elevation="15"
          >
            <component :is="getComponent(draggedItem.name)" v-bind="draggedItem.props"/>
        </v-card>
      </v-row>

    </v-container>
    <!-- <div
      class="overlay"
      v-for="(box,index) in collisionBoxes" :key="index"
      :style="`left:${box.left}px;top:${box.top}px;width:${box.width}px;height:${box.height}px;`"
      ></div> -->
  </v-app>
</template>

<script>
const Card1 = () => import('./components/Card1')
const Card2 = () => import('./components/Card2')
const Card3 = () => import('./components/Card3')
const Card4 = () => import('./components/Card4')

export default {
  name: 'App',
  components: {

  },  
  data: () => ({
    addlist:[
      { title: 'Card1' , name: 'Card1', cols:{cols:12,md:6,lg:4}},
      { title: 'Card2' , name: 'Card2', cols:{cols:12,md:6}},
      { title: 'Card3' , name: 'Card3', cols:{cols:12}},
      { title: 'Card4' , name: 'Card4', cols:{cols:12}},
    ],
    cards: [
      {
        id:0,
        name: 'Card1',
        props: {
          initialize: {
            cols:{
              cols:12,
              md:6,
              lg:4
            },
            options:{
              title:'Title 1',
              text:'Item 1 cols:12, md:6, lg:4',
              background: 'red'
            }
          }
        }
      },
      // {
      //   id:1,
      //   cols:{
      //     cols:12,
      //     md:6,
      //     lg:4
      //   },
      //   name: Card1,
      //   props: {
      //     initialize: {
      //       title:'Title 2',
      //       //text:'Item 1 cols:12, md:6, lg:4',
      //     }
      //   }
      // },
      // {
      //   id:2,
      //   cols:{
      //     cols:12,
      //     md:6,
      //     lg:4
      //   },
      //   name: Card1,
      //   props: {
      //     initialize: {
      //       title:'Title 3',
      //       text:'Item 1 cols:12, md:6, lg:4'
      //     }
      //   }
      // },
      // {
      //   id:3,
      //   cols:{
      //     cols:12,
      //     md:6,
      //     lg:4
      //   },
      //   name: Card1,
      //   props: {
      //     initialize: {
      //       title:'Title 1',
      //       text:'Item 1 cols:12, md:6, lg:4',
      //       background: 'red'
      //     }
      //   }
      // },
      // {
      //   id:4,
      //   cols:{
      //     cols:12,
      //     md:6,
      //     lg:4
      //   },
      //   name: Card1,
      //   props: {
      //     initialize: {
      //       title:'Title 2',
      //       //text:'Item 1 cols:12, md:6, lg:4',
      //     }
      //   }
      // },
      // {
      //   id:5,
      //   cols:{
      //     cols:12,
      //     md:6,
      //     lg:4
      //   },
      //   name: Card1,
      //   props: {
      //     initialize: {
      //       title:'Title 3',
      //       text:'Item 1 cols:12, md:6, lg:4'
      //     }
      //   }
      // },      
      // { 
      //   id:3,
      //   cols:{
      //     cols:12,
      //     md:6,
      //   },
      //   name: Card2,
      //   props: {
      //     content:'Content 1'
      //   }
      // },
      // { 
      //   id:4,
      //   cols:{
      //     cols:12,
      //     md:6,
      //   },
      //   name: Card2,
      //   props: {
      //     content:'Content 2'
      //   }
      // },
      // { 
      //   id:5,
      //   cols:{
      //     cols:12,
      //   },
      //   name: Card3,
      //   props: {
      //     list:[
      //       {title: 'Item 1'},
      //       {title: 'Item 2'},
      //       {title: 'Item 3'},
      //     ]
      //   }
      // },     
      // { 
      //   id:6,
      //   cols:{
      //     cols:12,
      //   },
      //   name: Card4,
      //   props: {
      //     list:[
      //       {title: 'Item 1'},
      //       {title: 'Item 2'},
      //       {title: 'Item 3'},
      //       {title: 'Item 4'},
      //       {title: 'Item 5'},
      //       {title: 'Item 6'},
      //       {title: 'Item 7'},
      //       {title: 'Item 8'},
      //       {title: 'Item 9'},
      //     ]
      //   }
      // },        
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
    console.log(this.is_touch_device())
    // Get the custom order from localStorage
    let config=localStorage.getItem('dashboard')
    if (config){
      let configData = JSON.parse(atob(config))
      this.cards = []
      for(let data of configData){
        let init = JSON.parse(data)
        console.log(init)
        let newCard = {
          name: init.name,
          props: {
            initialize:{
              cols: init.cols,
              options: init.options
            }
          }
        }
        this.cards.push(newCard)
      }
    }
    // if (order){
    //   this.cards = order.split(',').map(x => this.cards.filter(c => c.id == x)[0])
    // }
  },
  computed: {

  },
  methods: {
    getComponent(name){
      let r
      switch (name){
        case 'Card1' : r=Card1; break;
        case 'Card2' : r=Card2; break;
        case 'Card3' : r=Card3; break;
        case 'Card4' : r=Card4; break;
      }
      return r
    },
    addCard(item){
      let newCard={
        id : this.cards.length,
        name: item.name,
        props: {
          initialize: {
            cols: item.cols,
            options: {}
          }
        }
      }
      console.log(newCard)
      this.cards.push(newCard)
      this.$nextTick().then(()=>{
        this.saveToStorage()
      })
    },
    dump(){
      console.log(this.$refs)
      for(let c of this.cards){
        console.log(JSON.stringify(c.props.initialize))
      }
    },
    saveToStorage(){
      let config = []
      for(let c of this.cards){
        config.push(JSON.stringify(c.props.initialize))
      }
      console.log("saveToStorage",config)
      localStorage.setItem('dashboard', btoa(JSON.stringify(config)))
    },
    updateCard(item,data){
      console.log(item,data)
      // When a card is configured the changes are stored back in the props
      item.props.initialize = JSON.parse(data)
      this.saveToStorage()
    },
    removeCard(item){
      console.log("remove",item)
      this.startOrder = Object.assign([],this.cards) // array of original starting order
      this.cards = []
      this.$nextTick().then(()=>{
        this.cards = this.startOrder.filter(c => c!=item)
        this.$nextTick().then(()=>{
          this.saveToStorage()
        })
      })
    },
    duplicateCard(item,data){
      console.log("duplicate",item)
      let index=this.cards.indexOf(item)
      // console.log(this.$refs.card[index].getSetup())
      let copy={
        id : this.cards.length,
        name: item.name,
        props: {
          initialize: JSON.parse(data)
        }
      }
      copy.props.initialize.options.text += ' Copy'
      console.log("copy",copy)
      this.startOrder = Object.assign([],this.cards) // array of original starting order
      this.cards = []
      this.$nextTick().then(()=>{
        this.cards = this.startOrder
        this.cards.splice(index+1,0,copy)
        this.$nextTick().then(()=>{
          this.saveToStorage()
        })
      })
    },
    draggableitem(item){
      let r='draggable'
      if (this.is_touch_device()){
        r+=' disableevents'
      }
      if (item.blank){
        r+=' grey lighten-2'
      }
      return r
    },
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
          // let newcards = this.startOrder.filter((x,i) => i != this.startIndex)
          // newcards.splice(found,0,{ blank:true }) // insert blank

          this.cards = []
          this.$nextTick().then(()=>{
            this.cards = this.startOrder.filter((x,i) => i != this.startIndex)
            this.cards.splice(found,0,{
              blank:true,
              props: this.startOrder[this.startIndex].props
            }) // insert blank
          })

          // Update cards with the new order and vue updates visually
//          this.cards = newcards
          this.currentIndex = found
        }
      }
    },
    mousedown(e){
      // Cycle through the column grid using $refs and get the elements bounding box coordinates
      // The $refs.cards order might not be in the correct order but doesn't matter
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
        this.cards.splice(this.startIndex,0,{
          blank:true,
          props: this.startOrder[this.startIndex].props
        }) // insert blank
      }
    },
    mouseup(){
      if (this.draggedItem){
        if (!this.newIndex){
          this.newIndex = this.startIndex
        }
        this.cards.splice(this.newIndex,1,this.draggedItem) // delete blank and insert original item
//        localStorage.setItem('cardOrder', this.cards.map(x => x.id)) // save the new order in local storage
        this.draggedItem = null
        this.collisionBoxes = []
        this.$nextTick().then(()=>{
          this.saveToStorage()
        })
      }
    },
    is_touch_device() {
        var prefixes = ' -webkit- -moz- -o- -ms- '.split(' ');
        var mq = function (query) {
            return window.matchMedia(query).matches;
        }
        if (('ontouchstart' in window) || window.DocumentTouch) {
            return true;
        }
        // include the 'heartz' as a way to have a non matching MQ to help terminate the join
        // https://git.io/vznFH
        var query = ['(', prefixes.join('touch-enabled),('), 'heartz', ')'].join('');
        return mq(query);
    }    
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
  cursor: -webkit-grab;
  cursor: grab;
}
.disableevents{
  pointer-events: none;
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
  z-index:1;
  cursor: -webkit-grabbing;
  cursor: grabbing;
  animation-name: chosen;
  animation-duration: 0.1s;
  animation-fill-mode: forwards;
  animation-timing-function: ease-out;
}
</style>