<template>
  <div id="app">
    <header class="app-header">
      <div class="container">
        <div class="header-content">
          <h1 class="app-title">📋 Gestión de Tareas</h1>
          <div class="user-menu" v-if="user">
            <span class="user-name">👤 {{ user.name }}</span>
            <button @click="logout" class="btn btn-outline-danger btn-sm">Cerrar Sesión</button>
          </div>
        </div>
      </div>
    </header>

    <main class="main-content">
      <div class="container">
        <!-- Login/Register -->
        <div v-if="!user" class="auth-container">
          <div class="auth-card">
            <h2>{{ isLogin ? '🔐 Iniciar Sesión' : '📝 Registrarse' }}</h2>
            <form @submit.prevent="isLogin ? login() : register()">
              <!-- Login Form -->
              <div v-if="isLogin">
                <div class="mb-3">
                  <label class="form-label">Email</label>
                  <input type="email" v-model="authForm.email" class="form-control" required>
                </div>
                <div class="mb-3">
                  <label class="form-label">Contraseña</label>
                  <input type="password" v-model="authForm.password" class="form-control" required>
                </div>
              </div>
              
              <!-- Register Form -->
              <div v-else>
                <div class="mb-3">
                  <label class="form-label">Nombre</label>
                  <input type="text" v-model="authForm.name" class="form-control" required>
                </div>
                <div class="mb-3">
                  <label class="form-label">Email</label>
                  <input type="email" v-model="authForm.email" class="form-control" required>
                </div>
                <div class="mb-3">
                  <label class="form-label">Contraseña</label>
                  <input type="password" v-model="authForm.password" class="form-control" required>
                </div>
                <div class="mb-3">
                  <label class="form-label">Confirmar Contraseña</label>
                  <input type="password" v-model="authForm.password_confirmation" class="form-control" required>
                </div>
              </div>
              
              <button type="submit" class="btn btn-primary w-100">
                {{ isLogin ? 'Iniciar Sesión' : 'Registrarse' }}
              </button>
            </form>
            <div class="mt-3 text-center">
              <button @click="switchAuthMode" class="btn btn-link">
                {{ isLogin ? '¿No tienes cuenta? Regístrate' : '¿Ya tienes cuenta? Inicia sesión' }}
              </button>
            </div>
          </div>
        </div>

        <!-- Tasks Management -->
        <div v-else class="tasks-container">
          <div class="tasks-header">
            <h2>Mis Tareas</h2>
            <button @click="showCreateModal = true" class="btn btn-primary">
              ➕ Nueva Tarea
            </button>
          </div>

          <!-- Filters -->
          <div class="filters-section">
            <div class="row">
              <div class="col-md-3">
                <select v-model="filters.order_by" @change="fetchTasks" class="form-select">
                  <option value="fecha_vencimiento">📅 Fecha Vencimiento</option>
                  <option value="titulo">📝 Título</option>
                  <option value="created_at">🕐 Fecha Creación</option>
                </select>
              </div>
              <div class="col-md-3">
                <select v-model="filters.order_direction" @change="fetchTasks" class="form-select">
                  <option value="asc">⬆️ Ascendente</option>
                  <option value="desc">⬇️ Descendente</option>
                </select>
              </div>
              <div class="col-md-6">
                <input type="text" v-model="filters.search" @input="fetchTasks" 
                       placeholder="🔍 Buscar tareas..." class="form-control">
              </div>
            </div>
          </div>

          <!-- Tasks List -->
          <div class="tasks-list">
            <div v-if="loading" class="text-center py-4">
              <div class="spinner"></div>
              <p class="mt-2">Cargando tareas...</p>
            </div>
            
            <div v-else-if="tasks.length === 0" class="empty-state">
              <div class="empty-icon">📋</div>
              <h3>No tienes tareas aún</h3>
              <p>Crea tu primera tarea para comenzar</p>
            </div>

            <div v-else class="tasks-grid">
              <div v-for="task in tasks" :key="task.id" class="task-card" 
                   :class="{ 'completed': task.completado, 'overdue': isOverdue(task) }">
                <div class="task-header">
                  <h4>{{ task.titulo }}</h4>
                  <div class="task-badges">
                    <span v-if="task.completado" class="badge success">✅ Completada</span>
                    <span v-else class="badge warning">⏳ Pendiente</span>
                    <span v-if="isOverdue(task)" class="badge danger">⚠️ Vencida</span>
                  </div>
                </div>
                
                <p class="task-description">
                  {{ task.descripcion || 'Sin descripción' }}
                </p>
                
                <div class="task-meta">
                  <div class="task-date">
                    📅 {{ formatDate(task.fecha_vencimiento) }}
                  </div>
                </div>
                
                <div class="task-actions">
                  <button @click="toggleComplete(task)" 
                          class="btn btn-sm" 
                          :class="task.completado ? 'btn-secondary' : 'btn-success'">
                    {{ task.completado ? '↩️ Reabrir' : '✅ Completar' }}
                  </button>
                  <button @click="editTask(task)" class="btn btn-sm btn-warning">
                    ✏️ Editar
                  </button>
                  <button @click="deleteTask(task)" class="btn btn-sm btn-danger">
                    🗑️ Eliminar
                  </button>
                </div>
              </div>
            </div>
          </div>

          <!-- Pagination -->
          <div v-if="pagination.last_page > 1" class="pagination-container">
            <button @click="changePage(pagination.current_page - 1)" 
                    :disabled="pagination.current_page === 1" 
                    class="btn btn-outline-primary">
              ⬅️ Anterior
            </button>
            <span class="page-info">
              Página {{ pagination.current_page }} de {{ pagination.last_page }}
            </span>
            <button @click="changePage(pagination.current_page + 1)" 
                    :disabled="pagination.current_page === pagination.last_page" 
                    class="btn btn-outline-primary">
              Siguiente ➡️
            </button>
          </div>
        </div>
      </div>
    </main>

    <!-- Create/Edit Modal -->
    <div v-if="showCreateModal || showEditModal" class="modal-overlay" @click="closeModals">
      <div class="modal-content" @click.stop>
        <div class="modal-header">
          <h3>{{ showCreateModal ? '➕ Crear Nueva Tarea' : '✏️ Editar Tarea' }}</h3>
          <button @click="closeModals" class="btn-close">&times;</button>
        </div>
        
        <form @submit.prevent="showCreateModal ? createTask() : updateTask()">
          <div class="mb-3">
            <label class="form-label">📝 Título</label>
            <input type="text" v-model="taskForm.titulo" class="form-control" required>
          </div>
          
          <div class="mb-3">
            <label class="form-label">📄 Descripción</label>
            <textarea v-model="taskForm.descripcion" class="form-control" rows="3"></textarea>
          </div>
          
          <div class="mb-3">
            <label class="form-label">📅 Fecha y Hora de Vencimiento</label>
            <input type="datetime-local" v-model="taskForm.fecha_vencimiento" class="form-control" required>
          </div>
          
          <div class="modal-footer">
            <button type="button" @click="closeModals" class="btn btn-secondary">Cancelar</button>
            <button type="submit" class="btn btn-primary">
              {{ showCreateModal ? 'Crear' : 'Actualizar' }}
            </button>
          </div>
        </form>
      </div>
    </div>
  </div>
