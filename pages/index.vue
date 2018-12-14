<template>
  <section class="container" id="main-container">
    <div>
      <logo/>
      <h1 class="title">
        THESIS
      </h1>
      <h2 class="subtitle">
        "Human emotional state monitoring based on algorithms of facial expression recognition"
      </h2>
      <h3>Mykola Nesov</h3>
      <div class="links">
        <a
          href="/demo"
          class="btn-primary btn btn-lg">Demo</a>
        <a
          href="https://github.com/nuxt/nuxt.js"
          target="_blank"
          class="btn-outline-secondary btn btn-lg">GitHub</a>
      </div>
    </div>
  </section>
</template>

<script>
import Logo from '~/components/Logo.vue'

export default {
  components: {
    Logo
  }
}
</script>

<style>

.container {
  min-height: 100vh;
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.title {
  font-family: 'Quicksand', 'Source Sans Pro', -apple-system, BlinkMacSystemFont,
    'Segoe UI', Roboto, 'Helvetica Neue', Arial, sans-serif;
  display: block;
  font-weight: 300;
  font-size: 100px;
  color: #35495e;
  letter-spacing: 1px;
}

.subtitle {
  font-weight: 300;
  font-size: 42px;
  color: #526488;
  word-spacing: 5px;
  padding-bottom: 15px;
}

.links {
  padding-top: 15px;
}
</style>
