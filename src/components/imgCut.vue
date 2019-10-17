<template>
<div v-bind="$attrs">
  <canvas class="canvas" @mousedown="isPoint=true;oldOffset={x:0,y:0}" @mouseup="isPoint=false" @mouseleave="isPoint=false" @mousemove="moveCvs" ref='myImgCutCvs' v-bind="$attrs" :height="height" :width="width" />
  <img v-if="isReview" :class="reviewClass" :src='reviewData' style="border: solid 2px #e6e6e6;overflow: hidden;" :style="{borderRadius:arc+'px',height:arc+'px',width:arc+'px'}" />
  <canvas style="display:none;" ref='myCutImg' :width="cutImgWidth" :height="cutImgHeight" />
  <slot />
</div>
  
  
</template>

<script>
const radius = 45
export default {
  name: 'ImgCut',
  props: {
    reviewClass:{
      default:'',
      type:String
    },
    isReview:{
      default:true,
      type:Boolean
    },
    height:{
      default:300,
      type:Number
    },
    width:{
      default:300,
      type:Number
    },
    cutDown:{
      default:()=>{},
      type:Function
    },
    imgData:{
      default:()=>{},
      type:Object
    }
  },
  data(){
    return {
      centreOfCircleX:300,
      centreOfCircleY:300,
      lineWidth:900,
      isPoint:false,
      cutImgWidth:radius * 2,
      cutImgHeight:radius * 2,
      arc:radius * 2,
      oldOffset:{
        x:0,
        y:0
      },
      reviewData:''
    }
  },
  methods:{
    init(){
      this.centreOfCircleX = this.width/2
      this.centreOfCircleY = this.height/2
      this.cutImgWidth = radius * 2
      this.cutImgHeight = radius * 2
      this.lineWidth = (this.width > this.height ? this.width : this.height) * 3
      let el = this.$refs['myImgCutCvs']
      this.ctx = el.getContext('2d')
      this.drawing()
    },
    moveCvs(e){
      if(this.isPoint){
        let { offsetX, offsetY} = e
        if(this.oldOffset.x === 0 && this.oldOffset.y === 0){
          this.oldOffset = {
            x:offsetX,
            y:offsetY
          }
          return
        }
        let x = offsetX - this.oldOffset.x
        let y = offsetY - this.oldOffset.y
        let newX = this.centreOfCircleX + x
        let newY = this.centreOfCircleY + y
        
        let newXr = newX + radius
        let newYr = newY + radius
        
        if(newXr  > this.width){
          newX = this.width - radius
        }
        if(newX < radius){
          newX = radius
        }
        if(newYr > this.height){
          newY = this.height - radius
        }
        if(newY < radius){
          newY = radius
        }

        this.centreOfCircleX = newX
        this.centreOfCircleY = newY
        if(!this.timeout){
          this.timeout = setTimeout(()=>{
            this.timeout = undefined
            this.drawing()
          },100)
        }
        this.oldOffset = {
          x:offsetX,
          y:offsetY
        }
      }
      
    },
    drawing(){
      this.dwImg()
      this.dwArc()
    },
    dwImg(){
      this.ctx.clearRect(0, 0, this.width, this.height);
      let img = new Image()
      img.src=this.imgData
      this.ctx.drawImage(img, 0, 0, this.width, this.height)
    },
    dwArc(){
      this.ctx.beginPath()   
      this.ctx.lineWidth = this.lineWidth
      this.ctx.arc(this.centreOfCircleX,this.centreOfCircleY,this.lineWidth/2 + radius,0,360,false);   
      this.ctx.strokeStyle="rgba(24,25,27,0.64)";
      this.ctx.stroke()
      this.ctx.closePath()
      this.toDataURL()
    },
    toDataURL(){
      let ctx = this.$refs['myCutImg'].getContext('2d')
      let imgData = this.ctx.getImageData(this.centreOfCircleX-radius,this.centreOfCircleY-radius,radius*2,radius*2)
      ctx.clearRect(0,0,radius*2,radius*2)
      ctx.putImageData(imgData,0,0)
      this.reviewData = this.$refs['myCutImg'].toDataURL("image/png")
    },
  },
  mounted(){
    this.init()
  },
  
  created(){
    
  }
}
</script>

<style>
.canvas{
  border:solid 1px #3e3e3e;
  
}
</style>
