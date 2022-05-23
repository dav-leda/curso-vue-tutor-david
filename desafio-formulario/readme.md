# Curso Vue.js
## Tutor: David Leda
### Desafío: Formulario

<br>

__1.__ Para las validaciones pueden usar la librería que prefieran: [VueForm](https://www.npmjs.com/package/vue-form), [Vuelidate](https://vuelidate-next.netlify.app), [VeeValidate](https://vee-validate.logaretm.com/v4/) o crear sus propias validaciones usando Regular Expressions de JavaScript.

__2.__ En el objeto `data` del formulario sería mejor declarar cada propiedad dentro de un objeto que tener datos sueltos:

```js
data: () => ({

  form: {
    nombre: '',
    apellido: '',
    edad: '',
    email: '',
    cursos: [],
    beca: ''
  }
}),
```

__3.__ No es necesario que usen un objeto JSON para guardar los datos, ya que para eso deberían hacer un post a una API, algo que todavía no vimos. Para la carga de datos pueden hacer un `$emit` desde el componente del formulario.

__4.__ No les recomiendo que pongan el componente de la tabla como hijo del componente del formulario. Estos dos componentes deberían ser independientes ya que en una app real solamente el admin tendría acceso a la tabla de todos los usuarios, no cada usuario que carga el formulario. Sería mejor que pongan ambos componentes en App.vue, y que App.vue reciba los datos emitidos por el formulario y los agregue a un array vacío en el objeto data (`users: []`). Luego ese array puede pasarse como `prop` al componente de la tabla.

__5.__ No les recomiendo que usen filtros de vista (`filters`) para mostrar los datos en el formato correcto en la tabla. Sería mejor guardar los datos ya formateados, y no sólo mostrarlos en el formato correcto y guardarlos en el incorrecto. Por ejemplo, si un usuario escribe su nombre con minúsculas, no usar un filtro en la tabla para mostrar el nombre con la primera letra en mayúscula:

```js
filters: {
  capitalize(value) {
    return value.replace(/\b\w/g, l => l.toUpperCase())
  }
}    
```
Sería mejor usar un method en el componente del formulario y emitir el formulario con el formato correcto. Y antes de emitir el formulario no olviden chequear que todos los campos estén en el formato correcto, de lo contrario hacer un `window.scroll` hacia el formulario para que el usuario vuelva a ver los mensajes de error:

```js
emitForm () {
  if (!this.alert.nombre && !this.alert.email && !this.alert.edad) {
  
  this.$emit('submit-form', {
    nombre:  this.capitalize(this.form.nombre),
    apellido: this.capitalize(this.form.apellido),
    edad: this.form.edad,
    email: this.form.email.toLowerCase(),
    cursos: this.form.cursos,
    beca: this.form.beca
  });

  this.resetForm();

  } else {
    // Si hay errores, scroll hacia arriba para que el usuario los vea:
    window.scroll(0,0)
  }
},
```
__6.__ Para resetear los campos del formulario pueden vaciarlos uno por uno o usar la propiedad `Object.keys` de JavaScript:

```js
Object.keys(this.form).forEach(key => this.form[key] = '');
// Excepto la propiedad cursos que debe ser un array:
this.form.cursos = [];
```
__7.__ Les paso algunas Regular Expressions que les pueden ser útiles:

```js
// Para el nombre completo (minimo 2 palabras, minimo 2 caracteres cada una):

const fullNameRegExp = /[a-zA-Z]{2,}\s+[a-zA-Z]{2,12}/g;

// Para chequear que no haya <script> tags en el texto ingresado (muy importante para evitar hackeos):

const scriptRegExp = /<script[\s\S]*?>[\s\S]*?<\/script>/gi;

// Para el formato de e-mail (es un chequeo mas completo que el que hace poniendo type="email" en el template):

const emailRegExp = /^\w+([.-]?\w+)*@\w+([.-]?\w+)*(\.\w{2,3})+$/;
```

Y luego para testearlas:

```js
this.alert.email = emailRegExp.test(this.form.email) ? '' : 'Formato de email inválido.';
```
<hr>
