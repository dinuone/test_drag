<template>

  <div>
    <v-file-input
                  label="Upload document"
                  truncate-length="15"
                  @change="handleChange"
    ></v-file-input>
    <v-btn color="primary" class="mt-2 ml-3"  width="200" @click="addElementToBody">Append Elements</v-btn>
    <v-btn color="info" class="mt-2 ml-3"  width="200" @click="printWindow" >Download PDF </v-btn>
    <v-btn color="success" class="mt-2 ml-3"  width="200" @click="SaveCustChange">Save</v-btn>
    <v-dialog
        v-model="signmodal"
        width="600"
    >

      <v-card>
        <v-card-title class="text-h5 grey lighten-2">
          Your Signature
        </v-card-title>
        <p class="text-center">
          <canvas  ref="canvas" width='500' height='500' style="border: 1px solid black;" id="canvas"></canvas>
        </p>

        <v-card-actions>
          <v-spacer></v-spacer>
          <v-btn color="red" @click="resetCanvas">
            Reset
          </v-btn>
          <v-btn
              color="primary"
              text
              @click='saveImage'
          >
            Save
          </v-btn>
        </v-card-actions>
      </v-card>
    </v-dialog>

    <div  id="drop-zone" style="width:1000px; height: 100%;" >

      <div id="mypdf">
        <div v-for="data in Elementdata" :key="data.id" >
<!--          <p style="border: none" v-if="data.type == 'sign'"-->
<!--             :id="data.id"-->
<!--             class="mt-3"-->
<!--             width="200"-->
<!--             text-color="white"-->
<!--          >-->
<!--            {{data.text}}-->
<!--          </p>-->
          <img :src="DataUrl" alt="signature" style="width: 100px; height: 60px" v-if="data.type == 'sign'" ref="img" :id="data.id">

          <div v-if="data.type == 'checkbox'"  :id="data.id">
            <v-checkbox
                :label="data.text"
                v-model="custCheckBox"
            >
            </v-checkbox>
          </div>


          <div v-if="data.type == 'input'" :id="data.id">

            <v-text-field
                :label="data.text"
                :value="data.text"
                class="mx-4"
                v-model="custInput"
            ></v-text-field>
          </div>


        </div>

        <!--      final result-->
        <div v-for="finalData in custChanges" :key="finalData.id">
<!--          <p style="border: none" v-if="finalData.type == 'sign'"-->
<!--             :id="finalData.id"-->
<!--             class="mt-3"-->
<!--             width="200"-->
<!--             text-color="white"-->
<!--          >-->
<!--            {{finalData.text}}-->
<!--          </p>-->
          <img :src="DataUrl" alt="signature" style="width: 100px; height: 60px" v-if="finalData.type == 'sign'" ref="img" :id="finalData.id">

          <div v-if="finalData.type == 'checkbox'"  :id="finalData.id" >
            <input type="checkbox"  :value="finalData.value" :checked="finalData.value" disabled>
          </div>


          <div v-if="finalData.type == 'input'" :id="finalData.id">
            <p>
              {{finalData.text}}
            </p>

          </div>
        </div>

        <canvas :id="'the-canvas'+render"  style="width: 800px; height: 1100px" v-for="render in TotPage" :key="render.id"></canvas>

      </div>

    </div>

  </div>


</template>

