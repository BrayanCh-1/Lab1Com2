# Lab1Com2

INTEGRANTES:
- Esmeralda Isabel Cruz Roldán
- Brayan Audiel Chavarría Romero

SITUACIÓN PROBLEMÁTICA:
En la Universidad muchos estudiantes presentan dificultades para organizar sus tareas, actividades y fechas de entrega. Esto ocurre debido a la carga académica, la falta de herramientas adecuadas o el uso de métodos poco eficientes como anotaciones dispersas en papeles o en las notas de celular.

Como consecuencia, los estudiantes pueden olvidar entregas importantes, acumular tareas o no priorizar correctamente sus actividades, lo que afecta su rendimiento académico y genera estrés.

Esta problemática se presenta principalmente en el sector educativo, especialmente en estudiantes de nivel medio y superior que manejan múltiples asignaturas simultáneamente.


PROPUESTA DE SOLUCIÓN:
Se propone el desarrollo de una aplicación web utilizando Vue.js que permita a los estudiantes gestionar sus tareas académicas de manera organizada y sencilla. 
La aplicación permitirá:

- Registrar nuevas tareas con información como nombre, fecha de entrega y nivel de prioridad.
- Visualizar una lista de tareas pendientes.
- Marcar tareas como completadas.
- Eliminar tareas cuando ya no sean necesarias.
- Mostrar alertas visuales cuando una tarea esté próxima a vencer.


FUNCIONES PRINCIPALES DEL SISTEMA:
- Agregar tareas: mediante un formulario con validación de datos.
- Listar tareas: mostrando toda la información ingresada.
- Marcar como completada: cambiar el estado de una tarea.
- Eliminar tareas: quitar tareas de la lista.
- Filtrar o resaltar tareas: por prioridad o cercanía de fecha.



<<<<<< INTERROGANTES >>>>>>


- Explique con sus propias palabras qué es Vue.js y cuál es su función dentro de la página web desarrollada:
Vue.js es como un asistente inteligente que se encarga de mantener sincronizada la pantalla con los datos de nmuestra aplicación
Imaginemos que tenemos por ejemplo una pizarra blanca (en este caso seria la pantalla) y un cuaderno (serian los datos). Normalmente si queremos cambiar algo, tendriamos que borrar de la pizarra y escribir otra cosa manualmente. Con Vue,  solo cambiamos el cuaderno y Vue automáticamente actualiza la pizarra por nosotros.

Función en nuestra página web:

En TaskManager, Vue hace todo el trabajo pesado:
Sincroniza los formularios -> Cuando el usuario escribe en "Nombre de la tarea", Vue guarda ese texto en la variable nuevaTarea.nombre automáticamente.
Actualiza la lista -> Cuando se agrega una tarea, Vue automáticamente muestra la nueva tarea en pantalla sin necesidad de recargar la página.
Responde a los clics -> Cuando el usuario hace clic en "Completar", Vue ejecuta la función correspondiente y actualiza visualmente la tarea (la tacha o la deja normal).
Muestra errores -> Cuando el usuario deja un campo vacío, Vue muestra el mensaje de error solo cuando es necesario.
Sin Vue: Tendría que escribir mucho código JavaScript para buscar elementos HTML, cambiar su contenido, actualizar listas manualmente, etc.
Con Vue: Solo nos preocupamos por los datos y las funciones, Vue se encarga de todo el resto. Es como tener un empleado que hace todo el trabajo repetitivo por mí.



- Describa qué variables reactivas utilizó en su aplicación y cuál es la función de cada una dentro del sistema.
Una variable reactiva es como un botón mágico, cuando cambias su valor, todo lo que depende de ella en la pantalla se actualiza automáticamente.
En nuestra aplicación usamos estas 5 variables reactivas:

1. Tareas
const tareas = ref([])
Guarda la lista completa de todas las tareas que ha creado el usuario.
¿Para qué sirve? Es el CORAZÓN de la aplicación. Cada tarea es un objeto con:

nombre -> el título de la tarea
fecha -> cuándo debe entregarse
prioridad -> alta, media o baja
completada -> si ya se hizo o no
id -> identificador único

Ejemplo de cómo se ve:
tareas = [
  { nombre: "Entregar informe", fecha: "2024-12-10", prioridad: "alta", completada: false, id: 123 },
  { nombre: "Estudiar examen", fecha: "2024-12-15", prioridad: "media", completada: true, id: 456 }
]


2. nuevaTarea
const nuevaTarea = ref({
  nombre: '',
  fecha: '',
  prioridad: 'media',
  completada: false
})

Guarda temporalmente lo que el usuario está escribiendo en el formulario.
¿Para qué sirve? Es como un borrador del formulario. Cada vez que el usuario escribe en un campo, Vue actualiza esta variable. Cuando hace clic en "Agregar", tomo lo que está aquí y lo copio a la lista tareas.


3. errores
const errores = ref({
  nombre: '',
  fecha: ''
})

Guarda los mensajes de error cuando el usuario llena mal el formulario.
¿Para qué sirve? Es como un profesor que revisa la tarea. Si el usuario:

Deja el nombre vacío → guarda "El nombre es obligatorio"
Escribe fecha pasada → guarda "La fecha no puede ser anterior a hoy"
Todo bien → guarda strings vacíos (no muestra errores)
Luego uso v-if para mostrar esos mensajes solo cuando existen.


4. agregando
const agregando = ref(false)

Guarda si en este momento se está agregando una tarea (true) o no (false).
¿Para qué sirve? Mejora la experiencia del usuario. Cuando haces clic en "Agregar":

  1. agregando se pone en true
  2. El botón muestra "Agregando..." y se deshabilita
  3. Después de 0.3 segundos, se agrega la tarea y agregando vuelve a false
  4. El botón vuelve a estar disponible

