<template>
  <div class="container">
    <van-nav-bar
      style="background-color:#643D12"
    >
    <span slot="title" style="color:#fff">第{{id}} / {{array.length}}关</span>
    <span slot="right" style="color:#C1A06D">{{score}}</span>
    </van-nav-bar>
  <div class="picture">
    <van-image :src="obj.picture" class="img" />
  </div>
  <div class="tips">
    <van-button type="warning" size="small" @click="jumpLevel(id-1)" :disabled="!showPrev">上一关</van-button>
    <van-button type="warning" size="small" @click="checkLevel" style="margin: 0 10px 0 20px">选 关</van-button>
    <span>{{time+1}}</span>
    <van-button type="warning" size="small" @click="handlePrompt" style="margin: 0 20px 0 10px">提 示</van-button>
    <van-button type="warning" size="small" @click="jumpLevel(id+1)" :disabled="!showNext">下一关</van-button>
  </div>
  <div class="result">
    <div v-for="(item,i) in choiced" :key="i">
      <span @click="handleCancel(i)" class="choiced-text">{{item}}</span>
    </div>
  </div>
  <div class="choice">
    <div v-for="(item,i) in choices.slice(0,8)" :key="i">
      <span @click="handleChoice(item,i)" class="choice-text">{{item}}</span>
    </div>
  </div>
  <div class="choice">
    <div v-for="(item,i) in choices.slice(8,16)" :key="i">
      <span @click="handleChoice(item,i+8)" class="choice-text">{{item}}</span>
    </div>
  </div>
  <div class="choice">
    <div v-for="(item,i) in choices.slice(16,24)" :key="i">
      <span @click="handleChoice(item,i+16)" class="choice-text">{{item}}</span>
    </div>
  </div>
  <div style="width:100%;text-align:center">
    <van-button type="warning" size="small" style="margin: 20px auto" @click="clearGame">重置记录</van-button>
    <p style="color:#666;font-size:12px">copyright: <a href="https://www.52fansite.com" target="_blank">凡繁烦</a></p>
  </div>
  <van-popup v-model="showLevel" position="bottom" style="height:40%">
    <div class="levels">
      <div v-for="(item,i) in array" :key="i" :class="['level-item',id==item.id ? 'active':'',item.played?'played':'']" @click="handleClickLevel(item)">
        <p>第{{i+1}}关</p>
        <p>
          <van-rate :value="item.star" count="3" size="16px" color="#FFD21E" readonly/>
        </p>
      </div>
    </div>
  </van-popup>
  <div :class="['music-btn',isPlay == '0'? 'bgm':'']" @click="handleClickBgm()" ref="playbtn">
    <van-icon name="music-o" size="36px"/>
    <div v-if="isPlay=='1'" class="stop-play"></div>
  </div>
  <audio src="../assets/music/bgm.mp3" controls="true" ref="bgm" style="display:none" loop="loop"></audio>
  <audio src="../assets/music/error.wav" controls="true" ref="error" style="display:none" ></audio>
  </div>
</template>

