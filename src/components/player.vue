<template>
  <div class="player">
    <!-- 多媒体控件 -->
    <div class="player-controll">
      <div class="play-audio">
        <audio :src="url" controls=""  autoplay="" ref="player" preload="true" @timeupdate='playerTimeUpdate' @play="playerPlay" @ended="nextPlay"></audio>
      </div>
      <div class="timeLine"  v-bind:style="{width:timeLine+'%'}"></div>
      <!-- 音乐信息 -->
      <div class="player-info" @click="showDetails">
        <div class="songsImg">
          <img :src="songsInfo.al.picUrl" alt="">
        </div>
        <div class="songsTitle">
          <p><span v-text="songsInfo.name"></span>-<span v-text="songsInfo.ar[0].name"></span></p>
          <p v-text="songsInfo.al.name"></p>
        </div>
      </div>
      <!-- 播放控件 -->
      <div class="player-bar">
        <div class="player-icon listIcon" @click='showList'>
          <img src="../../static/img/menu.png" alt="">
        </div>
        <div class="player-icon play" @click="playerControll">
          <img :src="playerIcon" alt="">
        </div>
        <div class="player-icon next" @click="nextPlay">
          <img src="../../static/img/next.png" alt="">
        </div>
      </div>
    </div>

    <!-- 播放详情 -->
    <transition name="slide">
      <div class="player-details" v-if="details">
        <div class="player-details-top" :style="topStyle">
          <div class="player-details-wrap">
            <div class="backs" @click="back"> 
              <img src="../../static/img/down.png" alt="">
            </div>
            <h6 class="player-details-name" v-text="songsInfo.name"></h6>
            <p class="player-details-ar" v-text="songsInfo.ar[0].name"></p>
            <!-- <div class="player-details-img">
              <img :src="songsInfo.al.picUrl" alt="">
            </div> -->
            <div class="showLrc">
              <ul v-html="Lrcs" class="lrc" v-bind:style="{top:LrcTop}"> 
              </ul>
            </div>
          </div>
           <!-- v-bind:style="{transform:'rotate('+rotate+'deg)'}" -->
          
        </div>
        <div class="player-details-bottom">
          <div class="player-details-line">
            <i class="showTime TimeStart" v-text="startTime"></i>
            <mu-slider v-model="value" class="slider" @change="changeTime"/>
            <i class="showTime TimeEnd" v-text="endTime"></i>
          </div>
          <div class="player-details-bar">
            <div class="player-details-icon details-prev" @click="prePlay">
              <img src="../../static/img/prev.png" alt="">
            </div>
            <div class="player-details-icon details-play" @click="playerControll">
              <img :src="playerIcon" alt="">
            </div>
            <div class="player-details-icon details-next" @click="nextPlay">
              <img src="../../static/img/next.png" alt="">
            </div>
          </div>
          <div class="player-details-other">
            <div class="player-details-mode">
              <img src="../../static/img/mode.png" alt="">
            </div>
            <div class="player-details-dis" @click="getDiscuss(activeId)">
                <img src="../../static/img/dis.png" alt="">
            </div>
          </div>
        </div>
      </div>
    </transition> 
    <!-- 播放列表 -->
    <transition name="list-top">
      <div class="player-list-top" v-if="list" @click="showList"></div>
    </transition>
    <transition name="list-bot">
      <div class="player-list-bottom" v-if="list">
        <div class="play-list-title">
          播放列表
        </div>
        <ul  class="play-list-ul">
          <li class="play-list-li" v-for="(item,index) in playList" :key="item.id"  :style="{color:item.id==activeId?'#fff':'#f4ea2a'}">
            <i class="play-list-index">{{index+1}}</i>
            <p class="play-list-name" @click="setUrl(item.id)">
              <span  v-text="item.name"></span>-<span class="play-list-singer"  v-text="item.ar[0].name" :style="{color:item.id==activeId?'#fff':'#f4ea2a'}"></span>
            </p>
            <span class="delete" @click="pop(index,item.id)">X</span>
          </li>
        </ul>
      </div>
    </transition>
    <!-- 评论 -->
    <transition name="discuss">
    <div class="discuss" v-if="discussState">
      <div class="discuss-title">
        <div class="backs" @click="backDiscuss"> 
          <img src="../../static/img/back.png" alt="">
        </div>
        评论({{discuss.total}})
      </div>
      <div class="discuss-content">
        <div class="discuss-content-title">精彩评论</div>
        <ul class="discuss-ul">
          <li class="discuss-li" v-for="(item,index) in discuss.hotComments" :key="index">
            <div class="discuss-li-title">
              <div class="discuss-li-img">
                <img :src="item.user.avatarUrl" alt="">
              </div>
              <div class="discuss-li-name">
                <p class="discuss-li-t" v-text="item.user.nickname"></p>
                <p class="discuss-li-p" v-text="new Date(item.time).toLocaleDateString().substr(0,10).replace(/\//g,'-')"></p>
              </div>
            </div>
            <div class="discuss-li-info" v-text="item.content"></div>
          </li>
          <div class="discuss-content-title">最新评论</div>
          <li class="discuss-li" v-for="(item,index) in discuss.comments" :key="index+1000">
            <div class="discuss-li-title">
              <div class="discuss-li-img">
                <img :src="item.user.avatarUrl" alt="">
              </div>
              <div class="discuss-li-name">
                <p class="discuss-li-t" v-text="item.user.nickname"></p>
                <p class="discuss-li-p" v-text="new Date(item.time).toLocaleDateString().substr(0,10).replace(/\//g,'-')"></p>
              </div>
            </div>
            <div class="discuss-li-info" v-text="item.content"></div>
          </li>
        </ul>
      </div>
    </div>
    </transition>  
  </div>
