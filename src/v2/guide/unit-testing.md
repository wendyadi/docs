---
title: Unit Testing
type: guide
order: 402
---

> [Vue CLI](https://cli.vuejs.org/) memiliki opsi built-in untuk melakukan unit testing menggunakan [Jest](https://github.com/facebook/jest) atau [Mocha](https://mochajs.org/) yang bisa berjalan tanpa setup tambahan. Kami juga memiliki library resmi [Vue Test Utils](https://vue-test-utils.vuejs.org/) yang menyediakan tuntunan lebih detail untuk custom setups.


## Pengujian/Assertion Sederhana

Anda tidak perlu melakukan hal khusus untuk membuat komponen-komponen Anda testable (dapat dites). Export raw options:

``` html
<template>
  <span>{{ message }}</span>
</template>

<script>
  export default {
    data () {
      return {
        message: 'hello!'
      }
    },
    created () {
      this.message = 'bye!'
    }
  }
</script>
```

Kemudian import options komponen tersebut bersama dengan Vue, dan Anda dapat membuat banyak assertion umum (disini kita menggunakan Jasmine/Jest style `expect` assertion sebagai contoh)


``` js
// Import vue dan komponen yang akan dites
import Vue from 'vue'
import MyComponent from 'path/to/MyComponent.vue'

// Berikut adalah Jasmine 2.0 tests, Anda dapat
// menggunakan test runner / library assertion apa pun yang Anda sukai
describe('MyComponent', () => {

  // Periksa raw options komponen
  it('has a created hook', () => {
    expect(typeof MyComponent.created).toBe('function')
  })

  // Evaluasi hasil dari function-function di
  // raw options komponen
  it('sets the correct default data', () => {
    expect(typeof MyComponent.data).toBe('function')
    const defaultData = MyComponent.data()
    expect(defaultData.message).toBe('hello!')
  })

  // Periksa instance dari komponen saat mount
  it('correctly sets the message when created', () => {
    const vm = new Vue(MyComponent).$mount()
    expect(vm.message).toBe('bye!')
  })

  // Mount sebuah instance dan periksa hasil render
  it('renders the correct message', () => {
    const Constructor = Vue.extend(MyComponent)
    const vm = new Constructor().$mount()
    expect(vm.$el.textContent).toBe('bye!')
  })
})
```

## Tulis Komponen yang Dapat Dites

Hasil render dari sebuah komponen secara utama ditentukan oleh properti-properti yang diterima komponen tersebut. Jika hasil render dari sebuah komponen hanya bergantung pada properti-nya -- maka akan mudah untuk mengetesnya, mirip dengan melakukan assertion pada fungsi yang memiliki bermacam-macam argumen. Berikut contoh sederhana:

``` html
<template>
  <p>{{ msg }}</p>
</template>

<script>
  export default {
    props: ['msg']
  }
</script>
```

Anda dapat melakukan assertion pada hasil render dengan properti yang berbeda0-beda menggunakan opsi `propsData`:

``` js
import Vue from 'vue'
import MyComponent from './MyComponent.vue'

// fungsi helper / pembantu yang melakukan mounts kemudian me-return text yang di-render
function getRenderedText (Component, propsData) {
  const Constructor = Vue.extend(Component)
  const vm = new Constructor({ propsData: propsData }).$mount()
  return vm.$el.textContent
}

describe('MyComponent', () => {
  it('renders correctly with different props', () => {
    expect(getRenderedText(MyComponent, {
      msg: 'Hello'
    })).toBe('Hello')

    expect(getRenderedText(MyComponent, {
      msg: 'Bye'
    })).toBe('Bye')
  })
})
```

## Menguji/Melakukan Assertion pada Perubahan Asynchronous

Karena Vue [melakukan update pada DOM secara asynchronous](reactivity.html#Async-Update-Queue), assertion pada update DOM yang disebabkan oleh perubahan state harus dilakukan dalam callback `Vue.nextTick`:

``` js

// Periksa HTML yang dihasilkan setelah perubahan state
it('updates the rendered message when vm.message updates', done => {
  const vm = new Vue(MyComponent).$mount()
  vm.message = 'foo'

  // Tunggu satu tick setelah perubahan state sebelum menguji DOM yang terupdate
  Vue.nextTick(() => {
    expect(vm.$el.textContent).toBe('foo')
    done()
  })
})
```

Untuk informasi yang lebih mendalam untuk unit testing di Vue, kunjungi [Vue Test Utils](https://vue-test-utils.vuejs.org/) dan cookboook entry kami tentang [unit testing vue components](https://vuejs.org/v2/cookbook/unit-testing-vue-components.html)