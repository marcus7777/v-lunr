<template>
  <div>
    <div v-for="(item, key) in output" :key="key">
      <slot v-bind:item="item, key">
        {{ item }}
      </slot>
    </div>
  </div>
</template>

<script>
import lunr from 'lunr';

export default {
  methods:{
    hashThis(message, n = 8) {
      if (n === 0) return message
      let hash = 0
      if (message.length == 0) return this.hashThis(this.hex32(hash),n-1)
      for (let i = 0;i < message.length;i++) {
        const char = message.charCodeAt(i)
        hash = ((hash<<5)-hash)+char
        hash = hash & hash // Convert to 32bit integer
      }
      return this.hashThis(this.hex32(hash),n-1)
    },
    hex32(val) {
      val &= 0xFFFFFFFF;
      var hex = val.toString(16).toUpperCase();
      return ("00000000" + hex).slice(-8);
    },
  },
  computed:{
    idx() {
      const that = this
      const hashInput = this.hashThis(JSON.stringify(this.input))
      const loaded = localStorage.getItem("lunr"+hashInput)

      if (loaded){
        return lunr.Index.load(JSON.parse(loaded))
      } else if (this.input[0]){
        const first = this.input[0]
        if (this.log) console.log(first)
        const stopWords = this.stopWords
        const documents = this.input.map(function (val, i){
          let doc = {}
          let seen = []
          function replacer(key, value) {
            if (key[0] === '_') {
              return
            } else if (typeof value === 'object' && value !== null) {
              if (seen.indexOf(key) !== -1) return;
              else seen.push(key);
            }
            return value;
          }
          Object.keys(val).forEach(function(key) {
            doc[key] = JSON.stringify(val[key], replacer)
          })
          return Object.assign({__id: i}, val)
        })
        const idx = lunr(function () {
          this.ref('__id')
          if (!stopWords) {
            this.pipeline.remove(lunr.stopWordFilter)
            this.pipeline.remove(lunr.stemmer)
          }
          Object.keys(first).forEach(function (key) {
            if (key[0] !== '_') {
              this.field(key)
            }
          }, this)
          documents.forEach(function (doc) {
            this.add(doc)
          }, this)
          if (that.log) console.log(this)
        })
        localStorage.setItem("lunr"+hashInput, JSON.stringify(idx) )
        return idx
      } else {
        return {}
      }
    },
    words(){
      if (this.idx.invertedIndex) {
        return Object.keys(this.idx.invertedIndex)
      } else {
        return []
      }
    },
    output(){
      this.$emit("words", this.words)
      if (this.search.trim()) {
        const output = this.idx.search(this.search).map(function (valu){
          return this.input[+valu.ref];
        },this)
        this.$emit("results-length", output.length)
        this.$emit("results", output)
        return output
      } else {
        this.$emit("results-length", 0)
        this.$emit("results", this.input)
        return this.input
      }
    },
    outputAsText(){
      return JSON.stringify(this.output)
    },
  },
  props: {
    input: {
      type: Array,
      required: true,
    },
    search:{
      type: String,
      default: '',
    },
    stopWords: {
      type: Boolean,
      default: false,
    },
    log: Boolean,
  },
}

</script>
