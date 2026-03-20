<template>
  <div class="taskflow-container">
    <div class="taskflow-wrapper">
      <header class="header">
        <div class="header-content">
          <h1 class="title">
            TaskManager
          </h1>
          <p class="subtitle">Organiza tus tareas académicas de manera inteligente</p>
        </div>
      </header>

      <div class="form-card">
        <h2 class="form-title">
          Nueva Tarea
        </h2>
        
        <div class="form-grid">
          <div class="form-group">
            <label class="form-label">
              Nombre de la tarea:
            </label>
            <input 
              type="text" 
              v-model="nuevaTarea.nombre" 
              class="form-input"
              :class="{ 'input-error': errores.nombre }"
              placeholder="Ej: Entregar informe de investigación"
              @keyup.enter="agregarTarea"
            >
            <transition name="fade">
              <p v-if="errores.nombre" class="error-message">{{ errores.nombre }}</p>
            </transition>
          </div>

          <div class="form-group">
            <label class="form-label">
              Fecha de entrega:
            </label>
            <input 
              type="date" 
              v-model="nuevaTarea.fecha" 
              class="form-input"
              :class="{ 'input-error': errores.fecha }"
              @keyup.enter="agregarTarea"
            >
            <transition name="fade">
              <p v-if="errores.fecha" class="error-message">{{ errores.fecha }}</p>
            </transition>
          </div>

          <div class="form-group">
            <label class="form-label">
              Prioridad:
            </label>
            <select 
              v-model="nuevaTarea.prioridad" 
              class="form-select"
            >
              <option value="alta">Alta Prioridad</option>
              <option value="media">Media Prioridad</option>
              <option value="baja">Baja Prioridad</option>
            </select>
          </div>

          <div class="form-group form-button">
            <button 
              @click="agregarTarea" 
              class="btn-primary"
              :disabled="agregando"
            >
              <span v-if="!agregando">Agregar Tarea</span>
              <span v-else class="loading-spinner">Agregando...</span>
            </button>
          </div>
        </div>
      </div>

      <div class="tasks-card">
        <div class="tasks-header">
          <h2 class="tasks-title">
            Mis Tareas
          </h2>
          <div class="stats-badge">
            <span class="stats-number">{{ tareasPendientes }}</span>
            <span class="stats-text">pendiente(s)</span>
          </div>
        </div>
       
        <transition name="fade">
          <div v-if="tareas.length === 0" class="empty-state">
            <p class="empty-text">¡No hay tareas! Disfruta tu tiempo libre.</p>
            <p class="empty-subtext">Agrega una tarea para comenzar a organizarte</p>
          </div>
        </transition>

   
        <transition-group name="task-list" tag="div" class="tasks-list">
          <div 
            v-for="(tarea, index) in tareas" 
            :key="tarea.id || index" 
            :class="['task-item', `priority-${tarea.prioridad}`, { 'task-completed': tarea.completada }]"
          >
            <div class="task-content">
              <div class="task-info">
                <h3 class="task-title" :class="{ 'completed-text': tarea.completada }">
                  {{ tarea.nombre }}
                </h3>
                <div class="task-meta">
                  <span class="meta-item">
                    {{ formatFecha(tarea.fecha) }}
                  </span>
                  <span class="meta-item">
                    {{ getPrioridadTexto(tarea.prioridad) }}
                  </span>
                  <transition name="pulse">
                    <span v-if="esProxima(tarea.fecha) && !tarea.completada" class="deadline-warning">
                    Próxima a vencer
                    </span>
                  </transition>
                </div>
              </div>
              <div class="task-actions">
                <button 
                  @click="toggleCompletada(index)" 
                  class="btn-complete"
                  :class="{ 'btn-reopen': tarea.completada }"
                >
                  <span v-if="!tarea.completada">Completar</span>
                  <span v-else>Reabrir</span>
                </button>
                <button 
                  @click="eliminarTarea(index)" 
                  class="btn-delete"
                >
                  Eliminar
                </button>
              </div>
            </div>
          </div>
        </transition-group>
      </div>
    </div>
  </div>
</template>

<script setup>
import { ref, computed } from 'vue'

const tareas = ref([])
const agregando = ref(false)

const nuevaTarea = ref({
  nombre: '',
  fecha: '',
  prioridad: 'media',
  completada: false
})

const errores = ref({
  nombre: '',
  fecha: ''
})


