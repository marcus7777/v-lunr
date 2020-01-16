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
      return this.input
    },
  },	
  props: {
    input: { 
      default: [{1: "one"},{2: "two"},{3: "three"}],
      required: true,
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
