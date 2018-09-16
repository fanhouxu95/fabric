<template>
  <div id="container">
        <ul>
          <li @click="chooseText">文字</li>
          <li>线</li>
          <li>竖线</li>
          <li>矩形线</li>
          <li>图片</li>
        </ul>
        <canvas id="paint" width="500px" height="500px"></canvas>
        <div id="leftCont">
            

        </div>
        <button id="btn">删除</button>
    </div>
</template>

<script>
import {fabric} from 'fabric'
import Vue from 'vue'
export default {
  name: 'index',
  data () {
    return {
      config:[],
      fb: null,
      view: null,
    }
  },
  mounted(){
      this.init();
  },
  methods:{
    //选择文字
    chooseText(){
      let text = {
            type: 'Line',
            value:[10, 10, 100, 10],
            config: {
                strokeWidth: 1,
                stroke:'#000',
                angle: 0  //旋转角度
            },
          };

        // let text = {
        //     type: 'IText',
        //     value: 'aaaa',
        //     config: {
        //         left: 100,
        //         top: 200,
        //         fill: '#000',
        //         width: 20,
        //         height: 20,
        //         angle: 0  //旋转角度
        //     },
        //   };
        this.add(text);
    },

     test() { //测试config，需要删除
        let config = {
            type: 'Rect',
            config: {
                left: 100,
                top: 100,
                fill: 'red',
                width: 20,
                height: 20,
                angle: 45  //旋转45堵
            },
          };
        let config1 = {
            type: 'IText',
            value: 'aaaa',
            config: {
                left: 100,
                top: 200,
                fill: 'blue',
                width: 20,
                height: 20,
                angle: 45  //旋转45堵
            },
          };
        this.add(config);
        this.add(config1);


    },

    init() {
        this.fb = new fabric.Canvas('paint');
        this.test();
    },

    add(conf) {
        if(!conf)
            throw 'config error!'
        let ren, type = conf.type;
        
        if(type == 'IText') {
            ren = new fabric[type](conf.value, conf.config);
        }else if(type == 'Line'){
            ren = new fabric[type](conf.value, conf.config);
        }else{
            ren = new fabric[type](conf.config);

        }
        this.fb.add(ren);
        this.config.push(ren);
        this.addEvent(ren);
    },

    remove() {
        if(this.index()) {
            this.destroy();
            this.config.splice(this.index(), 1);
            this.fb.remove(this.selectConfig());
        }
    },

    addEvent(ren) {
        ren.on('selected', (e) => { //选择一个元素时
            document.querySelector('#leftCont').innerHTML = this.viewTemplet();
            ren.swidth = (ren.scaleX || 1) * ren.width;
            ren.sheight = (ren.scaleY || 1) * ren.height;
            this.view = new Vue({
                el: '#leftCont',
                data: ren,
                methods: {
                    refresh: () => {
                        this.refresh();
                    },
                    swFun: function () {
                        this.scaleX = this.swidth / this.width;
                        this.refresh();
                    },
                    shFun: function () {
                        this.scaleY = this.sheight / this.height;
                        this.refresh();
                    }
                }
            });
        });
        ren.on('deselected', () => {
            this.destroy();
        });
        ren.on('scaled', () => { //用于触发vue的计算属性改变width和height的显示
            if(!this.view) {
                console.error('selected error');
                return;
            }
            ren.swidth = ren.scaleX * ren.width;
            ren.sheight = ren.scaleY * ren.height;
        });
        document.querySelector('#btn').addEventListener('click',() => {
            this.remove();
        });
    },


    refresh() { //刷新画布
        this.fb.renderAll();
    },

    clear() { //清楚画布
        this.fb.clear();
        this.config.splice(0, this.config.length);
    },

     index(){ //当前选择的index
        return this.config.reduce( (p, c, i) => {
            this.fb.getActiveObject() === c && ( p = i );
            return p;
        }, -1);
    },

    selectConfig() {
        return this.fb.getActiveObject();
    },

   viewTemplet() { //vue实例时里面的html
        return `x<input type="number" v-model.number="left" @change="refresh"><br>
        y<input type="number" v-model.number="top" @change="refresh"><br>
        宽<input type="number" v-model.number="swidth" @change="swFun"><br>
        高<input type="number" v-model.number="sheight" @change="shFun"><br>
        角度<input type="number" v-model.number="angle" @change="refresh"><br>`;
    },
    
    destroy() { //注销vue实例
        if(this.view)
            this.view.$destroy();
        this.view = null;
    },

    save() {

    },
    load() {
        
    },
  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
#container {
            width: 100%;
            position: relative;
        }
        #paint {
            border: 1px solid #ccc;
        }
        #leftCont {
            width: 300px;
            position: absolute;
            right: 0;
            top: 0;
        }
</style>