const validarFormulario = () => {
  let valido = true
  errores.value = { nombre: '', fecha: '' }

  if (!nuevaTarea.value.nombre.trim()) {
    errores.value.nombre = 'El nombre de la tarea es obligatorio'
    valido = false
  } else if (nuevaTarea.value.nombre.trim().length < 3) {
    errores.value.nombre = 'El nombre debe tener al menos 3 caracteres'
    valido = false
  } else if (nuevaTarea.value.nombre.trim().length > 100) {
    errores.value.nombre = 'El nombre no puede exceder 100 caracteres'
    valido = false
  }

  if (!nuevaTarea.value.fecha) {
    errores.value.fecha = 'La fecha de entrega es obligatoria'
    valido = false
  } else {
    const fechaSeleccionada = new Date(nuevaTarea.value.fecha)
    const hoy = new Date()
    hoy.setHours(0, 0, 0, 0)
    if (fechaSeleccionada < hoy) {
      errores.value.fecha = 'La fecha debe ser mayor a hoy'
      valido = false
    }
  }

  return valido
}


const agregarTarea = async () => {
  console.log('Botón clickeado') 
  
  if (validarFormulario()) {
    agregando.value = true
    
    setTimeout(() => {
      const nuevaTareaObj = {
        id: Date.now(),
        nombre: nuevaTarea.value.nombre.trim(),
        fecha: nuevaTarea.value.fecha,
        prioridad: nuevaTarea.value.prioridad,
        completada: false,
        fechaCreacion: new Date().toISOString()
      }
      
      tareas.value.unshift(nuevaTareaObj)
      
    
      nuevaTarea.value = {
        nombre: '',
        fecha: '',
        prioridad: 'media',
        completada: false
      }
      errores.value = { nombre: '', fecha: '' }
      agregando.value = false
      
      console.log('Tarea agregada:', nuevaTareaObj) 
    }, 300)
  } else {
    console.log('Validación fallida:', errores.value) 
  }
}


const toggleCompletada = (index) => {
  tareas.value[index].completada = !tareas.value[index].completada
}

const eliminarTarea = (index) => {
  if (confirm('¿Estás seguro de eliminar esta tarea?')) {
    tareas.value.splice(index, 1)
  }
}

const tareasPendientes = computed(() => {
  return tareas.value.filter(t => !t.completada).length
})

const formatFecha = (fecha) => {
  if (!fecha) return ''
  const opciones = { year: 'numeric', month: 'long', day: 'numeric' }
  return new Date(fecha).toLocaleDateString('es-ES', opciones)
}

const getPrioridadTexto = (prioridad) => {
  const prioridades = {
    alta: 'Alta prioridad',
    media: 'Media prioridad', 
    baja: 'Baja prioridad'
  }
  return prioridades[prioridad] || prioridad
}

const esProxima = (fecha) => {
  if (!fecha) return false
  const fechaTarea = new Date(fecha)
  const hoy = new Date()
  const diffDias = Math.ceil((fechaTarea - hoy) / (1000 * 60 * 60 * 24))
  return diffDias <= 2 && diffDias >= 0
}
</script>

