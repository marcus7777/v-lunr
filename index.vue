<template>
  <div>
    output {{output}}
  </div>
</template>

<script>
import lunr from 'lunr';

export default {
  name:  'vLunr',
  components: {
  },
  data: () => {
    return {}
  },
  computed: {
    output: function () {
      if (this.input[0]){
        let documents = this.input.map(function (val, i){
          return Object.assign({__id: i},val)
        })
        let first = this.input[0]
        let idx = lunr(function () {
          this.ref('__id')
          Object.keys(first).forEach(function (key) {
            this.field(key)
          }, this)

          documents.forEach(function (doc) {
            this.add(doc)
          }, this)
        })
        return idx.search(this.search).map(function (val, i){
          return documents[+val.ref]
        })
      } else {
        return []
      }
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
  },
  mounted() {
    
  },
}
</script>
