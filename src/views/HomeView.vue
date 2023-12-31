<script setup>
import { reactive, ref } from 'vue'
import * as Tone from 'tone'
import playSVG from '../assets/svg/play.svg'
import pauseSVG from '../assets/svg/pause.svg'
import randomSVG from '../assets/svg/random.svg'
import plusSVG from '../assets/svg/plus.svg'
import minusSVG from '../assets/svg/minus.svg'
import deleteSVG from '../assets/svg/delete.svg'

const BPM = ref(120) // 定義BPM（每分鐘拍數）的變數，初始值為120
const isPlaying = ref(false) // 定義isPlaying變數，用於控制播放狀態，初始值為false
const currentIndex = ref(0) // 定義currentIndex變數，用於追蹤當前節拍的索引，初始值為0

const defaultSequencer = reactive({
  drum: {
    kick: new Array(16),
    hihat: new Array(16),
    snare: new Array(16),
    tomL: new Array(16),
    tomH: new Array(16),
    triangleBell: new Array(16),
  }
}) // 定義defaultSequencer，包含各種打擊樂器的節拍序列，初始為空序列

// 定義不同樂器的聲音合成器
const kick = new Tone.MembraneSynth({
  octaves: 3,
  envelope: {
    sustain: 0.2,
  },
}).toDestination();

const hihat = new Tone.NoiseSynth({
  playbackRate: 5,
  envelope: {
    sustain: 0.0001,
  },
}).toDestination();

const lowPass = new Tone.Filter({
  frequency: 11000,
  type: 'lowpass',
}).toDestination();

const noise = new Tone.NoiseSynth({
  noise: {
    type: 'pink',
    playbackRate: 3,
  },
  envelope: {
    attack: 0.001,
    decay: 0.15,
    sustain: 0.0001,
    release: 0.05,
  },
}).connect(lowPass);

const poly = new Tone.PolySynth({
  maxPolyphony: 20,
  oscillator: {
    partials: [0, 2, 3, 4],
  },
  envelope: {
    attack: 0.001,
    decay: 0.17,
    sustain: 0.0001,
    release: 0.08,
    releaseCurve: 'exponential',
  },
}).toDestination();

const note = ['C2', 'D#2', 'G2'];
const trigger = (time) => {
  noise.triggerAttack(time);
  note.forEach((note) => {
    poly.triggerAttackRelease(note, '16n', time);
  });
};

const tomL = new Tone.MembraneSynth({
  pitchDecay: 0.003,
  octaves: 0.5,
  envelope: {
    attack: 0.001,
    decay: 0.1,
    sustain: 0.0001,
    release: 0.1,
  },
}).toDestination();

const tomH = new Tone.MembraneSynth({
  pitchDecay: 0.008,
  octaves: 8,
  envelope: {
    attack: 0.001,
    decay: 0.1,
    sustain: 0.0001,
    release: 0.1,
  },
}).toDestination();

const triangleBell = new Tone.MetalSynth({
  frequency: 700,
  envelope: {
    attack: 0.001,
    decay: 0.1,
    release: 0.5,
  },
  harmonicity: 6,
  modulationIndex: 15,
}).toDestination();

// 定義控制節拍序列的函數
const togglekick = (index) => {
  defaultSequencer.drum.kick[index] = !defaultSequencer.drum.kick[index];
};

const toggleHihat = (index) => {
  defaultSequencer.drum.hihat[index] = !defaultSequencer.drum.hihat[index];
};

const toggleSnare = (index) => {
  defaultSequencer.drum.snare[index] = !defaultSequencer.drum.snare[index];
};

const toggleTomL = (index) => {
  defaultSequencer.drum.tomL[index] = !defaultSequencer.drum.tomL[index];
};

const toggleTriangleBell = (index) => {
  defaultSequencer.drum.triangleBell[index] = !defaultSequencer.drum.triangleBell[index];
};

const toggleTomH = (index) => {
  defaultSequencer.drum.tomH[index] = !defaultSequencer.drum.tomH[index];
};

// 生成隨機節拍序列
const getRandomArray = (length) => {
  return new Array(length).fill(0).map(() => Math.random() > 0.70);
};

const getRandomMusic = () => {
  defaultSequencer.drum = {
    kick: getRandomArray(16),
    hihat: getRandomArray(16),
    snare: getRandomArray(16),
    tomL: getRandomArray(16),
    tomH: getRandomArray(16),
    triangleBell: getRandomArray(16),
  };
};

// 重置節拍序列
const getResetMusic = () => {
  defaultSequencer.drum = {
    kick: new Array(16),
    hihat: new Array(16),
    snare: new Array(16),
    tomL: new Array(16),
    tomH: new Array(16),
    triangleBell: new Array(16),
  };
};

// 調整BPM（每分鐘拍數）
const minusBPM = () => {
  if (BPM.value <= 60) return (BPM.value = 60);
  BPM.value -= 5;
};

const plusBPM = () => {
  if (BPM.value >= 180) return (BPM.value = 180);
  BPM.value += 5;
};