<style scoped>
.taskflow-container {
  min-height: 100vh;
  background: linear-gradient(135deg, #667eea 0%, #764ba2 100%);
  padding: 2rem 1rem;
}

.taskflow-wrapper {
  max-width: 1200px;
  margin: 0 auto;
}

.header {
  text-align: center;
  margin-bottom: 2rem;
}

.header-content {
  background: rgba(255, 255, 255, 0.95);
  border-radius: 1rem;
  padding: 1.5rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
}

.title {
  font-size: 2.5rem;
  font-weight: bold;
  background: linear-gradient(135deg, #667eea, #764ba2);
  -webkit-background-clip: text;
  background-clip: text;
  color: transparent;
  margin-bottom: 0.5rem;
}

.subtitle {
  font-size: 1rem;
  color: #4a5568;
}

.form-card,
.tasks-card {
  background: white;
  border-radius: 1rem;
  box-shadow: 0 4px 6px rgba(0, 0, 0, 0.1);
  margin-bottom: 1.5rem;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
}

.form-card:hover,
.tasks-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 8px 12px rgba(0, 0, 0, 0.15);
}

.form-title,
.tasks-title {
  font-size: 1.5rem;
  font-weight: 600;
  color: #2d3748;
  padding: 1rem 1.5rem;
  margin: 0;
  background: #f7fafc;
  border-bottom: 2px solid #e2e8f0;
  border-radius: 1rem 1rem 0 0;
}

.form-grid {
  padding: 1.5rem;
  display: grid;
  grid-template-columns: repeat(auto-fit, minmax(280px, 1fr));
  gap: 1rem;
}

.form-group {
  display: flex;
  flex-direction: column;
  gap: 0.5rem;
}

.form-label {
  font-weight: 600;
  color: #2d3748;
  font-size: 0.875rem;
}

.form-input,
.form-select {
  padding: 0.625rem 0.875rem;
  border: 1px solid #e2e8f0;
  border-radius: 0.5rem;
  font-size: 0.875rem;
  transition: all 0.2s ease;
  background: white;
}

.form-input:focus,
.form-select:focus {
  outline: none;
  border-color: #667eea;
  box-shadow: 0 0 0 2px rgba(102, 126, 234, 0.1);
}

.input-error {
  border-color: #f56565;
  background: #fff5f5;
}

.error-message {
  color: #f56565;
  font-size: 0.75rem;
  margin-top: 0.25rem;
}

.btn-primary {
  background: linear-gradient(135deg, #667eea, #764ba2);
  color: white;
  border: none;
  padding: 0.625rem 1rem;
  border-radius: 0.5rem;
  font-size: 0.875rem;
  font-weight: 600;
  cursor: pointer;
  transition: opacity 0.2s ease, transform 0.2s ease;
  width: 100%;
}

.btn-primary:hover {
  opacity: 0.9;
  transform: translateY(-1px);
}

.btn-primary:active {
  transform: translateY(0);
}

.btn-primary:disabled {
  opacity: 0.6;
  cursor: not-allowed;
  transform: none;
}

.tasks-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 1.5rem;
  background: #f7fafc;
  border-bottom: 1px solid #e2e8f0;
}

.stats-badge {
  background: linear-gradient(135deg, #667eea, #764ba2);
  padding: 0.375rem 0.875rem;
  border-radius: 1.5rem;
  color: white;
  font-size: 0.875rem;
  font-weight: 500;
  display: flex;
  align-items: baseline;
  gap: 0.25rem;
}

.stats-number {
  font-size: 1.125rem;
  font-weight: bold;
}

.empty-state {
  text-align: center;
  padding: 2rem;
  color: #a0aec0;
}

.empty-text {
  font-size: 1rem;
  margin-bottom: 0.25rem;
}

.empty-subtext {
  font-size: 0.875rem;
}

.tasks-list {
  padding: 1rem;
  display: flex;
  flex-direction: column;
  gap: 0.75rem;
}

.task-item {
  background: white;
  border-radius: 0.75rem;
  padding: 1rem;
  transition: transform 0.2s ease, box-shadow 0.2s ease;
  border-left: 3px solid;
  box-shadow: 0 1px 2px rgba(0, 0, 0, 0.05);
}

.task-item:hover {
  transform: translateX(2px);
  box-shadow: 0 4px 8px rgba(0, 0, 0, 0.1);
}

.priority-high {
  border-left-color: #f56565;
}

.priority-medium {
  border-left-color: #ed8936;
}

.priority-low {
  border-left-color: #48bb78;
}

.task-completed {
  opacity: 0.6;
  background: #f7fafc;
}

.task-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  gap: 1rem;
  flex-wrap: wrap;
}

.task-info {
  flex: 1;
}

.task-title {
  font-size: 1rem;
  font-weight: 600;
  color: #2d3748;
  margin-bottom: 0.375rem;
}

.completed-text {
  text-decoration: line-through;
  color: #a0aec0;
}

.task-meta {
  display: flex;
  gap: 0.75rem;
  flex-wrap: wrap;
  font-size: 0.75rem;
  color: #718096;
}

.meta-item {
  display: flex;
  align-items: center;
  gap: 0.25rem;
}

.deadline-warning {
  background: #ed8936;
  color: white;
  padding: 0.125rem 0.375rem;
  border-radius: 0.25rem;
  font-size: 0.7rem;
  font-weight: 500;
}

.task-actions {
  display: flex;
  gap: 0.5rem;
}

.btn-complete,
.btn-delete,
.btn-reopen {
  padding: 0.375rem 0.75rem;
  border: none;
  border-radius: 0.375rem;
  font-size: 0.75rem;
  font-weight: 500;
  cursor: pointer;
  transition: opacity 0.2s ease;
}

.btn-complete {
  background: #48bb78;
  color: white;
}

.btn-complete:hover {
  opacity: 0.8;
}

.btn-reopen {
  background: #a0aec0;
  color: white;
}

.btn-reopen:hover {
  opacity: 0.8;
}

.btn-delete {
  background: #f56565;
  color: white;
}

.btn-delete:hover {
  opacity: 0.8;
}

.fade-enter-active,
.fade-leave-active {
  transition: opacity 0.2s ease;
}

.fade-enter-from,
.fade-leave-to {
  opacity: 0;
}

.task-list-enter-active,
.task-list-leave-active {
  transition: all 0.2s ease;
}

.task-list-enter-from {
  opacity: 0;
  transform: translateX(-10px);
}

.task-list-leave-to {
  opacity: 0;
  transform: translateX(10px);
}

@media (max-width: 640px) {
  .task-content {
    flex-direction: column;
    align-items: stretch;
  }
  
  .task-actions {
    justify-content: flex-end;
  }
  
  .tasks-header {
    flex-direction: column;
    gap: 0.5rem;
    text-align: center;
  }
  
  .title {
    font-size: 1.75rem;
  }
}
</style>