<style  scoped>


  .drag-scale-modal .modal-mask{
    position: fixed;
    left: 0;
    top: 0;
    width: 100%;
    background-color: rgba(55, 55, 55, 0.6);
    height: 100vh;
    overflow: hidden;
  }
  .drag-scale-modal .modal-content {
    z-index: 1;
    position: fixed;
    overflow: auto;
    top: 0;
    right: 0;
    bottom: 0;
    left: 0;
    box-shadow: 0 0 12px 0 rgba(38,38,38,0.1);
    overflow: hidden;




  }
  .fade-enter-active, .fade-leave-active {
    transition: opacity .5s;
  }
  .fade-enter, .fade-leave-to /* .fade-leave-active below version 2.1.8 */ {
    opacity: 0;
  }
  .drag-content:hover{
    border: 1px solid #2F9AFD;
  }
  .drag-content {
    background-color: #fff;
    border-radius: 6px;
    position: relative;
    overflow: hidden;
    border: 1px solid #F2FAFF;

  }
  .boundary {
    position: absolute;
    width: 15px;
    height: 15px;
  }

  .disabled {
    pointer-events: none;
  }

  .e-resize {
    right: 0;
    height: 100%;
    cursor: e-resize;
  }

  .w-resize {
    left: 0;
    height: 100%;
    cursor: w-resize;
  }

  .n-resize {
    top: 0;
    width: 100%;
    cursor: n-resize;
  }

  .s-resize {
    bottom: 0;
    width: 100%;
    cursor: s-resize;
  }

  .ne-resize {
    right: 0;
    cursor: ne-resize;

    z-index: 2;
  }

  .nw-resize {
    left: 0;
    cursor: nw-resize;

    z-index: 3;
  }

  .se-resize {
    right: 0;
    bottom: 0;
    cursor: se-resize;

    z-index: 4;
  }

  .sw-resize {
    bottom: 0;
    cursor: sw-resize;

    z-index: 5;
  }

</style>

<template>

  <div class="drag-scale-modal" v-show="value">
    <div class="modal-mask" v-show="config.mask" @click="maskClick"></div>
    <transition name="fade" >
      <div v-if="value" class="modal-content" :style="modalStyle">
        <div ref="dragContent" @mousedown.stop="dragContent" class="drag-content" :style="dragStyle">
            <span class="boundary" v-for="(item,index) in boundaryArr" :key="index"
                  :class="item.className+(item.disabled?' disabled':'')" @mousedown.stop="dragDown($event,item)"></span>
          <div :style="contentStyle">
            <slot name="content"></slot>
          </div>
        </div>
      </div>
    </transition>
  </div>
</template>