// 播放音樂
const playMusic = () => {
  Tone.start();
  isPlaying.value = !isPlaying.value;
  if (isPlaying.value) {
    Tone.Transport.clear();
    Tone.Transport.scheduleRepeat((time) => {
      Tone.Transport.bpm.value = BPM.value;
      let i = Math.round(Tone.Transport.getSecondsAtTime() * (BPM.value / 60) * 2) % 16;
      let index = i;
      index = ++index % 16;
      i = i % 16;
      currentIndex.value = index;
      if (defaultSequencer.drum.kick[index]) kick.triggerAttackRelease("C2", "8n", time);
      if (defaultSequencer.drum.hihat[index]) hihat.triggerAttackRelease("16n", time);
      if (defaultSequencer.drum.snare[index]) trigger(time);
      if (defaultSequencer.drum.tomL[index]) tomL.triggerAttackRelease("G2", "8n", time);
      if (defaultSequencer.drum.tomH[index]) tomH.triggerAttackRelease("G2", "8n", time);
      if (defaultSequencer.drum.triangleBell[index]) triangleBell.triggerAttackRelease("C2", "8n", time);
    }, "8n");
    Tone.Transport.start();
  } else {
    Tone.Transport.stop();
    currentIndex.value = 0;
  }
};
</script>

<template>
  <div class="container">
    <div class="button-wrapper">
      <button @click="getRandomMusic">
        <img :src="randomSVG" alt="randomSVG">
      </button>
      <button @click="playMusic">
        <img v-if="!isPlaying" :src="playSVG" alt="playSVG">
        <img v-if="isPlaying" :src="pauseSVG" alt="pauseSVG">
      </button>
      <div class="bpm-group">
        <button @click="minusBPM">
          <img :src="minusSVG" alt="minusSVG">
        </button>
        <span>{{ BPM }}bpm</span>
        <button @click="plusBPM">
          <img :src="plusSVG" alt="plusSVG">
        </button>
      </div>
      <button @click="getResetMusic">
        <img :src="deleteSVG" alt="deleteSVG">
      </button>
    </div>
    <div class="music-wrapper">
      <div class="timeLine">
        <div v-for="i in 16" :key="`index_${i}`" :class="{ 'time': true, 'active': currentIndex === i - 1 }">
        </div>
      </div>
      <ul>
        <li class="kick">
          <div v-for="(active, index) in defaultSequencer.drum.kick" :key="index"
            :class="{ 'item': true, 'active': active }" @click="togglekick(index)">
          </div>
        </li>
        <li class="hihat">
          <div v-for="(active, index) in defaultSequencer.drum.hihat" :key="index"
            :class="{ 'item': true, 'active': active }" @click="toggleHihat(index)">
          </div>
        </li>
        <li class="snare">
          <div v-for="(active, index) in defaultSequencer.drum.snare" :key="index"
            :class="{ 'item': true, 'active': active }" @click="toggleSnare(index)">
          </div>
        </li>
        <li class="tomL">
          <div v-for="(active, index) in defaultSequencer.drum.tomL" :key="index"
            :class="{ 'item': true, 'active': active }" @click="toggleTomL(index)">
          </div>
        </li>
        <li class="tomH">
          <div v-for="(active, index) in defaultSequencer.drum.tomH" :key="index"
            :class="{ 'item': true, 'active': active }" @click="toggleTomH(index)">
          </div>
        </li>
        <li class="triangleBell">
          <div v-for="(active, index) in defaultSequencer.drum.triangleBell" :key="index"
            :class="{ 'item': true, 'active': active }" @click="toggleTriangleBell(index)">
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<style lang="scss" scoped>
  .container{
    width: 100%;
    height: 100%;
    .button-wrapper{
      width: 100%;
      height: auto;
      display: flex;
      justify-content: space-around;
      align-items: center;
      background-color: #232222;
      button{
        width: 3.5rem;
        height: 3.5rem;
        margin: 1rem;
        img{
          width: 2rem;
          height: 2rem;
        }
      }
      .bpm-group{
        display: flex;
        align-items: center;
        span{
          font-size: 1.5rem;
          color: #fff;
        }
      }
    }
    .music-wrapper{
      width: 100%;
      height: auto;
      background-color: #232222;
      .timeLine{
      height: 3vh;
      display: flex;
      justify-content:space-around ;
      align-items: center;
      .time{
        width: 100%;
        height: 30%;
        margin: 0.2rem;
        border-radius: 0.5rem;
        background-color: #160076;
        &.active{
          background-color: #09b4de;
        }
      }
    }
    ul {
    li {
      width: 100%;
      height: auto; 
      display: flex;
      justify-content: space-around;

      .item {
        width:  calc(100% / 16);
        padding-top: calc(100% / 16); 
        border: 2px solid #232222;
        border-radius: 0.2rem;
        background-color: #3e3e3e;
        z-index: 10;


        &.active {
          background-color: #ba00f8;
        }
      }
    }
  }
    }
    
  }

  @media screen and (max-width:864px){
    .container{
      .button-wrapper{
        button{
          width: 2rem;
          height: 2rem;
          margin: 0.5rem;
          img{
            width:1rem ;
            height: 1rem;
          }
        }
        .bpm-group{
          span{
            font-size: 1rem;
          }
        }
      }
    }
  }
</style>