<script>
import { Dialog,Toast } from 'vant';
export default {
  components: {
    [Dialog.Component.name]: Dialog.Component,
    [Toast.Component]: Toast.Component
  },
  data(){
    return {
      id: null,
      score: 100,
      data: require('../assets/word'),
      array: [],
      obj: {},
      choices: [],
      choiced: [],
      showPrev: false,
      showNext: false,
      showLevel: false,
      star: 0,
      time: 1,
      timer: null,
      prompt: 0,
      isPlay: "0",
    }
  },
  watch: {
    id(val) {
      if(val <= 10){
        this.step = 2;
      }else if(val > 10 && val < 20){
        this.step = 5;
      }
      localStorage.setItem('level',val);
      if(val >=2){
        this.showPrev = true;
      }else{
        this.showPrev = false;
      }
      let playedArr = this.array.filter(it=>{
        return it.id>this.id && it.played
      })
      if(playedArr.length > 0 || this.obj.played){
        this.showNext = true;
      }else{
        this.showNext = false;
      }
    },
    score(val){
      localStorage.setItem('score',val);
    },
    choiced(val){
      if(val.join('').length == 4){
        let wordArr = this.obj.word.split('');
        let doms = document.getElementsByClassName('choiced-text');
        let count = 0;
        wordArr.map((a,i) =>{
          if(a != val[i]){
            doms[i].style.color='red';
            count = count+1;
          }else{
            doms[i].style.color='#E3D479'
          }
        })
        if(count>0){
          this.$refs.error.play();
          if(this.isPlay=='0'){
            this.$refs.bgm.pause();
            this.$refs.error.addEventListener('ended',()=>{
              this.$refs.bgm.play();
            })
          }
        }
      }
    }
  },
  mounted(){
    if(localStorage.getItem('level')){
      this.id = parseInt(localStorage.getItem('level'));
    }
    if(localStorage.getItem('score')){
      this.score = parseInt(localStorage.getItem('score'));
    }
    if(localStorage.getItem('array')){
      this.array = JSON.parse(localStorage.getItem('array'))
    }else{
      this.array = this.data;
    }
    if(localStorage.getItem('bgm')){
      this.isPlay = localStorage.getItem('bgm')
    }
    this.jumpLevel(this.id);
    this.$refs.bgm.play();
  },
  methods: {
    jumpLevel(id){
      if(id != null){
        this.id = id;
      }else{
        if(this.id){
          this.id = this.id+1;
        }else{
          this.id = 1;
        }
      }
      this.time = 0;
      this.prompt = 0;
      this.obj = this.array[this.id-1];
      this.choiced = ['','','',''];
      this.getChoice();
      if(this.timer){
        window.clearInterval(this.timer);
      }
      this.timer = window.setInterval(()=>{
        let time = this.time+1;
        this.time = time;
      },1000)
    },
    getChoice(){
      let arr = this.obj.word.split('');
      console.log(arr);
      while(arr.length < 24){
        let index = Math.floor(Math.random()*(this.array.length));
        let wordArr = this.array[index].word.split('');
        let w = wordArr[Math.floor(Math.random()*4)];
        if(arr.indexOf(w) < 0){
          arr.push(w)
        }
      }
      this.choices = this.shuffle(arr);
    },
    shuffle(a){
      a.sort(function() {
          return Math.random() - 0.5 // 随机返回正或负值 达到排序的目的
      })
      return a;
    },
    //点击下面选项
    handleChoice(item,index){
      if(!item){
        return;
      }
      let result = this.choiced.join('');
      if(result.length == 4){
        return ;
      }else{
        for(let i = 0; i<this.choiced.length;i++){
          if(this.choiced[i] == ''){
            this.$set(this.choiced,i,item)
            this.$set(this.choices,index,'')
            event.target.setAttribute('text',item)
            break;
          }
        }
        result = this.choiced.join('');
        if(result == this.obj.word){
          
          Dialog.alert({
            title: this.id== this.array.length ? '恭喜您已通关':'恭喜您已过关',
            message: '[词义]:'+this.obj.mean+(this.obj.story ? ('\n\n[故事]:'+ this.obj.story):''),
            confirmButtonText: this.id== this.array.length ? "通关":"下一关"
          }).then(() => {
            window.clearInterval(this.timer);
            this.timer = null;
            if(this.time <= 10){
              this.star =3;
            }else if(this.time <= 20){
              this.star = 2
            }else if(this.time <= 30 ){
              this.star = 1;
            }else{
              this.star = 0;
            }
            let s = this.star-this.prompt < 0 ? 0 : this.star-this.prompt
            this.array.map(a =>{
              if(a.id==this.id){
                  a.played = true;
                  a.star = (a.star && s <= a.star) ? a.star : s;
              }
            })
            localStorage.setItem('array',JSON.stringify(this.array));
            this.jumpLevel(this.id == this.array.length? 1: null);
            this.score = this.score + this.step*(this.star == 0 ? 1: this.star);
          });
        }
      }
    },
    //点击上面系选择的
    handleCancel(index){
      if(this.choiced[index]){
        let text = this.choiced[index];
        this.$set(this.choiced,index,'')
        let doms = document.getElementsByClassName('choice-text');
        for(let i=0;i < doms.length;i++){
          let dom =doms[i];
          if(dom.getAttribute('text') && dom.getAttribute('text') == text){
            dom.removeAttribute('text')
            this.$set(this.choices,i,text);
          }
        }
      }
    },
    //选关
    checkLevel(){
      this.showLevel = true;
    },
    //bgm
    handleClickBgm(){
      if(this.isPlay=='0'){
        this.$refs.bgm.pause();
        this.$refs.playbtn.classList.remove('bgm')
        this.isPlay = '1';
      }else{
        this.$refs.bgm.play();
        this.$refs.playbtn.classList.add('bgm');
        this.isPlay = '0';
      }
      localStorage.setItem('bgm',this.isPlay);
    },
    clearGame(){
      Dialog.confirm({
      title: '提示',
      message: '确认清空游戏记录?\n清空后从第一关开始哟~',
    })
      .then(() => {
        this.array = this.data;
        this.score = 100;
        this.id = 1;
        localStorage.clear();
        this.jumpLevel(1);
        this.isPlay = '0';
        this.$refs.bgm.play();
      })
      .catch(() => {
        // on cancel
      });
    },
    //提示
    handlePrompt(){
      let score = this.score;
      if(score < 10){
        Toast('分数不够,继续闯关吧~');
      }else if(this.prompt >=3){
        Toast('已经提示了三次了哟，还不会吗~');
      }else{
        let doms = document.getElementsByClassName('choice-text');
        for(let i=0;i < doms.length;i++){
          let dom =doms[i];
          let arr = this.obj.word.split('');
          this.score = score - 10;
          this.prompt = this.prompt+1;
          arr.map(a =>{
            if(dom.innerHTML && dom.innerHTML == a){
              dom.style.color='red'
              window.setTimeout(function(){
                dom.style.color='#E3D479'
              },10000)
            }
          })
        }
      }
    },
    handleClickLevel(item){
      this.showLevel = false;
      if(item.id == this.id || !item.played){
        if(!item.played && item.id != this.id){
          Toast('通关后才可以挑战此关哟~');
        }
        return ;
      }
      this.jumpLevel(item.id)
    },
  }

}
</script>

