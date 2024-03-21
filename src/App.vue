<script setup>
  import { ref } from 'vue'
  
  class Entry {
    constructor(type, value) {
      this.type = type;
      this.value = value;
    }
  }
  
  var title = ref("");
  var entries = ref([new Entry("a", "av"), new Entry("b", "bv")]);
  var log = ref("");
  var abortController = null;
    
  function onClickRead() {
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
          const message = event.message;
          const decoder = new TextDecoder();
          for (const record of message.records) {
            log.value += "\nRecord type:  " + record.recordType;
            log.value += "\nMIME type:    " + record.mediaType;
            log.value += "\nRecord id:    " + record.id;
            switch (record.recordType) {
              case "text":
                title.value = decoder.decode(record.data);
                break;
              case "url":
                title.value = decoder.decode(record.data);
                break;
              default:
                log.value += "\nRead unsupported record type."
            }
          }
        };
      }).catch(error => {
        log.value += `\nError! Scan failed to start: ${error}.`;
      });
    }
  }

  function onClickWrite() {
    log.value = "write started successfully";
    const ndef = new NDEFReader();
    ndef.write(title.value).then(() => {
      log.value += "\nMessage written.";
    }).catch(error => {
      log.value += `\nWrite failed :-( try again: ${error}.`;
    });
  }

  function addEntry(index) {
    entries.value.splice(index,0,new Entry("", ""));
  }

  function removeEntry(index) {
    entries.value.splice(index,1);
  }

</script>

<template>
  <header>
    <link rel="stylesheet" href="https://cdnjs.cloudflare.com/ajax/libs/font-awesome/4.7.0/css/font-awesome.min.css">
    <img alt="Asterics logo" class="logo" src="./assets/logo.svg" width="125" height="125" />
  </header>

  
  <main>
    <input type="text" id="title" placeholder="Title" v-model="title">
    <i class="fa fa-plus-square-o" @click="addEntry(0)"></i>
    <div id="entries">
      <template v-for="(entry, index) in entries" :key="index">
        <input type="text" :id="'entry_type_'+index" v-model="entries[index].type">
        <input type="text" :id="'entry_value_'+index" v-model="entries[index].value">
        <input type="text" :value="index">
        <i class="fa fa-trash-o" @click="removeEntry(index)"></i>
        <i class="fa fa-plus-square-o" @click="addEntry(index+1)"></i>
      </template>
    </div>
    <div id="buttons">
      <button type="button" @click="onClickRead"><span v-if="abortController!=null">stop </span> read</button>
      <button type="button" @click="onClickWrite">write</button>
    </div>
    <div id ="log">
      <textarea rows="10"  v-model="log"></textarea>
    </div>
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