</template>

<script>
import axios from 'axios'

export default {
  name: 'App',
  data() {
    return {
      user: null,
      isLogin: true,
      showCreateModal: false,
      showEditModal: false,
      loading: false,
      tasks: [],
      pagination: {
        current_page: 1,
        last_page: 1,
        per_page: 10,
        total: 0
      },
      filters: {
        order_by: 'fecha_vencimiento',
        order_direction: 'asc',
        search: ''
      },
      authForm: {
        name: '',
        email: '',
        password: '',
        password_confirmation: ''
      },
      taskForm: {
        id: null,
        titulo: '',
        descripcion: '',
        fecha_vencimiento: '',
        completado: false
      }
    }
  },
  mounted() {
    this.checkAuth()
  },
  methods: {
    async checkAuth() {
      const token = localStorage.getItem('token')
      if (token) {
        axios.defaults.headers.common['Authorization'] = `Bearer ${token}`
        try {
          const response = await axios.get('http://localhost:8000/api/user')
          this.user = response.data
          this.fetchTasks()
        } catch (error) {
          localStorage.removeItem('token')
          delete axios.defaults.headers.common['Authorization']
        }
      }
    },
    
    async login() {
      try {
        const response = await axios.post('http://localhost:8000/api/login', this.authForm)
        localStorage.setItem('token', response.data.token)
        axios.defaults.headers.common['Authorization'] = `Bearer ${response.data.token}`
        this.user = response.data.user
        this.fetchTasks()
      } catch (error) {
        alert('❌ Error al iniciar sesión')
      }
    },
    
    async register() {
      // Validar que las contraseñas coincidan
      if (this.authForm.password !== this.authForm.password_confirmation) {
        alert('❌ Las contraseñas no coinciden')
        return
      }
      
      try {
        const response = await axios.post('http://localhost:8000/api/register', this.authForm)
        localStorage.setItem('token', response.data.token)
        axios.defaults.headers.common['Authorization'] = `Bearer ${response.data.token}`
        this.user = response.data.user
        this.fetchTasks()
      } catch (error) {
        if (error.response && error.response.status === 422) {
          // Errores de validación
          const errors = error.response.data.errors
          let errorMessage = '❌ Error al registrarse:\n'
          for (const field in errors) {
            errors[field].forEach(msg => {
              errorMessage += `- ${msg}\n`
            })
          }
          alert(errorMessage)
        } else {
          alert('❌ Error al registrarse')
        }
      }
    },
    
    logout() {
      localStorage.removeItem('token')
      delete axios.defaults.headers.common['Authorization']
      this.user = null
    },
    
    switchAuthMode() {
      this.isLogin = !this.isLogin
      // Limpiar formulario
      this.authForm = {
        name: '',
        email: '',
        password: '',
        password_confirmation: ''
      }
    },
    
    async fetchTasks() {
      this.loading = true
      try {
        const params = {
          page: this.pagination.current_page,
          order_by: this.filters.order_by,
          order_direction: this.filters.order_direction,
          search: this.filters.search
        }
        
        const response = await axios.get('http://localhost:8000/api/tareas', { params })
        this.tasks = response.data.data
        this.pagination = {
          current_page: response.data.current_page,
          last_page: response.data.last_page,
          per_page: response.data.per_page,
          total: response.data.total
        }
      } catch (error) {
        console.error('Error fetching tasks:', error)
      } finally {
        this.loading = false
      }
    },
    
    async createTask() {
      try {
        await axios.post('http://localhost:8000/api/tareas', this.taskForm)
        this.closeModals()
        this.fetchTasks()
      } catch (error) {
        alert('❌ Error al crear tarea')
      }
    },
    
    async updateTask() {
      try {
        await axios.put(`http://localhost:8000/api/tareas/${this.taskForm.id}`, this.taskForm)
        this.closeModals()
        this.fetchTasks()
      } catch (error) {
        alert('❌ Error al actualizar tarea')
      }
    },
    
    async deleteTask(task) {
      if (confirm('¿Estás seguro de eliminar esta tarea?')) {
        try {
          await axios.delete(`http://localhost:8000/api/tareas/${task.id}`)
          this.fetchTasks()
        } catch (error) {
          alert('❌ Error al eliminar tarea')
        }
      }
    },
    
    async toggleComplete(task) {
      try {
        await axios.patch(`http://localhost:8000/api/tareas/${task.id}/toggle-complete`)
        this.fetchTasks()
      } catch (error) {
        alert('❌ Error al cambiar estado de tarea')
      }
    },
    
    editTask(task) {
      this.taskForm = {
        id: task.id,
        titulo: task.titulo,
        descripcion: task.descripcion,
        fecha_vencimiento: task.fecha_vencimiento.replace(' ', 'T'),
        completado: task.completado
      }
      this.showEditModal = true
    },
    
    closeModals() {
      this.showCreateModal = false
      this.showEditModal = false
      this.taskForm = {
        id: null,
        titulo: '',
        descripcion: '',
        fecha_vencimiento: '',
        completado: false
      }
    },
    
    changePage(page) {
      if (page >= 1 && page <= this.pagination.last_page) {
        this.pagination.current_page = page
        this.fetchTasks()
      }
    },
    
    isOverdue(task) {
      return !task.completado && new Date(task.fecha_vencimiento) < new Date()
    },
    
    formatDate(dateString) {
      const date = new Date(dateString)
      return date.toLocaleString('es-ES', {
        day: '2-digit',
        month: '2-digit',
        year: 'numeric',
        hour: '2-digit',
        minute: '2-digit'
      })
    }
  }
}
</script>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif;
  background: linear-gradient(135deg, #f5f7fa 0%, #c3cfe2 100%);
  min-height: 100vh;
  color: #2c3e50;
}