</template>                                                                                                                                                                                                                                                                                                                                                              

<script>
import { mapMutations, mapGetters } from 'vuex'
export default {
  data(){
    return{
      value:0,
      details:false,
      list:false,
      discussState:false,
      discuss:'',
      player:'',
      playerState: '',
      timeLine: 0,
      rotate: 0,
      startTime:0,
      endTime:0,
      LrcTop:120+'px',
      LrcIndex:0,
      playerIcon:'../../static/img/play.png',
      topStyle:{
        backgroundImage: '',
        backgroundRepeat: "no-repeat",
        backgroundSize: "100% 100%",
        backgroundPosition: "center center"
      }
    }
  },
  computed: {
    ...mapGetters([
      'url',
      'songsInfo',
      "Lrc",
      "Lrcs",
      'playList',
      'activeId'
    ]),
  },
  watch:{
    timeLine(n,o){
      this.value=n;
    },
    songsInfo(n,o){

    }
  },
  methods:{
    back(){
      this.details=!this.details;
    },
    backDiscuss(){
      this.discussState=!this.discussState;
      //this.details=true;
    },
    playerControll(){
      if(this.player.paused){
        this.player.play();
        this.playerIcon='../../static/img/pause.png'
      }else{
        this.player.pause();
        this.playerIcon='../../static/img/play.png'
      }
    },
    playerPlay(){
      this.playerIcon='../../static/img/pause.png';
      this.topStyle.backgroundImage='url('+this.songsInfo.al.picUrl+')'
    },
    playerTimeUpdate(){
      var currentTime=parseInt(this.player.currentTime);
      var duration=parseInt(this.player.duration);
      var timeLine=(currentTime/duration)*100;
      this.startTime=this.formatSeconds(currentTime);
      this.endTime=this.formatSeconds(duration);
      this.timeLine=timeLine;
      if(this.details&&this.Lrc.length!=0){
        for(var i=0;i<this.Lrc.length;i++){
          if(this.player.currentTime>this.Lrc[i][0]){
            this.LrcTop=-i*40+90+'px';
            for(var j=0;j<this.Lrc.length;j++){
              document.querySelectorAll('.lrc li')[j].style.color="#f4ea2a"
            }
            if(i>=1){
              document.querySelectorAll('.lrc li')[i].style.color="#fff";
            }
          }
        }
      }
    },
    setUrl(id){
      var _this=this;
      //获取歌曲地址
      this.$store.commit('setLrc','');
      this.$store.commit('setActiveId',id);
      this.$http.get(this.config.api+'/music/url',{
        params:{
          id:id,
          br:'320000'
        }
      }).then(function(res){
        if(res.data.code==200){
          _this.$store.commit('setUrl',res.data.data[0].url);
        };
      })
      //获取歌词
      this.$http.get(this.config.api+'/lyric',{
        params:{
          id:id
        }
      }).then(function(res){
        if(res.data.code==200){
          var str=_this.parseLyric(res.data.lrc.lyric);
          _this.$store.commit('setLrc',str); 
        };
      })
      //获取歌曲信息
      this.$http.get(this.config.api+'/song/detail',{
        params:{
          ids:id
        }
      }).then(function(res){
        if(res.data.code==200){
          _this.$store.commit('setSongsInfo',res.data.songs[0])
        };
      })
    },
    showDetails(){
      this.details=!this.details;
      this.list=false;
    },
    showList(){
      this.list=!this.list;
    },
    changeTime(){
      var time=(this.value*this.player.duration)/100;
      this.player.currentTime=time;
    },
    prePlay(){
      this.$store.dispatch("pre");
    },
    nextPlay(){
      this.$store.dispatch("next");
    },
    pop(index,id){
      this.$store.commit('removePlayList',index);
      console.log(id,this.activeId)
      if(this.playList.length==0){
        this.playerControll();
        this.timeLine=0;
        this.$store.commit('setActiveId',0);
      }else if(id==this.activeId){
        this.nextPlay()
      } 
    },      
    getDiscuss(id){
      //获取评论地址
      this.discuss='';
      this.backDiscuss();
     // this.details=false;
      var _this=this;
      this.$http.get(this.config.api+'/comment/music',{
        params:{
          id:id
        }
      }).then(function(res){
        if(res.data.code==200){
          _this.discuss=res.data;
        };
      })
    },
  },
  mounted(){
    this.player=this.$refs.player;
    this.topStyle.backgroundImage='url('+this.songsInfo.al.picUrl+')'
  }
}
</script>

