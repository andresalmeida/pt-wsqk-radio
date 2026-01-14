<template>
  <section class="noticias">
    <h2 class="section-title">Noticias de Hawkins</h2>
    
    <p v-if="loading" class="loading">Cargando noticias...</p>
    <p v-if="error" class="error">{{ error }}</p>
    
    <div v-if="!loading && !error" class="news-grid">
      <article 
        v-for="noticia in noticias" 
        :key="noticia.id" 
        class="news-card"
        @click="openModal(noticia)"
      >
        <div 
          class="news-card-bg" 
          :style="{ backgroundImage: getImage(noticia) ? `url(${getImage(noticia)})` : 'none' }"
        ></div>
        <div class="news-card-overlay"></div>
        <div class="news-card-content">
          <h3>{{ noticia.attributes.title }}</h3>
          <p class="news-card-date">{{ formatDate(noticia.attributes.created) }}</p>
          <button class="news-card-btn">Ver mas</button>
        </div>
      </article>
    </div>

    <!-- Modal -->
    <div v-if="selectedNoticia" class="modal-overlay" @click.self="closeModal">
      <div class="modal">
        <button class="modal-close" @click="closeModal">&times;</button>
        <img 
          v-if="getImage(selectedNoticia)" 
          :src="getImage(selectedNoticia)" 
          :alt="selectedNoticia.attributes.title"
          class="modal-image"
        />
        <div class="modal-content">
          <h2 class="modal-title">{{ selectedNoticia.attributes.title }}</h2>
          <p class="modal-date">{{ formatDate(selectedNoticia.attributes.created) }}</p>
          <div class="modal-body" v-html="getBodyHtml(selectedNoticia)"></div>
        </div>
      </div>
    </div>
  </section>
</template>

<script>
import axios from 'axios'

export default {
  name: 'Noticias',
  data() {
    return {
      noticias: [],
      included: [],
      loading: true,
      error: null,
      selectedNoticia: null
    }
  },
  async mounted() {
    try {
      const response = await axios.get('http://localhost/jsonapi/node/noticia?include=field_imagen1')
      this.noticias = response.data.data
      this.included = response.data.included || []
    } catch (err) {
      this.error = 'Error al cargar noticias'
    } finally {
      this.loading = false
    }
  },
  methods: {
    getBodyHtml(item) {
      return item.attributes.body?.processed || item.attributes.body?.value || '<p>Sin contenido</p>'
    },
    formatDate(date) {
      return new Date(date).toLocaleDateString('es-ES', {
        year: 'numeric',
        month: 'long',
        day: 'numeric'
      })
    },
    getImage(item) {
      const imageRef = item.relationships?.field_imagen1?.data
      if (!imageRef) return null
      
      const imageFile = this.included.find(
        inc => inc.type === 'file--file' && inc.id === imageRef.id
      )
      if (!imageFile) return null
      
      return 'http://localhost' + imageFile.attributes.uri.url
    },
    openModal(noticia) {
      this.selectedNoticia = noticia
      document.body.style.overflow = 'hidden'
    },
    closeModal() {
      this.selectedNoticia = null
      document.body.style.overflow = ''
    }
  }
}
</script>