#app {
  min-height: 100vh;
  background-color: #f8f9fa;
}

.app-header {
  background: white;
  border-bottom: 1px solid #e9ecef;
  box-shadow: 0 2px 4px rgba(0,0,0,0.1);
  position: sticky;
  top: 0;
  z-index: 1000;
}

.header-content {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1rem 0;
}

.app-title {
  color: #2c3e50;
  font-size: 1.5rem;
  font-weight: 600;
  margin: 0;
}

.user-menu {
  display: flex;
  align-items: center;
  gap: 1rem;
}

.user-name {
  color: #6c757d;
  font-weight: 500;
}

.main-content {
  padding: 2rem 0;
}

.container {
  max-width: 1200px;
  margin: 0 auto;
  padding: 0 1rem;
}

.auth-container {
  display: flex;
  justify-content: center;
  align-items: center;
  min-height: calc(100vh - 200px);
}

.auth-card {
  background: white;
  padding: 2rem;
  border-radius: 12px;
  box-shadow: 0 10px 30px rgba(0,0,0,0.1);
  width: 100%;
  max-width: 400px;
}

.auth-card h2 {
  color: #2c3e50;
  margin-bottom: 1.5rem;
  text-align: center;
}

.tasks-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  margin-bottom: 2rem;
}

.tasks-header h2 {
  color: #2c3e50;
  font-size: 2rem;
  font-weight: 600;
}

