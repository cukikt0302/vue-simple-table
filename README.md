# vue-simple-table Vue2+
simple table inline edit

Contain other component:
https://github.com/vmaimone/vue-context-menu
https://github.com/wxsms/uiv

Demo & example https://cukikt0302.github.io/vue-simple-table/index.html

```
dependencies": {
  "uiv": "^0.11.4",
  "vue": "^2.3.3",
  "vue-context-menu": "^2.0.2"
}
```

usage
```
<template>
  <my-table :config="config" :header="header" :data="data"></my-table>
</template>

<script>
import my_table form 'vue-simple-table'
components: {
  'my-table': my_table
}

or
components {
  'my-table': required('patTo/vueimpleTable.vue')
}
</script>
```