<script>
  /**
   *
   * 边界缩放功能，对应css3 的cursor值
   * e-resize  此光标指示矩形框的边缘可被向右（东）移动。
   * ne-resize  此光标指示矩形框的边缘可被向上及向右移动（北/东）。
   * nw-resize  此光标指示矩形框的边缘可被向上及向左移动（北/西）。
   * n-resize  此光标指示矩形框的边缘可被向上（北）移动。
   * se-resize  此光标指示矩形框的边缘可被向下及向右移动（南/东）。
   * sw-resize  此光标指示矩形框的边缘可被向下及向左移动（南/西）。
   * s-resize  此光标指示矩形框的边缘可被向下移动（南）。
   * w-resize  此光标指示矩形框的边缘可被向左移动（西）。
   */
  const boundaryArr = [
    { position: 'right', className: 'e-resize', disabled: false },
    { position: 'top-right', className: 'ne-resize', disabled: true },
    { position: 'top-left', className: 'nw-resize', disabled: true },
    { position: 'top', className: 'n-resize', disabled: true },
    { position: 'bottom-right', className: 'se-resize', disabled: false },
    { position: 'bottom-left', className: 'sw-resize', disabled: true },
    { position: 'bottom', className: 's-resize', disabled: false },
    { position: 'left', className: 'w-resize', disabled: true }
  ]
  export default {
    name: 'DragScaleModal',
    created () {
      // console.log(this.config)
      this.setDisabled()
      document.addEventListener('mousemove', this.dragMove)
      document.addEventListener('mouseup', this.dragUp)
    },
    beforeDestroy () {
      document.removeEventListener('mousemove', this.dragMove)
      document.removeEventListener('mouseup', this.dragUp)
    },
    data () {
      return {
        transform: {
          x: this.config.x ? this.config.x : (window.innerWidth - this.config.width) / 2,
          y: this.config.y ? this.config.y : 0
        },
        page: {
          transformX: 0,
          transformY: 0,
          x: 0,
          y: 0,
          width: 0,
          height: 0
        },
        curDragItem: null,
        curDragContent: null,
        width: this.config.width,
        height: this.config.height,
        boundaryArr: boundaryArr
      }
    },
    props: {
      value: {
        type: Boolean,
        default: false
      },
      // 可自定义开启8个方向的拖拽缩放，默认开启右，下，右下
      config: {
        type: Object,
        default () {
          return {
            x: 0,
            y: 0,
            // 默认显示mask
            mask: true,
            width: 520,
            height: 300,
            left: false,
            right: true,
            top: false,
            bottom: true,
            'bottom-right': true,
            'top-right': false,
            'bottom-left': false,
            'top-left': false,
            reset: true,
            allowOutOfBoundary: false,
            maskClose:true
          }
        }
      }
    },
    watch: {
      visible () {
        if (this.config.reset) {
          this.transform = {
            x: this.config.x ? this.config.x : (window.innerWidth - this.config.width) / 2,
            y: this.config.y ? this.config.y : 0
          }
          this.page = {
            transformX: 0,
            transformY: 0,
            x: 0,
            y: 0,
            width: 0,
            height: 0
          }
          this.height = this.config.height
          this.width = this.config.width
        }
      }
    },
    computed: {
      contentStyle () {
        return {
          userSelect: this.curDragItem || this.curDragContent ? 'none' : 'all',
          height: this.height ? this.height + 'px' : this.config.height + 'px'
        }
      },
      dragStyle () {
        return {
          cursor: this.curDragContent ? 'move' : 'move',
          width: this.width ? this.width + 'px' : this.config.width + 'px',
          height: this.height ? this.height + 'px' : this.config.height + 'px'
          // border: this.curDragContent || this.curDragItem ? '1px solid #2F9AFD' : '1px solid #2F9AFD'
        }
      },
      modalStyle () {
        return {
          transform: 'translate(' + this.transform.x + 'px,' + this.transform.y + 'px)',
          width: this.width ? this.width + 'px' : this.config.width + 'px',
          height: this.height ? this.height + 'px' : this.config.height + 'px'
        }
      }
    },
    methods: {
      maskClick(){
        if(this.config.maskClose){
          this.close();
        }
      },
      close(){
        this.$emit('input',false)
      },
      setDisabled () {
        this.boundaryArr.forEach(item => {
          item.disabled = !this.config[item.position]
        })
        // console.log(this.boundaryArr)
      },
      dragContent (e) {
        e.stopPropagation()
        this.curDragContent = this.$refs.dragContent
        this.setCoordinate(e)
      },
      setCoordinate (e) {
        const rect = this.$refs.dragContent.getBoundingClientRect()
        this.page = {
          x: e.pageX,
          y: e.pageY,
          width: rect.width,
          height: rect.height,
          transformX: this.transform.x,
          transformY: this.transform.y
        }
      },
      dragMove (e) {
        if (this.curDragItem) {
          e.stopPropagation()
          window.getSelection ? window.getSelection().removeAllRanges() : document.selection.empty()
          // console.log("缩放中",this.curDragItem.position)
          const pos = this.curDragItem.position
          if (pos.indexOf('right') !== -1) {
            const right = this.page.width + (e.pageX - this.page.x)
            if (this.transform.x + right <= window.innerWidth || this.config.allowOutOfBoundary) {
              this.width = right
            }
            //  this.page.transform.x=this.page.transformX+(e.pageX-this.page.x)/2
          }
          if (pos.indexOf('left') !== -1) {
            const left = this.page.width + (this.page.x - e.pageX)
            const x = this.page.transformX - (this.page.x - e.pageX)
            if (x >= 0 || this.config.allowOutOfBoundary) {
              this.width = left
              this.transform.x = x
            }
          }
          if (pos.indexOf('top') !== -1) {
            const top = this.page.height + (this.page.y - e.pageY)
            const y = this.page.transformY - (this.page.y - e.pageY)
            if (y >= 0 || this.config.allowOutOfBoundary) {
              this.height = top
              this.transform.y = y
            }
          }
          if (pos.indexOf('bottom') !== -1) {
            const bottom = this.page.height + (e.pageY - this.page.y)
            if (this.transform.y + bottom <= window.innerHeight || this.config.allowOutOfBoundary) {
              this.height = bottom
            }
          }
        } else if (this.curDragContent) {
          e.stopPropagation()
          window.getSelection ? window.getSelection().removeAllRanges() : document.selection.empty()
          // console.log(this.transform.x,this.transform.y)
          const curY = e.pageY - this.page.y + this.page.transformY
          const curX = e.pageX - this.page.x + this.page.transformX
          if (curX > 0 && curX <= window.innerWidth - this.width || this.config.allowOutOfBoundary) {
            this.transform.x = e.pageX - this.page.x + this.page.transformX
          }
          if (curY > 0 && curY <= window.innerHeight - this.height || this.config.allowOutOfBoundary) {
            this.transform.y = e.pageY - this.page.y + this.page.transformY
          }
        }
      },
      dragUp (e) {
        if (this.curDragItem) {
          e.stopPropagation()
          this.curDragItem = null
        }
        if (this.curDragContent) {
          e.stopPropagation()
          this.curDragContent = null
        }
      },
      dragDown (e, item) {
        e.stopPropagation()
        this.curDragItem = item
        this.setCoordinate(e)

        // console.log('缩放位置',item.position)
      }
    }
  }
</script>
