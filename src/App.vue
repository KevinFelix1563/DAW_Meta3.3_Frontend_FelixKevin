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

        <v-card title="Mis Tareas" flat border>
          <v-list>
            <v-list-item v-for="tarea in tareas" :key="tarea.id">
              
              <template v-slot:prepend>
                <v-checkbox-btn :model-value="tarea.completada" readonly></v-checkbox-btn>
              </template>
              
              <v-list-item-title>{{ tarea.titulo }}</v-list-item-title>

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

// Función para consultar las tareas (GET)
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

// Función para enviar la nueva tarea (POST)
const crearTarea = async () => {
  if (!nuevaTarea.value.trim()) return

  try {
    const response = await fetch('http://localhost:3000/api/tareas', {
      method: 'POST',
      headers: {
        'Content-Type': 'application/json'
      },
      body: JSON.stringify({
        titulo: nuevaTarea.value.trim(),
        completada: false
      })
    })

    if (!response.ok) throw new Error(`Error HTTP: ${response.status}`)
    
    nuevaTarea.value = ''
    await obtenerTareas()

  } catch (error) {
    console.error('Error al crear la tarea:', error)
  }
}

// Función para eliminar una tarea (DELETE)
const eliminarTarea = async (id) => {
  try {
    const response = await fetch(`http://localhost:3000/api/tareas/${id}`, {
      method: 'DELETE'
    })

    if (!response.ok) throw new Error(`Error HTTP: ${response.status}`)
    
    // Si se eliminó correctamente en el servidor, refrescamos la vista
    await obtenerTareas()

  } catch (error) {
    console.error('Error al eliminar la tarea:', error)
  }
}

onMounted(() => {
  obtenerTareas()
})
</script>