# v-lunr
## Usage
```
    <v-lunr :input="allItems" :search="searchText">
      <template v-slot:default="a">
        <input type="checkbox"> {{ a.item.name }}
      </template>
    </v-lunr>
```
## WIP