.filters-section {
  background: white;
  padding: 1.5rem;
  border-radius: 12px;
  box-shadow: 0 2px 10px rgba(0,0,0,0.05);
  margin-bottom: 2rem;
}

.tasks-list {
  min-height: 400px;
}

.empty-state {
  text-align: center;
  padding: 4rem 2rem;
  color: #6c757d;
}

.empty-icon {
  font-size: 4rem;
  margin-bottom: 1rem;
  opacity: 0.5;
}

.empty-state h3 {
  color: #2c3e50;
  margin-bottom: 0.5rem;
}

.tasks-grid {
  display: grid;
  grid-template-columns: repeat(auto-fill, minmax(350px, 1fr));
  gap: 1.5rem;
}

.task-card {
  background: white;
  border-radius: 12px;
  padding: 1.5rem;
  box-shadow: 0 2px 10px rgba(0,0,0,0.05);
  border-left: 4px solid #007bff;
  transition: transform 0.2s, box-shadow 0.2s;
}

.task-card:hover {
  transform: translateY(-2px);
  box-shadow: 0 4px 20px rgba(0,0,0,0.1);
}

.task-card.completed {
  opacity: 0.7;
  border-left-color: #28a745;
}

.task-card.overdue {
  border-left-color: #dc3545;
}

.task-header {
  display: flex;
  justify-content: space-between;
  align-items: flex-start;
  margin-bottom: 1rem;
}

.task-header h4 {
  color: #2c3e50;
  font-size: 1.1rem;
  font-weight: 600;
  margin: 0;
  flex: 1;
}

