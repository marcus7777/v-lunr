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
      let documents = this.input
      let idx = lunr(function () {
        this.ref('name')
        this.field('text')

        documents.forEach(function (doc) {
          this.add(doc)
        }, this)
      })

      return idx.search(this.search)
    },
  },	
  props: {
    input: { 
      default: [{1: "one"},{2: "two"},{3: "three"}],
//      required: true,
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
