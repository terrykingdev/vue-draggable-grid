<template>
    <v-row @mousemove='mousemove' @mousedown='mousedown' @mouseup='mouseup'>
        <v-col
            v-for="(item,index) in widgetList" :key="index"
            v-bind="item.props.initialize.cols"
            >
            <v-card
                ref="widgets"
                tile
                flat
                :style="item.blank?`width:${draggedItemInfo.width}px;height:${draggedItemInfo.height}px;`:''"
                :class="draggableitem(item)"
                >
                <component v-if="!item.blank" :is="getComponent(item.name)" v-bind="item.props" v-on:remove="removeCard(item)" v-on:duplicate="duplicateCard(item,...arguments)" v-on:update="updateCard(item,...arguments)"/>
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
        <!-- <div
            class="overlay"
            v-for="(box,index) in collisionBoxes" :key="index"
            :style="`left:${box.left}px;top:${box.top}px;width:${box.width}px;height:${box.height}px;`"
        ></div> -->
    </v-row>
</template>

<script>
const Card1 = () => import('../components/Card1')
const Card2 = () => import('../components/Card2')
const Card3 = () => import('../components/Card3')
const Card4 = () => import('../components/Card4')
const CardSpacer = () => import('../components/CardSpacer')

export default {
    name:'WidgetDashboard',
    props: ['widgets'],
    data: () => ({
        collisionBoxes: [],
        draggedItem: null,
        draggedItemInfo: null,
        startOrder: [],
        startIndex: null,
        currentIndex: null,
        newIndex: -1,
        widgetsCopy: [],
        useWidgetsCopy: false,
    }),
    mounted(){
    },
    computed:{
        widgetList(){
            return this.useWidgetsCopy?this.widgetsCopy:this.widgets
        } 
    },
    methods: {
        updateCard(item,data,save){
            // When a card is configured the changes are stored back in the props
            item.props.initialize = JSON.parse(data)
            this.$emit('update',this.widgets,save) 
        },
        removeCard(item){
            this.startOrder = Object.assign([],this.widgets) // array of original starting order
            let newwidgets = []
            newwidgets = this.startOrder.filter(c => c!=item)
            this.$emit('redraw',newwidgets)
        },
        duplicateCard(item,data){
            let index=this.widgets.indexOf(item)
            console.log("duplicate",item,data)
            let copy={
                name: item.name,
                props: {
                    initialize: JSON.parse(data)
                }
            }
            this.startOrder = Object.assign([],this.widgets) // array of original starting order
            let newwidgets = []
            newwidgets = this.startOrder
            newwidgets.splice(index+1,0,copy)
            this.$emit('redraw',newwidgets)

        },        
        getComponent(name){
            let r
            switch (name){
                case 'Card1' : r=Card1; break;
                case 'Card2' : r=Card2; break;
                case 'Card3' : r=Card3; break;
                case 'Card4' : r=Card4; break;
                case 'CardSpacer' : r=CardSpacer; break;
            }
            return r
        },
        draggableitem(item){
            let r='draggable'
            if (item.blank){
                r+=' grey lighten-2'
            }
            return r
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
                    // Update cards with the new order and vue updates visually
                    this.widgetsCopy = []
                    this.$nextTick(()=>{
                        this.widgetsCopy = this.startOrder.filter((x,i) => i != this.startIndex)
                        this.widgetsCopy.splice(found,0,{
                            blank:true,
                            props: this.startOrder[this.startIndex].props
                        }) // insert blank
                        this.currentIndex = found
                    })
                }
        }
        },
        mousedown(e){
            // Cycle through the column grid using $refs and get the elements bounding box coordinates
            // The $refs.cards order might not be in the correct order but doesn't matter
            this.collisionBoxes = this.$refs.widgets.map(x => {
                let bb=x.$el.getBoundingClientRect()
                return {left:bb.left, top:bb.top+window.scrollY, right:bb.right, bottom:bb.bottom+window.scrollY, width:bb.width, height:bb.height}
            }) // array of box collisions
            this.startOrder = Object.assign([],this.widgets) // array of original starting order
            this.widgetsCopy = Object.assign([],this.widgets)
            this.useWidgetsCopy = true
            this.startIndex = this.getBox(e)
            if (this.startIndex){
                // Clicked in a valid box
                this.currentIndex = this.startIndex
                this.newIndex = this.startIndex
                this.draggedItem = this.widgetsCopy.splice(this.startIndex,1)[0] // remove the item clicked on and assign it to the dragged item
                // this.draggedItem.props.data={}
                console.log("draggedItem",this.draggedItem)
                let box = this.collisionBoxes[this.startIndex]
                this.draggedItemInfo = {
                    width: box.width,
                    height: box.height,
                    offsetX: e.clientX-box.left,
                    offsetY: e.clientY-box.top,
                    posX: e.clientX-(e.clientX-box.left),
                    posY: e.clientY-(e.clientY-box.top)
                }
                this.widgetsCopy.splice(this.startIndex,0,{
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
                this.widgetsCopy.splice(this.newIndex,1,this.draggedItem) // delete blank and insert original item
                this.draggedItem = null
                this.collisionBoxes = []
                this.$nextTick().then(()=>{
                    this.useWidgetsCopy=false
                    console.log("mouseup",this.widgetsCopy)
                    this.$emit('update',this.widgetsCopy,true)
                })
            }
        },        
    }
}
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