.task-badges {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.task-description {
  color: #6c757d;
  margin-bottom: 1rem;
  line-height: 1.5;
}

.task-meta {
  margin-bottom: 1rem;
}

.task-date {
  color: #6c757d;
  font-size: 0.9rem;
  display: flex;
  align-items: center;
  gap: 0.5rem;
}

.task-actions {
  display: flex;
  gap: 0.5rem;
  flex-wrap: wrap;
}

.modal-overlay {
  position: fixed;
  top: 0;
  left: 0;
  right: 0;
  bottom: 0;
  background: rgba(0,0,0,0.5);
  display: flex;
  justify-content: center;
  align-items: center;
  z-index: 2000;
}

.modal-content {
  background: white;
  border-radius: 12px;
  padding: 0;
  width: 90%;
  max-width: 500px;
  max-height: 90vh;
  overflow-y: auto;
}

.modal-header {
  display: flex;
  justify-content: space-between;
  align-items: center;
  padding: 1.5rem;
  border-bottom: 1px solid #e9ecef;
}

.modal-header h3 {
  color: #2c3e50;
  margin: 0;
}

.btn-close {
  background: none;
  border: none;
  font-size: 1.5rem;
  cursor: pointer;
  color: #6c757d;
}

.modal-header .btn-close:hover {
  color: #2c3e50;
}

.modal-content form {
  padding: 1.5rem;
}

.modal-footer {
  display: flex;
  gap: 1rem;
  justify-content: flex-end;
  margin-top: 1.5rem;
  padding-top: 1.5rem;
  border-top: 1px solid #e9ecef;
}

.pagination-container {
  display: flex;
  justify-content: center;
  align-items: center;
  gap: 1rem;
  margin-top: 2rem;
}

.page-info {
  color: #6c757d;
  font-weight: 500;
}

.btn {
  padding: 0.5rem 1rem;
  border: none;
  border-radius: 6px;
  cursor: pointer;
  font-weight: 500;
  transition: all 0.2s;
  text-decoration: none;
  display: inline-flex;
  align-items: center;
  gap: 0.5rem;
}

.btn-primary {
  background: #007bff;
  color: white;
}

.btn-primary:hover {
  background: #0056b3;
}

.btn-success {
  background: #28a745;
  color: white;
}

.btn-success:hover {
  background: #1e7e34;
}

.btn-warning {
  background: #ffc107;
  color: #212529;
}

.btn-warning:hover {
  background: #e0a800;
}

.btn-danger {
  background: #dc3545;
  color: white;
}

.btn-danger:hover {
  background: #c82333;
}

.btn-secondary {
  background: #6c757d;
  color: white;
}

.btn-secondary:hover {
  background: #545b62;
}

.btn-outline-primary {
  background: transparent;
  color: #007bff;
  border: 1px solid #007bff;
}

.btn-outline-primary:hover {
  background: #007bff;
  color: white;
}

.btn-outline-danger {
  background: transparent;
  color: #dc3545;
  border: 1px solid #dc3545;
}

.btn-outline-danger:hover {
  background: #dc3545;
  color: white;
}

.btn-sm {
  padding: 0.25rem 0.5rem;
  font-size: 0.875rem;
}

.btn-link {
  background: none;
  border: none;
  color: #007bff;
  cursor: pointer;
  text-decoration: underline;
}

.btn-link:hover {
  color: #0056b3;
}

.form-control {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ced4da;
  border-radius: 6px;
  font-size: 1rem;
}

.form-control:focus {
  outline: none;
  border-color: #007bff;
  box-shadow: 0 0 0 0.2rem rgba(0,123,255,0.25);
}

.form-select {
  width: 100%;
  padding: 0.5rem;
  border: 1px solid #ced4da;
  border-radius: 6px;
  font-size: 1rem;
  background: white;
}

.form-label {
  display: block;
  margin-bottom: 0.5rem;
  color: #2c3e50;
  font-weight: 500;
}

.badge {
  padding: 0.25rem 0.5rem;
  border-radius: 4px;
  font-size: 0.75rem;
  font-weight: 600;
  text-transform: uppercase;
}

.success {
  background: #28a745;
  color: white;
}

.warning {
  background: #ffc107;
  color: #212529;
}

.danger {
  background: #dc3545;
  color: white;
}

.spinner {
  width: 2rem;
  height: 2rem;
  border: 0.25rem solid #f3f3f3;
  border-top: 0.25rem solid #007bff;
  border-radius: 50%;
  animation: spin 1s linear infinite;
  margin: 0 auto;
}

@keyframes spin {
  0% { transform: rotate(0deg); }
  100% { transform: rotate(360deg); }
}

@media (max-width: 768px) {
  .tasks-grid {
    grid-template-columns: 1fr;
  }
  
  .tasks-header {
    flex-direction: column;
    gap: 1rem;
    align-items: stretch;
  }
  
  .filters-section .row {
    gap: 1rem;
  }
  
  .filters-section .col-md-4,
  .filters-section .col-md-6 {
    flex: 1 1 100%;
  }
  
  .task-actions {
    justify-content: center;
  }
  
  .pagination-container {
    flex-direction: column;
    gap: 1rem;
  }
}
</style>
