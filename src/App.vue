<template>
  <v-app>
    <v-main>
      <v-container max-width="600">
        
        <v-card class="mb-4" flat border>
          <v-card-text>
            <v-form @submit.prevent="crearTarea">
              <v-row align="center">
                <v-col cols="8" sm="9">
                  <v-text-field
                    v-model="nuevaTarea"
                    label="Escribe una nueva tarea..."
                    variant="outlined"
                    density="compact"
                    hide-details
                  ></v-text-field>
                </v-col>
                <v-col cols="4" sm="3">
                  <v-btn type="submit" color="primary" block>
                    Agregar
                  </v-btn>
                </v-col>
              </v-row>
            </v-form>
          </v-card-text>
        </v-card>

        <v-text-field
          v-model="terminoBusqueda"
          label="Buscar tarea..."
          variant="outlined"
          density="compact"
          prepend-inner-icon="mdi-magnify"
          clearable
          hide-details
          class="mb-4"
          @input="buscarTareas"
          @click:clear="obtenerTareas"
        ></v-text-field>

        <v-card title="Mis Tareas" flat border>
          <v-list>
            <v-list-item v-for="tarea in tareas" :key="tarea.id">
              
              <template v-slot:prepend>
                <v-checkbox-btn 
                  v-model="tarea.completada" 
                  @change="actualizarEstadoTarea(tarea)"
                ></v-checkbox-btn>
              </template>
              
              <v-list-item-title>
                {{ tarea.titulo }}
              </v-list-item-title>

              <template v-slot:append>
                <v-btn
                  icon="mdi-delete"
                  variant="text"
                  color="error"
                  density="comfortable"
                  @click="eliminarTarea(tarea.id)"
                ></v-btn>
              </template>

            </v-list-item>

            <v-list-item v-if="tareas.length === 0">
              <v-list-item-title class="text-center text-medium-emphasis">
                No se encontraron tareas.
              </v-list-item-title>
            </v-list-item>
          </v-list>
        </v-card>

      </v-container>
    </v-main>
  </v-app>
</template>

<script setup>
import { ref, onMounted } from 'vue'

const tareas = ref([])
const nuevaTarea = ref('')
const terminoBusqueda = ref('') // NUEVO: Estado para el buscador

// GET: Consultar todas las tareas
const obtenerTareas = async () => {
  try {
    const response = await fetch('http://localhost:3000/api/tareas')
    if (!response.ok) throw new Error(`Error HTTP: ${response.status}`)
    const resultado = await response.json()
    tareas.value = resultado.data
  } catch (error) {
    console.error('Error al cargar tareas:', error)
  }
}

// NUEVO - GET: Buscar tareas por título
const buscarTareas = async () => {
  const termino = terminoBusqueda.value.trim()
  
  // Si el campo está vacío, volvemos a cargar todas las tareas
  if (!termino) {
    await obtenerTareas()
    return
  }

  try {
    const response = await fetch(`http://localhost:3000/api/tareas/buscar?q=${termino}`)
    if (!response.ok) throw new Error(`Error HTTP: ${response.status}`)
    const resultado = await response.json()
    tareas.value = resultado.data
  } catch (error) {
    console.error('Error al buscar tareas:', error)
  }
}

// POST: Enviar una nueva tarea
const crearTarea = async () => {
  if (!nuevaTarea.value.trim()) return

  try {
    const response = await fetch('http://localhost:3000/api/tareas', {
      method: 'POST',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        titulo: nuevaTarea.value.trim(),
        completada: false
      })
    })

    if (!response.ok) throw new Error(`Error HTTP: ${response.status}`)
    
    nuevaTarea.value = ''
    
    // Si había una búsqueda activa, la limpiamos para mostrar la nueva tarea en la lista completa
    if (terminoBusqueda.value) {
      terminoBusqueda.value = ''
    }
    
    await obtenerTareas()

  } catch (error) {
    console.error('Error al crear la tarea:', error)
  }
}

// DELETE: Eliminar una tarea
const eliminarTarea = async (id) => {
  try {
    const response = await fetch(`http://localhost:3000/api/tareas/${id}`, {
      method: 'DELETE'
    })

    if (!response.ok) throw new Error(`Error HTTP: ${response.status}`)
    
    // Si estamos filtrando, volvemos a aplicar el filtro, sino cargamos todas
    if (terminoBusqueda.value.trim()) {
      await buscarTareas()
    } else {
      await obtenerTareas()
    }

  } catch (error) {
    console.error('Error al eliminar la tarea:', error)
  }
}

// PATCH: Actualizar el estado completado/pendiente
const actualizarEstadoTarea = async (tarea) => {
  try {
    const response = await fetch(`http://localhost:3000/api/tareas/${tarea.id}`, {
      method: 'PATCH',
      headers: { 'Content-Type': 'application/json' },
      body: JSON.stringify({
        completada: tarea.completada
      })
    })

    if (!response.ok) throw new Error(`Error HTTP: ${response.status}`)

  } catch (error) {
    console.error('Error al actualizar la tarea:', error)
    tarea.completada = !tarea.completada
  }
}

onMounted(() => {
  obtenerTareas()
})
</script>