<script>
// import DragItDude from 'vue-drag-it-dude';
// import Vue from "vue";
// import VuePdfEmbed from 'vue-pdf-embed/dist/vue2-pdf-embed'
//
import html2pdf from 'html2pdf.js'
import * as pdfjsLib from 'pdfjs-dist'
import pdfjsWorker from 'pdfjs-dist/build/pdf.worker.entry'
export default {
  name: 'Customer-view',
  components: {
    // VuePdfEmbed

  },
  props:['id','reuslts'],
  mounted () {

    var vm = this
    vm.canvas = vm.$refs.canvas
    vm.context = vm.canvas.getContext("2d");
    vm.canvas.addEventListener('mousedown', vm.mousedown);
    vm.canvas.addEventListener('mousemove', vm.mousemove)
    document.addEventListener('mouseup', vm.mouseup);

  },

  data(){
    return{
      text: "Just move me!",
      parentHeight:null,
      parentWidth:null,
      source1: 'https://raw.githubusercontent.com/mozilla/pdf.js/ba2edeae/web/compressed.tracemonkey-pldi-09.pdf',
      Elementdata:this.reuslts,
      custCheckBox:null,
      custInput:null,
      custChanges:[],
      TotPage:0,
      Pageindex:[],
      signmodal:true,
      canvas: null,
      context: null,
      isDrawing: false,
      startX: 0,
      startY: 0,
      points: [],
      DataUrl:''
    }

  },

  methods:{
    printWindow: function () {
      var element = document.getElementById('drop-zone');
      var opt = {

        filename:     'myfile.pdf',
        image:        { type: 'jpeg', quality: 0.98 },
        html2canvas:  { scale: 1 },
        jsPDF:        { unit: 'mm', format: 'a4', orientation: 'portrait' },

      };

      // New Promise-based usage:
      html2pdf().set(opt).from(element).save();

      // Old monolithic-style usage:
      html2pdf(element, opt);
    },

    //drawing
    mousedown(e){
      var vm = this
      var rect = vm.canvas.getBoundingClientRect();
      var x = e.clientX - rect.left;
      var y = e.clientY - rect.top;

      vm.isDrawing = true;
      vm.startX = x;
      vm.startY = y;
      vm.points.push({
        x: x,
        y: y
      });
    },

    mousemove(e){
      var vm = this
      var rect = vm.canvas.getBoundingClientRect();
      var x = e.clientX - rect.left;
      var y = e.clientY - rect.top;

      if (vm.isDrawing) {
        vm.context.beginPath();
        vm.context.moveTo(vm.startX, vm.startY);
        vm.context.lineTo(x, y);
        vm.context.lineWidth = 2;
        vm.context.lineCap = 'round';
        vm.context.strokeStyle = "rgba(0,0,0,1)";
        vm.context.fillStyle = "#800";
        vm.context.stroke();

        vm.startX = x;
        vm.startY = y;

        vm.points.push({
          x: x,
          y: y
        });
      }
    },

    mouseup(){
      var vm = this
      vm.isDrawing = false;
      if (vm.points.length > 0) {
        localStorage['points'] = JSON.stringify(vm.points);
      }
    },

    resetCanvas(){
      var vm = this
      vm.canvas.width = 500;
      vm.points.length = 500; // reset points array
    },

    saveImage(){
      // var vm = this
      var dataURL = this.canvas.toDataURL();
      this.DataUrl = dataURL
      // var img = this.$refs.img;
      // img.src = dataURL;

      var link = document.createElement('a');
      link.download = 'sign.png';
      link.href = document.getElementById('canvas').toDataURL()
      link.click();
    },


    //get saved coordinates and append to body
    addElementToBody(){
      var component = this
      component.Elementdata.forEach((element) => {
        var elm = document.getElementById(element.id);
        // var content = document.getElementById('pdfview-1')
        console.log(elm);
        if(element.type =='checkbox'){
          // content.insertBefore(elm, content.childNodes[0])
          elm.style.transform =
              `translate(${element.Xcord}px, ${element.Ycord}px)`
          elm.style.position='absolute';
          elm.style.display='inline';

        }else if(element.type =='input'){

          // content.insertBefore(elm, content.childNodes[0])
          elm.style.transform =
              `translate(${element.Xcord}px, ${element.Ycord}px)`
          elm.style.position='absolute';
          elm.style.display='inline';

        }else if(element.type == 'sign'){

          // content.insertBefore(elm, content.childNodes[0])

          elm.style.transform =
              `translate(${element.Xcord}px, ${element.Ycord}px)`
          elm.style.borderRadius='10px';
          elm.style.border ='1px solid';
          elm.style.padding='6px';
          elm.style.position='absolute';


        }


      });

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

    SaveCustChange(){
      var component = this
      if(component.Elementdata.length > 0){
        component.Elementdata.forEach((element) => {
          if(element.type == 'input'){
            if(component.custInput != ''){
              component.custChanges.push({id:element.id,Xcord:element.Xcord,Ycord:element.Ycord,text:element.text,type:element.type,value:component.custInput});
            }else{
              component.custChanges.push({id:element.id,Xcord:element.Xcord,Ycord:element.Ycord,text:element.text,type:element.type,value:element.text});
            }


          }else if(element.type == 'checkbox'){
            component.custChanges.push({id:element.id,Xcord:element.Xcord,Ycord:element.Ycord,text:element.text,type:element.type,value:component.custCheckBox});
          }else{
            component.custChanges.push({id:element.id,Xcord:element.Xcord,Ycord:element.Ycord,text:element.text,type:element.type,value:element.text});
          }

        });
        component.Elementdata=[];

        this.custChanges.forEach((element) => {

          var elm = document.getElementById(element.id);
          if(element.type == 'sign'){
            elm.style.transform =
                `translate(${element.Xcord}px, ${element.Ycord}px)`
            elm.style.position='absolute';
            elm.style.display='inline';
            elm.style.border='none';
          }else if(element.type == 'checkbox'){
            elm.style.transform =
                `translate(${element.Xcord}px, ${element.Ycord}px)`
            elm.style.position='absolute';
            elm.style.display='inline';

          }else{
            elm.style.transform =
                `translate(${element.Xcord}px, ${element.Ycord}px)`
            elm.style.position='absolute';
            elm.style.display='inline';
          }


        });
      }
    },


  },




}
</script>

<style scoped>
#drop-zone{
position: relative;
}

#mypdf{
  position: relative;
}

#signpad {
  border: 3px solid black;
  height: 500px;
  width: 500px;
  margin: 10px 50px 10px 50px;
}



</style>




