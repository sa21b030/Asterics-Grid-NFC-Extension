<script setup>
import { ref } from 'vue'
import HelloWorld from './components/HelloWorld.vue'
import TheWelcome from './components/TheWelcome.vue'

var msg = ref("")
var text = ref("")
var log = ref("")
var isURL = ref(false)




   
    function onClickRead() {
      const ndef = new NDEFReader();
      ndef.scan().then(() => {
        log.value = "Scan started successfully.";
        ndef.onreadingerror = () => {
          log.value += "\nCannot read data from the NFC tag. Try another one?";
        };
        ndef.onreading = event => {
          const message = event.message;
          const decoder = new TextDecoder();
          for (const record of message.records) {
           // const record = message.records;
            log.value += "\nRecord type:  " + record.recordType;
            log.value += "\nMIME type:    " + record.mediaType;
            log.value += "\nRecord id:    " + record.id;
            switch (record.recordType) {
              case "text":
                isURL.value = false;
                msg.value = decoder.decode(record.data);
                break;
              case "url":
                isURL.value = true;
                msg.value = decoder.decode(record.data);
                break;
              default:
                // TODO: Handle other records with record data.
            }
          }
        };
      }).catch(error => {
        log.value += `\nError! Scan failed to start: ${error}.`;
      });
    }
    function onClickWrite() {
      log.value = "write started successfully";
      const ndef = new NDEFReader();
      ndef.write(
        isURL.value ? { records: [{recordType: "text", data: msg.value},{recordType: "url", data: }] } : msg.value
      ).then(() => {
        log.value += "\nMessage written.";
      }).catch(error => {
        log.value += `\nWrite failed :-( try again: ${error}.`;
      });
    }
  

</script>

<template>
  <header>
    <img alt="Vue logo" class="logo" src="./assets/logo.svg" width="125" height="125" />

    <div class="wrapper">
      <HelloWorld msg="You did it!" />
    </div>
  </header>

  
  <main>
    <input type="checkbox" id="is_url" v-model="isURL">URL</input>
    <input type="text" placeholder="NFC message" v-model="msg">
    <input type="text" placeholder="NFC message" v-model="text">
    <button type="button" @click="onClickRead">read</button>
    <button type="button" @click="onClickWrite">write</button>
    <textarea rows="10"  v-model="log"></textarea>
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
