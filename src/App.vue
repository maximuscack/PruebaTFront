<template>
  <div id="app">
    <header class="bg-primary text-white p-4 mb-4">
      <div class="container">
        <h1 class="text-center">Prueba Técnica - Gestión de Tareas</h1>
        <p class="text-center mb-0">Frontend Vue 3 + Backend Laravel</p>
      </div>
    </header>

    <main class="container">
      <div class="row">
        <div class="col-md-8 offset-md-2">
          <div class="card">
            <div class="card-header">
              <h2>Conexión con Backend Laravel</h2>
            </div>
            <div class="card-body">
              <div class="alert alert-info">
                <h5>🚀 Estado del Proyecto</h5>
                <ul class="mb-0">
                  <li>✅ Backend Laravel configurado</li>
                  <li>✅ Autenticación implementada</li>
                  <li>✅ CRUD de tareas completo</li>
                  <li>✅ Notificaciones por correo configuradas</li>
                  <li>✅ Frontend Vue 3 básico creado</li>
                </ul>
              </div>

              <div class="alert alert-success">
                <h5>📡 API Endpoints Disponibles</h5>
                <p class="mb-2">El backend Laravel está corriendo en: <code>http://localhost:8000</code></p>
                <ul class="mb-0">
                  <li><code>GET /tareas</code> - Listar tareas</li>
                  <li><code>POST /tareas</code> - Crear tarea</li>
                  <li><code>GET /tareas/{id}</code> - Ver tarea</li>
                  <li><code>PATCH /tareas/{id}</code> - Actualizar tarea</li>
                  <li><code>DELETE /tareas/{id}</code> - Eliminar tarea</li>
                  <li><code>PATCH /tareas/{id}/toggle-complete</code> - Marcar completada</li>
                </ul>
              </div>

              <div class="text-center">
                <h5>🔗 Enlaces Rápidos</h5>
                <div class="btn-group" role="group">
                  <a href="http://localhost:8000" target="_blank" class="btn btn-primary">
                    <i class="fas fa-external-link-alt"></i> Backend Laravel
                  </a>
                  <a href="http://localhost:8000/register" target="_blank" class="btn btn-success">
                    <i class="fas fa-user-plus"></i> Registrarse
                  </a>
                  <a href="http://localhost:8000/login" target="_blank" class="btn btn-info">
                    <i class="fas fa-sign-in-alt"></i> Iniciar Sesión
                  </a>
                </div>
              </div>
            </div>
          </div>

          <div class="card mt-4">
            <div class="card-header">
              <h5>🧪 Prueba de Conexión</h5>
            </div>
            <div class="card-body">
              <button @click="testConnection" class="btn btn-primary" :disabled="loading">
                <span v-if="loading" class="spinner-border spinner-border-sm me-2"></span>
                {{ loading ? 'Probando...' : 'Probar Conexión con Backend' }}
              </button>
              
              <div v-if="connectionResult" class="mt-3">
                <div :class="connectionResult.success ? 'alert alert-success' : 'alert alert-danger'">
                  <strong>{{ connectionResult.success ? '✅ Éxito' : '❌ Error' }}</strong>
                  <p class="mb-0">{{ connectionResult.message }}</p>
                </div>
              </div>
            </div>
          </div>
        </div>
      </div>
    </main>

    <footer class="bg-light text-center p-4 mt-5">
      <p class="mb-0">Prueba Técnica Laravel + Vue 3 © 2026</p>
    </footer>
  </div>
</template>

<script>
export default {
  name: 'App',
  data() {
    return {
      loading: false,
      connectionResult: null
    }
  },
  methods: {
    async testConnection() {
      this.loading = true
      this.connectionResult = null
      
      try {
        const response = await fetch('http://localhost:8000/api/test', {
          method: 'GET',
          headers: {
            'Accept': 'application/json',
            'Content-Type': 'application/json'
          }
        })
        
        if (response.ok) {
          const data = await response.json()
          this.connectionResult = {
            success: true,
            message: `Conexión exitosa. Respuesta: ${JSON.stringify(data)}`
          }
        } else {
          this.connectionResult = {
            success: false,
            message: `Error HTTP ${response.status}: ${response.statusText}`
          }
        }
      } catch (error) {
        this.connectionResult = {
          success: false,
          message: `Error de conexión: ${error.message}. Asegúrate de que el backend Laravel esté corriendo en http://localhost:8000`
        }
      } finally {
        this.loading = false
      }
    }
  }
}
</script>

<style>
#app {
  min-height: 100vh;
  display: flex;
  flex-direction: column;
}

main {
  flex: 1;
}

footer {
  margin-top: auto;
}

.card {
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
}

.spinner-border-sm {
  width: 1rem;
  height: 1rem;
}
</style>
