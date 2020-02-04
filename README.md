# v-lunr
## Usage
```
    <v-lunr :input="allItems" :search="searchText">
      <template v-slot:default="a">
        <input type="checkbox"> {{ a.item.name }}
      </template>
    </v-lunr>
```
```
    allItems = [
      {name:'A 0', tags:'main'},
      {name:'B 1', tags:''},
      {name:'C 2', tags:'main'},
      {name:'D 3', tags:'recent'},
      {name:'E 4', tags:'main recent'},
      {name:'F 5', tags:'recent current'},
      {name:'G 6', tags:'main recent current'},
    ]
```
## WIP
