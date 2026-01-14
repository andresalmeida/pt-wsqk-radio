<template>
  <div id="app">
    <!-- Hero Section with Video Background -->
    <section class="hero">
      <div class="hero-video">
        <iframe 
          src="https://www.youtube.com/embed/6FbHbF-sg3c?autoplay=1&mute=1&loop=1&playlist=6FbHbF-sg3c&controls=0&showinfo=0&rel=0&modestbranding=1"
          frameborder="0"
          allow="autoplay; encrypted-media"
          allowfullscreen
        ></iframe>
      </div>
      <div class="hero-overlay"></div>
      
      <nav class="nav">
        <button 
          :class="{ active: currentView === 'programas' }" 
          @click="currentView = 'programas'"
        >
          Programas
        </button>
        <button 
          :class="{ active: currentView === 'conductores' }" 
          @click="currentView = 'conductores'"
        >
          Conductores
        </button>
        <button 
          :class="{ active: currentView === 'noticias' }" 
          @click="currentView = 'noticias'"
        >
          Noticias
        </button>
      </nav>
      
      <div class="hero-content">
        <h1 class="hero-title">WSQK</h1>
        <h2 class="hero-subtitle">The Squawk</h2>
        <p class="hero-tagline">Hawkins, Indiana - We turn the volume up to 11</p>
      </div>
    </section>

    <!-- Main Content -->
    <main class="main">
      <Programas v-if="currentView === 'programas'" />
      <Conductores v-if="currentView === 'conductores'" />
      <Noticias v-if="currentView === 'noticias'" />
    </main>

    <!-- Footer -->
    <footer class="footer">
      <p>&copy; 2026 WSQK The Squawk - Hawkins, Indiana</p>
    </footer>
  </div>
</template>

<script>
import Programas from './components/Programas.vue'
import Conductores from './components/Conductores.vue'
import Noticias from './components/Noticias.vue'

export default {
  name: 'App',
  components: {
    Programas,
    Conductores,
    Noticias
  },
  data() {
    return {
      currentView: 'noticias'
    }
  }
}
</script>