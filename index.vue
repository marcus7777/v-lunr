<template>
  <div>
    <div v-for="(item, i) in output" :key="i">
      <slot v-bind:item="item">
        {{ item }}
      </slot>
    </div>
  </div>
</template>

<script>
import lunr from 'lunr';

export default {
  name:  'vLunr',
  data: () => {
    return {
    }
  },
  computed:{
    idx: function () {
      if (this.input[0]){
        const first = this.input[0]
        const stopWords = this.stopWords
        const documents = this.input.map(function (val, i){
          let doc = {}
          Object.keys(val).forEach(function(key) {
            doc[key] = JSON.stringify(val[key])
          })
          return Object.assign({__id: i}, val)
        })
        
        return lunr(function () {
          this.ref('__id')
          if (!stopWords) {
            this.pipeline.remove(lunr.stopWordFilter)
            this.pipeline.remove(lunr.stemmer)
          }
          Object.keys(first).forEach(function (key) {
            this.field(key)
          }, this)
          documents.forEach(function (doc) {
            this.add(doc)
          }, this)
        })
      } else {
        return {}
      }
    },
    output: function(){
      const output = this.idx.search(this.search).map(function (valu){
        return this.input[+valu.ref];
      },this)
      return output
    },
    outputAsText: function(){
      return JSON.stringify(this.output)
    },
  },
  props: {
    input: {
      type: Array,
    },
    search:{
      type: String,
      default: '',
    },
    update:{
      type: String
    },
    stopWords: {
      type: Boolean,
      default: false,
    }
  },
}

</script>
