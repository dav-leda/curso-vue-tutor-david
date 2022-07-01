# Curso Vue.js
## Tutor: David Leda
### Tercera Entrega del Proyecto Final

<hr>

Les paso algunas recomendaciones para la 3era entrega del proyecto final:

__1.__ Al incorporar Vuex al proyecto ya no es necesario concentrar la lógica global en `App.vue`, con lo cual la mayor parte de los `props` y `emits` pueden eliminarse porque con Vuex se puede acceder al estado global en forma directa desde cualquier componente. 

Por ejemplo, el template de `App.vue` puede ser solo esto:

```html
<template>
  <div>

    <NavBar/>

    <router-view/>

  </div>
</template>
```
Ya no es necesario que `NavBar` y `<router-view>` envíen `props` y reciban `emits`. Con lo cual el script de `App.vue` puede reducirse a esto:

```js
import NavBar from '@/components/NavBar.vue'

export default {
  name: 'App',
  components: {
    NavBar
  } 
}
```

Sin embargo, esto __no significa__ que tienen que __pasar toda la lógica__ que tienen en los `methods` de los componentes a Vuex. Ni __tampoco__ que tienen que usar los `states` de Vuex para __todas las variables reactivas (data)__ que usen en los componentes. Solamente usen los `states` que sean necesarios, o sea, cuando algo en el componente debe cambiar de acuerdo al estado global de la aplicación.

En el caso de la app que estamos haciendo los únicos estados que necesitan los componentes en forma global son los __productos__, el __carrito__ y el __usuario__ que está logueado. Para todo lo demás (formularios, registro, pedidos) no es necesario usar Vuex, pueden ser accedidos o modificados con Axios, o manejados con el estado __local__ del componente que se encuentra en el objeto `data`.

Incluso el listado de productos podría ser manejado sin Vuex, y ser accedido y modificado únicamente con Axios, pero la consigna pide que este listado sea reflejado en el store de Vuex 🤷‍♂️️

__2. Modules:__ Para que el store de Vuex no les quede demasiado largo e ilegible sería conveniente dividir estos tres estados (productos, carrito y usuario) en tres módulos de Vuex distintos. O al menos dos módulos (por ejemplo, uno para carrito y otro para usuario) y dejar la parte central del store para uno de los estados (por ejemplo, productos):

```js
import Vue from 'vue'
import Vuex from 'vuex'
import user from './modules/user'
import cart from './modules/cart'
import apiServices from '@/services/api.services'

Vue.use(Vuex)

const store = new Vuex.Store({

  strict: true,

  state: {
    products: []
  },

  getters: {

    products: state => state.products,

  // etc...

  },
  
  // etc...

  modules: { 
    user,
    cart
  }
});

export default store
```
__3.__ Ya no es necesario que la carga inicial de los productos esté en `App.vue`, puede estar en `HomeView.vue`, utilizando una `action` de Vuex para llamar a la API y un `getter` para obtener el array de productos. Luego pueden usar este array para renderizar las cards en un `v-for`. Para este `v-for` sí es más conveniente usar `props` para pasarle la data del producto a cada card que usar getters de Vuex.

Como en HomeView necesitan una sola `action` y un solo `getter` no es necesario usar `mapActions` y `mapGetters`, pero pueden hacerlo si prefieren.

```js
import ProductCard from '@/components/products/ProductCard.vue'

export default {
  name: 'HomeView',

  components: {
    ProductCard
  },

  created() {
    // Si prefieren pueden usar mapActions dentro de methods
    // y luego acá usar la action de Vuex:
    this.$store.dispatch('getProducts')
  },

  computed: {
    // Si prefieren pueden usar ...mapGetters(['products'])
    products() {
      return this.$store.getters.products
    }
  }
}
```
Y en Vuex:

```js
getProducts: ({ commit }) => {
  apiServices.getProducts()
    .then(products => commit('SET_PRODUCTS', products))
    .catch(err => console.log(err))
},
```

__4. Modules:__ Recuerden que cuando en un componente usen `getters` o `actions` que sean parte de un módulo deben referenciar el módulo por su nombre (o sea, su `namespace`) de esta forma:

```js
methods: {
  // 'user' es el nombre del módulo y 'setUser' es la action
  // que está dentro de ese módulo:
  ...mapActions('user', ['setUser']),
  
  // etc...
}
```
Y si están usando distintos módulos, pueden repetir el método `map` para cada uno:

```js
computed: {
  
  ...mapGetters('user', ['user']),
  ...mapGetters('cart', ['cart', 'cartQty']),

  // etc...
}
```
Siempre que usen `mapGetters` o `mapActions` recuerden __importarlos al comienzo del script__:

```js
import { mapGetters, mapActions } from 'vuex'
```
__5. Mutations:__ Ya que Vuex maneja el estado del carrito, toda la lógica de agregar productos al carrito pasa a formar parte de una mutation. En este caso, además de la data del producto le estoy pasando la cantidad (counter) que viene del componente ProductCounter, pero no es obligatorio hacerlo así:

```js
ADD_TO_CART: (state, { product, counter }) => {

  const inCart = state.cart.find(prod => prod.id === product.id)
  
  if (inCart) {
    inCart.qty = counter;
    inCart.total = inCart.price * counter;
  } else {
    state.cart.push({
      ...product,
      qty: counter,
      total: product.price * counter
    })
  }
},

```


__6. localStorage:__ Recuerden que cada vez que se recarga la página el store se borra. Si por ejemplo están guardando en el store la data del usuario que está logueado, al recargar o cerrar la página el usuario quedaría deslogueado. Si quieren persistir el estado en la recarga pueden usar `localStorage` de esta forma:

