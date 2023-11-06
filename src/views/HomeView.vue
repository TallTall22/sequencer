<script setup>
import { reactive, ref } from 'vue'
import * as Tone from 'tone'
import playSVG from '../assets/svg/play.svg'
import pauseSVG from '../assets/svg/pause.svg'
import randomSVG from '../assets/svg/random.svg'
import plusSVG from '../assets/svg/plus.svg'
import minusSVG from '../assets/svg/minus.svg'

const BPM=ref(120)
const isPlaying=ref(false)
const defaultSequencer =reactive({
    drum: {
      kick: new Array(16),
      hihat: new Array(16),
    }
  }) 


const kick = new Tone.MembraneSynth({
  octaves: 3,
  envelope: {
    sustain: 0.2,
  }
}).toDestination();

const hihat = new Tone.NoiseSynth({
  playbackRate: 5,
  envelope: {
    sustain: 0.0001,
  }
}).toDestination();


  const togglekick=(index)=>{
    defaultSequencer.drum.kick[index]= !defaultSequencer.drum.kick[index]
  }

  const toggleHihat = (index) => {
  defaultSequencer.drum.hihat[index] = !defaultSequencer.drum.hihat[index]
}

const getRandomArray=(length)=>{
  return new Array(length).fill(0).map(()=>Math.random()>0.80)
}

const getRandomMusic=()=>{
  defaultSequencer.drum={
    kick:getRandomArray(16),
    hihat:getRandomArray(16)
  }
}

const minusBPM=()=>{
  if(BPM.value<=60) return BPM.value=60
  BPM.value-=5
}

const plusBPM = () => {
  if (BPM.value >=180) return BPM.value = 180
  BPM.value += 5
}

const playMusic=()=>{
  Tone.start()
  isPlaying.value=!isPlaying.value
  if(isPlaying.value){
    Tone.Transport.clear()
    Tone.Transport.scheduleRepeat((time) => {
      let i = Math.round(Tone.Transport.getSecondsAtTime() * (BPM.value / 60) % 16)
      if (defaultSequencer.drum.kick[i]) kick.triggerAttackRelease("C2", "4n", time) 
      if (defaultSequencer.drum.hihat[i]) hihat.triggerAttackRelease("8n", time)
    }, "4n")
    Tone.Transport.start()
  }else{
    Tone.Transport.stop()
  }
  
}

</script>

<template>
  <div class="container">
    <div class="">
      <button @click="getRandomMusic">
        <img :src="randomSVG" alt="randomSVG">
      </button>
      <button @click="playMusic">
        <img v-if="!isPlaying" :src="playSVG" alt="playSVG">
        <img v-if="isPlaying" :src="pauseSVG" alt="pauseSVG">
      </button>
      <button @click="minusBPM">
        <img :src="minusSVG" alt="minusSVG">
      </button>
      <span>{{ BPM }}bpm</span>
      <button @click="plusBPM">
        <img :src="plusSVG" alt="plusSVG">
      </button>
    </div>
    <div class="kick">
      <div v-for="(active,index) in defaultSequencer.drum.kick" :key="index" :class="{ 'item': true, 'active': active }"
        @click="togglekick(index)">
      </div>
    </div>
    <div class="hihat">
      <div v-for="(active, index) in defaultSequencer.drum.hihat" :key="index" :class="{ 'item': true, 'active': active }"
      @click="toggleHihat(index)">
      </div>
    </div>
  </div>
  
</template>
<style lang="scss" scoped>
  .container{
    .kick,
    .hihat{
      height: 30vh;
      display: flex;
      justify-content:space-around ;
      align-items: center;
      .item{
        width: 100%;
        height: 100%;
        border: 1px solid #000;
        z-index: 10;
        &.active{
          background-color: #f8f300;
        }
      }
    }
  }
</style>
