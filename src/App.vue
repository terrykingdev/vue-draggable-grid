<template>
  <v-app>
    <v-container>
      <v-btn @click="dump" class="mr-2">Dump</v-btn>
      <v-btn @click="refresh" class="mr-2">Refresh Components</v-btn>
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
      <widget-dashboard @redraw="redrawWidgets" @update="updateWidgets" :widgets="cards"></widget-dashboard>
    </v-container>
  </v-app>
</template>

<script>
import WidgetDashboard from './components/WidgetDashboard.vue'

export default {
  name: 'App',
  components: {
    WidgetDashboard
  },  
  data: () => ({
    addlist:[
      { title: 'Card1' , name: 'Card1', cols:{cols:12,md:6,lg:4,xl:3}},
      { title: 'Card2' , name: 'Card2', cols:{cols:12,md:6}},
      { title: 'Card3' , name: 'Card3', cols:{cols:12}},
      { title: 'Card4' , name: 'Card4', cols:{cols:12,lg:4}},
      { title: 'Spacer' , name: 'CardSpacer', cols:{cols:12}},
    ],
    cards: [
      // {
      //   name: 'Card1',
      //   props: {
      //     initialize: {
      //       cols:{
      //         cols:12,
      //         md:6,
      //         lg:4
      //       },
      //       options:{
      //         title:'Card Title 1',
      //         text:'Initial text',
      //         background: 'red'
      //       }
      //     }
      //   }
      // },
    ],
    pickDelay: null,
    dashboardName: 'testdashboard'
  }),
  mounted(){
    // Get the custom order from localStorage
    let config=localStorage.getItem(this.dashboardName)
    if (config){
      let configData = JSON.parse(atob(config))
      console.log("loading dashboard",configData)
      this.cards = []
      for(let data of configData){
        let init = JSON.parse(data)
        console.log(init)
        let newCard = {
          name: init.name,
          props: {
            initialize:{
              // refresh: true, // tell component to refresh
              cols: init.cols,
              options: init.options
            }
          }
        }
        this.cards.push(newCard)
      }
    }
  },
  computed: {
  },
  methods: {
    redrawWidgets(widgets){
      this.cards=[]
      this.$nextTick(()=>{
        this.cards=widgets
        this.saveToStorage()
      })      
    },
    updateWidgets(newwidgets,save){
      console.log("updateWidgets",newwidgets)
      this.cards=newwidgets
        if (save){
          this.saveToStorage()
        }
    },
    refresh(){
      console.log("refresh")
      let newCards=[]
      for(let c of this.cards){
        c.props.initialize.refresh=true
        newCards.push(c)
      }
      this.cards=[]
      this.$nextTick(()=>{
        this.cards=newCards
      })      
    },
    addCard(item){
      let newCard={
        name: item.name,
        props: {
          initialize: {
            save: true,  // when adding, use the save flag so the widgets are saved after it's created
            refresh: true, // ensure the first time that all of the components private data is calculated
            cols: item.cols,
            options: {}
          }
        }
      }
      this.cards.push(newCard)
    },
    dump(){
      console.log("dump")
      for(let c of this.cards){
        console.log(JSON.stringify(c.props.initialize))
      }
    },
    saveToStorage(){
      let config = []
      for(let c of this.cards){
        config.push(JSON.stringify(c.props.initialize))
      }
      console.log("saving dashboard",config)
      localStorage.setItem(this.dashboardName, btoa(JSON.stringify(config)))
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