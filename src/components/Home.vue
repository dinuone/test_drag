<template>
  <v-app>
    <v-app-bar color="blue" class="flex-grow-0" app dark>
      <v-app-bar-title>Project</v-app-bar-title>
    </v-app-bar>
    <v-navigation-drawer app>
      <v-list-item>
        <v-list-item-content>
          <v-list-item-title class="text-h6" v-if="isCustomer"> Customer Dashboard</v-list-item-title>
          <v-list-item-title class="text-h6" v-if="isAgent"> Agent Dashboard</v-list-item-title>
          <v-list-item-subtitle> username</v-list-item-subtitle>
        </v-list-item-content>
      </v-list-item>

      <v-radio-group
          v-model="select"
          row
          @change="userActive"
      >
        <v-radio
            label="Customer"
            value="1"
        ></v-radio>
        <v-radio
            label="Agent"
            value="2"
        ></v-radio>
      </v-radio-group>
      <v-divider></v-divider>

      <v-list dense nav>
        <v-file-input v-if="isAgent"
                      label="Upload document"
                      truncate-length="15"
                      @change="handleChange"
        ></v-file-input>



        <!--        <v-btn color="red" class="mt-2 ml-3"  width="200" @click="CancelChanges" v-if="isCustomer && custChanges.length > 0" >Clear</v-btn>-->
        <div v-if="isAgent">
          <v-expansion-panels>
            <v-expansion-panel>
              <v-expansion-panel-header>
                Add Signature
              </v-expansion-panel-header>
              <v-expansion-panel-content>
                <v-text-field v-if="isAgent"
                              v-model="name"
                              :counter="10"
                              label="Signature..."
                              required
                ></v-text-field>

                <v-btn color="primary" class="mt-2" width="200" @click="addItem" elevation="2" v-if="isAgent">Add </v-btn>


              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>
        </div>

        <div v-if="isAgent">
          <v-divider class="mt-3"></v-divider>

          <v-expansion-panels>
            <v-expansion-panel>
              <v-expansion-panel-header>
                Add text field
              </v-expansion-panel-header>
              <v-expansion-panel-content>
                <v-text-field v-if="isAgent"
                              v-model="inputvalue"
                              :counter="10"
                              label="value..."
                              required
                ></v-text-field>

                <v-btn color="primary" class="mt-2" width="200" @click="addInputs" elevation="2" v-if="isAgent">Add</v-btn>


              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>

        </div>


        <div v-if="isAgent">
          <v-divider class="mt-3"></v-divider>
          <v-expansion-panels>
            <v-expansion-panel>
              <v-expansion-panel-header>
                Add checkbox
              </v-expansion-panel-header>
              <v-expansion-panel-content>
                <v-text-field v-if="isAgent"
                              v-model="checkboxName"
                              :counter="10"
                              label="value..."
                              required
                ></v-text-field>

                <v-btn color="primary" class="mt-2" width="200" @click="addCheckBox" elevation="2" v-if="isAgent">Add</v-btn>


              </v-expansion-panel-content>
            </v-expansion-panel>
          </v-expansion-panels>

        </div>



        <div class="bottom" v-if="isAgent">
          <!--        <v-btn color="info" class="mt-2 ml-3"  width="200" @click="printActive" v-if="visible">Print PDF</v-btn>-->
          <v-btn color="success" class="mt-2 ml-3"  width="200" @click="saveCordinate" v-if="items.length > 0 || inputs.length > 0 || checkboxes.length > 0 ">Save </v-btn>
          <v-btn color="red" class="mt-2 ml-3"  width="200" @click="removeItems" v-if="items.length > 0 || inputs.length > 0 || checkboxes.length > 0 ">Clear</v-btn>
        </div>
      </v-list>

    </v-navigation-drawer>

    <v-main>
      <v-dialog
          v-model="dialog"
          width="500"
      >

        <v-card>
          <v-card-title class="text-h5 grey lighten-2">
            Coordinates
          </v-card-title>

          <v-card-text v-for="result in this.results" :key="result.id">
            Item ID : {{result.id}} <br>
            X Coordinate : {{result.Xcord}}<br>
            Y Coordinate : {{result.Ycord}}<br>
          </v-card-text>

          <v-divider></v-divider>

          <v-card-actions>
            <v-spacer></v-spacer>
            <v-btn
                color="primary"
                text
                @click="dialog = false"
            >
              I accept
            </v-btn>
          </v-card-actions>
        </v-card>
      </v-dialog>

      <div  id="drop-zone" style="width:1000px; height: 100%;" class="drop-zone" v-if="isAgent" >
        <div v-for="item in items" :key="item.id"  ref="draggableContainer">

          <v-chip  @dragstart="initInteract(item.id,'sign',item.text)"  @mouseenter="enable(item.id)"
                   :id="item.id"
                   color="red"
                   text-color="white"
                   class="draggable p-3">
            {{item.text}}
          </v-chip>
        </div>

        <div v-for="inputdata in inputs" :key="inputdata.id"
             @dragstart="initInteract(inputdata.id,'input',inputdata.text)"
             :id="inputdata.id" style="width: 200px"
             class="draggable">
          <v-text-field
              :draggable="true"
              :value="inputdata.text"
          ></v-text-field>
        </div>

        <div v-for="checkbox in checkboxes" :key="checkbox.id"
             :id="checkbox.id"
             :draggable="true"
             class="draggable"
             @dragstart="initInteract(checkbox.id,'checkbox',checkbox.text)" >
          <v-checkbox
              :label="checkbox.text"
          ></v-checkbox>
        </div>

        <div>
          <canvas :id="'the-canvas'+render"  style="width: 800px; height: 1100px" v-for="render in TotPage" :key="render.id"></canvas>
          <!--          <vue-pdf-embed :source="embedSrc" id="pdfagent" style="margin-top: 100px"/>-->
        </div>


      </div>

      <Customer :embedSrc="this.embedSrc"  v-if="isCustomer" :reuslts="this.results" />
    </v-main>
  </v-app>
