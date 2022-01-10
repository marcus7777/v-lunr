<template>
  <div key="outputHash">
    <slot v-bind:item="item, key" v-for="(item, key) in theOutput">
      {{ item }}
    </slot>
  </div>
</template>

<script>
import lunr from 'lunr'

export default {
  methods:{
    async flattenObj(feilds, obj, parent, res = {}){
      for(let key in feilds){
        let propName = parent || key
        if (typeof obj[key] == 'object') {
          this.flattenObj(obj[key], obj[key], propName, res) // This updates the 'res' but how??
        } else {
          if (res[propName]) {
            res[propName] += " " + (await obj[key])
          } else {
            res[propName] = (await obj[key]) + ""
          }
        }
      }
      return res
    },
    output(search){
      let that = this
      if (this.idx && this.idx.search) {
        return this.idx.search(search).map(function (valu){
          return that.input[+valu.ref]
        }, that)
      }
      if (this.idx.then) return this.idx.then(index => {
        return index.search(search).map(function (valu){
          return that.input[+valu.ref]
        }, that)
      })
      // no index
      return this.input
    },
    makeIndex(hashInput){
      let that = this
      if (this.input[0] && this.search){
        const first = this.fields 
        if (this.log) console.log("feilds from ", first)
        let fieldHash = this.hashThis(JSON.stringify(first))

        if (!this.got[fieldHash]) {
          this.got[fieldHash] = {}
        }
        
        const stopWords = this.stopWords
        const documents = this.input.map(function (val, i){
          return {id: 1,...that.flattenObj(first, val)}
        })
        
        const idx = lunr(function () {
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
            let docHash = that.hashThis(JSON.stringify(doc))
            if (!that.got[fieldHash][docHash]) {
              this.add(doc)
              that.got[fieldHash][docHash] = true
            }
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
    hashFields() {
      return this.hashThis(JSON.stringify(this.fields))
    },
    idx() {
      if (!this.input[0]) return {}
      const loaded = this.cache[this.hashFields]
      if (loaded) {
        let idx = lunr.Index.load(loaded)
        const documents = this.input.map(function (val, i){
          return {id: i, ...this.flattenObj(this.fields, val)}
        })
        if (!this.got(this.fieldHash)){
          this.got[this.fieldHash] = {}
        }
        documents.forEach(doc => {
          let docHash = this.hashThis(JSON.stringify(doc))
          if (!this.got[this.fieldHash][docHash]) {
            idx.add(doc)
            this.got[this.fieldHash][docHash] = true
          }
        })
	return idx
      } else {
        return this.makeIndex(this.hashFields)
      }
    },
  },
  watch:{
    search:{
      handler(search = ""){
        if ((!search.trim() || search == "undefined") && Array.isArray(this.input)) {
          this.$emit("results-length", this.input.length)
          this.$emit("results", this.input.slice(0, this.limit))
          this.theOutput = this.input.slice(0, this.limit)
          return
        }
        const update = output => {
          if (output.then) {
            let that = this
            return output.then(newOutput => {
              const hash = that.hashThis(JSON.stringify(newOutput))
              if (that.outputHash === hash) return 
              that.outputHash = hash
              that.$emit("results-length", newOutput.length)
              that.$emit("results", newOutput.slice(0, that.limit))
              that.theOutput = newOutput.slice(0, that.limit)
            })
          }
          const hash = this.hashThis(JSON.stringify(output))
          if (this.outputHash === hash) return 
          this.outputHash = hash
          try {
            this.$emit("results-length", output.length)
            this.$emit("results", output.slice(0, this.limit))
            this.theOutput = output.slice(0, this.limit)
          } catch(e) {
            console.warn(e, "???", output)
          }
        }
        if (this.output.then) {
          return this.output(search).then(async output => {
            update((await output))
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
    limit: {
      type: Number,
      default: 1000,
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
      got: {},
    }
  },
}

</script>
