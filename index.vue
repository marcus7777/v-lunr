<template>
  <div>{{output}}</div>
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
        const documents = this.input.map(function (val, i){
          return Object.assign({__id: i}, val)
        })
        const first = this.input[0]
        return lunr(function () {
          this.ref('__id')
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
      return this.idx.search(this.search).map(function (valu){
        return this.input[+valu.ref];
      },this)
    }
  },
  watch: {
    search: function(val) {
      alert(val)
      this.update(this.output)
    }
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
      type: Function
    }
  },
  methods: {
    
  },
}

</script>