<style>
.player{
  width: 100%;
  height: 1rem;
  background:rgba(0, 0, 0, 1);
  position: fixed;
  bottom: 0;
  left: 0;
  z-index: 100;
}
.player-controll{
  width: 100%;
  height: 1rem;
  background:rgba(0, 0, 0, 1);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 100;
  position: fixed;
  bottom: 0;
  left: 0;
}
.play-audio{
  display: none;
}
.timeLine{
  position: absolute;
  bottom: 0;
  left: 0;
  width: 0%;
  height: 0.03rem;
  background: #f4ea2a;
}
.player-info{
  width: 5rem;
}
.songsImg{
  width: 0.8rem;
  height: 0.8rem;
  float: left;
  margin: 0.1rem;
}
.songsTitle{
  width: 3.8rem;
  height: 1rem;
  float: left;
}
.songsTitle p{
  color: #f4ea2a;
  width: 100%;
  height: 0.5rem;
  line-height: 0.5rem;
  text-indent: 0.2rem;
  font-size: 12px;
  font-family: 'microsoft yahei';
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
.player-bar{
  width: 2.5rem;
  height: 100%;
  display: flex;
  justify-content: space-around;
  align-items: center;
}
.player-icon{
  width: 0.6rem;
  height: 0.6rem;
}
.listIcon{
  width: 0.4rem;
  height: 0.43rem;
}
.next{
  width: 0.3rem;
  height: 0.3rem;
}
.player-details{
  width: 100%;
  height: 100%;
  position: fixed;
  left: 0;
  top: 0;
  background: rgba(0, 0, 0, 1);
  z-index: 110;
}
.player-details-top{
  width: 100%;
  height: 58%;
  min-height: 6rem;
  color: #f4ea2a;
  text-align: center;
  display: flex;
  flex-wrap: wrap;
  justify-content: center;
  align-items: flex-start;
}
.player-details-wrap{
  width: 100%;
  height: 58%;
  min-height: 6rem;
  background: rgba(20, 20, 20, 0.6);
  position: absolute;
  top: 0;
  left: 0;
}
.showLrc{
  width:100%;
  height:50%;
  margin:10% auto;
  overflow:hidden;
  color:#f4ea2a;
  position:relative;
}
.lrc{
  width:100%;
  position:absolute;
  top:200px;
}
.lrc li{
  font-size: 10px;
  height: 40px;
  line-height: 40px;
  font-family: 'microsoft yahei';
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
}
.player-details-name{
  width: 70%;
  height: 1rem;
  line-height: 0.5rem;
  margin: 0 auto;
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: normal;
  margin-top: 0.2rem;
}
.player-details-ar{
  height: 1rem;
  line-height: 1rem;
  width: 100%;
}
.player-details-img{
  width: 4rem;
  height: 4rem;
}
.player-details-img img{
  border-radius: 2rem; 
}
.player-details-bottom{
  width: 100%;
  height: 42%;
  color: #f4ea2a;
  text-align: center;
}
.player-details-line{
  width: 90%;
  display: flex;
  justify-content: space-around;
  align-items: center;
  margin: 0.5rem auto;
}
.showTime{
  float: left;
}
.slider{
  width: 60%;
  float: left;
  margin-bottom: 0; 
}
.mu-slider-fill,.mu-slider-thumb{
  color: #f4ea2a;
  background-color: #f4ea2a;
}
.player-details-bar{
  width: 100%;
  height: 2rem;
  display: flex;
  justify-content: space-around;
  align-items: center;
}
.player-details-icon{
  width: 1rem;
  height: 1rem;
}
.details-prev{
  width: 0.7rem;
  height: 0.7rem;
}
.details-next{
  width: 0.6rem;
  height: 0.6rem;
}
.player-details-other{
  width: 100%;
  height: 2rem;
  display: flex;
  justify-content: space-around;
  align-items: center;
}
.player-details-mode,
.player-details-dis{
  width: 0.4rem;
  height: 0.4rem;
}
.player-list{
  width: 100%;
  height: 100%;
  position: fixed;
  left: 0;
  top: 0;
  z-index: -1;
}
.player-list-top{
  width: 100%;
  height: 50%;
  background: rgba(0, 0, 0, 0.5);
  position: fixed;
  left: 0;
  top: 0;
  z-index: 99;
}
.player-list-bottom{
  width: 100%;
  height: 50%;
  background: rgba(0, 0, 0, 1);
  position: fixed;
  left: 0;
  top: 50%;
  z-index: 99;
  color: #f4ea2a;
}
.play-list-title{
  width: 100%;
  height: 10%;
  line-height: 200%;
  text-align: center;
  /* border-bottom: 0.01rem solid rgba(244,234,42,0.5); */
}
.play-list-ul{
  width: 100%;
  height: 5rem;
  color: #f4ea2a;
  overflow-y: scroll;
}
.play-list-li{
  width: 100%;
  height: 0.7rem;
  float: left;
  position: relative;
}
.play-list-index{
  float: left;
  width: 0.5rem;
  height: 0.7rem;
  line-height: 0.7rem;
  text-align: center
}
.play-list-name{
  float: left;
  width: 6.5rem;
  height: 0.7rem;
  line-height: 0.7rem;
  /* border-bottom: 1px solid rgba(244,234,42,0.5); */
  text-overflow: ellipsis;
  overflow: hidden;
  white-space: nowrap;
  font-family: 'microsoft yahei' 
}
.play-list-singer{
  font-size: 10px;
  color: rgba(244,234,42,0.5);
}
.delete{
  display: inline-block;
  width: 0.5rem;
  height: 0.7rem;
  line-height: 0.7rem;
  text-align: center;
  position: absolute;
  right: 0;
  top: 0;
}

.discuss{
  width: 100%;
  height: 100%;
  background: #fff;
  position: fixed;
  top: 0;
  left:0;
  z-index: 120;
  overflow: scroll;
}
.discuss-li{
  border-bottom: 1px solid #eee;
}
.discuss-title{
  width: 100%;
  height: 1rem;
  line-height: 1rem;
  background: rgba(0, 0, 0, 1);
  color: #f4ea2a;
  text-align: center;
  font-size: 15px;
  font-family: 'microsoft yahei';
  position: fixed;
  top: 0;
  left: 0;
}
.discuss-content{
  width: 100%;
  padding-top: 1rem;
}
.discuss-content-title{
  width: 100%;
  height: 0.5rem;
  line-height: 0.5rem;
  text-indent: 0.1rem;
  background: #eee;
  color: #666;
}
.discuss-li-title{
  width: 100%;
  height: 0.8rem;
  margin-top: 0.2rem; 
}
.discuss-li-img{
  width: 0.8rem;
  height: 0.8rem;
  float: left;
  margin: 0 0.1rem;
}
.discuss-li-img img{
  border-radius:0.4rem; 
}
.discuss-li-name p{
  
}
.discuss-li-info{
  width: 100%;
  box-sizing: border-box;
  padding: 0.2rem 0.1rem 0.2rem 1rem;
  text-align: justify;
}




.slide-enter,.slide-leave-to{
  opacity: 0;
  top: 100%;
}
.slide-enter-active,.slide-leave-active{
  transition: all 0.6s;
}
.slide-enter-to,.slide-leave{
  opacity: 1;
  top: 0;
}

.list-top-enter,.list-top-leave-to{
  opacity: 0;
  /* top: -50%; */
}
.list-top-enter-active{
  transition: all 0.5s ease 0.4s;
}
.list-top-leave-active{
  transition: all 0.5s 
}
.list-top-enter-to,.list-top-leave{
  opacity: 1;
  /* top: 0%; */
}


.list-bot-enter,.list-bot-leave-to{
  opacity: 0;
  top: 100%;
}
.list-bot-enter-active{
  transition: all 0.5s;
}
.list-bot-leave-active{
  transition: all 0.5s ease 0.2s;
}
.list-bot-enter-to,.list-bot-leave{
  opacity: 1;
  top: 50%;
}

.discuss-enter,.discuss-leave-to{
  opacity: 0;
  left: 100%;
}
.discuss-enter-active,.discuss-leave-active{
  transition: all 0.3s;
}
.discuss-enter-to,.discuss-leave{
  opacity: 1;
  left: 0;
}


</style>
