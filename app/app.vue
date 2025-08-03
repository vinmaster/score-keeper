<template>
  <div class="score-keeper">
    <div class="teams">
      <div class="team blue">
        <h2>Team Blue</h2>
        <div class="score">{{ scoreBlue }}</div>
        <div class="controls">
          <button class="u-button" @click="addPoint('Blue')">
            <Icon name="i-heroicons-plus" class="icon" />
          </button>
          <button class="u-button" @click="subtractPoint('Blue')">
            <Icon name="i-heroicons-minus" class="icon" />
          </button>
        </div>
      </div>
      <div class="team red">
        <h2>Team Red</h2>
        <div class="score">{{ scoreRed }}</div>
        <div class="controls">
          <button class="u-button" @click="addPoint('Red')">
            <Icon name="i-heroicons-plus" class="icon" />
          </button>
          <button class="u-button" @click="subtractPoint('Red')">
            <Icon name="i-heroicons-minus" class="icon" />
          </button>
        </div>
      </div>
    </div>
    <div class="voice-controls">
      <button class="u-button" @click="startListening">
        <Icon name="i-heroicons-microphone" class="icon" />
        Voice
      </button>
      <span v-if="listening">Listening...</span>
      <button class="u-button" @click="stopListening" v-if="listening">
        Stop
      </button>
    </div>
  </div>
</template>

<script setup lang="ts">
import { ref, stop } from 'vue'
const scoreBlue = ref(0)
const scoreRed = ref(0)

function addPoint(team: 'Blue' | 'Red') {
  if (team === 'Blue') scoreBlue.value++
  else scoreRed.value++
}

function subtractPoint(team: 'Blue' | 'Red') {
  if (team === 'Blue' && scoreBlue.value > 0) scoreBlue.value--
  else if (team === 'Red' && scoreRed.value > 0) scoreRed.value--
}

const listening = ref(false)
let recognition: any = null

function startListening() {
  if (!('webkitSpeechRecognition' in window || 'SpeechRecognition' in window)) {
    alert('Web Speech API not supported')
    return
  }
  if (recognition && recognition.recognizing) {
    recognition.stop()
    listening.value = false
    return
  }

  const words = [
    'team blue',
    'team red',
    'add blue',
    'subtract blue',
    'add red',
    'subtract red'
  ]
  const SpeechGrammarList = (window as any).SpeechGrammarList || (window as any).webkitSpeechGrammarList
  const grammar = new SpeechGrammarList()
  grammar.addFromString('#JSGF V1.0; grammar commands; public <command> = ' + words.join(' | ') + ' ;', 1)

  const SpeechRecognition = (window as any).SpeechRecognition || (window as any).webkitSpeechRecognition
  recognition = new SpeechRecognition()
  recognition.grammars = grammar
  recognition.lang = 'en-US'
  recognition.continuous = false
  recognition.interimResults = false
  recognition.onresult = (event: any) => {
    const transcript = event.results[0][0].transcript.toLowerCase()
    if (transcript.includes('team blue')) addPoint('Blue')
    else if (transcript.includes('team red')) addPoint('Red')
    else if (transcript.includes('add blue')) addPoint('Blue')
    else if (transcript.includes('subtract blue')) subtractPoint('Blue')
    else if (transcript.includes('add red')) addPoint('Red')
    else if (transcript.includes('subtract red')) subtractPoint('Red')
    // listening.value = false
  }
  recognition.onerror = (e: any) => {
    console.error('Speech recognition error:', e);
  }
  recognition.onend = () => {
    console.log('Speech recognition ended');
    if (listening.value) {
      recognition.start()
    }
  }
  recognition.start()
  listening.value = true
}

function stopListening() {
  if (recognition) {
    listening.value = false
    recognition.stop()
    console.log('Speech recognition stopped');
  }
}
</script>

<style scoped>
.score-keeper {
  min-height: calc(100vh - 60px);
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: stretch;
}
.teams {
  flex: 1 1 auto;
  display: flex;
  height: 60vh;
}
.team {
  flex: 1 1 50%;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: center;
  text-align: center;
  border-radius: 1rem;
  margin: 1rem;
  box-shadow: 0 2px 8px rgba(0,0,0,0.08);
  &.red {
    background: #ffdddd;
  }
  &.blue {
    background: #ddeaff;
  }
}
.score {
  font-size: 5rem;
  margin: 2rem 0;
  font-weight: bold;
}
.controls {
  display: flex;
  flex-direction: column;
  justify-content: center;
  gap: 2rem;
}
.controls .u-button {
  font-size: 2rem;
  padding: 1.5rem 2.5rem;
  min-width: 120px;
  min-height: 64px;
}
.voice-controls {
  text-align: center;
}
.voice-controls .u-button {
  font-size: 1.5rem;
  padding: 1rem 2rem;
  min-width: 100px;
  min-height: 56px;
}
</style>
