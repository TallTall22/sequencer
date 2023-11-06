<script setup>
import { reactive, ref } from 'vue'
import * as Tone from 'tone'
import play from '../assets/svg/play.svg'

const BPM=ref(120)

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

const defaultSequencer =reactive({
    drum: {
      kick: new Array(16),
      hihat: new Array(16),
    }
  }) 

  const togglekick=(index)=>{
    defaultSequencer.drum.kick[index]= !defaultSequencer.drum.kick[index]
  }

  const toggleHihat = (index) => {
  defaultSequencer.drum.hihat[index] = !defaultSequencer.drum.hihat[index]
}

Tone.Transport.scheduleRepeat((time)=>{
  let i = Math.round(Tone.Transport.getSecondsAtTime() * (BPM.value / 60) % 16)  

  if(kick[i]) kick.triggerAttackRelease("C2", "4n" ,time)
  if(hihat[i]) hihat.triggerAttackRelease("8n", time)
},"4n")

</script>

<template>
  <div class="container">
    <div class="">
      <button>
        <img :src="play" alt="playSVG">
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
