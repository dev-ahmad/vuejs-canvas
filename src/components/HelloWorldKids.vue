<template>
  <div class="helloWordlKids">

    <div class="ShapesContainer">

      <div  class="col-md-5 mainContainer">
          <draggable class="list-group" element="ul" v-model="list" :options="dragOptions" :move="onMove" @start="isDragging=true" @end="isDragging=false">
            <transition-group type="transition" :name="'flip-list'">
              <li class="list-shape-item" v-for="element in list" :key="element.order">
                <canvas v-draw-canvas="element" width="500px" height="170px" v-bind:style="{transform: 'rotate('+ element.rotate+'deg)'}"></canvas>
              </li>
            </transition-group>
        </draggable>
      </div>

      <div class="col-md-5 mainContainer">
          <draggable class="list-group" element="ul" v-model="list" :options="dragOptions" :move="onMove" @start="isDragging=true" @end="isDragging=true">
            <transition-group type="transition" :name="'flip-list'">
              <li class="list-group-item" v-for="element in list" :key="element.order">

                <div v-if="editCanvas == false">
                  <select
                    v-model="element.name"
                    v-on:change="onShapeChange(element)"
                    style="margin-top:50px;width:25%;float:left;">
                    <option value="circle" >Circle</option>
                    <option value="triangle" >Triangle</option>
                    <option value="rectangle" >Rictangle</option>
                  </select>
                  <input v-if="element.name == 'rectangle'" type="range" v-model="element.height" min="0" max="160"  style="margin-right:20px;margin-left:20px;margin-top:50px;width:50%;float:left;">
                  <input v-if="element.name == 'circle'" type="range"  v-model="element.radius "  min="0" max="50" style="margin-right:20px;margin-left:20px;margin-top:50px;width:50%;float:left;">
                  <input v-if="element.name == 'triangle'" type="range"  v-model="element.height" min="0" max="140" style="margin-right:20px;margin-left:20px;margin-top:50px;width:50%;float:left;">
                  <input type="color" v-model="element.backgroundColor" style="margin-top:50px;width:10%;float:left;">
                  <i class="el-icon-delete" @click="deleteShape(element)" style="margin-top:100px;float:left;"></i>
                </div>

                <div v-if="editCanvas == true">
                  <input v-if="element.name == 'rectangle'" type="text" v-model="element.text" style="margin-left:120px;margin-top:50px;width:50%;float:left;">
                  <input class="triangleInputRangeOne" v-if="element.name == 'triangle'" type="range" v-model="element.rotate" min="0" max="360">
                  <input class="circleInputRangeOne" v-if="element.name == 'circle'" type="range" v-model="element.width" min="0" max="230" @channge="resizeCircle(element)" >
                  <input class="circleInputRangeTwo" v-if="element.name == 'circle'" type="range" v-model="element.height" min="0" max="68" @channge="resizeCircle(element)"  >
                </div>

              </li>
            </transition-group>
            <el-button v-show="editCanvas == false" style="margin:10px;" class="bottomButton" type="primary" icon="el-icon-edit" @click="changeNotEditable()"></el-button>
            <el-button v-show="editCanvas == true" style="margin:10px;" class="bottomButton" type="primary" icon="el-icon-circle-check-outline" @click="changeNotEditable()"></el-button>
            <el-button style="margin:10px;" class="bottomButton" type="primary" icon="el-icon-plus" @click="addNewShap()"></el-button>
        </draggable>
      </div>

    </div>
  </div>
  </div>
</template>

<script>
import draggable from 'vuedraggable'

