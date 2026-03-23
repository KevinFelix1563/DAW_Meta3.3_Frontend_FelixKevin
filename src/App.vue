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
const nuevaTarea = ref('') // Variable reactiva para el input de texto

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
  // Validamos que el campo no esté vacío o solo contenga espacios
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
    
    // Si la petición fue exitosa, limpiamos el campo de texto
    nuevaTarea.value = ''
    
    // Volvemos a pedir la lista completa para ver la nueva tarea reflejada
    await obtenerTareas()

  } catch (error) {
    console.error('Error al crear la tarea:', error)
  }
}

// Al montar el componente, cargamos la lista inicial
onMounted(() => {
  obtenerTareas()
})
</script>