El beneficio es que el usuario sabe que la acción se está procesando y no puede hacer clic muchas veces por error.


5. tareasPendientes (propiedad computada)
const tareasPendientes = computed(() => {
  return tareas.value.filter(t => !t.completada).length
})

No guarda un valor fijo, sino que calcula automáticamente cuántas tareas NO están completadas.
¿Para qué sirve? Es como un contador automático. Cada vez que:

  1. Agrego una tarea
  2. Marco una como completada
  3. Elimino una tarea
  4. Este número se recalcula SOLO y aparece en el badge que dice "X pendiente(s)".

Es especial porque no necesito actualizarlo manualmente. Vue lo recalcula automáticamente cada vez que cambia tareas. Es como tener una calculadora que siempre muestra el resultado correcto sin que yo tenga que presionar "=".



- Explique la diferencia entre las siguientes directivas utilizadas en su proyecto: v-bind y v-model

v-bind es como un mensaje de ida, este va de JavaScript hacia el HTML. Solo actualiza la vista cuando la variable cambia, pero no al revés. 
v-model es como un chat de dos vías, conecta el input con la variable. Cuando escribimos en el campo, la variable se actualiza y cuando cambiamos la variable, el campo también cambia.


- Mencione al menos un ejemplo de evento utilizado dentro de su aplicación.
Usamos el evento @click en los botones:
<button @click="agregarTarea">Agregar Tarea</button>
<button @click="eliminarTarea(index)">Eliminar</button>
<button @click="toggleCompletada(index)"> Completar</button>

Lo que hace es que cuando el usuario hace clic en el botón, Vue ejecuta la función:

agregarTarea() → crea una nueva tarea
eliminarTarea(index) → borra la tarea
toggleCompletada(index) → marca como completada o la reabre

También usamos @keyup.enter para que al presionar Enter en los inputs también se agregue la tarea:
<input @keyup.enter="agregarTarea">



- Explique para qué utilizó la directiva v-for dentro de su aplicación.
Usamos v-for para mostrar la lista de tareas sin tener que escribir código repetido.
Lo que hace es que Vue recorre automáticamente el arreglo tareas y genera un elemento HTML por cada tarea. Si agregamos o eliminamos tareas, Vue actualiza la lista automáticamente.
El beneficio principal es que código es más limpio, mantenible y funciona con cualquier cantidad de tareas.

Esto hicimos usando v-for:
<div v-for="(tarea, index) in tareas" :key="index">
  {{ tarea.nombre }}
</div>



- Describa en qué situación utilizó v-if y qué problema resuelve dentro de su interfaz.
Usamos v-if en dos situaciones:

1. Mostrar mensaje cuando NO hay tareas:
<div v-if="tareas.length === 0" class="empty-state">
  ¡No hay tareas! Disfruta tu tiempo libre.
</div>

2. Mostrar mensajes de error en el formulario:
<p v-if="errores.nombre" class="error-message">
  {{ errores.nombre }}
</p>

En cuanto a los problemas que resuelve:

En el primer caso, si no hay tareas, muestra un mensaje amigable en lugar de dejar la lista vacía y confusa. El usuario sabe qué está pasando.
En el segundo caso, solo muestra errores cuando realmente hay un problema, evitando que el formulario se vea lleno de mensajes de advertencia todo el tiempo.
Así que podríamos resumir que v-if resuelve el problema de mostrar contenido condicionalmente, haciendo la interfaz más limpia y fácil de entender para el usuario.



- Explique cómo se realiza la validación de datos en su aplicación y por qué es importante validar la información ingresada por el usuario.
Para validad creamos una función llamada validarFormulario() que revisa cada campo antes de agregar una tarea:

const validarFormulario = () => {
  let valido = true
  errores.value = { nombre: '', fecha: '' }

  // Validar NOMBRE
  if (!nuevaTarea.value.nombre.trim()) {
    errores.value.nombre = 'El nombre es obligatorio'
    valido = false
  } else if (nuevaTarea.value.nombre.trim().length < 3) {
    errores.value.nombre = 'Mínimo 3 caracteres'
    valido = false
  }

  // Validar FECHA
  if (!nuevaTarea.value.fecha) {
    errores.value.fecha = 'La fecha es obligatoria'
    valido = false
  } else {
    const fechaSeleccionada = new Date(nuevaTarea.value.fecha)
    const hoy = new Date()
    hoy.setHours(0, 0, 0, 0)
    
    if (fechaSeleccionada < hoy) {
      errores.value.fecha = 'La fecha no puede ser anterior a hoy'
      valido = false
    }
  }

  return valido
}


Luego, antes de agregar la tarea, verrificamos:


const agregarTarea = () => {
  if (validarFormulario()) {
    // Todo bien, agrego la tarea
  } else {
    // Hay errores, NO agrego la tarea
  }
}


¿Por qué es importante validar?

Evitamos datos basura, de esta forma no se guardan tareas sin nombre o con fechas que no existen o no son validas. También mejora la experiencia, puesto que el usuario sabe inmediatamente qué corrigió mal. Asimismo, previene errores, ya que si la fecha es inválida, los cálculos de "próxima a vencer" funcionarían mal. Y mantiene los datos limpios, ya que la aplicación funciona mejor con información correcta

Por ejemplo, si no validaramos, alguien podría agregar una tarea llamada "" (vacía) o con fecha "2020-01-01" (hace 5 años), lo cual no tendría sentido y rompería la lógica de "próxima a vencer".



