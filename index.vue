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
        let documents = this.input
        let first = this.input[0]
        let idx = lunr(function () {
          this.ref('name')
          Object.keys(first).forEach(function (key) {
            this.field(key)
          }, this)

          documents.forEach(function (doc) {
            this.add(doc)
          }, this)
	})
        return idx.search(this.search)
      } else {
        return []
      }
    },
  },	
  props: {
    input: { 
      default: function () {
	return [{1: "one"},{2: "two"},{3: "three"}]
      },
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