<style lang="scss" scoped>
.container{
  background-color: #D7C097;
  height: 100vh;
}
.picture{
  margin: 20px auto;
  width: calc(100% - 100px);
  border: 1px ;
  border-radius: 8px;
  overflow: hidden;
  border: 1px solid #e0e0e0;
  background-color: #fff;

  .img{
    width: 100%;
    height: 160px;
  }
}
.tips{
  text-align: center;
}
.result{
  margin-top: 10px;
  padding: 10px 80px;
  display: flex;
  flex-flow: row;
  justify-content: space-between;
  div{
    width: 30px;
    height: 30px;
    border-radius: 4px;
    text-align: center;
    line-height: 30px;
    cursor: pointer;
    background-color: #643D12;
    color: #E3D479;
  }
}
.choice{
  display: flex;
  margin-top: 10px;
  justify-content: space-between;
  padding: 0 20px;

  div{
    width: 30px;
    height: 30px;
    border-radius: 5px;
    margin: 2px;
    text-align: center;
    line-height: 30px;
    cursor: pointer;
    background-color: #AD7F34;
    color: #E3D479;
  }
}
.music-btn{
  position: fixed;
  width: 36px;
  height: 36px;
  top: 50px;
  right: 5px;
  text-align: center;
  line-height: 34px;
  overflow: hidden;

  .stop-play{
    width: 28px;
    height: 0;
    border: 1px solid red;
    position: absolute;
    top: 50%;
    left: calc(50% - 14px);
    transform: rotate(-45deg);
  }
}
.bgm{
  animation: bgm 5s infinite linear;
}
@keyframes bgm {
  to {
    transform: rotate(0);
  }
  from {
    transform: rotate(360deg);
  }

}
.levels{
  display: flex;
  justify-content: space-between;
  flex-wrap: wrap;
  padding: 10px;

  .level-item{
    width: calc(25% - 10px);
    height: 80px;
    border: 1px solid #e0e0e0;
    margin-bottom: 10px;
    text-align: center;
    border-radius: 5px;

    &.active{
      background-color: rgba(153, 153, 153,.3);
    }
    &.played{
      background-color: rgba(158, 234, 106,.3);
    }
  }
}
</style>