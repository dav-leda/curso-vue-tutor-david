# Curso Vue.js
## Tutor: David Leda
### Desafío: Formulario con Vue 3

<hr>

Recuerden que este desafío es complementario, o sea, no es obligatorio entregarlo, aunque les recomiendo que lo hagan para familiarizarse con Vue 3.

__1.__ Aunque la consigna no especifica cuál de los desafíos deben convertir a Vue 3 (sólo dice que debe ser a partir de la clase 7) hay solamente dos opciones (el resto fueron entregas del proyecto final): la del formulario sin Vuex o la del formulario con Vuex.

__2. BootstrapVue:__ Si en el desafío del formulario usaron BootstrapVue, con Vue 3 no les va a funcionar. Una opción es usar [BootstrapVue 3](https://cdmoro.github.io/bootstrap-vue-3/). Otra opción es que usen Bootstrap básico, pero en ese caso tienen que crear los templates sin los componentes pre-armados de BootstrapVue. Para ahorrarles trabajo les dejo acá arriba unos templates básicos que pueden copiar y pegar.

__3.__ Si optan por el del formulario __con Vuex__ recomiendo que creen un proyecto en Vue 3 desde cero, para evitar problemas de compatibilidad entre librerías (Vue Router 4, Vuex 4, etc) y luego copien y peguen los templates y los styles (los scripts no los van a necesitar porque en Vue 3 son distintos).

__4. Props y Emits:__ Si optan por el desafío del formulario __sin Vuex__ recuerden que en Vue 3 los `props` y `emits` se declararan usando los métodos `defineProps` y `defineEmits`. No es necesario importar estos dos métodos, aunque puede ser que les de un error de `eslint` por no hacerlo. En ese caso desactiven `eslint` para esa línea de código.

En el componente de la tabla de usuarios:

```js
<script setup>

// eslint-disable-next-line
defineProps({
  users: Array
});

<script>
```

Y en el componente del formulario:

```js
<script setup>

import { reactive } from  'vue'
import useVuelidate from "@vuelidate/core";

const state = reactive({
  user: {
    nombre: '',
    apellido: '',
    edad: '',
    email: ''
  }
});

// Si usan Vuelidate deben importar los métodos de validación
// e incluir el objeto "rules" para luego usarlo en la instancia de Vuelidate:

const v$ = useVuelidate(rules, state);

// eslint-disable-next-line
const emit = defineEmits(['submit'])

const submitForm = async () => {

  // Método de Vuelidate para resetear la interacción
  v$.value.$touch();

  // Si hay errores en el formulario, no enviarlo
  if (v$.value.$invalid) return;

  // Extraer user del objeto reactivo state:
  const { user } = state;

  // Emitir el evento definido arriba al componente padre
  emit('submit', user);
  
  // Limpiar campos del formulario
  Object.keys(user).forEach(key => user[key] = '');

  // Método de Vuelidate para resetear el estado:
  v$.value.$reset();
}

</script>
```

Y luego recibir el evento en el componente padre, pusheando el nuevo  `user` dentro de un array reactivo (`users`). Al hacer el push recuerden usar el spread operator para clonar el objeto y desvincularlo del objeto reactivo `state`, de lo contrario al resetear el formulario el objeto `user` va a quedar vacío, porque reaccion a los cambios del objeto `state`.

```js
<script setup>

import { reactive } from 'vue'

const users = reactive([]);

// Usar el spread operator {...} para pushear el objeto en el array
// De lo contrario, al resetear el formulario el objeto reactivo user quedaría en blanco:
const submit = user => users.push({ ...user });

</script>
```

__5. Variables:__ En Vue 3 no es necesario declarar todas las variables como reactivas (como en el objeto `data` de Vue 2). Las variables que no cambian (por ejemplo, el array de títulos de la tabla) pueden ser declaradas como en JavaScript nativo:

```js
const titles = ['Nombre', 'Apellido', 'Edad', 'Email'];
```

__6. Vuex 4:__ Si optan por usar el del formulario con Vuex, recuerden que en Vuex 4 la store se declara de una forma distinta. Deben importar el método `createStore`:

```js
import { createStore } from 'vuex'

const store = createStore({

  // El resto es igual a Vuex 3

});

export default store
```
__7. Vuex 4:__ Para acceder a los `getters` y `actions` de Vuex 4 en los componentes ya no es posible usar los métodos `mapGetters` y `mapActions` de Vuex 3. Deben hacerlo de esta forma:

```js
<script setup>

import { computed } from 'vue';

// Importar el método useStore de Vuex
import { useStore } from 'vuex'

// Declarar la store
const store = useStore();

// Llamada a la action de Vuex
// No es necesario onMounted() porque setup se ejectuta "on created"
store.dispatch('getUsers')

// Llamada al getter de Vuex, deben usar computed para accederlos:
const users = computed( () => store.getters.users);

</script>
```

<hr>