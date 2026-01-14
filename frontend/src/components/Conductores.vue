<template>
  <section class="conductores">
    <h2 class="section-title">Conductores</h2>
    
    <p v-if="loading" class="loading">Cargando conductores...</p>
    <p v-if="error" class="error">{{ error }}</p>
    
    <div v-if="!loading && !error" class="grid">
      <article v-for="conductor in conductores" :key="conductor.id" class="card">
        <img 
          v-if="getImage(conductor)" 
          :src="getImage(conductor)" 
          :alt="conductor.attributes.title"
          class="card-image"
        />
        <h3>{{ conductor.attributes.title }}</h3>
        <p class="card-body">{{ getBody(conductor) }}</p>
        <p v-if="conductor.attributes.field_redes_sociales" class="card-meta">
          {{ conductor.attributes.field_redes_sociales }}
        </p>
      </article>
    </div>
  </section>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Conductores',
  data() {
    return {
      conductores: [],
      included: [],
      loading: true,
      error: null
    }
  },
  async mounted() {
    try {
      const API_URL = import.meta.env.VITE_API_URL || 'http://localhost';
      const response = await axios.get(`${API_URL}/jsonapi/node/conductor?include=field_foto`);
      this.conductores = response.data.data
      this.included = response.data.included || []
    } catch (err) {
      this.error = 'Error al cargar conductores'
    } finally {
      this.loading = false
    }
  },
  methods: {
    getBody(item) {
      const body = item.attributes.body?.value || ''
      const text = body.replace(/<[^>]*>/g, '')
      return text || 'Sin biografia'
    },
    getImage(item) {
      const imageRef = item.relationships?.field_foto?.data
      if (!imageRef) return null
      
      const imageFile = this.included.find(
        inc => inc.type === 'file--file' && inc.id === imageRef.id
      )
      if (!imageFile) return null
      
      return 'http://localhost' + imageFile.attributes.uri.url
    }
  }
}
</script>