```js
// store/modules/user.js
export default {

  namespaced: true,

  state: {
    user: JSON.parse(localStorage.getItem('user')) || null
  },

  getters: {
    user: state => state.user,
  },

  // etc...
}
```

Pero cuando guarden el usuario en `localStorage` recuerden borrar antes el password, ya que guardar passwords en `localStorage` es muy inseguro (cualquier otra persona que use el dispositivo los podría ver, además de que es posible hackear `localStorage`):

```js
mutations: {

  SET_USER: (state, user) => {
    if (user) {
      // Ojo: no guardar el password en localStorage
      delete user.password;
      state.user = user;
      localStorage.setItem('user', JSON.stringify(user));
    } else {
      state.user = null;
      localStorage.removeItem('user');
    }
  }
},

actions: {
  setUser: ({ commit }, user) => {
    commit('SET_USER', user)
  }
}
```
De todas formas, tengan en cuenta que __en una app real los datos del usuario nunca se guardan en localStorage__. Lo que se suele hacer es generar desde el Backend un token encriptado con [JWT](https://blog.logrocket.com/how-to-implement-jwt-authentication-vue-nodejs/) que es enviado al Front en los headers de la petición HTTP y luego guardar este token en `localStorage`. Luego, en cada petición del Front al Back este token es adjuntado a los headers y el Back debe validar el token antes de responder a la petición. 

Pero incluso esta opción no es del todo segura ya que `localStorage` puede ser hackeado y el token podría ser robado. La opción más segura (aunque más complicada de implementar que JWT, sobre todo cuando Front y Back no comparten el mismo dominio) es usar [Cookies](https://medium.com/developer-rants/session-cookies-between-express-js-and-vue-js-with-axios-98a10274fae7) del tipo [HTTP only](https://geekflare.com/es/enable-cors-httponly-cookie-secure-token/).

__7. Mutations:__ Por convención el nombre de las mutations se pone en mayúsculas, pero no es obligatorio hacerlo así (de hecho, en la documentación oficial de Vuex no usan mayúsculas para las mutations):

```js
EMPTY_CART: (state) => {
  state.cart = [];
  localStorage.removeItem('cart');
},
```

__8.__ Aunque existe la posibilidad de acceder al estado del store en forma directa usando `mapState` es recomendable hacerlo usando `getters` (recuerden declarar el getter en Vuex antes de usarlo en el componente):

```js
...mapGetters('user', ['user']),
```
Lo que nunca deben hacer es modificar el `state` en forma directa desde un componente. Aunque Vuex permite hacerlo, incluso en modo `strict` (tira un mensaje de error, pero el state es mutado igual), si lo hacen el uso de Vuex pierde sentido porque tendrían distintos estados según el componente en que estén.

__9.__ Este último punto es opcional: sería bueno que además de mostrar la información de los productos en una serie de cards dentro de HomeView, al cliquear en una card el usuario sea dirigido a una `view` (ProductView) donde vea el producto en detalle (en la mayoría de los e-commerce es así). 

Para hacer esto tienen 3 opciones: pasarle los datos del producto desde la card a la `view` en los `params` de `<router-link>` y en ProductView declarar esto en las `props`:

```html
<router-link 
  :to="{ 
    name: 'product', 
    params: { 
      id: product.id,
      product 
    } 
  }" 
  class="btn btn-outline-light router-button"
>Ver producto</router-link>
```
Y en ProductView:

```js
export default {
  name: 'ProductView',

  props: {
    id: {
      type: String,
      required: true
    },
    product: {
      type: Object
    }
  },
  // etc...
}
```

Otra opción es usar en ProductView un `getter` de Vuex al que se le pase por parámetro el `id` del producto que es generado en la ruta dinámica (`this.$route.params.id`):

```js
export default {
  name: 'ProductView',

  data () {
    return {
      productId: this.$route.params.id
    }
  },

  computed: {

    ...mapGetters(['getProductById']),

    product () {
      return this.getProductById(this.productId)
    },
    // etc...
  },
  // etc...
}
```
Y en Vuex:

```js
getters: {

  products: state => state.products,

  getProductById: state => id => {
    return state.products.find(product => product.id === id)
  }   
},
```

Y una tercera opción es realizar una petición a la API en ProductView para obtener el producto por su ID en caso de que el getter de Vuex no devuelva nada (porque el store de productos quedó vacío al recargar la página):

```js
created () {
  this.getProduct()
},

data () {
  return {
    product: null,
    productId: this.$route.params.id
  }
},

computed:
  // getter de Vuex para obtener el producto por su ID
  ...mapGetters(['getProductById']),
},

methods: {

  async getProduct() {
    // Primero intentar obtener el producto con el getter de Vuex:
    this.product = this.getProductById(this.productId)
    // Si esto falla (porque el store de productos fue vaciado al recargar la página)
    // obtener el producto de la API:
    if (!this.product) {
      this.product = await apiServices.getProductById(this.productId)
    }
  },
  // etc...
}
```
Lo bueno de esta última opción es que si el usuario recarga la página, o comparte la URL, el producto se sigue mostrando, mientras que si es accedido por un getter de Vuex al recargar la página la data del producto se pierde ya que en cada recarga el store de Vuex vuelve al estado inicial.

Lo malo de esta opción es que la petición a la API puede demorarse, y esta demora puede generar un error al intentar renderizar el template cuando la data aún no llegó. Para evitar este error pueden poner un `v-if` al comienzo del template, lo cual impide que el template sea renderizado antes de tiempo:

```html
<template>
  <div v-if="product" class="container mt-5">
```

<hr>

