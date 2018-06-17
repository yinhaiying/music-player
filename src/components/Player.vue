<template>
    <div class="hello">
      <h1>{{msg}}</h1>
      <audio id="audioId"  controls  :src="songUrl"></audio>
      <!--自定义按钮-->
      <div>
        <input type="button" value="上一首" @click="prevSong">
        <input type="button" value="播放" @click="playFn">
        <input type="button" value="暂停" @click="isPause">
        <input type="button" value="下一首" @click="nextSong">
      </div>
      <!--显示隐藏-->
      <div class="listTitle">
        <em @click="isHideFn">折叠</em>
        <i @click="isShowFn">展开</i>
      </div>
      <!--图片的旋转-->
      <div class="singerShow">
        <div class="imageRotate">
          <img :src="imgUrl" alt="">
        </div>
        <div class="sing">
          <span>歌名：{{musicname}}</span>
          <span>歌手：{{singer}}</span>
        </div>
      </div>
      <!--生成歌曲列表-->
      <transition name="fade">
        <div class="mp3list" ref="musicwrapper" v-show="showMusicList" >
          <ul  >
            <li v-for="(item,index) in musiclist " @click="switchSong(index)">
              <label for="">
                <img :src="item.img" alt="">
              </label>
              <span>{{item.name}}</span>
            </li>
          </ul>
        </div>
      </transition>



    </div>
</template>

<script>
  import axios from 'axios'
  // axios.default.baseURL='http://www.music/'

  // http://www.music/mp3.list
  // axios.get('mp3.list')



  import BScroll from 'better-scroll'
    export default {
        data(){
          return {
            msg:'音乐播放器',
            musiclist:'',
            musicname:'',
            singer:'',
            //歌曲索引
            songIndex:0,
            imgUrl:'',
            showMusicList:true,
            songUrl:''
          }
        },
      created(){
      //  可以在这里直接使用axios，也可以通过在这里调用methods中的方法
        this.getdata();

      },
      watch:{
          'songIndex':function(newValue,oldValue){
            this.playFn();
         }
      },
      methods:{
        //  通过ajax请求数据
        getdata() {
          // axios.get('http://localhost:233/mp3-list')
          axios.get('/apis/mp3-list')
            .then((res) => {
              console.log(res)
              this.musiclist = res.data.music;
              this.musicname = this.musiclist[this.songIndex].name;
              this.singer = this.musiclist[this.songIndex].singer;
              // console.log(this.musiclist)
              //设置默认播放歌曲
              this.songUrl = res.data.music[0].url;
              this.imgUrl = res.data.music[0].img;
              this.$nextTick(function () {
                // console.log(this.$refs.musicwrapper)
                let scroll = new BScroll(this.$refs.musicwrapper, {});
              })
            })
            .catch(function (error) {
              console.log(error)
            });
        //  添加请求拦截器
          axios.interceptors.request.use(function(config){
          //  在发送请求之前做些什么
            console.log(config)
            return config;
          },function(error){
          //  对请求错误做些什么
            return Promise.reject(error)
          })
        },
        //  播放方法
        playFn(){
          // this.$nextTick(function(){
          //   console.log(this.$refs.musicwrapper)
          //   let scroll = new BScroll(this.$refs.musicwrapper,{});
          // })
          //_是用来表示是局部变量
          let _audio=document.getElementById('audioId');
          //  改变audio的src属性实现歌曲的切换
        _audio.setAttribute('src',this.musiclist[this.songIndex].url)
          // console.log(this.musiclist[index])
         //  console.log(_audio.readyState)
         // if(_audio.readyState===4){
         //   _audio.play();
         // }
          _audio.play();
        },
        prevSong(){

          this.songIndex--;
          if(this.songIndex<0){
            this.songIndex=this.musiclist.length-1;
          }
        },
        nextSong(){
          this.songIndex++;
          if(this.songIndex===this.musiclist.length){
            this.songIndex=0;
          }
        },
        //暂停方法
        isPause(){
          let _audio=document.getElementById('audioId');
          _audio.pause();
        },
        //切换歌曲点击时调用当前播放的方法，同时还需要切换歌曲
        switchSong(index){
          //必须传入index因为在进行歌曲切换的时候，上一首和下一首是通过index来确认的
          //  在这里改变index，然后通过watch来监听index如果index发生变化则进行歌曲切换
          if(this.songIndex===index){
            this.playFn();
          }else{
            this.songIndex=index;
            this.imgUrl=this.musiclist[this.songIndex].img;
          }
          console.log('执行了切换方法')
        },

        // 显示列表
        isShowFn(){
          this.showMusicList = true;
        },
        // 隐藏列表
        isHideFn(){
          this.showMusicList = false;
        }
      }
    }
