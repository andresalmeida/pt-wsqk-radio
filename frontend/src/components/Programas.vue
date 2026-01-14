<template>
  <section class="programas">
    <h2 class="section-title">Programas</h2>
    
    <p v-if="loading" class="loading">Cargando programas...</p>
    <p v-if="error" class="error">{{ error }}</p>
    
    <div v-if="!loading && !error" class="grid">
      <article v-for="programa in programas" :key="programa.id" class="card">
        <img 
          v-if="getImage(programa)" 
          :src="getImage(programa)" 
          :alt="programa.attributes.title"
          class="card-image"
        />
        <h3>{{ programa.attributes.title }}</h3>
        <p class="card-meta">{{ programa.attributes.field_horario }}</p>
        <p class="card-body">{{ getBody(programa) }}</p>
      </article>
    </div>
  </section>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Programas',
  data() {
    return {
      programas: [],
      included: [],
      loading: true,
      error: null
    }
  },
  async mounted() {
    try {
      const API_URL = import.meta.env.VITE_API_URL || 'http://localhost';
      const response = await axios.get(`${API_URL}/jsonapi/node/programa?include=field_imagen`);
      this.programas = response.data.data
      this.included = response.data.included || []
    } catch (err) {
      this.error = 'Error al cargar programas'
    } finally {
      this.loading = false
    }
  },
  methods: {
    getBody(item) {
      const body = item.attributes.body?.value || ''
      const text = body.replace(/<[^>]*>/g, '')
      return text || 'Sin descripcion'
    },
    getImage(item) {
      const imageRef = item.relationships?.field_imagen?.data
      if (!imageRef) return null
      
      const imageFile = this.included.find(
        inc => inc.type === 'file--file' && inc.id === imageRef.id
      )
      if (!imageFile) return null
      
      const API_URL = import.meta.env.VITE_API_URL || 'http://localhost';
      const imageUrl = imageFile.attributes.uri.url;
      return imageUrl.startsWith('http') ? imageUrl : `${API_URL}${imageUrl}`;
    }
  }
}
</script>