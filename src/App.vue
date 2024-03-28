/* Links
https://upload.wikimedia.org/wikipedia/commons/2/25/Adult_Male_Lion.jpg
http://www.tunghai74.org/forums/movie/MGM-Lion-Roar.mp3
https://de.wikipedia.org/wiki/L%C3%B6we
Links end */

<script setup>
  import { ref } from 'vue'
  
  class Entry {
    constructor(type, value) {
      this.type = type;
      this.value = value;
    }
  }
  
  var title = ref("");
  var entries = ref([]);
  var log = ref("");
  var abortController = null;
  var modeProgram = ref(true);
  var audio = null;

  var store = "";  //for testing only

  const typeDomain = "asterics.eu";
  const typeSelections = ["speech", "sound", "image", "info", "link"];
  const typeSpeech = "speech";
  const typeSound = "sound";
  const typeImage = "image";
  const typeInfo = "info";
  const typeLink = "link";
  const typeDefault = typeImage;

  const languageGerman = "de-DE";
    
  function onClickRead() {
    /* for testing only */
    decodeNDEFMessage(store);
    return;
    /* */
    if (abortController!=null) { //read is active
      abortController.abort();
      abortController = null;
    }
    else {
      abortController = new AbortController();
      const ndef = new NDEFReader();
      ndef.scan({ signal: abortController.signal }).then(() => {
        log.value = "Scan started successfully.";
        ndef.onreadingerror = () => {
          log.value += "\nCannot read data from the NFC tag. Try another one?";
        };
        ndef.onreading = event => {
          decodeNDEFMessage(event.message);
        };
      }).catch(error => {
        log.value += `\nError! Scan failed to start: ${error}.`;
      });
    }
  }

  function onClickWrite() {
    /* for testing only */
    store = structuredClone(encodeNDEFMessage());
    return;
    /* */
    log.value = "write started successfully";
    const ndef = new NDEFReader();
    ndef.write(encodeNDEFMessage()).then(() => {
      log.value += "\nMessage written.";
    }).catch(error => {
      log.value += `\nWrite failed :-( try again: ${error}.`;
    });
  }

  function onClickChangeMode() {
    modeProgram.value = !modeProgram.value;
    if (!modeProgram.value) {
      if (abortController==null) onClickRead();
      const sounds = entries.value.filter((item)=>item.type==typeSound);
      if (sounds.length > 0) {
        if (audio == null) 
          audio = new Audio();
        console.log("should play: "+sounds[0].value);
        audio.src = sounds[0].value;
        audio.play();
      }
      const textsToSpeeech = entries.value.filter((item)=>item.type==typeSpeech);
      if (textsToSpeeech.length > 0) {
        let utterance = new SpeechSynthesisUtterance(textsToSpeeech[0].value);
        utterance.lang = languageGerman;
        speechSynthesis.speak(utterance);
      }
    }
  }

  function addEntry(index) {
    entries.value.splice(index,0,new Entry(typeDefault, ""));
  }

  function removeEntry(index) {
    entries.value.splice(index,1);
  }

  function encodeNDEFMessage() {
    const encoder = new TextEncoder();
    const records = [{ recordType: "text", data: encoder.encode(title.value) }];
    for (const entry of entries.value)
      records.push({ recordType: typeDomain+":"+entry.type, data: encoder.encode(entry.value), mediaType: "" });
    return { records: records };
  }

  function decodeNDEFMessage(message) {
    const decoder = new TextDecoder();
    let first = true;
    for (const record of message.records) {
      if (first) {
        entries.value = [];
        if (record.recordType != "text") {
          log.value += "\nMessage decoding failed: first entry must have recordType \"text\"";
          title.value = "";
          return;
        }
        title.value = decoder.decode(record.data);
        first = false;
      }
      else {
        if (!record.recordType.startsWith(typeDomain+":"))
          log.value += "\nMessage decoding failed: entries must have recordType in asterics domain";
        else
          entries.value.push(new Entry(record.recordType.substr(typeDomain.length+1), decoder.decode(record.data)));
      }
    }
  }

</script>

<template>
  <header>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">
    <img alt="Asterics logo" class="logo" src="./assets/logo.png" width="125" height="125" />
  </header>

  
  <main>
    <div id="program" v-if="modeProgram">
      <input type="text" id="title" placeholder="Title" v-model="title">
      <i class="fa fa-plus-square-o" @click="addEntry(0)"></i>
      <div id="entries">
        <template v-for="(entry, index) in entries" :key="index">
          <div id="'entry_'+index">
            <select :id="'entry_type_'+index" v-model="entries[index].type">
              <template v-for="theType in typeSelections" :key="theType">
                <option :value="theType">{{ theType }}</option>
              </template>
            </select>
            <input type="text" :id="'entry_value_'+index" placeholder="Value" v-model="entries[index].value">
            <i class="fa fa-trash-o" @click="removeEntry(index)"></i>
            <i class="fa fa-plus-square-o" @click="addEntry(index+1)"></i>
          </div>
        </template>
      </div>
      <div id="buttons">
        <button type="button" @click="onClickRead"><span v-if="abortController!=null">stop </span> read</button>
        <button type="button" @click="onClickWrite">write</button>
      </div>
      <div id ="log">
        <textarea rows="10"  v-model="log"></textarea>
      </div>
    </div>
    <div id="play" v-if="!modeProgram">
      <div id="playTitle">
        <h1>{{ title }}</h1>
      </div>
      <div id="playImage">
        <template v-for="(entry, index) in entries.filter((item)=>item.type==typeImage)" :key="index">
          <img :src="entry.value">
        </template>
      </div>
      <div id="playText">
        <template v-for="(entry, index) in entries.filter((item)=>item.type==typeInfo)" :key="index">
          <p> {{ entry.value }} </p>
        </template>
      </div>
      <div id="playLink">
        <template v-for="(entry, index) in entries.filter((item)=>item.type==typeLink)" :key="index">
          <a :href="entry.value" target="_blank">Mehr {{ index+1}}</a>
        </template>
      </div>
    </div>
    <button type="button" class="btn-bottom-right" @click="onClickChangeMode()">{{ modeProgram ? "play" : "program"}}</button>
  </main>
</template>

<script>

</script>

<style scoped>
header {
  line-height: 1.5;
}

.logo {
  display: block;
  margin: 0 auto 2rem;
}

.btn-bottom-right {
  position: fixed;
  bottom: 3px;
  right: 3px;
}

@media (min-width: 1024px) {
  header {
    display: flex;
    place-items: center;
    padding-right: calc(var(--section-gap) / 2);
  }

  .logo {
    margin: 0 2rem 0 0;
  }

  header .wrapper {
    display: flex;
    place-items: flex-start;
    flex-wrap: wrap;
  }
}
</style>