</script>

<style scoped>
  *{margin:0;padding:0;border:0;}
  .hello{
    width:400px;
    margin:100px auto;
  }
  h1, h2 {
    font-weight: normal;
  }
  ul {
    list-style-type: none;
    padding: 0;
  }
  li {
    display: inline-block;
    margin: 0 10px;
  }
  a {
    color: #42b983;
  }

  h1.listTitle{
    display: block;height: 50px;position: relative;
    width:80%;margin:0 auto;font-size: 14px;
  }
  /*显示按钮*/
  h1.listTitle i{
    display: block;position: absolute;top: 0;right: 0;
    cursor: pointer;font-style: normal;border:1px solid #000;
    width: 30px;height: 30px;border-radius: 100%;
    line-height: 30px;
  }
  .singerShow{
    width:200px;
    height:200px;
    margin:auto;
  }
  .singerShow .sing{

  }
  .singerShow .sing span{
    display:block;
    margin-top:15px;
    color:#aaa;

  }
  .singerShow .imageRotate{
    width:50%;
    height:50%;
  }
  .singerShow .imageRotate img{
    width:100%;
    height:100%;
    border-radius:50%;
    animation:rotateAnimation 10s linear infinite
  }
  @keyframes rotateAnimation {
    0% {transform: rotate(0);}
    100% {transform: rotate(360deg);}
  }
  /*隐藏按钮*/
  h1.listTitle em{
    display: block;position: absolute;top: 0;left: 0;
    cursor: pointer;font-style: normal;border:1px solid #000;
    width: 30px;height: 30px;border-radius: 100%;
    line-height: 30px;
  }
  div.btnWrap{
    width: 95%;margin:10px auto;height: 40px;
    border:1px solid #000;padding:10px 0 0 0;
  }
  div.btnWrap input{
    margin:0 10px;
  }

  .mp3list{
    /*display: none;*/
    height:400px;
    overflow:hidden;
    border:1px solid green;
  }
  .mp3list ul{
    border:1px solid red;
  }
  .mp3list li{overflow: hidden;
    display: block;clear: both;background: #ddd;margin:10px;
  }
  .mp3list li:hover{
    background: #eee;
  }
  .mp3list li label{float: left;}
  .mp3list li label img{width: 50px;height:50px;margin:10px;}
  .mp3list li span{
    float: left;font-size: 22px;line-height: 70px;
  }

  input{padding:10px;margin:10px;}
  input:hover{background: #eee;}
  /**/
  .mp3Title{
    position: relative;width: 80%;height: 60px;
    padding-top: 120px;
  }
  .mp3Title label,
  .mp3Title span{
    display: block;
  }
  .mp3Title label img{
    display: block;width:100px;height: 100px;
    border-radius: 100%;margin:0 auto;
    border:5px solid #666;
    position: absolute;top: 0;left: 50%;
    animation: imgGun 10s linear infinite;
  }
  @keyframes imgGun{
    from {transform: rotate(0);}
    to {transform: rotate(360deg);}
  }
  .fade-enter-active, .fade-leave-active{
    transition: all 0.5s ease;
  }
  .fade-enter, .fade-leave-active{
    transform:translateY(-20px);
    opacity: 0;
  }
</style>