</template>

<script>



import interact from "interactjs";
import * as pdfjsLib from 'pdfjs-dist'
import pdfjsWorker from 'pdfjs-dist/build/pdf.worker.entry'
import Customer from '@/components/Customer'
// import VuePdfEmbed from 'vue-pdf-embed/dist/vue2-pdf-embed'

export default {
  name: 'App',

  components: {
    // VuePdfEmbed
    Customer
  },



  props:['id'],

  data(){
    return{
      items: [],
      inputs:[],
      checkboxes:[],
      embedSrc:'',
      embedSrc2:'',
      name:'',
      cardId:0,
      cordinates:[],
      custChanges:[],
      results:[],
      dialog:false,
      checkbox:null,
      select:null,
      isCustomer:false,
      isAgent:false,
      checkboxName:null,
      visible:false,
      inputvalue:null,
      TotPage:0,
      Pageindex:[],
      signCoordinate:[],
      inputCoordinate:[],
      checkbxCoordinate:[],

      positions: {
        clientX: undefined,
        clientY: undefined,
        movementX: 0,
        movementY: 0
      },


    }
  },

  mounted() {

  },


  methods: {

    enable(id){
      var elm = document.getElementById(id)
      elm.draggable=true
      console.log(elm)
    },

    initInteract: function (id,type,Text) {

      var component = this
      // var content = document.getElementById('pdfview-1')
      let selector = document.getElementById(id);
      const position = { x: 0, y: 0 }
    console.log('drag start')

      interact(selector).draggable({

        inertia: true,

        modifiers: [
          interact.modifiers.restrict({
            restriction: '.drop-zone',
            endOnly: true,
            elementRect: { top: 0, left: 0, bottom: 1, right: 1 }

          })
        ],

        listeners: {
          start (event) {
            event.draggable=true
            console.log(event.type, event.target)
          },
          end (event) {
            console.log(event.type, event.target.id)
            console.log("item id: "+ id, "X Cordinate: "+position.x,"Y Cordinate: "+position.y, "type:"+type);

            if(type == 'sign'){
              console.log('sign')
              for (var i = 0; i < component.signCoordinate.length; i++) {
                  if (component.signCoordinate[i]['id'] == event.target.id) {
                    console.log('duplicate id')
                    component.signCoordinate.splice(component.signCoordinate[i]['id'], 1);

                  }
                }
                component.signCoordinate.push({id:event.target.id,Xcord:position.x,Ycord:position.y,text:Text,type:type})

            }else if(type == 'checkbox'){
              console.log('check')
              for (var x = 0; x < component.checkbxCoordinate.length; x++) {
                if (component.checkbxCoordinate[x]['id'] == event.target.id) {
                  console.log('duplicate id')
                  component.checkbxCoordinate.splice(component.checkbxCoordinate[x]['id'], 1);

                }
              }
              component.checkbxCoordinate.push({id:event.target.id,Xcord:position.x,Ycord:position.y,text:Text,type:type})
            }else{
              console.log('input')
              for (var y = 0; y < component.inputCoordinate.length; y++) {
                if (component.inputCoordinate[y]['id'] == event.target.id) {
                  console.log('duplicate id')
                  component.inputCoordinate.splice(component.inputCoordinate[y]['id'], 1);

                }
              }
              component.inputCoordinate.push({id:event.target.id,Xcord:position.x,Ycord:position.y,text:Text,type:type})
            }

          },

          move (event) {
            position.x += event.dx
            position.y += event.dy

            // content.insertBefore(id, content.childNodes[0])
            event.target.style.transform =
                `translate(${position.x}px, ${position.y}px)`
          },
        }
      })

    },


    //get upload file url
    handleChange(){
      var component = this

      var  url = URL.createObjectURL(event.target.files[0]);
      component.embedSrc = url+'#toolbar=0'
      component.visible = true;

      pdfjsLib.GlobalWorkerOptions.workerSrc = pdfjsWorker;
      var loadingTask = pdfjsLib.getDocument(url);

      loadingTask.promise.then(function(pdf) {
        console.log('PDF loaded');

        // Fetch the first page

        var totalpages = pdf.numPages;
        component.TotPage = totalpages

        for(var x= 1; x < totalpages; x++){
          component.Pageindex.push({pageIndex:x})
        }

        if(totalpages > 1) {
          component.Pageindex.forEach((element)=>{
            pdf.getPage(element.pageIndex).then(function(page) {
              console.log('Page loaded');

              var scale = 1.5;
              var viewport = page.getViewport({scale: scale});

              // Prepare canvas using PDF page dimensions
              var canvas = document.getElementById('the-canvas'+element.pageIndex);

              var context = canvas.getContext('2d');
              canvas.height = viewport.height;
              canvas.width = viewport.width;

              // Render PDF page into canvas context
              var renderContext = {
                canvasContext: context,
                viewport: viewport
              };
              var renderTask = page.render(renderContext);
              renderTask.promise.then(function () {
                console.log('Page rendered');
              });
            });
          })
        }else{
          var pagenumber = 1;
          pdf.getPage(pagenumber).then(function(page) {
            console.log('Page loaded');

            var scale = 1.5;
            var viewport = page.getViewport({scale: scale});

            // Prepare canvas using PDF page dimensions
            var canvas = document.getElementById('the-canvas1');

            var context = canvas.getContext('2d');
            canvas.height = viewport.height;
            canvas.width = viewport.width;

            // Render PDF page into canvas context
            var renderContext = {
              canvasContext: context,
              viewport: viewport
            };
            var renderTask = page.render(renderContext);
            renderTask.promise.then(function () {
              console.log('Page rendered');
            });
          });
        }


      }, function (reason) {
        // PDF loading error
        console.error(reason);
      });
    },

    printActive(){

      this.visible= true;
    },

    //add new items into array
    addItem(){

      this.items.push({id:this.cardId++,text:this.name,type:'sign'})
      // var content = document.getElementById('pdfagent-1');

    },


    //add inputs
    addInputs(){
      this.inputs.push({id:'input'+this.cardId++,text:this.inputvalue,type:'input'})
    },

    //add checkbox
    addCheckBox(){
      this.checkboxes.push({id:'checkbox'+this.cardId++,text:this.checkboxName,type:'checkbox'})
    },

    //save final cordinates into array
    saveCordinate(){
      var component = this
      if(component.signCoordinate.length > 0){
        component.signCoordinate.forEach((element) => {
          component.results.push({id:element.id,Xcord:element.Xcord,Ycord:element.Ycord,text:element.text,type:element.type});
        });

      }

      if(component.inputCoordinate.length > 0){
        component.inputCoordinate.forEach((element) => {
          component.results.push({id:element.id,Xcord:element.Xcord,Ycord:element.Ycord,text:element.text,type:element.type});
        });

      }

      if(component.checkbxCoordinate.length > 0){
        component.checkbxCoordinate.forEach((element) => {
          component.results.push({id:element.id,Xcord:element.Xcord,Ycord:element.Ycord,text:element.text,type:element.type});
        });

      }

      component.dialog=true
    },

    userActive(){
      var component = this
      if(component.select == 1){
        component.isCustomer = true
        component.isAgent = false

      }else{
        component.isAgent = true
        component.isCustomer = false


      }
    },

    //remove append elements from body
    removeItems(){
      var component = this

      for (var i =0; i < component.items.length; i++){
        var elem = document.getElementById(component.items[i]['id']);
        elem.style.display='none'
        component.items.splice(component.items[i]['id'], 1);
        component.cordinates.splice(component.items[i], 1)
      }

      for (var x =0; x < component.inputs.length; x++){
        var elem2 = document.getElementById(component.inputs[x]['id']);
        elem2.style.display='none'
        component.inputs.splice(component.inputs[x]['id'], 1);
        component.cordinates.splice(component.inputs[x], 1)
      }

      for (var y =0; y < component.checkboxes.length; y++){
        var elem3 = document.getElementById(component.checkboxes[y]['id']);
        elem3.style.display='none'
        component.checkboxes.splice(component.checkboxes[y]['id'], 1);
        component.cordinates.splice(component.checkboxes[y], 1)
      }

    },





    CancelChanges(){
      var component = this
      for (var i =0; i < component.custChanges.length; i++){
        var elem = document.getElementById(component.custChanges[i]['id']);
        elem.style.display='none'

      }
    }


  }


};
</script>

<style>
.bottom{
  position: absolute;
  width: 100%;
  bottom: 0;
  list-style-type: none;
  padding-bottom:5.5em;
}

.draggable {
  /*width: 100px;*/
  /*height: 100px;*/
  /*background-color: teal;*/
  /*color: #fff;*/
  /*padding: 5px;*/

  position: absolute;
}

#drop-zone{
  position: relative;
}
</style>
