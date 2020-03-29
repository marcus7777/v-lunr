# v-lunr
### A import for vue.js of  [LUNR](https://lunrjs.com/)

## Install 
```yarn add v-lunr ```


## Usage
```html
  <template>
    <v-lunr :input="allItems" :search="searchText">
      <template v-slot="{item}">
        <input type="checkbox"> {{ item.name }}
      </template>
    </v-lunr>
  </template> 
  <script>
    import vLunr from "v-lunr"
    export default {
      components: {
        vLunr,
      },
      data(){
        return {
          allItems: [
            {name:'A 0', tags:'main'},
            {name:'B 1', tags:''}, 
            {name:'C 2', tags:'main'},
            {name:'D 3', tags:'recent'},
            {name:'E 4', tags:'main recent'},
            {name:'F 5', tags:'recent current'},
            {name:'G 6', tags:'main recent current'},
          ],
        }
      },
    }
  </script>
```
## Props:

#### deep 
  `Boolean` - Searches within objects
#### input
  `Array` of objects - to search
#### search
  `String` - that you are searching for
#### stopWords 
  `Boolean` - Search little word