export default {
  name: 'hello',
  components: {
    draggable,
  },
  data () {
    return {
      elementOrder : 4,
      editCanvas : false,
      list: [ {    name: 'rectangle', order: 1, fixed : false ,width: 150, height: 80, radius : 1 ,  backgroundColor: "#03A9F4", rotate: 0, text: ''   },
              {    name: 'circle',  order: 3, fixed : false ,  width: 40, height: 40, radius : 1,  backgroundColor: "#FF0033",  rotate: 0, text: '' },
              {    name: 'triangle',  order: 2, fixed : false , width: 0, height: 100, radius : 1 ,  backgroundColor: "#F3A9F4", rotate: 0, text: ''  }],
      isDragging: false,
      delayedDragging:false
    }
  },
  directives: {
    drawCanvas: function (canvasElement, binding) {
      canvasElement.width = canvasElement.width;
      canvasElement.height = canvasElement.height;
      var ctx = canvasElement.getContext('2d');
      ctx.fillStyle = binding.value.backgroundColor;
      ctx.clearRect(0,0,canvasElement.width,canvasElement.height);
      if(binding.value.name == 'triangle'){
        var trianglex = 50-(binding.value.height / 3);
        var triangley = 220 - (binding.value.height / 3)
        ctx.moveTo((250 - binding.value.width)  , (trianglex - binding.value.width) );
        ctx.lineTo(triangley , 110 + (binding.value.height / 3));
        ctx.lineTo((280 - binding.value.width) + (binding.value.height / 3), 110 + (binding.value.height / 3) );
        ctx.fill();
        ctx.restore();
      }
      else if (binding.value.name == "rectangle") {
        var x = ( 500 -  (binding.value.height * 2) ) / 2
        var y = ( 170 -  (binding.value.height) ) / 2
        ctx.fillRect(x, y, binding.value.height * 2, binding.value.height);
        ctx.fillStyle = "white";
        ctx.font = "20px aral";
        ctx.textBaseline = 'hanging';
        ctx.fillText(binding.value.text,x,y);
      } else if (binding.value.name == "circle") {
        var Height = ( parseInt(binding.value.height) + parseInt(binding.value.radius));
        var Width =  (parseInt(binding.value.width) + parseInt(binding.value.radius));
        console.log(Height);
        ctx.ellipse(250, 80, Width ,Height, 0 * Math.PI/180, 0, 2 * Math.PI);
        ctx.fill();
        ctx.restore();
      }
    }
  },
  computed: {
    dragOptions () {
      return  {
        group: 'description',
        ghostClass: 'ghost'
      };
    }
  },
  watch: {
    isDragging (newValue) {
      if (newValue){
        this.delayedDragging= true
        return
      }
      this.$nextTick( () =>{
           this.delayedDragging =false
      })
    }
  },
  methods:{
    onShapeChange(shape)
    {
      if (shape.name == "triangle") {
        shape.width = 0;
        shape.height = 110;
        console.log('DONE');
      }
      if (shape.name == "circle") {
        shape.width = 30;
        shape.height = 30;
      }
      if (shape.name == 'rectangle') {
        shape.width = 150;
        shape.height = 100;
      }
    },
    onMove ({relatedContext, draggedContext}) {
      const relatedElement = relatedContext.element;
      const draggedElement = draggedContext.element;
      return (!relatedElement || !relatedElement.fixed) && !draggedElement.fixed
    },
    addNewShap(){
      this.list.push({
        name: 'circle',
        order: this.elementOrder ,
        radius: 2 ,
        fixed : false ,
        width : '50' ,
        height:'50' ,
        backgroundColor : '#D50000' ,
        text:'',
        borderLeft : 50 ,
        borderRight : 50 ,
        borderBottom : 100 ,
        rotate : 0
      })
      this.elementOrder++;
    },
    deleteShape(element){
      const index = this.list.indexOf(element);
      this.list.splice(index, 1);
    },
    changeNotEditable(){
      if(this.editCanvas == true){
        this.editCanvas = false
      }
      else {
        this.editCanvas = true
      }
    }
  }
}
</script>

<style>
  .helloWordlKids{
    overflow: scroll;
  }
  .mainContainer{
    bottom: 0px;
    margin-left: 80px;
    border: 1px solid;
  }
  .flip-list-move {
    transition: transform 0.5s;
  }
  .bottomButton{
    width:50px;
    height: 50px;
    float: right;
    padding-left: 18px !important;
    border-radius: 50% !important;
  }
  button:hover{
    background-color: #4CAF50;
    color: white !important;
  }
  .rectangle  {
    margin: auto;
    width: 200px;
    height: 100px;
  }
  .no-move {
    transition: transform 0s;
  }
  .ghost {
    opacity: .5;
    background: #C8EBFB;
  }
  .list-group {
    height: 20%;
  }
  .list-group {
    padding-left: 0;
    margin-bottom: 45px !important;
  }
  .list-group-item {
    border-top-left-radius: 16px !important;
    border-top-right-radius: 16px !important;
    border-bottom-right-radius: 16px !important;
    border-bottom-left-radius: 16px !important;
    margin:auto;
    cursor: move;
    margin:20px;
    height: 160px;
  }
  .list-shape-item{
    cursor: move;
    margin:20px;
    height: 160px;
  }
  ul{
    list-style-type: none;
  }
  .triangleInputRangeOne{
    margin-left:50px;
    margin-top:60px;
    width:400px !important;
    float:left;
  }
  .circleInputRangeOne{
    margin-left:80px;
    margin-top:60px;
    width:200px !important;
    float:left;
  }
  .circleInputRangeTwo{
    margin-right:50px;
    margin-top:60px;
    transform: rotate(90deg);
    width:120px !important;
    float:right;
  }
  .list-group-item i{
    cursor: pointer;
  }
</style>
