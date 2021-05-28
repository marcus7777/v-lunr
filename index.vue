<template>
  <div>
    <div v-for="(item, key) in theOutput" :key="perpendKey+hashInput+key">
      <slot v-bind:item="item, key">
        {{ item }}
      </slot>
    </div>
  </div>
</template>

<script>
import lunr from 'lunr'

export default {
  methods:{
    output(search){
      let that = this
      if (this.idx && this.idx.search) {
        return this.idx.search(search).map(function (valu){
          return that.input[+valu.ref];
        }, that)
      }
      if (this.idx.then) return this.idx.then(index => {
        return index.search(search).map(function (valu){
          return that.input[+valu.ref];
        }, that)
      })
      // no index
      return this.input
    },
    makeIndex(hashInput){
      let that = this
      if (this.input[0] && this.search){
        const first = this.fields
        if (this.log) console.log("feilds from ",first)

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
          return {__id: i, ...val}
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
        that.cache[hashInput] = idx.toJSON()
        return idx
      } else {
        return {}
      }
    },
    hashThis(message, n = 8) {
      if (+n === 0) return message
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
    hashInput() {
      return this.hashThis(JSON.stringify(this.input))
    },
    idx() {
      if (!this.input[0]) return {}
      const loaded = this.cache[this.hashInput]
      return loaded ? lunr.Index.load(loaded) : this.makeIndex(this.hashInput)
    },
  },
  watch:{
    search:{
      handler(search = ""){
        if (!search.trim() || search == "undefined") {
          this.$emit("results-length", this.input.length)
          this.$emit("results", this.input)
          this.theOutput = this.input
          return
        }
        const update = output => {
          const hash = this.hashThis(JSON.stringify(output))
          if (this.outputHash !== hash) {
            this.outputHash = hash
            this.$emit("results-length", output.length)
            this.$emit("results", output)
            this.theOutput = output
          }
        }
        if (this.output.then) {
          return this.output(search).then(async output => {
            update(await output)
          })
        }
        update(this.output(search))
      },
      immediate: true,
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
    fields: {
      type:Object,
      default(){
        if (typeof this.input[0] === "object"){
          return this.input[0]
        }
        return {}
      }
    },
    perpendKey:{
      type: String,
      default: '',
    },
  },
  data(){
    return {
      outputHash: "",
      theOutput:[],
      cache: {},
    }
  },
}

</script>
